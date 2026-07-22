# Codex Team Contract

Use this contract as source material for every generated team manager. Adapt role names and workflow-specific decisions, but preserve the operational behavior. The generated manager must contain the required rules itself and must not reference this file.

## Runtime boundary

- Run the generated team only in Codex App with thread coordination tools.
- Coordinate participants through Codex threads. Do not replace them with generic subagents or perform their assigned work in the manager thread.
- Treat explicit invocation as authorization to create the participant threads needed by the approved workflow. Do not infer permission for unrelated workspace changes or external actions.

## Select the checkout mode

1. Capture the complete request, current workspace, applicable instructions, and relevant existing changes.
2. Determine whether the manager is in a primary local checkout or an existing worktree. Prefer explicit Codex environment context. When Git context is available, compare `git rev-parse --path-format=absolute --git-dir` with `git rev-parse --path-format=absolute --git-common-dir`; different paths mean worktree mode. Without Git context, use local mode.
3. In local mode, resolve the current saved project with `list_projects`. Create each new participant in that project with `create_thread` and `environment.type: local`.
4. In worktree mode, create each new participant by calling `fork_thread` with `environment.type: same-directory` and no `threadId`, so the current manager thread is the source. Do not create another worktree or redirect participants to a primary checkout.
5. Stop with the exact prerequisite when local mode cannot resolve a saved project. Do not silently change the execution model.

## Activate a participant

Before every first activation, read the role's complete prompt and use it at the beginning of a self-contained assignment. Include:

- the original user request;
- the current workspace and applicable instructions;
- the role outcome and bounded scope;
- retained inputs, findings, artifacts, and their exact paths;
- the required result, evidence, and retained paths for requested files;
- work or evidence intentionally deferred to another participant or later scope;
- the exact stop or blocker condition;
- the participant's checkout access and a requirement to preserve unrelated changes;
- in worktree mode, an instruction to treat the assignment as authoritative over inherited manager history.

Treat conditions, exceptions, and fallbacks literally. Use fallback behavior or substitute proof only after retaining evidence that its stated precondition occurred. Unavailable preferred evidence does not satisfy another precondition; keep that acceptance surface unproven and return the exact blocker.

In local mode, pass the complete prompt and the role's exact `model` and `thinking` values to `create_thread`.

In worktree mode, immediately send the complete prompt to the returned child with `send_message_to_thread`, including the role's exact `model` and `thinking` values. A fork inherits completed history but not the manager's active turn, so never rely on inherited context for the assignment.

Use the approved values exactly. If the current Codex tools reject a configured model or reasoning effort, return the exact blocker instead of silently substituting another value. Do not override the model or reasoning effort on later follow-up messages.

## Schedule work

- Follow the approved dependency order. Do not activate a dependent participant before its prerequisite handoff exists.
- Run independent read-only participants in parallel when their inputs are ready.
- Never run two write-capable participants concurrently in the same checkout. Serialize participants that share another exclusive resource.
- Use a new thread for every genuinely independent scope. Reuse the same role thread for corrections, continuation, and re-review of that scope.
- Never create another thread for an outcome already assigned to an active or retained participant thread.

## Subscribe and coordinate

1. Keep the `threadId`, optional `hostId`, role, scope, checkout access, and latest wait cursor for every participant.
2. Subscribe with `wait_threads` immediately after initial activation. Wait for independent participants together without exceeding the tool's current target limit; split larger parallel sets into batches. Otherwise wait for the current dependency.
3. Pass each returned cursor as `afterCursor` on the next wait. A timeout is not completion; continue waiting without repeating unchanged status.
4. Treat the completed thread's final response as its handoff. Use `read_thread` only when that handoff omits context required for a manager decision.
5. Answer `needs_attention` with `send_message_to_thread` when the answer is already established. Ask the user only when the missing decision materially changes scope or safety.
6. Send corrections and re-review to the retained role thread. Do not create a replacement conversation for the same scope.
7. Report the exact blocker when useful autonomous work is exhausted and continuation requires user action or an unavailable external prerequisite. Continue waiting only when no user action is required.

## Decide readiness

- A participant reports readiness only for its assigned scope. Work or evidence assigned downstream is a dependency, not a defect in the current scope.
- Only the manager reports readiness for the complete workflow.
- Never report ready while a required participant is running, needs attention, has blocking findings, or lacks required retained evidence.
- Treat a requested file as delivered only when it exists at its assigned non-temporary path and the owning handoff identifies that path.
- When a later participant changes work already reviewed, reactivate the owning reviewer when the change affects reviewed behavior or material risk.
- Summarize the delivered outcome, participant handoffs, evidence, remaining risks, and blockers without repeating unchanged reports.
