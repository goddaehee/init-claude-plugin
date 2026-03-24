# Sample: TypeScript 모노레포 문서 구현 (C1-C7 9.0+)

> 실제 프로젝트에서 init-claude 프로세스를 통해 달성한 문서 세트.
> 프로젝트: Next.js App Router + pnpm 모노레포 + TypeScript + Zustand + ag-grid

## 달성 점수

| 기준 | 점수 | 핵심 |
|------|------|------|
| C1 Line Budget | 10 | always-loaded 147 lines (38+91+18) |
| C2 Non-discoverability | 9 | 8 traps, 전부 BAD/GOOD 포함 |
| C3 Single Source | 9 | 0 중복, CLAUDE.md는 순수 포인터, rules는 `AGENTS.md #N 참조`만 |
| C4 Cross-reference | 9.5 | 일관된 `AGENTS.md #N` 형식, 10개 교차 참조 |
| C5 Completeness | 9 | 5/5 시나리오 PASS |
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
    components.md     156 lines  (conditional: src/app/**, src/components/**)
    stores-utils.md    24 lines  (conditional: src/store/**, src/utils/**)
    testing.md        103 lines  (conditional: *.test.*)
```

## CLAUDE.md (root) — 38 lines

```markdown
# Project Name

프로젝트 한줄 설명.

## 모노레포 구조

- `apps/main-app/` — 메인 애플리케이션 (Next.js App Router)
- `apps/storybook/` — 컴포넌트 개발/문서화
- `packages/` — 공유 라이브러리 (`types`, `ui`, `utils`)

## 핵심 명령어

pnpm dev / build / typecheck / lint

## 치명적 제약사항 → AGENTS.md 참조

AGENTS.md의 모든 trap(#1-#N)은 필수 준수. 모든 코드 변경 시 확인할 것.

추가 제약:
- **UI 문자열은 한국어** (또는 프로젝트 언어)
- 기타 프로젝트 고유 제약

## 수정 금지

- `packages/types/` — 자동생성 파일
- 기타 수정 금지 대상

## 상세 규칙

@.claude/CLAUDE.md — 타입 소스, 금지사항, 도메인별 규칙(path trigger 자동 로드)
```

**설계 원칙:**
- 프로젝트 1줄 설명 → 구조 → 명령어 → 제약사항 포인터 → 수정 금지 → 명시적 import
- AGENTS.md trap 내용을 **절대 인라인으로 요약하지 않음** — 순수 포인터만 (C3 핵심)
- `@import`에 로드되는 내용 설명 추가 (C6 향상)

## AGENTS.md 구조

각 trap: 번호 + 제목 + 1줄 결과 + 최소 2줄 BAD/GOOD (제네릭 패턴으로 압축)

```markdown
# AGENTS.md — 코드에서 추론 불가능한 함정

> 코드/설정을 읽으면 알 수 있는 정보는 여기에 쓰지 않는다.

## 1. [Trap 제목] — [결과 요약]
[1줄 설명]
// BAD — [잘못된 패턴]
// GOOD — [올바른 패턴]

## 2~N. [동일 형식]
```

**설계 원칙:**
- P1(Non-discoverability), P2(BAD/GOOD), P3(Consequence-first) 적용
- BAD/GOOD은 제네릭 패턴으로 압축 (도메인 특화보다 메커니즘 전달이 중요)

## Conditional Rules 구조

```markdown
# .claude/CLAUDE.md — 규칙 인덱스
## 도메인별 규칙 (path trigger로 자동 로드)
- `rules/api-patterns.md` — API 레이어
- `rules/components.md` — 페이지/컴포넌트
- `rules/testing.md` — 테스트
```

각 rules 파일에서 trap 참조 시: `맥락 설명 — AGENTS.md #N` (em-dash canonical format)

## 개선 추이 (예시)

```
Round 1: 8.0  (C3=6.5 — rules에 AGENTS.md BAD/GOOD 중복)
Round 2: 9.0  (C3=9.0 — BAD/GOOD 중복 제거, 참조로 대체)
Round 3: 9.2+ (C3=9.5 — CLAUDE.md 순수 포인터화, @import 명시적 설명)
```

## 핵심 교훈

1. **C3가 가장 까다로움**: CLAUDE.md에서 trap 내용을 인라인 요약하면 중복으로 간주됨. 순수 포인터만 허용
2. **rules에서도 BAD/GOOD 복사 금지**: `AGENTS.md #N 참조`로 대체. rules의 BAD/GOOD은 convention 패턴만
3. **@import에 설명 추가**: `@.claude/CLAUDE.md — 설명` 형태로 계층 구조를 명시
4. **BAD/GOOD은 최소 2줄**: 제네릭 패턴으로 압축 — 메커니즘 전달이 핵심
5. **체크리스트가 C5를 올림**: 산재된 정보를 1곳에 모은 체크리스트가 completeness 향상
6. **교차 평가 필수**: 복수 모델 평가 시 C3/C6에서 편차 발생 → 양쪽 기준 동시 충족 필요
