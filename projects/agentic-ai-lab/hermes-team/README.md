# Hermes Lean Agent Team

A lean 4-agent AI engineering team built in Hermes to test practical multi-agent orchestration, cost control, independent verification, and reproducible AgentOps workflows.

The project began with a larger multi-agent design that proved too complex and expensive. It was redesigned around four persistent specialist profiles: Odin, Brokkr, Veritas, and Bao. The final design uses direct routing, Kanban execution, Git-backed configuration, and independent QA.

The final architecture was validated through an end-to-end Market Research Assistant workflow covering research, review, revision, artifact generation, failure detection, targeted correction, and final QA approval.

## Problem

The original experiment used a larger multi-agent structure with approximately seven agents plus coordination layers.

Parts of the design worked, but it introduced:

- higher latency,
- higher model cost,
- unnecessary orchestration complexity,
- backend slot limitations,
- inconsistent parent-agent resumption,
- excessive manual configuration,
- unexpected Mixture of Agents cost.

The project was redesigned around a simpler principle:

> Use the smallest capable team for each task.

## Architecture at a Glance

```text
User
  |
Odin — Chief Agent / Orchestrator
  |
  +--> Bao — Research
  +--> Brokkr — Development
  +--> Veritas — Independent QA
```

Execution model:

- direct routing by default,
- persistent named profiles through Hermes Kanban,
- smallest team necessary per task,
- independent QA when the cost is justified,
- Git/repository files as the source of truth,
- human approval for destructive or high-impact actions.

Validated model/provider split: Odin and Veritas use GPT-5.6 Sol through `openai-codex`, Brokkr uses GPT-5.5 through `openai-codex`, and Bao uses `deepseek/deepseek-v4-pro` through Nous. Nous remains available, but the final team uses it primarily for Bao research to reduce cost.

## First Design

The first architecture experimented with:

- multiple specialist agents,
- coordinator layers,
- broader delegation chains,
- heavier use of agent-to-agent orchestration,
- Mixture of Agents for some decisions.

That experiment produced useful evidence, but it was too complex for routine use.

## Observed Failures

Key lessons from the first experiment:

- more agents did not automatically improve results,
- delegation layers increased latency,
- parent agents did not always automatically resume,
- backend slot limits affected profile and settings loading,
- MoA could become expensive unexpectedly,
- manually configuring every bot was difficult to reproduce,
- orchestration behavior needed stronger verification.

These failures are treated as engineering evidence rather than hidden mistakes.

## Redesign

The redesigned team contains four agents.

### Odin — Chief Agent / Orchestrator

Responsibilities:

- understand the objective,
- choose the smallest effective team,
- route directly to specialists,
- integrate results,
- enforce approval boundaries.

### Brokkr — Developer Agent

Responsibilities:

- implementation,
- coding,
- configuration,
- debugging,
- repository changes.

### Veritas — Quality Assurance Agent

Responsibilities:

- independent verification,
- testing,
- requirements validation,
- regression review.

### Bao — Research Agent

Responsibilities:

- external research,
- current documentation,
- technical comparison,
- fact verification.

## Default Routing

Routine implementation:

```text
User
  |
Odin
  |
Brokkr
  |
Odin
  |
User
```

For independently verified implementation:

```text
User
  |
Odin
  |
Brokkr
  |
Veritas
  |
Odin
  |
User
```

Research is routed directly through Bao when needed.

## Design Principles

- No sub-managers initially.
- Prefer direct routing: Odin → specialist.
- Use the smallest team necessary.
- Use one primary model per agent by default.
- Use MoA only when the extra cost is justified.
- Keep status and final reports concise.
- Require human approval for destructive, irreversible, sensitive, or high-impact actions.
- Treat Git and repository files as the source of truth.
- Record failures, limitations, cost issues, and design changes.

## Repository Structure

```text
hermes-team/
├── README.md
├── team-config.md
├── rebuild-guide.md
├── odin-soul.md
├── brokkr-soul.md
├── veritas-soul.md
├── bao-soul.md
├── skills/
│   └── software-development/
│       └── lean-code-review/
│           └── SKILL.md
├── market-research-demo.md
├── market-research-portfolio-brief.md
└── change-log.md
```

Related validated research artifact:

```text
research/
└── 2026-smb-manufacturing-ai-adoption.md
```

## Reproducibility

Reusable agent behavior is stored in the repository rather than relying only on manually configured Hermes profiles.

This allows the team to be:

- rebuilt,
- reviewed,
- version controlled,
- tested,
- compared across experiments.

Hermes runs the team.

Git defines the team.

## Safety

Human approval is required before destructive, irreversible, sensitive, financially consequential, security-sensitive, or production-impacting actions.

Agents may prepare such actions but should stop before execution.

## Validation Results

The redesigned architecture was validated through a real end-to-end Market Research Assistant workflow.

The demonstration covered:

- orchestration,
- research,
- implementation,
- independent verification,
- tool use,
- cost control,
- failure recovery,
- reproducibility.

The final validation also covered Veritas' repository-managed `lean-code-review` skill: it is read-only, diff-first, and behaviorally validated with one clean PASS fixture and one intentionally flawed FAIL fixture.

The workflow produced a validated research artifact and a portfolio-ready market research brief.

Independent QA caught upstream research issues and a downstream unsupported claim introduced during artifact generation. Targeted correction tasks were used instead of restarting the full workflow.

Final result:

**PASS — portfolio artifact validated and ready for presentation.**

## Rebuild / Quick Start

The Hermes team can be rebuilt from the repository without relying on remembered bot configuration.

Core repository files:

- `team-config.md` — architecture, routing, approval, and execution rules
- `odin-soul.md` — Odin behavior
- `brokkr-soul.md` — Brokkr behavior
- `veritas-soul.md` — Veritas behavior
- `bao-soul.md` — Bao behavior
- `skills/software-development/lean-code-review/SKILL.md` — Veritas diff-first review skill
- `change-log.md` — design changes, failures, validation results, and lessons learned

Validated Hermes profiles:

- `odin`
- `brokkr`
- `veritas`
- `bao`

Persistent specialist work should be routed through Hermes Kanban.

`delegate_task` should be reserved for temporary child agents because it does not load the named persistent specialist profile.

Kanban requires the Hermes gateway/dispatcher to be running.

After Hermes updates, restart the gateway before validating dispatch; the validated setup also uses `gateway.multiplex_profiles = true` for named-profile routing.

The repository remains the source of truth for reusable configuration and documentation.

## Portfolio Story

The project is documented as:

**Problem → First Design → Observed Failures → Redesign → Validation → Rebuild**

The goal is to show engineering judgment, not just a working demo.

## Status

Current phase:

**Validated 4-agent Hermes team with persistent profile routing, independent QA, correction loops, and a completed Market Research Assistant demonstration.**

Latest validated workflow:

**Bao research → Veritas QA → Odin review → Bao revision → Veritas PASS → Brokkr artifact → Veritas FAIL → Brokkr correction → Veritas PASS**

Final portfolio artifact:

`market-research-portfolio-brief.md`
