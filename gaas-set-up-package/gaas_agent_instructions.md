# GAAS Cluster Instructions for Agents

This file tells you (the coding agent) how to work with the GAAS cluster.

You run on the researcher's **local computer**. You are never installed or run on
the GAAS login node itself. When you need to do something on the cluster, you
reach it over SSH only, and you run work only through the PBS scheduler on
compute nodes.

Below is how to connect, how to manage the two copies of the project, the
commands you will commonly use on the cluster, and the rules you must follow.

---

## 1. SSH access

The cluster is reached through the SSH alias `gaas`. It is already configured
in `~/.ssh/config` on the local computer as:

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
- Run a single remote command (this is the form you must use):

```bash
ssh gaas "<command>"
```

Examples:

```bash
ssh gaas "hostname"
ssh gaas "qstat"
```

Always use the `ssh gaas "<command>"` form for cluster operations. Do not open
an interactive shell and type commands into it. Do not ask for, read, store,
echo, or log the SSH password. Never use `sshpass`, password files, or any tool
that enters a password automatically.

---

## 2. The two clones: local and GAAS

The project exists as two Git clones that are kept in sync through a shared Git
remote (for example GitHub).

- **Local clone** — your primary workspace. This is where you read and edit
  files, prepare and review scripts, and make commits. You live and work here.
- **GAAS clone** — the cluster-side copy. It is used only for what requires the
  cluster: compiling, submitting jobs, running cluster-specific software, and
  accessing GAAS resources.

The two are kept consistent with Git, not by editing files in both places:

```text
Git remote (e.g. GitHub)
      ▲                ▲
  push/pull        push/pull
      │                │
  local clone      GAAS clone
      │                │
  you edit here    PBS / cluster execution
```

### Basic Git rules

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

---

## 3. Common cluster commands

The commands below cover typical cluster work. Use the `ssh gaas "..."` form for
all of them. The modules and job commands are read-write but non-destructive in
normal use; use them as needed rather than treating this as an exhaustive,
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

### Basic navigation (inside your own home area only)

```bash
ssh gaas "pwd"
ssh gaas "ls"
ssh gaas "cd <subdirectory> && pwd"
```

### Rules for working on the cluster

1. **Stay inside your own home directory.** For any setup or work performed on
   the login node, stay within your own home directory `/home/<your_gaas_username>`
   (ask the researcher for the username beforehand if you do not know it). Do not
   roam outside this directory.
2. **Do not run compute on the login node.** Never run computational work
   (builds, runs, experiments) directly on the login node. All compute must go
   through the scheduler — either a batch job (`qsub ...`) or an interactive job
   terminal.
3. **Prefer the local repository for reading.** Anything that can be answered by
   reading the repository locally (for example, reading files for context) should
   be done in the local clone first; SSH into the cluster only when the files are
   not present locally or the local information is not enough. For tasks that
   involve probing the cluster to discover information, you are responsible for
   deciding whether to use SSH or to search locally — do whichever is effective,
   provided you honour rules 1 and 2 above.

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
- **Work inside your own home directory.** Do not list or read other users'
  directories, do not run `find /`, and do not use `sudo`. Do not install
  software unless the researcher approves it.
- **Do not bypass cluster restrictions or exploit unintended system access.**