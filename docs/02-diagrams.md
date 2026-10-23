# 02. 다이어그램 구조도

> 배민의 메뉴 구조·기능을 기준으로, **고객 · 라이더 · 계약업체(업주) · 관리자** 4역할을 모두 담은 구조도입니다.
> GitHub에서 Mermaid로 바로 렌더링되며, 같은 내용이 `design/canvas.html`의 「다이어그램」 탭에도 있습니다.
> 단계 표시: **[1]** MVP · **[2]** 2차 · **[3]** 3차

## 목차

1. 전체 시스템 구조 (3 레포 · 4 역할)
2. 고객 앱 메뉴 구조 (IA)
3. 라이더 앱 메뉴 구조
4. 계약업체 대시보드 메뉴 구조
5. 관리자(본사) 대시보드 메뉴 구조
6. 주문 흐름 (시퀀스)
7. 주문 상태 머신
8. 라이더 위치 확인 (위치 기반 시퀀스)
9. 라이더 상태 머신 · 배차
10. 계약업체 입점 · 상품 등록 흐름
11. 결제 흐름 (결제 · 취소 · 환불)
12. 정산 흐름 · 정산 대시보드 데이터
13. ERD (확장)

---

## 1. 전체 시스템 구조

```mermaid
flowchart LR
  subgraph SPEC["dongne-baedal-spec (원본)"]
    T[tokens.json]
    D[docs · 다이어그램]
    C[design/canvas.html]
    S[supabase 스키마]
  end

  subgraph WEB["dongne-baedal-web (Next.js)"]
    W1[고객 웹 · PWA]
    W2["계약업체 대시보드 /partner"]
    W3["관리자 대시보드 /admin"]
  end

  subgraph APP["dongne-baedal-flutter"]
    A1[고객 앱]
    A2[라이더 앱 모드]
  end

  subgraph SB["Supabase"]
    AUTH[Auth]
    DB[(Postgres + PostGIS)]
    RT[Realtime]
    ST[Storage 이미지]
    EF[Edge Functions]
  end

  PG[PG사 결제]
  MAP[지도 · 주소 API]
  PUSH[FCM 푸시]

  SPEC -.->|서브모듈 spec/| WEB
  SPEC -.->|서브모듈 spec/| APP
  W1 & W2 & W3 & A1 & A2 --> AUTH
  W1 & W2 & W3 & A1 & A2 --> DB
  A1 & W1 & W2 <--> RT
  A2 -- 위치 전송 --> RT
  W2 --> ST
  EF <--> PG
  A1 & A2 & W1 --> MAP
  EF --> PUSH
```

| 역할 | 채널 | 주요 기능 |
|---|---|---|
| 고객 | Flutter 앱, 웹/PWA | 탐색 · 주문 · 결제 · 배달 추적 · 리뷰 |
| 라이더 | Flutter 앱(라이더 모드) | 배차 수락 · 픽업 · 배달 · 위치 전송 · 수익 |
| 계약업체 | 웹 `/partner` (태블릿·PC 중심) | 주문 접수 · 상품 등록 · 매출/정산 |
| 관리자 | 웹 `/admin` (PC) | 입점 심사 · 주문/결제 관제 · 라이더 관제 · 정산 승인 |

---

## 2. 고객 앱 메뉴 구조 (IA)

배민의 「하단 탭 + 홈 상단 서비스 탭」 구조를 따릅니다.

```mermaid
flowchart TD
  ROOT((고객 앱)) --> HOME[홈]
  ROOT --> SEARCH[검색]
  ROOT --> FAV[찜]
  ROOT --> ORD[주문내역]
  ROOT --> MY[마이]

  HOME --> ADDRBAR["상단: 배달지 선택 · 알림 · 장바구니"]
  HOME --> SVC["서비스 탭"]
  SVC --> S1["배달 [1]<br/>한집배달 · 알뜰배달"]
  SVC --> S2["가게배달 [1]"]
  SVC --> S3["포장 [1]"]
  SVC --> S4["장보기·쇼핑 [2]"]
  SVC --> S5["선물하기 [3]"]
  HOME --> BAN["프로모션 배너 [1]"]
  HOME --> CATG["음식 카테고리 그리드 [1]"]
  HOME --> REC["추천 섹션: 내 주변 인기 · 최근 주문 가게 [2]"]

  CATG --> LIST["가게 리스트<br/>정렬: 기본·배달빠른·배달팁낮은·별점·거리<br/>필터: 최소주문·쿠폰·배달방식"]
  SEARCH --> LIST
  FAV --> STORE
  LIST --> STORE["가게 상세<br/>메뉴 · 정보 · 리뷰 탭"]
  STORE --> MENU["메뉴 상세 시트<br/>필수/선택 옵션 · 수량"]
  MENU --> CART["장바구니<br/>최소주문 진행바 · 배달방식 선택"]
  CART --> CHK["주문서<br/>배달지 · 요청사항 · 쿠폰 · 결제수단"]
  CHK --> PAY["결제 (PG)"]
  PAY --> TRK["주문 현황<br/>단계 타임라인 · 라이더 지도 · 도착 예정"]
  TRK --> RV["리뷰 작성 [1]"]
  ORD --> TRK
  ORD --> REORD["재주문 [1]"]

  MY --> M1["배달지 관리 [1]"]
  MY --> M2["쿠폰함 [1]"]
  MY --> M3["포인트 [2]"]
  MY --> M4["결제수단 관리 [1]"]
  MY --> M5["리뷰 관리 [1]"]
  MY --> M6["멤버십 [3]"]
  MY --> M7["고객센터 · 공지 · 설정 [1]"]
```

### 고객 기능 목록

| 영역 | 기능 | 단계 |
|---|---|---|
| 인증 | 휴대폰 인증, 카카오·네이버·애플 로그인 | 1 |
| 위치 | 현재 위치로 배달지 설정, 주소 검색, 배달지 여러 개 저장 | 1 |
| 탐색 | 카테고리, 검색(가게·메뉴), 정렬·필터, 배달 가능 반경 내 가게만 노출 | 1 |
| 가게 | 메뉴 카테고리 스티키 탭, 대표 메뉴, 최소주문·배달팁·예상시간, 리뷰·사장님 댓글 | 1 |
| 주문 | 옵션 선택, 장바구니(한 가게 제한), 배달·가게배달·포장 선택, 요청사항 | 1 |
| 결제 | 카드·간편결제, 쿠폰 적용, 결제 취소 | 1 |
| 추적 | 실시간 상태, **라이더 위치 지도**, 도착 예정 시간, 푸시 알림 | 1 |
| 사후 | 리뷰(별점·사진), 재주문, 영수증 | 1 |
| 확장 | 포인트, 장보기, 선물하기, 멤버십(무료배달) | 2·3 |

---

## 3. 라이더 앱 메뉴 구조

```mermaid
flowchart TD
  R((라이더 모드)) --> RH["홈: 운행 시작/종료 토글"]
  R --> RL[배차 목록]
  R --> RD[진행 중 배달]
  R --> RE[수익]
  R --> RM[마이]

  RH --> ON["운행 중: 위치 전송 시작"]
  RL --> OFFER["배차 요청 카드<br/>가게·고객 거리 · 예상 수입 · 수락 제한시간"]
  OFFER -->|수락| RD
  RD --> STEP1["가게로 이동 · 길안내"]
  STEP1 --> STEP2["픽업 완료 (주문번호 확인)"]
  STEP2 --> STEP3["고객에게 이동 · 길안내 · 안심번호 통화"]
  STEP3 --> STEP4["배달 완료 (사진 · 문 앞 전달)"]
  RE --> RE1["일별·주별 수익 · 건수"]
  RE --> RE2["정산 내역 [2]"]
  RM --> RM1["서류·보험 정보 [2]"]
  RM --> RM2["운송수단 · 계좌"]
```

---

## 4. 계약업체(업주) 대시보드 메뉴 구조

```mermaid
flowchart TD
  P((계약업체 /partner)) --> P0[홈 요약]
  P --> P1[주문 관리]
  P --> P2[상품 관리]
  P --> P3[가게 관리]
  P --> P4[매출·정산]
  P --> P5[리뷰 관리]
  P --> P6[쿠폰·프로모션 2]
  P --> P7[계정·계약]

  P0 --> P0a["오늘 매출 · 주문 수 · 평균 조리시간 · 신규 리뷰"]
  P1 --> P1a["신규 → 접수(조리시간 선택) / 거절(사유)"]
  P1 --> P1b["조리중 → 조리완료(라이더 호출)"]
  P1 --> P1c["배달중 · 완료 · 취소 내역"]
  P2 --> P2a["메뉴 카테고리 등록 · 정렬"]
  P2 --> P2b["메뉴 등록: 이름 · 설명 · 가격 · 사진 · 대표/인기"]
  P2 --> P2c["옵션 그룹: 필수/선택 · 최소/최대 선택 수 · 추가금"]
  P2 --> P2d["품절 처리 · 숨김"]
  P3 --> P3a["영업시간 · 휴무일 · 임시 영업중지"]
  P3 --> P3b["배달 반경 · 최소주문 · 배달팁 구간"]
  P3 --> P3c["배달방식: 배달 / 가게배달 / 포장"]
  P4 --> P4a["매출 그래프: 일·주·월"]
  P4 --> P4b["정산 예정액 = 매출 − 수수료 − 배달비 − 쿠폰부담"]
  P4 --> P4c["정산 내역 · 세금계산서 다운로드"]
  P5 --> P5a["리뷰 목록 · 사장님 댓글"]
  P7 --> P7a["사업자 정보 · 정산 계좌 · 계약 수수료율"]
```

---

## 5. 관리자(본사) 대시보드 메뉴 구조

```mermaid
flowchart TD
  AD((관리자 /admin)) --> A0["관제 홈: 실시간 주문 · 결제액 · 배달 지연"]
  AD --> A1["입점 심사: 신청 → 서류 확인 → 계약 → 승인/반려"]
  AD --> A2["가게 관리: 수수료율 · 노출 · 제재"]
  AD --> A3["주문·결제 관리: 조회 · 강제 취소 · 환불"]
  AD --> A4["라이더 관제: 지도에 운행 중 라이더 · 배차 현황"]
  AD --> A5["정산 관리: 정산 생성 → 검토 → 지급 확정"]
  AD --> A6["쿠폰·배너 관리"]
  AD --> A7["고객 문의 · 신고"]
```

---

## 6. 주문 흐름 (시퀀스)

```mermaid
sequenceDiagram
  autonumber
  actor C as 고객
  participant APP as 고객 앱
  participant EF as Edge Function
  participant PG as PG사
  participant DB as Supabase DB
  participant RT as Realtime
  actor P as 계약업체
  actor R as 라이더

  C->>APP: 장바구니 → 주문하기
  APP->>EF: create_order(장바구니, 배달지, 쿠폰)
  EF->>DB: 금액 서버 재계산 · 최소주문/반경 검증
  EF-->>APP: 결제 요청 정보(orderId, amount)
  APP->>PG: 결제창
  PG-->>EF: 결제 승인 웹훅
  EF->>DB: payments=paid, orders=placed
  DB-->>RT: 신규 주문 이벤트
  RT-->>P: 신규 주문 알림(소리)
  P->>DB: 접수 + 조리시간 30분
  DB-->>RT: accepted
  RT-->>APP: "가게에서 주문을 접수했어요"
  P->>DB: 조리완료(ready)
  DB->>EF: 배차 요청
  EF-->>R: 배차 카드(푸시)
  R->>DB: 수락 → 픽업 → 배달완료
  DB-->>RT: 상태 · 위치 갱신
  RT-->>APP: 실시간 추적
  C->>APP: 리뷰 작성
```

---

## 7. 주문 상태 머신

```mermaid
stateDiagram-v2
  [*] --> pending_payment: 주문서 생성
  pending_payment --> placed: 결제 승인
  pending_payment --> [*]: 결제 실패·이탈
  placed --> accepted: 업주 접수(조리시간)
  placed --> canceled: 업주 거절 · 고객 취소
  accepted --> preparing
  preparing --> ready: 조리완료
  ready --> dispatched: 라이더 배정
  dispatched --> picked_up: 픽업
  picked_up --> delivered: 배달완료
  ready --> delivered: 포장 수령 / 가게배달 완료
  accepted --> canceled: 업주 취소(환불)
  delivered --> [*]
  canceled --> [*]
```

| 상태 | 고객 문구 | 업주 화면 | 색 토큰 |
|---|---|---|---|
| placed | 가게에서 확인하고 있어요 | 신규 | order.placed |
| accepted/preparing | 맛있게 조리하고 있어요 | 조리중 | order.preparing |
| ready/dispatched | 라이더가 가게로 가고 있어요 | 픽업대기 | order.ready |
| picked_up | 라이더가 출발했어요 | 배달중 | order.pickedUp |
| delivered | 배달이 완료됐어요 | 완료 | order.delivered |
| canceled | 주문이 취소됐어요 | 취소 | order.canceled |

---

## 8. 라이더 위치 확인 (위치 기반)

```mermaid
sequenceDiagram
  autonumber
  actor R as 라이더 앱
  participant GPS as 기기 GPS
  participant RT as Realtime 주문 위치 채널
  participant DB as rider_locations
  actor C as 고객 앱
  actor P as 업주 대시보드

  R->>GPS: 운행 시작(백그라운드 위치 권한)
  loop 배달 중 3~5초 간격 (정차 시 15초)
    GPS-->>R: lat, lng, heading, speed
    R->>RT: broadcast(위치)
    R->>DB: 30초마다 최신 위치 upsert(이력 축약)
  end
  RT-->>C: 지도 마커 이동 + 도착 예정 갱신
  RT-->>P: 픽업 전 "라이더 도착까지 n분"
  Note over C: 픽업 전에는 가게 방향,<br/>픽업 후에는 고객 방향 경로 표시
  R->>RT: 배달완료 → 채널 종료
  Note over DB: 배달 완료 후 위치 이력은<br/>7일 뒤 자동 삭제(개인정보)
```

**위치 설계 기준**

| 항목 | 기준 |
|---|---|
| 실시간 전달 | Supabase Realtime Broadcast (DB 쓰기 없이 전달) |
| 저장 | `rider_locations`에 라이더당 최신 1행 + 주기적 이력 |
| 공개 범위 | 배정된 주문의 고객·업주만 구독 가능, 배달 완료 시 차단 |
| 거리 계산 | PostGIS `geography`로 가게 반경·라이더 배차 거리 계산 |
| 배터리 | 이동 중 3~5초, 정차 시 15초로 간격 조절 |
| 도착 예정 | MVP: 직선거리 ÷ 평균속도, 2차: 지도 API 경로 시간 |

---

## 9. 라이더 상태 머신 · 배차

```mermaid
stateDiagram-v2
  [*] --> offline
  offline --> idle: 운행 시작
  idle --> offered: 배차 요청 수신
  offered --> idle: 거절 · 시간초과(30초)
  offered --> to_store: 수락
  to_store --> at_store: 가게 도착(반경 50m)
  at_store --> to_customer: 픽업 완료
  to_customer --> idle: 배달 완료
  idle --> offline: 운행 종료
```

```mermaid
flowchart LR
  A[조리완료 or 예상 픽업 n분 전] --> B["가게 반경 2km 내<br/>idle 라이더 검색 (PostGIS)"]
  B --> C[거리·수락률 순 정렬]
  C --> D[1순위에게 배차 요청 30초]
  D -->|거절·초과| E[다음 순위]
  E --> D
  D -->|수락| F[deliveries.rider_id 확정]
  B -->|라이더 없음| G[반경 확대 3km → 관리자 알림]
```

---

## 10. 계약업체 입점 · 상품 등록 흐름

```mermaid
flowchart TD
  A[입점 신청<br/>사업자등록증 · 영업신고증 · 통장사본] --> B{관리자 심사}
  B -->|반려| A
  B -->|승인| C[계약: 수수료율 · 정산주기 확정]
  C --> D[업주 계정 활성화]
  D --> E[가게 정보 입력<br/>주소 → 좌표 · 영업시간 · 배달반경]
  E --> F[메뉴 카테고리 생성]
  F --> G[메뉴 등록<br/>이름 · 가격 · 설명 · 사진]
  G --> H[옵션 그룹 연결<br/>필수/선택 · 최소/최대 · 추가금]
  H --> I{미리보기 확인}
  I -->|수정| G
  I -->|완료| J[영업 시작 토글]
  J --> K[고객 앱 노출]
```

```mermaid
sequenceDiagram
  actor P as 업주
  participant W as /partner 상품관리
  participant ST as Storage
  participant DB as menus · option_groups · options

  P->>W: 메뉴 추가
  P->>W: 사진 선택
  W->>ST: 업로드(자동 리사이즈 1:1, 16:9)
  ST-->>W: image_url
  P->>W: 이름 · 가격 · 옵션 입력
  W->>DB: insert (트랜잭션)
  DB-->>W: 저장 완료
  P->>W: 품절 토글
  W->>DB: is_sold_out = true
  DB-->>W: Realtime → 고객 화면 즉시 '품절'
```

---

## 11. 결제 흐름 (결제 · 취소 · 환불)

```mermaid
sequenceDiagram
  autonumber
  actor C as 고객
  participant APP as 앱
  participant EF as Edge Function
  participant PG as PG사
  participant DB as DB

  C->>APP: 결제하기
  APP->>EF: prepare(orderId)
  EF->>DB: payments(pending, 금액 고정)
  APP->>PG: 결제창(카드·간편결제)
  PG-->>APP: 인증 완료
  APP->>EF: confirm(paymentKey)
  EF->>PG: 승인 요청(금액 검증)
  PG-->>EF: 승인
  EF->>DB: payments=paid, orders=placed
  alt 업주 거절 · 접수 전 고객 취소
    EF->>PG: 전액 취소
    EF->>DB: payments=refunded, orders=canceled
  else 부분 취소(품절 메뉴 제외)
    EF->>PG: 부분 취소
    EF->>DB: refunds 기록
  end
```

---

## 12. 정산 흐름 · 정산 대시보드 데이터

```mermaid
flowchart LR
  O[배달완료 주문] --> L["정산 원장 settlement_items<br/>주문별: 매출 · 수수료 · 배달비 · 쿠폰부담 · 결제수수료"]
  L --> B["정산 배치 (주 1회)<br/>기간별 합계 → settlements"]
  B --> R[관리자 검토]
  R -->|확정| PAY[지급 · 상태 paid]
  R -->|보류| H[사유 기록]
  L --> DASH1[업주 매출·정산 대시보드]
  B --> DASH1
  L --> DASH2[관리자 정산 관리]
  DL[라이더 배달 완료] --> RL[라이더 수익 원장] --> DASH3[라이더 수익 화면]
```

**정산 계산식 (주문 1건)**

```
정산액 = 주문금액(메뉴합계)
       − 중개수수료 (주문금액 × 계약 수수료율)
       − 결제수수료 (결제금액 × PG 수수료율)
       − 배달비 업주부담분
       − 업주부담 쿠폰
       + 고객이 낸 배달팁 (가게배달인 경우)
```

| 대시보드 | 지표 |
|---|---|
| 업주 | 오늘·이번주·이번달 매출, 주문 수, 객단가, 취소율, 정산 예정액, 지급 내역, 시간대별 주문 |
| 관리자 | 총 거래액(GMV), 수수료 수익, 결제수단 비율, 환불액, 정산 대기·확정·지급 현황 |
| 라이더 | 오늘 수익, 건수, 운행시간, 주간 정산 |

---

## 13. ERD (확장)

```mermaid
erDiagram
  profiles ||--o{ addresses : has
  profiles ||--o| rider_profiles : "is rider"
  profiles ||--o{ orders : places
  profiles ||--o{ reviews : writes
  profiles ||--o{ favorites : likes
  profiles ||--o{ user_coupons : owns

  partner_applications ||--o| stores : "approved as"
  profiles ||--o{ stores : owns
  store_categories ||--o{ stores : groups
  stores ||--o{ store_hours : opens
  stores ||--o{ delivery_fee_rules : charges
  stores ||--o{ menu_categories : has
  menu_categories ||--o{ menus : has
  menus ||--o{ option_groups : has
  option_groups ||--o{ options : has

  profiles ||--o| carts : owns
  carts ||--o{ cart_items : contains

  stores ||--o{ orders : receives
  orders ||--|{ order_items : contains
  orders ||--o{ order_status_logs : tracks
  orders ||--o| payments : "paid by"
  payments ||--o{ refunds : has
  orders ||--o| deliveries : "delivered by"
  rider_profiles ||--o{ deliveries : performs
  rider_profiles ||--o| rider_locations : "current at"
  deliveries ||--o{ dispatch_offers : offers
  orders ||--o| reviews : "reviewed by"

  orders ||--o| settlement_items : "settled in"
  settlements ||--o{ settlement_items : groups
  stores ||--o{ settlements : receives
  coupons ||--o{ user_coupons : issued

  stores {
    uuid id PK
    uuid owner_id FK
    geography location
    int delivery_radius_m
    int min_order_amount
    numeric commission_rate
    bool is_open
  }
  rider_locations {
    uuid rider_id PK
    geography location
    float heading
    float speed
    timestamptz updated_at
  }
  settlements {
    uuid id PK
    uuid store_id FK
    date period_start
    date period_end
    int gross_amount
    int fee_amount
    int payout_amount
    text status
  }
```

> 참고: `supabase/migrations/0001_init.sql`은 이 확장 ERD 이전 초안입니다. 개발 단계에서 이 ERD 기준으로 다시 맞춥니다.
