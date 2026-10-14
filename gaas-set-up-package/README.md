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
| `gaas_agent_instructions.md` | Template for persistent GAAS rules. Onboarding installs and **personalizes a project-local copy** for the actual SSH access, approved GAAS paths, and synchronization method. |
| `AGENTS.md` | Automatic onboarding instructions. Tell a compatible agent `SET-UP` and it will follow this file. |

`AGENTS.md` is only for the one-time setup. After that,
`gaas_agent_instructions.md` is what governs how your agent uses GAAS day to
day. The package template stays unchanged; each project gets its own reviewed,
configured copy.

---

## Download only this package (without Git history)

From a terminal on your **local computer**, run:

```bash
npx degit QuangPham2404/HPCC-Agent-Set-Up-Package/gaas-set-up-package gaas-set-up-package
cd gaas-set-up-package
```

This downloads **only** `gaas-set-up-package/`, not the entire repository.
Unlike `git clone`, `degit` creates a plain directory **without a `.git/`
folder**, so the downloaded setup package is **not** tied to this repository's
Git history. It does not initialize Git in your research project either.

**Requirement:** `npx` needs Node.js and npm installed locally. If they are
not available, ask your local agent to help obtain just this folder using an
approved download method, without copying the repository's `.git/` metadata.
Do not run this setup package on the GAAS login node.

Open the downloaded folder with your **local** agent, make sure it can read
`AGENTS.md`, then tell it `SET-UP`. You can use this same downloaded folder
for multiple projects; the agent will ask for the target project's locations
and permissions during setup, without modifying this reusable tool.

---

## One local tool, multiple research projects

Keep the downloaded `gaas-set-up-package/` in any convenient **local**
directory (for example, `~/Downloads/gaas-set-up-package/`), **separate from**
your research projects. It is a reusable **read-only onboarding tool**, not
a project template or the place to store your research code.

```text
Local computer
├── gaas-set-up-package/       # reusable tool, downloaded once
├── research-project-A/       # separate project, configured as needed
└── research-project-B/       # another project, same setup tool
```

You can run `SET-UP` again for another project without downloading the package
again. **Never store project-specific paths, credentials, Git remotes, logs,
generated files, or setup state in the setup-package directory.** The agent
adds approved configuration to the **target project** or an explicitly
approved user configuration location instead.

### What happens when you run `SET-UP`

1. **Start locally:** Open the setup-package folder with your local AI agent
   and issue `SET-UP`. You do **not** need to open or download your research
   project beforehand.
2. **Prepare access:** The agent follows `user_guide.md` to identify your
   environment and help with SSH and Git/Git-host access.
3. **Choose the project:** During project setup, the agent follows the
   applicable case: **A** (new or existing local-only project without Git),
   **B** (existing Git project), or **C** (existing GAAS-only project).
4. **Confirm location and permissions:** At the appropriate point in that
   case, the agent asks you to confirm the **local and GAAS project paths** and
   authorize the specific inspections or **additive** changes it needs.
   If your agent cannot technically access a local directory outside its
   workspace, you may need to grant that access or reopen a **local**
   workspace with access to the project and setup guide. Confirmation is not
   blanket permission to commit, push, delete, or overwrite files.
5. **Finish onboarding:** The agent helps configure project-specific
   `.gitignore` rules, sync approved code (through shared Git or a reviewed,
   local-controlled SSH transfer), confirm the **actual GAAS/PBS execution
   path**, and install the persistent
   `gaas_agent_instructions.md` into your own project. You review significant
   changes and complete the **manual checks in Section 7** of `user_guide.md`.

After setup, open **your research project** with your local agent for normal
work. The original setup-package folder remains unchanged and ready for the
next project. Do not copy the setup package's one-time `AGENTS.md` into the
research project; the persistent GAAS instructions and their agent-specific
reference are what belong there.

---

## Two ways to set up

### Option 1 — Agent-assisted setup (easiest)

1. Download only this package onto your computer using the command above (or an equivalent plain-folder download).
2. Open the package folder with your preferred local AI agent.
3. Make sure the agent can read `AGENTS.md`.
4. Tell the agent:

   **`SET-UP`**

5. Answer the agent's questions, including the **project location and permission checkpoint** for your case (A/B/C). Approve any proposed changes to your project or configuration.
6. When configuration is complete, open **your own local research project** and manually run the checks in `user_guide.md` (Section 7) before normal use.

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

1. Set up or reuse key-based SSH access and check Git/Git-host authentication.
2. Choose Case A, B, or C in Section 3; **confirm project locations and scoped permissions before accessing or changing the project**.
3. Establish the appropriate local/GAAS code working copies, and review data exclusions in Section 3A before substantial commits or transfers.
4. Install `gaas_agent_instructions.md` in the confirmed local research project and configure your agent to read it.
5. Complete the manual verification in Section 7.

---

## What the final workflow looks like

```text
Local computer
│
├── VS Code / terminal
├── AI agent
├── Local code workspace
└──  SSH
      │
      ▼
 GAAS login node
       │
       ├── GAAS code / approved deployment directory
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
* **Approved Git or an explicitly reviewed code-transfer method** keeps the
  local and GAAS code copies synchronized.
* Computations are submitted through **PBS**, not run on the login node.

---

## After setup

After configuration, complete **Section 7 — Test the Setup** in `user_guide.md` before relying on the workflow.

Close the setup-tool workspace and open your **local research project** in your
preferred editor or agent. `gaas_agent_instructions.md` governs subsequent
GAAS operations. Keep the unchanged setup package in its local folder for
onboarding another project.

You should not normally need to run your coding agent directly on the GAAS login
node.

---

## Scope / proof of concept

This is an initial workflow to make agent-assisted GAAS usage easier and more
suitable for a shared HPC environment.

The instruction files provide a recommended workflow and behavioral guidance;
they are **not** a security sandbox and do not technically enforce every cluster
policy. More advanced technical controls (such as agent sandboxing or centrally
managed onboarding) may be added separately in the future.