---
name: init-claude
description: >
  Initializes or upgrades Claude Code documentation (CLAUDE.md, AGENTS.md, .claude/CLAUDE.md,
  .claude/rules/*.md) for any project using multi-agent team analysis and 3-5 adaptive PDCA
  quality evaluation loops. Based on 22 documentation principles (see references/guide-principles.md).
  Triggers on: "init claude", "init-claude", "claude 문서 세팅", "documentation 초기화",
  "CLAUDE.md 만들어", "AGENTS.md 설정", "프로젝트 문서 세팅".
  NOTE: Different from deepinit — deepinit creates hierarchical AGENTS.md in every directory.
  init-claude optimizes ROOT-level documentation quality with PDCA evaluation loops.
user-invocable: true
disable-model-invocation: true
argument-hint: "[project-path | quick | lightweight]"
---

# init-claude Skill

> Multi-agent team으로 프로젝트를 분석하고 Claude Code 문서 **초안을 생성/개선**한다 (최종 확정은 사용자 검증 후).
> P4 관계: 이 스킬은 "자동 생성 금지" 원칙의 예외가 아니라, **검증 가능한 초안 생성 + 사용자 승인** 워크플로우.
> 22개 원칙(`references/guide-principles.md`) 기준, **3-5회 적응형 PDCA 평가 루프**로 9.0/10+ 달성.
>
> **vs deepinit**: deepinit은 모든 디렉토리에 AGENTS.md 생성. init-claude는 루트 문서 품질 최적화.
>
> **실행 모드**: `standard` (기본, 3에이전트 병렬) | `lightweight` (단일 에이전트, no-MCP, 소규모 프로젝트용)

## 즉시 실행 프로토콜

1. PROJECT_PATH 확인 (인수 없으면 현재 디렉토리)
2. 기술 스택 자동 감지 (아래 참조)
3. Phase 1: Discovery — 3개 Agent tool 호출을 **하나의 메시지에서** 병렬 실행
4. 3개 에이전트 완료 알림 수신 후 Discovery Report 통합
5. Phase 2: Architect — 3계층 문서 초안 생성
6. Phase 3: 3-5 적응형 PDCA — 매 라운드 3개 Agent tool 병렬 호출
7. 최종 리포트 출력

## 기술 스택 자동 감지

Glob tool로 프로젝트 루트 파일을 검색하여 스택 판별:
- `package.json` → Node.js/TypeScript (`pnpm-workspace.yaml` → monorepo)
- `build.gradle` / `pom.xml` → Java/Kotlin
- `Cargo.toml` → Rust
- `pyproject.toml` / `setup.py` → Python
- `go.mod` → Go

감지 결과를 모든 에이전트 프롬프트의 컨텍스트로 전달.

## 문서 소유권 모델

채점 전에 소유권을 확인한다. 소유권이 명확해야 C3/C4/C5가 충돌하지 않는다.

| 파일 | 소유 범위 | 허용 내용 |
|------|----------|----------|
| `AGENTS.md` | trap 설명 + BAD/GOOD | 유일한 trap 소스. 다른 파일에서 복사 금지 |
| `CLAUDE.md` (root) | 구조 + 포인터 | trap은 `AGENTS.md 참조`만. 한줄 제약 요약 허용 |
| `.claude/CLAUDE.md` | 규칙 인덱스 | rules 파일 목록 + 프로젝트 고유 규칙 |
| `.claude/rules/*.md` | 도메인별 action | trap은 `AGENTS.md #N 참조`만. BAD/GOOD 복사 금지 |

## 7가지 평가 기준 + 채점 루브릭

| # | 기준 | 가중치 |
|---|------|--------|
| C1 | **Line Budget** | 2 |
| C2 | **Non-discoverability** | 2 |
| C3 | **Single Source of Truth** | 2 |
| C4 | **Trap Reference Coverage** | 1 |
| C5 | **Practical Completeness** | 2 |
| C6 | **Guide Compliance** | 2 |
| C7 | **Consistency** | 1 |

**채점 보간 규칙**: 앵커(10/8/5/0) 사이 소수점 점수 허용. 보간 기준: 앵커 조건 충족 여부 + 경미한 개선/결함 수. 예: C1이 148 lines면 10, 155 lines면 9 (8 앵커 충족 + 여유 있으므로 +1).

### C1: Line Budget (w2)
`wc -l CLAUDE.md AGENTS.md .claude/CLAUDE.md`의 합계. (guide-principles의 "≤200 lines" 예산 기준)
- **10**: ≤150 lines (여유 있음)
- **8**: 151-200 lines (예산 내)
- **5**: 201-250 lines (초과)
- **0**: >250 lines

### C2: Non-discoverability (w2)
각 trap에 대해 체크리스트 적용: (1)런타임 증상 명시 (2)컴파일/스타트업 미감지 (3)코드/설정에서 발견 불가 (4)BAD 예시 (5)GOOD 예시.
- **10**: ≥90% trap이 5/5 체크 통과, false trap 없음
- **8**: 75-89% 통과, 경미한 표현 문제만
- **5**: 50-74% 통과, 일부 discoverable 항목 혼재
- **0**: <50% 통과 또는 대부분 스타일/discoverable 이슈

### C3: Single Source of Truth (w2)
소유권 모델 기준. 허용/비허용 참조 형식:
- **허용** (중복 아님): `AGENTS.md #7`, `AGENTS.md #7 참조`, `이중 배열 래핑 금지 — AGENTS.md #7` (한줄 맥락 + 포인터)
- **비허용** (중복): trap의 BAD/GOOD 코드 블록 복사, 2줄 이상 trap 원인/결과 설명 반복

채점: "비허용" 항목이 소유 파일 밖에 몇 건인가?
- **10**: 비허용 0건
- **8**: 1-2건 경미한 설명 반복 (BAD/GOOD 복사 없음)
- **5**: BAD/GOOD 코드 블록 1건 이상 복사, 또는 3-5건 설명 반복
- **0**: 5건 초과 또는 섹션 단위 복사

### C4: Trap Reference Coverage (w2→1)
경로 범위(path-scoped) trap만 평가. 전역 trap은 root에서 참조되면 충분.
- **10**: 모든 path-scoped trap이 ≥1 rules 파일에서 참조. 고아 참조 없음
- **8**: 1건 path-scoped trap 미참조
- **5**: 2건+ 미참조 또는 고아 참조 존재
- **0**: 참조 체계 없음

### C5: Practical Completeness (w2)
스택별 5개 시나리오에 대해 "문서만 보고 작업 가능한가?" 시뮬레이션.
- **10**: 5/5 시나리오 PASS (복사 가능한 템플릿 또는 단계별 체크리스트 존재)
- **8**: 4/5 PASS
- **5**: 3/5 PASS
- **0**: ≤2/5 PASS

### C6: Guide Compliance (w2)
3계층 구조 + path frontmatter + `@` import.
- **10**: 3계층, 모든 rules에 `path:`/`paths:` frontmatter, `@.claude/CLAUDE.md` + 설명
- **8**: 구조 정확, 1건 frontmatter 누락 또는 설명 부족
- **5**: 2계층만, 또는 다수 frontmatter 누락
- **0**: 단일 파일 또는 구조 없음

### C7: Consistency (w1)
형식 일관성 + 의미 일관성. 정규 참조 형식: `맥락 설명 — AGENTS.md #N` (em-dash + 번호).
- **10**: 언어/네이밍/BAD-GOOD 형식 통일, 참조 형식 통일, 파일 간 모순 없음
- **8**: 1-2건 형식 불일치 (참조 스타일 혼재 등)
- **5**: 체계적 불일치 (한 파일 전체 영어, 다른 파일 전체 한국어)
- **0**: 일관성 없음

**C5 시나리오** — 감지된 스택에 따라 적용:

| # | TypeScript/React | Python/FastAPI | Go | Rust | Java/Kotlin |
|---|-----------------|----------------|-----|------|-------------|
| 1 | 새 POST API 엔드포인트 | 새 API route 추가 | 새 HTTP handler | 새 API handler | 새 REST controller |
| 2 | 새 페이지 생성 | 새 CLI 커맨드/뷰 | 새 서비스 레이어 | 새 모듈 추가 | 새 서비스 클래스 |
| 3 | 폼 + 유효성 검사 | 스키마 + validation | 구조체 validation | serde + validation | DTO + validation |
| 4 | 타입 찾기 + 사용 | 모델/타입 찾기 | 인터페이스 찾기 | trait/struct 찾기 | 엔티티/DTO 찾기 |
| 5 | 테스트 작성 | 테스트 작성 | 테스트 작성 | 테스트 작성 | 테스트 작성 |

**목표**: 가중 평균 9.0/10+. 계산: (C1×2+C2×2+C3×2+C4×1+C5×2+C6×2+C7×1) / 12

## Lightweight 모드

소규모 프로젝트(파일 <50개) 또는 시간/토큰 제약 시:
- Phase 1: 3개 에이전트 대신 **단일 에이전트**가 구조+패턴+trap을 순차 수집
- Phase 2: opus 에이전트 없이 **오케스트레이터가 직접 작성**
- Phase 3: 3개 에이전트 대신 **단일 Evaluator만** 실행, 최소 2라운드
- notepad MCP 불필요 — 에이전트 반환 텍스트를 직접 사용
- 예상 소요: ~15분, 에이전트 호출 ~5회

Standard 모드가 기본. `lightweight` 인수 또는 프로젝트 크기에 따라 자동 전환.

## Quick 모드 (5분 퀵스타트)

`init-claude --quick`: 에이전트 0회, 3가지 질문만으로 최소 문서 세트 즉시 생성. 상세: `references/v11-features.md`

## v1.1 기능 (Trap Evidence + Benchmark + Staleness)

- **Trap Evidence**: 모든 trap에 `evidence: file:line` + `sentinel` 필수. Trap Hunter 출력에 포함
- **Before/After 벤치마크**: Phase 3 Evaluator가 `trap-verification.md` 자동 생성 (필수 산출물)
- **Staleness Detection**: `verified_at` commit SHA + `git diff` 자동 체크. Phase 1 시작 시 실행

상세 사양: `references/v11-features.md`

## Phase 1: Discovery

**하나의 메시지에서** 3개 Agent tool을 동시 호출한다. 각 에이전트는 `run_in_background: true`로 실행.
Claude Code는 에이전트 완료 시 자동으로 알림을 보내므로 별도 폴링이 필요 없다.

### 에이전트 1 — Structure Explorer (`haiku`)

입력: PROJECT_PATH + 스택. 작업: 빌드 설정, 워크스페이스, 명령어, 수정 금지 파일 수집. 출력: `TECH_STACK`, `MONOREPO`, `COMMANDS`, `DO_NOT_MODIFY`.

### 에이전트 2 — Pattern Explorer

Agent tool 파라미터: `subagent_type="general-purpose"`, `model="haiku"`, `run_in_background=true`

프롬프트 내용 — 입력: PROJECT_PATH + 스택. 작업: API 패턴(실제 파일 2-3개 읽기), 컴포넌트 패턴, 타입 시스템, 언어 규칙. LSP 사용 가능 시 `lsp_document_symbols`로 모듈 구조 파악, `lsp_find_references`로 핵심 함수 사용 패턴 추적 (Glob/Grep보다 정확). LSP 미사용 환경에서는 Glob/Grep으로 동일 정보 수집. 출력: `API_PATTERN`, `COMPONENT_PATTERN`, `TYPE_SYSTEM`, `LANGUAGE_RULES`.

### 에이전트 3 — Trap Hunter

Agent tool 파라미터: `subagent_type="general-purpose"`, `model="sonnet"`, `run_in_background=true`

프롬프트 내용 — "컴파일 성공 → 런타임 실패" 함정 탐색. 포함 조건: (1)컴파일 성공 (2)런타임 실패 (3)코드만으론 발견 불가 (4)디버깅 30분+. 제외: 컴파일러/린터가 잡는 것, 설정 파일에서 발견 가능, 스타일 선호. LSP 사용 가능 시 `lsp_find_references`로 의심 패턴의 호출 범위를 검증하여 trap 후보 정확도 향상 (선택적). 출력: 각 trap별 `name, consequence, bad_code, good_code, why_hidden, evidence(file:line), sentinel(file)`. evidence 없으면 `unverified` 표시. trap 0건도 유효한 결과이며, `trap_count=0`으로 명시 출력.

### 결과 수집 및 저장

3개 에이전트 완료 알림 수신 후, 각 에이전트의 반환 텍스트를 통합하여 notepad에 저장한다.
MCP tool `notepad_write_working` 호출: topic="discovery-report", content=통합 텍스트.
**폴백**: notepad MCP 미사용 환경에서는 에이전트 반환 텍스트를 직접 활용하여 Phase 2로 전달.

### 에러 핸들링

- **에이전트 타임아웃**: 해당 에이전트만 sonnet으로 재실행 (1회 한정)
- **빈 결과**: 유효한 0-trap/0-pattern 결과로 취급 (trap_count=0). Glob/Grep/LSP는 새 trap 발굴용이 아니라 이미 의심된 trap의 근거 검증용으로만 사용. discoverable 정보로 trap을 조작하지 않는다 (P1).
- **전체 실패**: 사용자에게 수동 정보 요청 후 Phase 2 진행

## Phase 2: Architecture

Phase 1에서 notepad에 저장한 Discovery Report를 읽어(`notepad_read` topic="discovery-report") 3계층 문서 초안을 생성한다. Discovery Report 토큰 ≤ 2000이면 직접 작성, 초과하면 opus Agent를 호출한다. **두 경로 모두 아래 필수 산출물 계약을 따른다.**

opus Agent 사용 시 파라미터: `subagent_type="general-purpose"`, `model="opus"`

프롬프트 내용:
- 입력: Discovery Report 전문, 감지된 스택, 참조 원칙(`references/guide-principles.md` P1-P22), EXISTING_FILES=[기존 문서 목록]
- 작업: 아래 구조로 각 파일 초안 생성. AGENTS.md는 trap만 (P1,P11,P12). 파일 간 중복 금지 (P10) — root CLAUDE.md에서 trap 내용을 인라인 요약하지 않고 순수 포인터(`AGENTS.md 참조`)만 사용. root CLAUDE.md에서 `@.claude/CLAUDE.md` import 시 로드 내용 설명 포함 (P9). rules 파일에서 AGENTS.md trap의 BAD/GOOD을 복사하지 않고 `AGENTS.md #N 참조`로 대체. path frontmatter 설정 (P8). EXISTING_FILES에 있는 파일은 Read 후 diff-aware 개선, 없는 파일은 신규 생성.
- 출력: 각 파일의 전체 내용을 Write tool로 생성

**필수 산출물** (모든 실행에서 반드시 생성/갱신):
```
root/
  CLAUDE.md              ← 요약 인덱스 (≤50 lines)
  AGENTS.md              ← Runtime trap + BAD/GOOD (≤100 lines)
  .claude/
    CLAUDE.md            ← 상세 규칙 (≤30 lines)
    rules/*.md           ← 도메인별 패턴 (path trigger)
    rules/trap-verification.md  ← (v1.1) Before/After 벤치마크 프롬프트
```

**AGENTS.md 정책**: trap 0건이어도 항상 생성/유지한다.
- trap ≥ 1: 각 trap을 번호 + BAD/GOOD + `evidence`/`sentinel`/`verified_at`으로 작성
- trap = 0: empty-state 템플릿: `> 현재 검증된 non-discoverable runtime trap 없음. 새 trap은 검증 후만 추가.`
- 파일 존재 = "검토 완료" 시그널이며, 향후 trap 추가 위치를 명시

**주의 원칙**: 이 스킬은 초안을 생성하며, 사용자의 수동 검증이 필수 (P4). 자동 생성된 trap은 반드시 사용자/팀이 검증한 후에만 최종 확정한다. AGENTS.md의 trap은 코드 스멜이다 (P5) — 코드가 개선되면 제거한다. 잘못된 문서는 올바른 코드를 무시하게 만든다 — 앵커링 위험 (P13).

**교차 평가 권장**: 최종 라운드에서 Codex/Gemini 등 외부 모델로 독립 채점 수행. C1은 반드시 `wc -l`로 라인 수 측정 지시. 모델 간 ±1.0 이상 차이나는 기준은 추가 개선 대상.

### 에러 핸들링

- **Architect 실패**: 오케스트레이터가 Discovery Report만으로 직접 작성 (Phase 2는 항상 진행 가능)

## Phase 3: PDCA Evaluation Loop

**최소 3라운드, 9.0 미달 시 최대 5라운드**. 3라운드 후 9.0+ 달성 시 조기 종료 가능. 각 라운드마다 3개 Agent tool을 **하나의 메시지에서** 동시 호출한다.

### 매 라운드 실행 순서

**1단계: 3개 에이전트 동시 호출** (모두 `run_in_background: true`)
Agent tool 파라미터 형식은 Phase 1의 예시와 동일. description, subagent_type, model, run_in_background, prompt 키를 사용.

- **Practitioner** (`subagent_type="general-purpose"`, `model="sonnet"`):
  입력: 문서 경로 전체 목록 + 해당 라운드 C5 시나리오 (스택별 시나리오 테이블에서 선택).
  작업: (1) 각 문서를 Read tool로 읽는다 (2) 시나리오별로 문서만 보고 작업을 시뮬레이션한다 (3) 각 AGENTS.md trap을 회피할 수 있는지 확인한다.
  출력: 시나리오별 PASS/FAIL + 실패 시 원인(어떤 정보가 부족했는지) + 구체적 수정 제안(어떤 파일에 무엇을 추가해야 하는지)

- **Critic** (`subagent_type="general-purpose"`, `model="sonnet"`):
  입력: 문서 경로 전체 목록.
  작업: (1) 각 파일을 Read tool로 읽는다 (2) 파일 간 동일 문장/개념 중복 탐지 — 참조(AGENTS.md #N)와 코드 예시는 중복이 아님 (3) AGENTS.md에 discoverable 콘텐츠가 있는지 확인 (4) always-loaded 파일 라인 수 합산.
  출력: 중복 목록(file:line ↔ file:line), discoverable 항목 목록, 라인 수 보고(always-loaded N줄)

- **Evaluator** (`subagent_type="general-purpose"`, `model="opus"`):
  입력: 문서 경로 전체 목록 + C1-C7 채점 루브릭 전문(각 기준 10/8/5/0점 앵커 포함, 위 '채점 루브릭' 섹션 참조).
  작업: (1) 모든 문서를 Read tool로 읽는다 (2) 각 기준별 근거를 1-2줄로 작성 (3) 가중 평균 계산.
  출력: C1-C7 각 점수(소수점 1자리) + 가중 평균 + Top 3 개선사항(파일:라인 수준 구체성)

**2단계: 완료 알림 수신 대기** — Claude Code가 각 에이전트 완료 시 자동 알림. 3개 모두 완료될 때까지 다른 작업 가능.
Agent tool이 run_in_background=true로 실행되면, Claude Code가 에이전트 완료 시 자동으로 결과를 반환한다. 반환된 텍스트가 오케스트레이터의 대화 컨텍스트에 직접 나타나므로 별도 조회가 불필요하다.

**3단계: 결과 통합 + 교차 검증**
- practitioner FAIL 항목 중 evaluator도 지적 → 우선 수정
- critic 중복 발견 → 즉시 수정
- evaluator 점수 < 8 인 기준 → 해당 영역 집중 수정

**4단계: Edit/Write tool로 파일 수정**

**5단계: 점수 기록 후 다음 라운드 진행**

### 라운드별 초점

| Round | Practitioner 시나리오 | Critic 초점 | Evaluator 초점 |
|-------|---------------------|------------|---------------|
| 1 | C5 시나리오 1-2 | AGENTS.md trap 적정성 | C2 + C5 |
| 2 | C5 시나리오 3-4 | 파일 간 중복 탐지 | C3 + C4 |
| 3 | C5 시나리오 5 + 재검증 | discoverable 콘텐츠 제거 | C1 + C5 |
| 4 | 전체 시나리오 재실행 | 라인 수 최적화 | C1 + C3 |
| 5 | 최종 통합 시뮬레이션 | 참조 무결성, 일관성 | C4 + C6 + C7 |

### 에러 핸들링

- **에이전트 타임아웃**: 해당 역할만 재실행 (1회). 2회 연속 실패 → 건너뛰고 나머지로 진행
- **점수 하락**: 하락 기준의 수정 사항 식별 후 해당 변경만 git revert 또는 수동 롤백
- **5라운드 후 9.0 미달**: 사용자에게 현재 점수 + 병목 보고, 추가 라운드 여부 확인

## 산출물 형식 + 기존 문서 처리

각 라운드 후 C1-C7 스코어카드 출력 (형식은 `references/sample-typescript-monorepo.md` 참조).
**모드별 처리**:
- **기존 파일 있음**: diff-aware로 개선. 기존 trap은 재검증 후 유지/제거.
- **필수 파일 누락**: 필수 산출물 목록에서 누락된 파일은 신규 생성.
- **기존 trap 재검증 후 0건**: AGENTS.md는 유지하되 empty-state 템플릿으로 갱신.
git diff 제공.

**유지보수 원칙** (P14-P16): 문서는 코드와 함께 버전 관리. 코드 리팩토링 시 관련 trap 재검증. AGENTS.md가 줄어드는 것은 좋은 신호 (P15).

## 참조 문서

- `references/guide-principles.md`: 22개 문서 작성 원칙 (P1-P22)
- `references/sample-typescript-monorepo.md`: 실제 프로젝트 적용 사례 (C1-C7 9.0+)
- `SKILL-EVAL.md`: 이 스킬 자체의 품질 평가 기준 (S1-S8)
