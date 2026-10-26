<!-- page-journey: all -->
<!-- page-adventure: advanced -->
<!--
<research-metadata>
  <focus>Concurrency control — keeping overlapping runs of agentic workflows from colliding</focus>
  <sources>
    <source>https://github.github.com/gh-aw/reference/concurrency/</source>
    <source>https://github.github.com/gh-aw/reference/safe-outputs/</source>
  </sources>
  <rationale>The curriculum covers triggers, safe outputs, staged mode, and cost controls, but never explains what happens when two runs start at once. Rapid pushes, busy repositories, and shared Copilot capacity all create overlapping runs. This step shows learners the default per-workflow and per-engine groups, then has them override one and serialize safe-output writes.</rationale>
</research-metadata>
-->

# Control Overlapping Runs with Concurrency

> _Decide what happens when a second run starts before the first one finishes._

## :dart: What You'll Do

You'll read the concurrency group that `gh aw compile` generates for one of your workflows, then override it with your own group and serialize its safe-output writes. When you finish, you'll know how to stop duplicate runs from posting duplicate comments or pull requests.

## :clipboard: Before You Start

- You completed [Preview Your Workflow's Writes with Staged Mode](30-preview-with-staged-mode.md), or you have any workflow with a `safe-outputs:` block.
- You can compile workflows with `gh aw compile`.

## Steps

### Find the default group

[Concurrency](https://github.github.com/gh-aw/reference/concurrency/) is a GitHub Actions setting that puts runs into named groups. Runs in the same group don't execute at the same time. gh-aw adds groups for you at two levels:

- **Per workflow:** the group includes the workflow name plus the issue, pull request, or branch. Different pull requests run side by side. For pull requests, a new commit cancels the outdated run.
- **Per engine:** `gh-aw-copilot` (or your engine's ID) allows only one agent job per engine across all workflows.

1. In the Codespace terminal, compile your workflow:

   ```bash
   gh aw compile
   ```

2. Open the generated `.lock.yml` file and search for `concurrency:`.
3. Find the `group:` value at the top of the file and the one under the `agent` job. Match each to the levels above.

> [!NOTE]
> For triggers that don't cancel, gh-aw emits `queue: max`. Rapid pushes line up and run in order instead of being dropped.

### Override the workflow group

4. Add a `concurrency:` block to your workflow frontmatter. This example shares one group per branch and cancels older runs:

   ```yaml
   ---
   on:
     push:
       branches: [main]
   permissions:
     contents: read
   concurrency:
     group: review-${{ github.ref }}
     cancel-in-progress: true
   safe-outputs:
     add-comment:
   ---

   # Review pushes

   Summarize what changed in the latest push.
   ```

5. Recompile and confirm the new group appears in the lock file:

   ```bash
   gh aw compile
   ```

### Serialize safe-output writes

The agent job and the `safe_outputs` job run separately, so two runs can write at the same moment. Add `concurrency-group` so writes wait their turn:

6. Under `safe-outputs:`, add the group:

   ```yaml
   safe-outputs:
     concurrency-group: "safe-outputs-${{ github.repository }}"
     add-comment:
   ```

7. Recompile, then commit both the `.md` and `.lock.yml` files.

> [!TIP]
> Don't combine `cancel-in-progress: true` with `queue: max`. GitHub Actions rejects that pair. Pick cancelling for "only the latest matters" and queueing for "every run matters".

## :white_check_mark: Checkpoint

- [ ] You found both the workflow-level and engine-level `concurrency:` groups in a compiled `.lock.yml` file
- [ ] You can explain why pull request runs cancel but scheduled runs queue
- [ ] Your workflow frontmatter sets a custom `concurrency:` group and the lock file shows it
- [ ] Your `safe-outputs:` block sets `concurrency-group`, and you committed the `.md` and `.lock.yml` files

<!-- journey: all -->
Want to choose another branch from the workshop hub? Return to [What's Next? Keep Exploring](14-next-steps.md).
<!-- /journey -->
