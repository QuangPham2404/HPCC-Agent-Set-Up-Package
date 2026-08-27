# GAAS Agent Setup Package

A small, easy way to use AI coding and research agents (Claude Code, Codex,
OpenCode, VS Code Copilot, terminal-based agents, and similar) with the **GAAS**
cluster — without running the agent itself on the shared login node.

The recommended setup is:

```text
local agent → SSH → GAAS login node → PBS → compute nodes
```

You keep working in the tools you already know. The agent stays on **your**
computer and only reaches GAAS over SSH when a cluster task actually requires
it. This keeps the shared login node responsive for everyone while making
agent-assisted work more convenient.

Benefits:

* a familiar, convenient agent-assisted workflow (VS Code, terminal, etc.);
* no repeated SSH password entry once key-based access is set up;
* development and agent processes run on your own computer;
* GAAS compute still runs through PBS, the standard workflow;
* less unnecessary load on the shared login node.

---

## What's in this package

| File | What it is for |
| --- | --- |
| `user_guide.md` | The manual, step-by-step setup and reference guide. Follow this yourself if you want to set everything up by hand, or if you want to see exactly what the automated setup is doing. |
| `gaas_agent_instructions.md` | The persistent GAAS instructions your agent follows **after** setup. It covers SSH access, the basic PBS commands, and the general cluster usage rules. |
| `AGENTS.md` | Automatic onboarding instructions. Tell a compatible agent `SET-UP` and it will follow this file. |

`AGENTS.md` is only for the one-time setup. After that,
`gaas_agent_instructions.md` is what governs how your agent uses GAAS day to
day.

---

## Two ways to set up

### Option 1 — Agent-assisted setup (easiest)

1. Download or clone this package onto your computer.
2. Open the package folder with your preferred local AI agent.
3. Make sure the agent can read `AGENTS.md`.
4. Tell the agent:

   **`SET-UP`**

5. Answer the agent's questions and review its confirmation requests.
6. When the agent says setup is complete, run the short manual check described
   in `user_guide.md` (Section 7) before relying on it for normal work.

During setup the agent may ask about things like:

* your GAAS account and login details;
* your existing SSH setup (if any);
* where your project currently lives (locally, on GitHub, or only on GAAS);
* whether you already use Git;
* which files and folders belong to the project;
* whether large datasets or results should stay only on GAAS;
* your preferred agent or editor.

This is not a fixed questionnaire — the agent decides what it actually needs to
know to set things up correctly.

### Option 2 — Manual setup

If you prefer to do everything yourself, just follow `user_guide.md`:

1. Set up key-based SSH access.
2. Set up the local and GAAS project repositories.
3. Install `gaas_agent_instructions.md` in your local project.
4. Complete the verification steps.

---

## What the final workflow looks like

```text
Local computer
│
├── VS Code / terminal
├── AI agent
├── Local project repository
└──  SSH
      │
      ▼
 GAAS login node
       │
       ├── GAAS project repository
       ├── job submission / monitoring
       │
       ▼
      PBS
       │
       ▼
   Compute nodes
```

* The AI agent runs **on your computer**.
* It uses **SSH** only when GAAS access is actually needed.
* **Git** keeps the local and GAAS project copies in sync.
* Computations are submitted through **PBS**, not run on the login node.

---

## After setup

After initial set up, make sure to proceed with `Step 7. Test the set up` in `user_guide.md`

From then on, you normally work in your **local** project using your preferred
editor or agent. `gaas_agent_instructions.md` tells the agent how to interact
with GAAS.

You should not normally need to run your coding agent directly on the GAAS login
node.

Before relying on the workflow for real work, complete the short
smoke-test / verification section in `user_guide.md`.

---

## Scope / proof of concept

This is an initial workflow to make agent-assisted GAAS usage easier and more
suitable for a shared HPC environment.

The instruction files provide a recommended workflow and behavioral guidance;
they are **not** a security sandbox and do not technically enforce every cluster
policy. More advanced technical controls (such as agent sandboxing or centrally
managed onboarding) may be added separately in the future.