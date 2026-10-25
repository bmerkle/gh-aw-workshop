<!-- page-journey: all -->
<!-- page-adventure: advanced -->
<!--
<research-metadata>
  <focus>Per-user rate limiting for event-driven workflows</focus>
  <sources>
    <source>https://github.github.com/gh-aw/reference/rate-limiting-controls/</source>
  </sources>
  <rationale>Steps 14b and 26 teach event-driven triggers and cost caps, but no step covers how to stop one person from triggering a workflow over and over. The gh-aw `user-rate-limit` field closes that gap.</rationale>
</research-metadata>
-->

# Stop Trigger Floods with Per-User Rate Limits

> _An event-driven workflow that anyone can trigger is also a workflow anyone can overuse — a per-user limit keeps one person from burning your budget._

## :dart: What You'll Do

You'll add a [`user-rate-limit`](https://github.github.com/gh-aw/reference/rate-limiting-controls/#rate-limiting-per-user) block to an event-driven workflow, so each person can start only a few runs per hour. When you finish, you'll know how to cap runs per person and which roles skip the cap.

## :clipboard: Before You Start

- You completed [Build a PR Reviewer with an Agent and Skill](14b-pr-reviewer-workflow.md) or have any workflow triggered by `slash_command`, `issue_comment`, or `workflow_dispatch`.
- You completed [Manage Costs and AI Credit Budgets](26-manage-costs-and-budgets.md), or you understand why a workflow needs a spend cap.

## Steps

### Understand the problem

Scheduled workflows run on your clock. Event-driven workflows run on someone else's. If a slash command starts an AI run, a person who types it fifty times starts fifty runs. Each run uses AI Credits (AIC), the unit that measures agent spend.

The `max-daily-ai-credits` field caps total spend across all runs. A **per-user rate limit** is different: it caps how often one person can trigger runs.

### Add the rate limit

Open your event-driven workflow in the Codespace. Add `user-rate-limit` to the frontmatter:

```yaml
---
on:
  slash_command:
    name: review
    events: [pull_request_comment]
permissions:
  contents: read
  pull-requests: read
safe-outputs:
  add-comment:
user-rate-limit:
  max-runs-per-window: 3
  window: 60
---

# Review Helper

Review the pull request and post one comment with your findings.
```

Here is what each field does:

| Field | Meaning |
|-------|---------|
| `max-runs-per-window` | Most runs one person can start per window (1–10, required) |
| `window` | Length of the window in minutes (default 60, maximum 180) |
| `events` | Optional list of events to count; inferred from `on:` when omitted |
| `ignored-roles` | Roles that skip the limit; defaults to `admin`, `maintain`, and `write` |

### Compile the workflow

```bash
gh aw compile
```

Recompiling adds a check to the pre-activation job. Before the agent starts, that check counts the person's recent runs and cancels the new run if they are over the limit.

### Decide who is exempt

By default, people with `write` access or higher skip the limit. That suits most teams, because collaborators are trusted. To limit everyone, including maintainers, empty the list:

```yaml
user-rate-limit:
  max-runs-per-window: 3
  ignored-roles: []
```

> [!TIP]
> Keep the default exemptions for private team repositories. Empty the list when outside contributors can trigger the workflow, such as in a public repository.

### Test the limit

1. Commit and push the `.md` and `.lock.yml` files.
2. Trigger the workflow more times than `max-runs-per-window` allows, using an account the limit applies to.
3. Open the **Actions** tab and find the cancelled run.
4. Open the pre-activation job and confirm it reports the rate limit as the reason.

> [!NOTE]
> Manual `workflow_dispatch` runs from a user with `write` access are exempt by default, so test with a non-exempt account or set `ignored-roles: []` temporarily.

## :white_check_mark: Checkpoint

- [ ] Your workflow frontmatter includes `user-rate-limit` with a `max-runs-per-window` value between 1 and 10
- [ ] You can explain how a per-user limit differs from `max-daily-ai-credits`
- [ ] You recompiled with `gh aw compile` and committed both the `.md` and `.lock.yml` files
- [ ] You identified which roles are exempt by default and decided whether to change `ignored-roles`
- [ ] You found a cancelled run in the **Actions** tab, or you can describe what one looks like

<!-- journey: all -->
Want to choose another branch from the workshop hub? Return to [What's Next? Keep Exploring](14-next-steps.md).
<!-- /journey -->
