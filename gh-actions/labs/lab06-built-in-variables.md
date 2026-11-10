# Lab 06: Use Built-In GitHub Actions Variables

## Goal

Use built-in GitHub Actions values to identify a workflow run, show repository and runner details, and run a step only on the `main` branch.

## Step 1: Create a workflow

Create `.github/workflows/built-in-variables.yaml`:

```yaml
name: Built-In Variables
run-name: "Run ${{ github.run_number }} by ${{ github.actor }}"

on:
  push:
  workflow_dispatch:

jobs:
  show-info:
    runs-on: ubuntu-latest
    steps:
      - name: Show run and repository information
        run: |
          echo "Repository: $GITHUB_REPOSITORY"
          echo "Branch or tag: $GITHUB_REF_NAME"
          echo "Commit: $GITHUB_SHA"
          echo "Triggered by: $GITHUB_ACTOR"
          echo "Event: $GITHUB_EVENT_NAME"
          echo "Run number: $GITHUB_RUN_NUMBER"

      - name: Show runner information
        run: |
          echo "Runner OS: $RUNNER_OS"
          echo "Runner architecture: $RUNNER_ARCH"
          echo "Workspace: $GITHUB_WORKSPACE"

      - name: Run only on main
        if: ${{ github.ref_name == 'main' }}
        run: echo "This step runs on the main branch."

      - name: Add a summary
        run: |
          echo "### Workflow run information" >> "$GITHUB_STEP_SUMMARY"
          echo "- Repository: \`$GITHUB_REPOSITORY\`" >> "$GITHUB_STEP_SUMMARY"
          echo "- Ref: \`$GITHUB_REF_NAME\`" >> "$GITHUB_STEP_SUMMARY"
          echo "- Commit: \`$GITHUB_SHA\`" >> "$GITHUB_STEP_SUMMARY"
          echo "- Runner: $RUNNER_OS ($RUNNER_ARCH)" >> "$GITHUB_STEP_SUMMARY"
```

## Step 2: Push and run manually

Commit and push the workflow:

```bash
git add .github/workflows/built-in-variables.yaml
git commit -m "Add built-in variables lab"
git push
```

The push triggers a run. You can also open the repository's **Actions** tab, select **Built-In Variables**, and click **Run workflow** to start another run manually.

## Step 3: Inspect the values

1. Open a run and select the `show-info` job.
2. Expand **Show run and repository information** and **Show runner information**.
3. Check how the branch, commit, event, run number, and runner details change between runs.
4. Open the run summary and view the generated Markdown.
5. On a run for `main`, confirm **Run only on main** executes. On a different branch, that step is skipped.

## What these values are useful for

| Value | Example use |
| :--- | :--- |
| `GITHUB_REPOSITORY` | Identify which repository is being built. |
| `GITHUB_REF_NAME` | Show the branch or tag, or apply branch-specific logic. |
| `GITHUB_SHA` | Record the exact commit used for a build. |
| `GITHUB_ACTOR` | Display who started the run. |
| `GITHUB_EVENT_NAME` | Use different behavior for push versus manual runs. |
| `GITHUB_RUN_NUMBER` | Label a build or make a run easy to find. |
| `RUNNER_OS` and `RUNNER_ARCH` | Diagnose operating-system or architecture-specific issues. |
| `GITHUB_WORKSPACE` | Locate the checked-out project files on the runner. |
| `GITHUB_STEP_SUMMARY` | Add a short report to the workflow run page. |

These are runner environment variables, so use shell syntax such as `$GITHUB_SHA` inside `run:`. In workflow expressions such as `if:`, use contexts such as `${{ github.ref_name }}`.
