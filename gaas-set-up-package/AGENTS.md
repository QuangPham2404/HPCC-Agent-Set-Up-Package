# AGENTS.md — GAAS Onboarding Instructions

This file defines a one-time, agent-assisted **onboarding procedure** for a new
GAAS project. It is the agent's guide for turning a user's computer and project
into the target workflow:

```text
local agent → SSH → GAAS login node → PBS → compute nodes
```

The authoritative, human-facing setup procedure lives in `user_guide.md`. The
permanent operating rules that get installed live in
`gaas_agent_instructions.md`.

This file is only for onboarding. **Use `user_guide.md` as the comprehensive
source of truth for setup steps, supported alternatives, and verification.**
This file only orchestrates those steps for the researcher's own agent; it does
not prescribe a project directory structure or replace the permanent rules in
 `gaas_agent_instructions.md`. Do not duplicate manual procedures here.

**Reusable package invariant:** Researchers download this setup package once to
a folder on their **local computer**, separate from their projects, and may run
`SET-UP` from it for multiple projects. The **entire setup package is read-only**
during onboarding. Never save project-specific paths, credentials, Git remotes,
generated instructions, logs, progress, or configuration in it. Do not copy
this package's onboarding `AGENTS.md` into a research project. Project-specific
changes belong only in confirmed, authorized target project directories or
separately approved user configuration locations. See the **Starting point**
in `user_guide.md` Section 1.

---

## The `SET-UP` command

When the user issues the command

```text
SET-UP
```

you enter setup mode and help the user establish the workflow described in
`user_guide.md`. Do as much of the setup as you reasonably can, but never assume
the user's environment or project structure without evidence.

Before making changes, read both companion documents **in full**:

1. `user_guide.md` — the authoritative setup procedure, including environment
   discovery, Git credentials, the three project cases, Section 3A data review,
   and Section 7 manual smoke tests.
2. `gaas_agent_instructions.md` — the operating rules to install for normal
   GAAS use.

Follow the ordered stages below, selecting only the relevant branch of the
user guide for the researcher's situation. **`SET-UP` is an instruction trigger,
not a shell command or standalone installer.** Do not run any example command
until its prerequisites, paths, permissions, and environment are confirmed.

---

## General behavior

- **Determine the situation first.** Inspect the local agent environment with
  read-only checks. Do not inspect research project contents without prior
  authorization; Stage 3 establishes project paths and permissions.
- **Stay local.** Run the coding agent, editor, and development tools on the
  researcher's computer; never install or start an agent or VS Code server on
  the GAAS login node. Use GAAS only for approved remote commands and scheduled
  computation.
- **Adapt; do not migrate.** Preserve existing research directories, scripts,
  datasets, and outputs. Do not relocate, rename, delete, restructure, or
  overwrite project content to fit this package. Add only approved, identifiable
  configuration or code-only working copies.
- **Ask, don't guess.** Whenever a required detail (username, host, file
  locations, intent) cannot be safely inferred, ask the user.
- **Use your own judgment.** Decide which questions are actually necessary
  rather than following a rigid questionnaire.
- **Explain briefly.** For important decisions, especially for non-technical
  users, explain what you are doing and why in plain language, concisely.
- **Automate the safe parts.** Perform reversible, low-risk operations
  automatically.
- **Confirm before significant changes.** Request explicit confirmation before
  modifying the environment, repository, files, SSH configuration, or remote Git
  state.
- **Avoid destructive actions.** Only delete or overwrite when clearly necessary
  and explicitly approved. If you are unsure how to handle an error safely, stop
  and ask for confirmation, explaining the situation simply.
- **Never assume files can be committed.** Do not commit or push anything
  without reviewing the staged file list and confirming significant or uncertain
  changes, especially on existing projects.
- **Pause safely.** If required access, an approved synchronization method, or
  project-scope permission is missing, stop the affected setup stage, explain
  the blocker, and preserve the current project state. Never work around
  missing credentials or silently relax restrictions.

---

## 0. Discover the local environment and project

Follow the **"Before you start"** and **"Non-invasive setup rule"** parts of
`user_guide.md` Section 1 before configuring SSH or Git.

- Identify the OS and shell **where the agent itself runs**, its available SSH
  and Git tools, and the user's preferred editor (local VS Code, terminal
  agent, MobaXterm for SSH, etc.). Do not assume WSL or Linux syntax.
- Determine whether VS Code is using a **local** folder or a GAAS Remote-SSH
  workspace. If the agent would execute on GAAS, stop and help the user switch
  to an approved local agent environment before continuing.
- Reuse existing local SSH access, Git configuration, project directories, and
  documented cluster paths where possible. Check whether VPN/MFA is required.
- **Classify** the project as new, already Git-controlled, or GAAS-only from
  the researcher's answers and existing authorized context. Ask what paths and
  data policy apply, but **do not inspect target project contents yet** unless
  access was already approved. Stage 3 confirms exact paths and permissions.
  Do not crawl unrelated user directories.
- Choose the relevant platform-specific procedure in `user_guide.md`. Where
  a sample command is unavailable, explain the equivalent supported option or
  stop for user assistance. Do not install an alternate OS, SSH client, or
  remote development server merely to make the example commands work.

Ask only questions whose answers are necessary to proceed. Before any
configuration, briefly confirm the intended end state and additions with the
researcher; **do not require them to adopt the reference workflow's directory
structure**.

---

## 1. SSH setup

Set up key-based SSH access to GAAS following `user_guide.md` Section 2.

- **Do not overwrite** an existing SSH config entry or private key without
  explicit confirmation.
- **Reuse existing valid configuration** where possible (an existing key, an
  existing `Host` entry). Do not create duplicate keys or entries when a valid
  one already works.
- The end state is a simple command or alias (e.g. `ssh gaas`) that works from
  the **same local runtime used by the agent**; a working MobaXterm or WSL
  session does not prove that another SSH client can use the same keys.
- Use an approved public-key installation method when `ssh-copy-id` is not
  available (for example on some Windows setups). Have the researcher handle
  interactive password/MFA steps and verify unrecognized SSH host keys through
  a trusted source. Use a read-only non-interactive connection check before
  considering SSH configured.
- Never read, echo, store, or log SSH passwords or private keys. Never use
  `sshpass` or password automation.

---

## 2. Git and Git-host access preflight

Follow the **"Before creating/cloning repositories — Git and Git-host
credentials"** section of `user_guide.md` **before** Git clone, initial push,
or repository publication.

- Check Git availability and ask about existing repository/remote details.
  For any project-local `git status`, `git remote -v`, or related inspection,
  wait until Stage 3's location/permission checkpoint unless the relevant
  project path and inspection were already approved. Reuse working Git author
  settings and remote authentication where possible.
- Ask whether the proposed Git host and repository visibility are permitted for
  this research. Never assume uploading unpublished code to GitHub is allowed.
- Distinguish **GAAS SSH credentials** from **Git-host credentials**. Guide the
  researcher through any required local, interactive login or MFA; do not
  collect, display, log, or store tokens, passwords, SSH private keys, or MFA
  codes. Never place personal Git-host credentials on GAAS.
- Confirm that the selected Git remote can be accessed by the intended side(s).
  A private repository working locally may fail on GAAS. When GAAS-side access
  is unavailable, stop and agree on an **approved** alternative synchronization
  procedure; do not improvise credential workarounds.
- Check whether an existing remote repository already tracks large or
  sensitive data **before** cloning it. `.gitignore` cannot retroactively
  protect previously committed content.

If authorization or private-repository synchronization is unresolved, report
the blocker and **do not continue to any upload or remote clone requiring it**.

---

## 3. Project and Git setup

Follow the appropriate Case A, B, or C in `user_guide.md` Section 3. Its
**Project location and permission checkpoint** is mandatory at the case-specific
point; being able to read this setup-package folder does not authorize access
to a research project. Select the appropriate case:

- **New project:** create only the agreed local/cluster code workspace and
  approved Git remote.
- **Existing Git project:** inspect and reuse its repositories, remotes, and
  directories; do not recreate or replace them without approval.
- **GAAS-only project:** preserve the original research tree. With approval,
  add Git metadata **in place** or prepare a **separate, code-only companion
  working copy**. Never require migration or download of the whole project.

### Project location and permission checkpoint (mandatory)

Apply the checkpoint in `user_guide.md` Section 3 **before the first
unapproved inspection, clone, directory creation, Git initialization, or other
project-specific modification**. Do not create a separate bootstrap workflow.

1. **Confirm actual locations:** show the researcher the intended **local
   research project directory**, **GAAS-side project/code directory**, and
   only other relevant project paths. Separate these from the **reusable setup
   package's local folder**. Never assume example paths are correct.
2. **Confirm scoped permissions:** ask permission to inspect each relevant
   location and separately authorize the exact additive operations needed
   there. Access to a folder is **not** authorization for bulk transfers,
   commit/push, publication, overwrite, deletion, or unrelated changes;
   preserve the later approval gates in Sections 3 and 3A.
3. **Check real workspace access:** user consent alone may not grant the agent
   filesystem access to a target outside its initial workspace. If access is
   blocked, guide the researcher to add/authorize the specific folder or
   reopen an appropriately scoped **local** agent workspace with access to
   the target and setup guide. Never move the original project, copy the
   whole setup package, weaken the sandbox, or run the agent on GAAS to
   circumvent this restriction.
4. **Pause on uncertainty:** if the location, permission, or technical access
   is unresolved, stop the affected setup steps and explain the next user
   action. Do not inspect or modify unapproved files.

**Case-specific timing:**

- **Case A (new project):** after choosing the local and GAAS project paths,
  **before** creating directories, initializing Git, or cloning to GAAS.
- **Case B (existing Git project):** after identifying the Git URL and any
  existing working-copy paths, **before** inspecting, cloning into, or
  modifying the project directories. Reuse existing clones.
- **Case C (GAAS-only project):** after identifying the existing GAAS project
  and proposed local code path, **before** inspecting its contents,
  initializing Git, creating a companion directory, or transferring code.

Keep project-specific answers in the active session or explicitly approved
project-local records, **never in the reusable tool package**. Once the
checkpoint is satisfied, resume the relevant case in `user_guide.md`.

Do not introduce `builds/`, `experiments/`, or any structure from
`reference-workflow-package/`. The researcher's project layout remains theirs.

For **Case C**, it is acceptable to configure Git and the proposed remote
before the data review, but **defer the first substantial staging, commit,
push, or code transfer until Stage 4**. Treat `user_guide.md` Section 3A as a
mandatory gate before Case C's first commit/push. For Cases A/B, inspect
existing remote contents and permissions before cloning large or sensitive
material. Never use a bulk transfer as a shortcut around this review.

---

## 4. Research data review and `.gitignore`

Follow `user_guide.md` **Section 3A** only after the applicable Stage 3
location/permission checkpoint and identification of the Git working copies,
but **before the first substantial Git commit, push, or large transfer**.

- Offer to perform a read-only, targeted inventory of source, small
  configuration, scripts, documentation, datasets, checkpoints, binaries, and
  generated outputs. Avoid reading sensitive content or recursively scanning
  large datasets just to infer their purpose.
- Propose a **project-specific `.gitignore`** and a concise include/exclude
  summary. Keep large datasets, intermediate results, and checkpoints on GAAS
  when the researcher wants them there; do not force them onto the local
  computer or the Git host.
- Ask the researcher to review ambiguous paths and approve the rules and
  staged file list. Show the dry-run or staged file names before committing.
  Do **not** blindly run `git add .` on an existing research tree.
- Check whether files are **already tracked**. `.gitignore` does not remove
  tracked files or historical Git contents. If large or sensitive content is
  already tracked, **stop** and ask how to proceed; do not automatically
  rewrite history, untrack, delete, or publish it.
- Only after approval may the agent complete the relevant Git commit/push or
  code-only sync steps in Section 3, using the approved authentication path.

---

## 5. Install the persistent agent instructions

Follow `user_guide.md` Sections 5–6. **Copy** the actual file
`gaas_agent_instructions.md` **from the unchanged setup package** into the
researcher's **confirmed and authorized local project root** (or another
agreed project-local location), after checking for an existing file and
obtaining approval for any addition or replacement. Do **not** overwrite an
existing project `AGENTS.md` or other agent policy file. If the project already
has one, preserve its rules and propose a minimal, reviewed reference to the
GAAS instructions.

Then help configure the user's chosen agent or harness so it reads these
instructions for GAAS-related work.

Different tools use different persistent-instruction mechanisms (for example
`AGENTS.md`, `CLAUDE.md`, `SKILL.md`, or IDE-specific settings). **Do not assume**
a specific format is supported — determine which agent the user is using and
adapt accordingly. Verify that the chosen agent can read the installed rules,
including in a new session where practical. The simple fallback is the
per-session instruction to "read and follow `gaas_agent_instructions.md`" as
described in `user_guide.md` Section 6. A filename or unsupported instruction
mechanism must not be silently treated as successful installation.

---

## 6. Configuration handoff and completion

Before reporting setup as configured, inspect the applicable outcomes:

- The agent is confirmed to execute **locally**, with no new agent/VS Code
  server running on the GAAS login node as part of this workflow.
- A working SSH key/alias is available **to the local agent's environment**;
  authentication remains user-managed.
- An appropriate local code working copy and GAAS-side approved code location
  are identified without changing the existing research layout.
- Git-host access and the approved synchronization method are established,
  with no unapproved credentials on GAAS.
- The project-specific `.gitignore`, tracked-file scope, and any initial
  staging/commit/push have been reviewed and authorized.
- `gaas_agent_instructions.md` is installed, and the chosen agent knows how
  to load it for GAAS-related work.
- Summarize any onboarding additions in the handoff or an explicitly approved
  project-local record, **not in the setup package**, with no unresolved
  high-impact changes or hidden blockers.

**Distinguish configuration from verification.** This onboarding agent should
perform safe, read-only checks as it proceeds, but the final manual smoke tests
in `user_guide.md` Section 7 remain the researcher's responsibility,
including SSH, PBS, code synchronization/data exclusions, agent location, and a
small scheduled job. Never claim tests passed if they were not performed.

At handoff:

1. Summarize what was configured, what was reused, and any new files/settings
   or Git remote(s), in plain language.
2. Explain what remains unverified and explicitly identify any blocked or
   incomplete stage. **Do not declare setup complete** if a required stage is
   blocked; provide the next manual action instead.
3. If configuration is complete, say **"Configuration complete; manual
   verification pending"**, direct the researcher to `user_guide.md`
   Section 7, and stop making changes.
4. Remind the researcher that future GAAS operations use the installed
   `gaas_agent_instructions.md`, not this one-time `SET-UP` procedure.
   They can close the reusable tool workspace, open their **own local project**,
   and use the same unchanged setup package for another project later.

---

## Scope

This file governs the one-time onboarding process only. It is **not** the
permanent GAAS operating policy. Permanent cluster behavior — SSH access, PBS
commands, login-node safety, and usage rules — belongs in
`gaas_agent_instructions.md`.