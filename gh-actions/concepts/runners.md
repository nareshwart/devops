# GitHub Actions Runners

---

<details>
<summary><strong>What is a runner?</strong></summary>

<br>

A **runner** is the machine or execution environment that runs a GitHub Actions job. GitHub queues eligible jobs and assigns each to a runner that matches the job's `runs-on` requirement.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "This command runs on the job's runner"
```

Each job runs on its own runner. Steps within one job run sequentially in that job's environment and can use the same workspace. Different jobs do not share files or in-memory state automatically; use artifacts, outputs, or an external store to pass data between them.

</details>

---

<details>
<summary><strong>GitHub-hosted and self-hosted runners</strong></summary>

<br>

| Type | Who manages it? | Common use |
| :--- | :--- | :--- |
| **GitHub-hosted** | GitHub provides and maintains the runner image. | Standard Linux, Windows, and macOS builds without managing runner machines. |
| **Self-hosted** | You or your organization manage the machine and runner application. | Custom hardware, private network access, specialized software, or workloads requiring a managed environment. |

For GitHub-hosted runners, each job normally runs in a newly provisioned virtual machine, which is removed after the job. The exact isolation and cleanup behavior depends on the runner product and configuration. Self-hosted runners may be persistent, so maintain them, control access, and clean up workspaces and credentials appropriately.

Be especially careful with self-hosted runners that can access internal systems or secrets. Do not let untrusted pull request code execute on a runner with sensitive access unless the environment is specifically isolated for that purpose.

</details>

---

<details>
<summary><strong>Choosing a GitHub-hosted runner</strong></summary>

<br>

Use `runs-on` to select an available runner label. Common examples include:

```yaml
jobs:
  linux:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Linux job"

  windows:
    runs-on: windows-latest
    steps:
      - run: echo "Windows job"

  macos:
    runs-on: macos-latest
    steps:
      - run: echo "macOS job"
```

Runner labels, supported operating system versions, available architectures, and machine sizes vary by GitHub plan and runner product. Check the current GitHub documentation before selecting a specific image or relying on preinstalled software.

### Where to find official runner information

- **Available GitHub-hosted runners:** GitHub Docs, [Choosing the runner for a job](https://docs.github.com/actions/using-jobs/choosing-the-runner-for-a-job).
- **GitHub-hosted runner products and supported images:** GitHub Docs, [GitHub-hosted runners](https://docs.github.com/actions/using-github-hosted-runners/about-github-hosted-runners).
- **Preinstalled software on standard images:** [actions/runner-images](https://github.com/actions/runner-images). Check the image readme for the relevant operating system and image version.
- **Your repository's self-hosted runner setup and status:** Repository **Settings → Actions → Runners**. Organization-level runners are managed in the organization settings.

The `-latest` label is updated over time and may point to a newer OS image as GitHub rolls out changes. If a build depends on a specific OS version, consider using a supported version-specific label and review GitHub's image deprecation announcements.

</details>

---

<details>
<summary><strong>How runner scheduling works</strong></summary>

<br>

1. A workflow trigger creates a run and makes eligible jobs available.
2. GitHub schedules a job when a matching runner is available and the repository or organization has capacity.
3. The runner prepares the job environment and executes its steps in order.
4. The runner sends step output and status back to GitHub.
5. After the job finishes, the runner is released; hosted runner resources are cleaned up according to the product's lifecycle.

Jobs without dependencies can run in parallel, subject to available concurrency and runner capacity. A job that declares `needs:` waits for its dependencies to finish before it can be scheduled.

</details>

---

<details>
<summary><strong>Common runner issues and what to check</strong></summary>

<br>

| Symptom | Common causes | First checks |
| :--- | :--- | :--- |
| Job stays **Queued** | No matching runner is available, a self-hosted runner is offline, concurrency is occupied, or usage limits have been reached. | Inspect the job's `runs-on`, runner status and labels, organization/repository concurrency, and billing or usage limits. |
| Job says **Waiting for a runner** | `runs-on` labels do not match any available runner, especially for self-hosted runners. | Compare every requested label with the labels shown in **Settings → Actions → Runners**. |
| A command or tool is missing | The expected software is not installed on that image, or the image changed. | Check the runner image documentation; install and pin the required tool version explicitly. |
| Script works locally but fails in Actions | Different operating system, shell, path separator, file permissions, case sensitivity, or environment variables. | Check the selected OS and shell, use portable paths where possible, and inspect the failing command's log. |
| Job runs out of disk space or memory | Build outputs, dependency caches, containers, or parallel work exceed the runner's resources. | Review runner size limits, clean temporary outputs, reduce unnecessary downloads, or select an appropriate larger/self-hosted runner if available. |
| Job cannot reach a service | DNS, firewall, proxy, service health, or network access restrictions. | Check the endpoint, service-container health, network policy, and whether the runner can reach the destination. |
| Self-hosted runner behaves differently between runs | Persistent machine state, stale processes, changed tools, or an incomplete cleanup. | Check the runner service and logs, clean the workspace safely, and make setup steps reproducible. |
| Job was cancelled or timed out | Job timeout, workflow cancellation, or execution taking longer than the configured or product limit. | Inspect the run and job logs, `timeout-minutes`, cancellation history, and the task's expected duration. |

Start with the failed job's **Set up job** and step logs. They identify the runner image and usually show the command and exit status that caused the failure. For self-hosted runner service or registration problems, also inspect the runner application logs and online status in GitHub settings.

</details>

---

<details>
<summary><strong>Good practices</strong></summary>

<br>

- Choose the least powerful runner that reliably meets the job's needs.
- Pin tool versions in setup steps instead of relying on whichever version happens to be preinstalled.
- Use explicit runner labels for self-hosted systems; check that the labels match exactly.
- Keep job setup reproducible and avoid depending on state left by an earlier job.
- Use hosted runners for untrusted contributions when possible; isolate self-hosted infrastructure from untrusted code.
- Give workflows only the token permissions they need. Do not treat the runner as a safe place to expose secrets to untrusted code.
- Read image release and deprecation notes when a workflow depends on OS-specific behavior.

</details>

---

<details>
<summary><strong>Summary checklist</strong></summary>

<br>

- [x] A runner executes a job; the job's steps execute sequentially on that runner.
- [x] Separate jobs do not automatically share files or process state.
- [x] `runs-on` selects a hosted image or a set of self-hosted runner labels.
- [x] GitHub Docs and `actions/runner-images` describe hosted runner availability and preinstalled software.
- [x] Queues, label mismatches, missing tools, resource limits, networking, and persistent self-hosted state are common troubleshooting areas.

</details>
