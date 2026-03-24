# init-claude

> Claude Code 프로젝트 문서 품질을 9.0/10+ 수준으로 최적화하는 스킬 플러그인.

## 설치

```bash
claude plugin marketplace add goddaehee/init-claude-plugin
claude plugin install init-claude
```

## 무엇을 하나요?

1. **Discovery** — 프로젝트 구조, 코드 패턴, 런타임 trap을 자동 분석
2. **Architecture** — 3계층 문서 초안 생성 (CLAUDE.md → .claude/CLAUDE.md → rules/*.md)
3. **PDCA 평가** — 반복 개선으로 문서 품질 최적화

## 사용법

```
init-claude              # 현재 프로젝트
init-claude lightweight  # 소규모 프로젝트 경량 모드
```

## 지원 스택

- TypeScript / React / Next.js
- Java / Kotlin / Spring Boot
- Python / FastAPI
- Go
- Rust

## 업데이트

```bash
claude plugin update init-claude
```

## 라이선스

MIT
