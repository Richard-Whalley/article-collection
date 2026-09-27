# The Three Levels of Spec-Driven Development

*Spec-first → spec-anchored → spec-as-source*

"Spec-driven" is not one practice. It's a ladder of ambition, and most teams are standing on the bottom rung without knowing there are two more above them. The taxonomy comes from Birgitta Böckeler at Thoughtworks, published on martinfowler.com in October 2025, and it has already become the field's working vocabulary.

Her observation is the useful part: every spec-driven approach is spec-first, but only some climb higher. What all three levels share is the premise — the specification, not the code, is the primary expression of intent. What separates them is two questions. How long does the spec live? And who is allowed to touch the code?

Hold onto those two questions. They decide everything below.

Treat the three not as rival philosophies but as increasing levels of investment. Each one fixes the failure mode of the one beneath it, and pays for the fix with new discipline.

## The ladder at a glance

|  | Spec-first | Spec-anchored | Spec-as-source |
|---|---|---|---|
| **What you do** | Write a good spec, drive the build with it, then move on | Keep the spec maintained alongside the code as the feature evolves | The spec is the only file a human edits; code is regenerated output |
| **Spec lifespan** | May not outlive the feature | Lives as long as the code | Permanent — it is the source |
| **Who edits code?** | Humans, freely, after generation | Humans, but the spec is updated first when requirements change | Humans never touch code directly |
| **Source of truth** | Code, once shipped | Code and spec, kept in sync via CI | The spec |
| **Failure mode it solves** | Ambiguous prompting — vibe coding | Spec rot — the spec going stale the moment code ships | Drift between spec and code entirely |
| **Its own weakness** | The spec rots immediately | The discipline cost of keeping two things in sync | Weakest feedback loop; regeneration fidelity unproven |
| **Maturity** | Table stakes — where most teams start | The sensible default for durable code | Aspirational; few production teams trust it yet |

## How each level actually works

**Spec-first is the one you already know.** You open a markdown file, describe the feature — inputs, outputs, edge cases, constraints — and hand it to your agent. The spec drives the initial build and captures most of the per-feature reduction in rework. Then the code ships and the spec is quietly abandoned. Contract-style specs have worked this way for years: OpenAPI, Protobuf, JSON Schema. SDD just extends the pattern from the interface to the whole feature. The failure mode it solves is vibe coding — ambiguous prompting that produces plausible, wrong code. Its own weakness is that the spec rots the moment the code lands.

**Spec-anchored keeps the spec alive.** When requirements change, you update the spec first, then the code follows. When the next engineer — or the next agent — needs to understand intent, you point them at the spec, not just the code. The spec is checked into the repo, so CI can enforce it: divergence between spec and code shows up as a failing build instead of silent drift. That's the whole trick. You pay for it in discipline — two artefacts to keep in sync, forever — but durable intent is what you get back. This is the sensible default for code you mean to keep.

**Spec-as-source is the strong position** — Böckeler's term, though some write "spec-as-truth" — and the one most associated with Tessl. Here the spec is the artefact you maintain and the code is its compiled output, regenerated on change. Humans never hand-edit the generated code. In principle this is where the most forward-thinking teams are headed. It also draws the sharpest criticism, and the criticism is fair: it has the weakest feedback loop. If you never read or edit the code, you are trusting the generator completely. Round-trip regeneration fidelity is not yet proven at production scale. The dream is real. The evidence isn't in.

Notice the shape. Each rung buys back a failure mode of the last and charges you for the privilege — first ambiguity, then rot, then drift — until at the top there is nothing left to drift, because there is only one file a human ever touches.

## Who's building at each level

**Kiro (AWS)** is the cleanest embodiment of the lower rungs. It generates three files per feature: `requirements.md` (user stories plus acceptance criteria in EARS "WHEN… THE SYSTEM SHALL…" notation), `design.md` (architecture and sequence diagrams), and `tasks.md` (a trackable implementation checklist). Böckeler assessed it as mostly spec-first — the examples drive a single task, with no clear story for reusing the spec across many tasks over time. A concrete case: a developer building "Kaiord" used Kiro to spec a FIT-to-KRD file converter. The `requirements.md` captured the user story and GIVEN/WHEN/THEN acceptance criteria, became Kiro's guide for the build, and was committed at `.kiro/specs/…/requirements.md`. The spec drove the build. Whether anyone opened it again is the open question.

**GitHub Spec Kit** runs a gated flow — Specify → Plan → Tasks → Implement — and deliberately prefers a small spec per feature over one giant project spec. It sits between spec-first and spec-anchored, and there is active debate about whether it truly reaches the higher rung. It became one of the fastest-growing dev-tooling repos of the period.

**Tessl**, founded by Snyk's Guy Podjarny, is the explicit spec-as-source bet: a framework plus a package-and-spec registry and an evaluations layer, aimed at keeping agents "on the rails" by treating the spec as the durable artefact and the code as generated output.

## The ticket test

If the levels still feel abstract, use a comparison every team already carries in its head: the Jira ticket. Most engineering teams operate at the ticket equivalent of spec-first. A ticket captures intent, gets executed, and is never touched again once it closes. Spec-anchored would mean the ticket stays accurate through every scope change and lives on as a reference. Ticket-as-source — regenerating the implementation from the ticket — is essentially unheard of.

Which is the honest summary of where we are. The aspiration holds at both levels, and at both levels most teams are early. So come back to the two questions I opened with: how long does the spec live, and who is allowed to touch the code. Your answers put you on a rung whether you chose one or not. Most teams are on the first and calling it done. Climb one.
