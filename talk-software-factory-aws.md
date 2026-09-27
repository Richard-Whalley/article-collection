# The Software Factory — AWS Consumer Summit, Exec & Leadership Track

**Audience:** Technology and business leadership, consumer industries
**Format:** Single talk, no demo
**Runtime:** 20 minutes
**Headline argument:** AI coding tools give you speed. A factory gives you speed you can govern. The meta-repo is what makes the factory work at the scale of a system.

---

## 1 — The problem (0:00–2:30)

*[Slide 1: title card. Talk title, name, one line of credential-free context. Dark, plain.]*

*[Slide 2: "Individually impressive. Collectively ungoverned." Visual: a grid of twelve small pull-request tiles, every one with a green tick. Each tile is drawn in a slightly different style: different logging shape, different error handling, different endpoint pattern. Individually each looks fine. As a grid they clearly don't match. Build: the tiles land one at a time with their ticks as I describe the demo moment, then the headline drops in over them as I say the line.]*

Every engineering organisation in this room has AI coding tools in the hands of its developers by now. And I'd guess most of you have seen the same two things.

The first is the demo moment. A developer shows you a function written in seconds, a test suite generated from a description, a refactor that would have taken an afternoon done before the coffee's cold. It is impressive. It's real.

The second thing is quieter and takes longer to show up. A new endpoint that's patterned differently from the rest of the service. A pull request that passes every test and quietly violates three architecture decisions your teams spent months agreeing. Logs that leak customer data because nobody told the tool that was a rule. The tool didn't know. It can't know. Every session starts from zero, with no memory of your standards, your security posture, your architecture, or what the last ten services look like.

So at the level of an individual developer, you get speed. At the level of an organisation, every piece is impressive on its own and the whole doesn't hold together. It's inconsistent. It's hard to audit. It's disconnected from the standards you actually care about. Individually impressive. Collectively ungoverned.

And that's the gap I want to talk about. Not whether these tools are fast. They are. Whether the speed compounds, and whether you can govern it.

---

## 2 — The factory insight (2:30–4:30)

*[Slide 3: "The standard travels with the tooling." Visual: two horizontal pipelines stacked. Top: a familiar CI pipeline, commit → lint → test → build → deploy, with the lint stage highlighted and a caption "nobody remembers to run this; it just runs". Bottom: the agentic equivalent, brief → build → review → verify → deploy, with three labelled inputs feeding into the build stage from above: ADRs, security constraints, testing strategy. Build: the top pipeline first while I talk about CI; the bottom one appears when I say "that is exactly the move we've made"; the three inputs drop in as I name them.]*

The question we started with a little over a year ago was: what would it take to turn that into a factory?

Not factory in the sense of rigid or bureaucratic. Factory in the engineering sense. Opinionated tooling. An ordered process. Quality specification built into the machines themselves rather than held in the operator's memory.

You already run one of these. Think about a CI pipeline. Nobody asks engineers to remember to run the linter. Nobody puts it on a checklist. It's wired in, it runs every time, for everyone, and it enforces the same standard. The standard travels with the tooling.

That is exactly the move we've made for AI-assisted delivery. Your architecture decisions, your security constraints, your testing strategy: instead of documents that a developer might read and a tool will never see, they become behaviour the agent actually exhibits. Contract-first ordering isn't a wiki page — it's the order the build step follows. No customer data in logs isn't a code review comment — it's a constraint injected into every single build.

That's the insight. Everything else I'm going to show you is built on it.

---

## 3 — Where Kiro stops, and where the factory starts (4:30–8:00)

*[Slide 4: the coverage map. Visual: a single horizontal production line with six phases as stations: Foundations, Analyse, Design, Agentic Build, Verify, Operate. Left of the line, small icons for where the business lives: a policy document, a product requirement, a Jira epic. Right of the line, a single "verified, deployable increment" marker. Build in three steps: the bare line first; then a band labelled "Kiro" over Design and Agentic Build when I say "maps onto the middle of our production line"; then a wider band labelled "the harness" spanning all six when I say "but look at the two ends of the line". The two uncovered ends should be visibly the largest part of the picture before the harness band arrives.]*

We've built this on Kiro. The harness runs on Kiro CLI, and the team works through it as a custom workflow in the Kiro IDE. I want to be precise about what Kiro gives you out of the box and what we've had to add, because that's the honest picture of where most organisations are today.

Kiro's core idea is spec-driven development: requirements, design, tasks, in that order, before any code. That's the right idea, and it maps onto the middle of our production line. Specs cover design. The agent covers build. Steering files carry your standing context. Hooks let you automate behaviour at specific moments. That's a genuinely good substrate, and it's why we chose it.

But look at the two ends of the line.

On the left, before the spec, is where the business lives. Somebody has a policy document, a product requirement, a Jira ticket that represents a whole initiative. Getting from there to a specification precise enough to delegate a build from is real work: decomposing into epics and features, generating acceptance criteria at every level, checking the spec for coverage and coherence before anyone designs anything. Kiro doesn't do that for you. The harness does. A skill that elaborates business requirements into acceptance criteria. A scorecard that checks whether a spec is ready to build from. A stage gate where the team agrees shared understanding before design begins.

Testing moves left with it too. The functional and non-functional test scenarios are derived from those acceptance criteria before a line of code exists. Not written after the build to see what it does. Written before, to say what it must do.

*[Slide 5: "One quality specification, running in both directions." Visual: a single artefact in the centre labelled "acceptance criteria", drawn as one document, not two. An arrow running right into "the build" captioned "governs what gets built". An arrow running from the same document down and round into "the test suites" captioned "verifies what was produced". The point is that both arrows leave the same box. Optional small inset: the same acceptance criterion shown twice, once as a build instruction and once as a test scenario, word for word.]*

The acceptance criteria written in Analyse are the same artefact that drives the build, and the same artefact the test suites verify against. One quality specification, running in both directions: governing what gets built, and verifying what was produced. Precision at that stage is what makes confident delegation possible later. Vagueness there propagates forward as defects.

On the right, after the build, Kiro hands you code. The harness runs the full test suites against it, functional and non-functional. It remediates defects inside a governed loop. Then it runs a formal verification pass across correctness, alignment, trust and risk before anything is cleared to deploy.

So the short version for this room: Kiro gets you from a spec to working code. The factory gets you from a business requirement to a verified, deployable increment. And there's one more piece that makes that hold across a whole system rather than one service at a time. That's what I want to spend most of my time on.

---

## 4 — The meta-repo: the system is the unit (8:00–15:30)

### The inversion (8:00–9:30)

*[Slide 6: "The usual model" / "The meta-repo model." Visual: two columns. Left: five service boxes, each with its own repo icon and pipeline icon underneath, spaced apart. The lines between them are dashed and faint, annotated with where the coordination actually lives: "Confluence", "someone's head", "design doc, six months stale". Right: one large outer box labelled "meta-repo" containing the same five services as smaller, uniform boxes with solid contract lines between them. Along the top edge of the outer box, its contents as a strip: harness, organisation context, ADRs, NFRs, service templates, cross-cutting concerns, factory plan. Build: left column first, right column when I say "we inverted it".]*

Most engineering organisations treat the microservice as the primary unit. Each service is independent: owned by a team, with its own repository and its own pipeline. The macro tends to be informal — how those services compose into a system, what the contracts between them are, what the whole thing is supposed to do. It lives in Confluence, in people's heads, in the design document that was accurate six months ago. Services are independent. Their coordination is emergent.

That was always a weakness. With autonomous build agents, it becomes the weakness. Because an agent building one service has no idea what the other four are doing unless somebody tells it, precisely, every time.

We inverted it. The meta-repo is the primary unit. Services are composable components of it. The meta-repo holds the harness itself. It holds the organisation's context: the ADRs, the non-functional requirements, the service templates, the cross-cutting concerns. And for each increment it holds the factory plan — the macro-level implementation plan for the whole system.

I want to give you three concrete benefits of that inversion, because each one maps to something you're probably being asked about right now.

### Benefit one: context management (9:30–11:30)

*[Slide 7: "The agent's scarce resource is context." Visual: the factory plan as a single source document at the top, with its contents listed: system acceptance criteria, service contracts, event schemas, dependency graph, deployment order. From it, arrows fan out to five service branches, each receiving a per-service plan drawn as a slice of the same document, same colour, same shape. Contrast panel at the bottom left, small: a developer with a clipboard captioned "whatever they remembered to paste in". Build: the source document and its contents first; the fan-out on "from that one plan, a per-service plan is derived".]*

The scarce resource for an AI agent is not intelligence. It's context. What does it know about your world at the moment it starts work? Left to individual developers, the answer is: whatever they remembered to paste in. Which varies by developer, by day, by how much of a hurry they were in.

In the meta-repo model, the whole picture lives in one place. The system-level acceptance criteria. The contracts between services. The shared event schemas. The dependency graph. The deployment order. From that one plan, a per-service plan is derived and pushed to each service's branch. When an agent builds a service, it receives a precise, complete brief derived from the same source of truth as every other service in the increment. It knows what to build, what contracts to honour, and what its place in the system is.

That's the difference between an agent that is fast and an agent that is fast and right. The context problem isn't solved by a bigger model. It's solved by having a place where the whole system is described, and a mechanism that delivers the right fragment of it to every build.

### Benefit two: control of change and deployment (11:30–13:30)

*[Slide 8: "Change once. Propagate everywhere." Visual: an event schema card with one new field highlighted. Two halves. Left, the usual model: the field lands in four repositories at four different points on a timeline, with a "drift" marker where the four timelines fail to align, and a small "prod" flag where the drift is discovered. Right, the meta-repo model: the field changes once in the factory plan and propagates down four arrows to four services in a single step. Below that, a numbered deployment sequence, 1 to 5, with a stage-gate marker before it. Build: left half first; right half on "in the meta-repo model, that change happens once".]*

The second benefit is control.

Take the most common source of pain in a distributed system: a shared contract changes. A new field on an event that four services consume. In the usual model, that change is negotiated across four teams and lands in four repositories at four different times. You reconcile the drift afterwards. Usually in production.

In the meta-repo model, that change happens once, at the factory level. The factory plan reflects it. The per-service plans propagate it. Each service's implementation is derived from the same updated source. You're not reconciling four separately evolved schemas. You changed one thing and the system followed.

The same holds for deployment. The factory plan carries the deployment order and the cross-service dependencies. A feature that spans five services ships as one coordinated thing in a known sequence with a stage gate before it. Not five things that you hope arrive in the right order.

And the same holds for new services. When you add a service, it starts from a template in the meta-repo. Not from scratch, not from whichever existing service someone decided to copy. It inherits the patterns, the cross-cutting concerns, the hook configuration, the testing setup. It can't diverge on day one, which is when most divergence starts.

### Benefit three: governance and traceability (13:30–15:30)

*[Slide 9: "Every merged change is a breadcrumb." Visual: a real-looking pull request description, with the reference line to its factory plan highlighted. From that line, a chain of linked breadcrumbs runs left across the slide: PR → factory plan → stage gate → acceptance criteria → business requirement. Each link is a clickable-looking reference rather than a box in a diagram. Below, small: a standard (an ADR) changed once in the meta-repo, with a single arrow to "next build on every service". Build: the PR first, then the chain link by link as I say "to the acceptance criteria… to the stage gate… to the plan… to the business requirement".]*

The third benefit is the one your risk and audit functions will care about.

Every pull request the factory raises carries a reference back to the factory plan that drove it. Which means every change in production can be walked back: to the acceptance criteria that specified it, to the stage gate that passed it, to the plan that coordinated it across services, to the business requirement it came from. That isn't a report you compile after the fact. It's a property of how the work was produced.

And the standards themselves are governed the same way. The ADRs, the non-functional requirements, the constraints that get injected into every build: they live in the meta-repo, versioned, reviewed, and applied uniformly. When you change a standard, you change it once, and the next build on every service follows it. That's what it means for a standard to be enforced rather than published.

*[Slide 10: "The system is the unit. Services are instances of a pattern." Visual: the meta-repo as one outer frame. Inside it, a single service template at the top and five services beneath it, each visibly stamped from the same template, with the factory plan drawn as the thing composing them. This is a summary slide: reuse the shapes from slides 6 to 9 so it reads as the same picture completed, not a new one.]*

That's what makes this a factory rather than a collection of services that happen to talk to each other. The operational unit is the meta-repo. The services are instances of a pattern, composed according to a plan. The whole picture is held in one place.

---

## 5 — Bounded autonomy (15:30–17:30)

*[Slide 11: bounded autonomy. Visual: the production line from slide 4 again, now with three things marked on it. First, stage gates drawn as inspection points between Analyse, Design and Build, each with its checklist: requirements against acceptance criteria, acceptance criteria against design, design against plans. Second, the remediation envelope drawn as a loop on the Build station with a counter reading 1, 2, 3 and an exit arrow to a human icon captioned "stop, report, escalate; do not push". Either side of that loop, two small failure-mode panels: "always ask" as an engineer buried under approval prompts, captioned "yes ops"; "run until it works" as a spinning meter with a rising token count. Third, a single human tick immediately before Deploy, captioned "once, at shipping". Build: the three marks appear as I say "stage gates", "a defined envelope", "one confirmation gate"; the two failure-mode panels appear when I say "that number matters".]*

I've used the word "delegate" a lot. So let me be clear about what makes delegation safe, because I know that's the question underneath all of this.

Three things.

Stage gates. Before a build is delegated, the design package is checked for consistency: requirements against acceptance criteria, acceptance criteria against design, design against the implementation plans. Drift gets corrected upstream, so you've verified the inputs are coherent before you hand over. The gates aren't overhead. They're what lets you delegate in between them.

A defined envelope. During the build, no agent reviews its own work; independent reviewers check it against the spec and against the engineering standards. When tests fail, the machine gets three attempts to remediate. If the third fails, it stops. It reports exactly what failed. It does not push. It escalates to a human rather than trying harder.

That number matters because of what it rules out at either end. "Always ask" sounds safe, and it isn't. An engineer approving every step of an autonomous build stops reading by the twentieth prompt. Approval becomes a reflex — yes ops — and the quality of the decision degrades precisely because a human is nominally making it. "Run until it works" fails the other way: an agent that never gives up will spin for hours, burning time and tokens on a problem it has already shown it can't solve. Three attempts, then escalate. The engineer isn't babysitting the build, and the build isn't running unsupervised into the ground.

One confirmation gate. Before anything ships, a human sees the full summary of what was built and confirms. Not before every task, which would make autonomy meaningless. At the moment of shipping, which is where accountability actually sits.

That's the machine. Autonomous where autonomy is safe. Gated where judgement is required. And the quality specification holds even when nobody is watching.

---

## 6 — Where it stands, and what it means for you (17:30–20:00)

*[Slide 12: "Operational today." Visual: two large figures side by side, "3 months" struck through and "4 weeks" beside it, captioned "financial services proof of value; client's own estimate against actual". Beneath, one line: "programme extended to build the whole system". Under that, the generic left-to-right flow small: policy document → product requirements → acceptance criteria → factory plan → per-service builds → verified system, with one unbroken traceability line beneath it. Bottom corner, small: "also running in UK healthcare". No client names, logos or system descriptions. Build: the two figures first as I say "three months" and "four weeks"; the extension line on "has since been extended".]*

This is operational on live engagements today. The one I can say most about is a financial services client. They scoped the proof of value at around three months of work. We delivered it in four weeks. Policy document and product requirements in; a working, verified system out, with full traceability from the business requirement to the running service. That programme has since been extended to build the whole system. The same shape is running in UK healthcare. The core loop is solid. The shift-left testing, the verification layer, the multi-service coordination through the meta-repo: working.

*[Slide 13: "Do your standards exist in a form an agent can act on?" Visual: two columns under the question. Left: "documents a developer might read": a wiki page, a Confluence space, a PDF of architecture principles, a slide deck of security policy. Right: "behaviour an agent exhibits": the same four standards as a steering file, an injected constraint, a hook, a test scenario. The left column should look like every organisation's intranet. Build: question first, columns when I ask "or do they exist in documents".]*

So let me leave you with the question I'd ask of your own organisation.

You've bought the tools. Your developers are using them. The speed is real. The question is whether it compounds. And that comes down to one thing: do your engineering standards, your architecture decisions, your security constraints, exist in a form an agent can act on? Or do they exist in documents that a developer might read and a tool will never see?

If it's the second, you have what most organisations have today. I opened on that line and I'll close on it: individually impressive, collectively ungoverned. The move to a factory isn't a bigger model or a better prompt. It's putting the quality specification into the machinery. It's making the system the unit of work rather than the service. It's delegating inside gates you've already verified.

*[Slide 14: "Write rules that say yes." Visual: two concentric loops. Inner loop, small and fast: build → test → verify → repeat, with everything from this talk inside it. Outer loop, large and slow: the path to production, with three gates marked as calendar icons: change advisory board, architecture review, threat modelling, each captioned with a wait ("next slot: three weeks"). Build: as I say "what you need are rules that let you say yes", the three calendar icons transform into three rule cards written as conditions: "IF threat model attached AND ADRs honoured AND non-functional tests pass THEN deploy". The outer loop shrinks to match the inner one. Final state: both loops the same size, with the stage-gate marker from slide 11 visible on the outer loop so the callback lands.]*

One more thing, because it's the beat that gets missed. Everything I've described speeds up the inner loop: build, test, verify, repeat. It does nothing for the outer loop, which is what it takes to get that increment into production. If a verified increment then sits waiting for the next change advisory board, or an architecture review, or a threat modelling session before it can deploy, you've burnt a lot of the gain you just made. Those gates exist for good reasons. The problem is how they're written. Most of them today are meetings that tell you what you haven't done. What you need are rules that let you say yes: if the threat model is attached and the ADRs are honoured and the non-functional tests pass, this can deploy. Written down and checked by the machine, the same as everything else. The stage gates I described earlier are exactly this shape. It's the standard travelling with the tooling one more time — applied to governance rather than code.

*[Slide 15: "Two things to do when you get back." Visual: text only, two numbered lines. 1. Pick your three most important engineering standards and ask where they live. 2. Pick your slowest gate to production and ask what it would take to write it as a rule that says yes. Underneath, small, the opening line one last time: "Individually impressive. Collectively ungoverned." This stays up through the sign-off and into questions.]*

So, two things to do when you get back. Pick your three most important engineering standards and ask where they live. If the answer is a wiki, no agent has ever read them. Then pick your slowest gate to production and ask what it would take to write it as a rule that says yes. That's the gap we built the factory to close.

I'm happy to talk about any of it.
