# Lab 04: Use Pre-Built GitHub Actions

## Goal

Use actions maintained by GitHub to check out a repository, set up Python, and save a file as a workflow artifact.

## Step 1: Create a workflow

Create `.github/workflows/prebuilt-actions.yaml`:

```yaml
name: Pre-Built Actions

on:
  workflow_dispatch:

jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - name: Check out the repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Run a small Python command
        run: python -c "print('Python is ready')"

      - name: Create a file to save
        run: echo "Created by the workflow" > result.txt

      - name: Upload the file as an artifact
        uses: actions/upload-artifact@v4
        with:
          name: workflow-result
          path: result.txt
```

In this workflow, `uses:` runs a pre-built action, while `run:` executes a shell command.

## Step 2: Run the workflow

Commit and push the workflow file:

```bash
git add .github/workflows/prebuilt-actions.yaml
git commit -m "Add pre-built actions lab"
git push
```

On GitHub:

1. Open the repository's **Actions** tab.
2. Select **Pre-Built Actions** and click **Run workflow**.
3. Open the run and select the `example` job.
4. Expand each step and check that the checkout, Python setup, command, and upload steps succeed.

## Step 3: Download the artifact

Open the completed run's summary page. Under **Artifacts**, download `workflow-result` and check that it contains `result.txt`.

## Summary

- `actions/checkout` makes repository files available to the job.
- `actions/setup-python` selects a Python version.
- `actions/upload-artifact` saves a file from the run.
- Check an action's documentation for its inputs, outputs, and supported versions.
