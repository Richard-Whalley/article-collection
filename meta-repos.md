# Meta Repositories: A Pragmatic Middle Path for the AI-Augmented Enterprise SDLC
<subtitle></subtitle>
<!--
WHITEPAPER SCAFFOLD — "The Meta-Repo" (working title)
=====================================================
This is a WRITING GUIDE, not finished prose. Each section gives you:
  • GOAL — what this section must achieve for the reader
  • BEATS — the narrative/argument beats to hit, in order
  • EVIDENCE — specific stats/quotes/sources you can drop in (all from your research)
  • LEAN — where/how hard to lean toward metastack
HTML comments (<!-- -->) are notes to you and will not render. Delete them as you write.

TARGET LENGTH: ~3–4 pages. Rough budget per section in [brackets].
AUDIENCE: engineering leadership + practitioners (dual register — exec skim + practitioner depth).
ARC: monorepo vs polyrepo  →  polyrepo enterprises in microservices hell  →  meta-repo as concept  →  AI SDLC as catalyst  →  soft landing on metastack.
-->

## TL;DR
<!--
GOAL: Earn the next two minutes. Make a leader nod and a practitioner wince in recognition.
BEATS:
  1. Open in the pain, not the theory. A concrete micro-scene: an engineer (or an AI agent) chasing a single change across four repos — cd, git pull, cd, git pull.
  2. Name the false binary everyone assumes they're stuck with: monorepo OR polyrepo.
  3. Promise the turn: there's a third option that's been hiding in plain sight for a decade, and AI just made it urgent.
LEAN: none yet. Stay neutral — earn trust before selling.
TONE: punchy, present tense, concrete. One idea per sentence.
-->

## 1. The Binary We Inherited — Monorepo vs Polyrepo

Every engineering organisation has had this argument. Monorepo or polyrepo: every line of code in one repository, or a repository per service. It resurfaces in every RFC thread and architecture review; it never quite gets settled.

The monorepo promise is one source of truth. One clone gives you the whole estate. You can grep for every consumer of the API you're about to change; a breaking change lands in the same commit as the fixes to everything it breaks. Versioning is unified. Tooling is shared. Onboarding is `git clone` and you're done.

The polyrepo promise is autonomy. Each team owns its repository and everything attached to it: the pipeline, the deploy cadence, the on-call rota for what it ships. Boundaries are enforced by git itself rather than by convention. If you adopted microservices in the last decade this is the topology you defaulted into — often without making a deliberate choice at all.

The veterans of this debate ended up closer than their titles suggest. Matt Klein — creator of Envoy, and author of the canonical case against monorepos — observes that at scale "a monorepo must solve every problem that a polyrepo must solve".[^klein] Adam Jacob, who co-founded Chef and argued the opposite position, concedes "You can, and will, make either layout work."[^jacob] Opposite positions; the same conclusion. **Topology is not the prize. Tooling and defaults are.**

What a monorepo actually buys you is default behaviour. The engineer changing a shared library sees its consumers because they sit in the same tree. New starters absorb the shape of the system from the directory layout. Nobody maintains a wiki page mapping service to repository because there is nothing to map. None of that is git mechanics; it is what the layout makes easy. Hold that thought — the rest of this paper is about getting those defaults without the migration.

## 2. Microservices Hell

The last decade produced one architectural consensus: break the monolith into independently deployable services. Cloud-native went from contrarian to default, and the numbers say the transition is finished. The CNCF's 2024 survey puts cloud-native adoption at an all-time high of 89% of surveyed organisations; Kubernetes runs in production at 80%, up from 66% a year earlier.[^cncf] The sprawl isn't new either — as far back as 2019, organisations with over a thousand employees were already averaging 184 microservices, with six in ten running fifty or more.[^kong]

Microservices brought a repository topology with them. Independently deployable services invited independently owned repositories: one service, one repo, one pipeline, one team. Polyrepo wasn't a decision so much as a default that rode in on the back of the architecture, and it bought real things — team autonomy, isolated CI/CD, independent release cadence, clear ownership. The pipelines multiplied to match: CI/CD adoption grew 31% year on year, and 60% of organisations now run it for most or all of their applications.[^cncf] **Nobody chose a polyrepo estate of 184 repositories. It was organic.**

The tax comes due the first time a change refuses to fit inside one repository. No single place shows how the system fits together. A cross-cutting change becomes a project-managed event: coordinated PRs across four repos; merges landing in dependency order; deploys sequenced so nothing breaks in the gaps between them. You hold the version compatibility matrix in your head because it isn't written down anywhere. Dependencies drift because nothing forces them to agree; the lint config that was copied into every repo three years ago is now eleven different lint configs. Which repos exist, who owns them, how they call each other — tribal knowledge. A new starter spends their first weeks reconstructing a map nobody can clone.

The human cost is measurable. Atlassian and DX surveyed 2,100-plus developers and engineering leaders in 2024: 69% of developers lose eight or more hours a week to inefficiencies — about 20% of their time — and the top causes they name are technical debt and insufficient documentation.[^atlassian] That figure covers inefficiency broadly, not polyrepo specifically. But read the list back against the last paragraph: fragmented context, undocumented maps, drift. Repo fragmentation is precisely the kind of inefficiency the survey is counting.

The obvious fix is the trap. Consolidate into a monorepo and the unified context comes back — and the teams that did it wrote down what it cost. When Uber moved iOS to a monorepo, on the worst days roughly 10% of commits had to be reverted; keeping mainline green meant building a bespoke merge gate, the Submit Queue, after which mainline success rose to around 99%.[^uber-ios] On Android, Gradle builds didn't scale; switching to Buck took a fresh build from fifteen-plus minutes to under five, and incrementals to under a minute.[^uber-android] At backend scale, their Go monorepo needed SubmitQueue on Bazel to land thousands of commits a day without conflicts.[^uber-go] Three engineering posts; the same lesson each time. The monorepo became usable only after they built specialised infrastructure to survive it.

That's what a migration really is. Not a repo move — a simultaneous rework of CI/CD, build systems, VCS performance, the merge workflow and the muscle memory of every engineer you employ. For an enterprise mid-flight on microservices it's a year you can't spend: expensive, politically fraught, operationally risky. And there's a quiet irony in the poster child: Uber ran monorepos for its mobile apps while operating thousands of microservices behind them. Even Uber was a hybrid. The binary was never real.

So the enterprise is stuck: fragmentation taxes every week, and the rebuild costs a year plus a platform team. What if the choice itself is the mistake? What if you could keep every repo, pipeline and deploy exactly as they are — and still get the unified context and the onboarding back?

## 3. The Third Way: Meta-Repositories

A meta-repo is a thin coordinating workspace. It holds no application code. It holds a manifest that lists N independent repositories, clones each one to a known path, and runs operations across all of them at once. The member repos don't know they're members. Their hosting, their pipelines, their branch protection and their owners are untouched. **You keep every repo exactly as it is and add one small repo on top that knows where the others live.**

This is not new. Google shipped `repo` to hold the several hundred git repositories that make up Android; Zephyr has `west`; ROS has `vcstool`; gitslave predates all of them. The tool that gave the pattern its name is mateodelnorte's `meta`, whose README answers the mono-versus-many question "by saying 'both', with a meta repo!"[^meta] Burke Libbey put it more precisely in 2017: "a Manyrepo woven into a Monorepo".[^libbey] The idea has had a decade of production use in operating systems and robotics. What it hasn't had is a reason for the enterprise to care.

Now go back to what a monorepo actually buys you: default behaviour. A meta-repo manufactures those defaults on top of a polyrepo's operating model. Discovery comes back, because the services sit in one tree at known paths and grep works across all of them; Libbey's observation was that the hierarchy on disk is identical to a monorepo's, so the discoverability is too. Cross-repo tooling comes back, because one command fans out to every member: pull them all, show status for all, run a script in each. Onboarding comes back too. A new starter clones the meta-repo, runs one sync command and has the whole product at known paths in the time it takes to fetch. Which repos exist, who owns them and how they call each other stops being tribal knowledge and becomes a manifest you can clone.

Here is what you don't get, and it matters. There are no atomic cross-repo commits: a change that touches three services is still three PRs and three merges. There is no unified build graph; the meta-repo doesn't know that one service imports a package published by another unless you tell it. Refactor-the-estate-in-one-commit stays a monorepo-only trick. A meta-repo closes the gap in what you can see. It leaves the gap in what you can commit.[^devnews]

For an enterprise that is exactly the right trade. Look back at what a monorepo migration forces you to rework — CI/CD, build systems, release cadence, access control — and notice that a meta-repo touches none of them. The pipelines carry on. The branch protection carries on. The team that owns the payments repo still owns it. And the failure mode of adoption changes register: a monorepo migration that goes wrong breaks production; a meta-repo that goes wrong is a workspace that's out of date. One is an incident. The other is a `git pull`.

Practitioners will want the three terms kept apart. A true monorepo is one repository, one history. A synthetic monorepo — Nx's term — is several repositories stitched into one shared dependency graph by the build tool. A meta-repo, sometimes called a virtual monorepo, holds no application code at all: a manifest, some docs and the scripts that fan out. This paper is about the third. 

## 4. AI Is the Catalyst

The new actor in the estate is the coding agent. Claude Code, Cursor, Copilot and Cline stopped being autocomplete some time ago; they take a ticket, read the code, edit across files and open the PR. And they start every session with nothing: no memory of last sprint, no idea who owns the payments repo, no colleague to ask. An agent sees exactly what the structure makes visible and not one file more. **Every polyrepo pain in the microservices section was always there. Humans papered over it with memory and hallway knowledge. The agent can't.** AI doesn't create the problem. It removes our ability to ignore it.

The first objection is that context windows are enormous now, so load everything. The numbers don't allow it. Yellin, citing the EvoCodeBench measurements, puts the average real-world repository at 1.1 million tokens — one repository, not an estate of 184 of them.[^yellin] His own microservice-generation system shows what happens at the edge: it strips runtime error traces before feeding them back, and for some models has to "further limit the size of the error messages to fit into the context window".[^yellin] That's an agent choosing what not to see, mid-task, in published work. More tokens is not the fix. The agent will always be choosing.

So the question is what it chooses with. Tao and colleagues' survey of repository-level code generation puts long-range dependency modelling first among the field's open challenges: functionality in real software arises from interactions across files, and a model that only sees the file in front of it produces locally correct code that breaks global consistency.[^tao] The survey splits retrieval into graph-based methods, which follow declared structure, and non-graph methods, which rank by similarity. Athale and Vaddina measured the difference. Representing the repository as a knowledge graph of dependencies and hierarchy lifted GPT-4's pass@1 on EvoCodeBench from 20.73% with local-file context to 32.00%; same model, same benchmark, and the only change was structure.[^athale] Give the agent the dependency graph and it reasons. Take it away and it guesses.

And the reasoning is a walk, not a lookup. Ugare and Chandra at Meta define agentic code reasoning as the ability to "navigate files, trace dependencies, and gather context iteratively".[^ugare] Navigate, trace, gather, repeat — and every hop assumes the next dependency is reachable from where the agent is standing. In a polyrepo it usually isn't. The Order service publishes an event; the consumer lives in a repository the agent has never cloned and doesn't know exists. The question isn't whether the agent is smart enough. It's whether the path it has to walk is intact. Repo walls cut the path.

Put that against the estate from earlier. Each service is isolated three ways: structurally, in its own repo; informationally, in its own team's heads; and in its dependencies, which are message contracts and API calls that nothing in the repo declares. Now hand an agent a routine ticket: add a field to Order that flows over the event bus to Fulfilment and Invoicing. It has to discover which services are affected, and that's tribal knowledge. It has to pull three separate histories. It has to reason about consistency across them with no shared structural anchor. It is working blind on all three. What hurts the new starter in week one is exactly what hurts the agent in minute one: the boundaries exist, but nothing makes them explicit.

A meta-repo scoped to a bounded context makes them explicit, and it does the three things the papers say matter. The manifest declares the graph: these six services are Order Processing, and this is where each one lives. That is the hand-built version of the knowledge graph that moved Athale's metric, readable by any agent with a filesystem and no special API. It scopes the window: an agent working Order Processing loads Order's services, not Payments and not Logistics, so less irrelevant code goes in and less gets truncated. And it keeps the walk inside one tree: tracing Order Service to Order Events to Fulfilment is directory traversal, not a jump across repositories the agent can't see. Owen Zanzal, running Claude Code across 35 repositories on exactly this pattern, put the reframe in one line: "You don't need a monorepo. You need a monorepo view."[^zanzal] **Monorepos answered a human-scale discovery problem. The meta-repo answers an agent-scale reasoning problem.** That is why a decade-old pattern is suddenly urgent.

Two honesty notes. None of these papers recommend meta-repos. They establish that agents need structural, traversable context; the meta-repo is the delivery mechanism I'm proposing, an inference from the evidence rather than a finding of it. And a meta-repo makes boundaries visible, not enforced. It helps an agent see and respect the bounded context; it does nothing to stop a bad cross-context call at the git layer. There's a sharper edge to the same point. An agent follows the map you give it without checking, so a stale map is worse than none — as one write-up of the pattern has it, an unmaintained meta-repo becomes "a source of confidently wrong behavior at agent speed".[^devnews] The map has to be maintained like code, because to the agent it is code.

Is any of this measurable outside a paper? One vendor data point, labelled as such. Augment Code, which sells a context engine, reports that exposing a cross-repository dependency graph to Claude Code over MCP improved PR quality by 80% on a 300-PR Elasticsearch benchmark.[^augment] Commercial interest, so read it as direction rather than magnitude. But the direction is the same mechanism Athale measured: structure over similarity, now inside a production agent. A meta-repo pre-structures that graph at the filesystem level, before any engine sees it.

So the enterprise faces two doors. Migrate to a monorepo to give the agent structure, and pay the year and the platform team from the microservices section. Or declare meta-repos over bounded contexts: additive, reversible, every pipeline untouched. The organisations most stuck are precisely the ones that went all-in on microservices and polyrepo between 2015 and 2020 and now want agent-assisted delivery. They don't need to undo that bet. They need tooling built for this moment, and that is the last section.

<!--
LEDGER CORRECTIONS (2026-09-23, from source checks while drafting):
  • Tao et al.: author list is Tao, Li, Qin & Liu. The two scaffold quotes ("may lack structural
    understanding"; "excels at capturing architectural and dependency relationships") could not be
    found in the paper text — Tao is PARAPHRASED ONLY in the prose.
  • Athale & Vaddina: the ">10%" figure is not in the paper. Verified same-model result used instead:
    GPT-4 pass@1 on EvoCodeBench 20.73% (local-file infilling baseline) → 32.00% (graph retrieval);
    best overall 36.36% with Claude 3.5 Sonnet.
  • Augment: verified 80% for Claude Code + Opus 4.5, 300 Elasticsearch PRs / 900 attempts, 6 Feb 2026.
  • "narrows the visibility gap, not the transaction gap" and "confidently wrong behavior at agent
    speed" are BOTH from The Dev Newsletter, "The Meta-Repo Pattern" (30 Mar 2026) — one quote spent
    in §4; §3 paraphrases with citation.
-->
<!--
========================  EVIDENCE LEDGER (paper-native citations)  ========================
Use these as your reference list / footnotes. All verified via arXiv. Keep quotes ≤15 words,
one per source, paraphrase everything else (copyright + credibility).

[1] Yellin, D. M. (2024/2025). "LLM Agents for Generating Microservice-based Applications:
    How Complex is Your Specification?" arXiv:2508.20119 [cs.SE]. https://arxiv.org/abs/2508.20119
    USE FOR: context-window ceiling (Beat 1); the ~1.1M-token average-repo figure; agents
    truncating error traces to fit the window. Single author (IBM). v2 dated 26 Oct 2025.

[2] Tao, Y., Qin, Y., & Liu, Y. (2025). "Retrieval-Augmented Code Generation: A Survey with
    Focus on Repository-Level Approaches." arXiv:2510.04905 [cs.SE]. https://arxiv.org/abs/2510.04905
    (Carnegie Mellon / CUHK / SUSTech; CC BY 4.0; submitted 6 Oct 2025.)
    USE FOR: structure-vs-vector retrieval (Beat 2); the three RLCG failure drivers
    (context-window limits, lack of structural understanding, missing project knowledge).
    NOTE: this is the strongest single citation for the section — a survey, so it carries weight.

[3] Athale, M., & Vaddina, V. (2025). "Knowledge Graph Based Repository-Level Code Generation."
    arXiv:2505.14394 [cs.SE]. https://arxiv.org/abs/2505.14394 (Quantiphi; presented at
    ICSE 2025 / LLM4Code 2025.)
    USE FOR: the quantified >10% improvement from graph/structure-aware retrieval (Beat 2 payoff).

[4] Ugare, S., & Chandra, S. (2026). "Agentic Code Reasoning." arXiv:2603.01896 [cs.SE].
    https://arxiv.org/abs/2603.01896 (Meta.)
    USE FOR: the definition of agentic code reasoning as iterative navigate/trace/gather across
    multiple files (Beat 3). Good provenance (Meta) for an enterprise-leadership audience.

[5] (OPTIONAL) Adnan, B., et al. (2026). "Can AI Agents Generate Microservices? How Far are We?"
    arXiv:2603.09004 [cs.SE]. https://arxiv.org/abs/2603.09004
    USE FOR: corroborating that agent success is CONTEXT-CONDITIONAL — ~65% functional correctness
    when integrating into an existing system, across 144 generations / 3 agents / 4 projects.
    CITE CAREFULLY: the 65% figure is scenario-specific; state the conditions, don't generalise it.

NON-PAPER / VENDOR (label as such if used):
[6] Augment Code — "Context Engine" / MCP results (Feb 2026): large reported Claude Code quality
    improvement when a cross-service dependency graph is exposed as structured context.
    DISCLOSURE: tooling vendor, commercial interest. Directional corroboration only.

QUOTE-DISCIPLINE REMINDER: across the whole section, ≤1 short quote per source, under 15 words,
in quotation marks with attribution. Everything else in your own words. Several of these sources
are paraphrase-only by the time you've used your one quote elsewhere — track it.
============================================================================================

========================  WHAT THIS SECTION DELIBERATELY DOESN'T CLAIM  ====================
(Worth a one-line footnote or a sentence in the honesty beat, to stay bulletproof.)
  • It does NOT claim any paper recommends meta-repos. They establish need for structural,
    traversable context; the meta-repo is your proposed delivery mechanism.
  • It does NOT claim meta-repos ENFORCE boundaries — only that they make them VISIBLE/traversable.
  • It does NOT claim the AI angle is established consensus — it's the document's novel synthesis,
    combining agent-context constraints (papers) with bounded-context workspaces (your thesis).
    That's the strongest candidate for your original contribution — frame it as a reasoned argument,
    confidently, but as argument.
============================================================================================
-->
## 5. Where Metastack Fits [~half page]
<!--
GOAL: The soft-but-real landing. Position metastack as the modern, purpose-built entrant for THIS moment — without pretending it invented the category.
BEATS:
  1. Honest lineage first (this earns the pitch): "Meta-repos aren't new. repo, west, meta walked so this could run." metastack is the latest in a line — but purpose-built for the enterprise + AI-SDLC moment, not retrofitted from a robotics or Android workflow.
  2. What modern/purpose-built means concretely (pull from metastack docs, keep factual): declarative `metastack.yaml` workspace; cross-repo git ops (pull/status across all); `exec` arbitrary commands per repo; `doctor` toolchain verification w/ min-version floors — i.e., onboarding + context + toolchain-readiness in one layer.
  3. Tie each capability back to a pain you already established: discovery+onboarding (§2/§3), agent context layer (§4), zero pipeline rework (§3 honesty beat).
  4. The enterprise framing line: it lets enterprises "move quicker" — adopt in weeks, additive, reversible, no migration risk.
LEAN: this is the lean. But keep it confident-not-desperate: metastack as the pragmatic on-ramp, not a silver bullet. Reiterate it's a deliberate, bounded trade.
-->

## [Close / Call to action — short, ~quarter page]
<!--
GOAL: Land the plane. Leave them with the reframe and one action.
BEATS:
  1. Callback to the opening micro-scene — same engineer/agent, now with a meta-repo. The cd/git-pull death-march is gone.
  2. Restate the thesis in one breath: monorepo context + polyrepo independence, and AI is why it's now urgent not optional.
  3. One concrete next step (pick the register for your audience): "stand up a workspace for your highest-friction product domain this week," or a link to metastack docs, or a staged adoption nudge (start with one product spanning 3–10 repos → add the AI context layer → keep the map alive).
TONE: confident, forward-leaning, short sentences.
-->

<!--
========================  APPENDIX: ASSET CHECKLIST  ========================
Optional things that would lift the whitepaper (drop in as you see fit):

DIAGRAMS (1–2 max for a 3–4 pager):
  • The "three models" diagram: Monorepo (one box, one history) | Polyrepo (N scattered boxes) | Meta-repo (N boxes + a thin coordinating ring/manifest over them). This single visual carries §3.
  • Optional: "what an agent sees" — repo-scoped (1 of 4 lit up) vs meta-repo view (all 4 lit). Carries §4.

PULL-QUOTE CANDIDATES (for marketing pop — these are the lines to enlarge):
  • "You don't need a monorepo. You need a monorepo view."
  • "Repo boundaries act as walls for both humans and AI assistants."
  • "A source of confidently wrong behavior at agent speed."
  • "A Manyrepo woven into a Monorepo." (Libbey, the original framing)

HERO STATS (for callout boxes):
  • 184 microservices avg / 84% adoption (Kong/Vanson Bourne)
  • 69% of devs lose 8+ hrs/week to inefficiency = 20% of their time (Atlassian/DX 2024)
  • Uber: 10% of commits reverted on worst days (the migration-pain stat)

SOURCE-HONESTY NOTE (your credibility insurance):
  Much of the AI-SDLC material is 2025–26 practitioner/vendor writing (Nx, monorepo.tools have
  commercial interests). The rigorous anchors are the "Codified Context" arXiv paper + Google DORA 2025.
  Worth a one-line footnote if this goes to a skeptical technical-leadership audience.

REGISTER REMINDER: every section should give a leader a skimmable claim (bold sentence / pull quote)
AND a practitioner a concrete mechanism (the actual command, file, or workflow). Dual-audience throughout.
=============================================================================
-->

<!-- FOOTNOTE BLOCK — all citations live here as GFM footnotes; no inline links in the prose. -->
[^klein]: Klein, M. "Monorepos: Please don't!" https://medium.com/@mattklein123/monorepos-please-dont-e9a279be011b
[^jacob]: Jacob, A. "Monorepo: please do!" https://medium.com/@adamhjk/monorepo-please-do-3657e08a4b70
[^cncf]: CNCF, 2024 Annual Cloud Native Survey (Linux Foundation Research; fielded autumn 2024). https://www.cncf.io/reports/cncf-annual-survey-2024/
[^kong]: Kong / Vanson Bourne, 2020 Digital Innovation Benchmark (survey fielded August 2019; 200 senior IT leaders, US, 1,000+ employees). https://konghq.com/company/press-room/press-release/2020-digital-innovation-benchmark
[^atlassian]: Atlassian & DX (with Wakefield Research), State of Developer Experience Report 2024 (2,100+ developers and engineering leaders, fielded February 2024). https://www.atlassian.com/software/compass/resources/state-of-developer-2024
[^uber-ios]: Uber Engineering, "Faster Together: Uber Engineering's iOS Monorepo" (2017). https://www.uber.com/blog/ios-monorepo/
[^uber-android]: Uber Engineering, "The Journey to Android Monorepo" (2017). https://eng.uber.com/android-engineering-code-monorepo/
[^uber-go]: Uber Engineering, "Building Uber's Go Monorepo with Bazel" (2020). https://www.uber.com/us/en/blog/go-monorepo-bazel/
[^meta]: mateodelnorte, `meta` README. https://github.com/mateodelnorte/meta
[^libbey]: Libbey, B. "Monorepo, Manyrepo, Metarepo" (10 October 2017). https://notes.burke.libbey.me/metarepo/
[^devnews]: The Dev Newsletter, "The Meta-Repo Pattern" (30 March 2026). https://devnewsletter.com/p/meta-repo-pattern/
[^yellin]: Yellin, D. M. "LLM Agents for Generating Microservice-based Applications: How Complex is Your Specification?" arXiv:2508.20119 (v2, October 2025). The 1.1M-token figure is Yellin citing Li et al., EvoCodeBench (2024). https://arxiv.org/abs/2508.20119
[^tao]: Tao, Y., Li, Y., Qin, Y. & Liu, Y. "Retrieval-Augmented Code Generation: A Survey with Focus on Repository-Level Approaches." arXiv:2510.04905 (October 2025). https://arxiv.org/abs/2510.04905
[^athale]: Athale, M. & Vaddina, V. "Knowledge Graph Based Repository-Level Code Generation." arXiv:2505.14394 (LLM4Code @ ICSE 2025). Table I: GPT-4 pass@1 20.73% (local-file infilling) vs 32.00% (graph-based retrieval); best result 36.36% with Claude 3.5 Sonnet. https://arxiv.org/abs/2505.14394
[^ugare]: Ugare, S. & Chandra, S. "Agentic Code Reasoning." arXiv:2603.01896 (Meta, March 2026). https://arxiv.org/abs/2603.01896
[^zanzal]: Zanzal, O. "The 'Virtual Monorepo' Pattern: How I Gave Claude Code Full-System Context Across 35 Repos" (23 March 2026). https://medium.com/devops-ai/the-virtual-monorepo-pattern-how-i-gave-claude-code-full-system-context-across-35-repos-43b310c97db8
[^augment]: Augment Code, "Augment's Context Engine is now available for any AI coding agent" (6 February 2026). Vendor-reported: 80% improvement for Claude Code + Opus 4.5 on 300 Elasticsearch PRs, 900 attempts. Commercial interest. https://www.augmentcode.com/blog/context-engine-mcp-now-live
