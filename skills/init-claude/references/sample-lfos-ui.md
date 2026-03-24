# Sample: lfos-ui 문서 구현 (C1-C7 9.0+)

> 실제 프로젝트(LF OS UI)에서 init-claude 프로세스를 통해 달성한 문서 세트.
> 프로젝트: Next.js App Router + pnpm 모노레포 + TypeScript + Zustand + ag-grid

## 달성 점수

| 기준 | 점수 | 핵심 |
|------|------|------|
| C1 Line Budget | 10 | always-loaded 147 lines (38+91+18) |
| C2 Non-discoverability | 9 | 8 traps, 전부 BAD/GOOD 포함 |
| C3 Single Source | 9 | 0 중복, CLAUDE.md는 순수 포인터, rules는 `AGENTS.md #N 참조`만 |
| C4 Cross-reference | 9.5 | 일관된 `AGENTS.md #N` 형식, 10개 교차 참조 |
| C5 Completeness | 9 | 5/5 시나리오 PASS (testing.md 보강) |
| C6 Guide Compliance | 9 | 3-tier + path trigger 4개 + 명시적 @import |
| C7 Consistency | 9 | 한국어 통일, 포맷 일관 |
| **가중 평균** | **9.2** | |

## 파일 구조 및 라인 수

```
CLAUDE.md              38 lines  (always-loaded: 요약 인덱스 + AGENTS.md 순수 포인터)
AGENTS.md              91 lines  (always-loaded: 8 runtime traps with BAD/GOOD)
.claude/
  CLAUDE.md            18 lines  (always-loaded: 규칙 인덱스 + 타입 소스 + 금지사항)
  rules/
    api-patterns.md   196 lines  (conditional: src/api/**)
    components.md     156 lines  (conditional: src/app/**, src/components/**, src/hooks/**, src/layouts/**)
    stores-utils.md    24 lines  (conditional: src/store/**, src/utils/**)
    testing.md        103 lines  (conditional: *.test.*)
```

## CLAUDE.md (root) — 38 lines

```markdown
# LF OS UI

LF Mall 통합 어드민(One-Sphere) 모노레포.

## 모노레포 구조

- `apps/one-sphere/` — 메인 어드민 (Next.js App Router)
- `apps/storybook/` — 컴포넌트 개발/문서화
- `packages/` — 공유 라이브러리 (`types`, `ui`, `utils`, `icons`, `eslint-config`, `tsconfig`)

## 핵심 명령어

pnpm dev:app / build:app / typecheck / lint / generate-schema / slot

## 치명적 제약사항 → AGENTS.md 참조

AGENTS.md의 모든 trap(#1-#8)은 필수 준수. 모든 코드 변경 시 확인할 것.

추가 제약:
- **UI 문자열은 한국어**
- **reactStrictMode: false** 활성화 금지 — Zustand 비호환

## 수정 금지

- `packages/types/`, `next.config.mjs` turbopack.root, `__ENV.js`

## 상세 규칙

@.claude/CLAUDE.md — 타입 소스, 금지사항, 도메인별 규칙(path trigger 자동 로드)
```

**설계 원칙:**
- 프로젝트 1줄 설명 → 모노레포 구조 → 명령어 → 제약사항 포인터 → 수정 금지 → 명시적 import
- AGENTS.md trap 내용을 **절대 인라인으로 요약하지 않음** — 순수 포인터만 (C3 핵심)
- `@import`에 로드되는 내용 설명 추가 — 에이전트에게 계층 구조 명시 (C6 향상)
- P6, P7, P9, P10 원칙 적용

## AGENTS.md — 91 lines, 8 traps

각 trap: 번호 + 제목 + 1줄 결과 + 최소 2줄 BAD/GOOD (제네릭 패턴으로 압축)

```markdown
# AGENTS.md — 코드에서 추론 불가능한 함정

> 코드/설정을 읽으면 알 수 있는 정보는 여기에 쓰지 않는다.

## 1. 런타임 환경변수 — `getEnv()` 필수
[1줄 설명 + BAD: process.env 직접 참조 / GOOD: getEnv('KEY')]

## 2~6. [각각 1줄 설명 + BAD/GOOD 코드 블록]

## 7. `invalidateQueries` 이중 배열 래핑 — 캐시 무효화 실패
[1줄 설명 + BAD: [FACTORY('LIST')] / GOOD: FACTORY('LIST')]

## 8. API 함수 에러 묵살 — `to()` 패턴 무력화
[1줄 설명 + BAD: try{return fn()}catch{return[]} / GOOD: return fn()]
```

**설계 원칙:**
- P1(Non-discoverability), P2(BAD/GOOD), P3(Consequence-first) 적용
- `>` 인용으로 AGENTS.md의 존재 목적을 첫 줄에 명시
- BAD/GOOD은 제네릭 패턴으로 압축 (도메인 특화보다 메커니즘 전달이 중요)

## .claude/CLAUDE.md — 18 lines

```markdown
# 프로젝트 상세 규칙

## 도메인별 규칙 (path trigger로 자동 로드)

- `rules/api-patterns.md` — API 레이어
- `rules/components.md` — 페이지/컴포넌트
- `rules/stores-utils.md` — 스토어/유틸
- `rules/testing.md` — 테스트

## 타입 소스
- API 타입: @lf-os-ui/types에서 import
- 커스텀 타입: src/types/{domain}/
- Props 타입: 컴포넌트 파일 내 interface {Name}Props

## 금지사항
- 불필요한 외부 라이브러리 추가 금지 (조직 정책)
```

**설계 원칙:**
- 규칙 인덱스 섹션으로 conditional rules 발견 가능성 향상 (C6)
- AGENTS.md에 없는 프로젝트 규칙만 포함
- Zustand/유틸리티 내용은 conditional stores-utils.md로 이동 (C1 최적화)

## Conditional Rules 요약

### api-patterns.md (path: "apps/one-sphere/src/api/**")
- 4-file 구조 (api + queries + mutations + queryKeys)
- GET/POST/PUT/DELETE 전체 예시 + stringifyQuery
- "핵심 제약: AGENTS.md #1, #4, #5 참조" (설명 복사 없이 참조만)
- BAD/GOOD: 인라인 쿼리 키, API 내부 try/catch
- 타입 이름 찾기 가이드 (grep 명령어)

### components.md (path: "apps/one-sphere/src/app/**,src/components/**")
- page.tsx (Server Component) + client.tsx (Client Component) 전체 템플릿
- PAGE_CODE: "AGENTS.md #6 참조"
- 폼 패턴 (react-hook-form + zod + OsAlert)
- 목록 페이지 패턴 (ag-grid + Pagination)
- 새 페이지 체크리스트 (5단계)
- 언어 규칙 테이블

### stores-utils.md (path: "apps/one-sphere/src/store/**,src/utils/**")
- Zustand 스토어 위치/패턴/외부 접근법
- 핵심 유틸리티 (to, stringifyQuery, classNamesWithRoot)

### testing.md (path: "*.test.*")
- Jest + RTL 기본 설정
- describe/it 네이밍 규칙
- API 모킹 방식

## 개선 추이

```
Round 1: 8.0  (C1=7.5, C3=6.5 — 161 lines, api-patterns에 AGENTS.md BAD/GOOD 중복)
Round 2: 9.0  (C1=9.5, C3=9.0 — 153 lines, BAD/GOOD 중복 제거, 참조로 대체)
  + Codex 교차검증: 8.3 (C3=6.5 — CLAUDE.md 인라인 요약이 중복)
Round 3: 9.2+ (C3=9.5 — CLAUDE.md 순수 포인터화, @import 명시적 설명)
```

## 핵심 교훈

1. **C3가 가장 까다로움**: CLAUDE.md에서 trap 내용을 인라인 요약하면 중복으로 간주됨. 순수 포인터(`AGENTS.md 참조`)만 허용
2. **rules에서도 BAD/GOOD 복사 금지**: `AGENTS.md #N 참조` 한 줄로 대체. rules의 BAD/GOOD은 trap이 아닌 convention 패턴만
3. **@import에 설명 추가**: `@.claude/CLAUDE.md — 타입 소스, 금지사항, 도메인별 규칙` 형태로 계층 구조를 명시
4. **BAD/GOOD은 최소 2줄**: 제네릭 패턴으로 압축 가능 — 도메인 특화보다 핵심 메커니즘 전달이 중요
5. **체크리스트가 C5를 올림**: 산재된 정보를 1곳에 모은 체크리스트가 completeness 향상
6. **교차 평가 필수**: Opus와 Codex가 C3/C6에서 2.5점 차이 → 양쪽 기준을 동시 충족해야 안전
