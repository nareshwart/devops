# GitHub Actions — Multi-Job Pipelines, DAG Dependencies & Artifacts Lab

---

<details>
<summary><strong>Lab Scenario &amp; Architecture</strong></summary>

<br>

### The Scenario
In modern continuous delivery, monolithic single-job workflows are slow and difficult to maintain. In this lab, you will build a 3-stage modular pipeline connected via a **Directed Acyclic Graph (DAG)**:
1. **Job 1 (`build_dist`)**: Compiles application assets, generates a release bundle, and uploads it to GitHub Artifact storage using `actions/upload-artifact@v4`.
2. **Job 2 (`security_audit`)**: Runs a dependency vulnerability scan. Runs **in parallel** with Job 1 to minimize total pipeline duration.
3. **Job 3 (`package_release`)**: Declares `needs: [build_dist, security_audit]`. It starts **only after both upstream jobs pass**, downloads the artifact bundle via `actions/download-artifact@v4`, computes its SHA256 checksum, and outputs the final release package.

### Pipeline Dependency Graph
```text
┌─────────────────┐       ┌─────────────────┐
│   build_dist    │       │ security_audit  │  <-- Execute in Parallel on separate VMs
└────────┬────────┘       └────────┬────────┘
         │                         │
         │ (Uploads Artifact)      │ (Vulnerability check)
         └───────────┬─────────────┘
                     ▼
         ┌─────────────────────────┐
         │     package_release     │  <-- needs: [build_dist, security_audit]
         └─────────────────────────┘      (Downloads & verifies Artifact)
```

</details>

---

<details>
<summary><strong>Step 1: Create the Multi-Job Workflow File</strong></summary>

<br>

Create `.github/workflows/pipeline-artifacts.yml`:

```bash
cat << 'EOF' > .github/workflows/pipeline-artifacts.yml
name: Build, Audit & Package Pipeline

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  # ========================================================
  # STAGE 1: Build Application Distribution Bundle
  # ========================================================
  build_dist:
    name: Build Distribution Assets
    runs-on: ubuntu-latest
    outputs:
      release_version: ${{ steps.version.outputs.tag }}

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Compute Dynamic Release Version
        id: version
        run: |
          VERSION="v1.0.${{ github.run_number }}"
          echo "tag=$VERSION" >> "$GITHUB_OUTPUT"
          echo "Generated Release Tag: $VERSION"

      - name: Compile Distribution Package
        run: |
          mkdir -p dist
          echo "Application Build: ${{ steps.version.outputs.tag }}" > dist/build_info.txt
          echo "Built on: $(date -u)" >> dist/build_info.txt
          echo "Commit SHA: ${{ github.sha }}" >> dist/build_info.txt
          # Simulate compilation
          tar -czf dist/app-bundle.tar.gz -C dist build_info.txt

      - name: Upload Distribution Artifact
        uses: actions/upload-artifact@v4
        with:
          name: release-bundle
          path: dist/app-bundle.tar.gz
          retention-days: 5

  # ========================================================
  # STAGE 2: Security & Vulnerability Audit (Runs in Parallel!)
  # ========================================================
  security_audit:
    name: Security Vulnerability Scan
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Run Security Audit
        run: |
          echo "Auditing dependencies for known CVE vulnerabilities..."
          # Simulating scan check
          sleep 3
          echo "Zero critical vulnerabilities found."

  # ========================================================
  # STAGE 3: Final Package & Verification (Sequential Gate)
  # ========================================================
  package_release:
    name: Verify & Finalize Package
    runs-on: ubuntu-latest
    # Enforce dependencies: waits for BOTH upstream jobs to succeed
    needs: [ build_dist, security_audit ]

    steps:
      - name: Download Build Artifact from Stage 1
        uses: actions/download-artifact@v4
        with:
          name: release-bundle
          path: downloaded-dist/

      - name: Verify Bundle Integrity & Checksum
        run: |
          echo "=== Downloaded Artifacts ==="
          ls -lh downloaded-dist/

          echo "=== Computing SHA256 Checksum ==="
          sha256sum downloaded-dist/app-bundle.tar.gz | tee checksum.txt

          echo "=== Unpacking and Verifying Contents ==="
          tar -xzf downloaded-dist/app-bundle.tar.gz -C downloaded-dist/
          cat downloaded-dist/build_info.txt

      - name: Publish Run Summary
        run: |
          CHECKSUM=$(cut -d ' ' -f 1 checksum.txt)
          echo "### 📦 Release Artifact Verified" >> "$GITHUB_STEP_SUMMARY"
          echo "- **Version**: \`${{ needs.build_dist.outputs.release_version }}\`" >> "$GITHUB_STEP_SUMMARY"
          echo "- **Artifact**: \`app-bundle.tar.gz\`" >> "$GITHUB_STEP_SUMMARY"
          echo "- **SHA-256**: \`$CHECKSUM\`" >> "$GITHUB_STEP_SUMMARY"
          echo "- **Audit Status**: ✅ Passed security checks" >> "$GITHUB_STEP_SUMMARY"
EOF
```

</details>

---

<details>
<summary><strong>Step 2: Commit, Push &amp; Inspect Visual DAG in GitHub UI</strong></summary>

<br>

Commit and push your multi-job workflow:

```bash
git add .github/workflows/pipeline-artifacts.yml
git commit -m "ci: add multi-job DAG pipeline with artifact transfer"
git push origin main
```

### Inspect the DAG Workflow Graph:
1. Navigate to the **Actions** tab on your repository.
2. Click on the latest **Build, Audit & Package Pipeline** run.
3. Look at the visual interactive dependency graph:
   - Notice that **Build Distribution Assets** and **Security Vulnerability Scan** start executing at the same time in parallel.
   - Notice that **Verify & Finalize Package** shows an arrow from both jobs and remains queued until both finish.
4. When all three jobs complete with green checkmarks, scroll down to the **Artifacts** section at the bottom of the summary page:
   - Notice the downloadable `release-bundle.zip` artifact file!
   - You can click to download and inspect it locally on your machine.

</details>

---

<details>
<summary><strong>Step 3: Test Failure Propagation &amp; Upstream Dependency Gates</strong></summary>

<br>

What happens if an upstream job fails? Let's verify that downstream jobs are safely skipped:

1. Edit `.github/workflows/pipeline-artifacts.yml` and simulate a security failure:
   ```yaml
   # Inside security_audit job:
   - name: Run Security Audit
     run: |
       echo "CRITICAL CVE-2026-9999 DETECTED!"
       exit 1 # Deliberate non-zero failure code!
   ```
2. Commit and push:
   ```bash
   git commit -am "test: simulate security scan failure"
   git push origin main
   ```
3. Watch the Actions run:
   - `build_dist`: **SUCCEEDS (Green)**.
   - `security_audit`: **FAILS (Red)**.
   - `package_release`: **SKIPPED (Gray)**!
   - Downstream release was blocked, preventing compromised code from ever reaching the release stage!

</details>

---

<details>
<summary><strong>Summary Checklist: Lab 02</strong></summary>

<br>

- [x] Implemented multi-job DAG orchestration using `needs: [job_a, job_b]`.
- [x] Passed computed runtime variables across jobs using `$GITHUB_OUTPUT` and `needs.<job>.outputs`.
- [x] Uploaded binary build bundles using `actions/upload-artifact@v4`.
- [x] Downloaded and verified artifacts on a separate runner VM using `actions/download-artifact@v4`.
- [x] Inspected visual pipeline graphs and downloaded published zip assets from the GitHub UI.
- [x] Verified that upstream failures automatically skip downstream dependent jobs.

</details>

