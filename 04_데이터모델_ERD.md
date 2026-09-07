# 소때잡 — 데이터 모델 · CSV 파싱 명세

**버전:** v2.2 | **기준일:** 2026-09-07 | **담당:** 고현석

> **v2.2 변경 (2026-09-07):** 스키마 변경 없음(V5 없음). §3 산식에 **분모·null 정의**(01 E-61) · **롤업 상위 키 `카테고리|시간대||`**(E-59) · `analysisYearMonth` = **사용자의 최근 거래월**(E-60, 06 R11 종결)을 적었습니다. `BehaviorCluster.clusterKey`의 시간대 자리는 식사 목록(`rules.cluster.meal-categories`)이거나 **`기타`(미분류)일 때** 포함합니다 (E-58).
>
> **v2.1 변경 (2026-09-07):** `User.email` **nullable** · 소셜 계정 식별은 **`(authProvider, providerUserId)` 유일 제약** · **`User.nickname` 신설** (01 E-56). 카카오 이메일은 선택 동의라 비어서 올 수 있고, 표시 이름은 닉네임이 맡습니다. server **`V4` 마이그레이션**이 필요합니다 — `users.email`의 `NOT NULL` 해제 · **`uq_users_email` 유일 제약 삭제** · `users.nickname VARCHAR(100)` 추가(데모 계정은 `데모 사용자`로 채움). `(auth_provider, provider_user_id)` 부분 유일 인덱스는 **V1 `uq_users_provider`에 이미 있어** V4에서 만들지 않습니다 (9/7 확인). 카카오 로그인 흐름은 01 E-55.
>
> **v2.0 변경 (2026-09-07 — E-51~E-54):** §4 **카드사별 컬럼 매핑표 폐기**(E-51). 서식은 카드사가 아니라 머리글로 가릅니다 —
> 실제 파일 3종(카드 이용내역서 2 · 통장 거래내역 1)을 넣어 보니 카드사가 같아도 서식이 다르고, 사용자가 고른
> 카드사는 파일과 어긋날 수 있었습니다. `Transaction.cardIssuer` **삭제**(E-52 · server `V3` 마이그레이션) ·
> 업로드 API에서 `cardIssuer` 파라미터 삭제(05 §2) · RFC 4180 줄 자르기 · 날짜/시각 분리 칸 결합 ·
> 통장 `적요` 카드 결제 화이트리스트(E-53) · 합계·요약 행 조용히 무시(E-54).
>
> **v1.9 변경 (2026-09-03):** `Transaction.timeSlot` **4구간**으로 변경 (01 E-50). `AFTERNOON` 폐기 → `DAY`·`EVENING` 신설.
> 스키마 소유자는 server이므로 **`V2` 마이그레이션이 필요합니다** — `time_slot`의 `CHECK` 제약과 **이미 저장된 `AFTERNOON` 값**을 함께 옮겨야 합니다 (06 R12).
> `Retrospect`에는 `reasonCode`를 두지 않습니다 (01 E-49 계열 판단) — 후보 선별은 조회 시점 계산이라 재현되며, 지금 이력에서 선정 이유를 되짚는 요구사항이 없습니다. 필요해지면 그때 칸을 추가합니다.
>
> **v1.4~v1.6 변경 없음** (2026-09-02 밤 확인). 스키마는 server `V1__init.sql`(`f8e3067`)이 소유합니다. `analysisYearMonth` 산출 규칙(06 R11)은 아직 미결입니다.

> 마이그레이션은 **Flyway**로 관리합니다. 개발 중 엔티티 추가는 마이그레이션 파일로 반영합니다.
> v1.2 변경: `burdenRatio` 산식 변경(월 합계 기준) / **판정 2종 + `evaluationStatus` 분리(E-11)** /
> `monthlyTotalAmount`·`analysisYearMonth` 신설 / `MonthlySnapshot.repeatCount` 추가
>
> **v1.3 변경 (레포 우선):** 규칙 엔진 = **Spring**(E-18) / `satisfaction` enum `HIGH/LOW/UNKNOWN`(E-23) / `repeatIntent` boolean nullable · `status` `ACTIVE`(E-24) /
> `purpose`·`companion` 확정 방식(E-20) / **`FinancialChunk` 신설 (pgvector · P2, E-21)**

---

## 0. 스키마 변경 요약 (v1.2 → v1.3)

| 엔티티 | 변경 | 이유 |
|---|---|---|
| `BehaviorCluster` | **`monthlyTotalAmount` 신설** | 부담 분자를 월 합계로 변경 (결정로그 E-1·B-11) |
| `BehaviorCluster` | `burdenRatio` 산식 = `monthlyTotalAmount ÷ monthlyBudget` | 동상 |
| `BehaviorCluster` | **`analysisYearMonth` 신설** | 어느 달의 합계인지 명시 |
| `BehaviorCluster` | `verdict` enum → **`SUSTAIN`/`ADJUST` 2종, nullable** | `quadrant.KEEP` 충돌 제거(E-2) + 판정/상태 층위 분리(E-11) |
| `BehaviorCluster` | **`evaluationStatus` 신설** (`RESOLVED`/`PENDING`) | 보류는 판정이 아니라 상태 (E-11) |
| `BehaviorCluster` | `quadrant` **nullable** — `PENDING` 값 제거 | `PENDING`은 사분면이 아님 (E-11) |
| `MonthlySnapshot` | **`repeatCount` 신설** | FR-08-07 반복 횟수 변화 |
| `User` | **`authProvider` 신설** | SNS 간편 로그인 (E-9) |
| `Retrospect` | `satisfaction` → **`HIGH`/`LOW`/`UNKNOWN`** (v1.3) | 레포 enum 문자열 (E-23). `MEDIUM` 없음 |
| `Retrospect` | `repeatIntent` → **boolean nullable** (v1.3) | 레포 `repeat_intention: bool \| None` (E-24) |
| `Retrospect` | `status` → **`ACTIVE`/`PAUSED`/`COMPLETED`** (v1.3) | 레포 `TaskStatus` (E-24) |
| `Retrospect` | `purpose`·`companion` = **사용자 확인값** (v1.3) | AI 후보 → 사용자 확인 → 저장. 자유 문자열 거부 (E-20) |
| **`FinancialChunk`** | **신설 (P2, v1.3)** | 금융 RAG 저장소 — pgvector (E-21·E-22) |

---

## 1. 엔티티

### User

| 필드 | 타입 | 비고 |
|---|---|---|
| id | PK | |
| email | string | **nullable** (v2.1 — E-56). 카카오가 이메일 동의를 안 주면 `null`. 유일 제약 없음 — 이메일로 계정을 합치지도 막지도 않는다 |
| nickname | string | v2.1 추가 — **표시 이름** (E-56). 카카오 `properties.nickname` 저장, 데모 계정은 V4 시드 `데모 사용자`. nullable — 비면 클라이언트가 `사용자`로 대체. 마이페이지 `1. 프로필`(03 S13)에 표시 |
| authProvider | enum | v1.2 추가 — `LOCAL` / `KAKAO` / `NAVER` / `GOOGLE` (데모는 `LOCAL` 단일) |
| providerUserId | string | v1.2 추가 — SNS 계정 식별자(카카오 `id`), nullable. **`(authProvider, providerUserId)` 유일** — `providerUserId IS NOT NULL`인 행에만 (V1 `uq_users_provider` · E-56) |
| monthlyBudget | int | **지출 부담 분모** |
| outlierThreshold | float | 이상치 탐지 임계값 (사용자 설정) |
| avgSatisfaction | float | 전체 평균 — 축소 추정용 캐시 |
| retrospectDelayDays | int | 기본 **1** (D+1 — 결정로그 B-1) |
| onboardingCompleted | boolean | 온보딩 5단계 완료 플래그 (최초 진입 분기 — FR-09-03) |

### Transaction

| 필드 | 타입 | 비고 |
|---|---|---|
| id | PK | |
| userId | FK → User | |
| occurredAt | timestamp | |
| merchant | string | 원본 |
| merchantNormalized | string | 정규화 (고유명사 유지 / 편의점 지점 표기 제거) |
| amount | int | 원(KRW) |
| category | string | 내부 통합 카테고리 |
| sourceCategory | string | 카드사 원본 카테고리 |
| timeSlot | enum | `MORNING` 05~11(6h) · `DAY` 11~17(6h) · `EVENING` 17~22(5h) · `NIGHT` 22~05(7h) — v1.9 (E-50). `AFTERNOON` 폐기 |
| behaviorId | FK → BehaviorCluster | nullable — 회고 전 미확정 |
| importHash | string | 중복 업로드 방지 |

### Retrospect

| 필드 | 타입 | 비고 |
|---|---|---|
| id | PK | |
| transactionId | FK → Transaction | **UNIQUE** (1:1 보장 — 중복 저장 시 409) |
| satisfaction | enum | `HIGH` +1 / `LOW` −1 / `UNKNOWN` 제외 — 3택, 중간값 없음 (v1.3 — E-23) |
| purpose | string | 표준 태그 7종 중 하나 — **사용자가 확인한 값만**. 미확정 시 `null`. 자유 문자열은 400 거부 (v1.3 — E-20) |
| purposeRaw | string | 사용자 원문 |
| companion | string | 표준 태그 6종 중 하나 — **사용자가 확인한 값만**. 미확정 시 `null`. 자유 문자열은 400 거부 (v1.3 — E-20) |
| companionRaw | string | 사용자 원문 |
| repeatIntent | boolean | **nullable** — `true` / `false` / `null`(미확정) (v1.3 — E-24) |
| status | enum | `ACTIVE` / `PAUSED` / `COMPLETED` (FR-04-12~14 · v1.3 — E-24) |
| source | enum | v1.2 추가 — `CANDIDATE`(선별) / `ONBOARDING`(표본) / `MANUAL`(직접 추가, FR-03-07) |
| createdAt | timestamp | |

### BehaviorCluster

| 필드 | 타입 | 비고 |
|---|---|---|
| id | PK | |
| userId | FK → User | |
| clusterKey | string | `카테고리\|시간대\|목적\|동행인` — **시간대는 식사 카테고리에만 포함**(B-10), 그 외는 공백. **v2.2:** 식사 목록은 `rules.cluster.meal-categories`, `기타`(미분류)도 시간대 포함 (E-58) |
| displayName | string | **AI 생성** 이름 |
| parentId | FK → self | 롤업 상위 키 — **v2.2:** 상위 키는 `카테고리\|시간대\|\|`. 리프 회고 수 < `rules.rollup-min-count`이면 연결, 저장마다 재계산 (E-59) |
| retrospectCount | int | |
| rawAverage | float | 행동 평균 (−1 ~ +1) |
| adjustedSatisfaction | float | 축소 추정 결과 (−1 ~ +1) |
| avgAmount | int | **평균 거래금액 — 절감액 계산 전용** (FR-08-03) |
| **monthlyTotalAmount** | int | **v1.2 신설 — 분석 기준월 1개월 합계 금액** |
| **analysisYearMonth** | string | **v1.2 신설 — `2026-08`. 어느 달 합계인지 명시.** **v2.2:** 사용자의 **최근 거래월**(KST), 모든 묶음 동일 (E-60) |
| txCount | int | v1.2 추가 — 기준월 거래 건수 (`monthlyTotalAmount ÷ avgAmount` 검산용) |
| burdenRatio | float | **`monthlyTotalAmount ÷ User.monthlyBudget`** (v1.2 변경) |
| **evaluationStatus** | enum | **v1.2 신설 — `RESOLVED` / `PENDING`.** 회고 건수 < 보류 임계값이면 `PENDING` |
| quadrant | enum **nullable** | `PROTECT` / `KEEP` / `MINOR` / `PRIORITY` — **좌표. `PENDING`일 때 `null`** (v1.2 변경) |
| verdict | enum **nullable** | **`SUSTAIN`(지켜요) / `ADJUST`(바꿔볼까요)** — 처방. `PENDING`일 때 `null` (v1.2 변경) |

> `clusterKey` 조합·`parentId` 롤업·`burdenRatio`·`quadrant`·`verdict`는
> **규칙 엔진(Spring·정민규)** 이 결정론적으로 산출합니다 (v1.3 — E-18). `displayName`만 AI가 생성합니다 (`/chat` `CLUSTER_NAMING`).
>
> ⚠️ **`avgAmount`와 `monthlyTotalAmount`의 용도를 혼동하지 마십시오.**
> 지도 가로축 = `monthlyTotalAmount` / 절감액 = `avgAmount × adjustCount`.
>
> ⚠️ **`quadrant`(4, 좌표)와 `verdict`(2, 처방)는 다른 층위입니다** — 결정로그 E-11.
> `quadrant`는 화면에 노출하지 않고 **정렬(우선순위)에만** 씁니다. 점 색상은 `verdict` 2색이며,
> `evaluationStatus = PENDING`은 색상이 아니라 **회색 반투명 상태**로 그립니다.

### Goal

| 필드 | 타입 | 비고 |
|---|---|---|
| id | PK | |
| userId | FK → User | |
| name | string | 비상금 / 독립 / 여행 |
| targetAmount | int | |
| currentAmount | int | |
| deletedAt | timestamp | v1.2 추가 — soft delete (FR-01-02 삭제) |

### Suggestion

| 필드 | 타입 | 비고 |
|---|---|---|
| id | PK | |
| behaviorId | FK → BehaviorCluster | |
| adjustCount | int | 사용자 선택 **조정 횟수** |
| expectedSaving | int | `avgAmount × adjustCount` |
| goalId | FK → Goal | 배분 대상 |
| status | enum | `PROPOSED` / `ADOPTED` / `REJECTED` |
| createdAt | timestamp | |

### MonthlySnapshot

| 필드 | 타입 | 비고 |
|---|---|---|
| id | PK | |
| userId | FK → User | |
| yearMonth | string | `2026-08` |
| totalSpending | int | |
| unsatisfiedCount | int | 아쉬운 소비 건수 |
| **repeatCount** | int | **v1.2 신설 — 조정 대상 행동의 반복 횟수** (FR-08-07) |
| savedAmount | int | 전월 대비 감소액 |

### Notification

| 필드 | 타입 | 비고 |
|---|---|---|
| id | PK | |
| userId | FK → User | |
| type | enum | `RETROSPECT_DUE`(D+1 회고 요청) / `SUGGESTION`(제안 발생) |
| refId | int | 대상 리소스 id (candidate `transactionId` / `suggestionId`) |
| message | string | |
| isRead | boolean | 기본 false |
| createdAt | timestamp | |

### FinancialChunk (v1.3 신설 · P2 — E-21·E-22)

| 필드 | 타입 | 비고 |
|---|---|---|
| id | PK | |
| chunkId | string | 레포 `FinancialChunk.chunk_id` |
| content | text | 청크 본문 |
| source | string | 출처 (기관 · 문서명 · URL) |
| metadata | jsonb | 기준 시점 등 — 레포 `metadata` |
| embedding | vector(N) | pgvector — 차원 N은 임베딩 모델 확정 시 (P2) |

> 사용자와 무관한 공개 문서 저장소입니다. `userId`가 없습니다.
> 마이그레이션은 **P2 착수 시** `V3__financial_chunks.sql`로 추가하며, 그때 `CREATE EXTENSION IF NOT EXISTS vector`를 함께 실행합니다.

---

## 2. 관계

```
User 1 ── N Transaction
User 1 ── N BehaviorCluster
User 1 ── N Goal
User 1 ── N MonthlySnapshot
User 1 ── N Notification

Transaction 1 ── 0..1 Retrospect      (transactionId UNIQUE)
Transaction N ── 1 BehaviorCluster

BehaviorCluster 1 ── N Suggestion
BehaviorCluster N ── 1 BehaviorCluster (parent · 롤업)

Suggestion N ── 1 Goal

FinancialChunk — 독립 (사용자·거래와 관계 없음 · P2)
```

---

## 3. 파생값 산식 (v1.3 — Spring 규칙 엔진 계약)

```
purpose / companion   = 사용자가 확인한 표준 태그 (7종 / 6종) 또는 null           # E-20 · 확인 원칙
                        P0: 선택지 버튼 직접 선택 / P1: AI 후보 제안 → 사용자 확인
                        자유 문자열은 Spring이 거부한다 (400). null이면 되묻기(FR-04-08)

rawAverage            = Σ(HIGH:+1, LOW:−1) ÷ (UNKNOWN 제외 회고 수)             # E-23
                        UNKNOWN 제외 표본이 0이면 null                           # E-61 (v2.2)

adjustedSatisfaction  = (n × rawAverage + k × User.avgSatisfaction)
                        ÷ (n + k)                                    # k = #15 (v2.2 잠정 3 — E-57)
                        n = UNKNOWN 제외 회고 수 (rawAverage와 같은 분모)          # E-61
                        전부 UNKNOWN이면 adjusted = User.avgSatisfaction, 첫 회고의 avgSatisfaction = 0

monthlyTotalAmount    = Σ Transaction.amount
                        WHERE behaviorId = this AND yearMonth = analysisYearMonth

avgAmount             = monthlyTotalAmount ÷ txCount

burdenRatio           = monthlyTotalAmount ÷ User.monthlyBudget      # ← 가로축
                        monthlyBudget이 없거나 0이면 null → quadrant도 null, verdict는 세로축만   # E-61

evaluationStatus      = retrospectCount < PENDING_MIN_COUNT ? PENDING : RESOLVED   # #17 (v2.2 잠정 3) — UNKNOWN 포함 건수

parentId              = retrospectCount < ROLLUP_MIN_COUNT ? 상위 묶음(`카테고리|시간대||`) : null   # #16 (v2.2 잠정 3) · E-59
                        상위 묶음의 집계 = 자식 회고 전체의 합집합. 저장마다 재계산
analysisYearMonth     = 사용자의 최근 거래월 (KST)                              # E-60

quadrant              = evaluationStatus == PENDING ? null
                        : (burdenRatio ≥ Bx, adjustedSatisfaction ≥ By) 매트릭스     # 미결 #18
                          ( ≥Bx, ≥By )=PROTECT  ( <Bx, ≥By )=KEEP
                          ( <Bx, <By )=MINOR    ( ≥Bx, <By )=PRIORITY

verdict               = evaluationStatus == PENDING ? null
                        : adjustedSatisfaction ≥ By ? SUSTAIN : ADJUST
                        # 세로축 부호만으로 결정. 가로축은 정렬(우선순위)에만 관여 — E-11

정렬 우선순위          = ADJUST 먼저, 그 안에서 burdenRatio 내림차순
                        ( = 지도상 오른쪽 아래부터 )

expectedSaving        = avgAmount × Suggestion.adjustCount
```

> ⚠️ `k`(#15) · 롤업 기준(#16) · 보류 임계값(#17) · 축 경계 `Bx`/`By`(#18)는 **Spring `application.yml`의 `rules.*` 설정 파라미터**로 분리합니다 (07 §7). **v2.2:** 잠정값 `3 · 3 · 3 · 0.1 · 0`을 기본값으로 주입했습니다 (E-57). 회고 저장 시 **사용자 전체 묶음**을 재계산합니다 (E-61).
> 9/7 튜닝이 값 주입만으로 끝나야 합니다. `TAG_MATCH_MIN_SIMILARITY`·`EMBEDDING_MODEL`은 E-20으로 삭제되었습니다.

---

## 4. CSV 파싱 명세

**지원 카드사:** 가리지 않습니다 (v2.0)
**날짜 포맷:** `2026.08.25 20:22(:30)` — 초 단위 선택적. 날짜와 시각이 다른 칸에 있어도 됩니다
**필수 컬럼:** 거래일시 / 가맹점명 / 금액 / 카테고리

### 컬럼 매핑은 카드사가 아니라 머리글로 한다 (v2.0)

카드사별 매핑표를 두지 않습니다. 같은 은행이라도 카드 이용내역서와 통장 거래내역의 머리글이 전혀
다르고, 사용자가 고른 카드사는 파일과 어긋날 수 있습니다. **파일이 스스로 밝히는 것(머리글)만 믿습니다.**
업로드 API는 카드사를 받지 않고 `Transaction`에도 `cardIssuer`가 없습니다 (05 §2).

**서식 판별** — `출금액` 계열 컬럼이 있으면 통장, 없으면 카드 내역서입니다.

| 찾을 것 | 머리글 키워드 (앞의 것부터 맞춘다) |
|---|---|
| 거래일시 | `거래일시` `이용일시` `승인일시` `사용일시` `매출일시` `거래일자` `이용일자` `승인일자` `거래날짜` `거래일` `이용일` `승인일` `매출일` `일시` `날짜` |
| 시각 (별도 칸) | `이용시간` `거래시간` `승인시간` `결제시간` `매출시간` `시간` |
| 가맹점명 — 카드 | `가맹점명` `이용가맹점` `이용하신곳` `가맹점` `사용처` `상호명` `상호` `내용` `적요` |
| 가맹점명 — 통장 | `보낸분/받는분` `받는분` `보낸분` `의뢰인/수취인` `수취인` `가맹점명` `적요` |
| 금액 — 카드 | `이용금액` `승인금액` `거래금액` `사용금액` `결제금액` `금액` |
| 금액 — 통장 | `출금액` `출금금액` `지급액` `출금` |
| 카테고리 | `가맹점업종` `업종` `카테고리` `분류` |
| 거래 수단 (통장) | `적요` `거래구분` `거래종류` |

**이름이 정확히 같은 것을 먼저** 보고, 없을 때만 포함 관계를 봅니다. `금액`과 `해외이용금액`이 함께 있는
카드 내역서에서 비어 있는 `해외이용금액`을 잡지 않기 위해서입니다.

**통장은 카드 결제만 적재합니다 (화이트리스트).** `적요`에 `체크카드` · `신용카드`가 든 행만 남기고
오픈뱅킹출금 · 전자금융 · FBS출금 · 인터넷입금이체 · `현금IC`(ATM 인출)는 `카드 결제 아님`으로 건너뜁니다.
블랙리스트가 아닌 이유는, 통장의 수단 이름이 은행마다 끝없이 늘어나 새 이름 하나가 새면 송금이 가맹점
소비로 적재되기 때문입니다. 통장의 `적요`는 **수단**이고 상대방은 `보낸분/받는분`에 있습니다 — 적요를
가맹점으로 잡으면 모든 행의 가맹점이 같아집니다.

⚠️ **카테고리 분류 체계가 카드사마다 다릅니다.** 통합 매핑표가 필요합니다 (FR-02-03). 그전까지 `category`는
원본이 있으면 그대로, 없으면 `기타`입니다 — 지어내지 않습니다.

### 내부 통합 카테고리 (초안)

식비 · 배달 · 카페 · 교통 · 쇼핑 · 문화·여가 · 의료 · 주거·통신 · 교육 · 기타

### 공통 처리 규칙

| 항목 | 처리 |
|---|---|
| 인코딩 | EUC-KR / UTF-8 자동 감지. BOM 제거 |
| 줄 자르기 | RFC 4180. 따옴표 안의 쉼표와 **줄바꿈**은 글자다 — 머리글에 줄바꿈을 넣어 보내는 내역서가 있다 |
| 머리글 위치 | 앞 30줄 안에서 거래일시·금액 컬럼이 함께 있는 줄. 제목·조회조건 줄이 앞에 붙는 내보내기가 많다 |
| 금액 형식 | `1,200원` → 정수 변환 |
| 취소·환불 거래 | 제외 — 음수 금액으로 판별한다. `취소상태`·`상태` 컬럼은 보지 않는다 |
| 합계·요약·여백 행 | 조용히 무시. `skippedRows`에 넣지 않는다 — 날짜 칸이 비었거나(`,,,총 177건,...`) 날짜 칸에 글자만 있고 가맹점·금액이 함께 빈 행(`정상승인건수` · `이하 여백`) |
| 카테고리 미기재 | 가맹점명 기반 추정 → 실패 시 `기타`. **추정은 아직 미구현** — 지금은 원본이 없으면 `기타` |
| 시간 정보 없음 | 파싱 실패로 처리 (시간대 분류 불가). 시각이 옆 칸에 있으면 붙여서 먼저 읽어 본다 |
| 중복 판별 | `userId + occurredAt + merchant + amount` 해시 |

**성능 기준 (⚠️ 잠정):** 3개월분(약 1,000건) 10초 이내 — NFR-03

---

## 5. 익명화 데이터 생성 규칙

| 항목 | 처리 |
|---|---|
| 가맹점명 | 실명 → 가명 |
| 금액 | ±10~20% 랜덤 스케일 |
| 거래일시 | **원본 유지** |
| 카테고리 | 원본 유지 |
| 민감 업종 | 제외 또는 일반 카테고리로 치환 |

**스크립트 담당:** **정민규**

> ⚠️ 데모 대표 사례인 **심야 배달의 평균 단가는 12,000원**입니다 (결정로그 D-7).
> 익명화 스케일링 시 이 값이 크게 벗어나지 않도록 고정 시드를 사용하십시오.
