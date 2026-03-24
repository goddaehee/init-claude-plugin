# init-claude Plugin

Claude Code 프로젝트 문서(CLAUDE.md, AGENTS.md, .claude/rules/*)를 생성/개선하는 스킬.
22개 문서 작성 원칙 기반, 3-5회 적응형 PDCA 평가 루프로 C1-C7 가중 평균 9.0/10+ 달성.

## 사용법

- `init-claude` — 현재 프로젝트에 문서 생성/개선
- `init-claude /path/to/project` — 특정 경로 프로젝트
- `init-claude lightweight` — 소규모 프로젝트용 경량 모드

## 지원 스택

TypeScript/React, Java/Kotlin, Python, Go, Rust (자동 감지)

## 산출물

```
CLAUDE.md              ← 요약 인덱스 (≤50 lines)
AGENTS.md              ← Runtime trap + BAD/GOOD (≤100 lines)
.claude/CLAUDE.md      ← 상세 규칙 (≤30 lines)
.claude/rules/*.md     ← 도메인별 패턴 (path trigger)
```

## 평가 기준 (C1-C7)

| # | 기준 | 가중치 |
|---|------|--------|
| C1 | Line Budget (≤200 lines) | 2 |
| C2 | Non-discoverability (BAD/GOOD) | 2 |
| C3 | Single Source of Truth | 2 |
| C4 | Trap Reference Coverage | 1 |
| C5 | Practical Completeness | 2 |
| C6 | Guide Compliance | 2 |
| C7 | Consistency | 1 |
