# 소때잡 — API 명세서

**버전:** v2.21 | **기준일:** 2026-09-09 | **Base URL:** `______`

> **v2.21 변경 (2026-09-09 — 거래 목록 `category` 검증 철회):** §2 `GET /transactions`의 `category`가 "§0 enum 밖이면 400"이었으나 **§0에 카테고리 enum이 없고**(04 §4는 초안), 통합 매핑표(06 #11) 전까지 서버는 카드사 원본 문자열을 `category`에 그대로 저장합니다. 존재하지 않는 enum으로 400을 걸면 저장된 값으로 거르는 요청이 막히므로, **저장된 문자열 그대로 일치 필터 · 불일치는 빈 배열 200**으로 고칩니다. 매핑표가 생기면 05 §2 · 04 §4 · 컨트롤러 세 곳을 한 번에 enum으로 바꿉니다. server PR #37 리뷰(결정 1) · 06 R29.

> **v2.20 변경 (2026-09-09 — 거래 목록 `page` 하한):** §2 `GET /transactions`의 `page`에 **음수는 400 `INVALID_INPUT`**을 명시합니다. E-93은 `size`의 하한(1 미만은 400)만 적었고 `page`는 "0부터"만 있어, 음수를 받으면 서버가 500을 내는 자리였습니다. server 이슈 #29.

> **v2.19 변경 (2026-09-09 — 온보딩 표본 추출 규칙 정정):** §2 `POST /onboarding/start`의 **표본 선정 규칙 ③을 교체합니다** (01 v2.18 E-92 ③). 간격을 정수로 먼저 구하면 나머지가 통째로 남아 기간에서 가장 오래된 구간을 아예 보지 않습니다 — `i`번째 표본의 자리를 매번 되짚는 식으로 바꿉니다. ① 대상에 **읽는 범위 상한 2,000건**과 **D+1(E-62 ①)을 보지 않는다**를 함께 적습니다 (E-92 ① 보강 · 명시). server PR #35 리뷰.

> **v2.18 변경 (2026-09-09 — 월간 리포트 확정 규칙 보강):** §2 `GET /reports/monthly` — 목표 실적 배분은 **직전 달(현재 연월 − 1)의 첫 조회에서만**, 확정된 달의 전월 두 값은 굳은 값(`savedAmount` 역산 · 전월 스냅샷 또는 null), 첫 거래월 이전 달은 저장하지 않아 `finalized: false` (01 v2.17 E-94 보강 · server PR #32 리뷰).
>
> **v2.17 변경 (2026-09-09 — 업로드 뒤 재계산의 건너뜀 조건 · 제안 목록):** §2 `POST /transactions/upload`에서 **`importedCount`가 0이면 건너뛴다를 취소합니다** (E-96) — 그 조건이 재계산 실패 뒤의 복구 경로(같은 CSV 재업로드)를 막았습니다. 건너뛰는 조건은 "회고 0건"이고 서버가 판단합니다. **`GET /suggestions`의 `PROPOSED`도 새 기준월 기준으로 다시 계산되고 대상이 없으면 빈다**를 함께 적습니다 (E-97) — 종전에는 `GET /analysis` · `GET /satisfaction-map`만 적어 클라이언트가 버그로 볼 여지가 있었습니다. server PR #31 리뷰.

> **v2.16 변경 (2026-09-09 — 온보딩 표본 추출):** §2 `POST /onboarding/start`에 **Response와 표본 선정 규칙을
> 신설합니다** (E-92). 종전에는 Request만 있어 서버가 무엇을 돌려줘야 하는지 문서에 없었습니다. 응답은
> `GET /retrospects/candidates`와 **같은 `candidates` 배열**이고 `reasonCode`는 전부 `ONBOARDING_SAMPLE`입니다 —
> `ONBOARDING_SAMPLE`을 만드는 경로는 여기 하나뿐입니다. §2 `POST /onboarding/complete`의 `clusterCount` 정의와
> 호출 시점(4단계 — E-45)도 함께 적습니다. server 이슈 #28 · 06 R25.

> **v2.15 변경 (2026-09-09 — 업로드 뒤 묶음 재계산):** §2 `POST /transactions/upload`에 **업로드 커밋 뒤 묶음을 다시 계산한다**를 명시합니다 (E-95). 재계산은 다른 트랜잭션이고 실패해도 200이며, `importedCount`가 0이면 건너뜁니다. 요청 · 응답 본문은 그대로입니다 — 계약 변경이 아니라 종전에 적지 않았던 동작을 못 박는 것입니다. server 이슈 #27 · 06 R24.

> **v2.14 변경 (2026-09-09 — 거래 목록 · 월간 리포트 본문 신설):** §2에 **`GET /transactions`**(#7 · E-93)와 **`GET /reports/monthly`**(#17 · E-94) 상세 블록을 신설합니다. 목록 표에만 있고 본문이 없던 마지막 두 외부 API입니다. 거래 목록은 회고 요약을 실어 회고 이력 탭을 겸하고, 월간 리포트는 지난달을 첫 조회 때 확정하며 `Goal.currentAmount` 실적을 그때 배분합니다. server 이슈 #29 · #30.

> **v2.13 변경 (2026-09-08 — `highlight` AI 미호출 조건):** §2 `GET /analysis`의 `highlight` 폴백에서 **AI를 부르지 않는
> 조건에 `byCategory`가 비어 있는 경우를 더합니다** (E-91). 종전에는 "유효 묶음 0개"만 적어, 회고한 거래가 전부
> 기준월 밖인 사용자에게는 AI를 불렀습니다. 그 `state`는 `by_category []`에 금액이 전부 0이라 근거가 없고,
> E-89가 이 경우로 만든 템플릿 문장도 나가지 못했습니다 (server PR #25 리뷰).

> **v2.12 변경 (2026-09-08 — 재채택 goalId · PUT 전체 교체 · 이유 문장 표):** §2
> `POST /suggestions/{id}/adopt`에서 **`goalId`를 생략하면 기존 목표 연결을 그대로 둡니다.** 종전에는 첫 채택만
> 적어 두어, S5 스텝퍼가 횟수만 바꿔 보낼 때 연결이 끊기는지 유지되는지 알 수 없었습니다. §2
> `PUT /goals/{id}`는 **전체 교체**이고 `currentAmount`만 생략하면 유지입니다 — POST의 "생략하면 0"과 다릅니다.
> §2 `GET /suggestions`에 **이유 문장 3종 표**를 넣습니다. 예시 한 줄만 있으면 클라이언트가 그 문장을
> 하드코딩합니다 (server PR #18 리뷰).

> **v2.11 변경 (2026-09-08 — state 중첩 키 표기):** §3 `task_context.state`의 키는 **중첩 객체까지 snake_case**입니다.
> E-24가 정한 "`/chat` 경계는 snake_case"의 적용 범위를 밝히는 것이라 **계약을 새로 정하지 않습니다.** 종전에는
> `by_verdict[]` · `by_category[]`까지만 규정하고 그 안쪽 키를 적지 않아, Spring이 집계 record를 그대로 실으면
> `monthlyTotalAmount`처럼 camelCase가 섞여 나갔습니다. `ANALYSIS_NARRATE` 항목 키를 이름까지 적습니다 —
> 06 R19(AI의 `analysis_narrate` 숫자 가드)가 이 키로 값을 읽습니다. server `fe2eb86`이 이 모양으로 보냅니다
> (server PR #16 리뷰).

> **v2.10 변경 (2026-09-08 — 질문 길이 상한 · 알림 푸시 경로):** §2 `POST /chat/finance`의 `message`에 **1~500자** 상한을 둡니다.
> 같은 프롬프트에 `recent_messages` 6건이 함께 실리므로(§3), 상한이 없으면 붙여넣기 한 번에 그 예산이 통째로 무너집니다.
> 오류는 기존 **400 `INVALID_INPUT`** 그대로라 새 오류 코드가 없습니다. §2 `GET /notifications`에는
> **이 경로에서는 Web Push를 보내지 않는다**를 명시합니다 — 목록을 여는 사용자는 이미 앱을 보고 있습니다 (server PR #12 리뷰).

> **v2.9 변경 (2026-09-08 — LLM 타임아웃 · E-88):** §3 `fallback` 조건과 AI → OpenAI 타임아웃 잠정값을 **8초에서 6초**로 조정했습니다. 재시도 1회를 포함한 최악 12초를 Spring `/chat` 15초 예산 안에 맞추며, 실제 회고 추출 프롬프트로 재확인해야 합니다.

> **v2.9 변경 (2026-09-08 — 후보 조회 범위 명시):** §2 `GET /retrospects/candidates`는 **최신 100건 안에서** 규칙 ③④⑤를
> 적용합니다. `limit`이 100을 넘으면 오류가 아니라 100으로 자릅니다. 응답 모양이 바뀌지 않으므로 계약 변경은 아닙니다.
>
> **v2.8 변경 (2026-09-08 — 최근 대화 정렬 계약 · E-87):** §2 `POST /retrospects/chat`의 `recentMessages`와
> §3 내부 AI `POST /chat`의 `recent_messages`는 **오름차순(오래된 → 최신)** 입니다. 마지막 원소가 가장 최근 발화입니다.

> **v2.6 변경 (2026-09-07 — 제안 · 목표 슬라이스):** §0 공통 enum에 **`suggestionStatus`** 신설 (E-81) /
> §2 **본문 명세 신설 6종** — `GET /goals`(#4) · `POST /goals`(#5) · `PUT /goals/{id}` · `DELETE /goals/{id}`(#5a) ·
> `GET /suggestions`(#15) · `POST /suggestions/{id}/adopt|reject`(#16) /
> §3 내부 AI `suggestions` 응답 확정 — 외부 `GET /suggestions`의 기본 목록과 같습니다.
> 기존 응답이 바뀌지 않으므로 **계약 변경은 아닙니다**.
>
> **v2.5 변경 (계약 변경 — 07 §6 절차 · 분석 슬라이스):** §0 공통 enum에 **`ctaType`** 신설 (E-76) /
> §2 `GET /satisfaction-map` — **`boundaries`가 더 이상 `null`이 아닙니다**(E-74, "null이면 축을 그리지 않는다" 폐기) ·
> **CTA 3종**으로 확장(E-76, 03 §7 W-6 채택 — 종전 "PROTECT만"을 정정) · 처방 문구 5종 표 신설 /
> §2 `GET /analysis` — `share` 분모가 **월 예산**임을 명시하고 `highlight` 폴백 경로 추가 (E-73 · E-75) /
> §2 **본문 명세 신설 3종** — `GET /behaviors`(#12) · `GET /behaviors/{id}`(#13) · `PUT /users/me/settings`(#3) /
> §3 내부 AI `analysis` 응답 확정 — **`highlight`를 싣지 않습니다** (E-75)
>
> **v2.4 변경 (2026-09-07 — 회고 알림 시각):** §2 `GET /notifications` — 알림 시각이 **후보 거래의 결제 시각에서 계산**되고
> 스케줄이 30분마다 돕니다 (E-71). 경로 · DTO · enum은 그대로이므로 계약 변경이 아닙니다.
>
> **v2.3 변경 (2026-09-07 — 알림 전달 경로 · 대화 이력 · 내부 API 봉투):** §0 공통 enum에 **`reflectionStep`** 추가 (E-69) /
> §1에 **Web Push 3종 신설**(#25·#26·#27 — E-68) / §2에 명세가 없던 **`#20 POST /notifications/{id}/read`** ·
> **`#24 POST /chat/finance`** 본문 명세 신설 / §2 `GET /notifications`에 **목록 조회 시 생성** 동작 명시 (E-68) /
> §3에 **내부 AI API 성공 응답은 항상 `data` object** 규칙 추가 (E-70) / 금융 Q&A 대화 이력은 서버가 소유 (E-67)
>
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
| `timeSlot` | `MORNING` / `DAY` / `EVENING` / `NIGHT` — ⚠️ v1.9 변경 (E-50). `AFTERNOON` 폐기. 05~11 / 11~17 / 17~22 / 22~05 |
| `quadrant` | `PROTECT` / `KEEP` / `MINOR` / `PRIORITY` — **좌표. 보류 시 `null`** ⚠️ v1.2 변경 |
| `verdict` | **`SUSTAIN`(지켜요) / `ADJUST`(바꿔볼까요)** — 처방. **보류 시 `null`** ⚠️ v1.2 변경 |
| `evaluationStatus` | **`RESOLVED` / `PENDING`** (v1.2 신설) |
| `retrospectStatus` | **`ACTIVE`** / `PAUSED` / `COMPLETED` ⚠️ v1.3 변경 (E-24) |
| `repeatIntent` | `true` / `false` / `null` — boolean nullable (v1.3 — E-24) |
| `taskType` | `REFLECTION` / `ANALYSIS` / `ACTION_PLAN` / `CLUSTER_NAMING` / `ANALYSIS_NARRATE` / `FINANCE_QA` — AI `/chat` 전용 (v1.3, §3 · v1.8 — E-47) |
| `reflectionStep` | `INTRO` / `SATISFACTION` / `PURPOSE` / `COMPANION` / `REPEAT` / `CONFIRM` — 회고 대화 단계. **화면이 소유** (v2.3 — E-69) |
| `suggestionStatus` | **`PROPOSED` / `ADOPTED` / `REJECTED`** — `PROPOSED`는 재계산이 만드는 파생 행이고, 나머지 둘은 사용자가 정한 상태다 (v2.6 신설 — E-81) |
| `ctaType` | **`RESERVE_BUDGET`(예산 확보하기) / `ADJUST`(조정하기)** — 묶음 상세 CTA. `KEEP`·보류는 `null` (v2.5 신설 — E-76) |
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
| **25** | **GET** | **`/notifications/push-key`** | **Web Push VAPID 공개키** (v2.3 — E-68) | FR-10 | 석정한 |
| **26** | **POST** | **`/notifications/push-subscriptions`** | **Web Push 구독 등록** (v2.3 — E-68) | FR-10 | 석정한 |
| **27** | **DELETE** | **`/notifications/push-subscriptions`** | **Web Push 구독 해지** (v2.3 — E-68) | FR-10 | 석정한 |

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

업로드를 커밋한 뒤 **묶음을 다시 계산합니다** (E-95). 업로드가 분석 기준월(E-60)을 바꾸는 순간이라, 다시 계산하지
않으면 `GET /analysis`와 `GET /satisfaction-map`이 **새 기준월 이름표에 지난달 금액**을 보여 줍니다.
재계산은 업로드와 **다른 트랜잭션**이고 **실패해도 이 API는 200을 돌려줍니다** — 저장된 거래를 되돌리지 않고 서버
로그에 WARN을 남깁니다. 그때 값은 다음 회고 저장(`POST /retrospects`)이나 예산 변경(`PUT /users/me/settings`)의
재계산이 덮습니다. **`importedCount`가 0이어도 재계산합니다** (E-96) — 재계산이 실패했을 때 사용자가 같은 CSV를 다시
올려 복구할 수 있어야 합니다. 회고가 0건인 사용자에게는 재계산이 아무것도 바꾸지 않고 끝납니다(온보딩 3단계).

**`GET /suggestions`의 `PROPOSED`도 같이 다시 계산되고, 대상이 없으면 빕니다** (E-97). 새 달 CSV를 올린 직후에는
기준월 거래가 있는 묶음이 없어 절감액 기준(`avgAmount`)이 전부 `null`이 되고, 제안 대상에서 모두 빠집니다.
`ADOPTED` · `REJECTED`는 그대로 남습니다. **버그가 아닙니다** — 고치기 전에도 그 달 첫 회고를 저장하는 순간 같은
일이 일어났고, 지도 · 금액 · 제안이 같은 순간에 새 달 상태가 되는 것이 맞습니다.

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

### `GET /transactions` (v2.14 본문 신설 — #7 · FR-02-02 · E-93 · 회고 이력 겸용)

**Request** — Query

| 파라미터 | 타입 | 기본값 | 설명 |
|---|---|---|---|
| `from` | date · optional | 없음 | KST 날짜, 포함 |
| `to` | date · optional | 없음 | KST 날짜, 포함 |
| `category` | string · optional | 없음 | 매핑표(06 #11 · FR-02-03)가 생기기 전까지 **저장된 카테고리 문자열을 그대로 받아 일치하는 것만 거릅니다.** 400을 내지 않고, 일치하는 값이 없으면 빈 배열 200 |
| `hasRetrospect` | boolean · optional | 없음 | `true` = 회고 있는 거래만(회고 이력 탭) · `false` = 없는 거래만 · 생략 = 전체 |
| `page` | int | **0** | §0 페이징 규약. 0부터 — 음수는 400 |
| `size` | int | **20** | 상한 **100** — 넘으면 100으로 자른다(400 아님). 1 미만은 400 |

> 정렬은 내부 AI 조회와 같은 **`occurredAt desc, id desc`** 고정입니다. 정렬 파라미터는 없습니다.
> `from > to`는 400 `INVALID_INPUT`. 남의 거래는 보이지 않고, 조건에 맞는 거래가 없으면 빈 배열로 **200**입니다.
> 내부 AI `GET /internal/ai/users/{userId}/transactions`(§3 `TransactionAiView`)와 **DTO를 공유하지 않습니다** — 그쪽은 계약이 고정돼 있습니다.

**Response 200**
```json
{
  "success": true,
  "data": {
    "transactions": [{
      "id": 1043,
      "occurredAt": "2026-08-22T23:10:00+09:00",
      "merchant": "○○배달",
      "amount": 12000,
      "category": "배달",
      "timeSlot": "NIGHT",
      "retrospectId": 77,
      "satisfaction": "LOW"
    }],
    "page": 0,
    "size": 20,
    "totalElements": 143,
    "totalPages": 8
  }
}
```

| 필드 | 설명 |
|---|---|
| `retrospectId` · `satisfaction` | 회고가 없으면 둘 다 `null`. `satisfaction`은 §0 enum(`HIGH` · `LOW` · `UNKNOWN`) |
| `totalElements` | 필터를 적용한 총건수. `totalPages` = ⌈totalElements ÷ size⌉ |

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
> **v2.9:** 다만 규칙 ③④⑤는 **최신 100건 안에서** 고릅니다 — 쿼리는 날짜 조건만 알고 규칙은 모르므로 넉넉히 100건을
> 읽고 규칙을 적용한 뒤 `limit`으로 자릅니다. 최신 100건에 매칭이 하나도 없으면 후보는 0건입니다. 더 오래된 거래를
> 보려면 `from`·`to`로 범위를 지정합니다. `limit`이 100을 넘으면 400이 아니라 100으로 자릅니다.
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
| `recentMessages` | 최근 대화. **오름차순(오래된 → 최신)** 이며 마지막 원소가 가장 최근 발화입니다. 서버는 **최근 6개**만 AI에 전달합니다 (E-87) |

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
    "boundaries": { "x": 0.1, "y": 0 },
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
      "cta": { "type": "ADJUST", "label": "조정하기" }
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

> **(v2.5 — E-74) `boundaries`는 `rules.axis-x-boundary` · `axis-y-boundary` 잠정값을 그대로 싣습니다** (기본 0.1 / 0).
> "null이면 축을 그리지 않는다"는 v2.4까지의 계약이며 **폐기**했습니다. 값이 잠정이어도 축이 있는 편이 낫다는 판단입니다.
> `burdenRatio`는 **월 합계 ÷ 월 예산**입니다 (v1.2 — 96,000 ÷ 1,200,000 = 0.08 = 예산의 8%).
> **월 예산이 없으면** `burdenRatio` · `quadrant` · `cta`가 모두 `null`이고 `axisX.monthlyBudget`도 `null`입니다 (E-61).
> 예산은 `PUT /users/me/settings`(#3)로 받습니다 — 받기 전에는 지도의 가로축이 서지 않습니다.
> 거래가 하나도 없으면 `analysisYearMonth`가 `null`입니다 (E-60).

**처방 문구 5종 · CTA 3종 (v2.5 — E-76 · FR-07-04 · FR-07-07)**

| 상태 | `prescription` | `cta` |
|---|---|---|
| `PROTECT` | `Great! 이건 지킬 가치가 있어요. 예산을 미리 확보해둘까요?` | `{ RESERVE_BUDGET, "예산 확보하기" }` |
| `KEEP` | `Awesome!! 부담 없이 만족스러운 소비예요. 이대로 두셔도 좋아요.` | `null` |
| `MINOR` | `Umm… 만족은 낮았지만 부담은 크지 않아요. 급하게 바꾸지 않아도 돼요.` | `{ ADJUST, "조정하기" }` |
| `PRIORITY` | `Hmm! 여기부터 볼까요? 부담은 큰데 만족은 낮았던 소비예요.` | `{ ADJUST, "조정하기" }` |
| 보류(`PENDING`) | `아직 판단하기엔 이르네요. 조금 더 지켜볼게요.` | `null` |

> **v2.5 정정:** 종전 "`cta`는 `PROTECT`에만 채워집니다 (E-12)"를 **3종으로 확장**했습니다 — 03 §7 W-6 채택 (E-76).
> **예산이 없어 `quadrant`가 `null`인 RESOLVED 묶음**은 세로축 부호만 남으므로 처방을 판정으로 고릅니다 —
> `SUSTAIN`이면 `KEEP` 문장, `ADJUST`면 `MINOR` 문장이고 `cta`는 `null`입니다.

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

**집계 산식 (v2.5 — E-73)**

| 필드 | 산식 |
|---|---|
| 대상 묶음 | `retrospectCount > 0 && parentId == null` — **유효 묶음** (E-72). 롤업된 리프는 상위 묶음이 대신하므로 뺍니다(이중 집계 방지) |
| `share` | **`monthlyTotalAmount ÷ User.monthlyBudget`** — 분모는 월 예산입니다. 예산이 없으면 `null`이고 반올림하지 않습니다 |
| `byVerdict` | 항상 **`SUSTAIN` · `ADJUST` 2행**입니다. 해당 묶음이 없어도 0으로 채운 행을 냅니다 |
| `pending` | `evaluationStatus = PENDING` 유효 묶음의 합. `verdict`가 `null`이라 `byVerdict`에 넣을 수 없습니다 |
| `byCategory.category` | 묶음 키 `카테고리\|시간대\|목적\|동행인`의 첫 자리 (E-58 · E-59) |
| `byCategory.avgAmount` | `Σ monthlyTotalAmount ÷ Σ txCount` (정수). 건수가 0이면 `null` |
| `byCategory.dominantTimeSlot` | 월 합계가 가장 큰 시간대. 동률이면 이른 시간대. 키에 시간대가 없는 카테고리는 `null` (E-58) |
| `byCategory.verdict` | 월 합계가 가장 큰 **RESOLVED** 묶음의 판정. 전부 보류면 `null`. **보류 금액도 카테고리 합계에는 들어갑니다** |
| 정렬 · 제외 | 합계 내림차순 → `category` 오름차순. 합계 0인 카테고리는 뺍니다 |

> **`highlight` 폴백 (v2.5 — E-75).** 이 GET이 그 자리에서 `ANALYSIS_NARRATE`를 호출하므로 AI 왕복(최대 15초)이
> 응답 시간에 포함됩니다. **다만 어떤 경우에도 200입니다** — AI가 503이거나 빈 문장이거나 `fallback: true`를 주면
> Spring 템플릿 문장으로 갈음하고, **유효 묶음이 0개이거나 `byCategory`가 비어 있으면 AI를 아예 부르지 않습니다**
> (v2.13 — E-91). `byCategory`가 비면 `state`에 재구성할 수치가 하나도 없어, 부를수록 집계 밖 서술만 나옵니다.
> AI에 보내는 `state`에는 `analysis_year_month` · `by_verdict[]` · `by_category[]`만 싣습니다 — **`pending`은 보내지 않습니다**
> (판정이 없는 금액을 문장에 쓸 근거로 주지 않기 위한 가드).

---

### `GET /behaviors` (v2.5 신설 — #12 · FR-05)

지도와 **같은 데이터를 목록으로** 봅니다. 대상과 정렬이 `GET /satisfaction-map`과 같습니다 — 유효 묶음(E-72)을
`ADJUST` → `SUSTAIN` → 보류 순, 각 안에서 `burdenRatio` 내림차순(모르면 맨 뒤), 같으면 묶음 키 순으로 냅니다.

**Response 200**
```json
{
  "success": true,
  "data": {
    "behaviors": [{
      "behaviorId": 12,
      "name": "심야 배달",
      "clusterKey": "배달|NIGHT|충동|혼자",
      "parentId": null,
      "monthlyTotalAmount": 96000,
      "avgAmount": 12000,
      "txCount": 8,
      "retrospectCount": 4,
      "burdenRatio": 0.096,
      "adjustedSatisfaction": -0.42,
      "evaluationStatus": "RESOLVED",
      "quadrant": "MINOR",
      "verdict": "ADJUST"
    }]
  }
}
```

> `name`은 AI가 지은 이름이고(⑤ · E-64), 아직 붙지 않았으면 묶음 키에서 만든 템플릿 이름으로 대체합니다.

---

### `GET /behaviors/{id}` (v2.5 신설 — #13 · FR-07-05)

**Response 200**
```json
{
  "success": true,
  "data": {
    "behavior": { "behaviorId": 12, "name": "심야 배달", "clusterKey": "배달|NIGHT|충동|혼자", "parentId": null,
                  "monthlyTotalAmount": 96000, "avgAmount": 12000, "txCount": 8, "retrospectCount": 4,
                  "burdenRatio": 0.096, "adjustedSatisfaction": -0.42,
                  "evaluationStatus": "RESOLVED", "quadrant": "MINOR", "verdict": "ADJUST" },
    "transactions": [
      { "id": 1043, "occurredAt": "2026-08-24T23:30:00+09:00", "merchant": "배달의민족",
        "amount": 12000, "category": "배달", "timeSlot": "NIGHT", "behaviorId": 12 }
    ]
  }
}
```

> **목록과 달리 롤업된 리프도 열어 줍니다** (E-72). `POST /retrospects` 응답이 **리프** 묶음 id를 주는데
> 그 리프가 롤업됐다면 지도에는 상위 묶음만 있기 때문입니다 — 대신 `parentId`를 함께 주어 지도의 어느 점인지
> 알 수 있게 합니다. 회고 수가 0인 묶음과 남의 묶음은 구별하지 않고 둘 다 **404 `NOT_FOUND`**입니다.
> `transactions`는 **이 묶음과 자식 리프에 배정된 거래의 합집합**입니다 — 상위 묶음에는 직접 구성원만 배정돼
> 있어(E-59) 자식을 함께 읽지 않으면 상세가 비어 보입니다. 시각은 `+09:00` 표기입니다.

---

### `GET /suggestions` (v2.6 본문 신설 — #15 · FR-08-01)

`?status=PROPOSED|ADOPTED|REJECTED` 로 거를 수 있습니다. **생략하면 `PROPOSED`와 `ADOPTED`만** 내려갑니다 —
거절한 제안이 기본 목록에 다시 보이면 거절이 의미가 없습니다. 잘못된 값은 400 `INVALID_INPUT`입니다.

**Response 200**
```json
{
  "success": true,
  "data": {
    "suggestions": [{
      "id": 7,
      "behaviorId": 12,
      "behaviorName": "심야 배달",
      "monthlyTotalAmount": 96000,
      "avgAmount": 12000,
      "txCount": 8,
      "adjustedSatisfaction": -0.42,
      "quadrant": "PRIORITY",
      "adjustCount": 8,
      "expectedSaving": 96000,
      "goalId": null,
      "status": "PROPOSED",
      "reason": "심야 배달은(는) 이번 달 96,000원으로 부담이 컸고 만족도도 낮았어요. 횟수를 줄여볼까요?"
    }]
  }
}
```

> **(E-81) 제안은 재계산 파생 행입니다.** 회고를 저장할 때마다 도는 재계산이 대상 묶음마다 `PROPOSED` 1행을
> 두고 제자리 갱신하며, 대상에서 빠지면 그 `PROPOSED`를 지웁니다. 별도 생성 API가 없는 이유입니다.
> 대상은 **유효 묶음(E-72) ∧ `RESOLVED` ∧ `ADJUST` ∧ `avgAmount != null`** — 상위 묶음 · `MINOR` ·
> 예산이 없어 `quadrant`가 null인 것도 들어가고, `PROTECT`는 들어가지 않습니다.
> 정렬은 `PRIORITY` → `MINOR` → null, 각 안에서 `burdenRatio` 내림차순(모르면 마지막), `id` 오름차순입니다.
> `adjustCount`는 제안 시점에 `txCount`이고 채택할 때 사용자가 고칩니다.
> `reason`은 **Spring 템플릿 3종**입니다 — AI를 부르지 않으므로 `ai` 컨테이너가 내려가도 목록이 그대로 뜹니다 (E-38 · E-84).

**`reason` 문구 3종 (E-84)** — `quadrant`로 고릅니다. 클라이언트는 이 문장을 하드코딩하지 말고 응답을 그대로 씁니다.

| `quadrant` | 문구 |
|---|---|
| `PRIORITY` | `{묶음 이름}은(는) 이번 달 {monthlyTotalAmount}원으로 부담이 컸고 만족도도 낮았어요. 횟수를 줄여볼까요?` |
| `MINOR` | `{묶음 이름}은(는) 부담이 크진 않지만 만족도가 낮았어요. 조금만 줄여볼까요?` |
| `null` (예산 없음) | `{묶음 이름}은(는) 만족도가 낮았어요. 몇 번만 줄여볼까요?` |

> 금액은 천 단위 구분 쉼표를 넣습니다(`96,000`). **좌표를 모르면 부담을 언급하지 않습니다** — 모르는 것을
> "크지 않다"고 말할 수 없습니다 (NFR-02). 위 `GET /suggestions` 예시 응답의 `reason`도 `PRIORITY` 문구입니다.

---

### `POST /suggestions/{id}/adopt` · `POST /suggestions/{id}/reject` (v2.6 본문 신설 — #16 · FR-08-02~05)

**Request** (adopt)
```json
{ "adjustCount": 2, "goalId": 3 }
```

| 필드 | 규칙 |
|---|---|
| `adjustCount` | `1 ≤ n ≤ txCount`. 벗어나면 400 `INVALID_INPUT` |
| `goalId` | 내 목표이고 삭제되지 않았어야 합니다. 아니면 404 `NOT_FOUND`. **생략하면 그대로 둡니다** — 첫 채택이면 목표에 배분하지 않고 채택만 하고, 이미 채택한 제안이면 붙어 있던 목표가 유지됩니다 |

`reject`는 본문이 없습니다.

**Response 200** — 갱신된 제안 한 건 (`GET /suggestions` 항목과 같은 모양)

**상태 전이 (E-82)**

| 지금 | adopt | reject |
|---|---|---|
| `PROPOSED` | → `ADOPTED` | → `REJECTED` |
| `ADOPTED` | → `ADOPTED` (횟수·목표 수정 — FR-08-05) | → `REJECTED` (철회) |
| `REJECTED` | 400 `INVALID_INPUT` | 400 `INVALID_INPUT` |

> `expectedSaving = avgAmount × adjustCount`를 채택 시점에 계산해 **굳힙니다.** 이후 재계산이 평균 단가를
> 바꿔도 채택한 금액은 그대로입니다 — 사용자가 본 숫자와 달라지지 않아야 합니다 (E-81).
> **채택은 `Goal.currentAmount`를 바꾸지 않습니다** (E-82). 채택은 '예상'이고 `currentAmount`는 '실적'입니다.
> 없는 제안과 남의 제안은 구별하지 않고 둘 다 404입니다.

---

### `GET /goals` · `POST /goals` · `PUT /goals/{id}` · `DELETE /goals/{id}` (v2.6 본문 신설 — #4 · #5 · #5a · FR-01-02,05 · FR-08-04)

**`GET /goals` — Response 200**
```json
{
  "success": true,
  "data": {
    "goals": [{
      "id": 3,
      "name": "여행 자금",
      "targetAmount": 1000000,
      "currentAmount": 0,
      "adoptedSaving": 24000,
      "achievementRate": 0.0,
      "projectedRate": 0.024
    }]
  }
}
```

| 필드 | 산식 (E-83) |
|---|---|
| `adoptedSaving` | 이 목표에 붙은 `ADOPTED` 제안의 `expectedSaving` 합 |
| `achievementRate` | `currentAmount ÷ targetAmount` |
| `projectedRate` | `(currentAmount + adoptedSaving) ÷ targetAmount` |

> 비율은 **raw double**입니다 — 반올림은 화면이 합니다. `targetAmount ≤ 0`이면 둘 다 `null`입니다.
> **`achievementRate`는 당분간 0입니다** — 채택이 `currentAmount`를 바꾸지 않기 때문입니다 (E-82).
> 실적 반영 시점은 월간 리포트 판에서 정합니다. 그때까지 화면이 보여줄 수 있는 것은 `projectedRate`입니다.

**`POST /goals` · `PUT /goals/{id}` — Request**
```json
{ "name": "여행 자금", "targetAmount": 1000000, "currentAmount": 0 }
```

| 필드 | 규칙 |
|---|---|
| `name` | 1~50자. **`POST`·`PUT` 모두 필수** |
| `targetAmount` | 1 이상. **`POST`·`PUT` 모두 필수** |
| `currentAmount` | 0 이상. `POST`에서 생략하면 **0**, `PUT`에서 생략하면 **그대로 둡니다** |

어기면 400 `INVALID_INPUT`, 남의 목표·없는 목표는 404 `NOT_FOUND`입니다.

> **`PUT /goals/{id}`는 전체 교체입니다.** `name`과 `targetAmount`를 빼고 보내면 400입니다.
> `currentAmount`만 예외인 이유는 **실적이라 화면이 들고 있지 않기 때문**입니다 — 이름만 고치는 요청이
> 실적을 0으로 되돌리면 안 됩니다.

**`DELETE /goals/{id}`** — soft delete(`deletedAt`)입니다. 목록에서 빠지지만 **`suggestions.goal_id`는 그대로 둡니다** —
지운 목표에 붙어 있던 채택 이력을 잃지 않기 위해서입니다 (E-83). 응답은 `{ "success": true }`입니다.

---

### `GET /reports/monthly` (v2.14 본문 신설 — #17 · FR-08-06 P0 · FR-08-07 · E-94)

**Request** — Query

| 파라미터 | 타입 | 기본값 | 설명 |
|---|---|---|---|
| `yearMonth` | `YYYY-MM` · optional | **분석 기준월** (`GET /users/me`의 `analysisYearMonth`, E-60) | 미래 달(현재 KST 연월보다 뒤)은 400 |

> **확정 규칙 (E-94).** `yearMonth`가 **지난달 이전**이면 첫 조회 때 계산해 `monthly_snapshots`에 저장하고 이후 그 값을 그대로 돌려줍니다 — 확정된 달은 회고를 더 해도 다시 계산하지 않습니다. **이번 달**이면 매번 계산하고 저장하지 않습니다(`finalized: false`). **첫 거래월 이전 달**도 저장하지 않습니다(`finalized: false`, v2.18). 스케줄러는 없습니다.
> **확정된 달의 전월 값 (v2.18).** `previousTotalSpending`은 저장된 `savedAmount`에서 역산한 값이라 `savedAmount = previousTotalSpending − totalSpending`이 응답 안에서 항상 성립합니다. `previousRepeatCount`는 전월 스냅샷이 있으면 그 값, 없으면 `null`입니다 — 전월을 나중에 확정하면 한 번 채워지고 그 뒤로 움직이지 않습니다. 단, `previousTotalSpending`이 `null`(전월 없음으로 확정)이면 전월 스냅샷이 나중에 생겨도 같이 `null`입니다 — 두 값은 같은 전월을 말합니다.
> **목표 실적.** **직전 달(현재 연월 − 1)** 이 확정되는 그 요청에서만(v2.18 — 그 이전 달은 확정만 합니다) `savedAmount > 0`이면 `ADOPTED` 제안이 붙은 목표에 `expectedSaving` 비율로 배분해 `Goal.currentAmount`에 더합니다(04 §3). 그때부터 `GET /goals`의 `achievementRate`가 움직입니다. 채택 자체는 여전히 `currentAmount`를 바꾸지 않습니다(E-82).
> **데이터 없는 달도 200**입니다 — `totalSpending` 0, 전월이 없으면 `savedAmount` · `previousTotalSpending` · `previousRepeatCount`는 `null`.

**Response 200**
```json
{
  "success": true,
  "data": {
    "yearMonth": "2026-08",
    "finalized": true,
    "totalSpending": 412000,
    "previousTotalSpending": 448000,
    "savedAmount": 36000,
    "unsatisfiedCount": 4,
    "repeatCount": 3,
    "previousRepeatCount": 5,
    "goalAllocations": [{ "goalId": 3, "amount": 36000 }]
  }
}
```

| 필드 | 산식 (04 §3 · E-94) |
|---|---|
| `totalSpending` | 그 달 거래 금액 합 |
| `savedAmount` | `previousTotalSpending − totalSpending`. **음수 허용**(더 쓴 달). 예산 · 판정과 무관 |
| `unsatisfiedCount` | 그 달 거래의 회고 중 `LOW` 건수 (FR-08-07) |
| `repeatCount` · `previousRepeatCount` | 유효 묶음 ∧ RESOLVED ∧ ADJUST 묶음의 그 달 거래 건수 합 (FR-08-07 반복 횟수 변화) |
| `goalAllocations` | 이 요청에서 확정 · 배분이 일어났을 때만 채워진다. 그 외에는 빈 배열 |

### `PUT /users/me/settings` (v2.5 본문 신설 — #3 · FR-01-03,04,06)

온보딩 2단계와 마이페이지(S13)가 같은 본문을 씁니다.

**Request**
```json
{ "monthlyBudget": 1000000, "outlierThreshold": 2.0, "retrospectDelayDays": 1 }
```

| 필드 | 규칙 |
|---|---|
| `monthlyBudget` | 1 이상. **지출 부담의 분모**입니다 (FR-06-06) — 이 값이 없으면 지도의 가로축이 서지 않습니다 |
| `outlierThreshold` | 0 초과. 이상치 민감도 배수 (E-46) |
| `retrospectDelayDays` | 0~30. 회고 알림까지 기다리는 날 수 (D+N) |

> 세 값 모두 선택이고 **`null`은 "그대로 두기"** 입니다. 다만 셋이 전부 비면 **400 `INVALID_INPUT`** 입니다 —
> 필드 이름을 잘못 보낸 요청이 200으로 조용히 아무것도 안 하는 것보다 드러나는 편이 낫습니다.
> 범위를 벗어난 값도 400입니다.
>
> **`monthlyBudget`이 바뀌면 서버가 묶음을 다시 계산합니다.** `burdenRatio`·`quadrant`는 재계산이 묶음 행에
> 써 둔 값이라, 예산만 고치면 지도의 가로축과 처방·CTA가 옛 예산 기준으로 남습니다.

**Response 200** — `GET /users/me`와 같은 객체입니다.

---

### `POST /onboarding/start` (v2.16 — Response · 선정 규칙 신설 · v2.19 — 규칙 ③ 교체 · E-92)

과거 거래(26.06~07)에서 표본을 추출해 연속 회고를 시작합니다. 온보딩 3단계에서 업로드
(`POST /transactions/upload`) 직후에 부르고, 응답의 표본이 4단계 연속 회고의 목록이 됩니다.

**Request**
```json
{ "sampleSize": 20, "periodFrom": "2026-06-01", "periodTo": "2026-07-31" }
```

| 필드 | 규칙 |
|---|---|
| `sampleSize` | 1 이상. **필수.** 100을 넘으면 400이 아니라 **100으로 자릅니다**(후보 API와 같은 상한) |
| `periodFrom` · `periodTo` | `YYYY-MM-DD`(KST). **둘 다 필수.** 업로드 응답의 `periodFrom`·`periodTo`를 그대로 싣습니다 |

어기면 400 `INVALID_INPUT`입니다. `periodFrom`이 `periodTo`보다 뒤여도 400입니다.

**Response 200** — `GET /retrospects/candidates`와 **같은 `candidates` 배열**입니다.

```json
{
  "success": true,
  "data": {
    "candidates": [{
      "transactionId": 1043,
      "occurredAt": "2026-07-22T23:10:00+09:00",
      "merchant": "○○배달",
      "amount": 12000,
      "category": "배달",
      "timeSlot": "NIGHT",
      "reasonCode": "ONBOARDING_SAMPLE",
      "reason": "최근 소비 중에서 함께 돌아볼 거래로 골랐어요."
    }]
  }
}
```

**표본 선정 규칙 (E-92)**

| 단계 | 규칙 |
|---|---|
| ① 대상 | `periodFrom` 00:00 ~ `periodTo` 24:00(KST) 사이의 거래 중 **아직 회고하지 않은 것**(E-62 ②와 같은 조건). **D+1(E-62 ①)은 보지 않습니다** — 후보 API와 다른 점입니다. 기간 안에 2,000건이 넘게 있으면 **최신 2,000건 안에서** 고릅니다 |
| ② 정렬 | 최신순(`occurredAt desc, id desc`) |
| ③ 추출 | **`i`번째 표본은 목록의 `i × 전체 건수 ÷ 뽑을 개수`번째**입니다(`i`는 0부터, 소수점은 버립니다). `뽑을 개수 = min(sampleSize, 전체 건수)` — 거래가 표본보다 적으면 있는 만큼만 나갑니다 |
| ④ `reasonCode` | 전부 `ONBOARDING_SAMPLE`. `reason`은 그 코드의 Spring 템플릿 한 문장(E-62) |

> **후보 API(#8)로 갈음하지 않는 이유입니다.** 후보 규칙 ③④⑤ 중 ⑤는 이전 회고가 있어야 걸리는데 온보딩
> 시점의 회고는 0건이고, 규칙은 최신 100건 안에서만 봅니다(v2.9). 두 달치 CSV를 올려도 표본 20건이 모이지
> 않습니다 — 로컬 실측에서 `GET /retrospects/candidates?limit=20`이 0건이었습니다.
> **`ONBOARDING_SAMPLE`을 만드는 경로는 이 엔드포인트 하나뿐입니다** — `GET /retrospects/candidates`는
> 지금처럼 규칙 ③④⑤만 냅니다.
>
> **최신 20건을 자르지 않고 기간에 펼치는 이유입니다.** 앞에서 20건을 자르면 기간의 마지막 한 주만 남아
> 첫 만족도 지도의 묶음이 한 주에 쏠립니다.
>
> **간격을 정수로 먼저 구하지 않는 이유입니다** (v2.19 · E-92 ③). 그러면 나눗셈의 나머지가 통째로 남아
> 기간에서 **가장 오래된 구간을 아예 보지 않습니다** — 59건에서 20건을 고를 때 간격이 2가 되어 39번째에서
> 멈추고 뒤 20건(34%)이 빠집니다. 전체가 `sampleSize`의 두 배 미만이면 간격이 1이 되어 최신순 절삭과
> 같아집니다(25건에서 20건 → 최신 20건 그대로). 자리를 매번 되짚으면 어떤 비율에서도 처음과 끝을 함께
> 덮습니다 — 25건에서 20건이면 마지막 표본이 24번째, 59건에서 20건이면 57번째입니다.
>
> **표본은 저장하지 않습니다** (E-49 · E-65). 난수를 쓰지 않으므로 같은 요청이면 같은 표본이고(E-18),
> 회고를 저장하면 그 거래가 다음 호출에서 빠집니다. 기간 안에 회고할 거래가 없으면 `candidates`는 `[]`입니다.

### `POST /onboarding/complete` (v1.2 신설 — FR-09-02 · v2.16 동작 명시)

**4단계**(표본 회고) 완료 시 호출합니다 — 온보딩은 4단계입니다(E-45). `onboardingCompleted = true`를
저장한 뒤 초기 만족도 지도를 생성합니다. 요청 본문은 없습니다.

**Response 200**
```json
{ "success": true, "data": { "onboardingCompleted": true, "clusterCount": 15 } }
```

| 필드 | 의미 |
|---|---|
| `onboardingCompleted` | 항상 `true`입니다. 다음 로그인부터 `GET /users/me`가 같은 값을 줍니다 |
| `clusterCount` | **지도에 찍히는 점의 수** — `retrospectCount > 0`이고 상위가 없는 유효 묶음(E-72)의 개수입니다 |

> **초기 지도 생성은 사용자 전체 묶음 재계산입니다** (E-61). 회고를 한 건도 저장하지 않고 부르면
> `clusterCount`는 0이고, 그래도 플래그는 저장됩니다 — 온보딩을 끝냈는데 홈에 못 들어가는 상태를 만들지
> 않기 위해서입니다. 두 번 불러도 결과가 같습니다.
>
> 묶음 이름은 여기서 짓지 않습니다. `POST /retrospects`가 저장할 때마다 이름 없는 묶음을 이미 채웁니다(E-64).

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
      "message": "어제 이맘때 ○○배달 12,000원, 어땠는지 돌아볼까요?",
      "isRead": false,
      "createdAt": "2026-08-24T21:00:00+09:00"
    }]
  }
}
```

> **(v2.3 — E-68)** 이 목록을 여는 순간, 그날의 `RETROSPECT_DUE` 알림이 아직 없으면 **1건을 만듭니다.**
> 알림 생성의 본류는 `NOTIFICATION_DAILY_CRON`(기본 30분마다)에 도는 스케줄이고, 이 동작은
> 스케줄이 돌지 않았거나 그 사이 가입한 사용자를 위한 그물입니다. 어느 쪽이든 **하루 1건 가드**를
> 지나야 만들어지므로 겹쳐도 두 번 생기지 않습니다 (FR-03-03).
> `message`는 **Spring 템플릿**입니다 — AI를 부르지 않으므로 `ai` 컨테이너가 내려가도 알림은 뜹니다 (E-38).
>
> **(v2.4 — E-71)** 알림 시각은 사용자마다 다릅니다 — **후보 거래의 결제 시각 − 1시간**을 `07:00~21:00`으로
> 자르고 30분 격자에 내린 값입니다. 새벽 1시 결제는 07:00, 22시 결제는 21:00에 갑니다. 스케줄은 30분마다
> 돌며 지금이 그 시각인 사용자만 고릅니다. **그물 경로는 목표 시각이 지난 뒤에만** 만듭니다 — 오전에
> 목록을 열었다고 저녁 알림을 미리 소진하면 정작 그 시각에는 하루 1건 가드에 걸립니다.
> `message`의 "이맘때"가 사실인 이유도 이것입니다. 후보 거래는 `GET /retrospects/candidates`와
> **같은 규칙 엔진 ⓪**이 고릅니다 (E-62).
>
> **(v2.10)** **이 경로는 Web Push를 보내지 않습니다.** 푸시는 스케줄 경로에서만, 그것도 저장을 커밋한
> 뒤에 나갑니다. 목록을 여는 사용자는 이미 앱을 보고 있고, 응답이 발송(구독당 최대 10초)을 기다릴
> 이유가 없습니다. 클라이언트 service worker는 **목록을 열 때 푸시가 오지 않는 것을 정상으로** 봐야 합니다.

---

### `POST /notifications/{id}/read` (v2.2 — 본문 명세 신설)

**Response 200**
```json
{ "success": true }
```

**404 `NOT_FOUND`** — 없는 알림이거나 **다른 사용자의 알림**일 때. 존재 여부를 알려주지 않습니다.

> 읽음 처리가 곧 후보 제외(`skip`)입니다 — 별도 상태를 저장하지 않습니다 (E-49).
> 사용자는 같은 거래를 채팅·거래내역에서 다시 회고할 수 있어야 하기 때문입니다.

---

### Web Push 3종 (v2.3 신설 — E-68)

브라우저는 **HTTPS에서만** Push를 허용합니다(`localhost`만 예외). iOS Safari는 사용자가 **홈 화면에
추가한 PWA**에서만 동작합니다. 서버에 VAPID 키가 없으면 Web Push만 꺼지고 인앱 알림은 그대로입니다.

#### `GET /notifications/push-key`

```json
{ "success": true, "data": { "enabled": true, "publicKey": "BEl6…(87자 base64url)" } }
```

| 필드 | 규칙 |
|---|---|
| `enabled` | `false`면 서버에 VAPID 키가 없다는 뜻입니다. **클라이언트는 구독을 시도하지 말고 인앱 알림만 씁니다** |
| `publicKey` | `pushManager.subscribe`의 `applicationServerKey`. `enabled: false`면 빈 문자열입니다 |

#### `POST /notifications/push-subscriptions`

**Request** — 브라우저 `PushSubscription.toJSON()`을 **그대로** 보냅니다. `expirationTime`은 쓰지 않습니다.
```json
{
  "endpoint": "https://updates.push.services.mozilla.com/wpush/v2/…",
  "keys": { "p256dh": "BN…", "auth": "k9…" }
}
```

**Response 200** — `{ "success": true }`

> 같은 `endpoint`로 다시 등록하면 **행이 늘지 않고 키만 갱신**됩니다. 브라우저가 키를 새로 만들어
> 재구독하는 일이 흔하기 때문입니다.

#### `DELETE /notifications/push-subscriptions?endpoint=…`

**Response 200** — `{ "success": true }`. 없는 구독을 지워도 성공입니다(여러 번 눌러도 결과가 같아야 합니다).

> 구독이 폐기되면(브라우저 재설치·장기 미사용) 푸시 서비스가 404·410을 돌려주고 **서버가 그 행을
> 스스로 지웁니다.** 클라이언트는 `pushsubscriptionchange`에서 다시 구독해 등록하면 됩니다.

**서버가 보내는 푸시 본문** — service worker가 받는 것은 이 세 개뿐입니다.
```json
{ "title": "소때잡", "body": "9월 6일 ○○배달 12,000원, 어땠는지 돌아볼까요?", "url": "/notifications?ref=1043" }
```

---

### `POST /chat/finance` (v2.2 — 본문 명세 신설 · P2)

**Request**
```json
{ "message": "연금저축 세액공제가 뭐예요?" }
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `message` | string | ✅ | 질문. **1~500자**입니다. 공백만 보내거나 500자를 넘으면 400 `INVALID_INPUT`입니다 (v2.10) |

**Response 200**
```json
{
  "success": true,
  "data": {
    "reply": "연금저축 상품에 가입하고 납입한 금액에 대해 세액을 공제받을 수 있는 제도예요. …",
    "fallback": false
  }
}
```

- **출처는 별도 필드로 두지 않습니다** — `reply` 문장 안에서 느슨하게 언급합니다 (E-47).
- 근거를 찾지 못하면 지어내지 않고 **"확인할 수 없어요"** 라고 답하는 것이 정상입니다 (FR-12-02).
- 이전 질문의 맥락은 서버가 들고 있으므로 "그럼 한도는요?" 같은 되물음이 그대로 이어집니다 (E-67).
- **상한 500자는 프롬프트 예산에서 나온 값입니다** (v2.10). 서버가 최근 대화 6건을 같이 싣기 때문에,
  질문 하나가 길어지면 되물음의 맥락이 먼저 잘립니다.
- 원금 손실 위험 상품 추천과 개인화된 투자 권유는 답하지 않습니다 (FR-12-03 · NFR-05).

**503 `LLM_UNAVAILABLE`** — AI 서버가 응답하지 못할 때.

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
| `task_context.state` | 작업별 구조화 상태 (아래 표). **레포는 `dict`로 받으므로 구조는 이 문서가 정본** / 키는 **중첩 객체까지 snake_case**입니다 (E-24 · v2.11) — Spring은 record·엔티티를 그대로 싣지 않고 키를 옮겨 담습니다 |
| `recent_messages` | 최소 최근 대화. **오름차순(오래된 → 최신)** 이며 마지막 원소가 가장 최근 발화다 (E-87). 전체 이력을 보내지 않는다 (레포 원칙). **최근 6건**만 싣는다. `REFLECTION`은 클라이언트가 `recentMessages`로 보낸 것을 그대로 넘기고(E-63), `FINANCE_QA`는 Spring이 `chat_messages`에서 읽는다 (v2.3 — E-67) |
| `tool_results[].data` | 회고 후보(①)의 `purpose`·`companion`은 **표준 태그 또는 `null`** 이어야 한다. 자유 문자열이면 Spring이 버린다 (E-20) |
| `needs_clarification` | `true`면 클라이언트는 `uncertain_fields`만 선택지 버튼으로 되묻는다 (FR-04-08) |
| **`fallback`** | **v1.3 추가.** LLM 6초 초과·오류로 템플릿 응답을 돌려줄 때 `true` (FR-04-15 · E-88). **`OPENAI_API_KEY`가 비어 있을 때도 `true`** (E-38). 클라이언트는 템플릿 모드 배너를 띄운다 (S11) |

**`task_context.state` — 작업별 구조 (문서 정본)**

| `task` | `state` 필수 키 | AI가 돌려주는 것 (`reply`) |
|---|---|---|
| `REFLECTION` | `transaction` · `reason_code` · `reflection`(현재까지 확정값) · `step`(enum `reflectionStep` — v2.3 · E-69) | 다음 질문 또는 확인 문장. `INTRO`에서는 `reason_code` 재구성 설명 (FR-04-10·11) |
| `ANALYSIS` | `analysis_year_month` (집계는 AI가 `/internal/ai/…/analysis`로 pull) | 사용자 질문에 대한 설명 |
| `ACTION_PLAN` | `suggestion_ids[]` (상세는 pull) | 제안 이유 문장 (FR-08-01) |
| `CLUSTER_NAMING` | `cluster_key` · `sample_merchants[]` · `tx_count` | 묶음 이름 1개, 12자 이내 (⑤ · FR-05-05) |
| `ANALYSIS_NARRATE` | `analysis_year_month` · `by_verdict[]` · `by_category[]` (Spring 집계값). **항목 키 (v2.11)** — `by_verdict[]`는 `verdict` · `cluster_count` · `monthly_total_amount` · `share`, `by_category[]`는 `category` · `dominant_time_slot` · `avg_amount` · `monthly_total_amount` · `verdict`. `pending`은 싣지 않습니다 (E-75) | '나만의 특징' 한 문장 (⑨ · FR-11-03). **집계에 없는 수치 서술 금지** |
| `FINANCE_QA` | (거의 없음 — 빈 객체 `{}`) | 근거 기반 답변 문장. 출처는 문장에 자연스럽게 언급, 별도 필드 없음 (⑪ · FR-12 · v1.8 — E-47) |

> `reason_code`·집계값은 항상 Spring이 `state`에 실어 보냅니다. AI가 `reason_code` 없이 이유를 만드는 경로는 없습니다 (NFR-02).

### Spring 내부 AI API (Spring 제공 · FastAPI `SpringClient`가 호출)

레포 `app/clients/spring_client.py`의 메서드 6개와 1:1입니다. **v1.6에서 레포에 연결 완료** (06 R5). Spring 쪽 컨트롤러 6종은 `internalai/InternalAiController` (server `f8e3067`).

| `SpringClient` 메서드 | Method | Path | 응답 (camelCase · JSON object) |
|---|---|---|---|
| `get_transactions(user_id, query)` | GET | `/internal/ai/users/{userId}/transactions?from&to&category&size` | `{ "transactions": [ {id, occurredAt, merchant, amount, category, timeSlot, behaviorId} ] }` |
| `get_reflections(user_id)` | GET | `/internal/ai/users/{userId}/reflections` | `{ "reflections": [ {id, transactionId, satisfaction, purpose, companion, repeatIntent, status} ] }` |
| `save_reflection(user_id, reflection)` | POST | `/internal/ai/users/{userId}/reflections` | 외부 `POST /retrospects`와 같은 검증·응답. **사용자 확인이 끝난 값만** 보낸다. **(v2.2)** 본문은 `transaction_id` · `satisfaction` · `purpose` · `companion` · `repeat_intention` 5개다 — `transaction_id`가 없으면 어느 거래인지 특정할 수 없다 |
| `get_behavior_analysis(user_id)` | GET | `/internal/ai/users/{userId}/analysis` | 외부 `GET /analysis` + `GET /satisfaction-map`의 `points`. **(v2.5) `highlight`는 싣지 않는다** — 그 문장을 만드는 게 AI의 일이라, 미리 건네면 AI가 자기 출력을 근거로 삼는다 (E-75) |
| `get_action_plan(user_id)` | GET | `/internal/ai/users/{userId}/suggestions` | 외부 `GET /suggestions`의 **기본 목록**(`PROPOSED` + `ADOPTED`)과 같다. **(v2.6)** AI는 `suggestions[].id`를 `state.suggestion_ids`와 대조해 고르므로 `id`는 `suggestions` PK다 (E-81) |
| `get_memory(user_id)` | GET | `/internal/ai/users/{userId}/memory` | `{ "clusters": [ {behaviorId, name, clusterKey, retrospectCount, adjustedSatisfaction, verdict} ], "recentReflections": [ … ] }` — 개인 소비 메모리 요약 |

- 응답은 Spring 기본 **camelCase**입니다. AI 쪽에서 키를 변환하지 않습니다.
- 요청 본문(`save_reflection`)은 AI가 snake_case로 보내고, Spring은 DTO 필드의 **`@JsonProperty`**(E-24 · `ai/dto`와 같은 방식)로 받습니다 — v2.2 정정 (E-66). `source`가 없으면 `CANDIDATE`로 저장하고, 응답은 외부 `POST /retrospects`와 같은 객체입니다(`data`가 object여야 `SpringClient`가 봉투를 벗깁니다).
- 외부 API와 같은 `{ success, data }` 봉투를 씁니다. **(v1.6)** `SpringClient._request`가 봉투를 벗겨 `data`만 Tool에 넘기고, `success: false`면 `SpringApiError(code)`를 일으킵니다 (E-39). Tool은 위 표의 `data` 모양을 그대로 받습니다.
- **(v2.3 — E-70) 성공 응답의 `data`는 항상 JSON object입니다.** 돌려줄 것이 없어도 `null`이 아니라 빈 객체를 싣습니다.
  `SpringClient._request`가 object가 아닌 `data`를 예외로 처리하므로, `data` 없는 200은 **AI에서 500**이 되고 Spring은 그것을 **503 `LLM_UNAVAILABLE`** 로 보여줍니다 —
  LLM은 멀쩡한데 저장만 실패하는, 원인을 찾기 어려운 증상이 됩니다. Spring 쪽은 `ApiResponse<Void>`를 쓰지 않는 것으로 지킵니다 (server `InternalAiContractTest`).

### 타임아웃 · 폴백 (NFR-04 · FR-04-15)

| 구간 | 타임아웃 | 재시도 | 실패 시 |
|---|---|---|---|
| AI → OpenAI | **6초** (⚠️ 잠정 — `LLM_TIMEOUT_SECONDS`, E-88) | 1회 | AI가 **템플릿 응답** + `fallback: true`, HTTP 200 유지. 템플릿은 레포 `app/ai/fallback.py` (문구 담당 오진호). 키 미설정도 같은 경로 (E-38) |
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
