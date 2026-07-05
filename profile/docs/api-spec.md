# Hola Climbing Server - API 명세서

> 동기화 기준일: `2026-06-26`
> 기준 소스: `hola-climbing-server/src/main/java/com/holaclimbing/server/**Controller.java`, DTO `record`, `SecurityConfig`, `ErrorCode`.
> 이 문서는 현재 Spring 소스 기준으로 재생성했다. 오래된 Notion/수기 명세보다 현재 소스가 우선이다.

---

## 1. 공통 사항

### 1.1 Base URL

- Local: `http://localhost:8080`
- Swagger UI: `http://localhost:8080/swagger-ui.html`
- OpenAPI JSON: `GET /v1/api-docs`

### 1.2 공통 응답 포맷 `ApiResponse<T>`

성공 응답은 `isSuccess=true`, `code=OK`를 사용한다. `message`와 `data`는 `null`이면 직렬화에서 제외될 수 있다.

```json
{
  "isSuccess": true,
  "code": "OK",
  "message": null,
  "data": {},
  "timestamp": "2026-06-26T05:30:00Z"
}
```

실패 응답은 `data` 없이 내려간다.

```json
{
  "isSuccess": false,
  "code": "U002",
  "message": "이미 사용 중인 이메일입니다.",
  "timestamp": "2026-06-26T05:30:00Z"
}
```

### 1.3 목록 응답

- Offset page: `PageResponse<T> { content, page, size, totalElements, totalPages, hasNext }`
- Cursor page: `CursorPageResponse<T> { content, nextCursor, hasNext }`
- `GET /api/videos`와 `GET /api/recommendations/videos`는 `cursor`와 `nextCursor` query parameter를 모두 허용한다. 둘 다 오면 `cursor`가 우선이다.
- `size`는 대부분 기본 `20`, 최대 `100`이다. 채팅 메시지는 기본 `30`, 최대 `100`이다.

### 1.4 인증/권한

- `Public`: 인증 헤더 없이 호출 가능
- `Public/Optional JWT`: 공개 호출 가능하지만 JWT가 있으면 차단/즐겨찾기/본인 여부 같은 viewer context가 반영될 수 있음
- `Authenticated`: `Authorization: Bearer {accessToken}` 필요
- `ADMIN`: `ROLE_ADMIN` JWT 필요
- `AI_CALLBACK_SECRET`: `X-AI-Callback-Secret: {secret}` 필요
- STOMP handshake endpoint는 `/ws`, publish destination은 `/app/gyms/{gymId}/chat`, subscribe destination은 `/topic/gyms/{gymId}/chat`이다. 연결/메시지 인증은 STOMP interceptor/JWT 컨텍스트에서 처리한다.

### 1.5 전역 오류

- `400 C001 INVALID_INPUT`: Bean Validation, 타입 변환, 잘못된 query/body
- `401 C002 UNAUTHORIZED`: 인증 필요 또는 인증 실패
- `403 C003 FORBIDDEN`: 권한 부족
- `404 C004 NOT_FOUND`: 라우트 또는 리소스 없음
- `405 C005 METHOD_NOT_ALLOWED`: HTTP method 불일치
- `500 C999 INTERNAL_ERROR`: 서버 오류
- 전체 카탈로그: `GET /api/docs/error-codes`

---

## 2. 전체 API 목록

- 현재 소스 기준 endpoint 수: REST `112`개 + STOMP `1`개
- 구형 수기 명세 대비 반영된 누락 endpoint: `53`개
- 구형 수기 명세 대비 제거된 stale endpoint: `9`개

| 분류 | 기능 | Method | Path | Auth | Request | Response |
|---|---|---|---|---|---|---|
| 회원/인증 | 이메일 중복 확인 | GET | `/api/auth/email-check` | Public | `-` | `AvailabilityResponse` |
| 회원/인증 | 이메일 인증 | POST | `/api/auth/email/verify` | Public | `VerifyEmailRequest` | `Void` |
| 회원/인증 | 로그인 | POST | `/api/auth/login` | Public | `LoginRequest` | `TokenResponse` |
| 회원/인증 | 로그아웃 | POST | `/api/auth/logout` | Public | `LogoutRequest` | `Void` |
| 회원/인증 | 닉네임 중복 확인 | GET | `/api/auth/nickname-check` | Public | `-` | `AvailabilityResponse` |
| 회원/인증 | OAuth 결과 코드 소비 | POST | `/api/auth/oauth/result` | Public | `OAuthResultRequest` | `OAuthLoginResponse` |
| 회원/인증 | OAuth 신규 회원가입 | POST | `/api/auth/oauth/signup` | Public | `OAuthSignupRequest` | `TokenResponse` |
| 회원/인증 | OAuth 인증 시작 redirect | GET | `/api/auth/oauth/{provider}/authorize` | Public | `-` | `302 redirect / no body` |
| 회원/인증 | OAuth provider callback | GET | `/api/auth/oauth/{provider}/callback` | Public | `-` | `302 redirect / no body` |
| 회원/인증 | 비밀번호 재설정 | POST | `/api/auth/password/reset` | Public | `PasswordResetRequest` | `Void` |
| 회원/인증 | 비밀번호 재설정 요청 | POST | `/api/auth/password/reset-request` | Public | `PasswordResetEmailRequest` | `Void` |
| 회원/인증 | 토큰 재발급 | POST | `/api/auth/refresh` | Public | `RefreshRequest` | `TokenResponse` |
| 회원/인증 | 이메일 인증 재발송 | POST | `/api/auth/resend-verification` | Public | `ResendVerificationRequest` | `Void` |
| 회원/인증 | 회원가입 | POST | `/api/auth/signup` | Public | `SignupRequest` | `SignupResponse` |
| 사용자 | 회원 탈퇴 | DELETE | `/api/users/me` | Authenticated | `WithdrawRequest` | `Void` |
| 사용자 | 내 프로필 조회 | GET | `/api/users/me` | Authenticated | `-` | `MyProfileResponse` |
| 사용자 | 내 프로필 수정 | PATCH | `/api/users/me` | Authenticated | `UpdateProfileRequest` | `MyProfileResponse` |
| 사용자 | 차단 목록 조회 | GET | `/api/users/me/blocks` | Authenticated | `-` | `PageResponse<UserSummaryResponse>` |
| 사용자 | 디바이스 토큰 해제 | DELETE | `/api/users/me/device-tokens` | Authenticated | `UnregisterDeviceTokenRequest` | `Void` |
| 사용자 | 디바이스 토큰 등록 | POST | `/api/users/me/device-tokens` | Authenticated | `RegisterDeviceTokenRequest` | `Void` |
| 사용자 | 내 프로필 이미지 업로드 | POST | `/api/users/me/profile-image` | Authenticated | `image:MultipartFile` | `MyProfileResponse` |
| 사용자 | 사용자 프로필 조회 | GET | `/api/users/{userId}` | Public/Optional JWT | `-` | `UserProfileResponse` |
| 사용자 | 차단 해제 | DELETE | `/api/users/{userId}/block` | Authenticated | `-` | `Void` |
| 사용자 | 사용자 차단 | POST | `/api/users/{userId}/block` | Authenticated | `-` | `Void` |
| 사용자 | 언팔로우 | DELETE | `/api/users/{userId}/follow` | Authenticated | `-` | `Void` |
| 사용자 | 팔로우 | POST | `/api/users/{userId}/follow` | Authenticated | `-` | `Void` |
| 사용자 | 팔로워 목록 | GET | `/api/users/{userId}/followers` | Public/Optional JWT | `-` | `PageResponse<UserSummaryResponse>` |
| 사용자 | 팔로잉 목록 | GET | `/api/users/{userId}/following` | Public/Optional JWT | `-` | `PageResponse<UserSummaryResponse>` |
| 영상 | 댓글 삭제 | DELETE | `/api/comments/{commentId}` | Authenticated | `-` | `Void` |
| 영상 | 댓글 수정 | PATCH | `/api/comments/{commentId}` | Authenticated | `UpdateCommentRequest` | `CommentResponse` |
| 영상 | 영상 목록/피드 조회 | GET | `/api/videos` | Public/Optional JWT | `-` | `CursorPageResponse<VideoSummaryResponse>` |
| 영상 | 영상 등록 | POST | `/api/videos` | Authenticated | `CreateVideoRequest` | `VideoDetailResponse` |
| 영상 | 영상 썸네일 업로드 | POST | `/api/videos/thumbnail` | Authenticated | `image:MultipartFile` | `ThumbnailUploadResponse` |
| 영상 | 영상 업로드 Signed URL 발급 | POST | `/api/videos/upload-url` | Authenticated | `UploadUrlRequest` | `UploadUrlResponse` |
| 영상 | 영상 삭제 | DELETE | `/api/videos/{videoId}` | Authenticated | `-` | `Void` |
| 영상 | 영상 상세 조회 | GET | `/api/videos/{videoId}` | Public/Optional JWT | `-` | `VideoDetailResponse` |
| 영상 | 영상 수정 | PATCH | `/api/videos/{videoId}` | Authenticated | `UpdateVideoRequest` | `VideoDetailResponse` |
| 영상 | 댓글 목록 조회 | GET | `/api/videos/{videoId}/comments` | Public/Optional JWT | `-` | `PageResponse<CommentResponse>` |
| 영상 | 댓글 작성 | POST | `/api/videos/{videoId}/comments` | Authenticated | `CreateCommentRequest` | `CommentResponse` |
| 영상 | 영상 좋아요 취소 | DELETE | `/api/videos/{videoId}/like` | Authenticated | `-` | `LikeResponse` |
| 영상 | 영상 좋아요 | POST | `/api/videos/{videoId}/like` | Authenticated | `-` | `LikeResponse` |
| 영상 | 영상 공유 링크 발급 | POST | `/api/videos/{videoId}/share` | Authenticated | `-` | `ShareLinkResponse` |
| 영상 | 분석 상태 조회 | GET | `/api/videos/{videoId}/status` | Public/Optional JWT | `-` | `VideoStatusResponse` |
| AI 분석 | AI 워커 분석 결과 수신 | POST | `/api/analysis/videos/{videoId}` | AI_CALLBACK_SECRET | `AnalysisIngestRequest` | `VideoAnalysisResponse` |
| AI 분석 | 분석 결과 조회 | GET | `/api/videos/{videoId}/analysis` | Public/Optional JWT | `-` | `VideoAnalysisResponse` |
| AI 분석 | 분석 결과 피드백 | POST | `/api/videos/{videoId}/analysis/feedback` | Authenticated | `AnalysisFeedbackRequest` | `FeedbackResponse` |
| AI 분석 | 분석 재시도 | POST | `/api/videos/{videoId}/analysis/retry` | Authenticated | `-` | `VideoAnalysisResponse` |
| AI 분석 | 분석 진행률 SSE | GET | `/api/videos/{videoId}/analysis/stream` | Public/Optional JWT | `-` | `SSE progress events` |
| 통계 | 내 통계 조회 | GET | `/api/stats/me` | Authenticated | `-` | `UserStatsResponse` |
| 통계 | 월간 달력 조회 | GET | `/api/stats/me/calendar` | Authenticated | `-` | `MonthlyCalendarResponse` |
| 통계 | 날짜별 클라이밍 기록 조회 | GET | `/api/stats/me/calendar/{date}` | Authenticated | `-` | `List<ClimbingLogResponse>` |
| 통계 | 내 암장 방문 랭킹 | GET | `/api/stats/me/gyms/rankings` | Authenticated | `-` | `GymRankingResponse` |
| 통계 | 월간 AI 리포트 조회 | GET | `/api/stats/me/monthly-reports` | Authenticated | `-` | `MonthlyReportResponse` |
| 통계 | 월간 리포트 생성 월 목록 | GET | `/api/stats/me/monthly-reports/available` | Authenticated | `-` | `MonthlyReportAvailablePeriodsResponse` |
| 통계 | 내 기술별 통계 | GET | `/api/stats/me/techniques` | Authenticated | `-` | `TechniqueStatsResponse` |
| 통계 | 사용자 공개 통계 조회 | GET | `/api/stats/users/{userId}` | Public | `-` | `UserStatsResponse` |
| 클라이밍 기록 | 클라이밍 기록 생성 | POST | `/api/climbing-logs` | Authenticated | `CreateClimbingLogRequest` | `ClimbingLogResponse` |
| 클라이밍 기록 | 클라이밍 기록 삭제 | DELETE | `/api/climbing-logs/{logId}` | Authenticated | `-` | `Void` |
| 클라이밍 기록 | 클라이밍 기록 상세 조회 | GET | `/api/climbing-logs/{logId}` | Authenticated | `-` | `ClimbingLogResponse` |
| 클라이밍 기록 | 클라이밍 기록 수정 | PATCH | `/api/climbing-logs/{logId}` | Authenticated | `UpdateClimbingLogRequest` | `ClimbingLogResponse` |
| 암장 | 암장 목록/검색 | GET | `/api/gyms` | Public/Optional JWT | `-` | `PageResponse<GymSummaryResponse>` |
| 암장 | 암장 등록 제안 | POST | `/api/gyms` | Authenticated | `CreateGymRequest` | `CreateGymResponse` |
| 암장 | 주변 암장 조회 | GET | `/api/gyms/nearby` | Public/Optional JWT | `-` | `List<GymSummaryResponse>` |
| 암장 | 암장 상세 조회 | GET | `/api/gyms/{gymId}` | Public/Optional JWT | `-` | `GymDetailResponse` |
| 암장 | 암장 운영시간 수정 | PATCH | `/api/gyms/{gymId}/business-hours` | Authenticated | `UpdateBusinessHoursRequest` | `GymDetailResponse` |
| 암장 | 암장 난이도 목록 | GET | `/api/gyms/{gymId}/grades` | Public/Optional JWT | `-` | `List<GymGradeResponse>` |
| 암장 | 암장 영상 목록 | GET | `/api/gyms/{gymId}/videos` | Public/Optional JWT | `-` | `PageResponse<VideoSummaryResponse>` |
| 암장 리뷰 | 암장 리뷰 삭제 | DELETE | `/api/gyms/reviews/{reviewId}` | Authenticated | `-` | `Void` |
| 암장 리뷰 | 암장 리뷰 수정 | PATCH | `/api/gyms/reviews/{reviewId}` | Authenticated | `UpdateReviewRequest` | `GymReviewResponse` |
| 암장 리뷰 | 암장 리뷰 목록 | GET | `/api/gyms/{gymId}/reviews` | Public/Optional JWT | `-` | `PageResponse<GymReviewResponse>` |
| 암장 리뷰 | 암장 리뷰 작성 | POST | `/api/gyms/{gymId}/reviews` | Authenticated | `CreateReviewRequest` | `GymReviewResponse` |
| 즐겨찾기 | 즐겨찾기 암장 목록 | GET | `/api/favorites/gyms` | Authenticated | `-` | `PageResponse<GymSummaryResponse>` |
| 즐겨찾기 | 암장 즐겨찾기 해제 | DELETE | `/api/favorites/gyms/{gymId}` | Authenticated | `-` | `Void` |
| 즐겨찾기 | 암장 즐겨찾기 추가 | POST | `/api/favorites/gyms/{gymId}` | Authenticated | `-` | `Void` |
| 추천 | 주변/스타일 암장 추천 | GET | `/api/recommendations/gyms` | Authenticated | `-` | `List<RecommendedGymResponse>` |
| 추천 | 개인화 영상 추천 피드 | GET | `/api/recommendations/videos` | Authenticated | `-` | `CursorPageResponse<RecommendedVideoResponse>` |
| 추천 | 영상 기반 암장 추천 | GET | `/api/videos/{videoId}/recommendations/gyms` | Public/Optional JWT | `-` | `List<VideoRecommendedGymResponse>` |
| 알림 | 알림 목록 조회 | GET | `/api/notifications` | Authenticated | `-` | `PageResponse<NotificationResponse>` |
| 알림 | 알림 설정 조회 | GET | `/api/notifications/settings` | Authenticated | `-` | `NotificationSettingsResponse` |
| 알림 | 알림 설정 변경 | PATCH | `/api/notifications/settings` | Authenticated | `UpdateNotificationSettingsRequest` | `NotificationSettingsResponse` |
| 알림 | 미읽음 알림 수 | GET | `/api/notifications/unread-count` | Authenticated | `-` | `Long` |
| 알림 | 알림 읽음 처리 | PATCH | `/api/notifications/{id}/read` | Authenticated | `-` | `UnreadCountResponse` |
| 알림 | 알림 삭제 | DELETE | `/api/notifications/{notificationId}` | Authenticated | `-` | `Void` |
| 신고 | 신고 작성 | POST | `/api/reports` | Authenticated | `CreateReportRequest` | `ReportResponse` |
| 채팅 | 암장 채팅방 참여 | POST | `/api/chats/gyms/{gymId}/join` | Authenticated | `-` | `ChatRoomResponse` |
| 채팅 | 암장 채팅 메시지 조회 | GET | `/api/chats/gyms/{gymId}/messages` | Authenticated | `-` | `PageResponse<ChatMessageResponse>` |
| 채팅 | 암장 채팅 메시지 전송 | STOMP | `/app/gyms/{gymId}/chat` | STOMP CONNECT JWT | `-` | `ChatMessageResponse` |
| 관리자 | 관리자 AI 모델 지표 | GET | `/api/admin/analysis/models/{modelVersion}/metrics` | ADMIN | `-` | `AnalysisModelMetricsResponse` |
| 관리자 | 관리자 감사 로그 조회 | GET | `/api/admin/audit-logs` | ADMIN | `-` | `PageResponse<AdminAuditLogResponse>` |
| 관리자 | 관리자 대시보드 | GET | `/api/admin/dashboard` | ADMIN | `-` | `AdminDashboardResponse` |
| 관리자 | 관리자 암장 검색 | GET | `/api/admin/gyms` | ADMIN | `-` | `PageResponse<AdminGymSearchResponse>` |
| 관리자 | 관리자 암장 생성 | POST | `/api/admin/gyms` | ADMIN | `AdminGymUpsertRequest` | `GymDetailResponse` |
| 관리자 | 관리자 암장 일괄 import 적용 | POST | `/api/admin/gyms/import` | ADMIN | `AdminGymImportRequest` | `AdminGymImportApplyResponse` |
| 관리자 | 관리자 암장 일괄 import 미리보기 | POST | `/api/admin/gyms/import/preview` | ADMIN | `AdminGymImportRequest` | `AdminGymImportPreviewResponse` |
| 관리자 | 관리자 암장 상세 | GET | `/api/admin/gyms/{gymId}` | ADMIN | `-` | `GymDetailResponse` |
| 관리자 | 관리자 암장 수정 | PATCH | `/api/admin/gyms/{gymId}` | ADMIN | `AdminGymUpsertRequest` | `GymDetailResponse` |
| 관리자 | 암장 등록 제안 승인 | POST | `/api/admin/gyms/{gymId}/approve` | ADMIN | `AdminReasonRequest` | `GymDetailResponse` |
| 관리자 | 암장 폐업 처리 | POST | `/api/admin/gyms/{gymId}/close` | ADMIN | `AdminReasonRequest` | `GymDetailResponse` |
| 관리자 | 암장 난이도 전체 교체 | PUT | `/api/admin/gyms/{gymId}/grades` | ADMIN | `AdminGymGradeReplaceRequest` | `List<GymGradeResponse>` |
| 관리자 | 관리자 암장 프로필 이미지 업로드 | POST | `/api/admin/gyms/{gymId}/profile-image` | ADMIN | `image:MultipartFile` | `GymDetailResponse` |
| 관리자 | 암장 등록 제안 반려 | POST | `/api/admin/gyms/{gymId}/reject` | ADMIN | `AdminReasonRequest` | `GymDetailResponse` |
| 관리자 | 관리자 신고 검색 | GET | `/api/admin/reports` | ADMIN | `-` | `PageResponse<AdminReportResponse>` |
| 관리자 | 관리자 신고 상태 변경 | PATCH | `/api/admin/reports/{reportId}/status` | ADMIN | `AdminReportStatusRequest` | `AdminReportResponse` |
| 관리자 | 관리자 사용자 검색 | GET | `/api/admin/users` | ADMIN | `-` | `PageResponse<AdminUserSearchResponse>` |
| 관리자 | 관리자 사용자 상세 | GET | `/api/admin/users/{userId}` | ADMIN | `-` | `AdminUserDetailResponse` |
| 관리자 | 관리자 사용자 토큰 무효화 | POST | `/api/admin/users/{userId}/revoke-tokens` | ADMIN | `AdminReasonRequest` | `AdminUserDetailResponse` |
| 관리자 | 관리자 사용자 권한 변경 | PATCH | `/api/admin/users/{userId}/role` | ADMIN | `AdminUserRoleRequest` | `AdminUserDetailResponse` |
| 관리자 | 관리자 사용자 상태 변경 | PATCH | `/api/admin/users/{userId}/status` | ADMIN | `AdminUserStatusRequest` | `AdminUserDetailResponse` |
| 약관 | 활성 약관 조회 | GET | `/api/terms` | Public | `-` | `List<TermResponse>` |
| 약관 | 약관 동의 기록 | POST | `/api/terms/agree` | Authenticated | `AgreeTermsRequest` | `Void` |
| 약관 | 내 약관 동의 상태 조회 | GET | `/api/terms/agreement-status` | Authenticated | `-` | `TermsAgreementStatusResponse` |
| 공통 | Actuator health | GET | `/actuator/health` | Public | `-` | `Actuator health JSON` |
| 공통 | 에러 코드 카탈로그 | GET | `/api/docs/error-codes` | Public | `-` | `List<ErrorCodeDoc>` |
| 공통 | 애플리케이션 ping | GET | `/api/ping` | Public | `-` | `Map<String, String>` |

### 2.1 기존 명세에서 삭제한 stale endpoint

- `GET /api/stats/me/efficiency` - 효율 점수 통계 endpoint 없음
- `GET /api/gyms/{gymId}/board` - 한줄게시판은 채팅으로 대체
- `POST /api/gyms/{gymId}/board` - 한줄게시판은 채팅으로 대체
- `DELETE /api/gyms/board/{boardId}` - 한줄게시판은 채팅으로 대체
- `POST /api/gyms/{gymId}/photos` - 다중 사진 API 제거, admin profile-image로 대체
- `GET /api/gyms/{gymId}/photos` - 다중 사진 API 제거, profileImageUrl 응답으로 대체
- `GET /api/recommendations/videos/{videoId}/similar` - 비슷한 문제 endpoint 없음
- `POST /api/notifications/device-token` - /api/users/me/device-tokens로 이동
- `WS /ws/gyms/{gymId}/chat` - 실제 STOMP destination은 /app/gyms/{gymId}/chat

---

## 3. Domain Endpoint Details

### 3.1 회원/인증

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 이메일 중복 확인 | GET | `/api/auth/email-check` | `email:String` | `-` | - | `AvailabilityResponse` | - | `src/main/java/com/holaclimbing/server/domain/user/AuthController.java` |
| 이메일 인증 | POST | `/api/auth/email/verify` | `-` | `-` | body: `VerifyEmailRequest` | `Void` | U005(INVALID_TOKEN) | `src/main/java/com/holaclimbing/server/domain/user/AuthController.java` |
| 로그인 | POST | `/api/auth/login` | `-` | `-` | body: `LoginRequest` | `TokenResponse` | U001(USER_NOT_FOUND), U003(PASSWORD_MISMATCH), U004(EMAIL_NOT_VERIFIED), U012(USER_SUSPENDED) | `src/main/java/com/holaclimbing/server/domain/user/AuthController.java` |
| 로그아웃 | POST | `/api/auth/logout` | `-` | `-` | body: `LogoutRequest`<br>header: `authHeader:String` | `Void` | - | `src/main/java/com/holaclimbing/server/domain/user/AuthController.java` |
| 닉네임 중복 확인 | GET | `/api/auth/nickname-check` | `nickname:String` | `-` | - | `AvailabilityResponse` | - | `src/main/java/com/holaclimbing/server/domain/user/AuthController.java` |
| OAuth 결과 코드 소비 | POST | `/api/auth/oauth/result` | `-` | `-` | body: `OAuthResultRequest` | `OAuthLoginResponse` | U019(INVALID_OAUTH_RESULT_CODE) | `src/main/java/com/holaclimbing/server/domain/user/OAuthRedirectController.java` |
| OAuth 신규 회원가입 | POST | `/api/auth/oauth/signup` | `-` | `-` | body: `OAuthSignupRequest` | `TokenResponse` | U016(INVALID_OAUTH_SIGNUP_TOKEN), U017(OAUTH_EMAIL_ALREADY_EXISTS), U008(NICKNAME_ALREADY_EXISTS), U010(REQUIRED_TERMS_NOT_AGREED), U013(TERMS_NOT_CONFIGURED), U012(USER_SUSPENDED) | `src/main/java/com/holaclimbing/server/domain/user/OAuthRedirectController.java` |
| OAuth 인증 시작 redirect | GET | `/api/auth/oauth/{provider}/authorize` | `redirectUri:String` | `provider:String` | - | `302 redirect / no body` | U014(UNSUPPORTED_OAUTH_PROVIDER), U015(OAUTH_AUTHORIZATION_FAILED) | `src/main/java/com/holaclimbing/server/domain/user/OAuthRedirectController.java` |
| OAuth provider callback | GET | `/api/auth/oauth/{provider}/callback` | `code:String (optional), state:String, error:String (optional)` | `provider:String` | - | `302 redirect / no body` | U014(UNSUPPORTED_OAUTH_PROVIDER), U018(INVALID_OAUTH_STATE), U015(OAUTH_AUTHORIZATION_FAILED), U012(USER_SUSPENDED) | `src/main/java/com/holaclimbing/server/domain/user/OAuthRedirectController.java` |
| 비밀번호 재설정 | POST | `/api/auth/password/reset` | `-` | `-` | body: `PasswordResetRequest` | `Void` | U011(INVALID_RESET_TOKEN) | `src/main/java/com/holaclimbing/server/domain/user/AuthController.java` |
| 비밀번호 재설정 요청 | POST | `/api/auth/password/reset-request` | `-` | `-` | body: `PasswordResetEmailRequest` | `Void` | - | `src/main/java/com/holaclimbing/server/domain/user/AuthController.java` |
| 토큰 재발급 | POST | `/api/auth/refresh` | `-` | `-` | body: `RefreshRequest` | `TokenResponse` | U005(INVALID_TOKEN), U006(EXPIRED_TOKEN), U001(USER_NOT_FOUND), U012(USER_SUSPENDED) | `src/main/java/com/holaclimbing/server/domain/user/AuthController.java` |
| 이메일 인증 재발송 | POST | `/api/auth/resend-verification` | `-` | `-` | body: `ResendVerificationRequest` | `Void` | U001(USER_NOT_FOUND), C001(INVALID_INPUT) | `src/main/java/com/holaclimbing/server/domain/user/AuthController.java` |
| 회원가입 | POST | `/api/auth/signup` | `-` | `-` | body: `SignupRequest` | `SignupResponse` | U002(EMAIL_ALREADY_EXISTS), U008(NICKNAME_ALREADY_EXISTS), U010(REQUIRED_TERMS_NOT_AGREED), U013(TERMS_NOT_CONFIGURED) | `src/main/java/com/holaclimbing/server/domain/user/AuthController.java` |

### 3.2 사용자

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 회원 탈퇴 | DELETE | `/api/users/me` | `-` | `-` | body: `WithdrawRequest` | `Void` | U003(PASSWORD_MISMATCH), U001(USER_NOT_FOUND) | `src/main/java/com/holaclimbing/server/domain/user/UserProfileController.java` |
| 내 프로필 조회 | GET | `/api/users/me` | `-` | `-` | - | `MyProfileResponse` | - | `src/main/java/com/holaclimbing/server/domain/user/UserProfileController.java` |
| 내 프로필 수정 | PATCH | `/api/users/me` | `-` | `-` | body: `UpdateProfileRequest` | `MyProfileResponse` | U008(NICKNAME_ALREADY_EXISTS) | `src/main/java/com/holaclimbing/server/domain/user/UserProfileController.java` |
| 차단 목록 조회 | GET | `/api/users/me/blocks` | `page:int (default=0), size:int (default=20)` | `-` | - | `PageResponse<UserSummaryResponse>` | - | `src/main/java/com/holaclimbing/server/domain/user/UserProfileController.java` |
| 디바이스 토큰 해제 | DELETE | `/api/users/me/device-tokens` | `-` | `-` | body: `UnregisterDeviceTokenRequest` | `Void` | - | `src/main/java/com/holaclimbing/server/domain/user/DeviceTokenController.java` |
| 디바이스 토큰 등록 | POST | `/api/users/me/device-tokens` | `-` | `-` | body: `RegisterDeviceTokenRequest` | `Void` | - | `src/main/java/com/holaclimbing/server/domain/user/DeviceTokenController.java` |
| 내 프로필 이미지 업로드 | POST | `/api/users/me/profile-image` | `-` | `-` | form: `image:MultipartFile` | `MyProfileResponse` | C001(INVALID_INPUT), S001(GCS_UPLOAD_FAILED) | `src/main/java/com/holaclimbing/server/domain/user/UserProfileController.java` |
| 사용자 프로필 조회 | GET | `/api/users/{userId}` | `-` | `userId:Long` | - | `UserProfileResponse` | - | `src/main/java/com/holaclimbing/server/domain/user/UserProfileController.java` |
| 차단 해제 | DELETE | `/api/users/{userId}/block` | `-` | `userId:Long` | - | `Void` | - | `src/main/java/com/holaclimbing/server/domain/user/UserProfileController.java` |
| 사용자 차단 | POST | `/api/users/{userId}/block` | `-` | `userId:Long` | - | `Void` | C001(INVALID_INPUT) | `src/main/java/com/holaclimbing/server/domain/user/UserProfileController.java` |
| 언팔로우 | DELETE | `/api/users/{userId}/follow` | `-` | `userId:Long` | - | `Void` | - | `src/main/java/com/holaclimbing/server/domain/user/UserProfileController.java` |
| 팔로우 | POST | `/api/users/{userId}/follow` | `-` | `userId:Long` | - | `Void` | C001(INVALID_INPUT) | `src/main/java/com/holaclimbing/server/domain/user/UserProfileController.java` |
| 팔로워 목록 | GET | `/api/users/{userId}/followers` | `page:int (default=0), size:int (default=20)` | `userId:Long` | - | `PageResponse<UserSummaryResponse>` | - | `src/main/java/com/holaclimbing/server/domain/user/UserProfileController.java` |
| 팔로잉 목록 | GET | `/api/users/{userId}/following` | `page:int (default=0), size:int (default=20)` | `userId:Long` | - | `PageResponse<UserSummaryResponse>` | - | `src/main/java/com/holaclimbing/server/domain/user/UserProfileController.java` |

### 3.3 영상

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 댓글 삭제 | DELETE | `/api/comments/{commentId}` | `-` | `commentId:Long` | - | `Void` | C004(NOT_FOUND), C003(FORBIDDEN) | `src/main/java/com/holaclimbing/server/domain/video/CommentController.java` |
| 댓글 수정 | PATCH | `/api/comments/{commentId}` | `-` | `commentId:Long` | body: `UpdateCommentRequest` | `CommentResponse` | C004(NOT_FOUND), C003(FORBIDDEN) | `src/main/java/com/holaclimbing/server/domain/video/CommentController.java` |
| 영상 목록/피드 조회 | GET | `/api/videos` | `userId:Long (optional), cursor:String (optional), nextCursor:String (optional), recordedDate:LocalDate (optional), size:int (default=20)` | `-` | - | `CursorPageResponse<VideoSummaryResponse>` | C001(INVALID_INPUT) | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |
| 영상 등록 | POST | `/api/videos` | `-` | `-` | body: `CreateVideoRequest` | `VideoDetailResponse` | G001(GYM_NOT_FOUND), G005(INVALID_GYM_GRADE), V003(VIDEO_TOO_LONG), C001(INVALID_INPUT), C003(FORBIDDEN) | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |
| 영상 썸네일 업로드 | POST | `/api/videos/thumbnail` | `-` | `-` | form: `image:MultipartFile` | `ThumbnailUploadResponse` | C001(INVALID_INPUT), S001(GCS_UPLOAD_FAILED) | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |
| 영상 업로드 Signed URL 발급 | POST | `/api/videos/upload-url` | `-` | `-` | body: `UploadUrlRequest` | `UploadUrlResponse` | V004(UNSUPPORTED_VIDEO_FORMAT), V002(VIDEO_TOO_LARGE) | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |
| 영상 삭제 | DELETE | `/api/videos/{videoId}` | `-` | `videoId:Long` | - | `Void` | V001(VIDEO_NOT_FOUND), C003(FORBIDDEN) | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |
| 영상 상세 조회 | GET | `/api/videos/{videoId}` | `-` | `videoId:Long` | - | `VideoDetailResponse` | V001(VIDEO_NOT_FOUND), V006(VIDEO_NOT_ACCESSIBLE) | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |
| 영상 수정 | PATCH | `/api/videos/{videoId}` | `-` | `videoId:Long` | body: `UpdateVideoRequest` | `VideoDetailResponse` | V001(VIDEO_NOT_FOUND), G001(GYM_NOT_FOUND), G005(INVALID_GYM_GRADE), C001(INVALID_INPUT), C003(FORBIDDEN) | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |
| 댓글 목록 조회 | GET | `/api/videos/{videoId}/comments` | `page:int (default=0), size:int (default=20)` | `videoId:Long` | - | `PageResponse<CommentResponse>` | V001(VIDEO_NOT_FOUND), V006(VIDEO_NOT_ACCESSIBLE) | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |
| 댓글 작성 | POST | `/api/videos/{videoId}/comments` | `-` | `videoId:Long` | body: `CreateCommentRequest` | `CommentResponse` | V001(VIDEO_NOT_FOUND), V006(VIDEO_NOT_ACCESSIBLE), C001(INVALID_INPUT) | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |
| 영상 좋아요 취소 | DELETE | `/api/videos/{videoId}/like` | `-` | `videoId:Long` | - | `LikeResponse` | V001(VIDEO_NOT_FOUND), V006(VIDEO_NOT_ACCESSIBLE) | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |
| 영상 좋아요 | POST | `/api/videos/{videoId}/like` | `-` | `videoId:Long` | - | `LikeResponse` | V001(VIDEO_NOT_FOUND), V006(VIDEO_NOT_ACCESSIBLE), C001(INVALID_INPUT) | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |
| 영상 공유 링크 발급 | POST | `/api/videos/{videoId}/share` | `-` | `videoId:Long` | - | `ShareLinkResponse` | V001(VIDEO_NOT_FOUND), V006(VIDEO_NOT_ACCESSIBLE) | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |
| 분석 상태 조회 | GET | `/api/videos/{videoId}/status` | `-` | `videoId:Long` | - | `VideoStatusResponse` | V001(VIDEO_NOT_FOUND), V006(VIDEO_NOT_ACCESSIBLE) | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |

### 3.4 AI 분석

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| AI 워커 분석 결과 수신 | POST | `/api/analysis/videos/{videoId}` | `-` | `videoId:Long` | body: `AnalysisIngestRequest` | `VideoAnalysisResponse` | V001(VIDEO_NOT_FOUND), C001(INVALID_INPUT) | `src/main/java/com/holaclimbing/server/domain/analysis/AnalysisController.java` |
| 분석 결과 조회 | GET | `/api/videos/{videoId}/analysis` | `-` | `videoId:Long` | - | `VideoAnalysisResponse` | V001(VIDEO_NOT_FOUND), V006(VIDEO_NOT_ACCESSIBLE) | `src/main/java/com/holaclimbing/server/domain/analysis/AnalysisController.java` |
| 분석 결과 피드백 | POST | `/api/videos/{videoId}/analysis/feedback` | `-` | `videoId:Long` | body: `AnalysisFeedbackRequest` | `FeedbackResponse` | V001(VIDEO_NOT_FOUND), V006(VIDEO_NOT_ACCESSIBLE) | `src/main/java/com/holaclimbing/server/domain/analysis/AnalysisController.java` |
| 분석 재시도 | POST | `/api/videos/{videoId}/analysis/retry` | `-` | `videoId:Long` | - | `VideoAnalysisResponse` | V001(VIDEO_NOT_FOUND), C003(FORBIDDEN) | `src/main/java/com/holaclimbing/server/domain/analysis/AnalysisController.java` |
| 분석 진행률 SSE | GET | `/api/videos/{videoId}/analysis/stream` | `-` | `videoId:Long` | - | `SSE progress events` | - | `src/main/java/com/holaclimbing/server/domain/video/VideoController.java` |

### 3.5 통계

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 내 통계 조회 | GET | `/api/stats/me` | `-` | `-` | - | `UserStatsResponse` | U001(USER_NOT_FOUND) | `src/main/java/com/holaclimbing/server/domain/stats/StatsController.java` |
| 월간 달력 조회 | GET | `/api/stats/me/calendar` | `year:int, month:int` | `-` | - | `MonthlyCalendarResponse` | - | `src/main/java/com/holaclimbing/server/domain/stats/StatsController.java` |
| 날짜별 클라이밍 기록 조회 | GET | `/api/stats/me/calendar/{date}` | `-` | `date:LocalDate` | - | `List<ClimbingLogResponse>` | - | `src/main/java/com/holaclimbing/server/domain/stats/StatsController.java` |
| 내 암장 방문 랭킹 | GET | `/api/stats/me/gyms/rankings` | `month:String (optional YYYY-MM), limit:int (default 10, max 50), cursor:String (optional)` | `-` | - | `GymRankingResponse` | U001(USER_NOT_FOUND), T003(INVALID_MONTH), C001(INVALID_INPUT) | `src/main/java/com/holaclimbing/server/domain/stats/StatsController.java` |
| 월간 AI 리포트 조회 | GET | `/api/stats/me/monthly-reports` | `month:String (optional), gymId:Long (optional)` | `-` | - | `MonthlyReportResponse` | U001(USER_NOT_FOUND), G001(GYM_NOT_FOUND), T003(INVALID_MONTH), T004(MONTHLY_REPORT_GENERATION_FAILED) | `src/main/java/com/holaclimbing/server/domain/stats/MonthlyReportController.java` |
| 월간 리포트 생성 월 목록 | GET | `/api/stats/me/monthly-reports/available` | `-` | `-` | - | `MonthlyReportAvailablePeriodsResponse` | U001(USER_NOT_FOUND) | `src/main/java/com/holaclimbing/server/domain/stats/MonthlyReportController.java` |
| 내 기술별 통계 | GET | `/api/stats/me/techniques` | `-` | `-` | - | `TechniqueStatsResponse` | U001(USER_NOT_FOUND) | `src/main/java/com/holaclimbing/server/domain/stats/StatsController.java` |
| 사용자 공개 통계 조회 | GET | `/api/stats/users/{userId}` | `-` | `userId:Long` | - | `UserStatsResponse` | U001(USER_NOT_FOUND) | `src/main/java/com/holaclimbing/server/domain/stats/StatsController.java` |

### 3.6 클라이밍 기록

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 클라이밍 기록 생성 | POST | `/api/climbing-logs` | `-` | `-` | body: `CreateClimbingLogRequest` | `ClimbingLogResponse` | G001(GYM_NOT_FOUND) | `src/main/java/com/holaclimbing/server/domain/stats/ClimbingLogController.java` |
| 클라이밍 기록 삭제 | DELETE | `/api/climbing-logs/{logId}` | `-` | `logId:Long` | - | `Void` | T001(CLIMBING_LOG_NOT_FOUND), C003(FORBIDDEN) | `src/main/java/com/holaclimbing/server/domain/stats/ClimbingLogController.java` |
| 클라이밍 기록 상세 조회 | GET | `/api/climbing-logs/{logId}` | `-` | `logId:Long` | - | `ClimbingLogResponse` | T001(CLIMBING_LOG_NOT_FOUND), C003(FORBIDDEN) | `src/main/java/com/holaclimbing/server/domain/stats/ClimbingLogController.java` |
| 클라이밍 기록 수정 | PATCH | `/api/climbing-logs/{logId}` | `-` | `logId:Long` | body: `UpdateClimbingLogRequest` | `ClimbingLogResponse` | T001(CLIMBING_LOG_NOT_FOUND), C003(FORBIDDEN), G001(GYM_NOT_FOUND) | `src/main/java/com/holaclimbing/server/domain/stats/ClimbingLogController.java` |

### 3.7 암장

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 암장 목록/검색 | GET | `/api/gyms` | `keyword:String (optional), q:String (optional), name:String (optional), region:String (optional), page:int (default=0), size:int (default=20)` | `-` | - | `PageResponse<GymSummaryResponse>` | - | `src/main/java/com/holaclimbing/server/domain/gym/GymController.java` |
| 암장 등록 제안 | POST | `/api/gyms` | `-` | `-` | body: `CreateGymRequest` | `CreateGymResponse` | - | `src/main/java/com/holaclimbing/server/domain/gym/GymController.java` |
| 주변 암장 조회 | GET | `/api/gyms/nearby` | `lat:double, lng:double, radius:double (default=5), size:int (default=20)` | `-` | - | `List<GymSummaryResponse>` | C001(INVALID_INPUT) | `src/main/java/com/holaclimbing/server/domain/gym/GymController.java` |
| 암장 상세 조회 | GET | `/api/gyms/{gymId}` | `-` | `gymId:Long` | - | `GymDetailResponse` | G001(GYM_NOT_FOUND) | `src/main/java/com/holaclimbing/server/domain/gym/GymController.java` |
| 암장 운영시간 수정 | PATCH | `/api/gyms/{gymId}/business-hours` | `-` | `gymId:Long` | body: `UpdateBusinessHoursRequest` | `GymDetailResponse` | G001(GYM_NOT_FOUND), C003(FORBIDDEN) | `src/main/java/com/holaclimbing/server/domain/gym/GymController.java` |
| 암장 난이도 목록 | GET | `/api/gyms/{gymId}/grades` | `-` | `gymId:Long` | - | `List<GymGradeResponse>` | G001(GYM_NOT_FOUND) | `src/main/java/com/holaclimbing/server/domain/gym/GymController.java` |
| 암장 영상 목록 | GET | `/api/gyms/{gymId}/videos` | `gymGradeId:Long (optional), page:int (default=0), size:int (default=20)` | `gymId:Long` | - | `PageResponse<VideoSummaryResponse>` | - | `src/main/java/com/holaclimbing/server/domain/gym/GymController.java` |

### 3.8 암장 리뷰

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 암장 리뷰 삭제 | DELETE | `/api/gyms/reviews/{reviewId}` | `-` | `reviewId:Long` | - | `Void` | G004(REVIEW_NOT_FOUND), C003(FORBIDDEN) | `src/main/java/com/holaclimbing/server/domain/gym/GymReviewController.java` |
| 암장 리뷰 수정 | PATCH | `/api/gyms/reviews/{reviewId}` | `-` | `reviewId:Long` | body: `UpdateReviewRequest` | `GymReviewResponse` | G004(REVIEW_NOT_FOUND), C003(FORBIDDEN) | `src/main/java/com/holaclimbing/server/domain/gym/GymReviewController.java` |
| 암장 리뷰 목록 | GET | `/api/gyms/{gymId}/reviews` | `page:int (default=0), size:int (default=20)` | `gymId:Long` | - | `PageResponse<GymReviewResponse>` | - | `src/main/java/com/holaclimbing/server/domain/gym/GymReviewController.java` |
| 암장 리뷰 작성 | POST | `/api/gyms/{gymId}/reviews` | `-` | `gymId:Long` | body: `CreateReviewRequest` | `GymReviewResponse` | G001(GYM_NOT_FOUND), G003(ALREADY_REVIEWED) | `src/main/java/com/holaclimbing/server/domain/gym/GymReviewController.java` |

### 3.9 즐겨찾기

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 즐겨찾기 암장 목록 | GET | `/api/favorites/gyms` | `page:int (default=0), size:int (default=20)` | `-` | - | `PageResponse<GymSummaryResponse>` | - | `src/main/java/com/holaclimbing/server/domain/favorite/FavoriteController.java` |
| 암장 즐겨찾기 해제 | DELETE | `/api/favorites/gyms/{gymId}` | `-` | `gymId:Long` | - | `Void` | - | `src/main/java/com/holaclimbing/server/domain/favorite/FavoriteController.java` |
| 암장 즐겨찾기 추가 | POST | `/api/favorites/gyms/{gymId}` | `-` | `gymId:Long` | - | `Void` | G001(GYM_NOT_FOUND), C001(INVALID_INPUT) | `src/main/java/com/holaclimbing/server/domain/favorite/FavoriteController.java` |

### 3.10 추천

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 주변/스타일 암장 추천 | GET | `/api/recommendations/gyms` | `lat:double, lng:double, radius:double (default=10), size:int (default=20)` | `-` | - | `List<RecommendedGymResponse>` | - | `src/main/java/com/holaclimbing/server/domain/recommendation/RecommendationController.java` |
| 개인화 영상 추천 피드 | GET | `/api/recommendations/videos` | `cursor:String (optional), nextCursor:String (optional), size:int (default=20)` | `-` | - | `CursorPageResponse<RecommendedVideoResponse>` | - | `src/main/java/com/holaclimbing/server/domain/recommendation/RecommendationController.java` |
| 영상 기반 암장 추천 | GET | `/api/videos/{videoId}/recommendations/gyms` | `lat:double, lng:double, radius:double (default=10), size:int (default=20)` | `videoId:Long` | - | `List<VideoRecommendedGymResponse>` | V001(VIDEO_NOT_FOUND), V006(VIDEO_NOT_ACCESSIBLE), C001(INVALID_INPUT) | `src/main/java/com/holaclimbing/server/domain/video/VideoRecommendationController.java` |

### 3.11 알림

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 알림 목록 조회 | GET | `/api/notifications` | `unreadOnly:boolean (default=false), page:int (default=0), size:int (default=20)` | `-` | - | `PageResponse<NotificationResponse>` | - | `src/main/java/com/holaclimbing/server/domain/notification/NotificationController.java` |
| 알림 설정 조회 | GET | `/api/notifications/settings` | `-` | `-` | - | `NotificationSettingsResponse` | - | `src/main/java/com/holaclimbing/server/domain/notification/NotificationController.java` |
| 알림 설정 변경 | PATCH | `/api/notifications/settings` | `-` | `-` | body: `UpdateNotificationSettingsRequest` | `NotificationSettingsResponse` | - | `src/main/java/com/holaclimbing/server/domain/notification/NotificationController.java` |
| 미읽음 알림 수 | GET | `/api/notifications/unread-count` | `-` | `-` | - | `Long` | - | `src/main/java/com/holaclimbing/server/domain/notification/NotificationController.java` |
| 알림 읽음 처리 | PATCH | `/api/notifications/{id}/read` | `-` | `id:String` | - | `UnreadCountResponse` | N001(NOTIFICATION_NOT_FOUND), C001(INVALID_INPUT) | `src/main/java/com/holaclimbing/server/domain/notification/NotificationController.java` |
| 알림 삭제 | DELETE | `/api/notifications/{notificationId}` | `-` | `notificationId:Long` | - | `Void` | N001(NOTIFICATION_NOT_FOUND) | `src/main/java/com/holaclimbing/server/domain/notification/NotificationController.java` |

### 3.12 신고

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 신고 작성 | POST | `/api/reports` | `-` | `-` | body: `CreateReportRequest` | `ReportResponse` | C001(INVALID_INPUT), R001(SELF_REPORT_NOT_ALLOWED), R002(ALREADY_REPORTED), U001(USER_NOT_FOUND), V001(VIDEO_NOT_FOUND), C004(NOT_FOUND) | `src/main/java/com/holaclimbing/server/domain/report/ReportController.java` |

### 3.13 채팅

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 암장 채팅방 참여 | POST | `/api/chats/gyms/{gymId}/join` | `-` | `gymId:Long` | - | `ChatRoomResponse` | G001(GYM_NOT_FOUND) | `src/main/java/com/holaclimbing/server/domain/chat/ChatController.java` |
| 암장 채팅 메시지 조회 | GET | `/api/chats/gyms/{gymId}/messages` | `page:int (default=0), size:int (default=30)` | `gymId:Long` | - | `PageResponse<ChatMessageResponse>` | - | `src/main/java/com/holaclimbing/server/domain/chat/ChatController.java` |
| 암장 채팅 메시지 전송 | STOMP | `/app/gyms/{gymId}/chat` | `-` | `-` | destination: `gymId:Long` | `ChatMessageResponse` | - | `src/main/java/com/holaclimbing/server/domain/chat/ChatMessageController.java` |

### 3.14 관리자

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 관리자 AI 모델 지표 | GET | `/api/admin/analysis/models/{modelVersion}/metrics` | `-` | `modelVersion:String` | - | `AnalysisModelMetricsResponse` | C002(UNAUTHORIZED), C003(FORBIDDEN) | `src/main/java/com/holaclimbing/server/domain/analysis/AnalysisController.java` |
| 관리자 감사 로그 조회 | GET | `/api/admin/audit-logs` | `targetType:String (optional), targetId:Long (optional), adminId:Long (optional), page:int (default=0), size:int (default=20)` | `-` | - | `PageResponse<AdminAuditLogResponse>` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminAuditLogController.java` |
| 관리자 대시보드 | GET | `/api/admin/dashboard` | `-` | `-` | - | `AdminDashboardResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminDashboardController.java` |
| 관리자 암장 검색 | GET | `/api/admin/gyms` | `status:String (optional), keyword:String (optional), regionCode:String (optional), page:int (default=0), size:int (default=20)` | `-` | - | `PageResponse<AdminGymSearchResponse>` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminGymController.java` |
| 관리자 암장 생성 | POST | `/api/admin/gyms` | `-` | `-` | body: `AdminGymUpsertRequest` | `GymDetailResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminGymController.java` |
| 관리자 암장 일괄 import 적용 | POST | `/api/admin/gyms/import` | `-` | `-` | body: `AdminGymImportRequest` | `AdminGymImportApplyResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminGymController.java` |
| 관리자 암장 일괄 import 미리보기 | POST | `/api/admin/gyms/import/preview` | `-` | `-` | body: `AdminGymImportRequest` | `AdminGymImportPreviewResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminGymController.java` |
| 관리자 암장 상세 | GET | `/api/admin/gyms/{gymId}` | `-` | `gymId:Long` | - | `GymDetailResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminGymController.java` |
| 관리자 암장 수정 | PATCH | `/api/admin/gyms/{gymId}` | `-` | `gymId:Long` | body: `AdminGymUpsertRequest` | `GymDetailResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminGymController.java` |
| 암장 등록 제안 승인 | POST | `/api/admin/gyms/{gymId}/approve` | `-` | `gymId:Long` | body: `AdminReasonRequest` | `GymDetailResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminGymController.java` |
| 암장 폐업 처리 | POST | `/api/admin/gyms/{gymId}/close` | `-` | `gymId:Long` | body: `AdminReasonRequest` | `GymDetailResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminGymController.java` |
| 암장 난이도 전체 교체 | PUT | `/api/admin/gyms/{gymId}/grades` | `-` | `gymId:Long` | body: `AdminGymGradeReplaceRequest` | `List<GymGradeResponse>` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminGymController.java` |
| 관리자 암장 프로필 이미지 업로드 | POST | `/api/admin/gyms/{gymId}/profile-image` | `-` | `gymId:Long` | form: `image:MultipartFile` | `GymDetailResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminGymController.java` |
| 암장 등록 제안 반려 | POST | `/api/admin/gyms/{gymId}/reject` | `-` | `gymId:Long` | body: `AdminReasonRequest` | `GymDetailResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminGymController.java` |
| 관리자 신고 검색 | GET | `/api/admin/reports` | `status:String (optional), targetType:String (optional), category:String (optional), page:int (default=0), size:int (default=20)` | `-` | - | `PageResponse<AdminReportResponse>` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminReportController.java` |
| 관리자 신고 상태 변경 | PATCH | `/api/admin/reports/{reportId}/status` | `-` | `reportId:Long` | body: `AdminReportStatusRequest` | `AdminReportResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminReportController.java` |
| 관리자 사용자 검색 | GET | `/api/admin/users` | `status:String (optional), role:String (optional), keyword:String (optional), emailVerified:Boolean (optional), page:int (default=0), size:int (default=20)` | `-` | - | `PageResponse<AdminUserSearchResponse>` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminUserController.java` |
| 관리자 사용자 상세 | GET | `/api/admin/users/{userId}` | `-` | `userId:Long` | - | `AdminUserDetailResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminUserController.java` |
| 관리자 사용자 토큰 무효화 | POST | `/api/admin/users/{userId}/revoke-tokens` | `-` | `userId:Long` | body: `AdminReasonRequest` | `AdminUserDetailResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminUserController.java` |
| 관리자 사용자 권한 변경 | PATCH | `/api/admin/users/{userId}/role` | `-` | `userId:Long` | body: `AdminUserRoleRequest` | `AdminUserDetailResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminUserController.java` |
| 관리자 사용자 상태 변경 | PATCH | `/api/admin/users/{userId}/status` | `-` | `userId:Long` | body: `AdminUserStatusRequest` | `AdminUserDetailResponse` | - | `src/main/java/com/holaclimbing/server/domain/admin/AdminUserController.java` |

### 3.15 약관

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| 활성 약관 조회 | GET | `/api/terms` | `-` | `-` | - | `List<TermResponse>` | U013(TERMS_NOT_CONFIGURED) | `src/main/java/com/holaclimbing/server/domain/terms/TermsController.java` |
| 약관 동의 기록 | POST | `/api/terms/agree` | `-` | `-` | body: `AgreeTermsRequest` | `Void` | C001(INVALID_INPUT), U013(TERMS_NOT_CONFIGURED) | `src/main/java/com/holaclimbing/server/domain/terms/TermsController.java` |
| 내 약관 동의 상태 조회 | GET | `/api/terms/agreement-status` | `-` | `-` | - | `TermsAgreementStatusResponse` | U013(TERMS_NOT_CONFIGURED) | `src/main/java/com/holaclimbing/server/domain/terms/TermsController.java` |

### 3.16 공통

| 기능 | Method | Path | Query | Path Vars | Body/Form/Header | Response Data | Errors | Source |
|---|---|---|---|---|---|---|---|---|
| Actuator health | GET | `/actuator/health` | `-` | `-` | - | `Actuator health JSON` | - | `src/main/java/com/holaclimbing/server/common/security/SecurityConfig.java` |
| 에러 코드 카탈로그 | GET | `/api/docs/error-codes` | `-` | `-` | - | `List<ErrorCodeDoc>` | - | `src/main/java/com/holaclimbing/server/common/exception/docs/ErrorCodeDocsController.java` |
| 애플리케이션 ping | GET | `/api/ping` | `-` | `-` | - | `Map<String, String>` | - | `src/main/java/com/holaclimbing/server/domain/health/controller/HealthController.java` |

---

## 4. DTO Schema Index

아래 DTO 필드는 Java `record` 선언에서 추출했다. `@JsonAlias`가 붙은 필드는 요청에서 alias도 허용될 수 있다.

### 4.1 `AdminAuditLogResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/response/AdminAuditLogResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `adminId` | `Long` | `-` |
| `action` | `String` | `-` |
| `targetType` | `String` | `-` |
| `targetId` | `Long` | `-` |
| `reason` | `String` | `-` |
| `beforeJson` | `String` | `-` |
| `afterJson` | `String` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |

### 4.2 `AdminDashboardResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/response/AdminDashboardResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `pendingGymCount` | `long` | `-` |
| `pendingReportCount` | `long` | `-` |
| `failedAnalysisVideoCount` | `long` | `-` |
| `newUserCountToday` | `long` | `-` |

### 4.3 `AdminGymGradeReplaceRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/request/AdminGymGradeReplaceRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `grades` | `List<AdminGymGradeRequest>` | `@NotEmpty, @Size(max = 100), @Valid` |
| `reason` | `String` | `@Size(max = 500)` |

### 4.4 `AdminGymGradeRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/request/AdminGymGradeRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `label` | `String` | `@NotBlank, @Size(max = 50)` |
| `difficultyOrder` | `Integer` | `@NotNull` |

### 4.5 `AdminGymImportApplyResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/response/AdminGymImportApplyResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `importedCount` | `int` | `-` |

### 4.6 `AdminGymImportInvalidRowResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/response/AdminGymImportInvalidRowResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `rowIndex` | `int` | `-` |
| `externalKey` | `String` | `-` |
| `errors` | `List<String>` | `-` |

### 4.7 `AdminGymImportPreviewResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/response/AdminGymImportPreviewResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `totalCount` | `int` | `-` |
| `validCount` | `int` | `-` |
| `invalidCount` | `int` | `-` |
| `invalidRows` | `List<AdminGymImportInvalidRowResponse>` | `-` |

### 4.8 `AdminGymImportRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/request/AdminGymImportRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `rows` | `List<AdminGymImportRow>` | `@NotEmpty, @Size(max = 500), @Valid` |

### 4.9 `AdminGymImportRow`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/request/AdminGymImportRow.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `externalKey` | `String` | `-` |
| `name` | `String` | `-` |
| `address` | `String` | `-` |
| `lat` | `Double` | `-` |
| `lng` | `Double` | `-` |
| `regionCode` | `String` | `-` |
| `phone` | `String` | `-` |
| `website` | `String` | `-` |
| `description` | `String` | `-` |
| `businessHours` | `Map<String, DayHours>` | `-` |
| `grades` | `List<AdminGymGradeRequest>` | `-` |

### 4.10 `AdminGymSearchResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/response/AdminGymSearchResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `name` | `String` | `-` |
| `address` | `String` | `-` |
| `regionCode` | `String` | `-` |
| `status` | `String` | `-` |
| `createdBy` | `Long` | `-` |
| `ratingAvg` | `BigDecimal` | `-` |
| `ratingCount` | `int` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |
| `updatedAt` | `OffsetDateTime` | `-` |

### 4.11 `AdminGymUpsertRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/request/AdminGymUpsertRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `name` | `String` | `@NotBlank, @Size(max = 100)` |
| `address` | `String` | `@Size(max = 200)` |
| `lat` | `Double` | `-` |
| `lng` | `Double` | `-` |
| `phone` | `String` | `@Size(max = 30)` |
| `website` | `String` | `@Size(max = 300)` |
| `description` | `String` | `-` |
| `businessHours` | `Map<String, DayHours>` | `-` |
| `regionCode` | `String` | `@Size(max = 20)` |

### 4.12 `AdminReasonRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/request/AdminReasonRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `reason` | `String` | `@Size(max = 500)` |

### 4.13 `AdminReportResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/response/AdminReportResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `reporterId` | `Long` | `-` |
| `targetType` | `String` | `-` |
| `targetId` | `Long` | `-` |
| `category` | `String` | `-` |
| `reason` | `String` | `-` |
| `status` | `String` | `-` |
| `reviewedBy` | `Long` | `-` |
| `reviewedAt` | `OffsetDateTime` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |

### 4.14 `AdminReportStatusRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/request/AdminReportStatusRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `status` | `String` | `@NotBlank` |
| `resolutionAction` | `String` | `@NotBlank` |
| `reason` | `String` | `@Size(max = 500)` |

### 4.15 `AdminUserDetailResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/response/AdminUserDetailResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `email` | `String` | `-` |
| `nickname` | `String` | `-` |
| `profileImage` | `String` | `-` |
| `bio` | `String` | `-` |
| `role` | `String` | `-` |
| `status` | `String` | `-` |
| `emailVerified` | `boolean` | `-` |
| `lastLoginAt` | `OffsetDateTime` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |
| `updatedAt` | `OffsetDateTime` | `-` |

### 4.16 `AdminUserRoleRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/request/AdminUserRoleRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `role` | `String` | `@NotBlank` |
| `reason` | `String` | `@NotBlank, @Size(max = 500)` |

### 4.17 `AdminUserSearchResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/response/AdminUserSearchResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `email` | `String` | `-` |
| `nickname` | `String` | `-` |
| `role` | `String` | `-` |
| `status` | `String` | `-` |
| `emailVerified` | `boolean` | `-` |
| `lastLoginAt` | `OffsetDateTime` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |

### 4.18 `AdminUserStatusRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/admin/dto/request/AdminUserStatusRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `status` | `String` | `@NotBlank` |
| `reason` | `String` | `@Size(max = 500)` |

### 4.19 `AgreeTermsRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/terms/dto/request/AgreeTermsRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `agreements` | `List<TermAgreementRequest>` | `@NotEmpty, @Valid` |

### 4.20 `AnalysisFeedbackRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/analysis/dto/request/AnalysisFeedbackRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `techniques` | `List<String>` | `@NotBlank` |
| `isDynamic` | `Boolean` | `@JsonAlias("is_dynamic")` |
| `note` | `String` | `-` |
| `techniqueLabel` | `String` | `@JsonAlias("technique_label")` |
| `timestampSec` | `Double` | `@JsonAlias("timestamp_sec")` |
| `isCorrect` | `Boolean` | `@JsonAlias("is_correct")` |
| `correctLabel` | `String` | `@JsonAlias("correct_label")` |

### 4.21 `AnalysisIngestRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/analysis/dto/request/AnalysisIngestRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `status` | `String` | `@NotEmpty` |
| `modelVersion` | `String` | `@JsonAlias("model_version")` |
| `segments` | `List<AnalysisSegmentPayload>` | `@Valid` |
| `techniques` | `List<String>` | `-` |
| `isDynamic` | `Boolean` | `@JsonAlias("is_dynamic")` |
| `dynamicProbability` | `Float` | `@JsonAlias("dynamic_probability"), @DecimalMin("0.0"), @DecimalMax("1.0")` |

### 4.22 `AnalysisModelMetricsResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/analysis/dto/response/AnalysisModelMetricsResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `modelVersion` | `String` | `-` |
| `feedbackCount` | `long` | `-` |
| `dynamicEvaluatedCount` | `long` | `-` |
| `dynamicAccuracy` | `Double` | `-` |
| `techniqueExactMatchAccuracy` | `Double` | `-` |
| `perTechnique` | `Map<String, AnalysisTechniqueMetricResponse>` | `-` |

### 4.23 `AnalysisSegmentPayload`

- Source: `src/main/java/com/holaclimbing/server/domain/analysis/dto/request/AnalysisSegmentPayload.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `sequenceIndex` | `Integer` | `@JsonAlias("sequence_index"), @NotNull, @PositiveOrZero` |
| `startTimeMs` | `Integer` | `@JsonAlias("start_time_ms")` |
| `endTimeMs` | `Integer` | `@JsonAlias("end_time_ms")` |
| `technique` | `String` | `@NotBlank` |
| `isDynamic` | `Boolean` | `@JsonAlias("is_dynamic")` |
| `confidence` | `Float` | `-` |

### 4.24 `AnalysisSegmentResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/analysis/dto/response/AnalysisSegmentResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `sequenceIndex` | `int` | `-` |
| `startTimeMs` | `Integer` | `-` |
| `endTimeMs` | `Integer` | `-` |
| `technique` | `String` | `-` |
| `isDynamic` | `Boolean` | `-` |
| `confidence` | `Float` | `-` |

### 4.25 `AnalysisTechniqueMetricResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/analysis/dto/response/AnalysisTechniqueMetricResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `truePositive` | `long` | `-` |
| `falsePositive` | `long` | `-` |
| `falseNegative` | `long` | `-` |
| `trueNegative` | `long` | `-` |
| `accuracy` | `Double` | `-` |
| `precision` | `Double` | `-` |
| `recall` | `Double` | `-` |
| `f1` | `Double` | `-` |

### 4.26 `AvailabilityResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/response/AvailabilityResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `available` | `boolean` | `-` |

### 4.27 `CalendarDayResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/stats/dto/response/CalendarDayResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `date` | `LocalDate` | `-` |
| `logCount` | `int` | `-` |
| `totalProblems` | `int` | `-` |
| `videoCount` | `int` | `-` |

### 4.28 `ChatMessageResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/chat/dto/response/ChatMessageResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `roomId` | `Long` | `-` |
| `userId` | `Long` | `-` |
| `nickname` | `String` | `-` |
| `content` | `String` | `-` |
| `verifiedAtGym` | `boolean` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |

### 4.29 `ChatRoomResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/chat/dto/response/ChatRoomResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `gymId` | `Long` | `-` |
| `name` | `String` | `-` |

### 4.30 `ClimbingLogResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/stats/dto/response/ClimbingLogResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `userId` | `Long` | `-` |
| `gymId` | `Long` | `-` |
| `climbedOn` | `LocalDate` | `-` |
| `gradeCounts` | `Map<String, Integer>` | `-` |
| `totalProblems` | `int` | `-` |
| `memo` | `String` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |
| `updatedAt` | `OffsetDateTime` | `-` |

### 4.31 `CommentResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/response/CommentResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `videoId` | `Long` | `-` |
| `userId` | `Long` | `-` |
| `nickname` | `String` | `-` |
| `profileImage` | `String` | `-` |
| `parentId` | `Long` | `-` |
| `content` | `String` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |

### 4.32 `CreateClimbingLogRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/stats/dto/request/CreateClimbingLogRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `gymId` | `Long` | `@NotNull` |
| `climbedOn` | `LocalDate` | `@NotNull` |
| `gradeCounts` | `Map<String, Integer>` | `@NotEmpty, @NotBlank, @PositiveOrZero` |
| `memo` | `String` | `@Size(max = 1000)` |

### 4.33 `CreateCommentRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/request/CreateCommentRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `content` | `String` | `@NotBlank, @Size(max = 1000)` |
| `parentId` | `Long` | `-` |

### 4.34 `CreateGymRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/gym/dto/request/CreateGymRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `name` | `String` | `@NotBlank, @Size(max = 100)` |
| `address` | `String` | `@Size(max = 200)` |
| `lat` | `Double` | `-` |
| `lng` | `Double` | `-` |
| `phone` | `String` | `@Size(max = 30)` |
| `website` | `String` | `@Size(max = 300)` |
| `description` | `String` | `-` |
| `businessHours` | `Map<String, DayHours>` | `-` |
| `regionCode` | `String` | `@Size(max = 20)` |

### 4.35 `CreateGymResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/gym/dto/response/CreateGymResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `name` | `String` | `-` |
| `status` | `String` | `-` |

### 4.36 `CreateReportRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/report/dto/request/CreateReportRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `targetType` | `String` | `@NotBlank` |
| `targetId` | `Long` | `@NotNull, @Positive` |
| `category` | `String` | `@NotBlank` |
| `reason` | `String` | `@Size(max = 500)` |

### 4.37 `CreateReviewRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/gym/dto/request/CreateReviewRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `rating` | `Integer` | `@NotNull, @Min(1), @Max(5)` |
| `content` | `String` | `@Size(max = 1000)` |

### 4.38 `CreateVideoRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/request/CreateVideoRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `gymId` | `Long` | `@NotNull` |
| `title` | `String` | `@Size(max = 100)` |
| `description` | `String` | `-` |
| `gymGradeId` | `Long` | `@NotNull` |
| `objectPath` | `String` | `@NotBlank, @Size(max = 500)` |
| `thumbnailPath` | `String` | `@Size(max = 500)` |
| `durationSeconds` | `Integer` | `@Positive` |
| `recordedDate` | `LocalDate` | `@NotNull` |
| `isPublic` | `Boolean` | `-` |

### 4.39 `DayHours`

- Source: `src/main/java/com/holaclimbing/server/domain/gym/dto/DayHours.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `open` | `String` | `-` |
| `close` | `String` | `-` |

### 4.40 `ErrorCodeDoc`

- Source: `src/main/java/com/holaclimbing/server/common/exception/docs/ErrorCodeDoc.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `code` | `String` | `@Schema(example = "V001")` |
| `status` | `int` | `@Schema(example = "404")` |
| `domain` | `String` | `@Schema(example = "비디오")` |
| `defaultMessage` | `String` | `@Schema(example = "영상을 찾을 수 없습니다.")` |

### 4.41 `FeedbackResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/analysis/dto/response/FeedbackResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `videoId` | `Long` | `-` |

### 4.42 `GymDetailResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/gym/dto/response/GymDetailResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `name` | `String` | `-` |
| `address` | `String` | `-` |
| `lat` | `Double` | `-` |
| `lng` | `Double` | `-` |
| `description` | `String` | `-` |
| `phone` | `String` | `-` |
| `website` | `String` | `-` |
| `thumbnailUrl` | `String` | `-` |
| `businessHours` | `Map<String, DayHours>` | `-` |
| `regionCode` | `String` | `-` |
| `ratingAvg` | `BigDecimal` | `-` |
| `ratingCount` | `int` | `-` |
| `status` | `String` | `-` |
| `isFavorite` | `boolean` | `-` |

### 4.43 `GymGradeResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/gym/dto/response/GymGradeResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `gymId` | `Long` | `-` |
| `label` | `String` | `-` |
| `difficultyOrder` | `int` | `-` |

### 4.44 `GymReviewResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/gym/dto/response/GymReviewResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `gymId` | `Long` | `-` |
| `userId` | `Long` | `-` |
| `nickname` | `String` | `-` |
| `profileImage` | `String` | `-` |
| `rating` | `int` | `-` |
| `content` | `String` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |
| `updatedAt` | `OffsetDateTime` | `-` |

### 4.45 `GymSummaryResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/gym/dto/response/GymSummaryResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `name` | `String` | `-` |
| `address` | `String` | `-` |
| `thumbnailUrl` | `String` | `-` |
| `regionCode` | `String` | `-` |
| `ratingAvg` | `BigDecimal` | `-` |
| `ratingCount` | `int` | `-` |
| `businessHours` | `Map<String, DayHours>` | `-` |
| `isOpen` | `boolean` | `-` |
| `isFavorite` | `boolean` | `-` |
| `distanceKm` | `Double` | 주변/거리 기반 조회에서 현재 위치 기준 거리(km), 없으면 생략 가능 |

### 4.46 `LikeResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/response/LikeResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `isLiked` | `boolean` | `-` |
| `likeCount` | `long` | `-` |

### 4.47 `LoginRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/LoginRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `email` | `String` | `@NotBlank, @Email` |
| `password` | `String` | `@NotBlank` |

### 4.48 `LogoutRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/LogoutRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `refreshToken` | `String` | `@NotBlank` |

### 4.49 `MonthlyCalendarResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/stats/dto/response/MonthlyCalendarResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `year` | `int` | `-` |
| `month` | `int` | `-` |
| `totalVideos` | `int` | `-` |
| `totalProblems` | `int` | `-` |
| `totalVideoDurationSeconds` | `long` | `-` |
| `totalGymVisits` | `int` | `-` |
| `days` | `List<CalendarDayResponse>` | `-` |

### 4.50 `MonthlyReportAvailablePeriodsResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/stats/dto/response/MonthlyReportAvailablePeriodsResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `periods` | `List<String>` | `-` |

### 4.51 `MonthlyReportResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/stats/dto/response/MonthlyReportResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `period` | `String` | `-` |
| `status` | `String` | `-` |
| `source` | `String` | `-` |
| `generatedAt` | `OffsetDateTime` | `-` |
| `metrics` | `Metrics` | `-` |
| `grade` | `Grade` | `-` |
| `tip` | `Tip` | `-` |
| `nextMonthGoal` | `Goal` | `-` |
| `recommendedGyms` | `List<RecommendedGym>` | `-` |
| `narrative` | `Narrative` | `-` |
| `requirement` | `Requirement` | `-` |

#### 4.51.1 `MonthlyReportResponse.Metrics`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `sessions` | `int` | `-` |
| `videos` | `int` | `-` |
| `analyzedVideos` | `int` | `-` |
| `problemsSolved` | `int` | `-` |
| `gymsVisited` | `int` | `-` |
| `primaryGymId` | `Long` | 없으면 생략 가능 |
| `primaryGymName` | `String` | 없으면 생략 가능 |
| `dynamicCount` | `int` | `-` |
| `staticCount` | `int` | `-` |
| `dynamicRatio` | `Double` | 없으면 생략 가능 |
| `staticRatio` | `Double` | 없으면 생략 가능 |
| `techniqueCounts` | `Map<String, Integer>` | `-` |

#### 4.51.2 `MonthlyReportResponse.Grade`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `gymId` | `Long` | 없으면 생략 가능 |
| `gymName` | `String` | 없으면 생략 가능 |
| `maxGrade` | `String` | 없으면 생략 가능 |
| `maxGradePrevMonth` | `String` | 없으면 생략 가능 |

#### 4.51.3 `MonthlyReportResponse.Tip`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `type` | `String` | `-` |
| `techniqueKeys` | `List<String>` | `-` |
| `message` | `String` | `-` |

#### 4.51.4 `MonthlyReportResponse.Goal`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `title` | `String` | `-` |
| `metric` | `String` | `-` |
| `target` | `int` | `-` |
| `techniqueKeys` | `List<String>` | `-` |
| `rationale` | `String` | `-` |

#### 4.51.5 `MonthlyReportResponse.RecommendedGym`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `gymId` | `Long` | `-` |
| `name` | `String` | `-` |
| `matchedTechniqueKeys` | `List<String>` | `-` |
| `matchingVideoCount` | `int` | `-` |
| `reason` | `String` | `-` |

#### 4.51.6 `MonthlyReportResponse.Narrative`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `headline` | `String` | `-` |
| `summary` | `String` | `-` |
| `highlights` | `List<String>` | `-` |

#### 4.51.7 `MonthlyReportResponse.Requirement`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `minVideos` | `int` | 리포트 생성 최소 분석 영상 수 |
| `minProblems` | `int` | 리포트 생성 최소 문제 수 |

### 4.52 `MyProfileResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/response/MyProfileResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `userId` | `Long` | `-` |
| `email` | `String` | `-` |
| `nickname` | `String` | `-` |
| `profileImage` | `String` | `-` |
| `bio` | `String` | `-` |
| `emailVerified` | `boolean` | `-` |
| `followerCount` | `long` | `-` |
| `followingCount` | `long` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |

### 4.53 `NotificationResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/notification/dto/response/NotificationResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `type` | `String` | `-` |
| `targetType` | `String` | `-` |
| `targetId` | `Long` | `-` |
| `senderId` | `Long` | `-` |
| `title` | `String` | `-` |
| `content` | `String` | `-` |
| `isRead` | `boolean` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |

### 4.54 `NotificationSettingsResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/notification/dto/response/NotificationSettingsResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `notifyComment` | `boolean` | `-` |
| `notifyReply` | `boolean` | `-` |
| `notifyLike` | `boolean` | `-` |
| `notifyFollow` | `boolean` | `-` |
| `notifyChat` | `boolean` | `-` |
| `notifySystem` | `boolean` | `-` |

### 4.55 `OAuthLoginResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/response/OAuthLoginResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `status` | `Status` | `-` |
| `signupRequired` | `boolean` | `-` |
| `token` | `TokenResponse` | `-` |
| `signupToken` | `String` | `-` |
| `email` | `String` | `-` |
| `suggestedNickname` | `String` | `-` |
| `profileImage` | `String` | `-` |

### 4.56 `OAuthResultRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/OAuthResultRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `code` | `String` | `@NotBlank` |

### 4.57 `OAuthSignupRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/OAuthSignupRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `signupToken` | `String` | `@NotBlank` |
| `nickname` | `String` | `@NotBlank, @Size(min = 2, max = 20)` |
| `termsAgreed` | `List<TermAgreementRequest>` | `@Valid` |

### 4.58 `PasswordResetEmailRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/PasswordResetEmailRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `email` | `String` | `@NotBlank, @Email` |

### 4.59 `PasswordResetRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/PasswordResetRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `token` | `String` | `@NotBlank` |
| `newPassword` | `String` | `@NotBlank, @Size(min = 8, max = 64)` |

### 4.60 `RecommendedGymResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/recommendation/dto/response/RecommendedGymResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `name` | `String` | `-` |
| `address` | `String` | `-` |
| `thumbnailUrl` | `String` | `-` |
| `regionCode` | `String` | `-` |
| `ratingAvg` | `BigDecimal` | `-` |
| `ratingCount` | `int` | `-` |
| `businessHours` | `Map<String, DayHours>` | `-` |
| `isOpen` | `boolean` | `-` |
| `isFavorite` | `boolean` | `-` |
| `distanceKm` | `Double` | `-` |
| `rankingDistance` | `Double` | `-` |
| `source` | `String` | `-` |

### 4.61 `RecommendedVideoResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/recommendation/dto/response/RecommendedVideoResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `userId` | `Long` | `-` |
| `nickname` | `String` | `-` |
| `profileImage` | `String` | `-` |
| `gymId` | `Long` | `-` |
| `gymName` | `String` | `-` |
| `gymGrade` | `GymGradeResponse` | `-` |
| `title` | `String` | `-` |
| `thumbnailPath` | `String` | `-` |
| `thumbnailUrl` | `String` | `-` |
| `streamUrl` | `String` | `-` |
| `durationSeconds` | `Integer` | `-` |
| `recordedDate` | `LocalDate` | `-` |
| `status` | `String` | `-` |
| `viewCount` | `int` | `-` |
| `likeCount` | `int` | `-` |
| `commentCount` | `int` | `-` |
| `source` | `String` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |

### 4.62 `RefreshRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/RefreshRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `refreshToken` | `String` | `@NotBlank` |

### 4.63 `RegisterDeviceTokenRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/RegisterDeviceTokenRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `token` | `String` | `@NotBlank, @Size(max = 500)` |
| `platform` | `String` | `@NotBlank, @Pattern(regexp = "ios\|android\|web")` |

### 4.64 `ReportResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/report/dto/response/ReportResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `reportId` | `Long` | `-` |

### 4.65 `ResendVerificationRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/ResendVerificationRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `email` | `String` | `@NotBlank, @Email` |

### 4.66 `SendMessageRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/chat/dto/request/SendMessageRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `content` | `String` | `-` |
| `lat` | `Double` | `-` |
| `lng` | `Double` | `-` |

### 4.67 `ShareLinkResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/response/ShareLinkResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `shareUrl` | `String` | `-` |

### 4.68 `SignupRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/SignupRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `email` | `String` | `@NotBlank, @Email, @Size(max = 255)` |
| `password` | `String` | `@NotBlank, @Size(min = 8, max = 64)` |
| `nickname` | `String` | `@NotBlank, @Size(min = 2, max = 20)` |
| `termsAgreed` | `List<TermAgreementRequest>` | `@Valid` |

### 4.69 `SignupResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/response/SignupResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `userId` | `Long` | `-` |
| `email` | `String` | `-` |
| `nickname` | `String` | `-` |
| `emailVerified` | `boolean` | `-` |

### 4.70 `TechniqueStatsResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/stats/dto/response/TechniqueStatsResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `techniqueCounts` | `Map<String, Integer>` | `-` |
| `mostUsed` | `String` | `-` |
| `leastUsed` | `String` | `-` |

### 4.71 `TermAgreementRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/terms/dto/request/TermAgreementRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `termId` | `Long` | `@NotNull` |
| `agreed` | `Boolean` | `@NotNull` |

### 4.72 `TermAgreementStatusItemResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/terms/dto/response/TermAgreementStatusItemResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `termId` | `Long` | `-` |
| `type` | `String` | `-` |
| `version` | `String` | `-` |
| `title` | `String` | `-` |
| `required` | `boolean` | `-` |
| `agreed` | `boolean` | `-` |
| `agreedAt` | `OffsetDateTime` | `-` |

### 4.73 `TermResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/terms/dto/response/TermResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `termId` | `Long` | `-` |
| `type` | `String` | `-` |
| `version` | `String` | `-` |
| `required` | `boolean` | `-` |
| `title` | `String` | `-` |
| `content` | `String` | `-` |

### 4.74 `TermsAgreementStatusResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/terms/dto/response/TermsAgreementStatusResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `allRequiredAgreed` | `boolean` | `-` |
| `terms` | `List<TermAgreementStatusItemResponse>` | `-` |

### 4.75 `ThumbnailUploadResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/response/ThumbnailUploadResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `thumbnailPath` | `String` | `-` |
| `thumbnailUrl` | `String` | `-` |

### 4.76 `TokenResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/response/TokenResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `accessToken` | `String` | `-` |
| `refreshToken` | `String` | `-` |
| `tokenType` | `String` | `-` |

### 4.77 `UnreadCountResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/notification/dto/response/UnreadCountResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `unreadCount` | `long` | `-` |

### 4.78 `UnregisterDeviceTokenRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/UnregisterDeviceTokenRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `token` | `String` | `@NotBlank, @Size(max = 500)` |

### 4.79 `UpdateBusinessHoursRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/gym/dto/request/UpdateBusinessHoursRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `businessHours` | `Map<String, DayHours>` | `@NotNull` |

### 4.80 `UpdateClimbingLogRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/stats/dto/request/UpdateClimbingLogRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `gymId` | `Long` | `@NotNull` |
| `climbedOn` | `LocalDate` | `@NotNull` |
| `gradeCounts` | `Map<String, Integer>` | `@NotEmpty, @NotBlank, @PositiveOrZero` |
| `memo` | `String` | `@Size(max = 1000)` |

### 4.81 `UpdateCommentRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/request/UpdateCommentRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `content` | `String` | `@NotBlank, @Size(max = 1000)` |

### 4.82 `UpdateNotificationSettingsRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/notification/dto/request/UpdateNotificationSettingsRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `notifyComment` | `Boolean` | `-` |
| `notifyReply` | `Boolean` | `-` |
| `notifyLike` | `Boolean` | `-` |
| `notifyFollow` | `Boolean` | `-` |
| `notifyChat` | `Boolean` | `-` |
| `notifySystem` | `Boolean` | `-` |

### 4.83 `UpdateProfileRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/UpdateProfileRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `nickname` | `String` | `@Size(min = 2, max = 20)` |
| `profileImage` | `String` | `@Size(max = 500)` |
| `bio` | `String` | `@Size(max = 1000)` |

### 4.84 `UpdateReviewRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/gym/dto/request/UpdateReviewRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `rating` | `Integer` | `@NotNull, @Min(1), @Max(5)` |
| `content` | `String` | `@Size(max = 1000)` |

### 4.85 `UpdateVideoRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/request/UpdateVideoRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `title` | `String` | `@Size(max = 100)` |
| `description` | `String` | `-` |
| `isPublic` | `Boolean` | `-` |
| `gymId` | `Long` | `-` |
| `gymGradeId` | `Long` | `-` |
| `recordedDate` | `LocalDate` | `-` |

### 4.86 `UploadUrlRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/request/UploadUrlRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `fileName` | `String` | `@NotBlank` |
| `fileSize` | `Long` | `@NotNull, @Positive` |
| `mimeType` | `String` | `@NotBlank` |

### 4.87 `UploadUrlResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/response/UploadUrlResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `uploadUrl` | `String` | `-` |
| `objectPath` | `String` | `-` |
| `expiresIn` | `long` | `-` |

### 4.88 `UserProfileResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/response/UserProfileResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `userId` | `Long` | `-` |
| `nickname` | `String` | `-` |
| `profileImage` | `String` | `-` |
| `bio` | `String` | `-` |
| `followerCount` | `long` | `-` |
| `followingCount` | `long` | `-` |
| `isFollowing` | `boolean` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |

### 4.89 `UserStatsResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/stats/dto/response/UserStatsResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `userId` | `Long` | `-` |
| `totalVideos` | `int` | `-` |
| `totalClimbingSeconds` | `long` | `-` |
| `techniqueCounts` | `Map<String, Integer>` | `-` |
| `dynamicCount` | `long` | `-` |
| `staticCount` | `long` | `-` |
| `isDynamic` | `boolean` | `-` |
| `lastClimbedAt` | `OffsetDateTime` | `-` |

### 4.90 `UserSummaryResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/response/UserSummaryResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `userId` | `Long` | `-` |
| `nickname` | `String` | `-` |
| `profileImage` | `String` | `-` |
| `bio` | `String` | `-` |

### 4.91 `VerifyEmailRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/VerifyEmailRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `token` | `String` | `@NotBlank` |

### 4.92 `VideoAnalysisResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/analysis/dto/response/VideoAnalysisResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `videoId` | `Long` | `-` |
| `status` | `String` | `-` |
| `modelVersion` | `String` | `-` |
| `techniques` | `List<String>` | `-` |
| `isDynamic` | `Boolean` | `-` |
| `dynamicProbability` | `Float` | `-` |
| `feedbackApplied` | `boolean` | `-` |

### 4.93 `VideoDetailResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/response/VideoDetailResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `userId` | `Long` | `-` |
| `gymId` | `Long` | `-` |
| `gymName` | `String` | `-` |
| `gymGrade` | `GymGradeResponse` | `-` |
| `title` | `String` | `-` |
| `description` | `String` | `-` |
| `gcsPath` | `String` | `-` |
| `gcsStreamingPath` | `String` | `-` |
| `thumbnailPath` | `String` | `-` |
| `thumbnailUrl` | `String` | `-` |
| `streamUrl` | `String` | `-` |
| `durationSeconds` | `Integer` | `-` |
| `recordedDate` | `LocalDate` | `-` |
| `status` | `String` | `-` |
| `isPublic` | `boolean` | `-` |
| `viewCount` | `int` | `-` |
| `likeCount` | `int` | `-` |
| `commentCount` | `int` | `-` |
| `isLiked` | `boolean` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |
| `updatedAt` | `OffsetDateTime` | `-` |

### 4.94 `VideoFeedCursor`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/VideoFeedCursor.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `recordedDate` | `LocalDate` | `-` |
| `id` | `Long` | `-` |

### 4.95 `VideoRecommendedGymResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/response/VideoRecommendedGymResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `name` | `String` | `-` |
| `address` | `String` | `-` |
| `thumbnailUrl` | `String` | `-` |
| `regionCode` | `String` | `-` |
| `ratingAvg` | `BigDecimal` | `-` |
| `ratingCount` | `int` | `-` |
| `businessHours` | `Map<String, DayHours>` | `-` |
| `isOpen` | `boolean` | `-` |
| `isFavorite` | `boolean` | `-` |
| `distanceKm` | `Double` | `-` |
| `similarityScore` | `Double` | `-` |
| `techniqueScore` | `Double` | `-` |
| `dynamicScore` | `Double` | `-` |
| `locationRatingScore` | `Double` | `-` |
| `source` | `String` | `-` |
| `reasons` | `List<String>` | `-` |

### 4.96 `VideoStatusResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/response/VideoStatusResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `videoId` | `Long` | `-` |
| `status` | `String` | `-` |
| `progress` | `int` | `-` |
| `stage` | `String` | `-` |
| `estimatedSecondsRemaining` | `Integer` | `-` |

### 4.97 `VideoSummaryResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/video/dto/response/VideoSummaryResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `id` | `Long` | `-` |
| `userId` | `Long` | `-` |
| `nickname` | `String` | `-` |
| `profileImage` | `String` | `-` |
| `gymId` | `Long` | `-` |
| `gymName` | `String` | `-` |
| `gymGrade` | `GymGradeResponse` | `-` |
| `title` | `String` | `-` |
| `thumbnailPath` | `String` | `-` |
| `thumbnailUrl` | `String` | `-` |
| `streamUrl` | `String` | `-` |
| `durationSeconds` | `Integer` | `-` |
| `recordedDate` | `LocalDate` | `-` |
| `status` | `String` | `-` |
| `viewCount` | `int` | `-` |
| `likeCount` | `int` | `-` |
| `commentCount` | `int` | `-` |
| `createdAt` | `OffsetDateTime` | `-` |

### 4.98 `WithdrawRequest`

- Source: `src/main/java/com/holaclimbing/server/domain/user/dto/request/WithdrawRequest.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `password` | `String` | `@NotBlank` |
| `reason` | `String` | `-` |

### 4.99 `GymRankingResponse`

- Source: `src/main/java/com/holaclimbing/server/domain/stats/dto/response/GymRankingResponse.java`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `period` | `String` | `month`가 없으면 생략 |
| `scope` | `String` | `monthly` 또는 `all` |
| `sort` | `String` | `mostVisited` |
| `content` | `List<Item>` | `visitCount DESC`, `latestVisitDate DESC`, `gymId ASC` |
| `nextCursor` | `String` | 다음 페이지가 없으면 생략 |
| `hasNext` | `boolean` | `-` |

#### 4.99.1 `GymRankingResponse.Item`

| Field | Type | Annotation / Constraint |
|---|---|---|
| `rank` | `int` | `1`부터 시작, cursor 페이지에서는 이어지는 순번 |
| `gymId` | `Long` | `-` |
| `gymName` | `String` | `-` |
| `visitCount` | `int` | `climbing_logs` 기록 수 |
| `latestVisitDate` | `LocalDate` | 해당 암장의 마지막 기록일 |

---

## 5. Error Code Catalog

| Enum | HTTP | Code | Message |
|---|---|---|---|
| `INVALID_INPUT` | `BAD_REQUEST` | `C001` | 입력값이 올바르지 않습니다. |
| `UNAUTHORIZED` | `UNAUTHORIZED` | `C002` | 인증이 필요합니다. |
| `FORBIDDEN` | `FORBIDDEN` | `C003` | 접근 권한이 없습니다. |
| `NOT_FOUND` | `NOT_FOUND` | `C004` | 리소스를 찾을 수 없습니다. |
| `METHOD_NOT_ALLOWED` | `METHOD_NOT_ALLOWED` | `C005` | 허용되지 않은 메서드입니다. |
| `INTERNAL_ERROR` | `INTERNAL_SERVER_ERROR` | `C999` | 서버 오류가 발생했습니다. |
| `USER_NOT_FOUND` | `NOT_FOUND` | `U001` | 사용자를 찾을 수 없습니다. |
| `EMAIL_ALREADY_EXISTS` | `CONFLICT` | `U002` | 이미 사용 중인 이메일입니다. |
| `PASSWORD_MISMATCH` | `UNAUTHORIZED` | `U003` | 비밀번호가 일치하지 않습니다. |
| `EMAIL_NOT_VERIFIED` | `FORBIDDEN` | `U004` | 이메일 인증이 필요합니다. |
| `INVALID_TOKEN` | `UNAUTHORIZED` | `U005` | 유효하지 않은 토큰입니다. |
| `EXPIRED_TOKEN` | `UNAUTHORIZED` | `U006` | 만료된 토큰입니다. |
| `USER_BLOCKED` | `FORBIDDEN` | `U007` | 차단된 사용자입니다. |
| `NICKNAME_ALREADY_EXISTS` | `CONFLICT` | `U008` | 이미 사용 중인 닉네임입니다. |
| `REQUIRED_TERMS_NOT_AGREED` | `BAD_REQUEST` | `U010` | 필수 약관에 모두 동의해야 합니다. |
| `INVALID_RESET_TOKEN` | `BAD_REQUEST` | `U011` | 유효하지 않거나 만료된 토큰입니다. |
| `USER_SUSPENDED` | `FORBIDDEN` | `U012` | 정지된 계정입니다. |
| `TERMS_NOT_CONFIGURED` | `SERVICE_UNAVAILABLE` | `U013` | 활성 약관 정보가 없습니다. 관리자에게 문의해 주세요. |
| `UNSUPPORTED_OAUTH_PROVIDER` | `BAD_REQUEST` | `U014` | 지원하지 않는 소셜 로그인 제공자입니다. |
| `OAUTH_AUTHORIZATION_FAILED` | `UNAUTHORIZED` | `U015` | 소셜 로그인 인증에 실패했습니다. |
| `INVALID_OAUTH_SIGNUP_TOKEN` | `BAD_REQUEST` | `U016` | 소셜 회원가입 토큰이 유효하지 않거나 만료되었습니다. |
| `OAUTH_EMAIL_ALREADY_EXISTS` | `CONFLICT` | `U017` | 해당 이메일로 가입된 계정이 이미 있습니다. |
| `INVALID_OAUTH_STATE` | `UNAUTHORIZED` | `U018` | 소셜 로그인 요청 상태가 유효하지 않거나 만료되었습니다. |
| `INVALID_OAUTH_RESULT_CODE` | `BAD_REQUEST` | `U019` | 소셜 로그인 결과 코드가 유효하지 않거나 만료되었습니다. |
| `VIDEO_NOT_FOUND` | `NOT_FOUND` | `V001` | 영상을 찾을 수 없습니다. |
| `VIDEO_TOO_LARGE` | `CONTENT_TOO_LARGE` | `V002` | 영상 용량이 너무 큽니다. (최대 200MB) |
| `VIDEO_TOO_LONG` | `BAD_REQUEST` | `V003` | 영상 길이가 너무 깁니다. (최대 60초) |
| `UNSUPPORTED_VIDEO_FORMAT` | `BAD_REQUEST` | `V004` | 지원하지 않는 영상 포맷입니다. |
| `ANALYSIS_FAILED` | `INTERNAL_SERVER_ERROR` | `V005` | 영상 분석에 실패했습니다. |
| `VIDEO_NOT_ACCESSIBLE` | `FORBIDDEN` | `V006` | 비공개 영상에 접근할 수 없습니다. |
| `GYM_NOT_FOUND` | `NOT_FOUND` | `G001` | 암장을 찾을 수 없습니다. |
| `GYM_OUT_OF_RANGE` | `FORBIDDEN` | `G002` | 암장 반경 내에 있지 않습니다. |
| `ALREADY_REVIEWED` | `CONFLICT` | `G003` | 이미 리뷰를 작성한 암장입니다. |
| `REVIEW_NOT_FOUND` | `NOT_FOUND` | `G004` | 리뷰를 찾을 수 없습니다. |
| `INVALID_GYM_GRADE` | `BAD_REQUEST` | `G005` | 암장 난이도가 올바르지 않습니다. |
| `NOTIFICATION_NOT_FOUND` | `NOT_FOUND` | `N001` | 알림을 찾을 수 없습니다. |
| `SELF_REPORT_NOT_ALLOWED` | `BAD_REQUEST` | `R001` | 자기 자신의 콘텐츠는 신고할 수 없습니다. |
| `ALREADY_REPORTED` | `CONFLICT` | `R002` | 이미 신고한 대상입니다. |
| `CLIMBING_LOG_NOT_FOUND` | `NOT_FOUND` | `T001` | 클라이밍 기록을 찾을 수 없습니다. |
| `MONTHLY_REPORT_NOT_FOUND` | `NOT_FOUND` | `T002` | 월간 리포트를 찾을 수 없습니다. |
| `INVALID_MONTH` | `BAD_REQUEST` | `T003` | 월 형식이 올바르지 않습니다. |
| `MONTHLY_REPORT_GENERATION_FAILED` | `INTERNAL_SERVER_ERROR` | `T004` | 월간 리포트 생성에 실패했습니다. |
| `GCS_UPLOAD_FAILED` | `INTERNAL_SERVER_ERROR` | `S001` | 영상 업로드에 실패했습니다. |
| `AI_SERVER_UNAVAILABLE` | `SERVICE_UNAVAILABLE` | `S002` | AI 분석 서버에 연결할 수 없습니다. |

---

## 6. 동기화 메모

- 소스에 없는 `한줄게시판`, `gym photos`, `similar videos`, `stats/me/efficiency`, 구형 `notifications/device-token` 명세를 제거했다.
- OAuth redirect flow, admin API, climbing log CRUD, gym review CRUD, gym grade, nearby gym, SSE, video-based gym recommendation, terms agreement status, notification unread/delete, monthly report endpoint, gym ranking endpoint를 추가했다.
- `GymSummaryResponse.distanceKm`와 `MonthlyReportResponse` 내부 record 스키마를 현재 DTO 기준으로 보강했다.
- `{id}`처럼 모호했던 path variable은 현재 컨트롤러 이름(`{userId}`, `{videoId}`, `{gymId}`, `{reviewId}` 등)으로 정리했다.
- `ApiResponse` 필드는 현재 코드의 `isSuccess`를 사용한다. 예전 `success` 필드는 더 이상 문서 기준으로 사용하지 않는다.
- 상세 예시 JSON은 Swagger UI 또는 `/v1/api-docs`가 1차 확인 수단이다. 이 Markdown은 사람이 빠르게 보는 source-synced 계약표다.
