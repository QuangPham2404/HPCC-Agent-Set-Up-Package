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

### Starting point — a reusable local setup tool

Download `gaas-set-up-package/` **once to any convenient folder on your own computer**, separate from research projects (for example, `~/Downloads/gaas-set-up-package/`). Open the folder with a **local** AI agent, let it read `AGENTS.md` and this guide, and issue `SET-UP`. There is no need to download a research project first: its location will be established during Section 3.

Treat the setup package as a **read-only, reusable tool** for any number of projects. **Never write project-specific paths, credentials, Git remotes, configuration, logs, generated instructions, or setup state back into this package.** Project-specific changes belong in the confirmed research project or in separately approved user configuration locations (such as SSH settings). The tool folder is not the research project's Git working directory, and it should not be copied wholesale into that project.

The target research project may be outside the folder in which the agent starts. The location and permission checkpoint in Section 3 handles this transition. **User permission and technical workspace access are distinct:** if the agent cannot access a confirmed local project directory, ask the researcher to grant/add that folder in the local agent's workspace or reopen a **local** agent workspace with access to the project and the setup guide. Do not bypass restrictions by relocating project files, modifying the reusable package, or running the agent on GAAS.

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

Afterwards, **test manually**:

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

For **manual, interactive terminal use**, you can now connect using:

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

**Command examples:** Paths such as `~/project` are illustrative, and anything inside angle brackets (for example, `<repository-url>` or `<username>`) is a placeholder that **must be replaced before executing a command**. Do not paste unedited examples into a shell. Commands involving existing projects must also pass the Section 3 location/permission checkpoint. An AI agent should issue GAAS commands using one-shot `ssh gaas "<remote-command>"`, specifying the approved remote working directory where needed; interactive `ssh gaas` examples below are for human-operated terminals only.

## Before creating/cloning repositories — Git and Git-host credentials

**GAAS SSH authentication and GitHub authentication are different.** Before following Case A, B, or C, check Git access from the researcher's *local agent environment* and determine how the cluster-side code copy will be synchronized. The agent can diagnose problems and explain choices; the researcher must complete any interactive login/MFA or credential enrollment personally.

1. **Check Git locally.** Run `git --version` in the agent's local environment, and ask whether an existing project already uses Git or has a remote. **Do not execute project-local `git status`, `git remote -v`, or similar inspection until that project's path and inspection permission are confirmed at the Section 3 location and permission checkpoint**, unless the scope was explicitly approved earlier. Once approved, inspect existing repository/remote configuration before modifying it. If new commits will be made, check that the Git author name/email are correctly configured. Do not overwrite global Git settings without permission.
2. **Choose an approved Git host and visibility.** Check whether source code may be sent to GitHub or another host under the researcher's project/institutional rules. Discuss private vs public repository visibility before publishing anything.
3. **Check local Git-host authentication.** Reuse a working credential manager, provider login (e.g. GitHub CLI if already installed), or user-managed Git SSH key. A key for `ssh gaas` does **not** grant access to `git@github.com`. Never ask the agent to read, display, persist, or transmit passwords, PATs, private keys, or MFA codes.
4. **Check cluster-side repository access separately.** Public Git repositories usually allow read-only HTTPS cloning on GAAS. Private repositories may not be accessible from the cluster. **Never put personal GitHub credentials or private keys on GAAS.** If GAAS cannot access the chosen remote by an approved method, pause: agree with the researcher and cluster administrators on an approved, local-controlled SSH synchronization mechanism or Git host before continuing. Do not silently set up a credential workaround.
5. **Check the scope before cloning.** If the Git remote already contains large datasets or sensitive tracked content, a new `.gitignore` will not remove those files from Git history or prevent a full clone. Review an alternative code-only plan first.

Proceed only once the user understands the chosen repository, authorization, and synchronization path.

---

## Project location and permission checkpoint (Cases A, B, C)

At the appropriate point in the case below, **pause before the first project-specific file inspection, directory creation, clone, Git initialization, or edit** in a location not already approved. This checkpoint happens *within project setup*; it is not a separate preliminary installation step. Keep the downloaded setup-package folder read-only.

The agent should summarize the intended setup and ask the researcher to confirm:

1. **Exact locations:** the existing or proposed **local research project directory**, the **GAAS-side project/code working directory**, and any additional relevant data/script directories. Distinguish all of these from the **downloaded setup package**; `~/project` and `/home/<username>/<project>` are illustrative placeholders, not assumed paths.
2. **Repository relationship:** the existing or proposed Git remote and which code working copies it will connect. Do not assume all research data belongs in Git; Section 3A determines that scope.
3. **Permissions by scope:** ask separately for permission to inspect specified locations and to perform needed **additive** changes (for example, creating a directory, initializing Git in place, preparing a code-only companion directory, or adding `.gitignore`/agent instructions). Directory access does **not** authorize overwrites, deletion, publication, bulk transfer, commit, or push; obtain subsequent confirmations at the relevant steps.
4. **Workspace access:** verify that the local agent can actually reach the approved local path and that GAAS access uses approved SSH. If access is blocked by the agent's workspace/sandbox, guide the researcher to grant access or reopen an appropriately scoped **local** workspace. Do not alter the reusable setup tool or silently widen permissions.

**Do not proceed without confirmed locations and permission for the intended operation.** If uncertain, stop and ask the researcher. Keep all project-specific answers in the active session or in explicitly approved project-local records, never inside the reusable setup package.

| Case | When to perform the checkpoint |
| --- | --- |
| **A — New project** | After deciding the local/GAAS paths, **before** creating local folders, initializing Git, or cloning to GAAS. |
| **B — Existing Git project** | After identifying the repo URL and any existing local/GAAS clones, **before** cloning into, inspecting, or changing the project folders. |
| **C — GAAS-only project** | After identifying the existing GAAS project and proposed local code workspace, **before** inspecting the GAAS project directory, initializing Git, creating a companion folder, or copying files. |

---

There are three common ways to reach this setup, depending on where the project
currently lives. Pick the case that matches your situation.

## Case A — Creating a brand-new project

If you are starting a fresh project (no code and no repository yet), set up Git
from the beginning so the two-clone workflow works immediately. **If your files
already exist locally but Git is not configured, use the Case A variant below
instead of creating a fresh project directory.**

**Step 1 — Create a new, empty repository on GitHub (or your Git host).**

In your browser, create a new repository and leave it **empty**: do not add a
README, `.gitignore`, or license. An empty repository is easiest to push a new
project into. Copy the repository URL, for example
`git@github.com:<you>/<project>.git`. Creating this Git-host repository requires the researcher's approval.

**Project location & permission checkpoint:** Identify the intended **local project directory** and **GAAS-side code directory** with the researcher and apply the Section 3 checkpoint **before Step 2 creates or changes anything in those directories**. Resolve any workspace-access restrictions first; the setup-package folder must remain separate and unchanged.

**Step 2 — Set up the project locally and push it.**

On your local computer, use the **confirmed new local project path** (the `~/project` path below is an example, not a required location):

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
If GAAS cannot access a private remote without personal credentials, **skip
the GAAS clone** and use the approved local-controlled transfer in Section 3B
after reviewing the specific files in Section 3A.

### Case A variant — Existing local project without Git

At the Section 3 location/permission checkpoint, confirm the **existing local
project** and intended GAAS code/PBS directory. **Do not run the fresh-project
`mkdir` example or reorganize existing files.** After permission to inspect
the directory, review code, data, secrets, and `.gitignore` under Section 3A
**before** staging or transfer. With approval, `git init` may be run *in
place* if it is not already a repository. Stage only reviewed source/scripts,
and push from the local computer to an approved remote if one is desired.
Existing remotes and tracked work must be reused rather than overwritten.

For GAAS deployment, use the shared-Git clone only if its authentication is
approved and functional. Otherwise, use Section 3B's **explicit-file
local-to-GAAS** procedure. Either choice keeps the existing local project
structure intact.

---

## Case B — Your project already has a GitHub repository

**Project location & permission checkpoint:** Identify the repository URL and any existing **local** and **GAAS-side** working copies. Confirm their exact paths and the requested access/creation scope using the Section 3 checkpoint **before cloning into or inspecting those project directories**. If an existing copy is already correct, reuse it rather than cloning over it. If the local agent lacks workspace access, have the researcher grant it while keeping the downloaded setup package unchanged.

If a suitable local clone does not already exist, on your local computer:

```bash
git clone <repository-url>
cd <repository>
```

For **manual terminal setup only**, connect interactively to GAAS if a remote clone still needs to be created:

```bash
ssh gaas
```

Only if the **confirmed GAAS-side directory does not already contain an appropriate working copy**, clone the same repository **using an approved GAAS-side access method**. The next commands are run *inside the manual GAAS session*; a local agent must instead perform the approved operations using non-interactive `ssh gaas "<remote-command>"` calls with the confirmed working directory:

```bash
git clone <approved-repository-url>
cd <repository>
```

If this is a private repository and GAAS cannot access it under the credential
policy above, **skip the GAAS clone** and choose the concrete local-controlled
transfer in **Section 3B** after reviewing the file list in Section 3A, or an
other documented institutionally approved method. Do not install personal
Git-host credentials on GAAS.

You now have:

```text
Local machine
~/project

GAAS
~/project
```

connected through the same Git repository **when shared-Git mode is used**.
With Section 3B, the GAAS code directory is a deployment destination, not
necessarily a second Git clone.

---

## Case C — Your project exists only on GAAS (no GitHub repository yet)

This is a common and tricky situation: research files already live on GAAS, but there may be no shared Git repository or local code copy. **Do not migrate or reorganize the existing GAAS project.** The goal is to add a safe synchronization workflow for *approved code and scripts*, while leaving datasets, outputs, and existing paths alone.

**Choose the sync mode first.** The numbered Steps 1–5 below apply to
**shared-Git mode**, only when GAAS can access the approved Git remote.
If it cannot access a private Git repository, **do not initialize Git,
configure a remote, or attempt GitHub authentication on GAAS** just to follow
those steps. Instead, confirm the Case C project locations/permissions using
the Section 3 checkpoint, inspect only approved code paths, complete Section
3A's file inventory, and use **Section 3B's GAAS-to-local import**. The local
working copy may then be Git-controlled and pushed to a private remote
**from the local computer only**, if the researcher approves.

**Step 1 — Create an empty repository on GitHub (or your approved Git host).**

In your browser, create a new repository with appropriate visibility and leave it **empty**: do not add a README, `.gitignore`, or license. Copy its repository URL. Confirm that uploading the selected research code to this host is permitted.

**Step 2 — Choose where to add Git, without moving the project.**

**Project location & permission checkpoint:** Confirm with the researcher the **exact GAAS project path** and proposed **local code working directory**. Apply the checkpoint in Section 3, including permission to inspect the specified GAAS project and separate approval for any in-place Git initialization or code-only companion copy, **before** the inspection below or any changes. SSH login access alone does not grant permission to inspect unrelated research locations; any local workspace restrictions must be resolved by the researcher without modifying the downloaded tool.

For **human-operated manual setup only**, connect interactively to GAAS and inspect your confirmed project directory:

```bash
ssh gaas
cd /home/<your_gaas_username>/<project>
pwd
ls
```

A local AI agent must **not** open an interactive GAAS session to execute those steps. It should instead use non-interactive `ssh gaas "<remote-command>"` calls and an explicit `cd` to the approved remote project directory for each relevant command.

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

### Case C — Decide where PBS will execute the project code

Before marking the GAAS-only setup complete, confirm the **original GAAS
project path**, any **separate code-only companion**, the **exact PBS job
script/execution directory**, and authorized existing data paths. The
researcher must explicitly approve one arrangement:

- **Original project executes jobs:** Deploy reviewed code there only with
  permission, checking each proposed replacement. Keep datasets, outputs,
  and unrelated files unchanged.
- **Companion code directory executes jobs:** Submit from the approved
  companion directory, referencing original datasets through existing,
  individually authorized paths. Do not move data, alter existing jobs, or
  redirect running workloads.

Identify how code reaches the chosen **actual execution directory**, and
confirm the intended revision and script path there before PBS submission.
Record the decision in the **personalized project-local**
`gaas_agent_instructions.md`, never in the reusable tool. If no safe mapping
is approved, onboarding is incomplete.

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

# 3B. Local-controlled SSH code transfer (private Git or other approved alternative)

This is an **optional, concrete alternative** when GAAS cannot access a
private Git remote. Git credentials remain **only on the local machine**.
Apply the **Section 3 location/permission checkpoint** and **Section 3A
approved-file review** before transferring anything. This is **not** a
project migration; the original GAAS research tree remains in place.

**Prerequisites:** Confirm an operational local SSH connection, the approved
local code directory, the GAAS code source/destination and PBS execution
directory, and whether `rsync` is installed and permitted **on both sides**.
Do not assume native Windows PowerShell has `rsync` or require the user to
install WSL. If `rsync` is unavailable, use an **approved individually scoped
`scp` transfer** or seek administrator help; do not copy the full project.

Have the researcher review a **project-local**
`approved-code-files.txt` manifest containing **one relative regular-file
path per line**, relative to the chosen source directory. For example,
`src/main.py` and `scripts/run.pbs` are only illustrative. Exclude data,
checkpoints, secrets, output, symlinks requiring special handling, absolute
paths, `..` traversal, and **directory-only entries**. This manifest and any
transfer logs belong in the research project, **not the setup tool**.

All paths/aliases in the examples below are **placeholders**. Run these
commands on the researcher's **local machine** after replacing them with the
actual authorized locations. `--files-from` restricts transfer to the
explicitly reviewed list; do not add recursion flags or `--delete`.

### GAAS-only project: import reviewed code to a local working copy

After permission to create the local code directory, **preview** the pull:

```bash
rsync -tv --dry-run --itemize-changes --files-from="<approved-local-manifest>" "gaas:<approved-gaas-source-root>/" "<approved-local-code-root>/"
```

Check the preview with the researcher. Only after approval, execute:

```bash
rsync -tv --itemize-changes --files-from="<approved-local-manifest>" "gaas:<approved-gaas-source-root>/" "<approved-local-code-root>/"
```

The GAAS originals remain untouched. If the user wants a private Git remote,
initialize/reuse Git and publish **only reviewed local code** from the local
computer, after Section 3A's `.gitignore` and staged-file checks. No GitHub
credential is stored on GAAS.

### Deploy local code to the approved GAAS execution directory

With the approved local source and GAAS deployment destination, first preview:

```bash
rsync -tv --dry-run --itemize-changes --files-from="<approved-local-manifest>" "<approved-local-code-root>/" "gaas:<approved-gaas-code-root>/"
```

Review any existing destination files and resolve divergent edits. Only after
explicit approval for each intended replacement, perform:

```bash
rsync -tv --itemize-changes --files-from="<approved-local-manifest>" "<approved-local-code-root>/" "gaas:<approved-gaas-code-root>/"
```

**Never** automatically delete remote files, deploy datasets, or silently
overwrite divergent work. Confirm that the code and PBS script exist in the
**agreed execution directory** before submitting. If the transfer method
cannot be made safe and operational, stop rather than installing GitHub
credentials on GAAS or inventing an alternative.

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
gaas_agent_instructions.md
```

into the root of your **confirmed local project** and customize **that
project-local copy** for the researcher's actual configuration. Keep the
original inside the reusable setup package unchanged.

For example:

```text
project/
├── gaas_agent_instructions.md
├── README.md
├── src/
├── scripts/
└── ...
```

### Personalize the installed GAAS instructions

Before normal GAAS work, have the local agent configure the **copied**
`gaas_agent_instructions.md` and ask you to review the changes:

- Fill its **Project-local configuration** table: local project and runtime,
  actual SSH alias/command, GAAS account, approved code/job directory, and any
  additional GAAS paths with explicitly approved read/write permissions.
- Select the real **code synchronization method**. Use a shared Git remote
  only where both sides have approved access. Otherwise, document the exact
  institutionally approved local-controlled/other transfer procedure; do not
  assume GAAS has GitHub credentials.
- Update the copied file's SSH, path, and Git-command examples to match this
  project. Mark inapplicable settings accordingly; do not store passwords,
  tokens, private keys, or MFA information.
- Inspect the finished project-local file for unresolved required values and
  contradictory instructions. If a working connection, path authorization,
  or synchronization procedure is missing, onboarding is **not yet complete**.

The original file in the reusable setup package is **never edited**.

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

> Whenever the agent performs GAAS-related work, it should read and follow `gaas_agent_instructions.md`.

## Simple method

At the beginning of a session, tell your agent:

> Read `gaas_agent_instructions.md` and follow these instructions for all GAAS-related operations in this session.

This method works with almost any agent that can access files in your project.

---

## Recommended method

If your agent supports persistent project instructions, configure it once so that it automatically refers to:

```text
gaas_agent_instructions.md
```

For example, the agent-specific instruction file may contain:

```text
For any task involving the GAAS cluster, first read and follow
gaas_agent_instructions.md.
```

The exact configuration depends on the agent being used. Refer the agent to
the **personalized project-local copy**, not the source template in the
downloaded setup-package folder.

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
ssh gaas "qstat"
```

according to the commands specified in `gaas_agent_instructions.md`.

---


## Test 3A — Verify code synchronization and data exclusions

Before any experiment, verify that the local and GAAS-side **approved code files** match the reviewed revision or documented synchronization method. Inspect `git status` and, when applicable, `git log -1 --oneline` in each Git working copy. Confirm that large datasets, credentials, checkpoints and generated outputs are **not** staged, uploaded, or transferred unexpectedly.

## Test 3B — Confirm where the agent is running

Ask the agent to explain which computer/OS is executing it and which SSH client it uses. If using VS Code, ensure the opened workspace is **local**, not GAAS Remote-SSH. Confirm that the intended work does not create agent or VS Code server processes on the login node.

## Test 4 — Submit a small test job

After the earlier checks, confirm the **approved GAAS code/PBS directory**,
the user's permission to submit, and any site-required queue/account settings.
Do not guess a queue or project allocation.

With approval, create `gaas_setup_smoke.pbs` in the **local research
project** from this minimal example (adapt resource directives to actual GAAS
policy):

```bash
#!/bin/bash
#PBS -N gaas_setup_smoke
#PBS -l select=1:ncpus=1
#PBS -l walltime=00:02:00
#PBS -j oe

cd "$PBS_O_WORKDIR" || exit 1
printf 'GAAS_SETUP_SMOKE_OK host=%s\n' "$(hostname)"
```

Using the project's **approved Git or Section 3B explicit-file transfer**,
place only this approved script in the confirmed GAAS PBS execution directory.
Then ask the researcher to approve submission. The local agent should use an
equivalent non-interactive command (substitute the real alias and authorized
directory, and quote the remote path correctly for its shell):

```bash
ssh gaas 'cd <approved-gaas-execution-directory> && qsub gaas_setup_smoke.pbs'
```

Capture the **job ID** returned by `qsub`; inspect `qstat -f <job-id>` where
supported and the job's actual PBS output path. A successful scheduled job
prints `GAAS_SETUP_SMOKE_OK` and a **compute-node hostname**. In this
example, `#PBS -j oe` merges standard error into standard output. Completed
jobs may disappear from `qstat`, so inspect their output/status records.

If PBS rejects the submission or the marker is missing, report the error
and leave the test **unverified**. **Never** run the test computation directly
on the login node as a workaround. The researcher performs the final manual
verification before trusting the workflow.

---

# 8. Normal Workflow After Setup

Once configured, the researcher can close the setup-package workspace and open **their own local research project** with the agent. Keep the downloaded package in its original local folder for future projects; it must not contain the current project's configuration, paths, or state.

Day-to-day usage should feel very similar to a normal local agent workflow.

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
