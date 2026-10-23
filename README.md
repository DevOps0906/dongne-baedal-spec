# dongne-baedal-spec

동네배달(배달 앱) **기획·디자인·토큰·DB 스키마의 단일 원본**입니다.
`dongne-baedal-web`과 `dongne-baedal-flutter`가 이 레포를 `spec/` 서브모듈로 참조합니다.

## 구조

| 경로 | 내용 |
|---|---|
| `docs/01-benchmark.md` | 배달의민족 벤치마킹 분석 (기획·사업·디자인) |
| `docs/02-diagrams.md` | 다이어그램 13장: 시스템, 4역할 메뉴 구조, 주문·라이더 위치·상품 등록·결제·정산 흐름, ERD (Mermaid) |
| `docs/03-design-structure.md` | 고객·라이더·계약업체·관리자 화면 목록, 반응형 3구간, 지도 UI, 카피 톤 |
| `design/canvas.html` | 디자인 캔버스: 디자인 시스템, 반응형 와이어, 주요 화면 시안, 다이어그램 |
| `tokens/tokens.json` | 색·간격·모서리·반응형 글자 크기 토큰 |
| `supabase/migrations/0001_init.sql` | 테이블, 트리거, RLS |
| `supabase/seed.sql` | 카테고리·쿠폰 시드 |

## 3 레포 관계

```
dongne-baedal-spec     ← 원본 (이 레포)
  ├─ dongne-baedal-web      (Next.js, 소비자 웹 + 업주 관리자)
  └─ dongne-baedal-flutter  (Flutter, 소비자 앱 + 라이더 화면)
```

토큰이나 스키마를 바꾸면 이 레포에서 먼저 커밋하고, 두 구현 레포에서 서브모듈 포인터를 올립니다.

```bash
git submodule update --remote spec
```

## 반응형 기준

모바일 ~767 / 태블릿 768~1199 / 데스크탑 1200~ (구간별 글자 크기는 `tokens.json` 참고)

## 상태

- v0.1: 분석·구조·토큰·초기 스키마
- 스키마는 실제 Postgres에서 아직 실행 검증하지 않았습니다.
- 다음: `0002` 마이그레이션 (`place_order()` RPC, 거리 검색, Realtime 설정)

## 저작권 주의

배달의민족의 이름, 로고, 색 코드, 전용 서체는 사용하지 않습니다. 벤치마킹은 구조와 흐름에 한정합니다.
