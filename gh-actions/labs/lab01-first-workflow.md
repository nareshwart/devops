# GitHub Actions — First Workflow & UI Inspection Lab

---

<details>
<summary><strong>Lab Objectives &amp; Environment Setup</strong></summary>

<br>

### Objectives
- Create the mandatory `.github/workflows/` directory structure.
- Write your first declarative workflow file with multiple event triggers (`push` and `workflow_dispatch`).
- Inspect runner host facts (OS version, compute specs, available disk, user permissions).
- Trigger workflows manually with custom inputs from the GitHub UI.
- Generate rich markdown run summaries using the `$GITHUB_STEP_SUMMARY` environment file.
- Inspect real-time execution logs, timestamps, and error traces.

### Prerequisites
- A GitHub repository (public or private) with permissions to push commits.
- Git installed locally.

</details>

---

<details>
<summary><strong>Step 1: Create the Workflows Directory</strong></summary>

<br>

In the root of your local Git repository, create the hidden `.github/workflows` directory:

```bash
mkdir -p .github/workflows
```

> [!IMPORTANT]
> The folder path **must be exact**: `.github/workflows/` (with a leading dot before github, and plural workflows). Any other path will be ignored by GitHub.

</details>

---

<details>
<summary><strong>Step 2: Create Your First Workflow File (`hello-world.yml`)</strong></summary>

<br>

Create `.github/workflows/hello-world.yml`:

```bash
cat << 'EOF' > .github/workflows/hello-world.yml
name: Hello DevOps Actions

# Dynamic name shown in the GitHub Actions dashboard
run-name: Run by @${{ github.actor }} on ${{ github.ref_name }}

on:
  # Triggers when code is pushed to main
  push:
    branches: [ main ]

  # Enables manual execution button in the GitHub Actions tab
  workflow_dispatch:
    inputs:
      target_greeting:
        description: 'Who to greet?'
        required: true
        default: 'DevOps Engineers'
        type: string

jobs:
  explore_runner:
    name: Inspect Runner & Environment
    runs-on: ubuntu-latest

    steps:
      - name: Greet the User
        run: |
          echo "Hello, ${{ github.event.inputs.target_greeting || 'DevOps World' }}!"
          echo "Triggered by event: ${{ github.event_name }}"

      - name: Inspect Runner Hardware & Operating System
        run: |
          echo "=== Operating System Information ==="
          uname -a
          cat /etc/os-release | grep PRETTY_NAME

          echo "=== Runner User & Sudo Access ==="
          whoami
          sudo whoami

          echo "=== CPU & RAM Information ==="
          nproc && echo "vCPU cores"
          free -h

          echo "=== Disk Space Available ==="
          df -h /

      - name: Generate Rich Run Summary
        run: |
          echo "### 🚀 Execution Report" >> "$GITHUB_STEP_SUMMARY"
          echo "| Metric | Value |" >> "$GITHUB_STEP_SUMMARY"
          echo "|---|---|" >> "$GITHUB_STEP_SUMMARY"
          echo "| **Triggered By** | @${{ github.actor }} |" >> "$GITHUB_STEP_SUMMARY"
          echo "| **Branch** | \`${{ github.ref_name }}\` |" >> "$GITHUB_STEP_SUMMARY"
          echo "| **Commit SHA** | \`${{ github.sha }}\` |" >> "$GITHUB_STEP_SUMMARY"
          echo "| **Runner OS** | ${{ runner.os }} (${{ runner.arch }}) |" >> "$GITHUB_STEP_SUMMARY"
EOF
```

</details>

---

<details>
<summary><strong>Step 3: Commit, Push &amp; Watch the Pipeline Run</strong></summary>

<br>

Add, commit, and push your new workflow to GitHub:

```bash
git add .github/workflows/hello-world.yml
git commit -m "feat: add first github actions workflow"
git push origin main
```

### Viewing the Live Run in GitHub:
1. Open your repository in your web browser.
2. Click on the **Actions** tab at the top.
3. You will see your newly triggered workflow: **"Run by @your-username on main"**.
4. Click on the run, then click on the **Inspect Runner & Environment** job.
5. Click each step arrow to expand the real-time logs and verify:
   - Sudo executed as `root` without prompting for password.
   - The runner provides 2 vCPUs and ~7GB RAM.
   - The GitHub Step Summary table is rendered at the bottom of the overview page.

</details>

---

<details>
<summary><strong>Step 4: Trigger the Workflow Manually via UI (`workflow_dispatch`)</strong></summary>

<br>

Because we included `workflow_dispatch:`, you can trigger this pipeline on-demand without making any Git commits!

1. Navigate to the **Actions** tab on GitHub.
2. On the left sidebar under "Workflows", click **Hello DevOps Actions**.
3. Notice the blue banner that appears: **"This workflow has a workflow_dispatch event trigger."**
4. Click the **Run workflow** dropdown button.
5. In the input box **"Who to greet?"**, enter a custom name (e.g. `Platform SRE Team`).
6. Click the green **Run workflow** button.
7. Refresh after 3 seconds, click into the new run, and verify that Step 1 outputs:
   `Hello, Platform SRE Team!`

</details>

---

<details>
<summary><strong>Summary Checklist: Lab 00</strong></summary>

<br>

- [x] Initialized `.github/workflows/` in the repository root.
- [x] Created `hello-world.yml` with dual triggers (`push` and `workflow_dispatch`).
- [x] Inspected the underlying Ubuntu VM compute specs (vCPUs, RAM, disk).
- [x] Verified passwordless `sudo` execution on GitHub-hosted runners.
- [x] Dispatched on-demand workflow runs with custom inputs via the GitHub UI.
- [x] Published markdown execution dashboards using `$GITHUB_STEP_SUMMARY`.

</details>

