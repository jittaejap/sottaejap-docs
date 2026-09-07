# 소때잡 — API 명세서

**버전:** v2.2 | **기준일:** 2026-09-07 | **Base URL:** `______`

> **v2.2 변경 (계약 변경 — 07 §6 절차):** §2 `GET /retrospects/candidates`의 **`reason`은 Spring 템플릿**(01 E-62) — 회고된 거래 제외 · reasonCode 우선순위 · `limit` 상한 100 명시 / §2 **`POST /retrospects/chat` 본문 신설** (E-63) / §2 `POST /retrospects` 비고에 재계산 범위 · 리프 묶음 · 예산 없을 때 `null` (E-59 · E-61 · E-64) / §1 #9 `skip`은 알림 #20으로 갈음 (E-65) / §3 규칙 파라미터 **잠정값 주입** + `rules.candidate.*` · `rules.cluster.meal-categories` (E-57) / §3 내부 API 요청 본문은 필드 `@JsonProperty` (E-66)
>

> **v2.1 변경 (계약 변경 — 07 §6 절차):** §2 `POST /auth/login`에 **`KAKAO` 본문 확정** — `code` · `redirectUri` 필드 신설 (01 E-55) / §0 오류 코드 `OAUTH_CODE_INVALID` · `OAUTH_PROVIDER_ERROR` 신설 / `GET /users/me`에 **`nickname` 신설**, `email`은 nullable (01 E-56)
>
> **v2.0 변경 (계약 변경 — 07 §6 절차 · E-52):** §0 공유 enum에서 `cardIssuer` **삭제** ·
> §2 `POST /transactions/upload`에서 `cardIssuer` 파라미터 **삭제**. 서식은 파일 머리글로 가릅니다 (04 §4).
>
> **v1.9 변경 (계약 변경 — 07 §6 절차):** §0 `timeSlot` **4종**(`MORNING`·`DAY`·`EVENING`·`NIGHT`) — `AFTERNOON` 폐기 (E-50) /
> §2 `GET /retrospects/candidates`에 **`from`·`to` 쿼리 신설** — 채팅 3일 창과 날짜 지정 회고 (E-48) / §3 규칙 파라미터에 `rules.chat-window-days` 추가
>
> v1.2 변경: 엔드포인트 4건 신설(21·22·23, `DELETE /goals`) / `#9` 경로 오류 정정 / `/users/me` 응답 명세 추가 /
> **판정 2종 + `evaluationStatus` 분리(E-11)** / `boundaries` 미확정 표기 / `burdenRatio` 정의 변경
>
> **v1.3 변경 (레포 우선):** §3 전면 개정 — `/internal/*` 7종 폐기, **AI `POST /chat` 1종 + Spring 내부 AI API 6종**(E-19) /
> `satisfaction` `HIGH/LOW/UNKNOWN`(E-23) · `retrospectStatus` `ACTIVE` · `repeatIntent` boolean(E-24) / `#24 POST /chat/finance` 신설(FR-12, P2)
>
> **v1.5 변경 (서버 스캐폴딩):** §0 오류 코드에 공통·인증 코드 6종 추가 / `POST /auth/login` 본문 명세 신설 (§2) / `/internal/ai/*`는 시크릿 미설정 시 전부 401
>
> **v1.6 변경 (AI 레포 조치 완료):** §3 — AI `/chat` 시크릿 검사(E-37) / `fallback` 필드 · `TaskType` 5종 · `SpringClient` 경로 6종 **레포 반영 완료**(06 R3~R5) / 봉투 해제 규칙(E-39) / 키 미설정 시 `fallback: true`(E-38)
>
> **v1.8 변경 (2026-09-03):** `taskType`에 `FINANCE_QA` 신설(E-47, FR-12) — `task_context.state`는 거의 빈 객체, 출처는 `reply` 문장에 자연스럽게 언급

---

## 0. 공통 규약

| 항목 | 규약 |
|---|---|
| 인증 | `Authorization: Bearer <token>` (데모 단일 계정 시 생략) |
| 형식 | `application/json` (업로드만 `multipart/form-data`) |
| 날짜 | ISO 8601 — `2026-08-25T20:22:00+09:00` |
| 금액 | 원(KRW), 정수 |
| 페이징 | `?page=0&size=20` |

### 응답 형식

```json
{ "success": true, "data": { } }
```
```json
{ "success": false, "error": { "code": "PARSE_FAILED", "message": "..." } }
```

| 코드 | HTTP | 의미 |
|---|---|---|
| `INVALID_FILE_FORMAT` | 400 | 지원하지 않는 CSV 포맷 |
| `PARSE_FAILED` | 422 | 파싱 실패 |
| `NOT_FOUND` | 404 | 리소스 없음 |
| `DUPLICATE_RETROSPECT` | 409 | 해당 거래에 이미 회고가 있음 (v1.2) |
| `ONBOARDING_REQUIRED` | 428 | 온보딩 미완료 상태에서 본 기능 호출 (v1.2) |
| `INVALID_TAG` | 400 | `purpose`/`companion`이 표준 태그 밖 (v1.3 — E-20) |
| `LLM_UNAVAILABLE` | 503 | AI `/chat` 15초 초과·5xx → 클라이언트는 템플릿 모드 전환 (§3 타임아웃) |
| `INVALID_INPUT` | 400 | 본문 JSON 오류 · `@Valid` 실패 · 파라미터 타입 불일치 · 필수 파라미터 누락 (v1.5) |
| `UNAUTHORIZED` | 401 | Bearer 토큰 없음·만료·위조. `/internal/ai/*`의 `X-Internal-Secret` 불일치도 같은 코드 (v1.5) |
| `FORBIDDEN` | 403 | 인증됐지만 권한 없음 (v1.5) |
| `UNSUPPORTED_PROVIDER` | 400 | `POST /auth/login`의 `provider`가 아직 구현되지 않은 값 (v1.5) |
| `DEMO_ACCOUNT_DISABLED` | 403 | `DEMO_ACCOUNT_ENABLED=false`인데 `LOCAL` 로그인 요청 (v1.5) |
| `OAUTH_CODE_INVALID` | 400 | `POST /auth/login`의 카카오 인가 코드를 카카오가 거부 — 만료 · 재사용 · `redirectUri` 불일치 (v2.1 — E-55) |
| `OAUTH_PROVIDER_ERROR` | 502 | 카카오 토큰 교환·프로필 조회 실패 · 타임아웃 → 클라이언트는 다시 시도 안내 (v2.1 — E-55) |
| `NOT_IMPLEMENTED` | 501 | 뼈대만 있는 엔드포인트. 본선 중 임시 코드이며 시연 경로에는 남지 않아야 함 (v1.5) |

### 공통 enum

| enum | 값 |
|---|---|
| `satisfaction` | **`HIGH` / `LOW` / `UNKNOWN`** — 3택, `MEDIUM` 없음 ⚠️ v1.3 변경 (E-23) |
| `timeSlot` | `MORNING` / `DAY` / `EVENING` / `NIGHT` | ⚠️ v1.9 변경 (E-50) — `AFTERNOON` 폐기. 05~11 / 11~17 / 17~22 / 22~05 |
| `quadrant` | `PROTECT` / `KEEP` / `MINOR` / `PRIORITY` — **좌표. 보류 시 `null`** ⚠️ v1.2 변경 |
| `verdict` | **`SUSTAIN`(지켜요) / `ADJUST`(바꿔볼까요)** — 처방. **보류 시 `null`** ⚠️ v1.2 변경 |
| `evaluationStatus` | **`RESOLVED` / `PENDING`** (v1.2 신설) |
| `retrospectStatus` | **`ACTIVE`** / `PAUSED` / `COMPLETED` ⚠️ v1.3 변경 (E-24) |
| `repeatIntent` | `true` / `false` / `null` — boolean nullable (v1.3 — E-24) |
| `taskType` | `REFLECTION` / `ANALYSIS` / `ACTION_PLAN` / `CLUSTER_NAMING` / `ANALYSIS_NARRATE` / `FINANCE_QA` — AI `/chat` 전용 (v1.3, §3 · v1.8 — E-47) |
| `notificationType` | `RETROSPECT_DUE` / `SUGGESTION` |
| `authProvider` | `LOCAL` / `KAKAO` / `NAVER` / `GOOGLE` (v1.2) |

> ⚠️ **v1.1 → v1.2 브레이킹 체인지 (2건)**
> 1. `verdict.KEEP` → `SUSTAIN`, `verdict.CHANGE` → `ADJUST` — `quadrant.KEEP`(Ⅱ사분면)과 문자열이 같아
>    프론트에서 분기 사고가 나던 문제를 제거했습니다.
> 2. `verdict.PENDING` **삭제** → `evaluationStatus: PENDING`으로 분리 (E-11).
>    보류는 판정이 아니라 "아직 판정할 수 없음"이라는 **상태**입니다.
>    보류 묶음은 `quadrant`·`verdict`가 모두 `null`이며, 클라이언트는 **회색 반투명 점**으로 그립니다.
> 3. **(v1.3)** `satisfaction` `SATISFIED/UNSATISFIED/UNSURE` → **`HIGH/LOW/UNKNOWN`**, `retrospectStatus.IN_PROGRESS` → **`ACTIVE`**,
>    `repeatIntent` enum → **boolean nullable** — AI 레포 DTO 문자열에 맞췄습니다 (E-23·E-24). 클라이언트·Spring 동시 반영.
>
> **클라이언트 분기 규칙**
> ```
> if (p.evaluationStatus === 'PENDING') → 회색 반투명 점, 배지 없음, 툴팁에 보류 문구
> else → 색상 = p.verdict (SUSTAIN | ADJUST) 2색, 배지 = 지켜요 | 바꿔볼까요
> 정렬  = ADJUST 먼저, 그 안에서 burdenRatio 내림차순 (quadrant는 정렬에만 사용)
> ```

---

## 1. 엔드포인트 목록

| # | Method | Path | 기능 | FR | 담당 |
|---|---|---|---|---|---|
| 1 | POST | `/auth/login` | 로그인 (SNS provider 지원) | FR-01-01 | 고현석 |
| 2 | GET | `/users/me` | 내 정보 · 예산 · 임계값 · **온보딩 플래그** | FR-01-03, FR-09-03 | 고현석 |
| 3 | PUT | `/users/me/settings` | 예산 · 임계값 · D+N 설정 | FR-01-03,04,06 | 고현석 |
| 4 | GET | `/goals` | 목표 목록 · 달성률 | FR-01-05 | 고현석 |
| 5 | POST | `/goals` | 목표 등록 (`PUT /goals/{id}` 수정 포함) | FR-01-02 | 고현석 |
| **5a** | **DELETE** | **`/goals/{id}`** | **목표 삭제 (soft)** | FR-01-02 | 고현석 |
| 6 | POST | `/transactions/upload` | 거래내역 업로드 | FR-02-01 | 고현석 |
| 7 | GET | `/transactions` | 거래 목록 (필터·페이징, 회고 이력 겸용) | FR-02-02, FR-03-07 | 고현석 |
| 8 | GET | `/retrospects/candidates` | 회고 후보 + 선정 근거 | FR-03 | 고현석 |
| **9** | POST | **`/retrospects/candidates/{transactionId}/skip`** | 후보 제외 ⚠️ **경로 정정** — **v2.2: 엔드포인트를 두지 않는다.** 상태를 저장하지 않으므로(E-49) 알림 읽음 #20으로 갈음 (E-65) | FR-03-06 | 고현석 |
| 10 | POST | `/retrospects` | 회고 응답 저장 | FR-04-14 | 고현석 |
| 11 | POST | `/retrospects/chat` | 대화형 회고 턴 — Spring이 `task_context`를 만들어 AI `POST /chat`에 위임 (§3) | FR-04 | 고현석(프록시) · 오진호(AI) |
| 12 | GET | `/behaviors` | 반복 행동 묶음 목록 | FR-05 | 정민규 |
| 13 | GET | `/behaviors/{id}` | 묶음 상세 | FR-07-05 | 정민규 |
| 14 | GET | `/satisfaction-map` | 만족도 지도 데이터 | FR-07-01 | 정민규 |
| 15 | GET | `/suggestions` | 행동 조정안 | FR-08-01 | 오진호 |
| 16 | POST | `/suggestions/{id}/adopt` | 조정 횟수 선택 · 채택 (`/reject` 포함) | FR-08-02~05 | 오진호 |
| 17 | GET | `/reports/monthly` | 전월 대비 감소액 · 보조 지표 | FR-08-06,07 | 오진호 |
| 18 | POST | `/onboarding/start` | 과거 거래 표본 연속 회고 | FR-09-01 | 오진호 |
| 19 | GET | `/notifications` | 인앱 알림 목록 | FR-10-01,02 | 석정한 |
| 20 | POST | `/notifications/{id}/read` | 알림 읽음 처리 | FR-10-02 | 석정한 |
| **21** | **POST** | **`/onboarding/complete`** | **온보딩 완료 플래그 저장 + 초기 지도 생성** | FR-09-02 | 오진호 |
| **22** | **GET** | **`/analysis`** | **소비 분석 (라벨별·카테고리별 요약 + 나만의 특징)** | FR-11 | 정민규·오진호 |
| **23** | **GET** | **`/retrospects/{id}`** | **중단 회고 재개용 상태 조회** (P2) | FR-04-13 | 고현석 |
| **24** | **POST** | **`/chat/finance`** | **금융 지식 Q&A** (P2, v1.3) — AI `POST /chat` + 금융 RAG 위임 | FR-12 | 오진호 |

> 외부 API는 모두 Spring이 제공합니다. 12~14·22는 **Spring 규칙 엔진**이 직접 산출한 값입니다 (v1.3 — E-18). **11·24만 AI `/chat`으로 위임**합니다.

---

## 2. 상세 명세

### `POST /auth/login` (v1.5 — 본문 명세 신설 · v2.1 — `KAKAO` 본문 확정, E-55)

**Request — 데모 계정**
```json
{ "provider": "LOCAL" }
```

**Request — 카카오**
```json
{ "provider": "KAKAO", "code": "<카카오 인가 코드>", "redirectUri": "http://localhost:5173/auth/callback" }
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `provider` | enum `authProvider` | ✅ | `LOCAL` = 데모 계정 폴백 (E-15). `KAKAO` = 카카오 로그인 (E-55). `NAVER` · `GOOGLE`은 400 `UNSUPPORTED_PROVIDER` |
| `code` | string | `KAKAO`만 ✅ | 카카오가 클라이언트 콜백으로 돌려준 인가 코드. 1회용이며 수 분 안에 만료 |
| `redirectUri` | string | `KAKAO`만 ✅ | 인가 요청에 썼던 클라이언트 콜백 URL. 서버 `KAKAO_REDIRECT_URIS` 목록에 없으면 400 `INVALID_INPUT` |

**흐름 (E-55)** — 클라이언트가 `https://kauth.kakao.com/oauth/authorize?client_id=<REST API 키>&redirect_uri=<redirectUri>&response_type=code&state=<난수>`로 이동 → 카카오가 `redirectUri?code=...&state=...`로 돌려줌 → 클라이언트가 `state`를 대조하고 이 API를 호출 → 서버가 카카오와 토큰을 교환하고 프로필(`id` · `properties.nickname` · `kakao_account.email`)을 조회한 뒤 우리 JWT만 발급합니다. 카카오 첫 로그인이면 `users` 행을 만듭니다 — **별도 회원가입 API는 없습니다** (E-56). 카카오 액세스 토큰은 저장하지 않습니다.

> **9/7 스파이크 실측 (서버 구현 시 지킬 것):** ① 닉네임은 `kakao_account.profile.nickname`과 `properties.nickname` 두 곳에 같은 값이 온다 — 서버는 **`kakao_account.profile.nickname`을 우선**하고 없으면 `properties.nickname`. ② 이메일은 `kakao_account.has_email == true`이고 `email_needs_agreement == false`일 때만 `kakao_account.email`을 읽는다. 그 외는 `null`. ③ 같은 코드를 두 번 교환하면 카카오가 **400 `invalid_grant` (`error_code: KOE320`)** — 토큰 엔드포인트의 400은 전부 `OAUTH_CODE_INVALID`로, 5xx·타임아웃은 `OAUTH_PROVIDER_ERROR`로 매핑한다. ④ 콘솔의 OpenID Connect는 **끈다** (9/7 확인 후 비활성화). 켜져 있으면 scope에 `openid`가 붙고 토큰 응답에 `id_token`이 오는데, 서버는 어느 쪽이든 `id_token`을 읽지 않는다.

**오류** — `OAUTH_CODE_INVALID` 400 (카카오가 코드 거부) · `OAUTH_PROVIDER_ERROR` 502 (카카오 응답 실패·타임아웃) · `UNSUPPORTED_PROVIDER` 400 · `DEMO_ACCOUNT_DISABLED` 403 · `INVALID_INPUT` 400 (`KAKAO`인데 `code`·`redirectUri` 누락 또는 목록 밖)

**Response 200** (두 provider 동일)
```json
{
  "success": true,
  "data": {
    "accessToken": "<JWT>",
    "tokenType": "Bearer",
    "expiresAt": "2026-09-03T18:00:00+09:00"
  }
}
```

> 이후 모든 요청은 `Authorization: Bearer <accessToken>`. refresh 토큰·쿠키는 없습니다. 만료(기본 24시간)되면 다시 로그인합니다.
> 데모 계정은 `V1__init.sql`이 만드는 `demo@sottaejap.kr` 1건입니다.

---

### `GET /users/me` (v1.2 — 응답 명세 신설)

최초 진입 분기(FR-09-03)에 반드시 필요합니다.

```json
{
  "success": true,
  "data": {
    "id": 1,
    "email": "demo@sottaejap.kr",
    "nickname": "데모 사용자",
    "authProvider": "LOCAL",
    "monthlyBudget": 1200000,
    "outlierThreshold": 1.5,
    "retrospectDelayDays": 1,
    "onboardingCompleted": false,
    "analysisYearMonth": "2026-08"
  }
}
```

> 클라이언트는 `onboardingCompleted == false`이면 **2-1 온보딩**, `true`이면 **3-1 홈**으로 진입합니다.
> `email`은 v2.1부터 **nullable**입니다 (E-56) — 카카오 계정이 이메일 동의를 거부하면 `null`. 화면에서 이메일을 필수로 그리지 마십시오.
> `nickname`은 v2.1 신설 (E-56) — 마이페이지 `1. 프로필`의 표시 이름. 카카오 닉네임을 저장하고, 데모 계정은 `데모 사용자`. nullable이며 `null`이면 클라이언트가 `사용자`로 표시합니다.
> ⚠️ **갭 (v1.5):** `analysisYearMonth`는 04 `User` 엔티티에 없고 산출 규칙(최근 거래월? 사용자 설정?)이 미정입니다. 서버는 확정 전까지 `null`을 내려줍니다 — 액션시트에 결정 항목으로 올립니다.

---

### `POST /transactions/upload`

**Request** — `multipart/form-data`

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `file` | File | ✅ | CSV 또는 XLSX |

카드사를 받지 않습니다 (v2.0). 서식은 파일의 머리글로 가릅니다 — 04 §4.

**Response 200**
```json
{
  "success": true,
  "data": {
    "importedCount": 142,
    "skippedCount": 3,
    "periodFrom": "2026-06-01",
    "periodTo": "2026-08-31",
    "skippedRows": [{ "row": 17, "reason": "날짜 형식 오류" }]
  }
}
```

---

### `GET /retrospects/candidates`

**Request** — Query

| 파라미터 | 타입 | 기본값 | 설명 |
|---|---|---|---|
| `limit` | int | **1** | 인앱 알림 경로 = 1건. 온보딩만 예외 |
| `from` | date · optional | 없음 | v1.9 신설 (E-48). 조회 시작일 |
| `to` | date · optional | 없음 | v1.9 신설 (E-48). 조회 종료일 |

> **`from`·`to`가 없으면** v1.7과 동일하게 동작합니다 — `limit`만큼 반환 (기본 1건). 기존 클라이언트는 그대로 둡니다.
> **채팅 회고 진입(FR-03-08)** 은 `from = 오늘 − (rules.chat-window-days − 1)`, `to = 오늘`로 호출하고 `limit`을 넉넉히 둡니다.
> **날짜·기간 지정(FR-03-09)** 은 사용자가 말한 범위를 그대로 싣습니다. 3일 창은 적용하지 않습니다.
> 어느 경로든 **오래된 거래를 제외하는 상한은 없습니다.** D+1(FR-03-02)은 하한입니다.
> **v2.2 (E-62):** 이미 회고가 있는 거래는 제외합니다. `limit` 상한은 **100**, 정렬은 최신순. 한 거래에 여러 규칙이 맞으면 `THRESHOLD_EXCEEDED` > `TIMESLOT_OUTLIER` > `REPEATED_LOW_SATISFACTION` 순으로 하나만 씁니다. 선별 수치는 §3 `rules.candidate.*` 잠정값(E-57)이고, 이상치 배수는 `User.outlierThreshold`(없으면 `rules.sensitivity.standard`)입니다.

**Response 200**
```json
{
  "success": true,
  "data": {
    "candidates": [{
      "transactionId": 1043,
      "occurredAt": "2026-08-22T23:10:00+09:00",
      "merchant": "○○배달",
      "amount": 12000,
      "category": "배달",
      "timeSlot": "NIGHT",
      "reasonCode": "TIMESLOT_OUTLIER",
      "reason": "최근 비슷한 심야 배달이 반복됐고, 이전 회고에서 만족도가 낮아 다시 확인해볼 소비로 선정했어요."
    }]
  }
}
```

**후보가 없을 때 (FR-03-04 · 화면 S10)**
```json
{ "success": true, "data": { "candidates": [] } }
```

> `reasonCode` = **규칙 엔진 산출값** (결정론적, Spring 선별 쿼리 — 결정로그 §2 ⓪)
> `reason` = ~~AI가 `reasonCode`를 **재구성한** 문장 (FR-04-11). LLM 장애 시 `reasonCode` 기반 템플릿으로 대체~~ → **v2.2 (E-62): `reasonCode`당 1문장인 Spring 템플릿**(AI `app/ai/fallback.py`와 같은 문장). 후보 N건마다 LLM 왕복을 만들지 않습니다. AI가 재구성한 문장은 `POST /retrospects/chat`의 `INTRO` 응답에서 받습니다 (FR-04-10·11)
> ⚠️ AI가 `reasonCode` 없이 이유를 생성하는 경로는 존재하지 않습니다 (NFR-02)

**`reasonCode` 목록 (확정)**

| 코드 | 의미 |
|---|---|
| `TIMESLOT_OUTLIER` | 해당 시간대 평균 대비 이상치 |
| `THRESHOLD_EXCEEDED` | 사용자 설정 임계값 초과 |
| `REPEATED_LOW_SATISFACTION` | 동일 묶음에서 낮은 만족도 반복 |
| `ONBOARDING_SAMPLE` | 온보딩 표본 추출 |
| `MANUAL_PICK` | 사용자 직접 추가 (FR-03-07) |

---

### `POST /retrospects/candidates/{transactionId}/skip` (v1.2 — 경로 정정)

> v1.1의 `POST /retrospects/{id}/skip`은 **회고가 아직 생성되지 않은 후보**를 회고 리소스로 지칭해 구현 불가였습니다.
> 경로 파라미터는 `transactionId`입니다.

---

### `POST /retrospects`

**Request**
```json
{
  "transactionId": 1043,
  "satisfaction": "LOW",
  "purpose": "충동",
  "companion": "혼자",
  "repeatIntent": false,
  "source": "CANDIDATE"
}
```

**Response 200**
```json
{
  "success": true,
  "data": {
    "behaviorId": 12,
    "behaviorName": "심야 배달",
    "retrospectCount": 4,
    "adjustedSatisfaction": -0.42,
    "monthlyTotalAmount": 96000,
    "avgAmount": 12000,
    "txCount": 8,
    "burdenRatio": 0.08,
    "evaluationStatus": "RESOLVED",
    "quadrant": "PRIORITY",
    "verdict": "ADJUST"
  }
}
```

**409 `DUPLICATE_RETROSPECT`** — 해당 `transactionId`에 이미 회고가 있는 경우 (ERD UNIQUE 제약).

> **9/7 실측 (demo):** 같은 키 회고 1·2건째는 `PENDING` · `quadrant/verdict null` + 상위 묶음 `기타|NIGHT||` 생성, 3건째에 `RESOLVED` · `verdict` 산출, 같은 거래 재저장 409, `purpose: "야식"` 400 `INVALID_TAG`. 이름은 AI 폴백 문장(`○○배달`)이 그대로 `behaviorName`이 됐고, 이후 저장으로 사용자 평균이 바뀌면 이전 묶음의 `adjustedSatisfaction`도 다시 계산됩니다(E-61). **결과에서 빠진 묶음(자식이 롤업을 벗어난 상위 묶음)은 지우지 않고 값만 비웁니다** — 회고 수 0인 묶음은 지도·메모리에서 뺍니다.

> 저장 시 **Spring 규칙 엔진**이 묶음·보정·판정을 재계산합니다 (v1.3 — E-18). HTTP 호출 없음.
> 이름이 없는 새 묶음은 AI `POST /chat`(`task = CLUSTER_NAMING`)으로 `displayName`을 받습니다 (§3).
> `purpose`·`companion`이 표준 태그 7/6종 밖이면 **400 `INVALID_TAG`** (E-20). `null`은 미확정으로 허용합니다.
> **v2.2:** 재계산은 **사용자 전체 묶음**과 `User.avgSatisfaction`입니다 (E-61). 응답과 `transactions.behaviorId`는 항상 **리프 묶음**이고, 리프 회고 수가 `rules.rollup-min-count` 미만이면 상위 묶음(`카테고리|시간대||`)에 `parentId`로 붙습니다 (E-59). `monthlyBudget`이 없으면 `burdenRatio`·`quadrant`는 `null`, `verdict`는 세로축만으로 냅니다 (E-61). `behaviorName`은 커밋 후 `CLUSTER_NAMING`으로 받고, AI가 없으면 템플릿 `"{시간대 라벨} {카테고리}"`입니다 (E-64). 남의 거래·없는 거래는 **404 `NOT_FOUND`**.
> `source`는 `CANDIDATE` / `ONBOARDING` / `MANUAL`. 내부 AI 경로(§3 `save_reflection`)는 `source`를 보내지 않으므로 `CANDIDATE`로 저장합니다 (E-66).

---

### `POST /retrospects/chat` (v2.2 신설 — E-63 · FR-04 P1 자연어 경로)

**상태 없는 프록시입니다.** 클라이언트가 `step`과 지금까지 확인한 값을 들고 다니고, 서버는 거래·`reasonCode`·`step`으로 `task_context`를 만들어 AI `POST /chat`(§3 `REFLECTION`)에 위임합니다. **회고 행을 만들지 않습니다** — 저장은 `POST /retrospects`.

**Request**
```json
{
  "transactionId": 1043,
  "message": "어제 밤 배달, 그냥 배고파서 혼자 시켰어요",
  "step": "SATISFACTION",
  "reflection": { "satisfaction": "UNKNOWN", "purpose": null, "companion": null, "repeatIntent": null },
  "recentMessages": [ { "role": "assistant", "content": "지난 금요일 밤 11시 배달, 12,000원이었어요. 만족하셨나요?" } ]
}
```

| 필드 | 규칙 |
|---|---|
| `transactionId` | 필수. 내 거래가 아니면 **404 `NOT_FOUND`**, 이미 회고가 있으면 **409 `DUPLICATE_RETROSPECT`** |
| `message` | `INTRO`에서는 생략 가능 — 서버가 고정 문구로 대체해 AI에 보냅니다(AI는 빈 메시지를 받지 않음). 그 외 단계는 필수 |
| `step` | `INTRO` / `SATISFACTION` / `PURPOSE` / `COMPANION` / `REPEAT` / `CONFIRM`. 생략 시 `INTRO` |
| `reflection` | 사용자가 **이미 확인한** 값. 표준 태그 밖 문자열은 **400 `INVALID_TAG`** |
| `recentMessages` | 최근 대화. 서버는 **최근 6개**만 AI에 전달합니다 |

**Response 200**
```json
{
  "success": true,
  "data": {
    "reply": "혼자 드신 충동 소비로 보이는데, 맞을까요? 만족도는 어땠는지도 알려주세요.",
    "step": "PURPOSE",
    "reflection": { "satisfaction": "LOW", "purpose": "충동", "companion": "혼자", "repeatIntent": null },
    "needsClarification": true,
    "uncertainFields": ["repeatIntent"],
    "fallback": false
  }
}
```

> `reflection`은 AI가 `tool_results[tool_name = "reflection"].data`로 돌려준 **후보값**입니다. 표준 태그 7/6종 밖 문자열은 서버가 `null`로 바꿉니다 (E-20). **사용자가 확인한 뒤 `POST /retrospects`로 저장**합니다.
> `step`은 서버가 계산한 **다음 단계**입니다 — 응답 `reflection`에서 **아직 미확정인 첫 항목**(`satisfaction`이 `UNKNOWN` → `purpose` null → `companion` null → `repeatIntent` null) 순이고, 전부 확정이면 `CONFIRM`. AI가 폴백이라 아무것도 추출하지 못해도 남은 항목을 계속 묻습니다 (9/7 실측 정정 — `uncertainFields`만 보면 폴백에서 `CONFIRM`으로 건너뛰었습니다).
> `uncertainFields`에는 AI의 `uncertain_fields`에 더해 **서버가 표준 태그 밖이라 버린 항목**도 들어갑니다. AI `data`에 키가 없는 항목은 요청의 확정값을 유지합니다.
> `uncertainFields`는 AI의 `uncertain_fields`를 camelCase로 바꾼 것입니다 (`repeat_intention` → `repeatIntent`).
> `reason_code`는 서버가 ⓪ 규칙(E-62)으로 구합니다. 어떤 규칙에도 맞지 않는 거래(직접 선택)는 `MANUAL_PICK`. `INTRO`의 `reply`가 선정 이유 설명입니다 (FR-04-10·11).
> `task_context.status`는 항상 `ACTIVE`입니다. `PAUSED` 저장·재개(FR-04-12·13, `GET /retrospects/{id}`)는 P2.

**503 `LLM_UNAVAILABLE`** — AI `/chat` 15초 초과·5xx. 클라이언트는 P0 선택지 버튼 모드로 전환합니다 (S11).

> **9/7 실측 (demo · AI 컨테이너 키 없음):** `INTRO` → `reply`에 reasonCode 재구성 문장 + `step: SATISFACTION` + `fallback: true` 200. 폴백 모드는 `tool_results`가 비어 오므로 서버가 `reflection`의 남은 항목으로 `step`을 정합니다.

---

### `GET /satisfaction-map`

**Response 200**
```json
{
  "success": true,
  "data": {
    "analysisYearMonth": "2026-08",
    "axisX": {
      "label": "지출 부담",
      "formula": "MONTHLY_TOTAL_OVER_BUDGET",
      "monthlyBudget": 1200000
    },
    "axisY": { "label": "보정 만족도", "range": [-1, 1] },
    "boundaries": { "x": null, "y": null },
    "points": [{
      "behaviorId": 12,
      "name": "심야 배달",
      "monthlyTotalAmount": 96000,
      "avgAmount": 12000,
      "txCount": 8,
      "burdenRatio": 0.08,
      "adjustedSatisfaction": -0.42,
      "retrospectCount": 4,
      "evaluationStatus": "RESOLVED",
      "quadrant": "PRIORITY",
      "verdict": "ADJUST",
      "prescription": "Hmm! 여기부터 볼까요? 부담은 큰데 만족은 낮았던 소비예요.",
      "cta": null
    }, {
      "behaviorId": 21,
      "name": "주말 브런치",
      "monthlyTotalAmount": 168000,
      "avgAmount": 28000,
      "txCount": 6,
      "burdenRatio": 0.14,
      "adjustedSatisfaction": 0.71,
      "retrospectCount": 5,
      "evaluationStatus": "RESOLVED",
      "quadrant": "PROTECT",
      "verdict": "SUSTAIN",
      "prescription": "Great! 이건 지킬 가치가 있어요. 예산을 미리 확보해둘까요?",
      "cta": { "type": "RESERVE_BUDGET", "label": "예산 확보하기" }
    }, {
      "behaviorId": 33,
      "name": "편의점 간식",
      "monthlyTotalAmount": 24000,
      "avgAmount": 3000,
      "txCount": 8,
      "burdenRatio": 0.02,
      "adjustedSatisfaction": -0.05,
      "retrospectCount": 1,
      "evaluationStatus": "PENDING",
      "quadrant": null,
      "verdict": null,
      "prescription": "아직 판단하기엔 이르네요. 조금 더 지켜볼게요.",
      "cta": null
    }]
  }
}
```

> ⚠️ **`boundaries`는 아직 `null`입니다** — 액션시트 #18, 9/7 데이터 확보 후 주입.
> v1.1 예시의 `{x:0.05, y:0.0}`은 임의값이었으므로 폐기했습니다. `null`이면 클라이언트는 축을 그리지 않습니다.
> `burdenRatio`는 **월 합계 ÷ 월 예산**입니다 (v1.2 — 96,000 ÷ 1,200,000 = 0.08 = 예산의 8%).
> `cta`는 `PROTECT`에만 채워집니다 (E-12). `KEEP`을 포함한 나머지는 `null`입니다.

---

### `GET /analysis` (v1.2 신설 — FR-11)

**Response 200**
```json
{
  "success": true,
  "data": {
    "analysisYearMonth": "2026-08",
    "byVerdict": [
      { "verdict": "SUSTAIN", "clusterCount": 5, "monthlyTotalAmount": 430000, "share": 0.36 },
      { "verdict": "ADJUST",  "clusterCount": 3, "monthlyTotalAmount": 210000, "share": 0.18 }
    ],
    "pending": { "clusterCount": 7, "monthlyTotalAmount": 180000, "share": 0.15 },
    "byCategory": [
      {
        "category": "배달",
        "dominantTimeSlot": "NIGHT",
        "avgAmount": 12000,
        "monthlyTotalAmount": 96000,
        "verdict": "ADJUST"
      }
    ],
    "highlight": "배달은 대부분 심야에 몰려 있고, 예산의 8%를 쓰면서 만족도는 가장 낮았어요."
  }
}
```

> `byVerdict`·`byCategory` = **규칙 엔진 집계** (결정론적).
> `highlight` = AI가 위 수치를 **재구성한** 한 문장 — `POST /chat`(`task = ANALYSIS_NARRATE`, §3). 집계에 없는 값은 서술하지 않습니다 (FR-11-03 · NFR-02).

---

### `POST /onboarding/start`

과거 거래(26.06~07)에서 표본을 추출해 연속 회고를 시작합니다.

**Request**
```json
{ "sampleSize": 20, "periodFrom": "2026-06-01", "periodTo": "2026-07-31" }
```

### `POST /onboarding/complete` (v1.2 신설 — FR-09-02)

5단계 완료 시 호출. `onboardingCompleted = true` 저장 후 초기 만족도 지도를 생성합니다.

**Response 200**
```json
{ "success": true, "data": { "onboardingCompleted": true, "clusterCount": 15 } }
```

---

### `GET /notifications`

**Response 200**
```json
{
  "success": true,
  "data": {
    "unreadCount": 2,
    "notifications": [{
      "id": 3,
      "type": "RETROSPECT_DUE",
      "refId": 1043,
      "message": "어제의 심야 배달, 어땠는지 돌아볼까요?",
      "isRead": false,
      "createdAt": "2026-08-24T09:00:00+09:00"
    }]
  }
}
```

---

## 3. Spring ↔ FastAPI 연동 규격 (v1.3 — 레포 `sottaejap-ai` 기준 · 표기 E-32)

> **9/2 최우선 작업.** 통합 담당(Integration Owner): **고현석**
> v1.2의 `/internal/*` 7종은 **폐기**되었습니다 (E-19). AI 레포가 여는 엔드포인트는 `POST /chat` · `GET /health` 둘뿐입니다.

### 역할 분리 (v1.3 확정 — E-18)

| 담당 | 범위 |
|---|---|
| **Spring** | 인증 / 저장·조회 API 전부 / 파일 파싱 / **규칙 엔진 전부**(⓪②③④⑦⑧ — 후보 선별·묶음·롤업·축소 추정·판정·부담·절감·집계) / 알림 생성 / **AI에 열어주는 내부 조회·저장 API** |
| **FastAPI** | **Single Agent**(`/chat`) / 자연어 의도 파악 / 회고 후보 구조화(①) / Tool Calling / 묶음 명명(⑤) · 회고 대화·설명(⑥) · 소비 분석 문장화(⑨) / 금융 RAG(⑪, P2) / LLM 폴백 |

```
Spring = 데이터 / 규칙 / 계산 / 최종 판정
Python = Agent / 자연어 / Tool Calling / RAG / 설명
```

### 호출 방향 (E-19)

```
클라이언트 → Spring 외부 API (camelCase)
                 │
                 ├─ AiClient ──────────────► FastAPI  POST /chat   (snake_case · X-Internal-Secret)
                 │                                │
                 │                                └─ SpringClient ─► Spring  /internal/ai/*  (X-Internal-Secret)
                 │
                 └─ 규칙 엔진 (Spring 내부 호출 — HTTP 없음)
```

- Spring이 AI를 부르는 경로는 **`POST /chat` 하나**입니다. 작업 종류는 `task_context.task`로 구분합니다.
- AI가 데이터가 필요하면 **Spring 내부 AI API를 다시 부릅니다** (pull). 그래서 Spring → AI 타임아웃은 AI → Spring 왕복을 포함해야 합니다 (아래 타임아웃 표).
- 두 방향 모두 헤더 `X-Internal-Secret: <공유 시크릿>` 으로 인증합니다. 두 레포 `.env`에 같은 값을 둡니다 (07 §3).
- **(v1.5)** Spring의 `AI_SHARED_SECRET`이 비어 있으면 `/internal/ai/*`는 **모든 요청을 401로 거부**합니다. 설정 실수로 열리지 않게 한 것이니, 로컬에서도 값을 채우십시오.
- **(v1.6)** AI도 같습니다. `INTERNAL_SHARED_SECRET`이 비어 있거나 헤더가 다르면 `POST /chat`은 **모든 요청을 401**로 거부합니다 (E-37). `/health`만 열려 있습니다. 401 본문은 `{"detail":{"code":"UNAUTHORIZED","message":"…"}}`입니다. 두 쪽 다 시크릿이 비면 `ai-ping`은 절대 200이 나오지 않습니다.

### `POST /chat` (FastAPI 제공 · Spring이 호출) — 레포 `app/schemas/chat.py`

**Request** — snake_case (레포 `ChatRequest`)
```json
{
  "message": "어제 밤 배달, 그냥 배고파서 혼자 시켰어요",
  "user_id": "1",
  "task_context": {
    "task": "REFLECTION",
    "status": "ACTIVE",
    "state": {
      "transaction": { "id": 1043, "occurred_at": "2026-08-22T23:10:00+09:00", "merchant": "○○배달", "amount": 12000, "category": "배달", "time_slot": "NIGHT" },
      "reason_code": "TIMESLOT_OUTLIER",
      "reflection": { "satisfaction": "UNKNOWN", "purpose": null, "companion": null, "repeat_intention": null },
      "step": "SATISFACTION"
    }
  },
  "recent_messages": [
    { "role": "assistant", "content": "지난 금요일 밤 11시 배달, 12,000원이었어요. 만족하셨나요?" }
  ]
}
```

**Response** — 레포 `ChatResponse` (`fallback` 포함 — v1.6 레포 반영 완료)
```json
{
  "reply": "혼자 드신 충동 소비로 보이는데, 맞을까요? 만족도는 어땠는지도 알려주세요.",
  "tool_results": [
    { "tool_name": "reflection", "success": true,
      "data": { "purpose": "충동", "companion": "혼자", "satisfaction": "UNKNOWN", "repeat_intention": null,
                "needs_clarification": true, "uncertain_fields": ["satisfaction", "repeat_intention"] },
      "message": null }
  ],
  "needs_clarification": true,
  "fallback": false
}
```

| 필드 | 규칙 |
|---|---|
| `task_context.task` | `REFLECTION` / `ANALYSIS` / `ACTION_PLAN` / **`CLUSTER_NAMING`** / **`ANALYSIS_NARRATE`** / **`FINANCE_QA`** — `CLUSTER_NAMING`·`ANALYSIS_NARRATE`는 v1.3에서 문서가 정의, **v1.6 레포 반영 완료** (06 R4). `FINANCE_QA`는 v1.8 신설 (E-47) |
| `task_context.status` | `ACTIVE` / `PAUSED` / `COMPLETED` — Spring이 소유. AI는 바꾸지 않는다 |
| `task_context.state` | 작업별 구조화 상태 (아래 표). **레포는 `dict`로 받으므로 구조는 이 문서가 정본** |
| `recent_messages` | 최소 최근 대화. 전체 이력을 보내지 않는다 (레포 원칙) |
| `tool_results[].data` | 회고 후보(①)의 `purpose`·`companion`은 **표준 태그 또는 `null`** 이어야 한다. 자유 문자열이면 Spring이 버린다 (E-20) |
| `needs_clarification` | `true`면 클라이언트는 `uncertain_fields`만 선택지 버튼으로 되묻는다 (FR-04-08) |
| **`fallback`** | **v1.3 추가.** LLM 8초 초과·오류로 템플릿 응답을 돌려줄 때 `true` (FR-04-15). **`OPENAI_API_KEY`가 비어 있을 때도 `true`** (E-38). 클라이언트는 템플릿 모드 배너를 띄운다 (S11) |

**`task_context.state` — 작업별 구조 (문서 정본)**

| `task` | `state` 필수 키 | AI가 돌려주는 것 (`reply`) |
|---|---|---|
| `REFLECTION` | `transaction` · `reason_code` · `reflection`(현재까지 확정값) · `step`(`INTRO` / `SATISFACTION` / `PURPOSE` / `COMPANION` / `REPEAT` / `CONFIRM`) | 다음 질문 또는 확인 문장. `INTRO`에서는 `reason_code` 재구성 설명 (FR-04-10·11) |
| `ANALYSIS` | `analysis_year_month` (집계는 AI가 `/internal/ai/…/analysis`로 pull) | 사용자 질문에 대한 설명 |
| `ACTION_PLAN` | `suggestion_ids[]` (상세는 pull) | 제안 이유 문장 (FR-08-01) |
| `CLUSTER_NAMING` | `cluster_key` · `sample_merchants[]` · `tx_count` | 묶음 이름 1개, 12자 이내 (⑤ · FR-05-05) |
| `ANALYSIS_NARRATE` | `by_verdict[]` · `by_category[]` (Spring 집계값) | '나만의 특징' 한 문장 (⑨ · FR-11-03). **집계에 없는 수치 서술 금지** |
| `FINANCE_QA` | (거의 없음 — 빈 객체 `{}`) | 근거 기반 답변 문장. 출처는 문장에 자연스럽게 언급, 별도 필드 없음 (⑪ · FR-12 · v1.8 — E-47) |

> `reason_code`·집계값은 항상 Spring이 `state`에 실어 보냅니다. AI가 `reason_code` 없이 이유를 만드는 경로는 없습니다 (NFR-02).

### Spring 내부 AI API (Spring 제공 · FastAPI `SpringClient`가 호출)

레포 `app/clients/spring_client.py`의 메서드 6개와 1:1입니다. **v1.6에서 레포에 연결 완료** (06 R5). Spring 쪽 컨트롤러 6종은 `internalai/InternalAiController` (server `f8e3067`).

| `SpringClient` 메서드 | Method | Path | 응답 (camelCase · JSON object) |
|---|---|---|---|
| `get_transactions(user_id, query)` | GET | `/internal/ai/users/{userId}/transactions?from&to&category&size` | `{ "transactions": [ {id, occurredAt, merchant, amount, category, timeSlot, behaviorId} ] }` |
| `get_reflections(user_id)` | GET | `/internal/ai/users/{userId}/reflections` | `{ "reflections": [ {id, transactionId, satisfaction, purpose, companion, repeatIntent, status} ] }` |
| `save_reflection(user_id, reflection)` | POST | `/internal/ai/users/{userId}/reflections` | 외부 `POST /retrospects`와 같은 검증·응답. **사용자 확인이 끝난 값만** 보낸다 |
| `get_behavior_analysis(user_id)` | GET | `/internal/ai/users/{userId}/analysis` | 외부 `GET /analysis` + `GET /satisfaction-map`의 `points` |
| `get_action_plan(user_id)` | GET | `/internal/ai/users/{userId}/suggestions` | 외부 `GET /suggestions`와 동일 |
| `get_memory(user_id)` | GET | `/internal/ai/users/{userId}/memory` | `{ "clusters": [ {behaviorId, name, clusterKey, retrospectCount, adjustedSatisfaction, verdict} ], "recentReflections": [ … ] }` — 개인 소비 메모리 요약 |

- 응답은 Spring 기본 **camelCase**입니다. AI 쪽에서 키를 변환하지 않습니다.
- 요청 본문(`save_reflection`)은 AI가 snake_case로 보내고, Spring은 DTO 필드의 **`@JsonProperty`**(E-24 · `ai/dto`와 같은 방식)로 받습니다 — v2.2 정정 (E-66). `source`가 없으면 `CANDIDATE`로 저장하고, 응답은 외부 `POST /retrospects`와 같은 객체입니다(`data`가 object여야 `SpringClient`가 봉투를 벗깁니다).
- 외부 API와 같은 `{ success, data }` 봉투를 씁니다. **(v1.6)** `SpringClient._request`가 봉투를 벗겨 `data`만 Tool에 넘기고, `success: false`면 `SpringApiError(code)`를 일으킵니다 (E-39). Tool은 위 표의 `data` 모양을 그대로 받습니다.

### 타임아웃 · 폴백 (NFR-04 · FR-04-15)

| 구간 | 타임아웃 | 재시도 | 실패 시 |
|---|---|---|---|
| AI → OpenAI | **8초** (⚠️ 잠정 — `LLM_TIMEOUT_SECONDS`) | 1회 | AI가 **템플릿 응답** + `fallback: true`, HTTP 200 유지. 템플릿은 레포 `app/ai/fallback.py` (문구 담당 오진호). 키 미설정도 같은 경로 (E-38) |
| AI → Spring 내부 API | 10초 (레포 `SPRING_TIMEOUT_SECONDS`) | 0회 | `tool_results[].success = false`, `reply`는 데이터 없이 진행 가능한 문장 |
| Spring → AI `/chat` | **15초** (⚠️ 잠정 — 위 둘을 포함, `AI_TIMEOUT_MS`) | 0회 | Spring이 **템플릿 응답**을 직접 생성해 200, 또는 `LLM_UNAVAILABLE` 503 → 클라이언트 템플릿 모드 |

> P0 회고 경로(선택지 버튼)는 LLM 없이 완주합니다. 폴백은 P1 자연어 경로와 ⑤⑥⑨ 표현 계층에만 걸립니다.

### Spring 규칙 파라미터 (9/7 주입 — 코드 상수 금지 · v1.3: `application.yml`)

| 키 (`rules.*`) | 액션시트 | 잠정값 |
|---|---|---|
| `rules.shrinkage-k` | #15 | **3** (v2.2 잠정 — E-57) |
| `rules.rollup-min-count` | #16 | **3** (v2.2 잠정) |
| `rules.pending-min-count` | #17 | **3** (v2.2 잠정, **≤ rollup-min-count** — 기동 시 검증) |
| `rules.axis-x-boundary` | #18 | **0.1** (v2.2 잠정 — 월 합계가 예산의 10%) |
| `rules.axis-y-boundary` | #18 | **0** (v2.2 잠정 — 중립) |
| `rules.chat-window-days` | — | **3** (v1.9 확정 — E-48) |
| `rules.sensitivity.{conservative,standard,sensitive}` | #15~18과 함께 | **3.0 / 2.0 / 1.5** (v2.2 잠정 — 이상치 중앙값 배수, E-46 프리셋) |
| `rules.candidate.outlier-baseline-days` | **#20** | **90** (v2.2 잠정 — 같은 카테고리·시간대의 기준선 기간) |
| `rules.candidate.outlier-min-samples` | #20 | **5** (v2.2 잠정 — 미만이면 카테고리 전체로 롤업) |
| `rules.candidate.big-amount-budget-ratio` | #20 | **0.1** (v2.2 잠정 — `THRESHOLD_EXCEEDED` = 월 예산 대비 비율) |
| `rules.candidate.repeated-low-min-count` | #20 | **2** (v2.2 잠정 — 같은 상위 키의 `LOW` 회고 수) |
| `rules.cluster.meal-categories` | — | **`식사,식비,음식점,외식,배달,한식,중식,일식,양식,분식,패스트푸드,카페`** (v2.2 — E-58 · B-10) |

> **v2.2:** 위 잠정값은 `application.yml`의 **기본값**이고 환경변수 `RULES_*`로 덮어씁니다 (07 §7). 근거 있는 확정값이 아니며, 데이터 확보 후 값만 바꿉니다 (06 #15~#18 · #20).

> `TAG_MATCH_MIN_SIMILARITY`·`EMBEDDING_MODEL`은 E-20으로 **삭제**되었습니다. 임베딩 모델은 금융 RAG(P2) 착수 시 AI 레포 `.env`에 추가합니다.

**통합 담당(Integration Owner):** **고현석**
