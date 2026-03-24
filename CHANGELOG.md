# Changelog

## [1.1.0] - 2026-03-24

### Added
- 5분 퀵스타트 모드 (`init-claude --quick`)
- Trap evidence field — 각 trap에 `file:line` 증거 필수
- Before/after 벤치마크 — trap별 검증 프롬프트 자동 생성
- Staleness detection — `verified_at` commit hash + sentinel file 기반

### Changed
- 샘플 파일 익명화 (프로젝트명 제거)
- `disable-model-invocation: true` 추가 — 명시적 `/init-claude`로만 실행

## [1.0.0] - 2026-03-24

### Added
- init-claude 스킬 v1.0.0 출시
- 22개 문서 작성 원칙 (guide-principles.md)
- C1-C7 채점 루브릭 (10/8/5/0 앵커 + 보간 규칙)
- 문서 소유권 모델 (AGENTS.md → trap, rules → action)
- 적응형 PDCA 루프 (3라운드 기본, 최대 5)
- Lightweight 모드 (단일 에이전트, no-MCP)
- 교차 평가 권장 (Codex/Gemini)
- 스킬 자체 평가 기준 (SKILL-EVAL.md, S1-S8)
