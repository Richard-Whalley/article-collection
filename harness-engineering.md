# Harness Engineering: The Hidden Architecture Shaping AI-Assisted Development

## What Is a Harness?

When you use an AI coding tool — Claude Code, Codex, Cursor — you're interacting with two things at once:

1. **The model** — the intelligence that understands your request and generates a response. This is what benchmarks measure and headlines fight over.
2. **The harness** — everything else. Where does the AI execute? What can it access? Does it remember previous sessions? Can it reach your project management tools, your test systems, your design files? When you need five things done at once, does it coordinate them or run each in isolation?

The model determines how smart your AI is. The harness determines how useful it is — how it fits into your work, what it can touch, what it remembers, how it fails, and what happens when you want to switch tools in six months.

**The harness is a performance multiplier, not an optimisation layer.** At the AI Engineer Summit in January 2026, Anthropic presented results from the CORE benchmark (which tests agents' ability to reproduce published scientific results). The same Claude model — identical weights, identical training — scored **78%** inside Claude Code's harness but only **42%** inside Small Agents, a different harness built by another startup. Same brain, different body, nearly double the performance.

## Why This Matters Now

Most people assume all harnesses are roughly the same — that the model is the special ingredient and the surrounding infrastructure is interchangeable. That assumption is wrong, and it's getting more wrong over time.

Claude Code and Codex are not two flavours of the same thing. They embody fundamentally different ideas about how humans and AI should work together, and the gap between their architectures is widening deliberately.

---

## Two Competing Philosophies

### Anthropic's Bet (Claude Code): The Collaborator at Your Desk

Claude Code runs in your actual terminal — your shell, your environment variables, your SSH keys. Anthropic's philosophy is summed up as **"bash is all you need."**

Rather than building dozens of specialised tools with lengthy descriptions, Claude Code gives the agent access to composable Unix primitives (grep, git, npm) and lets it chain them together with pipes. A single line of bash can query a database, filter results, and write them to a file. This is cheaper in tokens and far more flexible than maintaining separate tool integrations.

**Key characteristics:**

- **Full local access** — the agent works on your machine, with your tools, your file system, your credentials
- **Incremental, session-aware work** — a structured progress file and git history act as institutional memory across sessions
- **Sub-agent orchestration** — Claude spawns multiple sub-agents with dedicated context windows, coordinated by a central planner. One builds the API, another builds the front end, a third writes tests, and they can communicate along the way
- **Just-in-time tool loading** — rather than preloading all tool descriptions into context (expensive in tokens), Claude stores tools as files and retrieves them only when needed
- **Human-in-the-loop oversight** — the trust boundary is your entire workstation; safety comes from incrementalism and human review

### OpenAI's Bet (Codex): The Contractor in a Clean Room

Codex runs tasks in isolated cloud containers. Your code is cloned into the container, internet access is disabled by default, and the agent works independently before sliding finished results back to you.

OpenAI's team discovered that early progress was slower than expected — not because Codex couldn't write code, but because **the environment was underspecified.** The agent lacked the structure and feedback mechanisms to make progress toward high-level goals.

Their response was to make the repository the single source of truth for everything — architecture decisions, alignment threads, product principles. Anything not in the repo is invisible to the agent.

**Key characteristics:**

- **Sandboxed execution** — each task runs in its own container; agents can't interfere with each other or cascade failures
- **Repo-as-memory** — institutional knowledge lives in documentation, golden principles, and automated linting encoded directly in the codebase
- **Built-in browser tooling** — Chrome DevTools protocol is wired directly in, giving the agent DOM snapshots, screenshots, and navigation to reproduce and validate UI bugs
- **Per-task observability** — each agent gets its own ephemeral logging and metrics stack, making prompts like "make the service start in under 800ms" into testable acceptance criteria
- **Safety through isolation** — rather than trusting the agent with your whole machine, the environment constrains what the agent can do

---

## Five Points of Divergence

### 1. Execution Philosophy

| | Claude Code (Anthropic) | Codex (OpenAI) |
|---|---|---|
| **Approach** | Unix primitives composed via bash pipes | Purpose-built tools exposed as RPC endpoints |
| **Token cost** | Low — a CLI command is a few tokens | Higher — each specialised tool needs a description in context |
| **Flexibility** | Very high — anything you can do in a terminal | Constrained to what the harness exposes |
| **Example** | GitHub CLI via bash achieves the same as GitHub MCP Server's 38 tools (which consume ~15,000 tokens of descriptions) | Chrome DevTools, Victoria Logs/Metrics, app server endpoints all wired in natively |

### 2. State and Memory

- **Claude Code** makes the **agent** remember. Structured progress files (e.g., `claude-progress.txt`) and feature lists (stored as JSON, not Markdown — the model is less likely to corrupt structured data formats) create a trail any new session can follow. Investment in files like `CLAUDE.md` compounds over time.
- **Codex** makes the **codebase** remember. Architecture decisions, principles, and conventions are encoded as repo documentation. Background Codex tasks scan for deviations from golden patterns and open targeted refactoring PRs — the repo polices itself.

### 3. Context Management

- **Claude Code** manages context through **compaction and delegation**. It automatically summarises older context within a session and spins up parallel sub-agents that each get their own context window. Better suited when a single task needs deep understanding of an entire codebase.
- **Codex** manages context through **isolation**. Each task runs in a clean sandbox with no shared state. Better suited when running many independent tasks in parallel without polluting a central context window.

### 4. Tool Integration

Both tools support MCP (Model Context Protocol), the open standard Anthropic created for connecting agents to external tools — now backed by OpenAI, Google, Microsoft, and governed by the Linux Foundation.

But the integration philosophies differ:

- **Claude Code** uses a **skills system** — tools are stored as markdown files and scripts on the file system. The agent sees only short names and descriptions (~50-100 tokens), loading full definitions only when needed. This is context management as harness design.
- **Codex** uses a **bidirectional JSON-RPC harness** (the Codex App Server) that exposes tools as programmatic endpoints. The agent calls into git, test runners, Chrome DevTools, logs, and metrics via RPC. Deep integration, but assumes the agent is working in a server-mediated cloud environment, not on your machine.

In practice, this difference matters when integrating with enterprise toolchains. Composio's testing team had to build a custom proxy adapter to get Codex working with Figma and Jira MCP servers — the protocol is shared but the implementation depth beneath it differs significantly.

### 5. Multi-Agent Architecture

- **Claude Code** uses **orchestrated collaboration**. A coordinator manages sub-agents, each with a dedicated context window, shared task lists, and dependency tracking. The Explore sub-tool uses a fast, cheap model (Haiku) to process large volumes of code, handing results back to Opus for decision-making.
- **Codex** uses **isolated parallelism**. Each task runs in its own sandbox, coordinating through the codebase itself (typically via git branches that get merged). Safer for autonomous operation — agents can't interfere with each other — but parallelism and delegation aren't as mature.

---

## The Lock-In Nobody Is Pricing

This isn't subscription lock-in. It's lock-in to a **philosophy of how work should happen**, expressed through architecture.

Teams build habits, processes, verification steps, and integration plumbing around whichever harness they choose. That investment compounds every month. Switching harnesses doesn't just mean learning new commands — it means rebuilding the entire stack of accumulated automation from scratch, in an architecture that may not even support the same abstractions.

Consider a typical progression: an engineer starts with a simple custom skill for consistent commits, then adds workspace isolation for parallel work, then a planning-to-execution workflow, then batch operations across features. Six layers of workflow automation, each building on the last, each specific to that harness's skill system, context model, and sub-agent architecture. Moving to a different harness means rebuilding all of that from zero.

Multiply that by every engineer on the team, every project, every accumulated configuration file, every MCP connector deployed.

---

## Practical Implications

**For individual developers:** Understanding the harness you're working within matters more than chasing model benchmarks. The architectural choices your organisation makes will shape what's possible in your day-to-day workflow — whether that's deep codebase exploration with sub-agents or parallelised task execution in sandboxes.

**For engineering leaders:** In most enterprise environments, you're choosing one platform, not mixing and matching. That makes the decision heavier. The question isn't "which tool scores higher on SWE-bench?" — it's "which architectural philosophy matches how our team works?" Full local access with human-in-the-loop oversight suits teams with strong trust models and complex, interconnected codebases. Sandboxed isolation suits teams prioritising autonomous execution, security boundaries, and parallelised workloads. How you handle task routing, verification, and institutional memory should drive the decision — these are process design problems, not procurement problems.

**For non-technical leaders:** Your team isn't asking you to buy a wrench. They're asking you to commit to a workbench — a way of organising the relationship between humans and AI agents that will shape velocity, security posture, hiring, and switching costs for years to come.

---

## The Bottom Line

The question everyone asks — "which model is best?" — has a short shelf life. Models keep improving and any advantage from a given release is temporary.

The question underneath it is more durable: **two of the most important companies in AI are making genuinely different bets about how humans and AI agents should work together, and they believe in those bets so strongly they are training their models to work within those specific harnesses.**

The harness decision is a strategic commitment. The right question isn't "which tool is cheapest?" — it's "which architectural philosophy matches how our team works, and how much does it cost to change our mind?" The answer to the second question is typically: a lot, and it goes up every quarter.
