# ricks-voice Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.
>
> **NOTE:** Tasks 2 and 6 require live interaction with Rick (AskUserQuestion). Inline execution is the better fit for this plan; a subagent cannot run the interview.

**Goal:** Build a personal skill at `~/.claude/skills/ricks-voice/` that lets Claude draft, rewrite, and review long-form technical prose in Rick's voice.

**Architecture:** Hybrid voice skill — distilled, checkable rules in `SKILL.md` (loaded on trigger), plus curated annotated exemplars and an anti-pattern list in `references/` (loaded per mode). Built via an interview that confirms/corrects rule claims pre-extracted from the corpus in `docs/example-text/`.

**Tech Stack:** Markdown skill files only. No code, no build. "Tests" are validation checks: a rewrite trial judged by Rick, and trigger-phrase evals.

**Spec:** `docs/superpowers/specs/2026-06-10-ricks-voice-skill-design.md`

---

## File Structure

| File | Responsibility |
|---|---|
| `~/.claude/skills/ricks-voice/SKILL.md` | Frontmatter trigger + voice rules + 3 mode runbooks + maintenance loop. Under 150 lines. |
| `~/.claude/skills/ricks-voice/references/voice-examples.md` | 6 corpus excerpts, each annotated with the rule it demonstrates. Loaded by Draft mode. |
| `~/.claude/skills/ricks-voice/references/anti-patterns.md` | Not-Rick patterns with before/after pairs. Loaded by Rewrite and Review modes. |

Corpus (read-only input, stays in this repo): `docs/example-text/example1.txt` (mentoring post, rough draft), `example2.txt` (skills 101 blog post, polished), `example3.txt` (bio), `example4.txt` (AI-tooling post), `example5.txt` (talk script).

---

### Task 1: Scaffold the skill directory

**Files:**
- Create: `~/.claude/skills/ricks-voice/SKILL.md` (skeleton)
- Create: `~/.claude/skills/ricks-voice/references/` (directory)

- [ ] **Step 1: Create directories**

Run: `mkdir -p ~/.claude/skills/ricks-voice/references`
Expected: no output, exit 0.

- [ ] **Step 2: Write SKILL.md skeleton**

Write to `~/.claude/skills/ricks-voice/SKILL.md`:

```markdown
---
name: ricks-voice
description: Use when drafting, rewriting, or reviewing long-form technical writing for Rick — essays, whitepapers, blog posts, talk scripts — or when asked to make prose "sound like me" / "in my voice". Not for commit messages, code comments, or API docs.
---

# Rick's Voice

Write long-form technical prose that sounds like Rick. Identify the mode first, then follow its runbook. Every mode ends with the Review rubric as a self-check.

## Modes

- **Draft** (writing new prose): read `references/voice-examples.md` BEFORE writing anything. Write in voice from the first sentence — do not draft neutral and restyle.
- **Rewrite** (existing text → Rick's voice): read `references/anti-patterns.md`. Pass 1: strip every anti-pattern. Pass 2: restyle per the Voice Rules below. Preserve the argument and facts exactly; change only the prose.
- **Review** (critique only): do NOT edit the text. Output a list of failures, each quoting the offending sentence and the specific rule it breaks. No vague feedback ("could be punchier") — every flag cites a rule.

## Voice Rules

(populated by Task 3 from the interview record)

## Maintenance

When Rick corrects the style of something written with this skill, do not just fix the draft: propose a one-line amendment to the Voice Rules (or a new before/after pair for `references/anti-patterns.md`) and ask Rick to confirm before moving on. This skill accumulates taste; never let a correction evaporate.
```

- [ ] **Step 3: Verify the skeleton parses as a skill**

Run: `head -5 ~/.claude/skills/ricks-voice/SKILL.md`
Expected: the YAML frontmatter block exactly as above.

---

### Task 2: Voice-claims interview with Rick

**Files:**
- Create: `docs/superpowers/plans/voice-interview-record.md` (working notes, this repo)

The claims below were extracted from the corpus at plan time. Present them to Rick **as claims to confirm, correct, or kill** — batches of 3–4 via AskUserQuestion, multiSelect where sensible. Record every correction verbatim; corrections outrank confirmations.

- [ ] **Step 1: Present Batch A — Stance claims**

1. **Practitioner-first credibility.** You establish authority through concrete war stories and numbers ("23k requests a second at peak", "40k vehicles", "co-pilot technical preview user way back in October 2021"), never through titles or abstractions.
2. **Opinionated, with receipts.** You commit to positions plainly ("I was wrong.", "The answer is actually alarmingly simple.") and you show the trade-off rather than hedging.
3. **Direct reader address.** You talk *to* the reader, including mid-argument interjections: "Wait. Bear with me.", "If you take one thing away from this post, take this one."
4. **Generous, not gatekeeping.** Mentoring register — you want the reader to be able to do the thing, not to be impressed.

- [ ] **Step 2: Present Batch B — Rhythm & structure claims**

5. **Staccato emphasis.** Long explanatory sentences punctuated by very short ones for landing points: "That's it.", "I was wrong.", "It's MCP. It's spec driven prompting. It's context engineering." (anaphora as a list device).
6. **Question-then-answer signposting.** You structure explanations as self-posed questions: "What is it? … When is it used? … How is it different?"
7. **Plant and harvest.** You open with a frame and explicitly return to it at the close: "I opened with harness engineering, and I'll close there too."
8. **Actionable closers.** Endings give one concrete next step plus an emphatic final line: "Start with one. … That's the thing worth building."
9. **Spoken-word connectives.** "So,", "Firstly… Secondly… Then, finally", "Which brings us to…" — prose that reads aloud well.

- [ ] **Step 3: Present Batch C — Vocabulary & register claims**

10. **British English** throughout ("organisation", "sceptic", "behaviour", "dialling").
11. **Plain words over corporate ones.** "use" not "leverage/utilise"; "the whole game", "played out", "an unholy pace" — colloquial intensifiers are in-voice.
12. **Coined formulations.** You compress ideas into memorable lines: "the description is a classifier prompt", "institutional memory encoded as a file tree", "system design speed run™".
13. **Em-dashes and parentheticals are in-voice** (contrary to generic AI-ism lists) — including self-deprecating asides: "(I'm very approachable honestly 🙏)". Confirm whether emoji are in-voice for polished long-form or only for casual posts.
14. **Numbers stay specific.** "fifteen plus minutes… below five minutes… under one minute" — never "significantly faster".

- [ ] **Step 4: Present Batch D — Hard nevers (proposed)**

15. Never throat-clear: no "In today's rapidly evolving landscape", no "It's worth noting that".
16. Never hedge-stack: no "could potentially perhaps".
17. Never "delve", "moreover", "furthermore", "robust", "seamless", "unlock", "in conclusion".
18. Never end a section on a bullet list — sections land on a prose sentence.
19. Never bullet-wall: bullets carry short enumerable facts only; argument lives in prose.

Ask Rick for additional personal nevers the corpus can't show.

- [ ] **Step 5: Write the interview record**

Write all confirmations, corrections, and additions to `docs/superpowers/plans/voice-interview-record.md` with one line per claim: `[#N] CONFIRMED` / `[#N] CORRECTED: <Rick's wording>` / `[NEW] <addition>`.

- [ ] **Step 6: Commit the record**

```bash
git add docs/superpowers/plans/voice-interview-record.md
git commit -m "record voice-claims interview for ricks-voice skill"
```

---

### Task 3: Write the Voice Rules into SKILL.md

**Files:**
- Modify: `~/.claude/skills/ricks-voice/SKILL.md` (replace the `(populated by Task 3…)` line)

- [ ] **Step 1: Draft the Voice Rules section**

Convert every CONFIRMED/CORRECTED claim from `voice-interview-record.md` into the five-group structure below. Use the corrected wording wherever Rick gave it. Every rule must be checkable against a draft — if a rule can't fail a specific sentence, rewrite it until it can.

```markdown
## Voice Rules

### Stance
(rules from Batch A, as confirmed/corrected)

### Rhythm & structure
(rules from Batch B, as confirmed/corrected)

### Vocabulary & register
(rules from Batch C, as confirmed/corrected)

### Characteristic moves
(coined formulations, plant-and-harvest, question-signposting — as confirmed)

### Hard nevers
(Batch D survivors plus Rick's additions, as a flat banned list)
```

- [ ] **Step 2: Verify length budget**

Run: `wc -l ~/.claude/skills/ricks-voice/SKILL.md`
Expected: under 150 lines. If over, move detail to the reference files — do not delete rules.

---

### Task 4: Write references/voice-examples.md

**Files:**
- Create: `~/.claude/skills/ricks-voice/references/voice-examples.md`

- [ ] **Step 1: Write the file with these six excerpts**

Each excerpt is verbatim from the corpus, followed by a one-line annotation naming the rule it demonstrates. Drop or swap any excerpt whose underlying claim Rick killed in Task 2.

```markdown
# Voice Examples

Read these before drafting. Imitate the *moves*, not the topics.

## 1. Reframe compressed into a rule (example2)
> "The name is an identifier, the description is the trigger. … the description is not documentation for humans — it's a classifier prompt."

Move: coin a formulation the reader can carry away; em-dash pivot into the sharper restatement.

## 2. Self-correction beat (example2)
> "That sounds almost too boring to be interesting, and for a while I dismissed it. I was wrong."

Move: long set-up sentence, then a three-word verdict. Credibility through admitting the earlier position.

## 3. Direct address mid-argument (example1)
> "My example for this would be something like a list app. Wait. Bear with me. I know the to-do app tutorial is played out."

Move: anticipate the reader's objection and answer it conversationally, in staccato.

## 4. Anaphoric list as escalation (example4)
> "It's MCP. It's spec driven prompting. It's context engineering. It's custom rulesets. It's agentic workflows."

Move: repetition instead of a bullet list; rhythm does the emphasis.

## 5. Question-then-answer signposting (example5)
> "What is it? Phonebook aims to be the source of truth… When is it used? Fundamentally, it's there to ensure…"

Move: structure exposition as self-posed questions the reader was about to ask.

## 6. Actionable closer with emphatic last line (example2)
> "Start with one. Semantic commits is a good first skill — small, self-contained, and the feedback loop is immediate. … That's the thing worth building."

Move: end on one concrete action, then a short declarative final sentence that calls back to the piece's frame.
```

- [ ] **Step 2: Verify file exists and is complete**

Run: `grep -c "^## " ~/.claude/skills/ricks-voice/references/voice-examples.md`
Expected: `6` (or the post-interview count).

---

### Task 5: Write references/anti-patterns.md

**Files:**
- Create: `~/.claude/skills/ricks-voice/references/anti-patterns.md`

- [ ] **Step 1: Write the file**

Adjust per Task 2 outcomes (e.g., em-dashes are NOT banned here — they're in-voice; only include what Rick confirmed as not-him).

```markdown
# Anti-Patterns: Things Rick's Prose Never Does

Strip these on sight in Rewrite mode; flag them in Review mode.

## Throat-clearing openers
- ✗ "In today's rapidly evolving software landscape, observability has become increasingly important."
- ✓ "Your system is lying to you, and observability is how you catch it."
Open with the claim, a scene, or a war story. Never with the weather report.

## Announcing instead of saying
- ✗ "It's worth noting that the description drives triggering."
- ✓ "The description drives triggering."

## Hedge-stacking
- ✗ "This could potentially help teams perhaps improve reliability."
- ✓ "This makes teams more reliable. The trade-off is X."
Commit to the claim and name the trade-off instead of hedging it.

## Corporate vocabulary
- ✗ leverage, utilise, robust, seamless, unlock, delve, moreover, furthermore, "in conclusion"
- ✓ use, solid, "works", "and", "so" — plain words, spoken-word connectives.

## Bullet-walls
- ✗ Six symmetric bullets carrying the argument.
- ✓ Argument in prose with rhythm; bullets only for short enumerable facts. Never end a section on a bullet.

## Symmetric blandness
- ✗ Every sentence 15–25 words, every paragraph 3 sentences.
- ✓ Vary hard. Long explanatory sentence, then a three-word landing. That's it.
```

- [ ] **Step 2: Verify**

Run: `grep -c "^## " ~/.claude/skills/ricks-voice/references/anti-patterns.md`
Expected: `6` (or the post-interview count).

---

### Task 6: Validation — the rewrite trial

This is the acceptance test from the spec: "Given an AI-ish paragraph, rewrite mode produces prose Rick reads as 'sounds like me'."

- [ ] **Step 1: Run Rewrite mode on this fixed test paragraph**

Input (do not improve it before the trial):

> "In today's rapidly evolving software landscape, it's worth noting that observability has become increasingly important. Moreover, teams should consider leveraging distributed tracing to gain valuable insights into their systems. By adopting these best practices, organizations can unlock significant benefits and ensure robust, scalable solutions moving forward."

Follow `SKILL.md` Rewrite mode exactly: strip pass against anti-patterns, then re-voice pass against Voice Rules.

- [ ] **Step 2: Run Review mode on the original paragraph as a control**

Expected: Review mode flags at minimum — throat-clearing opener, "it's worth noting", "Moreover", "leveraging", "unlock", "robust", hedge-stack ("should consider"), British English violation ("organizations"). Every flag must quote a rule. If any of these passes unflagged, the corresponding rule is not checkable — fix the rule in SKILL.md, not the rubric.

- [ ] **Step 3: Rick judges the rewrite**

Show Rick the before/after. Ask: "Does the rewrite sound like you? What specifically doesn't?" Each "doesn't sound like me" answer becomes a rule amendment (Voice Rules) or a new before/after pair (anti-patterns.md).

- [ ] **Step 4: Iterate until pass**

Repeat Steps 1–3 with the amended skill until Rick accepts the rewrite. Two to three rounds is normal; if it's not converging by round four, the rules are too vague — return to the interview record and tighten the corrected wordings.

---

### Task 7: Trigger evals and final checks

- [ ] **Step 1: Description trigger check**

For each prompt below, judge against the SKILL.md description alone (fresh eyes — pretend you've never seen the body): would it fire?

SHOULD fire: "draft a blog post about event-driven architecture", "rewrite this section so it sounds like me", "review the style of this essay", "help me write §2 of the meta-repos whitepaper".
Should NOT fire: "write a commit message for this change", "add API docs for this endpoint", "summarise this arXiv paper", "fix the typos in this README".

Expected: 4/4 and 0/4. Any miss → amend the description's trigger phrases, not the body.

- [ ] **Step 2: Length and placeholder scan**

Run: `wc -l ~/.claude/skills/ricks-voice/SKILL.md && grep -rn "populated by Task\|TBD\|TODO" ~/.claude/skills/ricks-voice/`
Expected: under 150 lines; grep finds nothing.

- [ ] **Step 3: Update CLAUDE.md in this repo**

Add one line to the repo's CLAUDE.md under Writing Conventions:

```markdown
- **Voice**: drafting/rewriting/reviewing prose in this repo should use the `ricks-voice` personal skill.
```

- [ ] **Step 4: Commit repo-side changes**

```bash
git add CLAUDE.md docs/superpowers/plans/
git commit -m "wire ricks-voice skill into repo conventions; record voice interview"
```

---

## Self-Review (done at plan time)

- **Spec coverage:** structure → Task 1/4/5; voice definition → Tasks 2–3; three modes → Task 1 skeleton + Task 6 exercises Draft-adjacent rewrite and Review; build process → Task 2 (claims-as-interview) + Task 6 (validation loop); maintenance loop → Task 1 skeleton; success criteria → Tasks 6 (rewrite + review rubric) and 7 (trigger, length). No gaps found.
- **Placeholder scan:** the one intentional deferred section (`Voice Rules`) is gated on live interview output by design, with its exact structure and source (interview record) specified; all other content is written out in full.
- **Consistency:** file paths, claim numbering, and mode names match across tasks.
