# Antigravity CLI Harness

A meta-skill that turns a one-line domain description into a working multi-agent team for the **Antigravity CLI**.

## What it does

Tell the harness *"build me a harness for X"* and it scaffolds, in one go:

- A **plugin** under `.agents/plugins/{domain}-plugin/`
- A team of **subagents** (`agents/{name}/agent.json`), each with a defined role, tool set, and system prompt
- A set of **skills** (`skills/{name}/SKILL.md`) those agents use to do their work
- An **orchestrator skill** that wires the team into a workflow with explicit data-passing and error-handling rules
- A pointer entry in **`AGENTS.md`** so future sessions auto-trigger the harness

The result is a reusable, evolvable team architecture — not a one-shot script.

## Why a harness?

Single-agent prompts hit a ceiling once a task crosses multiple specializations (e.g. analysis → build → QA). A harness solves this by:

1. **Splitting expertise** into focused subagents, each with its own context window.
2. **Standardizing collaboration** through a file-based workspace (`_workspace/`) and an orchestrator.
3. **Surviving across sessions** — definitions live on disk, so the team is reproducible and improvable over time.

## Generated structure

```
.agents/
└── plugins/
    └── {domain}-plugin/
        ├── plugin.json
        ├── agents/
        │   └── {agent-name}/
        │       └── agent.json          # subagent definition
        └── skills/
            ├── {orchestrator}/
            │   └── SKILL.md            # workflow that wires the team
            └── {skill-name}/
                ├── SKILL.md            # how a single capability works
                └── rules/              # progressively-loaded references
AGENTS.md                               # trigger pointer + change log
```

## Architectural patterns

The harness picks one of six team patterns based on the domain:

| Pattern | When to use |
|---|---|
| **Pipeline** | Sequential, dependent steps |
| **Fan-out / Fan-in** | Independent work done in parallel |
| **Expert Pool** | Conditional routing to specialists |
| **Producer–Reviewer** | Generate then QA in a loop |
| **Supervisor** | Central agent manages state and dispatch |
| **Hierarchical Delegation** | Recursive sub-delegation |

## Execution modes

| Mode | When |
|---|---|
| **Subagent** *(default)* | ≥2 specializations collaborating; each runs in an isolated context via `invoke_subagent` |
| **Parallel subagent** | Independent work to run concurrently |
| **Direct execution** | Simple, one-shot tasks where agent separation is overhead |

## Workflow

The meta-skill runs through seven phases:

0. **Audit** — detect existing plugins, decide new build vs. extension vs. maintenance.
1. **Domain analysis** — identify task types, codebase, user skill level.
2. **Team architecture** — choose execution mode + pattern, split work into specializations.
3. **Subagent definitions** — write each `agent.json` with role, tools, and I/O protocol.
4. **Skill creation** — write `SKILL.md` files with pushy, trigger-friendly descriptions and progressive disclosure into `rules/`.
5. **Orchestration** — wire the team with file-based data passing, error handling, and follow-up support.
6. **Validation** — structure checks, trigger checks (should-trigger + near-miss), dry-run.
7. **Evolution** — collect feedback after each run; update agents/skills and log changes in `AGENTS.md`.

## Installation

Drop the `.agents/plugins/harness-plugin/` directory into any Antigravity CLI project. The `harness` skill auto-triggers on requests like:

- "build a harness for {domain}"
- "set up a harness", "design a harness"
- "audit / sync the harness", "harness status"

## Usage

In an Antigravity CLI session:

```
> Build a harness for a content marketing pipeline.
```

The skill will:

1. Audit any existing `.agents/plugins/` and `AGENTS.md`.
2. Propose a team (e.g. researcher → writer → editor → QA) and confirm with you.
3. Generate the plugin, write `agent.json`/`SKILL.md` files, and register the trigger in `AGENTS.md`.
4. Run validation and report.

Follow-up turns (`"redo the analyst step"`, `"add a security reviewer"`, `"harness audit"`) are handled by the same skill in extension / maintenance mode.

## References

The skill ships with internal guides under `.agents/plugins/harness-plugin/skills/harness/rules/`:

- `agent-design-patterns.md` — pattern catalog, separation criteria
- `team-examples.md` — full example team definitions
- `orchestrator-template.md` — orchestrator skeleton with error handling
- `skill-writing-guide.md` — `SKILL.md` authoring patterns
- `skill-testing-guide.md` — trigger and execution testing methodology
- `qa-agent-guide.md` — designing QA subagents

## Acknowledgements

Ported from [revfactory/harness](https://github.com/revfactory/harness), with rework for Antigravity CLI's plugin and subagent model.
