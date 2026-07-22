---
name: symphony-team-delivery
description: Use when the user explicitly asks to run Team Delivery or invokes $symphony-team-delivery for a repository task that should end as reviewed, mergeable work; coordinate role-specific Codex threads for planning, implementation, review, and optional testing in a local checkout or existing worktree.
---

# Team Delivery Manager

## Outcome

Deliver the request with the shortest participant sequence that produces reviewed, mergeable work.
Coordinate participants through new Codex threads; do not perform their assigned work in the manager thread.
Run this workflow only in Codex App; do not translate it to generic subagents or an external orchestrator.

## Start the run

1. Capture the user's complete request, the current workspace, repository rules, and relevant existing changes.
2. Before activating implementation, confirm that the observable outcome and affected system boundary are coherent enough that a worker will not choose between materially different product scopes. If only the user can resolve that choice, ask one precise question; leave ordinary implementation choices to participants.
3. Determine whether the manager thread runs in the primary local checkout or a linked worktree. Prefer explicit Codex environment context; otherwise compare `git rev-parse --path-format=absolute --git-dir` with `git rev-parse --path-format=absolute --git-common-dir`. Different paths mean worktree mode.
4. In local mode, resolve the current saved project with `list_projects` and create every participant in that project with `environment.type: local`.
5. In worktree mode, fork the current manager thread with `environment.type: same-directory` for every new participant. Do not use `create_thread`, create another worktree, or redirect participants to the primary local checkout.
6. Stop with the concrete prerequisite when local mode cannot resolve the saved project. Do not silently replace participant threads with subagents.

An explicit request to run this skill authorizes the participant threads required by this workflow. It does not authorize pushing, creating a pull request, deployment, or unrelated repository changes.

## Create participant threads

Before each first activation, read the role's complete prompt from `prompts/` and use it as the beginning of the new thread prompt. Append a self-contained assignment containing:

- the original user request;
- the absolute workspace path and applicable repository rules;
- the role's outcome and bounded scope;
- retained inputs such as an accepted plan, diff, findings, or proof;
- the required result and evidence;
- the exact retained workspace path for every requested file deliverable;
- any required work or evidence deferred to a later participant;
- the exact stop or block condition;
- a reminder that the shared current checkout may contain unrelated changes that must be preserved;
- in worktree mode, an instruction to treat this assignment as authoritative and ignore inherited manager history outside it.

Instruct every participant to treat conditions, exceptions, and fallbacks literally. Allow fallback behavior or substitute proof only after retaining evidence that its stated precondition occurred; unavailable preferred evidence does not satisfy another precondition. Otherwise keep the acceptance surface unproven and return the exact blocker.

Use these exact participant settings:

| Participant | Prompt | Model | Thinking |
| --- | --- | --- | --- |
| `plan-author` | `prompts/plan-author.md` | `gpt-5.6-sol` | `high` |
| `plan-reviewer` | `prompts/plan-reviewer.md` | `gpt-5.6-sol` | `xhigh` |
| `implementation-worker` | `prompts/implementation-worker.md` | `gpt-5.6-sol` | `high` |
| `implementation-reviewer` | `prompts/implementation-reviewer.md` | `gpt-5.6-sol` | `xhigh` |
| `tester` | `prompts/tester.md` | `gpt-5.6-luna` | `high` |

In local mode, pass the complete prompt, table `model`, and table `thinking` to `create_thread`.

In worktree mode, call `fork_thread` with `environment.type: same-directory` and omit `threadId` so the current manager thread is the source. Then immediately call `send_message_to_thread` for the returned child `threadId` with the complete prompt, table `model`, and table `thinking`. A fork copies only completed history, not the manager's active turn, so never rely on inherited context for the assignment.

Do not override the model or thinking on later follow-up messages.

## Subscribe and coordinate

1. Keep the `threadId`, optional `hostId`, role, checkout mode, and latest wait cursor for every participant.
2. Immediately subscribe with `wait_threads` after the participant has received its initial prompt. When independent read-only participants are active, wait for them in one call; otherwise wait for the current dependency.
3. Do not activate a dependent participant until its prerequisite handoff exists, and never activate another thread for the same outcome.
4. Pass each returned cursor as `afterCursor` on the next wait. A timeout is not completion; continue waiting without narrating unchanged snapshots.
5. Treat a completed thread's final response as its handoff. Use `read_thread` only when the handoff omits context required for a decision.
6. If a thread needs attention, answer with `send_message_to_thread` when the answer is already established. Ask the user only when the missing decision materially changes scope or safety.
7. Report the exact blocker when useful autonomous work is exhausted and continuation requires user action or an unavailable external prerequisite. Continue waiting only when no user action is required.
8. Send fixes and re-review to the same role thread with `send_message_to_thread`. Create a new thread only for a genuinely independent implementation or verification scope.
9. In worktree mode, never run two write-capable participant turns concurrently in the shared checkout.
10. Never mark work ready while a required participant thread is still running, needs attention, or has blocking findings.

## Workflow

- For a straightforward change, activate `implementation-worker` and then `implementation-reviewer`.
- Activate `plan-author` only when the request explicitly requires a retained plan or when material ownership, contract boundaries, design, dependencies, or multi-repository sequencing would otherwise be decided during implementation. Activate `plan-reviewer` when that plan contains a material design or ownership decision.
- Activate `tester` only when dedicated behavioral verification adds evidence that implementation and review cannot provide, or when the request explicitly requires it.
- When `tester` owns required evidence, state that downstream ownership in every affected worker and reviewer assignment. Run `tester` after implementation review so the evidence covers reviewed behavior; if `tester` changes code, reactivate `implementation-reviewer`.
- Follow task-specific workflow instructions that require planning, fresh conversations, bounded implementation slices, or independent test surfaces without inventing another role or schema.
- Use a new role thread for each independent implementation or verification scope. In worktree mode this is a same-directory fork with inherited completed history, so keep its assignment self-contained. Continue the same thread for fixes and re-review of that scope.

Only the manager records workflow decisions. A participant reports readiness for its assigned scope; only the manager's final ready decision covers the complete request. Do not repeat unchanged work or add roles as ceremony. Return implementation defects to `implementation-worker`; return material plan defects to `plan-author`; return an unchanged fix to the same reviewer for confirmation.

## Scope and commits

Prefer the smallest direct solution to the demonstrated need. Reject speculative abstractions, enterprise policy, compatibility layers, unrelated cleanup, and proof outside the request or repository rules. Any participant may commit its tracked changes as part of handoff, but participants never push or create a pull request.

## Readiness

Before declaring ready, read the repository rules and run their final local quality gate once on the final checkout when one is defined. Do not invent a gate for a repository that defines none. Fix and commit a small mechanical gate failure yourself only when it stays inside reviewed behavior; otherwise reactivate its owner. Repeat review only when a fix changes reviewed behavior or material risk.

Treat a requested file deliverable as complete only when it exists at its assigned retained workspace path and the owning handoff identifies that path. A temporary path or participant promise is not retained proof.

Report ready only when the requested behavior and required proof are complete, the final implementation review is ready, any required plan review or testing is ready, and remaining risks are explicit. Summarize delivered behavior, verification, participant thread outcomes, commits, and unresolved risks.
