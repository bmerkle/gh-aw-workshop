<!-- page-journey: all -->
<!-- page-adventure: advanced -->
<!--
<research-metadata>
  <focus>Concurrency controls — stopping overlapping runs of the same agentic workflow</focus>
  <sources>
    <source>https://github.github.com/gh-aw/reference/concurrency/</source>
    <source>https://github.github.com/gh-aw/reference/frontmatter/</source>
  </sources>
  <rationale>The curriculum covers triggers, safe outputs, costs, and staged mode, but no step explains what happens when a workflow is triggered twice at once. Overlapping runs duplicate comments and double AI spend. The gh-aw `concurrency:` frontmatter key reuses standard Actions syntax, so it fits the Actions-superset story.</rationale>
</research-metadata>
-->

# Stop Overlapping Runs with Concurrency Controls

> _Keep one run at a time so your agent doesn't post twice or spend twice._

## :dart: What You'll Do

You'll add a [`concurrency:` block](https://github.github.com/gh-aw/reference/concurrency/) to a workflow so that a second trigger waits for, or replaces, a run that is already going. When you finish, you'll know which setting to pick for scheduled reports and which to pick for pull request workflows.

Why this matters: every run costs AI credits and may write to GitHub. Two runs started a few seconds apart can post duplicate comments.

## :clipboard: Before You Start

- You completed [Preview Your Workflow's Writes with Staged Mode](30-preview-with-staged-mode.md), or you have any workflow that responds to `pull_request` or `workflow_dispatch`.
- You can compile workflows with `gh aw compile`.

## Steps

### Understand the two behaviors

A **concurrency group** is a name. GitHub Actions allows only one running job per group name. Anything else with the same name has to wait or be cancelled. `concurrency:` is standard Actions syntax, so it works the same way in agentic workflows.

| Setting | What happens to a new run | Good for |
|---------|---------------------------|----------|
| `cancel-in-progress: false` | It waits in the queue | Scheduled reports, where every run matters |
| `cancel-in-progress: true` | The older run is cancelled | Pull request reviews, where only the latest commit matters |

> [!NOTE]
> gh-aw adds a default concurrency group for you. Add your own `concurrency:` block only when you want different behavior.

### Add a group to a pull request workflow

Open `.github/workflows/pr-reviewer.md` (or any pull request workflow you built) and add `concurrency:` to the frontmatter:

```yaml
---
on:
  pull_request:
    types: [opened, synchronize]
permissions:
  contents: read
  pull-requests: read
concurrency:
  group: pr-review-${{ github.event.pull_request.number }}
  cancel-in-progress: true
safe-outputs:
  add-comment:
---
```

The group name includes the pull request number. Two different pull requests can review at the same time. Two pushes to the _same_ pull request can't, so the older review is cancelled.

### Recompile and test it

1. Recompile so the change takes effect:

   ```bash
   gh aw compile
   ```

2. Commit and push the `.md` and `.lock.yml` files.
3. Open a pull request, then push two commits quickly, one after the other.
4. Open the **Actions** tab and look at the runs for your workflow.

You should see the first run marked **Cancelled** and the second run completing.

### Queue instead of cancel

For a scheduled report, switch the setting so no run is lost:

```yaml
concurrency:
  group: daily-status
  cancel-in-progress: false
```

> [!TIP]
> Ask yourself: "If a newer run starts, is the older run's work still useful?" If yes, queue. If no, cancel.

## :white_check_mark: Checkpoint

- [ ] Your workflow frontmatter contains a `concurrency:` block with a `group` name
- [ ] The group name for the pull request workflow includes the pull request number
- [ ] You recompiled with `gh aw compile` and committed both the `.md` and `.lock.yml` files
- [ ] Two quick pushes produced one **Cancelled** run and one completed run in the **Actions** tab
- [ ] You can explain when to use `cancel-in-progress: false` instead of `true`

<!-- journey: all -->
Want to choose another branch from the workshop hub? Return to [What's Next? Keep Exploring](14-next-steps.md).
<!-- /journey -->
