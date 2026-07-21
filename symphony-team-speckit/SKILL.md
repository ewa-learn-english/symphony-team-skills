---
name: symphony-team-speckit
description: Coordinate the standalone Team SpecKit workflow in Codex App by creating or forking role-specific Codex threads for specification, planning, tasks, phased implementation, and review in a local checkout or existing worktree. Use when the user explicitly asks to run Team SpecKit or invokes $symphony-team-speckit for a feature in a repository with SpecKit installed.
---

# Team SpecKit Manager

## Outcome

Deliver one approved SpecKit feature through explicit specification, planning, task, phased implementation, and readiness boundaries.
Coordinate participants through new Codex threads; do not perform their assigned work in the manager thread.
Run this workflow only in Codex App; do not translate it to generic subagents or an external orchestrator.

## Start the run

1. Capture the user's complete request, the current workspace, repository rules, SpecKit installation, and relevant existing changes.
2. Determine whether the manager thread runs in the primary local checkout or a linked worktree. Prefer explicit Codex environment context; otherwise compare `git rev-parse --path-format=absolute --git-dir` with `git rev-parse --path-format=absolute --git-common-dir`. Different paths mean worktree mode.
3. In local mode, resolve the current saved project with `list_projects` and create every participant in that project with `environment.type: local`.
4. In worktree mode, fork the current manager thread with `environment.type: same-directory` for every new participant. Do not use `create_thread`, create another worktree, or redirect participants to the primary local checkout.
5. Stop with the concrete prerequisite when local mode cannot resolve the saved project or the repository lacks the SpecKit commands/templates needed by a participant. Do not invent a substitute workflow or silently replace participant threads with subagents.

An explicit request to run this skill authorizes the participant threads required by this workflow. It does not authorize pushing, creating a pull request, deployment, or unrelated repository changes.

## Create participant threads

Before each first activation, read the role's complete prompt from `prompts/` and use it as the beginning of the new thread prompt. Append a self-contained assignment containing:

- the original user request;
- the absolute workspace path and applicable repository rules;
- the feature directory, stage or implementation phase, and task IDs;
- retained inputs and their exact paths;
- the required artifact, verdict, or implementation evidence;
- any accepted requirements or proof deferred to a later phase;
- the exact stop or block condition;
- a reminder that the shared current checkout may contain unrelated changes that must be preserved;
- in worktree mode, an instruction to treat this assignment as authoritative and ignore inherited manager history outside it.

Use these exact participant settings:

| Participant | Prompt | Model | Thinking |
| --- | --- | --- | --- |
| `spec-author` | `prompts/spec-author.md` | `gpt-5.6-terra` | `high` |
| `spec-reviewer` | `prompts/spec-reviewer.md` | `gpt-5.6-sol` | `high` |
| `plan-author` | `prompts/plan-author.md` | `gpt-5.6-sol` | `high` |
| `plan-reviewer` | `prompts/plan-reviewer.md` | `gpt-5.6-sol` | `xhigh` |
| `tasks-author` | `prompts/tasks-author.md` | `gpt-5.6-terra` | `high` |
| `implementation-worker` | `prompts/implementation-worker.md` | `gpt-5.6-sol` | `high` |
| `implementation-reviewer` | `prompts/implementation-reviewer.md` | `gpt-5.6-sol` | `xhigh` |

In local mode, pass the complete prompt, table `model`, and table `thinking` to `create_thread`.

In worktree mode, call `fork_thread` with `environment.type: same-directory` and omit `threadId` so the current manager thread is the source. Then immediately call `send_message_to_thread` for the returned child `threadId` with the complete prompt, table `model`, and table `thinking`. A fork copies only completed history, not the manager's active turn, so never rely on inherited context for the assignment.

Do not override the model or thinking on later follow-up messages.

## Subscribe and coordinate

1. Keep the `threadId`, optional `hostId`, role, checkout mode, assigned stage or phase, and latest wait cursor for every participant.
2. Immediately subscribe with `wait_threads` after the participant has received its initial prompt. Wait for the current dependency before starting its reviewer or the next stage.
3. Pass each returned cursor as `afterCursor` on the next wait. A timeout is not completion; continue waiting without narrating unchanged snapshots.
4. Treat a completed thread's final response as its handoff. Use `read_thread` only when the handoff omits context required for a decision.
5. If a thread needs attention, answer with `send_message_to_thread` when the answer is already established. Ask the user only when the missing decision materially changes scope or safety.
6. Send author fixes, worker fixes, and re-review to the same thread with `send_message_to_thread`. Do not create a replacement thread for the same artifact or phase.
7. Create new `implementation-worker` and `implementation-reviewer` threads for every new implementation phase. Never reuse a phase thread for a different phase.
8. In worktree mode, never run two write-capable participant turns concurrently in the shared checkout.
9. Never mark a stage or phase ready while its required thread is still running, needs attention, or has blocking findings.

## Required workflow

For an initial feature, preserve this order:

1. `spec-author` writes the specification; `spec-reviewer` reviews it; the author fixes findings; the reviewer reports it ready.
2. `plan-author` writes the plan; `plan-reviewer` reviews it; the author fixes findings; the reviewer reports it ready.
3. `tasks-author` writes dependency-ordered tasks grouped into coherent implementation phases; inspect the author's evidence and return material defects to the same thread.
4. For each ready phase, assign the whole phase to a new `implementation-worker` thread. At phase end obtain a ready disposition from a new `implementation-reviewer` thread before starting the next phase.
5. After all phases, confirm that retained artifacts and implementation evidence agree before recording ready. Do not introduce another review or acceptance role.

Each phase disposition covers only that assigned phase. Keep requirements and proof assigned to later accepted phases visible as downstream dependencies; only the manager's final ready decision covers the complete feature.

Do not collapse or skip specification, plan, tasks, or the specification and plan reviews. Do not insert review or test gates between individual tasks inside an implementation phase. On a successor audit cycle, treat human feedback as a delta: reactivate only affected authors, phases, and downstream reviews instead of replaying the whole initial workflow.

## Scope and decomposition policy

- Design only for the demonstrated need and repository rules. Reject speculative scale, multi-tenancy, approval systems, compatibility layers, and infrastructure that the request and repository evidence do not require.
- Use one implementation phase by default. Split only for a real dependency or independently acceptable outcome that cannot be delivered safely in one scope. Never create a docs-only, proof-only, validation-only, or commit-only phase.
- Do not create participant-shaped tasks, one-file microtasks, repeated planning reconciliation, or implementation/reviewer loops without new evidence. Route design defects back to the owning author rather than retrying implementation.
- Keep this workflow local. Do not plan or run deployment, staging, release, smoke, or live external-system operations unless the user explicitly expands the scope outside Team SpecKit.

## Commits

Every participant shares the manager's current checkout and may create an intermediate commit for tracked changes made during its turn. A commit is part of the participant's handoff, not a separate phase or approval. Participants never push or create a pull request.

## Readiness

Before declaring ready, read the repository rules and run their permitted final local quality gate once on the final checkout when one is defined. Do not invent a gate for a repository that defines none. Fix and commit a small mechanical gate failure yourself only when it stays inside reviewed behavior; otherwise reactivate its owning author or worker. Repeat specialized review only when the fix changes reviewed behavior or material risk.

Report ready only when the final spec, plan, tasks, code, tests, and documentation agree; specification, plan, and every implementation phase have a current ready disposition; and every required local proof passes. Summarize delivered behavior, proof, participant thread outcomes, commits, remaining risks, and externally blocked evidence.
