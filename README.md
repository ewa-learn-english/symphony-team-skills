# Symphony Team Skills

Two standalone, Codex-only skills that run software-delivery workflows through coordinated Codex App threads.

The skills are adapted from the Team Delivery and Team SpecKit role prompts. They do not require, call, or integrate with the Symphony runtime.

## Included skills

| Skill | Workflow |
| --- | --- |
| `symphony-team-delivery` | Selective planning, implementation, implementation review, and optional dedicated testing. |
| `symphony-team-speckit` | Specification, spec review, planning, plan review, tasks, and reviewed implementation phases. |

Each `SKILL.md` is the team manager. Role prompts live in the adjacent `prompts/` directory. The manager creates every first role activation as a separate Codex thread, waits for completion with `wait_threads`, and sends fixes or re-review back to the same thread.

## Requirements

- Codex App with Codex thread coordination tools.
- The target repository saved as a Codex project. Participant threads use its local environment and share the current checkout.
- Access to the models configured by the selected skill: `gpt-5.6-sol`, `gpt-5.6-terra`, and `gpt-5.6-luna`, with the recorded `high` or `xhigh` thinking levels.
- For `symphony-team-speckit`, a working SpecKit installation in the target repository.

These workflows intentionally stay local. Participants may create commits when useful, but they do not push, create pull requests, deploy, or change unrelated work.

## Install

Ask Codex to install either GitHub directory with `$skill-installer`:

```text
Use $skill-installer to install https://github.com/ewa-learn-english/symphony-team-skills/tree/main/symphony-team-delivery
```

```text
Use $skill-installer to install https://github.com/ewa-learn-english/symphony-team-skills/tree/main/symphony-team-speckit
```

Alternatively, clone the repository and link both skill folders into the user skill directory:

```bash
git clone https://github.com/ewa-learn-english/symphony-team-skills.git "$HOME/.agents/symphony-team-skills"
mkdir -p "$HOME/.agents/skills"
ln -s "$HOME/.agents/symphony-team-skills/symphony-team-delivery" "$HOME/.agents/skills/symphony-team-delivery"
ln -s "$HOME/.agents/symphony-team-skills/symphony-team-speckit" "$HOME/.agents/skills/symphony-team-speckit"
```

Codex detects skill changes automatically. Restart Codex if the skills do not appear. See the official [Codex skills documentation](https://developers.openai.com/codex/skills) for discovery locations and invocation behavior.

To update a linked installation:

```bash
git -C "$HOME/.agents/symphony-team-skills" pull --ff-only
```

## Use

Open the target repository in Codex App and explicitly invoke the appropriate manager:

```text
Use $symphony-team-delivery to implement and review this repository task: <request>
```

```text
Use $symphony-team-speckit to deliver this feature through SpecKit: <request>
```

Explicit invocation authorizes the manager to create the participant threads required by that workflow. It does not expand permission to publish or deploy changes.

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
```

## License

[MIT](LICENSE)
