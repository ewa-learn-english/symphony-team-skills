---
name: symphony-team-creator
description: Use when the user asks to design, generate, install, or update a custom multi-participant Codex team; propose the team name or exact update, roles, model and reasoning choices, dependencies, and readiness contract first, then write the self-contained team skill only after explicit approval of that draft.
---

# Symphony Team Creator

## Outcome

Turn a natural-language workflow into a proposed Codex team, or turn a requested change into a proposed update to an installed team. Let the user revise or approve the draft, and only then create or update the globally installed, self-contained skill.

Read [references/codex-team-contract.md](references/codex-team-contract.md) completely before drafting a team. Embed its applicable runtime rules into every generated manager; generated teams must not depend on this creator at runtime.

## Phase 1: Draft only

Start with a proposal even when the user says to create, install, or update a team. An initial request cannot approve a draft the user has not seen.

1. For a new team, extract the intended outcome, participant responsibilities, dependencies, outputs, review or correction loops, and final readiness boundary from the user's description.
2. For an update, resolve the named team in the standard user skills directory and read its complete skill directory, including the manager, UI metadata, participant prompts, and referenced resources, before proposing changes. Stop with the exact prerequisite when the team cannot be found or identified unambiguously.
3. Prefer the smallest team in which every participant has a distinct outcome. Support any number of participants without inventing ceremonial roles.
4. Preserve exact roles, models, reasoning efforts, ordering, or constraints supplied by the user. Propose the remaining choices instead of silently treating them as decided.
5. Always propose an exact model and reasoning effort for every role. Prefer identifiers and efforts exposed by the current Codex environment. When availability cannot be verified, mark the exact recommendation as `availability unverified` and explain the uncertainty; never leave either field blank or make the user design the participant table.
6. Do not create directories, write team files, install anything, or modify the existing team during this phase.

Present one draft containing:

- the operation (`create` or `update`) and proposed `symphony-team-<name>` skill name;
- the team's outcome and explicit non-goals;
- a participant table with role, bounded responsibility, checkout access (`read-only` or `write-capable`), retained handoff, proposed model, proposed reasoning effort, and a concise rationale for both choices;
- the dependency order, allowed parallel work, repeated scopes, and correction or re-review routes;
- the manager's final readiness condition and required evidence;
- assumptions and unresolved choices that the user may change.

For an update, also show:

- every role, prompt, dependency, model, reasoning effort, readiness rule, or UI field to add, change, rename, or remove;
- the files that will change and any exact directory rename;
- the existing behavior and files that will remain untouched;
- the complete resulting participant table, not only the delta.

Ask the user to approve or revise that exact draft. Treat only an unambiguous approval of the displayed draft as confirmation. Any requested change creates a new draft that requires new approval.

## Phase 2: Write after approval

After explicit approval:

1. Resolve the standard user skills directory as `$CODEX_HOME/skills` when `CODEX_HOME` is set, otherwise `~/.codex/skills`.
2. For creation, use the approved name as the destination directory. If it already exists, do not overwrite or merge it without a new update draft and explicit approval.
3. For an update, re-read the installed team before editing. If it changed materially after the approved draft, stop and present a revised draft. Modify only the approved delta and preserve all other files and behavior.
4. Keep the existing skill name during an update unless the approved draft explicitly includes a rename. Treat removal of the old directory as part of the rename and perform it only when that exact removal was approved.
5. For creation, create only this structure:

```text
symphony-team-<name>/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── prompts/
    └── <participant>.md
```

6. Generate or update one complete role prompt per participant and one manager `SKILL.md`. Do not create auxiliary guides, examples, changelogs, or installation files.
7. Keep the resulting team independent of this creator and of every other team skill. Copy all required orchestration behavior into its manager instead of linking to this skill or its reference.

## Generate the manager

Write a frontmatter description beginning with `Use when` that precisely names the generated team's trigger. Make explicit invocation authorize only the participant threads and in-scope work required by that team.

Include:

- the approved outcome, workflow, dependencies, parallel branches, and readiness boundary;
- a table mapping every role to its prompt file, exact model, and exact reasoning effort;
- the complete local-checkout and existing-worktree creation paths from the Codex team contract;
- self-contained assignment fields, waiting and cursor behavior, correction routing, concurrency rules, blockers, retained evidence, and final reporting;
- a rule to keep manager decisions in the manager thread and participant work in participant threads.

Use imperative instructions. Keep the manager lean and specific to the approved workflow; do not preserve the draft discussion or rationale inside the generated skill.

## Generate participant prompts

Give every participant:

- one role outcome and bounded authority;
- required inputs and applicable workspace instructions;
- allowed actions and checkout access;
- required output, evidence, retained file paths, and disposition;
- exact handoff, stop, blocker, correction, and reactivation behavior;
- scoped readiness: the participant judges only its assignment, while the manager judges the whole run;
- literal fallback handling: substitute behavior or proof requires retained evidence that its stated precondition occurred;
- an instruction to preserve unrelated changes and avoid external actions outside the approved team outcome.

Do not add nested subagents, extra participants, publishing, deployment, or destructive actions unless the approved draft explicitly requires them.

## Generate UI metadata

Create `agents/openai.yaml` with quoted `display_name`, a 25–64 character `short_description`, and a one-sentence `default_prompt` that explicitly invokes `$symphony-team-<name>`. Do not add icons, colors, dependencies, or policy fields unless the user requested them in the approved draft.

## Validate and hand off

Validate the created or updated skill before reporting success:

- the directory name and frontmatter `name` match and use lowercase letters, digits, and hyphens;
- the frontmatter contains only `name` and `description`, and the description starts with `Use when`;
- every participant table entry points to an existing prompt and every prompt has exactly one owning role;
- every approved model, reasoning effort, dependency, handoff, and readiness condition appears in the manager;
- no unfinished markers, draft rationale, creator references, or machine-specific absolute paths remain;
- all requested files exist in the standard user skills directory;
- an update contains only the approved changes and preserves the stated untouched behavior.

Use an installed skill-validation helper when available. If it is unavailable, retain that evidence and perform the checks above directly; do not skip validation silently.

Report the installed path, team name, participant settings, created or changed files, invocation example, and validation result. Tell the user the created or updated skill is available on the next Codex turn.
