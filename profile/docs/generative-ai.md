# 생성형 AI 활용 내용 정리

> Hola(올라) — AI 동작 분석 기반 클라이밍 영상 SNS
> 본 문서는 제품 내 생성형 AI 기능의 정의·연동·안전 설계를 정리한다.

---

## 개요

Hola는 제품 기능으로 생성형 AI를 활용한다.

| 기능 | 내용 | 모델 |
|---|---|---|
| 월간 리포트 서술 생성 | 월간 등반 통계를 자연어 리포트로 변환 | OpenAI `gpt-4.1-mini` |

> 핵심 동작 분석(MediaPipe 포즈 추정 + 6종 동작 분류)은 생성형 AI가 아닌 **판별형(discriminative) 모델**이므로 본 문서가 아니라 최종보고서 4장에서 다룬다.

---

## 1. 기능 정의

사용자의 월간 등반 통계(영상 수, 등반 볼륨, 동작 기술별 사용 횟수, 미사용 기술 등)를 입력받아, 사람이 읽기 좋은 **월간 리포트 문장**(headline / summary / highlights)을 생성한다.

| 항목 | 내용 |
|---|---|
| 모델 | OpenAI `gpt-4.1-mini` |
| API | Chat Completions (`/chat/completions`) |
| 출력 형식 | `response_format: json_object` — JSON 강제 |
| 구현 위치 | `domain/stats/service/OpenAiMonthlyReportNarrativeClient.java` |
| 인터페이스 | `MonthlyReportNarrativeClient` (전략 패턴) |

---

## 2. 모드 전환 설계 (rule ↔ openai)

```yaml
app.monthly-report.llm:
  mode: ${MONTHLY_REPORT_LLM_MODE:rule}        # rule(기본) / openai
  base-url: ${MONTHLY_REPORT_LLM_BASE_URL:https://api.openai.com/v1}
  api-key: ${MONTHLY_REPORT_LLM_API_KEY:}
  model: ${MONTHLY_REPORT_LLM_MODEL:gpt-4.1-mini}
  timeout-seconds: ${MONTHLY_REPORT_LLM_TIMEOUT_SECONDS:20}
```

- `@ConditionalOnProperty(... mode = "openai")` 로 빈을 조건부 주입
- 기본값은 `rule` (외부 비용·의존성 없이 동작), 운영에서 `openai`로 전환
- LLM 응답 파싱 실패·타임아웃·예외 시 **규칙 기반 서술기(`RuleBasedMonthlyReportNarrativeClient`)로 자동 폴백** → 외부 API 장애가 기능을 막지 않음

---

## 3. 안전 프롬프트 설계 (Responsible AI)

생성형 AI의 환각·부적절한 조언 위험을 막기 위해 시스템 프롬프트에 **명시적 제약**을 걸었다.

```
너는 클라이밍 앱의 월간 리포트 문장을 작성하는 한국어 리포트 writer다.
- 서버가 제공한 JSON만 근거로 사용하고, JSON에 없는 사실은 만들지 않는다.
- 숫자, 횟수, 퍼센트, 난이도 값은 새로 쓰거나 추정하지 않는다.
- 부상 위험이 있는 공격적인 무브 처방을 하지 않는다.
- 의학적 조언이나 신체 상태 판단을 하지 않는다.
- 기술 다양성, 등반 볼륨, 기록 습관에 관한 문장만 작성한다.
- 출력은 headline, summary, highlights 키만 가진 JSON 객체여야 한다.
```

### 안전장치 요약

| 위험 | 대응 |
|---|---|
| 환각(없는 사실 생성) | "제공된 JSON만 근거", 숫자 추정 금지 |
| 부상 유발 조언 | 공격적 무브 처방 금지 |
| 의료/신체 판단 | 의학적 조언 금지 명문화 |
| 출력 포맷 깨짐 | JSON 모드 + 스키마 키 강제 + 파싱 검증 |
| 외부 장애 | 규칙 기반 폴백 |

---

## 4. 호출 데이터 흐름

```
월간 통계 집계(MonthlyReportAggregate)
   → metrics / techniqueCounts / underusedTechniques 를 user 메시지로 직렬화
   → system(안전 프롬프트) + user(통계 JSON) 으로 호출
   → 응답 content(JSON) 파싱 → headline/summary/highlights 검증
   → 검증 실패 시 규칙 기반 폴백
```

- 입력은 **집계된 수치만** 전달(원본 영상·개인정보 미전송)
- `timeout-seconds`로 응답 지연 상한 설정, API Key는 환경변수 주입

---

## 부록: 관련 코드/설정 위치

| 항목 | 위치 |
|---|---|
| 월간 리포트 LLM 클라이언트 | `hola-climbing-server/.../domain/stats/service/OpenAiMonthlyReportNarrativeClient.java` |
| 규칙 기반 폴백 | `.../domain/stats/service/RuleBasedMonthlyReportNarrativeClient.java` |
| LLM 설정 프로퍼티 | `.../domain/stats/MonthlyReportProperties.java`, `application.yaml` |
| 동작 분석(판별형 AI) | `hola-climbing-ai/` (MediaPipe + 분류기) — 본 문서 범위 외 |
