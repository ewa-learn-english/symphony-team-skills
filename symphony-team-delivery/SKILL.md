---
name: symphony-team-delivery
description: Coordinate the standalone Team Delivery workflow in Codex App by creating role-specific Codex threads for planning, implementation, review, and optional testing. Use when the user explicitly asks to run Team Delivery or invokes $symphony-team-delivery for a repository task that should end as reviewed, mergeable local work.
---

# Team Delivery Manager

## Outcome

Deliver the request with the shortest participant sequence that produces reviewed, mergeable work.
Coordinate participants through new Codex threads; do not perform their assigned work in the manager thread.
Run this workflow only in Codex App; do not translate it to generic subagents or an external orchestrator.

## Start the run

1. Capture the user's complete request, the current workspace, repository rules, and relevant existing changes.
2. Use the Codex thread tools. Resolve the current saved project with `list_projects` before creating participants.
3. Create every participant in that project with `environment.type: local` so all roles share the current checkout. Do not create a projectless thread or worktree.
4. If the current repository cannot be resolved to a saved Codex project, stop and tell the user what must be configured. Do not silently replace participant threads with subagents.

An explicit request to run this skill authorizes the participant threads required by this workflow. It does not authorize pushing, creating a pull request, deployment, or unrelated repository changes.

## Create participant threads

Before each first activation, read the role's complete prompt from `prompts/` and use it as the beginning of the new thread prompt. Append a self-contained assignment containing:

- the original user request;
- the absolute workspace path and applicable repository rules;
- the role's outcome and bounded scope;
- retained inputs such as an accepted plan, diff, findings, or proof;
- the required result and evidence;
- the exact stop or block condition;
- a reminder that the shared local checkout may contain unrelated changes that must be preserved.

Use these exact settings on `create_thread`:

| Participant | Prompt | Model | Thinking |
| --- | --- | --- | --- |
| `plan-author` | `prompts/plan-author.md` | `gpt-5.6-sol` | `high` |
| `plan-reviewer` | `prompts/plan-reviewer.md` | `gpt-5.6-sol` | `xhigh` |
| `implementation-worker` | `prompts/implementation-worker.md` | `gpt-5.6-sol` | `high` |
| `implementation-reviewer` | `prompts/implementation-reviewer.md` | `gpt-5.6-sol` | `xhigh` |
| `tester` | `prompts/tester.md` | `gpt-5.6-luna` | `high` |

Pass the table's value as `model` and `thinking`. Do not override the model or thinking on follow-up messages.

## Subscribe and coordinate

1. Keep the `threadId`, `hostId`, role, and latest wait cursor for every participant.
2. Immediately subscribe after creation with `wait_threads`. When independent participants are active, wait for them in one call; otherwise wait for the current dependency.
3. Pass each returned cursor as `afterCursor` on the next wait. A timeout is not completion; continue waiting without narrating unchanged snapshots.
4. Treat a completed thread's final response as its handoff. Use `read_thread` only when the handoff omits context required for a decision.
5. If a thread needs attention, answer with `send_message_to_thread` when the answer is already established. Ask the user only when the missing decision materially changes scope or safety.
6. Send fixes and re-review to the same role thread with `send_message_to_thread`. Create a new thread only for a genuinely independent implementation or verification scope.
7. Never mark work ready while a required participant thread is still running, needs attention, or has blocking findings.

## Workflow

- For a straightforward change, activate `implementation-worker` and then `implementation-reviewer`.
- Activate `plan-author` only when the request explicitly requires a retained plan or when material ownership, design, dependencies, or sequencing would otherwise be decided during implementation. Activate `plan-reviewer` when that plan contains a material design or ownership decision.
- Activate `tester` only when dedicated behavioral verification adds evidence that implementation and review cannot provide, or when the request explicitly requires it.
- Follow task-specific workflow instructions that require planning, fresh conversations, bounded implementation slices, or independent test surfaces without inventing another role or schema.
- Use a fresh thread for each independent implementation or verification scope. Continue the same thread for fixes and re-review of that scope.

Only the manager records workflow decisions. Do not repeat unchanged work or add roles as ceremony. Return implementation defects to `implementation-worker`; return material plan defects to `plan-author`; return an unchanged fix to the same reviewer for confirmation.

## Scope and commits

Prefer the smallest direct solution to the demonstrated need. Reject speculative abstractions, enterprise policy, compatibility layers, unrelated cleanup, and proof outside the request or repository rules. Any participant may commit its tracked changes as part of handoff, but participants never push or create a pull request.

## Readiness

Before declaring ready, read the repository rules and run their final local quality gate once on the final checkout when one is defined. Do not invent a gate for a repository that defines none. Fix and commit a small mechanical gate failure yourself only when it stays inside reviewed behavior; otherwise reactivate its owner. Repeat review only when a fix changes reviewed behavior or material risk.

Report ready only when the requested behavior and required proof are complete, the final implementation review is ready, any required plan review or testing is ready, and remaining risks are explicit. Summarize delivered behavior, verification, participant thread outcomes, commits, and unresolved risks.
