# Design: `ricks-voice` — Personal Voice & Writing Style Skill

**Date:** 2026-06-10
**Status:** Approved
**Scope:** Personal skill at `~/.claude/skills/ricks-voice/`, available across all projects.

## Purpose

A skill that captures Rick's voice and writing style for long-form technical writing (essays, whitepapers, technical blog posts), so Claude can draft, rewrite, and review prose that sounds like Rick rather than like an AI.

## Decisions Made

- **Voice corpus:** Rick's external, unassisted writing (blog posts, newsletters, docs) — not the AI-assisted essays in this repo. Repo essays may serve as format/register examples only.
- **Jobs:** all three — draft in voice, rewrite to voice, review against voice.
- **Register:** long-form technical only. One register, sharper skill. Other registers are out of scope for v1.
- **Approach:** hybrid — distilled rules in `SKILL.md` plus curated annotated exemplars in `references/`. Chosen over rules-only (loses tacit rhythm/register) and exemplar-heavy few-shot (token-hungry; imitates content and opinions, not just style; nothing inspectable to debug).
- **Location:** personal skill (`~/.claude/skills/ricks-voice/`), not project-level.

## Structure

```
~/.claude/skills/ricks-voice/
├── SKILL.md                  # frontmatter + voice rules + mode instructions
└── references/
    ├── voice-examples.md     # 5–8 curated excerpts, each annotated with why it's characteristic
    └── anti-patterns.md      # AI-isms and not-Rick patterns, with before/after pairs
```

`SKILL.md` stays under ~150 lines so it is cheap to load on every writing task. References load only when the active mode needs them. The frontmatter description triggers on long-form technical writing tasks: drafting, rewriting, or reviewing essays, whitepapers, and technical blog posts.

## Voice Definition (heart of SKILL.md)

Explicit, checkable rules distilled from the corpus, grouped as:

1. **Stance** — how Rick positions himself relative to the reader (e.g., practitioner-to-practitioner; opinionated but shows the trade-offs).
2. **Rhythm & structure** — sentence-length patterns, paragraph shape, how sections open and close.
3. **Vocabulary & register** — words Rick reaches for, words he would never use, British English, how technical to go before explaining.
4. **Characteristic moves** — recurring rhetorical patterns (e.g., plant an insight early, harvest it later).
5. **Hard nevers** — banned list: generic AI-isms ("delve", "it's worth noting", em-dash overuse, hedge-stacking) plus Rick's personal nevers.

**Quality bar:** every rule must be checkable against a draft. Vague rules ("be engaging") are banned from the skill itself.

## Operating Modes

- **Draft** — read voice rules + `voice-examples.md` before writing anything; write in voice from the first sentence.
- **Rewrite** — two passes over existing text: strip (against `anti-patterns.md`), then re-voice (per the rules). Preserve the argument; change only the prose.
- **Review** — do not touch the text. Produce a list of specific sentences that fail specific rules, quoting the rule each time. This rubric doubles as the self-check the other two modes run before finishing.

## Build Process (extraction session)

1. Rick provides 3–6 pieces of unassisted external writing (a few thousand words total) — as a folder of files or pasted text.
2. Claude analyses them and proposes voice rules back **as claims to confirm or correct** ("you tend to open sections with a concrete scene, not a definition — true?").
3. Corrections are encoded, not just confirmations — what Claude gets wrong about the voice is the highest-value signal.
4. Validation: Claude rewrites a paragraph of neutral/AI-ish prose using the skill; Rick judges whether it sounds like him; iterate until it does.

## Maintenance Loop

A short section in `SKILL.md` instructs future sessions: when Rick corrects a draft's style, propose a one-line amendment to the voice rules before moving on. The skill accumulates taste rather than fossilising at version one.

## Out of Scope (v1)

- Other registers (social posts, newsletters, docs, email).
- Automated corpus ingestion or fine-tuning; this is a prompt-level skill.
- Capturing Rick's *opinions* or positions — the skill encodes style, not stances on topics.

## Success Criteria

- Given an AI-ish paragraph, rewrite mode produces prose Rick reads as "sounds like me".
- Review mode flags style failures by quoting specific rules, with no vague feedback.
- The skill triggers on long-form writing tasks without being explicitly named.
- `SKILL.md` remains under ~150 lines after the extraction session.
