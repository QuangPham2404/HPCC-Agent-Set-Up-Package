# GAAS Agent Workflow — User Guide

## 1. Overview

AI coding agents such as Claude Code, Codex, OpenCode, and IDE-based agents can be very useful when working with HPC projects. A convenient way to use them with GAAS is to keep the **agent running on your local computer** and allow it to access the cluster through SSH when cluster operations are required.

The recommended workflow is:

```text
Local computer
│
├── VS Code / terminal
├── AI agent
├── Local Git repository
│
│        SSH
│         │
│         ▼
└────── GAAS login node
          │
          ├── Cluster-side Git repository
          ├── Job submission / monitoring
          │
          ▼
         PBS
          │
          ▼
     Compute nodes
```

This provides a similar experience to working directly on the cluster while keeping development tools and agent runtimes on your own computer.

### Benefits

* Your AI agent continues to work normally from your local VS Code or terminal.
* Cluster commands can still be executed automatically through SSH.
* SSH key authentication avoids repeated password prompts.
* Your project remains available both locally and on GAAS through Git.
* Computation continues to run through PBS on the appropriate compute nodes.
* The shared login node remains responsive for all users.

Setup is required only once for each computer/project.

### Before you start — identify the researcher's environment

This guide describes the **desired end state**, not a requirement to use WSL or a particular editor. Researchers may work on Windows (PowerShell or WSL), macOS, Linux, local VS Code, terminal-based agents, or with an SSH client such as MobaXterm. **The AI agent must run on the researcher's own computer**, not as an agent or VS Code Remote-SSH extension process on the shared GAAS login node.

Before making changes, determine:

- where the researcher's chosen agent actually runs (Windows, WSL, macOS, or Linux) and what terminal and SSH executable that agent can invoke;
- whether their current VS Code workspace is local or remote, whether they use MobaXterm, and whether VPN/MFA or existing SSH keys/configuration are already working;
- where their project currently resides, whether it uses Git, and which remote Git host is permitted for the research.

Commands below illustrate an OpenSSH/Git environment. Some tools (notably `ssh-copy-id`) are not installed in every Windows environment. A working connection in MobaXterm does not guarantee the local agent uses the same keys/configuration. Reuse an existing working setup where possible; when a command is unavailable, use the equivalent **approved** procedure for the user's actual environment. The agent should explain differences rather than assume everyone uses WSL.

### Non-invasive setup rule

**Adapt to the researcher's project; never migrate, move, rename, delete, or restructure the project merely to use this package.** Setup can add an SSH alias, local code workspace, Git configuration, and agent instructions, but should leave existing GAAS data, output, and project paths intact. For a GAAS-only project, ask whether to add Git metadata (`.git/`) *in place*, or create a separate, small code-only working copy alongside the original. Either option requires the researcher's approval; neither should move their original files.

Large datasets, checkpoints, and experiment outputs can remain on GAAS and need not be copied to the laptop or GitHub. Identify proposed changes so they can be reversed later without disturbing the research files. Obtain confirmation before changing SSH settings, initializing Git in an existing directory, copying files, staging, committing, or pushing.


---

# 2. Set Up Key-Based SSH Access

Normally, an agent running locally would need to enter your GAAS password every time it executes an SSH command.

For example:

```bash
ssh <username>@<gaas-host> "qstat"
```

Key-based SSH authentication allows your computer to authenticate securely using an SSH key instead.

After setup, both you and your local agent can access GAAS using normal SSH commands without repeatedly entering your cluster password.

## Step 1 — Check whether you already have an SSH key

**Environment note:** The commands below assume an OpenSSH client. Windows/PowerShell, WSL, MobaXterm, and local VS Code may use different SSH clients or key locations. Check what the *agent* can invoke, not just what works in the user's terminal. If `ssh-copy-id` is unavailable, ask the user to follow an approved procedure to install only the **public** key on GAAS. Password/MFA prompts must be completed by the user, never recorded or automated by the agent. Verify an unfamiliar server host-key fingerprint through a trusted cluster-admin source before accepting it.

On your local computer, run:

```bash
ls ~/.ssh
```

Look for files such as:

```text
id_ed25519
id_ed25519.pub
```

If you already have an SSH key that you want to use for GAAS, you may use it.

Otherwise, generate a new one.

---

## Step 2 — Generate an SSH key

Run locally:

```bash
ssh-keygen -t ed25519
```

Follow the prompts.

This normally creates:

```text
~/.ssh/id_ed25519       Private key
~/.ssh/id_ed25519.pub   Public key
```

Keep the **private key** on your own computer. Do not upload or share it.

---

## Step 3 — Add the public key to GAAS

Use the appropriate GAAS login address:

```bash
ssh-copy-id <username>@<gaas-host>
```

You may be asked for your GAAS password during this initial setup.

Afterwards, test:

```bash
ssh <username>@<gaas-host>
```

You should now be able to authenticate using your SSH key.

---

## Step 4 — Configure a simple SSH alias

To make cluster access easier for both you and your agent, configure an SSH alias.

Edit:

```text
~/.ssh/config
```

and add:

```sshconfig
Host gaas
    HostName <gaas-host>
    User <username>
    IdentityFile ~/.ssh/id_ed25519
```

You can now connect using:

```bash
ssh gaas
```

instead of typing the full hostname and username.

Your agent can also execute individual commands remotely:

```bash
ssh gaas "hostname"
```

or:

```bash
ssh gaas "qstat"
```

### Why this helps agents

The agent remains on your local machine but can request cluster information whenever it needs it:

```text
Local agent
    │
    ├── "I need to check the job status."
    │
    └── ssh gaas "qstat ..."
                     │
                     ▼
                   GAAS
```

The agent therefore does not need to be installed or run directly on the GAAS login node.

---

# 3. Set Up Your Project Locally and on GAAS

For this workflow, we recommend keeping two Git clones of your project:

```text
GitHub / Git remote
      ▲            ▲
      │            │
   push/pull    push/pull
      │            │
Local clone     GAAS clone
      │            │
VS Code +       PBS / cluster
AI agent        execution
```

The **local clone** is your primary development workspace.

The **GAAS clone** is used when you need to compile, submit jobs, run cluster-specific software, or access GAAS resources.

## Before creating/cloning repositories — Git and Git-host credentials

**GAAS SSH authentication and GitHub authentication are different.** Before following Case A, B, or C, check Git access from the researcher's *local agent environment* and determine how the cluster-side code copy will be synchronized. The agent can diagnose problems and explain choices; the researcher must complete any interactive login/MFA or credential enrollment personally.

1. **Check Git locally.** Run `git --version`. In an existing project, inspect `git status` and `git remote -v`. If new commits will be made, check that the Git author name/email are configured correctly. Do not overwrite global Git settings without permission.
2. **Choose an approved Git host and visibility.** Check whether source code may be sent to GitHub or another host under the researcher's project/institutional rules. Discuss private vs public repository visibility before publishing anything.
3. **Check local Git-host authentication.** Reuse a working credential manager, provider login (e.g. GitHub CLI if already installed), or user-managed Git SSH key. A key for `ssh gaas` does **not** grant access to `git@github.com`. Never ask the agent to read, display, persist, or transmit passwords, PATs, private keys, or MFA codes.
4. **Check cluster-side repository access separately.** Public Git repositories usually allow read-only HTTPS cloning on GAAS. Private repositories may not be accessible from the cluster. **Never put personal GitHub credentials or private keys on GAAS.** If GAAS cannot access the chosen remote by an approved method, pause: agree with the researcher and cluster administrators on an approved, local-controlled SSH synchronization mechanism or Git host before continuing. Do not silently set up a credential workaround.
5. **Check the scope before cloning.** If the Git remote already contains large datasets or sensitive tracked content, a new `.gitignore` will not remove those files from Git history or prevent a full clone. Review an alternative code-only plan first.

Proceed only once the user understands the chosen repository, authorization, and synchronization path.

---

There are three common ways to reach this setup, depending on where the project
currently lives. Pick the case that matches your situation.

## Case A — Creating a brand-new project

If you are starting a fresh project (no code and no repository yet), set up Git
from the beginning so the two-clone workflow works immediately.

**Step 1 — Create a new, empty repository on GitHub (or your Git host).**

In your browser, create a new repository and leave it **empty**: do not add a
README, `.gitignore`, or license. An empty repository is easiest to push a new
project into. Copy the repository URL, for example
`git@github.com:<you>/<project>.git`.

**Step 2 — Set up the project locally and push it.**

On your local computer:

```bash
mkdir ~/project && cd ~/project     # create and enter the project folder
git init                          # start a Git repository here
echo "# My Project" > README.md   # create an initial file
git add README.md                 # stage the file
git commit -m "Initial commit"    # save the first commit
git branch -M main                # name the default branch "main"
git remote add origin git@github.com:<you>/<project>.git   # link to GitHub
git push -u origin main           # upload to GitHub
```

**Step 3 — Clone the repository on GAAS.**

```bash
ssh gaas "git clone <approved-clone-url>"
```

You now have a local clone at `~/project` and a GAAS clone at `~/project` on the
cluster, connected through the same Git repository. For a public GitHub repository,
the approved cluster URL can be an HTTPS clone URL. Do not try to clone a private
repository from GAAS unless the approved access method is already in place.

---

## Case B — Your project already has a GitHub repository

On your local computer:

```bash
git clone <repository-url>
cd <repository>
```

Then connect to GAAS:

```bash
ssh gaas
```

and clone the same repository **using an approved GAAS-side access method**:

```bash
git clone <approved-repository-url>
cd <repository>
```

If this is a private repository and GAAS cannot access it under the credential
policy above, pause before cloning and agree on an alternative synchronization
method. Do not install personal Git-host credentials on GAAS.

You now have:

```text
Local machine
~/project

GAAS
~/project
```

connected through the same Git repository.

---

## Case C — Your project exists only on GAAS (no GitHub repository yet)

This is a common and tricky situation: research files already live on GAAS, but there may be no shared Git repository or local code copy. **Do not migrate or reorganize the existing GAAS project.** The goal is to add a safe synchronization workflow for *approved code and scripts*, while leaving datasets, outputs, and existing paths alone.

**Step 1 — Create an empty repository on GitHub (or your approved Git host).**

In your browser, create a new repository with appropriate visibility and leave it **empty**: do not add a README, `.gitignore`, or license. Copy its repository URL. Confirm that uploading the selected research code to this host is permitted.

**Step 2 — Choose where to add Git, without moving the project.**

From your own terminal, connect to GAAS and inspect your project directory:

```bash
ssh gaas
cd /home/<your_gaas_username>/<project>
pwd
ls
```

Replace the placeholders with your actual paths. With your approval, either initialize Git **in this existing directory** (which only adds `.git/` metadata) or establish a **separate code-only companion directory** containing only explicitly approved source/scripts. If you choose the latter, keep the original directory and data unchanged; do not copy the entire research tree or redirect existing running jobs.

In the **approved Git working directory**, initialize Git only if it is not already a repository:

```bash
git status             # inspect first; if already a Git repository, reuse it
git init               # only if not already initialized, after approval
```

**Do not run `git add .` or commit the research directory yet.** First complete the data inventory and `.gitignore` review in Section 3A below.

**Step 3 — Configure the approved Git remote.**

In that same Git working directory, inspect existing Git remotes. Only add a remote if one is needed:

```bash
git remote -v
git remote add origin <approved-repository-url>
```

Do not run `git remote add` if the remote already exists. A cluster-side remote must use an approved authentication path; a private GitHub remote is **not** assumed to be accessible from GAAS.

**Step 4 — Review the file scope, then make the first commit and push.**

Follow **Section 3A** to identify what should stay on GAAS, create/review `.gitignore`, and preview the exact files to be staged. Only after the researcher approves the file list and the Git-host policy permits the transfer should the approved Git working directory be committed and pushed:

```bash
git status --short
git add .gitignore <approved-source-or-script-paths>
git diff --cached --stat
git diff --cached --name-only
git commit -m "Initial code-only project snapshot"
git branch -M main
git push -u origin main
```

`<approved-source-or-script-paths>` means explicitly selected source/scripts; it is **not** a command to copy wholesale. Review the staged file list before committing. The commands above require approved remote Git access from the machine where they are run. If GAAS cannot authenticate to a private Git host without violating the credential policy, **stop here** and use the agreed local-controlled sync option instead; do not attempt a credential workaround.

**Step 5 — Get the approved code on your computer.**

After the approved code-only repository is available from the Git host, clone it into a **new local directory** appropriate for your operating system:

```bash
git clone <approved-repository-url> ~/project
```

This creates a code working copy, **not** a downloaded copy of all GAAS data. If your environment uses another approved synchronization method, follow that method instead.

**Step 6 — Open the local working copy in your editor or agent environment.**

For example, with local VS Code:

```bash
code ~/project
```

The editor and agent now operate on the local code copy. The original GAAS project, its datasets, and its existing directory layout are preserved. If you chose a separate companion Git directory, document which project scripts/paths it corresponds to; do not silently substitute it for the original research workspace.

The intended relationship is:

```text
Existing GAAS research project (unchanged)
         |
         +---- Approved source/scripts under Git
                    |
                    v
              Git remote / approved sync
                    |
                    v
             Local code working copy
```

---

# 3A. Review research data and configure `.gitignore`

This step belongs **after identifying or initializing the Git working copies, but before the first substantial `git add`, commit, push, or large transfer**. For the GAAS-only Case C, complete this review before Step 4's first commit/push. **Using an agent is strongly recommended**, especially for projects with large scientific outputs, unfamiliar directory structures, or many file types; the researcher must review the final proposal.

**Step 1 — Inventory the project without modifying it.** Have the agent inspect only the approved project scope. Classify files into: (a) source, scripts, small configuration, and documentation to track; (b) datasets, checkpoints, models, compiled artifacts, logs, intermediate/simulation outputs and other files to keep on GAAS; (c) uncertain/sensitive files requiring the researcher's decision. Avoid recursive scans or transfers of huge datasets when filenames, directory summaries, and targeted inspection are sufficient.

**Step 2 — Draft a project-specific `.gitignore`.** Use actual directory names; do not blindly exclude a directory that holds valuable source or reproducibility metadata. A *starting example only*:

```gitignore
# Adjust paths to the actual research project; do not apply blindly
datasets/
checkpoints/
build/
outputs/
*.ckpt
*.log
.env
```

Datasets and results need not be transferred to the laptop or Git host. Some projects deliberately version-control *small* sample data or result summaries; confirm those exceptions with the researcher. Do not put private keys, access tokens, credentials, or secrets in Git.

**Step 3 — Review, then stage only approved files.** Use `git status --short` and `git ls-files` to check what is already tracked. Check exclusions using `git check-ignore -v <path>` where useful. Preview a proposed addition with `git add -n <approved-paths>`, then ask the researcher to confirm the exact inclusion/exclusion list. For the first substantial commit, prefer explicit approved paths over `git add .`. After staging, review `git diff --cached --name-only` or `git diff --cached --stat` before committing or pushing.

**Step 4 — Handle already-tracked data separately.** `.gitignore` does **not** untrack files that Git already tracks and does not remove files from prior commits. If large or sensitive files are already tracked, **stop and consult the researcher** about an approved cleanup or alternative repository; do not delete local files, rewrite history, or publish the repository automatically.

**Completion condition:** the researcher approves the synchronized code scope, the ignored GAAS-only data paths, the chosen Git host, and the first staged file list. Preserve the existing research directory layout.

---

# 4. Using VS Code

If you normally use VS Code Remote SSH, the main change is simply **which copy of the project you open**.

### Previous workflow

```text
VS Code
   │
Remote SSH
   │
   ▼
GAAS project
   │
VS Code server + extensions + agent
run remotely
```

### Recommended workflow

```text
VS Code
   │
   ▼
Local project
   │
AI extension / agent
runs locally
   │
   ├── edit local files
   │
   └── SSH → GAAS when needed
```

You can continue using the normal VS Code interface, extensions, integrated terminal, Git tools, and AI agent features.

The difference is that the development environment now runs on your own computer.

### Using other editors and SSH tools

- **VS Code:** Open the **local** project folder with `File → Open Folder` (or `code <local-project-path>`). Do not use a GAAS Remote-SSH workspace for the agent session, because its server/extensions may run on the shared login node. A local VS Code window can still run SSH commands to GAAS from its integrated terminal.
- **MobaXterm:** You can keep using it for manual SSH/file access. It is not automatically the execution environment for the AI agent. Check that the agent's own local SSH client has the correct alias and keys; do not assume a MobaXterm session grants access to PowerShell, WSL, or the editor.
- **Windows PowerShell / WSL / macOS / Linux:** Use the shell, path syntax, Git installation, and SSH configuration belonging to the agent's actual local runtime. `~` paths and example command syntax elsewhere in this guide may need to be adapted. No particular editor or OS is required.

If the agent cannot use the researcher's existing SSH method safely, stop and ask for help rather than installing a remote coding agent or VS Code server on GAAS.


---

# 5. Add the GAAS Agent Instructions

Copy the provided:

```text
gaas_agent_instruction.md
```

into the root of your local project.

For example:

```text
project/
├── gaas_agent_instruction.md
├── README.md
├── src/
├── scripts/
└── ...
```

This file tells your agent:

* how to connect to GAAS;
* which commands to use for cluster operations;
* how to submit and monitor PBS jobs;
* what operations should remain on the login node;
* what cluster usage rules it should follow.

---

# 6. Make Your Agent Read the Instructions

Different AI agents support persistent project instructions in different ways.

The goal is simply:

> Whenever the agent performs GAAS-related work, it should read and follow `gaas_agent_instruction.md`.

## Simple method

At the beginning of a session, tell your agent:

> Read `gaas_agent_instruction.md` and follow these instructions for all GAAS-related operations in this session.

This method works with almost any agent that can access files in your project.

---

## Recommended method

If your agent supports persistent project instructions, configure it once so that it automatically refers to:

```text
gaas_agent_instruction.md
```

For example, the agent-specific instruction file may contain:

```text
For any task involving the GAAS cluster, first read and follow
gaas_agent_instruction.md.
```

The exact configuration depends on the agent being used.

Examples include:

* `AGENTS.md`
* `CLAUDE.md`
* VS Code / Copilot project instructions
* agent-specific rules or configuration files

Once configured, you should not need to remind the agent manually every session.

---

# 7. Test the Setup

Before using the workflow for normal project work, perform a few simple checks.

## Test 1 — SSH access

From your local computer:

```bash
ssh gaas "hostname"
```

You should receive the GAAS hostname without needing to enter your password repeatedly.

---

## Test 2 — Ask your agent to access GAAS

From your **local** agent, try a simple request such as:

> Check which GAAS host I am connected to.

The agent should perform something equivalent to:

```bash
ssh gaas "hostname"
```

rather than attempting to start another agent on GAAS.

---

## Test 3 — Check PBS

Ask the local agent to inspect your current jobs.

It should use an appropriate command such as:

```bash
ssh gaas "qstat ..."
```

according to the commands specified in `gaas_agent_instruction.md`.

---


## Test 3A — Verify code synchronization and data exclusions

Before any experiment, verify that the local and GAAS-side **approved code files** match the reviewed revision or documented synchronization method. Inspect `git status` and, when applicable, `git log -1 --oneline` in each Git working copy. Confirm that large datasets, credentials, checkpoints and generated outputs are **not** staged, uploaded, or transferred unexpectedly.

## Test 3B — Confirm where the agent is running

Ask the agent to explain which computer/OS is executing it and which SSH client it uses. If using VS Code, ensure the opened workspace is **local**, not GAAS Remote-SSH. Confirm that the intended work does not create agent or VS Code server processes on the login node.

## Test 4 — Submit a small test job

Once basic access works, use a small PBS test job to confirm that the agent understands the intended execution path:

```text
Local agent
     │
     ▼
SSH into GAAS
     │
     ▼
qsub test.pbs
     │
     ▼
PBS scheduler
     │
     ▼
Compute node
```

The computational workload should run through PBS rather than directly on the login node.

---

# 8. Normal Workflow After Setup

Once configured, day-to-day usage should feel very similar to a normal local agent workflow.

For example:

```text
You:
"Run this experiment on GAAS."

        ↓

Local agent:
reads project
edits files if necessary

        ↓

Agent accesses GAAS:
ssh gaas ...

        ↓

Cluster repository updated as required

        ↓

Agent submits:
qsub ...

        ↓

PBS executes job on compute node

        ↓

Agent checks:
qstat ...

        ↓

Agent retrieves output and continues working
```

The important architectural change is simply:

```text
Agent runtime → local computer

Cluster access → SSH

Computation → PBS / compute nodes
```

This keeps the workflow convenient while making agent-assisted cluster usage more suitable for a shared HPC environment.
