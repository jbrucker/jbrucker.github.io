---
title: Approach to Agentic Software Development with Codex and Copilot
---

Prompt to ChatGPT:

> What is your recommended approach to designing and implementing software by creating, using, and managing a coding agent(s) using the Copilot or Codex platforms?

Treat a coding agent like a **junior-to-mid-level software engineer who can work continuously on a Git branch, execute tests and tools, investigate the codebase, and produce a pull request**—while you remain the architect and reviewer.

Both OpenAI Codex and GitHub Copilot are moving strongly in this direction. Codex explicitly supports parallel agent work and reusable Skills, while Copilot supports custom agents, persistent instructions, skills, MCP tools, and cloud-agent workflows. ([OpenAI][1])

ChatGPT's recommended approach is:

## 1. Separate software engineering decisions from implementation

The biggest mistake is to give an agent a vague instruction such as:

> "Implement the customer management module."

Instead, establish a **development process** such as:

```text
                 YOU
                  │
        requirements / architecture
                  │
                  ▼
            ┌──────────┐
            │   PLAN   │
            └────┬─────┘
                 │
          well-defined task
                 │
                 ▼
        ┌─────────────────┐
        │ CODING AGENT    │
        │                 │
        │ inspect         │
        │ implement       │
        │ test            │
        │ investigate     │
        │ commit          │
        └────────┬────────┘
                 │
                 ▼
             PULL REQUEST
                 │
          ┌──────┴──────┐
          ▼             ▼
       automated       human
         tests         review
          │             │
          └──────┬──────┘
                 ▼
               MERGE
```

The important principle is:

**The agent should generally decide *how* to implement something; you should decide *what* should be built and the important architectural constraints.**

---

# 2. Put the project's durable knowledge in the repository

This is probably the most important design decision.

Don't repeatedly tell the agent:

> "We use pytest, don't use inheritance here, database access goes through repositories, etc."

Record those things as persistent project instructions.

For Copilot, GitHub currently supports repository-wide `.github/copilot-instructions.md`, path-specific instructions, `AGENTS.md`, reusable prompt files, and custom agents. ([GitHub Docs][2])

ChatGPT suggests organizing the repository approximately like this:

```text
project/
│
├── .github/
│   ├── copilot-instructions.md
│   ├── instructions/
│   │   ├── python.instructions.md
│   │   ├── frontend.instructions.md
│   │   └── tests.instructions.md
│   └── agents/
│       ├── architect.agent.md
│       ├── implementer.agent.md
│       ├── tester.agent.md
│       └── reviewer.agent.md
│
├── docs/
│   ├── architecture/
│   ├── requirements/
│   ├── design/
│   └── decisions/
│
├── src/
├── tests/
└── ...
```

Do **not** put changing task instructions into the main source tree.

This is closely related to the point you raised previously about keeping AI instructions separate from source code. I think that instinct is correct.

The repository should contain **durable engineering knowledge**, not a transcript of conversations with agents.

---

# 3. Have a very small set of permanent instructions

For example, the project-wide instructions might establish:

### Project

* What the application does
* Major architectural components
* Directory structure
* Supported platforms
* Important external dependencies

### Development

* Language/version
* Build system
* Test framework
* Linting/Type checking
* Formatting
* CI commands

### Architecture

* architectural rules
* dependency direction
* database conventions
* API conventions
* error handling
* logging
* security requirements

### Coding conventions

Only conventions that actually matter.

Don't create a 20-page "AI constitution" containing every conceivable coding preference. Agents perform better with **short, precise, enforceable rules**.

GitHub itself recommends persistent repository instructions containing project structure, coding conventions, and build/test procedures. ([GitHub Docs][3])

---

# 4. Treat requirements as first-class artifacts

I would **not** make the GitHub issue itself the entire specification.

For a nontrivial feature, I'd have something like:

```text
Requirement
    │
    ├── functional requirements
    ├── non-functional requirements
    ├── acceptance criteria
    └── constraints
             │
             ▼
          Design
             │
             ├── architecture
             ├── interfaces
             ├── data model
             └── alternatives / decisions
                     │
                     ▼
                    Task
                     │
                     ▼
                   Agent
```

This is particularly useful with agents because it dramatically reduces ambiguity.

For example, rather than:

> Implement password reset.

give the agent a task such as:

> Implement password-reset token generation and validation.
>
> Constraints:
>
> * Tokens must be single-use.
> * Tokens expire after 30 minutes.
> * Token values must not be stored in plaintext.
> * Existing authentication interfaces must remain unchanged.
>
> Acceptance criteria:
>
> * expired token rejected
> * previously consumed token rejected
> * invalid token rejected
> * successful reset invalidates token
> * tests cover all cases
> * existing authentication tests continue to pass

That's a task an agent can execute very effectively.

---

# 5. Use **one general agent first**, not a swarm

I would resist the temptation to create 15 specialized agents.

Initially, use:

**one primary coding agent + human architect/reviewer.**

The agent should be able to:

1. inspect the repository
2. understand the task
3. formulate an implementation plan
4. modify code
5. run tests
6. investigate failures
7. revise its implementation
8. produce a PR

That already represents a very substantial capability.

Only introduce specialized agents when you discover a recurring problem that benefits from specialization.

For example:

```text
                    Coding Agent
                         │
              ┌──────────┼──────────┐
              │          │          │
              ▼          ▼          ▼
          researcher   tester    security
```

Copilot now explicitly supports custom agents and sub-agent delegation, with each agent able to have its own prompt, tool restrictions, and MCP servers. ([GitHub Docs][4])

Codex similarly supports multi-agent workflows and reusable Skills. ([OpenAI][1])

---

# 6. Specialized agents should represent **roles**, not programming languages

I would prefer:

```text
architect
implementer
test-engineer
security-reviewer
```

over:

```text
python-agent
javascript-agent
database-agent
docker-agent
```

The latter tends to fragment the project knowledge.

The former reflects the actual software-development process.

For example:

### Architect

Can:

* inspect repository
* research existing implementation
* identify architectural implications
* propose alternatives
* produce implementation plan

But ideally **cannot modify production code**.

### Implementer

Can:

* edit code
* create tests
* run build/test tools
* modify documentation

### Test agent

Can:

* inspect implementation
* create tests
* run tests
* identify missing coverage

but preferably does not modify production code.

### Security reviewer

Can:

* inspect code
* inspect dependencies/configuration
* run security tools
* report vulnerabilities

Again, preferably read-only.

This separation makes agent output much easier to reason about.

---

# 7. Give agents progressively greater autonomy

I would use four autonomy levels.

### Level 1 — Advisor

```text
Read repository
       ↓
Explain / recommend
       ↓
Human decides
```

No modifications.

### Level 2 — Implementer

```text
Read
 ↓
Plan
 ↓
Modify branch
 ↓
Run tests
 ↓
Human reviews
```

This should be the normal mode.

### Level 3 — Autonomous PR

```text
Issue
 ↓
Agent investigates
 ↓
implements
 ↓
tests
 ↓
PR
 ↓
human review
```

This is ideal for well-defined work.

GitHub's current cloud-agent workflow is explicitly designed around this model: assign a task, let the agent work on a branch, then review the resulting PR and iterate. ([GitHub Docs][5])

### Level 4 — Autonomous maintenance

Things like:

* dependency updates
* documentation fixes
* test coverage
* lint fixes
* issue triage
* routine refactoring

can eventually be delegated almost entirely.

Codex currently supports scheduled/background work such as issue triage and CI/CD-related tasks. ([OpenAI][1])

---

# 8. Make tests the agent's principal feedback mechanism

This is perhaps the second most important principle.

Don't have:

```text
Human → Agent → Code → Human
```

Have:

```text
                 ┌──────────┐
                 │  AGENT   │
                 └────┬─────┘
                      │
                      ▼
                    CODE
                      │
                      ▼
                    TESTS
                      │
              ┌───────┴───────┐
              │               │
             pass            fail
              │               │
              ▼               ▼
             done          agent fixes
```

The agent should be able to execute:

```text
pytest
mypy
ruff
npm test
npm run lint
docker compose ...
```

whatever applies to the project.

A good agent is not one that produces code that *looks* correct.

It's one that can **produce a change and obtain objective evidence that the change works**.

GitHub likewise recommends automated tests, linting, code scanning and other tooling as additional validation of Copilot's work. ([GitHub Docs][6])

---

# 9. Use Git branches/worktrees aggressively

Every meaningful agent task should have an isolated workspace.

For example:

```text
main
 │
 ├── feature/customer-import
 │
 ├── bugfix/login-timeout
 │
 └── refactor/query-layer
```

An agent should not normally be editing your working copy of `main`.

This has several benefits:

* easy rollback
* easy comparison
* independent agent sessions
* multiple agents can work concurrently
* human can inspect every change
* PR becomes the natural review boundary

This becomes particularly valuable with Codex because its current workflow explicitly provides isolated worktrees/cloud environments for parallel agents. ([OpenAI][1])

---

# 10. Don't make the agent commit every five seconds

I'd establish a meaningful commit policy.

For example:

```text
One coherent task
       ↓
implementation
       ↓
tests
       ↓
verification
       ↓
one or a few meaningful commits
       ↓
PR
```

Avoid agent-generated commit histories such as:

```text
fix typo
fix typo again
add function
fix function
test
fix test
fix test again
oops
final
final2
final3
```

The agent's internal iterations don't need to become project history.

---

# 11. Make the PR the primary unit of human review

This is where I think the AI-assisted development process becomes quite elegant.

The human shouldn't have to inspect every keystroke the agent makes.

Instead:

```text
                  Agent
                    │
                    ▼
              isolated branch
                    │
                    ▼
             automated tests
                    │
                    ▼
                   PR
                    │
             ┌──────┴──────┐
             ▼             ▼
       automated        human
         review         review
             │             │
             └──────┬──────┘
                    ▼
                  MERGE
```

And the human review should concentrate on:

* Does this solve the actual problem?
* Is the architecture correct?
* Are the assumptions correct?
* Are there security implications?
* Are there missing edge cases?
* Is the API appropriate?
* Is the implementation unnecessarily complicated?

Not:

> "Did the agent correctly type this `for` loop?"

Copilot's own documentation emphasizes that agent-generated PRs deserve the same thorough review as contributions from human developers. ([GitHub Docs][7])

---

# 12. Use AI code review as a **second reviewer**, not the authority

I'd actually use multiple passes:

```text
Agent implementation
        ↓
automated tests
        ↓
AI review
        ↓
human review
```

And preferably have the reviewer operate with a **different prompt/role** from the implementer.

For example:

> You are reviewing a proposed implementation. Do not assume the implementation is correct. Look specifically for violations of the requirements, architectural inconsistencies, security problems, race conditions, error handling problems, and inadequate tests.

That creates useful adversarial pressure.

---

# 13. Give agents access to tools—but use least privilege

An agent that can:

```text
read source
edit source
run tests
run git
```

is relatively low risk.

An agent that can:

```text
read source
edit source
run arbitrary commands
access production databases
access AWS
deploy software
send email
modify DNS
```

is a very different proposition.

I'd establish explicit boundaries:

```text
                    Agent
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       source       tests       local DB
        ✓             ✓             ✓
        
       production DB     ✗
       production deploy ✗
       secrets            ✗
```

Only introduce higher-risk capabilities when there is a compelling reason.

OpenAI's current guidance on running Codex safely emphasizes technical boundaries, human approval for higher-risk actions, and telemetry/auditability. ([OpenAI][8])

---

# 14. Use MCP selectively

MCP becomes interesting when the agent needs information outside the repository.

For example:

```text
Agent
 │
 ├── GitHub
 ├── issue tracker
 ├── documentation
 ├── database schema
 ├── cloud monitoring
 └── browser/API
```

But I wouldn't connect everything merely because you can.

Each MCP connection increases:

* available context
* possible capabilities
* attack surface
* potential for accidental actions
* difficulty of understanding what the agent can do

So my rule would be:

> **Give an agent the minimum external capabilities required to perform its role.**

Copilot's current customization system explicitly supports MCP servers attached to agents. ([GitHub Docs][9])

---

# 15. Keep architectural decisions outside the agent

This is where I would retain a strong human role.

For example, suppose you're developing a service and the agent proposes:

```text
REST API
   ↓
service layer
   ↓
repository
   ↓
PostgreSQL
```

That's fine.

But if the project has a major decision between:

```text
PostgreSQL
vs
MongoDB
```

or

```text
monolith
vs
microservices
```

or

```text
synchronous
vs
event-driven
```

I wouldn't ask the coding agent to silently make that decision.

Instead:

```text
              Human
                │
        architectural decision
                │
                ▼
          documented decision
                │
                ▼
              Agent
                │
          implementation
```

The agent can certainly **research and recommend** alternatives. But consequential architectural decisions should become explicit human decisions.

---

# 16. Maintain an Architecture Decision Record

This works particularly well with agents.

For example:

```text
docs/decisions/
    001-use-postgresql.md
    002-use-rest-api.md
    003-authentication-model.md
    004-background-jobs.md
```

Each says roughly:

```text
Decision
Context
Alternatives considered
Decision
Consequences
```

Then the agent can understand *why* the system looks the way it does.

This prevents a common AI failure mode:

> "I found a cleaner way to do this, so I changed the architecture."

No.

The agent should understand:

> "The architecture looks unusual for reason X; don't 'fix' it."

---

# 17. Don't create an "AI instruction wiki"

This is related to your earlier question about separating developer instructions from the source tree.

I'd divide information into three categories:

### A. Project truth

```text
docs/
```

Requirements, architecture, decisions, API specifications.

### B. Agent behavior

```text
.github/
AGENTS.md
skills/
```

How the agent should work.

### C. Task definition

```text
GitHub Issues / Projects
```

What needs to be done now.

So:

```text
        PROJECT KNOWLEDGE
               │
        ┌──────┴──────┐
        ▼             ▼
     docs/         .github/
   "what is       "how agent
   the system?"   should work"
        │             │
        └──────┬──────┘
               ▼
          GitHub Issue
          "do this now"
               │
               ▼
             Agent
```

I think this is a particularly good long-term architecture.

---

# 18. My preferred agent setup

If I were starting a new project today, I'd begin with approximately this:

```text
                    ┌─────────────────────┐
                    │       HUMAN         │
                    │                     │
                    │ requirements        │
                    │ architecture        │
                    │ priorities          │
                    │ final review        │
                    └──────────┬──────────┘
                               │
                               ▼
                     ┌──────────────────┐
                     │ GitHub Issue /   │
                     │ Feature Spec     │
                     └────────┬─────────┘
                              │
                              ▼
                   ┌────────────────────┐
                   │   CODING AGENT     │
                   │                    │
                   │ research           │
                   │ plan               │
                   │ implement          │
                   │ test               │
                   │ debug              │
                   └─────────┬──────────┘
                             │
                             ▼
                       ┌───────────┐
                       │    PR     │
                       └─────┬─────┘
                             │
                    ┌────────┴─────────┐
                    ▼                  ▼
              AI code review      HUMAN review
                    │                  │
                    └────────┬─────────┘
                             ▼
                           MERGE
```

And I'd add specialized agents only when experience demonstrates that they're worthwhile.

---

# 19. Codex vs. Copilot

I wouldn't choose based primarily on which model writes better individual functions.

I'd choose based on **where you want the agent to live in your development process**.

### Copilot

Attractive if your development process is already centered on:

* GitHub
* GitHub Issues
* GitHub PRs
* GitHub Actions
* VS Code
* repository-centric collaboration

Its current agent ecosystem is quite mature around repository instructions, custom agents, cloud agents, PRs, code review, MCP and Skills. ([GitHub Docs][3])

### Codex

Attractive if you want:

* an agent-centric development environment
* substantial autonomous coding tasks
* multiple agents working concurrently
* isolated worktrees
* cloud execution
* reusable Skills
* background/long-running work

That's explicitly the direction OpenAI is taking Codex. ([OpenAI][1])

### But I wouldn't necessarily make it an either/or choice

A very reasonable setup is:

```text
                 GitHub
                   │
          issues / PRs / CI
                   │
          ┌────────┴────────┐
          │                 │
       Copilot             Codex
          │                 │
     repo-integrated     autonomous
     development         implementation
          │                 │
          └────────┬────────┘
                   ▼
                  Git
```

The **repository, requirements, tests and CI should be the source of truth**, not whichever agent happens to be working on the project.

---

## The most important conceptual shift

I'd summarize the whole approach this way:

**Don't ask "How do I get an AI to write my software?"**

Ask:

> "How do I construct a software-development process in which an AI agent can safely perform most of the implementation work while the human remains responsible for requirements, architecture, and acceptance?"

That leads naturally to:

**requirements → design → task → isolated agent → implementation → automated verification → PR → AI review → human review → merge**

rather than:

**prompt → generated code → copy/paste → hope.**

And I think the former is where both Codex and Copilot are now sufficiently capable to make a **fundamental change in how a small software team—or even a single experienced developer—can build software**.


[1]: https://openai.com/codex/?utm_source=chatgpt.com "Codex in ChatGPT | AI Coding Agents for Software Engineering | OpenAI"
[2]: https://docs.github.com/en/copilot/reference/customization-cheat-sheet?utm_source=chatgpt.com "Copilot customization cheat sheet - GitHub Docs"
[3]: https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-copilot-overview?utm_source=chatgpt.com "Customize Copilot for your project - GitHub Docs"
[4]: https://docs.github.com/en/copilot/concepts/agents/copilot-cli/about-custom-agents?utm_source=chatgpt.com "About custom agents - GitHub Docs"
[5]: https://docs.github.com/en/copilot/how-tos/copilot-on-github/use-copilot-agents/overview?utm_source=chatgpt.com "Get started with Copilot agents on GitHub - GitHub Docs"
[6]: https://docs.github.com/en/copilot/get-started/best-practices?utm_source=chatgpt.com "Best practices for using GitHub Copilot - GitHub Docs"
[7]: https://docs.github.com/en/copilot/how-tos/copilot-on-github/use-copilot-agents?utm_source=chatgpt.com "Use Copilot agents - GitHub Docs"
[8]: https://openai.com/index/running-codex-safely/?utm_source=chatgpt.com "Running Codex safely at OpenAI | OpenAI"
[9]: https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/create-custom-agents-in-your-ide?utm_source=chatgpt.com "Creating custom agents for Copilot cloud agent in your IDE - GitHub Docs"


