# Pre-Built GitHub Actions

---

<details>
<summary><strong>What is a pre-built action?</strong></summary>

<br>

A **pre-built action** is a reusable unit of automation that you can call from a workflow step with `uses:`. Actions package common tasks so you do not have to write and maintain every implementation yourself.

For example, `actions/checkout` checks out repository code, while setup actions can install a language runtime. Actions run as steps inside a job on its runner.

```yaml
steps:
  - name: Check out the repository
    uses: actions/checkout@v4

  - name: Set up Node.js
    uses: actions/setup-node@v4
    with:
      node-version: 22

  - name: Install dependencies and test
    run: npm ci && npm test
```

In this example, the first two steps use actions and the third runs shell commands directly.

</details>

---

<details>
<summary><strong>How actions help</strong></summary>

<br>

- **Save setup time:** Reuse common operations such as checking out code, configuring a toolchain, caching packages, or uploading build artifacts.
- **Keep workflows readable:** A step can state its intent without embedding a large script.
- **Reuse tested integrations:** An action can handle details such as credentials, API calls, or platform differences.
- **Share automation:** Teams can use actions maintained by GitHub, other publishers, or their own organization.

An action does not replace the workflow. The workflow still chooses the trigger, job, runner, order, permissions, and conditions; an action performs a particular step.

</details>

---

<details>
<summary><strong>Where to find actions</strong></summary>

<br>

- **GitHub Marketplace — Actions:** [github.com/marketplace?type=actions](https://github.com/marketplace?type=actions) is a searchable directory of published actions. It is a discovery source, not a guarantee that every listed action is safe, maintained, or suitable.
- **GitHub's actions organization:** [github.com/actions](https://github.com/actions) contains commonly used official actions such as `checkout`, `setup-node`, `setup-python`, and artifact actions.
- **Publisher repositories:** Open the action's repository to read its `README`, inspect its `action.yml` or `action.yaml`, review source code and releases, and check open issues and maintenance activity.
- **GitHub Docs:** [Using actions in your workflows](https://docs.github.com/actions/learn-github-actions/essential-features-of-github-actions#using-actions-in-your-workflows) explains the `uses:` syntax and how to reference actions.

Marketplace is not an exhaustive catalog of every action that can be used. Actions can also be hosted in repositories that are not listed there, or maintained locally within your own repository.

</details>

---

<details>
<summary><strong>How to reference an action</strong></summary>

<br>

An action reference usually has the form `owner/repository@ref`:

```yaml
- uses: actions/checkout@v4
```

Here, `actions/checkout` identifies the repository and `v4` is the reference. Depending on the action, the reference can be a version tag, branch, or commit SHA. A tag is convenient, but its owner may move it. For stronger reproducibility, pin to a reviewed full commit SHA and update it deliberately.

Actions may accept inputs under `with:` and expose outputs for later steps. Check the action's own documentation for the exact input names, required values, outputs, supported runner environments, and version requirements.

```yaml
- name: Upload test results
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: reports/
```

</details>

---

<details>
<summary><strong>Common action issues</strong></summary>

<br>

| Symptom | Common causes | What to check |
| :--- | :--- | :--- |
| `Unable to resolve action` or action not found | Typo in owner/repository, inaccessible or deleted repository, or invalid `@ref`. | Check spelling, repository visibility/access, and that the requested tag, branch, or SHA exists. |
| Action reports a missing or invalid input | Input name or value does not match the action's interface. | Read the README and `action.yml` for that exact version; YAML input names and types must match. |
| Action runs but cannot find project files | Checkout was omitted or the action expects a different working directory/path. | Add `actions/checkout` before the step when needed and verify the workspace and action's `path` inputs. |
| Permission denied or API request fails | `GITHUB_TOKEN` lacks the required scope, or the action uses credentials that are not configured. | Review the action documentation, workflow `permissions`, repository settings, and secret names. Grant only the permissions needed. |
| Action fails after a runtime or runner update | The action may depend on an outdated runtime, an unsupported OS, or a changed preinstalled tool. | Check the action's release notes, supported environments, and runner image notes; update to a maintained compatible release. |
| Action is unexpectedly slow or downloads fail | Network access, rate limits, large downloads, or external service availability. | Inspect the failing log, retry policy, external service status, and network requirements. |
| Output is empty or unavailable | Wrong output identifier, missing step `id`, or the action did not set the output. | Check the documented output name and define a step `id` if referencing `steps.<id>.outputs.<name>`. |
| Workflow becomes unsafe or changes unexpectedly | An unreviewed action, mutable ref, excessive permissions, or untrusted code can run with access to secrets. | Review source and release history, pin a trusted ref, restrict token permissions, and avoid exposing secrets to untrusted pull request code. |

Start with the exact failing step's logs, then compare the workflow's inputs and permissions against the documentation for the action version being used.

</details>

---

<details>
<summary><strong>Choosing actions safely</strong></summary>

<br>

Before adding an action:

1. Confirm it solves a real need; a short `run:` command may be simpler.
2. Verify the publisher and repository, and review the action's code and documentation.
3. Check recent releases, maintenance, compatibility, and known issues.
4. Use a specific trusted version; consider pinning to a full commit SHA where appropriate.
5. Grant the workflow only the permissions the action requires.
6. Avoid passing secrets to actions or code that you have not reviewed and trust.
7. Update action versions deliberately and verify workflow behavior after an update.

Actions execute as part of your workflow, so treat them like third-party dependencies in your software supply chain.

</details>

---

<details>
<summary><strong>Summary checklist</strong></summary>

<br>

- [x] A pre-built action is a reusable step invoked using `uses:`.
- [x] Find actions in GitHub Marketplace, GitHub's actions organization, and publisher repositories.
- [x] Check the action's version-specific documentation for inputs, outputs, permissions, and compatibility.
- [x] Investigate resolution, input, path, permission, and runtime failures using the step logs.
- [x] Review and pin trusted action versions, and keep permissions and secret exposure limited.

</details>
