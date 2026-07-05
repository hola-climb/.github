# 외부 API 활용 내용 정리

> Hola(올라) — AI 동작 분석 기반 클라이밍 영상 SNS
> 본 문서는 프로젝트에서 사용한 외부 API/서비스의 용도, 연동 방식, 보안 처리를 정리한다.

---

## 요약

| # | 외부 서비스 | 분류 | 용도 | 적용 영역 |
|---|---|---|---|---|
| 1 | GCP Cloud Storage | 스토리지 | 영상 업로드/재생 (Signed URL) | API 서버 |
| 2 | Google OAuth 2.0 | 인증 | 구글 소셜 로그인 | API 서버 |
| 3 | Kakao OAuth 2.0 | 인증 | 카카오 소셜 로그인 | API 서버 |
| 4 | Naver OAuth 2.0 | 인증 | 네이버 소셜 로그인 | API 서버 |
| 5 | Kakao Maps JS SDK | 지도 | 암장 지도·탐색·검색 | 웹/모바일 |
| 6 | SMTP (메일) | 메일 | 회원가입 이메일 인증 | API 서버 |
| 7 | Firebase Cloud Messaging | 푸시 | 알림 푸시 | API 서버 / 모바일 |
| 8 | OpenAI API | 생성형 AI | 월간 리포트 서술 생성 | API 서버 |
| 9 | Capacitor Geolocation | 디바이스 | 채팅 GPS 암장 인증 | 모바일 |

> 8번 OpenAI는 생성형 AI 항목으로, 상세는 [생성형 AI 활용 정리](./generative-ai.md) 문서에서 다룬다.

---

## 1. GCP Cloud Storage — 영상 스토리지

| 항목 | 내용 |
|---|---|
| 용도 | 클라이밍 영상 원본 업로드 / 재생, 썸네일 공개 호스팅 |
| 연동 방식 | Google Cloud Storage Java SDK, **v4 Signed URL** |
| 핵심 설계 | 영상 바이너리가 Spring Boot를 경유하지 않고 클라이언트 ↔ GCS 직접 전송 |

### 동작 흐름 (2단계 업로드)

```
1. 클라이언트 → API:  POST /api/videos/upload-url  (확장자·크기 검증)
2. API → 클라이언트:  업로드용 Signed URL 발급 (유효 15분)
3. 클라이언트 → GCS:  Signed URL로 영상 PUT 직접 업로드
4. 클라이언트 → API:  POST /api/videos  (메타데이터 등록, status=pending)
5. 영상 조회 시:      재생용 읽기 Signed URL(streamUrl) 응답에 포함
```

### 설정 (`application.yaml`)

```yaml
gcs:
  bucket: ${GCS_BUCKET:hola-climbing-log-videos}
  upload-prefix: videos/uploads
  signed-url-minutes: 15
  thumbnail-public-bucket: ${GCS_THUMBNAIL_PUBLIC_BUCKET:hola-climbing-thumbnails-public}
```

- 업로드 제한(NF-05): 확장자 `mp4/mov/hevc`, 최대 60초, 최대 200MB
- 인증: 운영은 Application Default Credentials, 테스트는 NoCredentials + 런타임 RSA 키
- 테스트: `fake-gcs-server` 컨테이너로 SDK 라운드트립 + Signed URL 발급 구조 검증

---

## 2~4. 소셜 로그인 — Google / Kakao / Naver OAuth 2.0

| 항목 | 내용 |
|---|---|
| 용도 | 소셜 계정 기반 회원가입·로그인 |
| 연동 방식 | OAuth 2.0 Authorization Code Grant (서버 사이드 토큰 교환) |
| 보안 | `state` 파라미터 검증, 허용 redirect URI 화이트리스트, 결과 TTL 단기화 |

### 프로바이더별 엔드포인트 (`application.yaml`)

| 프로바이더 | Authorization | Token | UserInfo | Scope |
|---|---|---|---|---|
| Google | `accounts.google.com/o/oauth2/v2/auth` | `oauth2.googleapis.com/token` | `openidconnect.googleapis.com/v1/userinfo` | openid,email,profile |
| Kakao | `kauth.kakao.com/oauth/authorize` | `kauth.kakao.com/oauth/token` | `kapi.kakao.com/v2/user/me` | account_email, profile_nickname, profile_image |
| Naver | `nid.naver.com/oauth2.0/authorize` | `nid.naver.com/oauth2.0/token` | `openapi.naver.com/v1/nid/me` | email, nickname, profile_image |

### 보안 처리

```yaml
app.oauth:
  state-ttl-minutes: 5           # CSRF 방지용 state 유효시간
  result-ttl-minutes: 1          # 인증 결과 단기 보관
  pending-signup-ttl-minutes: 10 # 신규 가입 대기 TTL
  allowed-redirect-uris: ...     # redirect URI 화이트리스트 (오픈 리다이렉트 차단)
```

- Client ID / Secret은 전부 환경변수(`*_OAUTH_CLIENT_ID/SECRET`)로 주입, 소스에 미포함
- 소셜 로그인 성공 후에도 자체 JWT(Access 30분 / Refresh 14일)를 발급해 인증 체계 통일

---

## 5. Kakao Maps JS SDK — 암장 탐색

| 항목 | 내용 |
|---|---|
| 용도 | 암장 지도 뷰, 위치 기반 탐색·검색, 추천 암장 표시 |
| 연동 방식 | Kakao Maps JavaScript SDK (프론트엔드) |
| 관련 파일 | `composables/useKakaoMap.ts`, `components/gym/GymMap.vue`, `pages/explore/ExplorePage.vue`, `types/kakao.d.ts` |

- TypeScript 타입 선언(`kakao.d.ts`)으로 SDK 타입 안정성 확보
- 암장 좌표를 지도에 마커로 렌더링하고, 지도 뷰 ↔ 리스트 뷰를 연동

---

## 6. SMTP 메일 — 이메일 인증

| 항목 | 내용 |
|---|---|
| 용도 | 자체 회원가입 시 이메일 소유 인증 |
| 연동 방식 | Spring Mail (SMTP) — 모드 전환형 발송기 |
| 관련 파일 | `infrastructure/mail/` (`SmtpVerificationEmailSender`, `LoggingVerificationEmailSender`) |

### 모드 전환 설계

```yaml
app.mail:
  mode: ${APP_MAIL_MODE:log}     # log: 콘솔 출력(개발), smtp: 실제 발송(운영)
  from: ${APP_MAIL_FROM:no-reply@hola.local}
```

- 개발 환경에서는 `LoggingVerificationEmailSender`가 주입되어 실제 메일 미발송(콘솔 로그)
- 운영에서는 `smtp` 모드로 전환해 실제 발송 — 외부 SMTP 의존을 환경별로 격리

---

## 7. Firebase Cloud Messaging (FCM) — 푸시 알림

| 항목 | 내용 |
|---|---|
| 용도 | 영상 분석 완료/실패, 댓글·답글·좋아요·팔로우 알림 푸시 |
| 연동 방식 | Firebase Admin SDK(서버) + `@capacitor-firebase/messaging`(모바일) |
| 관련 파일 | `infrastructure/fcm/`, `domain/notification/` |

### 모드 전환 설계

```yaml
app.fcm:
  enabled: ${FCM_ENABLED:false}              # 개발·테스트 기본 off
  credentials-path: ${FCM_CREDENTIALS_PATH:}
```

- `enabled=false`면 `NoopFcmSender`가 주입되어 호출이 모두 no-op (개발·테스트 안전)
- 운영은 서비스 계정 키로 실제 발송
- 디바이스 토큰 등록 API + 알림 도메인 이벤트와 연동 (분석 완료 → SSE → FCM 파이프라인)

---

## 8. OpenAI API — 월간 리포트 (생성형 AI)

| 항목 | 내용 |
|---|---|
| 용도 | 사용자 월간 등반 통계를 자연어 리포트로 서술 생성 |
| 모델 | `gpt-4.1-mini` (Chat Completions, JSON 모드) |
| 폴백 | 호출 실패 시 규칙 기반 서술기로 자동 대체 |

> 상세 프롬프트 설계·안전장치·폴백 전략은 [생성형 AI 활용 정리](./generative-ai.md) 문서 참조.

---

## 9. Capacitor Geolocation — GPS 암장 인증

| 항목 | 내용 |
|---|---|
| 용도 | 암장 채팅 메시지 작성 시 실제 방문 인증 (GPS 300m 반경) |
| 연동 방식 | `@capacitor/geolocation` (디바이스 위치 권한) |
| 관련 파일 | `domain/chat/dto/request/SendMessageRequest.java`, `domain/chat/service/ChatServiceImpl.java` |

- 메시지 전송 시 클라이언트 GPS 좌표를 함께 전달 → 서버가 암장 좌표와 300m 이내인지 검증
- 위치 기반 신뢰도(실제 방문자만 채팅)를 확보

---

## 공통 보안·운영 원칙

1. **시크릿 외부화**: 모든 API 키/Secret은 환경변수로 주입, 소스코드·git에 미포함
2. **모드 전환 설계**: 메일·FCM·LLM은 개발(no-op/log) ↔ 운영(실호출) 모드를 분리해 외부 의존성을 환경별 격리
3. **장애 격리**: 외부 호출 실패가 핵심 기능을 막지 않도록 폴백(LLM→규칙) 또는 비동기 큐잉(분석) 적용
4. **콜백 검증**: AI 워커 → API 콜백은 `AI_CALLBACK_SECRET`으로 위변조 차단

---

## 환경변수 목록 (외부 연동)

| 환경변수 | 서비스 |
|---|---|
| `GCS_BUCKET`, `GOOGLE_APPLICATION_CREDENTIALS` | GCP Cloud Storage |
| `GOOGLE_OAUTH_CLIENT_ID/SECRET` | Google OAuth |
| `KAKAO_OAUTH_CLIENT_ID/SECRET` | Kakao OAuth |
| `NAVER_OAUTH_CLIENT_ID/SECRET` | Naver OAuth |
| `APP_MAIL_MODE`, `APP_MAIL_FROM` | SMTP 메일 |
| `FCM_ENABLED`, `FCM_CREDENTIALS_PATH` | FCM |
| `MONTHLY_REPORT_LLM_MODE/API_KEY/MODEL` | OpenAI |
| `AI_CALLBACK_SECRET` | AI 워커 콜백 검증 |
