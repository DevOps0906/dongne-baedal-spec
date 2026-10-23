# 04. 개발지시서

> 동네배달을 실제로 개발할 때 따르는 작업 지시서입니다.
> 사람 개발자와 AI 코딩 에이전트(Claude Code 등)가 같은 문서를 보고 작업합니다.
> 기준 문서: `01-benchmark.md`(왜), `02-diagrams.md`(무엇이 어떻게 흐르는가), `03-design-structure.md`(어떻게 보이는가), `tokens/tokens.json`(값).

---

## 0. 작업 원칙

| # | 원칙 | 의미 |
|---|---|---|
| 1 | spec이 원본 | 토큰·스키마·화면 규칙은 spec에서 먼저 바꾸고, 구현 레포는 서브모듈 포인터만 올린다 |
| 2 | 티켓 단위 작업 | 한 번에 티켓 하나. 브랜치 하나, PR 하나 |
| 3 | 금액은 서버가 계산 | 클라이언트가 보낸 금액은 믿지 않는다. 주문 금액은 RPC·Edge Function에서 다시 계산 |
| 4 | RLS 먼저 | 테이블을 만들면 같은 마이그레이션에서 RLS 정책도 만든다 |
| 5 | 반응형 3구간 | 모든 화면은 ~767 / 768~1199 / 1200~ 에서 확인하고 PR에 캡처를 붙인다 |
| 6 | 토큰만 사용 | 색·글자 크기·간격·모서리에 하드코딩 값을 쓰지 않는다 |
| 7 | 브랜드 요소 금지 | 배달의민족의 이름·로고·색·폰트·문구를 코드와 에셋에 넣지 않는다 |

---

## 1. 기술 스택 (고정)

| 영역 | 선택 | 버전 기준 |
|---|---|---|
| DB·인증·실시간 | Supabase (Postgres 15+, PostGIS, Auth, Realtime, Storage, Edge Functions) | Supabase CLI 최신 |
| 웹 | Next.js App Router + TypeScript + Tailwind CSS v4 + shadcn/ui | Next.js 15, React 19 |
| 웹 상태·데이터 | `@supabase/ssr`, TanStack Query, Zod, React Hook Form | |
| 웹 차트 | Recharts | |
| 앱 | Flutter + Dart, Riverpod, GoRouter, Freezed | Flutter 3.32 / Dart 3.8, Riverpod 2.6, GoRouter 14 |
| 앱 지도·위치 | `flutter_naver_map`, `geolocator`, 백그라운드 위치 서비스 | |
| 결제 | 토스페이먼츠 (MVP는 `mock` 공급자) | |
| 푸시 | FCM (`firebase_messaging`) | |
| 배포 | 웹 Vercel, 앱 Play Console 내부 테스트 → TestFlight | |

앱 패키지명: `com.nomad.dongnebaedal`

---

## 2. 레포별 책임

| 레포 | 만드는 것 | 만들지 않는 것 |
|---|---|---|
| `dongne-baedal-spec` | 문서, 토큰, `supabase/migrations`, `supabase/functions`, 시드 | 화면 코드 |
| `dongne-baedal-web` | 고객 웹/PWA, `/partner` 계약업체 대시보드, `/admin` 관리자 | DB 스키마 (spec에서 가져옴) |
| `dongne-baedal-flutter` | 고객 앱, 라이더 모드 | DB 스키마, 업주·관리자 화면 |

> Supabase 마이그레이션과 Edge Function은 **spec 레포에만** 둡니다. web·flutter는 `spec/supabase`를 참조해 타입만 생성합니다.

---

## 3. 단계 계획 (MVP 10주)

| 단계 | 기간 | 목표 | 완료 판단 |
|---|---|---|---|
| P0 기반 | 1주 | 스키마, 토큰 생성, 앱·웹 뼈대, 인증 | 세 채널에서 로그인 후 빈 홈이 뜬다 |
| P1 탐색 | 2주 | 홈·카테고리·가게 리스트·가게 상세·메뉴 옵션 | 시드 가게 20개를 반경 안에서 탐색 |
| P2 주문·결제 | 2주 | 장바구니·주문서·결제(mock→토스)·주문내역 | 결제 후 주문이 `placed`로 생성 |
| P3 업주 | 2주 | 입점 신청, 상품 등록, 주문 접수 칸반 | 업주가 메뉴 등록 → 고객 주문 → 접수까지 |
| P4 라이더·위치 | 2주 | 배차, 라이더 앱, 실시간 위치, 주문 현황 지도 | 고객 지도에서 라이더가 움직인다 |
| P5 정산·관리자 | 1주 | 정산 원장·배치, 업주 정산 화면, 관리자 대시보드 | 주간 정산이 생성되고 관리자가 확정 |

2차(장보기·포인트)와 3차(선물하기·멤버십)는 MVP 이후 별도 지시서로 작성합니다.

---

## 4. 작업 티켓

표기: `SPEC-`, `WEB-`, `APP-` 접두사는 작업할 레포입니다. **선행**은 먼저 끝나야 하는 티켓입니다.

### P0 기반

| 티켓 | 내용 | 선행 | 완료 기준 |
|---|---|---|---|
| SPEC-001 | `supabase/` 초기화, `02-diagrams.md` 13장 ERD 기준 `0001_core.sql` (profiles, addresses, store_categories, stores, store_hours, delivery_fee_rules, menu_categories, menus, option_groups, options) + RLS | – | `supabase db reset` 성공, RLS 테스트 SQL 통과 |
| SPEC-002 | PostGIS 활성화, `stores.location geography(Point)`, `nearby_stores(lat,lng,radius)` RPC | 001 | 좌표 기준 반경 검색 결과가 거리순 |
| SPEC-003 | 시드: 카테고리 10, 가게 20(신갈·구갈·상갈), 메뉴·옵션, 테스트 계정 4역할 | 001 | 역할별 로그인 가능 |
| WEB-001 | Next.js 15 생성, `spec/` 서브모듈, `scripts/gen-tokens.ts` → `src/styles/tokens.css`, Tailwind v4 테마 연결 | – | `npm run tokens && npm run build` 통과 |
| WEB-002 | Supabase 클라이언트(서버·브라우저), 미들웨어 세션 갱신, `supabase gen types` → `src/types/database.ts` | WEB-001, SPEC-001 | 로그인 후 서버 컴포넌트에서 세션 조회 |
| WEB-003 | 레이아웃 3종: 고객(하단 탭/상단 헤더), `/partner`(사이드바), `/admin`(사이드바) + 역할 가드 | WEB-002 | 역할이 다르면 403 페이지 |
| APP-001 | Flutter 생성, `spec/` 서브모듈, `tool/gen_tokens.dart` → `lib/core/theme/tokens.g.dart`, 반응형 `Breakpoint` 유틸 | – | `flutter analyze` 0건 |
| APP-002 | Clean Architecture 폴더, Riverpod, GoRouter 5탭 셸, 라이더 모드 라우트 분리 | APP-001 | 탭 전환, 딥링크 `/store/:id` 동작 |
| APP-003 | 인증: 휴대폰 OTP(Supabase), 카카오·네이버(Edge Function → JWT), 애플 | APP-002, SPEC-001 | 세 방식 로그인 후 `profiles` 생성 |

### P1 탐색

| 티켓 | 내용 | 선행 | 완료 기준 |
|---|---|---|---|
| SPEC-010 | `search_stores(q, lat, lng)` RPC (가게명·메뉴명) | 002 | 메뉴명으로 가게 검색 |
| APP-010 · WEB-010 | 배달지 설정 (C02): 현재 위치, 주소 검색, 핀 조정, 저장 | 003 / 002 | 기본 배달지 1개 유지 |
| APP-011 · WEB-011 | 홈 (C03): 서비스 탭, 카테고리 그리드, 배너, 인기 가게 | 010 | 반경 밖 가게 미노출 |
| APP-012 · WEB-012 | 가게 리스트 (C04): 정렬 5종, 필터 시트 | 011 | 정렬이 서버 쿼리로 반영 |
| APP-013 · WEB-013 | 가게 상세 (C06): 배달정보 바, 스티키 메뉴 탭, 리뷰 탭 | 012 | 품절 메뉴 흐리게 표시 |
| APP-014 · WEB-014 | 메뉴 옵션 시트 (C07): 필수·최대 검증, 금액 계산 | 013 | 필수 미선택 시 담기 비활성 |
| APP-015 · WEB-015 | 검색 (C05), 찜 (C13) | 010 | 최근 검색어 로컬 저장 |

### P2 주문·결제

| 티켓 | 내용 | 선행 | 완료 기준 |
|---|---|---|---|
| SPEC-020 | `0002_order.sql`: carts, cart_items, coupons, user_coupons, orders, order_items, order_status_logs, payments, refunds + RLS | 001 | 타 사용자 주문 조회 불가 |
| SPEC-021 | Edge Function `create-order`: 금액 재계산, 최소주문·반경·영업시간 검증, `pending_payment` 생성 | 020 | 조작된 금액 요청 거부 |
| SPEC-022 | Edge Function `payment-confirm`, `payment-cancel` (공급자 인터페이스: mock / toss) | 021 | mock 결제 성공·취소 흐름 |
| APP-020 · WEB-020 | 장바구니 (C08): 한 가게 제한, 배달방식 전환, 최소주문 진행바 | 014, 020 | 다른 가게 담으면 교체 확인 |
| APP-021 · WEB-021 | 주문서·결제 (C09): 배달지, 요청사항, 쿠폰, 결제수단 | 022 | 결제 성공 시 주문 현황으로 이동 |
| APP-022 · WEB-022 | 주문내역 (C11), 재주문, 리뷰 작성 (C12) | 021 | 배달완료 주문만 리뷰 가능 |

### P3 계약업체 (web)

| 티켓 | 내용 | 선행 | 완료 기준 |
|---|---|---|---|
| SPEC-030 | `0003_partner.sql`: partner_applications, 계약 필드, Storage 버킷 `menu-images` 정책 | 020 | 본인 가게 이미지만 업로드 |
| WEB-030 | 입점 신청 폼 + 서류 업로드 | 030 | 신청 상태 `submitted` |
| WEB-031 | P05 가게 관리: 영업시간 주간표, 배달 반경 지도, 배달팁 구간 | 030 | 영업시간 밖이면 고객 주문 불가 |
| WEB-032 | P03·P04 상품 관리: 카테고리 정렬, 메뉴·옵션 빌더, 사진 업로드, 고객 화면 미리보기, 품절 토글 | 030 | 품절 토글이 고객 화면에 1초 내 반영 |
| WEB-033 | P02 주문 관리 칸반: Realtime 신규 주문, 알림음, 조리시간 칩 접수·거절, 조리완료 | 022 | 접수 시 고객 앱 상태 변경 |
| WEB-034 | P01 홈 요약, P07 리뷰 관리(사장님 댓글) | 033 | KPI가 오늘 주문과 일치 |

### P4 라이더·위치

| 티켓 | 내용 | 선행 | 완료 기준 |
|---|---|---|---|
| SPEC-040 | `0004_delivery.sql`: rider_profiles, deliveries, dispatch_offers, rider_locations + 7일 이력 삭제 크론 | 020 | 배정된 주문의 고객·업주만 위치 조회 |
| SPEC-041 | Edge Function `dispatch`: 반경 2km idle 라이더 거리순, 30초 응답, 실패 시 3km 확대·관리자 알림 | 040 | 거절·시간초과 시 다음 라이더 |
| APP-040 | 라이더 R01 운행 홈: 운행 토글, 백그라운드 위치 권한 | 040 | 운행 종료 시 위치 전송 중단 |
| APP-041 | 위치 전송: 이동 3~5초 / 정차 15초 Broadcast, 30초마다 DB upsert | 040 | 배터리 1시간 사용량 측정 기록 |
| APP-042 | R02 배차 요청 (30초 타이머), R03 진행 중 배달, R04 완료 사진 | 041 | 상태 변경이 고객·업주에 반영 |
| APP-043 · WEB-043 | C10 주문 현황: 지도 마커 보간, 도착 예정, 타임라인, 신호 끊김 표시 | 041 | 30초 끊김 시 반투명 마커 |
| SPEC-042 | FCM 푸시: 주문 상태 변경, 배차 요청 | 041 | 앱 백그라운드에서 수신 |

### P5 정산·관리자

| 티켓 | 내용 | 선행 | 완료 기준 |
|---|---|---|---|
| SPEC-050 | `0005_settlement.sql`: settlement_items(주문 완료 트리거), settlements, 주간 배치 함수 | 040 | 계산식이 `02-diagrams.md` 12장과 일치하는 테스트 |
| WEB-050 | P06 매출·정산: 기간 탭, KPI, 일별 매출 차트, 정산 계산 펼침표, 정산 내역 | 050 | 합계가 원장 합과 1원 단위로 일치 |
| WEB-051 | A01 관제 홈, A02 입점 심사(승인 시 가게·계약 생성) | 030 | 승인 → 업주 로그인 가능 |
| WEB-052 | A03 주문·결제 조회·강제 취소·환불 | 022 | 환불 시 refunds 기록 |
| WEB-053 | A04 라이더 관제 지도, 미배차 리스트, 수동 배차 | 041 | 수동 배차가 라이더 앱에 도착 |
| WEB-054 | A05 정산 관리: 대기 → 검토 → 확정 → 지급 | 050 | 확정 후 금액 수정 불가 |

---

## 5. 폴더 구조 기준

### web

```
src/
  app/
    (customer)/          홈, 가게, 장바구니, 주문
    partner/             계약업체 대시보드
    admin/               관리자
    api/                 웹훅 등 서버 라우트
  components/
    ui/                  shadcn/ui
    customer/ partner/ admin/ shared/
  lib/
    supabase/            server.ts, client.ts, middleware.ts
    queries/             TanStack Query 훅
    validators/          Zod 스키마
  styles/tokens.css      (생성 파일, 수정 금지)
  types/database.ts      (생성 파일, 수정 금지)
spec/                    서브모듈
```

### flutter

```
lib/
  core/
    theme/tokens.g.dart  (생성 파일, 수정 금지)
    theme/app_theme.dart
    router/ network/ utils/breakpoint.dart
  features/
    auth/ address/ home/ store/ cart/ order/ review/ my/
    rider/               라이더 모드
      data/ domain/ presentation/
  main.dart
tool/gen_tokens.dart
spec/                    서브모듈
```

---

## 6. 공통 규칙

**브랜치·커밋**
- 브랜치: `feat/WEB-032-menu-builder`, `fix/APP-041-location-interval`
- 커밋: Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`), 제목에 티켓 번호
- PR 본문: 티켓 번호, 변경 요약, 3구간 캡처, 완료 기준 체크

**완료 정의 (모든 티켓 공통)**
- [ ] 티켓의 완료 기준 충족
- [ ] 웹: `npm run lint && npm run typecheck && npm run build` 통과 / 앱: `flutter analyze && flutter test` 통과
- [ ] 반응형 3구간 확인 (해당 화면)
- [ ] 토큰 외 하드코딩 색·크기 없음
- [ ] 새 테이블은 RLS 정책과 테스트 포함
- [ ] 화면 문구는 `03-design-structure.md` 9장 카피 톤을 따름

**보안**
- 서비스 롤 키는 Edge Function과 서버 라우트에서만 사용, 클라이언트 번들 금지
- 위치·주소·연락처는 본인과 배정된 상대만 조회
- 결제 웹훅은 서명 검증 후 처리

---

## 7. AI 코딩 에이전트 지시 템플릿

티켓 하나를 맡길 때 아래 형식을 그대로 붙여 씁니다.

```text
[역할] 너는 dongne-baedal-{web|flutter|spec} 레포의 개발자다.
[티켓] {티켓 번호} {제목}
[참고] spec/docs/02-diagrams.md {장}, spec/docs/03-design-structure.md {화면 번호}, spec/tokens/tokens.json
[범위] 이 티켓의 완료 기준만 구현한다. 다른 티켓 범위는 건드리지 않는다.
[규칙]
- 토큰만 사용하고 반응형 3구간(~767 / 768~1199 / 1200~)을 지킨다
- 금액 계산은 서버에서 한다
- 새 테이블은 RLS를 같이 만든다
- 배달의민족 브랜드 요소를 쓰지 않는다
[검증] lint·typecheck·build(또는 flutter analyze·test)를 실행하고 결과를 보고한다
[산출] 브랜치 feat/{티켓}-{요약}, 커밋, PR 본문 초안
```

예시:

```text
[역할] 너는 dongne-baedal-web 레포의 개발자다.
[티켓] WEB-033 주문 관리 칸반
[참고] spec/docs/02-diagrams.md 4장·6장·7장, spec/docs/03-design-structure.md P02, design/canvas.html 계약업체 섹션
[범위] 신규·조리중·픽업대기/배달중·완료 4열 칸반, Realtime 신규 주문 구독, 알림음, 조리시간 칩 접수, 거절 사유, 조리완료
...
```

---

## 8. 결정 대기 항목

| 항목 | 현재 가정 | 결정 시점 |
|---|---|---|
| 지도 SDK | 네이버 지도 (대안 카카오맵) | P0 전 |
| PG사 | 토스페이먼츠, MVP는 mock | P2 전 |
| 중개수수료율 | 9.8% (임시) | P5 전, 계약 조건 |
| 정산 주기 | 주 1회 | P5 전 |
| 첫 서비스 지역 | 용인 기흥구 신갈·구갈·상갈 | P1 시드 작성 전 |
