# init-claude Skill Evaluation Framework

> 이 문서는 init-claude **스킬 자체**의 품질을 평가하는 기준이다.
> C1-C7은 스킬이 **생산하는 문서**의 품질 기준, S1-S8은 **스킬 자체**의 품질 기준.

## 평가 기준 (S1-S8)

| # | 기준 | 설명 | 가중치 |
|---|------|------|--------|
| S1 | **Executability** | 실제 Claude Code Agent tool API로 즉시 실행 가능한가. pseudocode/가상 API 0건 | 3 |
| S2 | **Agent Prompt Quality** | 각 에이전트 프롬프트가 역할 설명이 아닌 실행 가능 지시인가. 입력/작업/출력 형식 명시 | 2 |
| S3 | **Guide Principle Embedding** | `references/guide-principles.md`의 22개 원칙이 스킬 워크플로우에 반영되었는가 | 2 |
| S4 | **Error Handling** | 에이전트 실패, 빈 결과, 점수 하락 시 폴백 경로가 정의되었는가 | 1 |
| S5 | **Portability** | 하드코딩 절대경로 0건, 기술 스택 자동 감지, 모노레포/단일 프로젝트 모두 지원 | 2 |
| S6 | **Sample Quality** | 참조 구현(references/sample-lfos-ui.md)이 C1-C7 9.0+ 달성 | 1 |
| S7 | **Orchestration Clarity** | Phase 1→2→3 에이전트 spawn→대기→통합→수정 플로우가 단계별 명확 | 2 |
| S8 | **Progressive Disclosure** | SKILL.md ≤300줄, 상세 내용은 references/로 분리 | 1 |

**가중 점수** = (S1×3 + S2×2 + S3×2 + S4×1 + S5×2 + S6×1 + S7×2 + S8×1) / 14

**목표**: 9.0/10 이상

## 채점 가이드

### S1 Executability (가중치 3)
- **10점**: 모든 에이전트 호출이 `Agent(subagent_type=..., model=..., prompt=...)` 형태로 실제 API 사용. `run_in_background` 병렬 패턴 적용
- **7점**: 대부분 실제 API지만 일부 pseudocode 잔존
- **5점**: 절반 이상이 pseudocode
- **0점**: `Agent()` 가상 함수 호출, `for loop` pseudocode 등

### S2 Agent Prompt Quality (가중치 2)
- **10점**: 모든 프롬프트가 "이 파일을 읽고 → 이 기준으로 분석하고 → 이 형식으로 출력하라" 구조
- **7점**: 대부분 실행 가능하나 일부 역할 설명에 그침
- **5점**: "당신은 X 전문가입니다" 수준
- **0점**: 프롬프트 없음 또는 1줄 설명

### S3 Guide Principle Embedding (가중치 2)
- **10점**: 22개 원칙 중 핵심 15개+ 가 워크플로우에 구체적으로 반영
- **7점**: 10-14개 반영
- **5점**: 5-9개 반영
- **0점**: 원칙 참조 없음

### S4 Error Handling (가중치 1)
- **10점**: 에이전트 타임아웃, 빈 결과, 점수 하락, Discovery 실패 모두 폴백 정의
- **7점**: 주요 실패 경로 대부분 커버
- **5점**: 일부만 커버
- **0점**: 에러 경로 미정의

### S5 Portability (가중치 2)
- **10점**: 절대경로 0건, `package.json`/`build.gradle` 등으로 자동 감지, 다국어 프로젝트 지원
- **7점**: 하드코딩 경로 1-2건 또는 특정 스택 가정
- **5점**: 하드코딩 경로 3건+ 또는 TypeScript 전용
- **0점**: 특정 머신 경로 의존

### S6 Sample Quality (가중치 1)
- **10점**: `references/sample-lfos-ui.md`가 C1-C7 9.0+ 점수의 실제 산출물 포함
- **7점**: 샘플 존재하나 일부 기준 미달
- **5점**: 샘플 불완전 또는 구버전
- **0점**: 샘플 없음

### S7 Orchestration Clarity (가중치 2)
- **10점**: Phase별 에이전트 spawn 순서, 결과 전달 방식, 통합 로직이 단계별로 명시
- **7점**: 대략적 흐름은 있으나 결과 전달 방식 불명확
- **5점**: 흐름만 있고 구체적 오케스트레이션 없음
- **0점**: Phase 구분 없음

### S8 Progressive Disclosure (가중치 1)
- **10점**: SKILL.md ≤300줄, 상세 채점 가이드/원칙/샘플이 `references/`에 분리
- **7점**: SKILL.md ≤400줄, 일부 분리
- **5점**: SKILL.md 400-500줄, 분리 없음
- **0점**: 500줄+ 단일 파일

## 평가 프로토콜 (3라운드)

### 라운드 구성

각 라운드에서 3개 에이전트 병렬 실행:

| 에이전트 | 역할 | 모델 |
|---------|------|------|
| `skill-practitioner` | SKILL.md 지시대로 가상 프로젝트에 init-claude 실행 시뮬레이션 | sonnet |
| `skill-critic` | SKILL.md의 API 호출, 프롬프트, 오케스트레이션 정확성 감사 | sonnet |
| `skill-evaluator` | S1-S8 기준으로 0-10 채점 + 개선 권고 | opus |

### 라운드별 초점

| Round | 초점 |
|-------|------|
| 1 | S1 Executability + S7 Orchestration: 실제 Agent tool 호출 가능 여부 검증 |
| 2 | S2 Prompt Quality + S5 Portability: 에이전트 프롬프트 실행성 + 범용성 |
| 3 | S3-S4 + S6 + S8: 원칙 반영, 에러 핸들링, 샘플 품질, 파일 분리 |

### 평가 결과 출력 형식

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔄 SKILL-EVAL Round N/3
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
S1 Executability:      8.0 (pseudocode 2건 잔존)
S2 Prompt Quality:     7.0 (explorer 프롬프트 역할 설명 수준)
S3 Guide Embedding:    6.0 (12/22 원칙 반영)
S4 Error Handling:     5.0 (타임아웃 폴백 미정의)
S5 Portability:        7.0 (하드코딩 경로 1건)
S6 Sample Quality:     9.0
S7 Orchestration:      8.0
S8 Progressive:        9.0

가중 평균: 7.4 / 10.0

개선사항:
- [S1] line 302: Agent() pseudocode → Agent tool 호출로 교체
- [S4] Phase 1 에이전트 타임아웃 시 재시도 로직 추가
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
