# GitHub Actions — Secrets Management & Production Environments Lab

---

<details>
<summary><strong>Lab Scenario &amp; Objectives</strong></summary>

<br>

### The Scenario
In this lab, you will configure an enterprise deployment workflow with two tiers of environments:
1. **Staging Environment**: Automatically deploys non-sensitive preview builds on every commit.
2. **Production Environment**: Protected by **GitHub Environment Protection Rules**. It pauses execution, requires a **Manual Reviewer Approval** in the GitHub UI, and only unlocks production secrets after approval is granted.
3. You will verify secret masking in console logs, preventing credential leaks.

### Project Architecture
```text
Commit to main
      │
      ▼
┌──────────────┐
│deploy_staging│  (Auto-deploys with Staging secrets)
└──────┬───────┘
       │
       ▼
 ⏸️ MANUAL APPROVAL GATE  (Required Reviewer signs off in GitHub UI)
       │
       ▼
┌──────────────┐
│ deploy_prod  │  (Unlocks Environment: production secrets & deploys)
└──────────────┘
```

</details>

---

<details>
<summary><strong>Step 1: Configure Repository Secrets &amp; Variables in GitHub</strong></summary>

<br>

First, let's create a repository-wide secret and a non-sensitive configuration variable:

### 1. Add Repository Secret
1. In your GitHub repository, click **Settings** (top bar).
2. In the left sidebar, expand **Secrets and variables** &rarr; click **Actions**.
3. Under the **Repository secrets** tab, click **New repository secret**.
4. Configure:
   - **Name**: `GLOBAL_API_KEY`
   - **Secret**: `secret-key-super-secure-token-987654`
5. Click **Add secret**.

---

### 2. Add Configuration Variable
1. On the same page, switch to the **Variables** tab (next to Secrets).
2. Click **New repository variable**.
3. Configure:
   - **Name**: `DEPLOY_REGION`
   - **Value**: `us-east-1`
4. Click **Add variable**.

</details>

---

<details>
<summary><strong>Step 2: Create Protected "production" Environment in GitHub</strong></summary>

<br>

Now let's configure an Environment with protection rules:

1. In repository **Settings**, click **Environments** in the left sidebar.
2. Click **New environment**.
3. Name it **`production`** and click **Configure environment**.
4. Under **Deployment protection rules**:
   - Check **Required reviewers**.
   - Search for and select your own GitHub username.
5. Under **Environment secrets**:
   - Click **Add secret**.
   - **Name**: `PROD_DB_PASSWORD`
   - **Secret**: `SuperProductionPass2026!`
   - Click **Add secret**.
6. Click **Save protection rules**.

</details>

---

<details>
<summary><strong>Step 3: Write the Deployment Workflow (`deploy-environments.yml`)</strong></summary>

<br>

Create `.github/workflows/deploy-environments.yml`:

```bash
cat << 'EOF' > .github/workflows/deploy-environments.yml
name: Multi-Environment Deployment

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  # ========================================================
  # STAGE 1: Staging Deployment (Automatic)
  # ========================================================
  deploy_staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest

    steps:
      - name: Deploy Staging Application
        env:
          REGION: ${{ vars.DEPLOY_REGION }}
          KEY: ${{ secrets.GLOBAL_API_KEY }}
        run: |
          echo "Deploying to Staging in region: $REGION"
          echo "Using Global Key: $KEY" # Notice: GitHub will mask this!
          echo "Staging deployment successful."

  # ========================================================
  # STAGE 2: Production Deployment (Protected Gate)
  # ========================================================
  deploy_prod:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: [ deploy_staging ]

    # Binds to the protected environment created in Step 2!
    environment:
      name: production
      url: https://my-production-app.example.com

    steps:
      - name: Deploy Production Application
        env:
          REGION: ${{ vars.DEPLOY_REGION }}
          DB_PASS: ${{ secrets.PROD_DB_PASSWORD }}
          KEY: ${{ secrets.GLOBAL_API_KEY }}
        run: |
          echo "=== Deploying to Production Tier ==="
          echo "Region Target: $REGION"
          echo "Database Password: $DB_PASS" # Redacted to ***
          echo "Global Key: $KEY"             # Redacted to ***
          echo "Production live verification complete!"
EOF
```

</details>

---

<details>
<summary><strong>Step 4: Commit, Push &amp; Test the Approval Gate</strong></summary>

<br>

Commit and push the workflow:

```bash
git add .github/workflows/deploy-environments.yml
git commit -m "ci: add protected production deployment environment"
git push origin main
```

### Experience the Manual Approval Gate:
1. Navigate to the **Actions** tab on GitHub and open the latest run.
2. Watch `Deploy to Staging` execute and finish with a green checkmark.
3. Notice that `Deploy to Production` **does NOT start immediately**!
4. It displays an amber status badge: **"Waiting for review"**.
5. Above the jobs, a banner appears:
   > *`deploy_prod` is waiting for review to deploy to `production`.*
6. Click the button **Review deployments**.
7. Check the box for `production`, add an optional comment (e.g. *"Approved after QA sign-off"*), and click **Approve and deploy**.
8. Notice that `Deploy to Production` immediately transitions to active running status!
9. Open the step logs and verify:
   - Secret values like `SuperProductionPass2026!` were printed as `***` (Masked).
   - The deployment URL `https://my-production-app.example.com` appears linked directly in the GitHub UI summary.

</details>

---

<details>
<summary><strong>Summary Checklist: Lab 03</strong></summary>

<br>

- [x] Configured encrypted repository secrets and non-sensitive configuration variables.
- [x] Created a protected `production` environment with **Required Reviewers**.
- [x] Injected environment secrets that remain locked until approval is granted.
- [x] Verified automatic log masking (`***`) for sensitive credentials.
- [x] Successfully reviewed, approved, and promoted a release from Staging to Production.

</details>

