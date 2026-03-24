# Claude Code 문서 작성 원칙 (22개)

> agents-md-guide.html + claude-memory-guide.html에서 추출한 핵심 원칙.
> init-claude 스킬의 모든 에이전트는 이 원칙을 기반으로 문서를 평가/생성한다.

## Content Quality (콘텐츠 품질)

### P1. Non-discoverability Test
**AGENTS.md에 포함할 내용의 유일한 기준**: 코드/설정/빌드 로그를 읽어도 알 수 없는 정보만 기록한다.
컴파일 성공 → 런타임 실패하는 함정(trap)이 핵심 대상.

### P2. BAD/GOOD Code Pattern
모든 trap에는 `// BAD`와 `// GOOD` 코드 블록을 포함한다.
산문 설명보다 코드 예시가 에이전트의 실수 방지에 2-3배 효과적 (ETH Zurich 연구 기반).

### P3. Consequence-first Documentation
각 trap은 "무엇이 잘못되는지" 결과를 먼저 기술한다.
왜 발생하는지(원인)는 그 다음에 간략히 설명.

### P4. Auto-generation Prohibition
AGENTS.md/CLAUDE.md를 자동 생성하지 않는다.
ETH Zurich 연구: 자동 생성 문서는 수동 작성 대비 성능 저하 발생.

### P5. Trap as Code Smell
AGENTS.md의 각 trap은 코드 개선이 필요한 "코드 스멜"로 취급한다.
코드가 개선되면 해당 trap은 제거한다 — AGENTS.md는 줄어드는 것이 이상적.

## Structure (구조)

### P6. Three-tier Hierarchy
```
root CLAUDE.md     → 요약 인덱스 (always-loaded)
.claude/CLAUDE.md  → 상세 규칙 (always-loaded)
.claude/rules/*.md → 도메인별 규칙 (경로 트리거로 conditional-loaded)
AGENTS.md          → runtime trap ONLY (always-loaded)
```

### P7. Token Budget Management
Always-loaded 문서(root CLAUDE.md + .claude/CLAUDE.md + AGENTS.md) 합계 ≤200 lines.
컨텍스트 경쟁: 문서가 길어질수록 에이전트가 중요 정보를 놓칠 확률 증가 (Lost in the Middle 효과).

### P8. Conditional Loading via Path Triggers
`.claude/rules/*.md` 파일에 `path:` frontmatter 설정으로 특정 경로 작업 시에만 로드.
상세 패턴/예시 코드는 rules 파일로 분리하여 토큰 예산 절약.

### P9. Import Directive
root CLAUDE.md에서 `@.claude/CLAUDE.md` 지시자로 하위 파일 import.
계층 구조를 명시적으로 연결.

## Anti-patterns (안티패턴)

### P10. No Content Duplication
동일 정보는 단 하나의 파일에만 존재. 다른 파일에서는 `AGENTS.md #N` 형태로 참조.
중복 → 불일치 → 에이전트 혼란.

### P11. No Discoverable Content in AGENTS.md
`package.json`, `tsconfig.json`, 코드 자체에서 알 수 있는 정보는 AGENTS.md에 넣지 않는다.
이미 알 수 있는 정보를 중복하면 토큰 낭비 + 에이전트 주의 분산.

### P12. No Architecture Overview
프로젝트 아키텍처 설명, 디렉토리 구조 상세 설명은 AGENTS.md에 넣지 않는다.
에이전트는 코드를 직접 탐색할 수 있으므로 이런 정보는 discoverable.

### P13. Anchoring Hazard
문서에 잘못된 정보가 있으면 에이전트가 그 정보에 "앵커링"되어 올바른 코드를 무시한다.
정확성을 보장할 수 없는 내용은 넣지 않는 것이 낫다.

## Maintenance (유지보수)

### P14. Living Document
AGENTS.md는 코드 변경에 따라 trap을 추가/제거하는 살아있는 문서.
리뷰 주기: 코드 리팩토링 후 해당 trap이 여전히 유효한지 확인.

### P15. Shrinking is Good
AGENTS.md가 줄어드는 것은 코드 품질이 개선되었다는 신호.
trap이 0개인 프로젝트가 이상적.

### P16. Version-control with Code
AGENTS.md는 코드와 같은 저장소에서 버전 관리.
코드 변경 PR에 관련 AGENTS.md 변경도 함께 포함.

## Scope (범위)

### P17. Team vs Personal
root CLAUDE.md / AGENTS.md: 팀 전체에 적용 → 저장소에 체크인.
`.claude/CLAUDE.md`: 팀 단위 상세 규칙 → 저장소에 체크인.
`~/.claude/CLAUDE.md`: 개인 설정 → 저장소 밖.

### P18. Human vs LLM Perspective
에이전트에게는 **실패 모드와 제약사항**이 필요하다 (아키텍처 개요가 아니라).
개발자가 유용하다고 느끼는 정보 ≠ 에이전트가 실수 방지에 필요한 정보.

## Memory System (메모리 시스템)

### P19. Six-tier Storage Hierarchy
1. `CLAUDE.md` (root) — always loaded, 팀 공유
2. `.claude/CLAUDE.md` — always loaded, 상세 규칙
3. `.claude/rules/*.md` — conditional loaded, 경로별
4. `AGENTS.md` — always loaded, runtime traps
5. `~/.claude/CLAUDE.md` — always loaded, 개인 설정
6. Auto Memory (`~/.claude/projects/`) — persistent memory across sessions

### P20. 200-line MEMORY.md Limit
`MEMORY.md` 인덱스는 200줄 이후 truncate됨.
메모리 내용은 개별 파일에 작성하고 MEMORY.md에는 포인터만 유지.

### P21. Content Hierarchy Priority
충돌 시 우선순위: project rules > user rules > system instructions.
더 구체적인 규칙이 더 일반적인 규칙을 override.

## Research-backed Metrics (연구 기반 지표)

### P22. Quantitative Targets
- Always-loaded 문서 합계: ≤200 lines
- AGENTS.md trap 수: ≤10개 (이상적으로 ≤5개)
- 파일 간 중복 문장: 0건
- BAD/GOOD 코드 블록 비율: trap당 1쌍 이상
- 에이전트 작업 성공률: 80%+ (재질문 없이)
