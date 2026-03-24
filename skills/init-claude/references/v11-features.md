# v1.1 Features — 상세 사양

## Quick 모드 (5분 퀵스타트)

`init-claude --quick` 또는 `init-claude quick`:
1. 기술 스택 자동 감지 (10초)
2. 사용자에게 3가지 질문:
   - "Claude가 반복적으로 틀리는 패턴이 있나요?" → trap 후보
   - "절대 수정하면 안 되는 파일/설정이 있나요?" → DO NOT MODIFY
   - "프로젝트 고유 컨벤션이 있나요?" → rules 후보
3. 답변 기반으로 최소 문서 세트 즉시 생성 (에이전트 호출 없음):
   - CLAUDE.md (≤20 lines): 구조 + 명령어 + AGENTS.md 포인터
   - AGENTS.md (≤30 lines): 사용자가 제공한 trap만 (0건이면 empty-state). 사용자 제공 trap은 `unverified` 상태로 생성
   - .claude/CLAUDE.md (≤10 lines): 규칙 인덱스
4. 예상 소요: **5분 이내**, 에이전트 호출 **0회**
5. 이후 `init-claude` (standard)로 업그레이드 가능

## Trap Evidence

모든 trap에 **증거 필드** 필수:

```markdown
## N. [Trap 제목]
> evidence: `src/api/common/fetch.ts:42` — 실제 코드에서 확인된 패턴
> verified_at: abc1234 (2026-03-24)
> sentinel: `src/api/common/fetch.ts`
[설명 + BAD/GOOD]
```

- Trap Hunter 출력 형식: `name, consequence, bad_code, good_code, why_hidden, evidence, sentinel`
- evidence 없는 trap은 **unverified** 표시 → 사용자 검증 전까지 초안 상태
- 사용자가 `file:line` 확인 후 evidence를 승인하면 확정

## Before/After 벤치마크

문서 생성 후 자동으로 **검증 프롬프트 세트** 생성:

```markdown
# .claude/rules/trap-verification.md

## Trap #1 검증
- Prompt: "새 API 함수를 작성해주세요" (getEnv 미사용 유도)
- Expected (without docs): process.env.NEXT_PUBLIC_* 직접 참조
- Expected (with docs): getEnv('KEY') 사용

## Trap #N 검증
- Prompt: ...
```

생성 위치: `.claude/rules/trap-verification.md` (path trigger 없음, 수동 실행용)
Phase 3 Evaluator가 생성 담당. 필수 산출물에 포함.

## Staleness Detection

각 trap에 **검증 메타데이터** 추가:

- `verified_at`: trap 검증 시점의 git commit SHA
- `sentinel`: 이 파일이 변경되면 trap 재검증 필요
- 스킬 실행 시 자동 체크: `git diff {verified_at} -- {sentinel}` → 변경 있으면 경고
- 경고 형식: `⚠️ AGENTS.md #N의 sentinel 파일이 변경됨 — 재검증 필요`
- Phase 1 시작 시 기존 AGENTS.md의 metadata를 읽어 staleness 자동 체크
