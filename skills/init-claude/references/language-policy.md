# Language Policy for Agent Documentation

## Default: Team's primary language

> Use the team's primary language for documentation prose. Switch to English when cross-tool portability (Copilot/Cursor) is needed.

## Content-Language Matrix

| Content | Language | Example |
|---------|:--------:|---------|
| Headings, explanations, rules | English | `Wrap POST body in { data: {...} }` |
| BAD/GOOD labels + descriptions | English | `// BAD — wrong datasource` |
| Cross-references | English | `— AGENTS.md #7` |
| UI string literals in code | Local | `OsAlert.alert('필수 입력 항목입니다')` |
| Domain proper nouns | Local | `breadcrumb={['브랜드', '목록']}` |

## Why English by Default

### Token Efficiency (~45% savings)
- BPE tokenizers are trained predominantly on English corpora
- Korean/CJK: 1 syllable = 2-3 BPE tokens. English: 1 word = 1-2 tokens
- Same semantic content: Korean ~9,500 tokens vs English ~5,300 tokens (measured across 9 files)

### Constraint Adherence
- LLM instruction-following is most accurate in English (training data distribution)
- AGENTS.md traps are precise technical constraints — precision favors English
- BAD/GOOD pattern effectiveness is higher in English

### Cross-Model Compatibility

| AI Tool | Korean Support | English Support |
|---------|:--------------:|:---------------:|
| Claude | Excellent | Excellent |
| Codex/GPT | Good | Excellent |
| Gemini | Good | Excellent |
| Cursor | Backend-dependent | Excellent |
| Copilot (8K context) | Limited | **Critical** — 8K window makes token efficiency essential |

### Community Standard
- Anthropic CLAUDE.md examples: English
- .cursorrules community: ~95% English
- Windsurf AGENTS.md: English
- GitHub Copilot instructions: English
- AGENTS.md open standard: English

## When to Switch to English

Consider English prose when ANY condition applies:
1. Cross-tool use: Copilot (8K context), Cursor, Windsurf — token budget critical
2. International team: Non-native speakers need to read the docs
3. Open-source/community sharing: Broader audience

For single-model teams (Claude) with large context (≥100K): **team language is fine.** The ~45% token savings is <2% of Claude's context window.

## Migration Guide

When converting existing local-language docs to English:
1. Convert section headings, bullet points, explanatory prose
2. Keep code examples unchanged (code is already English)
3. Keep UI string literals in local language (they represent actual runtime values)
4. Keep domain proper nouns in local language (treat as proper nouns in English text)
5. Estimated effort: ~2-3 hours per project for manual review

## Sources

- Anthropic: Claude performs best in English (support.claude.com)
- OpenAI: API works with many languages, best performance in English (help.openai.com)
- XIFBench: Multilingual instruction-following benchmark shows English advantage (openreview.net)
- Token measurement: js-tiktoken cl100k_base encoder, measured across 9 real project files
