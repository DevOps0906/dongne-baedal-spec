# dongne-baedal-spec

동네배달(배달 앱) **기획·디자인·토큰·DB의 단일 원본**입니다.
`dongne-baedal-web`과 `dongne-baedal-flutter`가 이 레포를 `spec/` 서브모듈로 참조합니다.

## 문서

| 경로 | 내용 |
|---|---|
| `docs/01-benchmark.md` | 배달의민족 벤치마킹 분석 (기획·사업·디자인) |
| `docs/02-diagrams.md` | 다이어그램 13장: 시스템, 4역할 메뉴 구조, 주문·라이더 위치·상품 등록·결제·정산 흐름, ERD (Mermaid) |
| `docs/03-design-structure.md` | 고객·라이더·계약업체·관리자 화면 목록, 반응형 3구간, 지도 UI, 카피 톤 |
| `docs/04-dev-instructions.md` | **개발지시서**: 단계 계획, 작업 티켓, 완료 기준, AI 에이전트 지시 템플릿 |
| `design/canvas.html` | 디자인 캔버스: 디자인 시스템, 반응형 와이어, 주요 화면 시안, 다이어그램 |
| `tokens/tokens.json` | 색·간격·모서리·반응형 글자 크기 토큰 |

읽는 순서: 01 → 02 → 03 → 04. 디자인은 `design/canvas.html`을 브라우저로 열어 봅니다.

## 3 레포 구조

```
dongne-baedal-spec        ← 원본 (이 레포): 문서 · 토큰 · supabase 마이그레이션 · Edge Functions
  ├─ dongne-baedal-web        Next.js: 고객 웹/PWA · /partner 계약업체 · /admin 관리자
  └─ dongne-baedal-flutter    Flutter: 고객 앱 · 라이더 모드
```

| 역할 | 채널 |
|---|---|
| 고객 | Flutter 앱, 웹/PWA |
| 라이더 | Flutter 앱(라이더 모드) |
| 계약업체 | 웹 `/partner` |
| 관리자 | 웹 `/admin` |

---

## 개발 방법

### 1. 준비물

| 도구 | 용도 |
|---|---|
| Git 2.40+ | 서브모듈 사용 |
| Node.js 20+ | 웹, 토큰 생성 |
| Flutter 3.32 / Dart 3.8 | 앱 |
| Supabase CLI + Docker | 로컬 DB, 마이그레이션, Edge Functions |

### 2. 세 레포 받기

```bash
mkdir dongne-baedal && cd dongne-baedal
git clone https://github.com/DevOps0906/dongne-baedal-spec.git
git clone --recurse-submodules https://github.com/DevOps0906/dongne-baedal-web.git
git clone --recurse-submodules https://github.com/DevOps0906/dongne-baedal-flutter.git
```

### 3. 로컬 DB (P0 SPEC-001 이후)

```bash
cd dongne-baedal-spec
supabase start                 # 로컬 Postgres · Auth · Storage
supabase db reset              # migrations + seed 적용
supabase functions serve       # Edge Functions 로컬 실행
```

`supabase start`가 출력하는 `API URL`, `anon key`를 web의 `.env.local`, flutter의 `--dart-define`에 넣습니다.

### 4. spec을 바꾸는 순서

토큰·스키마·화면 규칙은 **항상 이 레포에서 먼저** 바꿉니다.

```bash
# 1) spec에서 수정 후 커밋·push
cd dongne-baedal-spec
git checkout -b feat/SPEC-020-order-schema
# ...수정...
git commit -m "feat(SPEC-020): 주문 스키마와 RLS 추가"
git push -u origin feat/SPEC-020-order-schema   # PR → master 병합

# 2) 구현 레포에서 서브모듈 포인터 갱신
cd ../dongne-baedal-web
git submodule update --remote spec
npm run tokens            # 토큰이 바뀐 경우
git commit -am "chore: spec 갱신 (SPEC-020)"
```

flutter도 같은 방식이며 토큰 생성은 `dart run tool/gen_tokens.dart`입니다.

### 5. 마이그레이션 규칙

- 파일명: `supabase/migrations/000N_<영역>.sql` (예: `0002_order.sql`)
- 한 번 `master`에 병합된 마이그레이션은 수정하지 않고 새 파일로 고칩니다
- 테이블을 만들면 같은 파일에서 RLS 정책을 함께 만듭니다
- 타입 생성: web에서 `supabase gen types typescript --local > src/types/database.ts`

### 6. 작업 진행

1. `docs/04-dev-instructions.md`에서 티켓을 고릅니다 (선행 티켓 완료 확인)
2. 브랜치: `feat/<티켓>-<요약>`
3. 커밋: Conventional Commits, 제목에 티켓 번호
4. PR: 완료 기준 체크리스트 + 반응형 3구간 캡처
5. AI 에이전트에 맡길 때는 지시서 7장의 템플릿을 사용합니다

## 반응형 기준

모바일 ~767 / 태블릿 768~1199 / 데스크탑 1200~
구간별 글자 크기는 `tokens/tokens.json`의 `typography.responsive`에 있습니다.

## 상태

- v0.1: 벤치마킹 분석, 다이어그램 13장, 디자인 구조와 캔버스, 토큰, 개발지시서
- 다음: P0 착수 (SPEC-001 스키마부터)

## 저작권 주의

배달의민족의 이름, 로고, 색 코드, 전용 서체, 문구는 사용하지 않습니다. 벤치마킹은 구조와 흐름에 한정합니다.
