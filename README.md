# Agent Set-Up Package

A collection of reusable packages for running AI coding agents against HPC
clusters, without placing the agent runtime on the shared login node.

## What's in this repository

| Package | Purpose |
| --- | --- |
| [`gaas-set-up-package/`](gaas-set-up-package/) | One-time, agent-assisted onboarding that turns a computer and project into the GAAS workflow (`local agent → SSH → GAAS login node → PBS → compute nodes`), plus the day-to-day operating rules for the agent. |
| [`reference-workflow-package/`](reference-workflow-package/) | A project-adaptation workflow pack for Codex-assisted HPC optimization: repository structure, safe remote execution, Git sync policy, build/experiment lifecycle, probing, error handling, and handoff. |

## GAAS set-up package

For researchers who want the convenience of AI coding and research agents
(Claude Code, Codex, OpenCode, VS Code Copilot, and similar) with the **GAAS**
cluster, while keeping the agent on their own computer.

- Agent stays local; GAAS is reached over SSH only when a cluster task needs it.
- Computation still runs through PBS, not on the login node.
- `AGENTS.md` drives a one-time `SET-UP`; `gaas_agent_instructions.md` governs
  day-to-day cluster use; `user_guide.md` is the human-facing manual.

See [gaas-set-up-package/README.md](gaas-set-up-package/README.md) for full
instructions and the two setup options (agent-assisted or manual).

## Reference workflow pack

A reusable workflow template for agent-assisted optimization of HPC
applications (based on the tested LULESH workflow). It defines universal safety
rules and clearly marked placeholders that must be adapted for each project and
cluster.

See [reference-workflow-package/README.md](reference-workflow-package/README.md)
for mandatory reading order, instruction precedence, and adaptation guidance.

## Layout

```text
Agent-Set-Up-Package/
├── gaas-set-up-package/
│   ├── README.md
│   ├── AGENTS.md                        # one-time SET-UP instructions
│   ├── user_guide.md                    # manual, step-by-step guide
│   └── gaas_agent_instructions.md       # day-to-day cluster rules
└── reference-workflow-package/
    ├── README.md
    ├── AGENTS_EXAMPLE.md                # example project-root activation file
    ├── 00-General-SSH-Rules.md
    ├── 01-Git-Sync-Policy.md
    ├── 02-Repo-Structure.md
    ├── 03-Workflow-General-Notes.md
    ├── 04-Workflow-Probing-Scripts-Rules.md
    ├── 05-Workflow-Error-Patching-Procedures.md
    ├── 06-Workflow-Automation-and-Authorization.md
    ├── 07-Workflow.md
    └── clusters/                        # optional reference material
```