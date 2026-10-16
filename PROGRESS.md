# Progress Log — GAAS Agent Setup Package

**Project:** GAAS Agent Setup Package  
**Record date:** 2026-10-16  
**Milestone:** Version 1 (V1) complete; Pilot Test 1 complete  
**Current status:** **V1 development closed — initial pilot successful; not yet ready for general researcher deployment**

## 1. Milestone summary

Version 1 of the reusable, local-agent onboarding package is complete and has undergone its first hands-on pilot. The package is intended to let a researcher run `SET-UP` in the downloaded tool directory, establish an approved local-to-GAAS development workflow, and install project-specific persistent agent instructions. The agent runs locally and accesses GAAS through SSH; compute work belongs on PBS-managed nodes.

**Pilot Test 1 demonstrated that the central onboarding design works in the environment tested.** The agent recognized `SET-UP`, conducted an adaptive setup conversation, handled project-location questions and Git exclusions, installed project-local instructions, and ended by asking the researcher to perform manual verification.

This is an **initial functional validation**, not a claim that all six planned environment/case combinations have passed. V1 is closed as a development milestone; the issues below remain before wider deployment.

## 2. Version 1 completion

The V1 package provides:

- A reusable local setup entry point (`SET-UP`) and onboarding instructions in `gaas-set-up-package/AGENTS.md`.
- A researcher-facing reference in `gaas-set-up-package/user_guide.md`.
- A template for **personalized, project-local** permanent GAAS agent instructions (`gaas_agent_instructions.md`), without modifying the reusable download.
- Support for three onboarding cases: **Case 1** — new/local project; **Case 2** — existing Git project; **Case 3** — GAAS-only project.
- Permission checkpoints, Git/`.gitignore` data-scope review, approved code synchronization methods, PBS working-directory considerations, and final manual verification.
- A tiny `hello_world/` fixture for pilot testing; a separate `hello-world` GitHub repository was also prepared for existing-Git tests.

**V1 development milestone:** Complete. These are documented capabilities; their reliability across fresh researcher environments remains subject to testing.

## 3. Pilot Test 1 — scope and conditions

| Item | Record |
| --- | --- |
| Environment | Researcher's own **WSL**, using a **CLI coding agent** |
| Testing objective | Exercise `SET-UP` and observe onboarding decisions, safeguards, and user interaction |
| Project cases | Observations include **Case 3 (GAAS-only)**; this log does **not** assert full coverage of all six planned runs |
| Local SSH setup | **Already configured** before the pilot |
| Local Git setup | **Already configured** before the pilot |
| GAAS-side Git setup | **Already configured** before the pilot |
| VS Code agent environment | **Not yet tested** |
| Fresh SSH/Git configuration | **Not yet tested** |

Because SSH and Git were already functional, the pilot mainly exercised the agent's **reuse of an existing environment**, rather than first-time SSH-key, Git-host authentication, or GAAS-side Git provisioning.

## 4. Successful observations

1. **`SET-UP` trigger worked.** The agent recognized the command and initiated the intended onboarding process.
2. **Adaptive, interactive setup worked.** The agent continued asking relevant questions rather than assuming the user's project layout and configuration.
3. **Project locations were handled.** The agent requested location information and adapted its workflow to the user's project.
4. **Git exclusions were handled correctly.** The agent detected the need for `.gitignore` and implemented the appropriate exclusions in the tested setup.
5. **Persistent instructions were generated appropriately.** The creation of the project-specific agent instruction file worked well.
6. **Final handoff respected manual verification.** The agent prompted the researcher to verify the workflow manually rather than claiming full end-to-end validation on its own.
7. **Case 3 completed after researcher guidance.** Once directed to use the existing GAAS project directory, the agent continued successfully.

**Overall assessment:** The V1 architecture functioned substantially as intended during this first WSL/CLI pilot.

## 5. Issues and noteworthy behavior

| ID | Observation | Assessment | Follow-up |
| --- | --- | --- | --- |
| P1-01 | During early SSH inspection, the agent requested access to `.ssh` and `/etc`. | **Needs investigation.** The request may reflect environment probing, but broad access should not be assumed necessary. No specific unsafe read or modification is established by this observation. | Audit exactly what was requested and why; prefer minimum-scope read-only checks, existing SSH aliases, and user-managed authentication. Do not expose private keys or other secrets. |
| P1-02 | After creating a local project directory while the agent was running from the downloaded setup-package directory, it requested permission to access the separate project folder. | **Expected workspace boundary, not a functional defect.** The researcher must approve access; this matches the intended permission model. | Keep as an onboarding usability note. Make the next required workspace permission clear to the researcher. |
| P1-03 | The agent did **not** ask the researcher for the PBS **project/account** and **queue**. | **Configuration gap.** These site-specific scheduler values may be necessary to submit jobs correctly. | Ensure onboarding explicitly discovers or asks for required project/account and queue, validates them against GAAS policy, and carries approved values into the project configuration. Do not invent them. |
| P1-04 | In **Case 3**, the agent initially proposed a **separate project clone/working copy** rather than working in the existing GAAS project directory, citing reluctance to make the original directory Git-tracked. | **Workflow ambiguity / excess conservatism.** The motive is the agent's reported rationale, not a proven root cause. The user had to steer it to the in-place choice; it then worked. | Ensure the agent presents **in-place Git metadata** and **separate code-only companion** as options, asks which is approved, and does not choose a new clone automatically. Preserve safeguards against unapproved modifications. |

These are **pilot observations**, not fixes already implemented. Avoid broadening filesystem access or weakening approval safeguards merely to suppress user prompts.

## 6. Validation gaps before researcher deployment

The planned lightweight test matrix is **WSL CLI / local VS Code × Case 1 / Case 2 / Case 3**. The goal is to evaluate onboarding behavior, not deploy substantial scientific workloads.

Priority follow-up checks:

1. **SSH preflight:** Determine why `.ssh` and `/etc` permissions were requested, and whether least-privilege alternatives work.
2. **PBS settings:** Confirm project/account and queue discovery or prompting and correct representation in the final project-local instructions.
3. **Case 3 directory decision:** Retest with a GAAS-only project to verify that the agent asks the researcher to choose between an approved in-place setup and a companion workspace.
4. **Fresh-configuration path:** Exercise initial SSH/Git setup in an environment where these are **not already configured**; Pilot Test 1 did not cover this.
5. **Local VS Code:** Test the agent from a **local** VS Code workspace, including SSH-client selection and access to a project outside the setup-package workspace.
6. **Remaining case coverage:** Complete only the most useful outstanding combinations in the existing **2 environments × 3 cases** matrix; avoid expanding the matrix without a concrete reason.

A successful future run should not be counted as full verification unless the required SSH, approved Git/sync configuration, project-local instructions, and any chosen PBS smoke check are actually verified. Do not infer these from setup dialogue alone.

## 7. V1 closure decision

**Decision: Close V1 and Pilot Test 1 as completed milestones.**

**Reasoning:** The initial pilot provides meaningful evidence that the core interaction model, project-aware onboarding, Git exclusion handling, personalized policy installation, and manual verification handoff work as designed. The remaining issues are bounded enough to track explicitly rather than block recognition of the V1 milestone.

**Deployment decision: Not yet approved for general researcher use.** Resolve or consciously accept the observed access, scheduler-configuration, and Case 3-selection issues, and run representative fresh-environment/VS Code checks before wider rollout.

**Next phase:** Targeted V1 follow-up validation and workflow refinement based on pilot evidence. Keep the test projects minimal so effort goes into improving the user experience and agent decision-making rather than preparing applications.

---

*This file records V1 and Pilot Test 1 progress. It does not assert that the above follow-up fixes have been implemented or that unperformed tests have passed.*
