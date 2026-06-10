# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A writing repository — long-form essays and whitepapers on AI-assisted software engineering. There is no build system, test suite, or linting. The deliverables are the markdown documents themselves.

## Documents

- `harness-engineering.md` — finished essay comparing Claude Code and Codex harness architectures.
- `meta-repos.md` — whitepaper in progress ("The Meta-Repo"). Currently a **scaffold, not prose**: each section contains HTML comments (`<!-- -->`) acting as a writing guide with GOAL / BEATS / EVIDENCE / LEAN annotations. These comments are instructions to the author and must be deleted as sections are drafted into prose. Do not treat scaffold notes as finished content.

## Writing Conventions

These are established in the existing documents and the scaffold's own instructions:

- **British English** spelling ("optimisation", "flavours", "centre").
- **Dual register**: every section should give a leader a skimmable claim (bold sentence or pull quote) and a practitioner a concrete mechanism (command, file, workflow).
- **Quote discipline** (meta-repos.md): at most one short quote per source, under 15 words, in quotation marks with attribution; paraphrase everything else. Track quote usage across the document.
- **Citation honesty**: the cited papers establish premises, not the meta-repo conclusion — keep "this suggests / it follows" framing on inferential joins, and label vendor-sourced claims as such (commercial interest). Honesty beats (what the argument does NOT claim) are deliberate and must not be cut.
- **Lean discipline** (meta-repos.md): the document soft-lands on metastack. Sections marked LEAN: none must stay neutral; the pitch belongs only in §5.
- Target length for the meta-repos whitepaper: ~3–4 pages, with per-section budgets noted in the scaffold.
- **Voice**: drafting/rewriting/reviewing prose in this repo should use the `ricks-voice` personal skill.

## Architecture of meta-repos.md

The argument arc is fixed: monorepo vs polyrepo → polyrepo enterprises in microservices hell → meta-repo as concept → AI SDLC as catalyst → soft landing on metastack. §1 plants the "default behaviors" insight that §3 harvests; §4's evidence ledger (in the closing HTML comment) holds the verified arXiv citations and the rules for using them. Preserve these cross-section callbacks when drafting or editing.
