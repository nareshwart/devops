# Lab 05: Custom Composite Actions & Reusable Workflows

---

<details>
<summary><strong>Lab Scenario &amp; Objectives</strong></summary>

<br>

### The Scenario
As your DevOps organization grows, duplicating setup scripts and deployment boilerplate across multiple workflows becomes an anti-pattern. In this lab, you will:
1. Build a custom **Composite Action** (`.github/actions/setup-app-environment/action.yml`) that standardizes system preparation into a single line.
2. Build a **Reusable Workflow** (`.github/workflows/reusable-deploy-template.yml`) using `workflow_call:` that encapsulates deployment logic.
3. Consume both modular components inside a master application workflow.

### Target Project Architecture
```text
my-repo/
├── .github/
│   ├── actions/
│   │   └── setup-app-environment/
│   │       └── action.yml                <-- Custom local Composite Action
│   └── workflows/
│       ├── reusable-deploy-template.yml  <-- Reusable Called Workflow (workflow_call)
│       └── app-master-pipeline.yml       <-- Main Caller Workflow
```

</details>

---

<details>
<summary><strong>Step 1: Create the Custom Composite Action (`action.yml`)</strong></summary>

<br>

Create the directory for your custom action:

```bash
mkdir -p .github/actions/setup-app-environment
```

Create `.github/actions/setup-app-environment/action.yml`:

```bash
cat << 'EOF' > .github/actions/setup-app-environment/action.yml
name: 'Setup App Environment'
description: 'Prepares system tools, prints diagnostic info, and configures cache'

inputs:
  app-name:
    description: 'Name of the application'
    required: true
    default: 'core-service'
  target-tier:
    description: 'Target tier (dev, staging, prod)'
    required: false
    default: 'dev'

outputs:
  setup-timestamp:
    description: 'ISO timestamp when setup finished'
    value: ${{ steps.record-time.outputs.timestamp }}

runs:
  using: 'composite'
  steps:
    - name: Display Application Banner
      shell: bash
      run: |
        echo "=========================================="
        echo "Configuring App: ${{ inputs.app-name }}"
        echo "Target Tier:     ${{ inputs.target-tier }}"
        echo "Host Runner:     ${{ runner.os }} (${{ runner.arch }})"
        echo "=========================================="

    - name: Ensure Tooling Directories
      shell: bash
      run: |
        mkdir -p /tmp/app-cache
        echo "Cache directory ready at /tmp/app-cache"

    - name: Record Setup Timestamp
      id: record-time
      shell: bash
      run: |
        NOW=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
        echo "timestamp=$NOW" >> "$GITHUB_OUTPUT"
        echo "Environment initialized at: $NOW"
EOF
```

> [!IMPORTANT]
> Notice the **`shell: bash`** declaration on each step! This is mandatory for composite actions because steps run directly on the host shell.

</details>

---

<details>
<summary><strong>Step 2: Create the Reusable Workflow (`reusable-deploy-template.yml`)</strong></summary>

<br>

Create `.github/workflows/reusable-deploy-template.yml`:

```bash
cat << 'EOF' > .github/workflows/reusable-deploy-template.yml
name: Reusable Deployment Engine

on:
  workflow_call:
    inputs:
      service_name:
        description: 'Microservice name'
        required: true
        type: string
      environment_name:
        description: 'Deployment target environment'
        required: true
        type: string
    secrets:
      AUTH_TOKEN:
        required: false

jobs:
  execute_deploy:
    name: Deploy ${{ inputs.service_name }} to ${{ inputs.environment_name }}
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Run Standardized Deployment Steps
        env:
          SERVICE: ${{ inputs.service_name }}
          TARGET_ENV: ${{ inputs.environment_name }}
          TOKEN: ${{ secrets.AUTH_TOKEN }}
        run: |
          echo "Connecting to $TARGET_ENV deployment gateway..."
          echo "Rolling out service: $SERVICE"
          if [ -n "$TOKEN" ]; then
            echo "Authenticated with token: $TOKEN" # Masked if secret
          else
            echo "No auth token required for $TARGET_ENV."
          fi
          echo "Health check: HTTP 200 OK"
          echo "Rollout completed successfully."
EOF
```

</details>

---

<details>
<summary><strong>Step 3: Call Both Components in Master Pipeline (`app-master-pipeline.yml`)</strong></summary>

<br>

Now create `.github/workflows/app-master-pipeline.yml` to consume both the local composite action and the reusable workflow:

```bash
cat << 'EOF' > .github/workflows/app-master-pipeline.yml
name: Master Application Pipeline

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  # ========================================================
  # JOB 1: Uses the Local Composite Action
  # ========================================================
  prepare_and_test:
    name: Build & Prepare
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      # Invoke our custom local composite action!
      - name: Initialize App Environment
        id: init_env
        uses: ./.github/actions/setup-app-environment
        with:
          app-name: 'payment-gateway'
          target-tier: 'staging'

      - name: Use Composite Action Output
        run: |
          echo "Setup completed at: ${{ steps.init_env.outputs.setup-timestamp }}"

  # ========================================================
  # JOB 2: Calls the Reusable Workflow
  # ========================================================
  deploy_service:
    name: Trigger Reusable Deployment
    needs: [ prepare_and_test ]
    # Syntax for local reusable workflow: ./.github/workflows/<file>.yml
    uses: ./.github/workflows/reusable-deploy-template.yml
    with:
      service_name: 'payment-gateway'
      environment_name: 'staging'
EOF
```

</details>

---

<details>
<summary><strong>Step 4: Commit, Push &amp; Inspect in GitHub Actions</strong></summary>

<br>

Add and push all modular components:

```bash
git add .github/actions/ .github/workflows/
git commit -m "ci: add custom composite action and reusable deployment workflow"
git push origin main
```

### Inspecting the Run:
1. Open the **Actions** tab on GitHub.
2. Select the latest **Master Application Pipeline** run.
3. Open Job 1 (`Build & Prepare`):
   - Notice that **Initialize App Environment** expands to show the 3 internal steps bundled inside our composite action!
   - Step `Use Composite Action Output` successfully reads the timestamp output from the composite action.
4. Notice Job 2 (`Trigger Reusable Deployment`):
   - It appears linked with an arrow from Job 1.
   - Clicking into it shows the execution of the called `reusable-deploy-template.yml` workflow running inside its own fresh VM!

</details>

---

<details>
<summary><strong>Summary Checklist: Lab 05</strong></summary>

<br>

- [x] Created an `action.yml` metadata file specifying `using: 'composite'`.
- [x] Bound inputs, default values, and step output variables in the composite action.
- [x] Declared mandatory `shell: bash` parameters on all composite steps.
- [x] Authored a reusable workflow powered by the `on: workflow_call:` event trigger.
- [x] Orchestrated both composite actions and reusable workflows seamlessly from a master caller pipeline.

</details>
