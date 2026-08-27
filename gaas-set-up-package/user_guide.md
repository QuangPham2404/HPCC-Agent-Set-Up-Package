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
ssh gaas "git clone git@github.com:<you>/<project>.git"
```

You now have a local clone at `~/project` and a GAAS clone at `~/project` on the
cluster, connected through the same Git repository.

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

and clone the same repository:

```bash
git clone <repository-url>
cd <repository>
```

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

This is the most common and trickiest case: the project already lives on GAAS,
but it is not yet under version control, so there is no local copy and no shared
Git remote. The goal is to bring the existing GAAS project into Git, publish it
to a remote, and get a local copy for your editor and agent.

**Step 1 — Create an empty repository on GitHub (or your Git host).**

In your browser, sign in to GitHub and click "New repository". Give it a name
and leave it **empty**: do not add a README, `.gitignore`, or license. An empty
repository is easier to push an existing project into. Copy the repository URL,
for example `git@github.com:<you>/<project>.git`.

**Step 2 — Turn the existing GAAS project into a Git repository.**

Connect to GAAS and change into your project directory:

```bash
ssh gaas
cd /home/<your_gaas_username>/<project>
```

Replace `<your_gaas_username>` and `<project>` with your actual username and
project folder name.

Now initialize Git and make the first commit:

```bash
git init
git add .
git commit -m "Initial commit"
```

This records all of your existing files as the first snapshot of the project.

**Step 3 — Link the GAAS project to GitHub and push it.**

Still inside the project directory on GAAS:

```bash
git branch -M main
git remote add origin git@github.com:<you>/<project>.git
git push -u origin main
```

This uploads the existing GAAS project to the repository you created in Step 1.
Using the SSH form `git@github.com:<you>/<project>.git` avoids password prompts,
just like your GAAS key. If you set up GitHub with a personal access token or
password instead, you can use the `https://github.com/<you>/<project>.git` form,
but do not store your GitHub credentials on GAAS.

**Step 4 — Get a local copy on your computer.**

On your local computer, clone the newly published repository:

```bash
git clone git@github.com:<you>/<project>.git ~/project
```

**Step 5 — Open the local copy in your editor or agent environment.**

For example, with VS Code:

```bash
code ~/project
```

Your VS Code interface remains the same, but your editor and AI extensions now
run locally rather than through a Remote SSH workspace.

The target state you have just created is:

```text
Existing GAAS project
        │
        ▼
     Git repository
        │
        ▼
GitHub / Git remote
        │
        ▼
Local clone
```

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
