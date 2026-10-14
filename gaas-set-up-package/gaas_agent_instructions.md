# GAAS Cluster Instructions for Agents

This file tells you (the coding agent) how to work with the GAAS cluster.

You run on the researcher's **local computer**. You are never installed or run on
the GAAS login node itself. When you need to do something on the cluster, you
reach it over SSH only, and you run work only through the PBS scheduler on
compute nodes.

Below is how to connect, how to synchronize approved project code, the
commands you will commonly use on the cluster, and the rules you must follow.

---

## Project-local configuration — complete during `SET-UP`

**This source file is a reusable template, not a configured project policy.**
The onboarding agent copies it into the researcher's **approved local project**
and personalizes **the installed copy only**. Never modify the template inside
the downloaded setup package or store secrets in either copy. The researcher
must review the adapted instructions.

| Setting | Confirmed, project-specific value |
| --- | --- |
| Local agent runtime and local code working directory | <approved-runtime-and-local-path> |
| Working SSH alias/command from the **agent's** local runtime | <approved-ssh-connection> |
| GAAS username | <approved-gaas-username> |
| GAAS code working directory and PBS job-script path | <approved-gaas-code-path> |
| Other authorized GAAS paths and access level (read-only / read-write) | <approved-path-allowlist-or-none> |
| Code synchronization mode | <shared-Git-remote / approved-local-controlled-transfer / other-approved-method> |
| Git remote and branch, **if shared Git** | <confirmed-remote-branch-or-not-applicable> |
| Exact alternative synchronization procedure, **if applicable** | <approved-procedure-or-not-applicable> |

**Before ordinary use:** Replace applicable configuration placeholders with
confirmed, nonsensitive values; mark irrelevant fields **Not applicable**.
Adapt SSH examples, working directories and Git instructions below to the
selected method. **Do not leave `ssh gaas` or Git push/pull as active assumptions
if they do not work for this project.** Record no tokens, passwords, private
keys, MFA codes or credential contents. If any required location, connection
or sync procedure is unknown, report onboarding as incomplete rather than
guessing.

Distinguish approved **code write/execute locations** from directories where
the researcher is permitted only to **read existing datasets/results**.
SSH access and filesystem visibility do not authorize access to unrelated
locations. Use only the paths and access levels explicitly confirmed with the
researcher and allowed by cluster policy.

---

## 1. SSH access

The example below uses SSH alias `gaas` with OpenSSH **for illustration only**.
The installed project copy must use the **actual working SSH connection**
available to the researcher's local agent, as recorded above. Reuse the approved
client/key/username/port settings rather than changing working SSH configuration
to match the example:

```sshconfig
Host gaas
    HostName <gaas-login-host>
    User <your_gaas_username>
    IdentityFile ~/.ssh/id_ed25519
```

Authentication uses an SSH key; a local key passphrase, MFA challenge, or other
user-managed authentication step may still apply, depending on the environment.
Never ask the agent to collect or automate those credentials.

- **Human-operated terminal only:** `ssh gaas` opens an interactive login
  session. Coding agents must use the non-interactive form below instead.
- Run a single command using the approved **non-interactive** SSH connection
  (the example alias is shown below):

```bash
ssh gaas "<command>"
```

Examples:

```bash
ssh gaas "hostname"
ssh gaas "qstat"
```

Always use one-shot, non-interactive SSH commands using the **approved local
connection**. Substitute the confirmed command/alias for `ssh gaas` throughout
the installed project copy wherever it differs. Do not open an interactive
shell and type commands into it. Do not ask for, read, store,
echo, or log the SSH password. Never use `sshpass`, password files, or any tool
that enters a password automatically.

---

## 2. Code synchronization — use the approved project method

**The synchronization method chosen during onboarding controls this section.**
A shared Git remote with two working copies is the recommended **default**,
not a requirement for every research project. Some projects instead use an
institutionally approved local-controlled transfer or other approved system.
Never migrate or reorganize existing research files merely to adopt this
workflow.

### Option A — Shared Git remote (only when approved and operational)

In this mode, the project has two Git working copies that can access the
approved remote (for example GitHub).

- **Local clone** — your primary workspace. This is where you read and edit
  files, prepare and review scripts, and make commits. You live and work here.
- **GAAS clone** — the cluster-side copy. It is used only for what requires the
  cluster: compiling, submitting jobs, running cluster-specific software, and
  accessing GAAS resources.

When Option A is configured, keep them consistent through Git instead of
editing code independently in both places:

```text
Git remote (e.g. GitHub)
      ▲                ▲
  push/pull        push/pull
      │                │
  local clone      GAAS clone
      │                │
  you edit here    PBS / cluster execution
```

### Git rules (only for Option A)

- Before working in either clone, run `git status` to inspect its state.
- Prepare, review, and commit changes in the **local** clone, then `git push`
  when authorized.
- Bring the **GAAS** clone up to date with `git pull --ff-only` before using it
  to build or submit jobs. Verify the intended commit and scripts are present
  remotely before running anything.
- Do not store GitHub or cluster credentials on the cluster.
- If `git pull --ff-only` fails because the clones have diverged, stop and
  inspect the difference. Do not create a merge commit automatically and do not
  overwrite or delete the researcher's changes.

### Option B — Another explicitly approved synchronization method

If this project uses local-controlled file transfer or another approved
non-shared-Git workflow, use **only its documented, approved procedure** and
source/destination paths. **Do not run `git push`, `git pull`, or `git status`
as mandatory steps on directories that are not Git repositories.** Do not
assume private GitHub credentials or keys are available on GAAS, and never
copy personal Git-host credentials to the cluster.

Before transferring code, confirm the source of truth, included file list,
destination, and conflict/overwrite policy. Do not silently transfer datasets,
delete destination files, overwrite divergent work, or run destructive sync
options. Verify the intended code revision at the authorized GAAS working
path before job submission. **If no exact alternative procedure has been
approved, stop; do not invent one.**

**All methods:** Honor the approved project-directory allowlist, keep original
research data in place, and do not edit source on both sides without an
agreed reconciliation procedure.

---

## 3. Common cluster commands

The commands below illustrate alias `gaas`. **In the project-local copy,
adapt the alias, username, paths, and method-dependent steps to the confirmed
project configuration.** The modules and job commands are read-write but
non-destructive in normal use; use them as needed rather than treating this as an exhaustive,
"nothing else is ever allowed" list. However, always honour the working rules at
the end of this section.

### Modules

```bash
ssh gaas "module avail"                      # list available modules
ssh gaas "module avail <name>"               # search for a matching module
ssh gaas "module list"                       # show currently loaded modules
ssh gaas "module show <module-name>"         # show what a module sets
ssh gaas "module load <module-name>"         # load a module
ssh gaas "module load <name>/<version>"      # load a specific version
ssh gaas "module swap <old> <new>"           # swap one loaded module for another
ssh gaas "module unload <module-name>"       # unload a module
ssh gaas "module purge"                      # unload all modules
```

### Submitting and managing PBS jobs

```bash
ssh gaas "qsub <job>.pbs"                              # submit a job (prints job-id)
ssh gaas "qsub -q <queue> <job>.pbs"                   # submit to a specific queue
ssh gaas "qstat"                                       # show your queued/running jobs
ssh gaas "qstat -u <your_gaas_username>"               # show jobs for one user
ssh gaas "qstat -f <job-id>"                           # show full details for one job
ssh gaas "watch -n 5 qstat -u <your_gaas_username>"    # monitor jobs, refresh every 5s
ssh gaas "qdel <job-id>"                               # cancel a job
```

`watch` repeats its command and blocks until stopped (Ctrl+C); for non-blocking
monitoring, poll with repeated `qstat` calls instead.

### Inspecting job output

PBS writes each job's standard output and error to files named after the job,
for example `<job-name>.o<job-id>` and `<job-name>.e<job-id>`.

```bash
ssh gaas "cat <job-name>.o<job-id>"      # view standard output
ssh gaas "tail -n 50 <job-name>.o<job-id>"   # view the end of the output
ssh gaas "cat <job-name>.e<job-id>"      # view error output
```

### Basic navigation (inside the project's approved GAAS paths only)

```bash
ssh gaas "pwd"
ssh gaas "ls"
ssh gaas "cd <subdirectory> && pwd"
```

### Rules for working on the cluster

1. **Stay inside explicitly approved GAAS paths.** The researcher's home
   directory is the default; approved project/shared-group/scratch/data paths
   outside it are allowed **only** when the researcher has permission and
   GAAS policy permits their use. Use the project-local allowlist and observe
   each path's read-only or read-write scope. Do not infer permission simply
   from filesystem visibility or enter unrelated user/project directories.
2. **Do not run compute on the login node.** Never run computational work
   (builds, runs, experiments) directly on the login node. All compute must go
   through the scheduler — either a batch job (`qsub ...`) or an interactive job
   terminal.
3. **Prefer the local project for reading.** Read approved local code first
   when it contains enough information. Use SSH for information that genuinely
   requires GAAS, honoring the approved path allowlist and rules 1–2 above.

---

## 4. Example: submit a small job and check it

```bash
ssh gaas "qsub test.pbs"          # submit, note the job-id it prints
ssh gaas "qstat -u <your_gaas_username>"   # check it is queued/running

# once done, read the output
ssh gaas "cat test.pbs.o<job-id>"
ssh gaas "cat test.pbs.e<job-id>"
```

---

## 5. Cluster usage rules

Beyond the working rules in Section 3, you must also follow these:

- **Do not run computational work on the login node.** Use PBS jobs on compute
  nodes for anything that actually computes (builds, runs, experiments).
- **Do not install or launch another coding agent on the login node.** You are
  the agent, and you live on the local computer only.
- **Use SSH for remote operations.** The login node is not your development
  environment. Edit files locally and use SSH for the cluster-specific steps.
- **Work only in approved GAAS paths.** Honor the project's home, code,
  scratch, and dataset allowlist and each path's read/write permissions.
  Never inspect unrelated users' personal directories, run `find /`, or
  use `sudo`. Do not install software unless the researcher approves it.
- **Do not bypass cluster restrictions or exploit unintended system access.**