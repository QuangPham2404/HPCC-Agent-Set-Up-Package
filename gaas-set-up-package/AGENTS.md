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

This file is only for onboarding. Refer to `user_guide.md` for the exact desired
end state, and install `gaas_agent_instructions.md` for day-to-day cluster work.

---

## The `SET-UP` command

When the user issues the command

```text
SET-UP
```

you enter setup mode and help the user establish the workflow described in
`user_guide.md`. Do as much of the setup as you reasonably can, but never assume
the user's environment or project structure without evidence.

Before making changes, read both companion documents:

1. `user_guide.md` — the source of truth for the desired setup.
2. `gaas_agent_instructions.md` — the operating rules you are installing.

---

## General behavior

- **Determine the situation first.** Inspect the user's machine and project
  before changing anything.
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
- **Never assume files can be committed.** Do not commit anything without
  reviewing and, for significant or uncertain changes, confirming first.

---

## 1. SSH setup

Set up key-based SSH access to GAAS following `user_guide.md` Section 2.

- **Do not overwrite** an existing SSH config entry or private key without
  explicit confirmation.
- **Reuse existing valid configuration** where possible (an existing key, an
  existing `Host` entry). Do not create duplicate keys or entries when a valid
  one already works.
- The end state is a simple command or alias (e.g. `ssh gaas`) that the local
  agent can use.
- Never read, echo, store, or log SSH passwords or private keys. Never use
  `sshpass` or password automation.

---

## 2. Project and Git setup

Support at least these cases (see `user_guide.md` Section 3):

- **New project** — no code, no repository yet.
- **Existing Git project** — already has a repository.
- **GAAS-only project** — exists on the cluster but has no remote repository.

### Inspect a messy or GAAS-only project

For a GAAS-only or disorganized existing project, inspect the structure and work
with the user to determine:

- which files are source code, configuration, scripts, and documentation that
  should be version controlled;
- which files are generated output, build artifacts, datasets, checkpoints,
  large files, temporary files, or other content that should stay out of Git;
- whether important project files are spread across multiple directories;
- whether the proposed repository structure needs clarification from the user.

Create or update a `.gitignore` when appropriate.

### Before the first significant commit or push

Before the first significant commit or push of an existing project, show the
user a concise summary of:

- what will be included;
- what will be excluded;
- any files or directories you are uncertain about.

Ask for confirmation before continuing.

### Credentials and secrets

Never commit credentials, private keys, secrets, tokens, or clearly sensitive
configuration. Never store GitHub or cluster credentials on GAAS.

---

## 3. Agent instruction setup

Ensure `gaas_agent_instructions.md` is available in the user's local project root. Copy it there if the original file is in another directory (e.g. in a set-up-guide package). Ensure the project's root AGENT.md have the correct path and specifications for that `gaas_agent_instructions.md` file.

Then help configure the user's chosen agent or harness so it reads these
instructions for GAAS-related work.

Different tools use different persistent-instruction mechanisms (for example
`AGENTS.md`, `CLAUDE.md`, `SKILL.md`, or IDE-specific settings). **Do not assume**
a specific format is supported — determine which agent the user is using and
adapt accordingly. The simple fallback is the per-session instruction of
"read and follow `gaas_agent_instructions.md`" described in `user_guide.md`
Section 6.

---

## 4. Completion

`SET-UP` is not complete until, as applicable:

- the user has a local working copy of the project;
- there is an appropriate GAAS-side copy;
- Git synchronization is configured appropriately;
- key-based SSH access is configured;
- the local agent can reach GAAS with a simple command/alias (e.g. `ssh gaas`);
- `gaas_agent_instructions.md` is installed;
- the user's agent has a mechanism to read the GAAS instructions;
- no unresolved high-impact changes remain.

You do **not** need to run the final workflow test yourself. When setup is
complete:

1. Stop making configuration changes.
2. Tell the user clearly that setup is complete.
3. Point the user to the verification/smoke-test section of `user_guide.md`
   (Section 7).
4. Tell the user to run that test manually before relying on the workflow for
   normal work.

---

## Scope

This file governs the one-time onboarding process only. It is **not** the
permanent GAAS operating policy. Permanent cluster behavior — SSH access, PBS
commands, login-node safety, and usage rules — belongs in
`gaas_agent_instructions.md`.