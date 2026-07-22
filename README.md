# Symphony Team Skills

Standalone, Codex-only skills for designing and running workflows through coordinated Codex App threads.

Team Delivery and Team SpecKit are adapted from their corresponding role prompts. The creator is a generic team designer. None of the skills require, call, or integrate with the Symphony runtime.

## Included skills

| Skill | Workflow |
| --- | --- |
| `symphony-team-creator` | Draft, install, or update a custom team after explicit user approval. |
| `symphony-team-delivery` | Selective planning, implementation, implementation review, and optional dedicated testing. |
| `symphony-team-speckit` | Specification, spec review, planning, plan review, tasks, and reviewed implementation phases. |

The creator proposes a team name, roles, model and reasoning settings, dependencies, and readiness contract before writing anything. For updates it first inspects the installed team and presents the exact delta and resulting team. Only explicit approval allows it to create or modify a self-contained team skill in the standard user skills directory.

In generated and included teams, `SKILL.md` is the manager and role prompts live in `prompts/`. The manager creates every first role activation as a separate Codex thread, keeps it in the current checkout, waits for completion with `wait_threads`, and sends fixes or re-review back to the same thread.

## Requirements

- Codex with filesystem access to design and install a team with `symphony-team-creator`.
- Codex App with thread coordination tools to run a team.
- In local mode, the target repository saved as a Codex project.
- In worktree mode, the manager thread already running in the worktree that should receive the changes.
- Access to the exact models and reasoning efforts configured by the selected or generated team. The included Delivery and SpecKit teams currently use `gpt-5.6-sol`, `gpt-5.6-terra`, and `gpt-5.6-luna`.
- For `symphony-team-speckit`, a working SpecKit installation in the target repository.

Generated teams follow the action boundaries approved in their draft. Participants in the included Delivery and SpecKit teams may create commits when useful, but they do not push, create pull requests, deploy, or change unrelated work.

## Install

Ask Codex to install a GitHub directory with `$skill-installer`:

```text
Use $skill-installer to install https://github.com/ewa-learn-english/symphony-team-skills/tree/main/symphony-team-creator
```

```text
Use $skill-installer to install https://github.com/ewa-learn-english/symphony-team-skills/tree/main/symphony-team-delivery
```

```text
Use $skill-installer to install https://github.com/ewa-learn-english/symphony-team-skills/tree/main/symphony-team-speckit
```

The installed skill becomes available on the next Codex turn. Restart Codex if it does not appear. See the official [Codex skills documentation](https://developers.openai.com/codex/skills) for discovery locations and invocation behavior.

## Use

Invoke the creator anywhere Codex can write to the user skills directory:

```text
Use $symphony-team-creator to design a Codex team for this workflow: <description>
```

```text
Use $symphony-team-creator to update $symphony-team-example as follows: <description>
```

Open a target project in Codex App before invoking a team manager:

```text
Use $symphony-team-delivery to implement and review this repository task: <request>
```

```text
Use $symphony-team-speckit to deliver this feature through SpecKit: <request>
```

The creator first returns a draft and waits for explicit approval before creating or updating a team. Invoking a team authorizes its manager to create the participant threads required by that workflow; it does not expand permission to unrelated or external actions.

## Checkout behavior

| Manager environment | Participant creation | Shared checkout |
| --- | --- | --- |
| Primary local checkout | `create_thread` with `environment.type: local` | Saved project's local checkout |
| Existing Codex worktree | `fork_thread` with `environment.type: same-directory`, then a role prompt through `send_message_to_thread` | The manager's exact worktree |

Same-directory forks inherit the manager thread's completed history but not its active turn. Managers therefore send a complete, authoritative role assignment after every fork. Write-capable participants run sequentially in a shared worktree.

## Workflow notes

### Team Delivery

The manager uses the shortest sequence warranted by the task:

1. Add plan author and reviewer only when material design or ownership decisions require a plan.
2. Run an implementation worker.
3. Run an independent implementation reviewer.
4. Add a tester only when dedicated behavioral evidence is required.

### Team SpecKit

The manager preserves the complete staged workflow:

1. Specification and spec review.
2. Plan and plan review.
3. Dependency-ordered tasks grouped into coherent phases.
4. A fresh implementation worker and reviewer pair for each phase.
5. Final agreement across the SpecKit artifacts, implementation, and local proof.

## Validate

Validate each package with the `quick_validate.py` script bundled with Codex's `skill-creator` skill:

```bash
python3 /path/to/skill-creator/scripts/quick_validate.py symphony-team-delivery
python3 /path/to/skill-creator/scripts/quick_validate.py symphony-team-speckit
python3 /path/to/skill-creator/scripts/quick_validate.py symphony-team-creator
```

## License

[MIT](LICENSE)
