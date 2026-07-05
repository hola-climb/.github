# Hola(올라) 최종 보고서

> AI 동작 분석 기반 클라이밍 영상 SNS
> SSAFY 자율 프로젝트 · 2026-05-15 ~ 2026-06-25 (약 6주)

---

## 1. 프로젝트 개요

### 1.1 한 줄 소개

**Hola(올라)** 는 클라이머가 직접 등반 영상을 올리면 AI가 자세·동작 유형을 자동 분석하고, 누적된 기록을 통계·추천으로 환원해 성장을 돕는 **클라이밍 특화 영상 SNS** 다.

### 1.2 기획 배경 및 문제 정의

| 사용자 문제 | Hola의 해결 방식 |
|---|---|
| 내 클라이밍 스타일/약점을 객관적으로 알기 어렵다 | MediaPipe 포즈 추정 + 동작 분류로 영상 자동 분석 |
| 어떤 문제를 풀어야 성장하는지 모르겠다 | 사용자 스타일 임베딩 기반 영상·암장 추천(pgvector) |
| 클라이밍 기록이 사진/메모로 흩어져 있다 | 영상 업로드 + 분석 결과 + 통계/달력 누적 |

기존 SNS(인스타·유튜브)는 영상을 "보여주기"에 그치지만, Hola는 영상을 **분석 데이터**로 전환해 개인 성장 루프를 만든다는 점에서 차별화된다.

### 1.3 팀 구성

| 이름 | 역할 | 담당 영역 |
|---|---|---|
| **김민준** | 백엔드 / AI 추론 | Spring Boot API 서버 전반, Python AI 서버 추론 로직, DB 설계 |
| **곽예경** | 프론트엔드 / AI 추론 | Vue 3 웹 + Capacitor 모바일 빌드, UI/UX, Python AI 서버 추론 로직 |

### 1.4 개발 기간

- 2026-05-15: 기획 시작
- 2026-06-25: 최종 발표
- 총 기간: 약 6주 (42일)

---

## 2. 요구사항 대비 구현 결과

### 2.1 공통 필수 ① — 콘텐츠·리뷰·회원 관리

| 요구사항 | 구현 기능 | 상태 |
|---|---|---|
| F01 콘텐츠 등록 | 클라이밍 영상 업로드 (GCS Signed URL 2단계) | ✅ |
| F02 콘텐츠 조회 | 영상 목록 / 상세 조회 (페이징·정렬) | ✅ |
| F03 콘텐츠 수정 | 영상 메타정보(제목·등급·암장·태그) 수정 | ✅ |
| F04 콘텐츠 삭제 | 영상 삭제 (Soft Delete, 30일 후 영구삭제) | ✅ |
| F05 콘텐츠 검색/정렬 | 카테고리·인기·최신·등급별 검색 | ✅ |
| F06~F09 리뷰 CRUD | 영상 댓글 작성/조회/수정/삭제 (대댓글 1뎁스) | ✅ |
| F10 회원 등록 | 자체 회원가입 (이메일 인증, BCrypt strength=12) | ✅ |
| F11~F13 회원 조회/수정/삭제 | 프로필 조회·수정, 탈퇴(Soft Delete, 30일 후 익명화) | ✅ |
| F14 로그인/로그아웃 | JWT 기반 (Access 30분 / Refresh 14일, Refresh 토큰 회전) | ✅ |

### 2.2 공통 필수 ② — 추가·심화 기능

| 요구사항 | 구현 기능 | 분류 |
|---|---|---|
| F15 찜/즐겨찾기 | 영상 즐겨찾기, 암장 즐겨찾기 | 추가 ✅ |
| F16 팔로우/팔로잉 | 사용자 팔로우/팔로워, 차단 | 추가 ✅ |
| F19 AI 추천 | pgvector 코사인 유사도 기반 영상·암장 추천 | 심화 ✅ |
| F20 AI 코칭/분석 | MediaPipe 포즈 추정 + 동작 분류(6종) | 심화 ✅ |

### 2.3 자체 확장 기능

- **소셜 로그인**: Google / Kakao / Naver OAuth 2.0
- **암장 탐색**: Kakao Map 기반 지도 뷰 / 검색 / 추천
- **실시간 채팅**: WebSocket(STOMP) + 메시지별 GPS 300m 암장 인증
- **푸시 알림**: FCM (영상 분석 완료·실패, 댓글·답글·좋아요·팔로우)
- **월간 리포트**: 생성형 AI(OpenAI) 기반 개인 등반 리포트 서술 생성
- **신고/관리자**: 신고 등록·중복 차단, 관리자 대시보드·감사 로그

### 2.4 미구현 (발표 후 백로그)

- F17 계획/일정 관리, F18 챌린지 관리 (선택 항목)
- LSTM 파인튜닝 / VideoMAE (라벨 데이터가 충분히 쌓인 뒤 검토할 장기 백로그)
- e_trajectory / e_arm 효율 점수 (PoC 코드만 존재, 정식 기능 제외)

---

## 3. 시스템 아키텍처

### 3.1 전체 구성

![](../assets/architecture.png)

### 3.2 계층별 기술 스택

| 계층 | 기술 | 선택 이유 |
|---|---|---|
| Interface (Web) | Vue 3 + Vite + TypeScript, Pinia | 경량 빌드, 컴포넌트 재사용, 타입 안정성 |
| Interface (Mobile) | Vue 3 + Capacitor 8 | Vue 코드 그대로 iOS/Android 앱 빌드(WebView) |
| Processing (API) | Spring Boot 4.0.6 (Spring MVC) + Java 25 LTS | 표준 MVC, 강력한 생태계 |
| Processing (Auth) | Spring Security + JWT | 토큰 기반 Stateless 인증 |
| Processing (AI) | Python 3.11 + FastAPI + Redis Streams consumer | AI 추론 전담, GPU 워커 분리 |
| Infra (DB) | PostgreSQL 16 + pgvector + MyBatis 4.0.1 | 관계형 + 벡터 유사도 검색, SQL 직접 제어 |
| Infra (Cache/Broker) | Redis 7 | 세션, JWT 블랙리스트, Redis Streams 큐 |
| Infra (Storage) | GCP Cloud Storage | 영상 파일 (Original/Processed/Streaming 경로) |
| Observability | Actuator + Prometheus + Grafana | 메트릭 수집·대시보드 |

### 3.3 핵심 설계 원칙

1. **영상 바이너리는 Spring Boot를 절대 경유하지 않는다.** 클라이언트가 GCS Signed URL로 직접 업로드/재생하여 API 서버 부하·대역폭을 제거.
2. **AI 추론은 비동기로 분리한다.** 영상 등록 시 Redis Streams로 작업을 큐잉하고, Python 워커가 처리 후 콜백. GPU가 꺼져 있어도 조회/통계/추천은 정상 운영, 분석만 큐 대기.
3. **WebSocket은 Spring Boot 전담.** 실시간 채팅·SSE는 API 서버가 담당하고 Python은 추론에만 집중.
4. **일관된 응답·예외 체계.** 모든 응답은 `ApiResponse<T>` 래퍼, 모든 에러는 `ErrorCode` enum + `BusinessException`.

---

## 4. AI 분석 파이프라인

### 4.1 분석 대상 (6종 동작)

하이스텝 / 플래깅 / 훅(힐·토) / 락오프 / 다이노 / 코디네이션

### 4.2 처리 흐름

```
영상 등록(status=pending)
   → Redis Streams 작업 큐잉
   → Python Redis Streams 워커: MediaPipe 33-keypoint 포즈 추정
   → 규칙 기반 분류기로 6종 동작 분류
   → Spring Boot 콜백 (AI_CALLBACK_SECRET 검증)
   → 분석 결과 저장 + SSE 푸시 + FCM 알림
```

### 4.3 모델 고도화 전략

| 단계 | 모델 | 트리거 |
|---|---|---|
| MVP | MediaPipe Pose + 규칙 기반 분류 | 현재 적용 ✅ |
| v1.1 | LSTM 파인튜닝 | 사용자 피드백 라벨이 충분히 누적된 뒤 검토 |
| 장기 | VideoMAE 영상 분류 | 데이터 충분 시 |

사용자 "맞아요/틀렸어요" 피드백을 자동 라벨로 누적하고, 임계치 도달 시 `train_worker`가 파인튜닝을 실행해 모델을 교체하는 **셀프 러닝 루프**를 설계했다.

### 4.4 레퍼런스

- CS231N 2024 *"Using Pose Estimation to Analyze Rock Climbing Technique"* — 임계값 재사용
- BoulderVision (Roboflow) — 슬라이딩 윈도우/velocity ratio 기법 참고, 자체 Python 서버로 완전 대체

---

## 5. 데이터베이스 설계

- **테이블 22개** 구성, 도메인 10개(user, video, stats, gym, chat, recommendation, favorite, analysis, notification, report)
- **pgvector v0.8.2** 설치 — 사용자 스타일 임베딩 코사인 유사도 검색
- **Flyway** 마이그레이션 버전 관리
- Soft Delete + 30일 보존 정책 (영상·회원)

![](../assets/erd.png)

---

## 6. 품질 보증 (테스트)

| 항목 | 내용 |
|---|---|
| 통합 테스트 | 발표 산출물 기준 **177개** 통합 테스트, 2026-07-02 기준 backend full suite **397 tests passing** |
| PostgreSQL | 실제 컨테이너 — JSONB·인덱스·ON CONFLICT까지 검증 |
| Redis | 실제 컨테이너 — Streams 큐 / Pub-Sub / 상태 저장소 |
| GCS | fake-gcs-server 컨테이너 — Storage SDK 라운드트립 + Signed URL 발급 구조 검증 |
| FCM | NoopFcmSender — 호출 인자 검증 (도달은 수동 smoke) |

각 도메인은 `Controller → Service/ServiceImpl → Mapper(+XML)` 구조에 `*IntegrationTest` + 테스트 픽스처 SQL을 포함한다.

---

## 7. 외부 서비스 연동 요약

| 서비스 | 용도 |
|---|---|
| GCP Cloud Storage | 영상 업로드/재생 Signed URL |
| Google/Kakao/Naver OAuth | 소셜 로그인 |
| Kakao Map JS SDK | 암장 지도·탐색 |
| SMTP 메일 | 회원가입 이메일 인증 |
| FCM (Firebase) | 푸시 알림 |
| OpenAI API (gpt-4.1-mini) | 월간 리포트 서술 생성 |
| Capacitor Geolocation | 채팅 GPS 암장 인증(300m) |

> 상세 내역: [외부 API 활용 정리](./external-apis.md), [생성형 AI 활용 정리](./generative-ai.md)

---

## 8. 트러블슈팅 & 의사결정 하이라이트

| 이슈 | 해결 |
|---|---|
| 대용량 영상 업로드 시 API 서버 부하 | GCS Signed URL 2단계 직접 업로드로 서버 미경유 |
| GPU 가용 시간 제약(데스크톱 1일 9시간) | Redis Streams 비동기 큐 — GPU 다운 시에도 핵심 기능 정상, 분석만 지연 |
| 생성형 AI의 환각·부적절 조언 위험 | 안전 시스템 프롬프트 + JSON 스키마 강제 + 규칙 기반 폴백 (5절·생성형 AI 문서 참조) |
| GCS Signed URL 자동화 테스트 한계 | fake-gcs-server로 구조 검증 + 운영 IAM/실서명은 수동 smoke 1회 |
| MediaPipe-Python 버전 호환 | Python 3.11.9 고정 (3.14 비호환) |

---

## 9. 회고 및 향후 계획

### 9.1 성과

- SSAFY 공통 필수 + 추가/심화 요구사항을 모두 충족하고, 소셜 로그인·실시간 채팅·생성형 AI 리포트 등 자체 확장 기능까지 구현.
- API/AI 서버를 물리적으로 분리하고 비동기 파이프라인을 설계해 **장애 격리**와 **확장성**을 확보.
- 발표 산출물 기준 177개 통합 테스트, 2026-07-02 기준 backend full suite 397 tests passing으로 회귀 안전망을 확장.

### 9.2 향후 계획

- LSTM/VideoMAE 기반 학습 루프 검토 (라벨 데이터 축적 후)
- 효율 점수(e_trajectory/e_arm) 정식 기능화
- F17/F18 선택 기능 (계획·챌린지) 추가
- RunPod/Modal 서버리스 GPU로 추론 가용성 보완

---

## 부록: 산출물 목록

| 산출물 | 위치 |
|---|---|
| Organization README | [`../README.md`](../README.md) |
| API 명세서 | [`./api-spec.md`](./api-spec.md) |
| ERD | [`../assets/erd.png`](../assets/erd.png) |
| 화면설계서 (22종) | [`./screen-designs/`](./screen-designs/) |
| 아키텍처 다이어그램 | [`../assets/architecture.png`](../assets/architecture.png) |
| 디자인 시스템 | [`./design-system.md`](./design-system.md) |
| 외부 API 활용 정리 | [`./external-apis.md`](./external-apis.md) |
| 생성형 AI 활용 정리 | [`./generative-ai.md`](./generative-ai.md) |
