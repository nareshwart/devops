# GitHub Actions Variables and Contexts

GitHub Actions has several kinds of values that workflows can read: built-in **contexts**, default **environment variables**, user-defined `env` values, configuration `vars`, secrets, and values passed between steps or jobs. They are accessed differently depending on where they are used.

> This guide covers the built-in context families and commonly used default variables. `github.event` is a payload whose properties depend on the event type, so its possible fields are not a fixed list. See the official reference links for the complete and current property lists.

---

<details>
<summary><strong>1. Expressions, contexts, and shell environment variables</strong></summary>

<br>

Use the expression syntax `${{ ... }}` to read a context or evaluate an expression in workflow YAML:

```yaml
name: Build ${{ github.ref_name }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Show the trigger
        run: echo "Event: ${{ github.event_name }}"
```

In a `run:` script, the shell also reads environment variables using its own syntax:

```yaml
steps:
  - name: Print values
    env:
      BRANCH_NAME: ${{ github.ref_name }}
    run: |
      echo "Branch from shell environment: $BRANCH_NAME"
      echo "Run number from GitHub default environment: $GITHUB_RUN_NUMBER"
```

These are different evaluation systems:

| Form | Evaluated by | Example |
| :--- | :--- | :--- |
| `${{ context.property }}` | GitHub Actions expression evaluator | `${{ github.sha }}` |
| `$NAME` | Bash and other Unix-like shells | `$GITHUB_SHA` |
| `$env:NAME` | PowerShell | `$env:GITHUB_SHA` |
| `%NAME%` | Windows `cmd` | `%GITHUB_SHA%` |

Prefer the expression form in workflow keys and use explicit `env:` mappings for values passed into shell scripts. Shell syntax depends on the runner's operating system and selected shell.

</details>

---

<details>
<summary><strong>2. Built-in contexts: quick map</strong></summary>

<br>

A context is a named object accessed with expressions such as `${{ github.ref }}`. The context is available only in supported workflow locations, and its values may not exist until the relevant stage has started.

| Context | What it contains | Example |
| :--- | :--- | :--- |
| `github` | Workflow run, repository, actor, ref, and event metadata | `${{ github.repository }}` |
| `env` | Custom environment values defined in the workflow, job, or step | `${{ env.LOG_LEVEL }}` |
| `vars` | Repository, organization, or environment configuration variables | `${{ vars.DEPLOY_REGION }}` |
| `secrets` | Encrypted secrets made available to the workflow or job | `${{ secrets.DEPLOY_TOKEN }}` |
| `runner` | Information about the current runner | `${{ runner.os }}` |
| `job` | Current job status and container/service information | `${{ job.status }}` |
| `steps` | Outcomes and outputs of earlier steps in this job | `${{ steps.build.outputs.version }}` |
| `needs` | Results and outputs of jobs this job depends on | `${{ needs.test.result }}` |
| `strategy` | Strategy configuration, including matrix fail-fast and job index | `${{ strategy.job-total }}` |
| `matrix` | Values for the current matrix job combination | `${{ matrix.python-version }}` |
| `inputs` | Inputs provided to a manually triggered or reusable workflow | `${{ inputs.target }}` |
| `jobs` | Outputs from jobs in a reusable workflow, exposed to its caller | `${{ jobs.build.outputs.package-url }}` |

Not every context is available in every YAML key. For example, `matrix` is meaningful inside a matrix job, and `steps` refers only to earlier steps in the current job. Consult GitHub's [Contexts reference](https://docs.github.com/actions/learn-github-actions/contexts#context-availability) when using a context in a particular key such as `if`, `name`, or `with`.

</details>

---

<details>
<summary><strong>3. The `github` context</strong></summary>

<br>

The `github` context describes the workflow run and the repository event. Common properties include:

| Property | Meaning |
| :--- | :--- |
| `github.actor` | Username that initiated the workflow run. |
| `github.actor_id` | Numeric account ID of the actor. |
| `github.repository` | Repository name in `owner/repository` form. |
| `github.repository_owner` | Repository owner's name. |
| `github.event_name` | Triggering event name, such as `push`, `pull_request`, or `workflow_dispatch`. |
| `github.event` | Event payload object; fields depend on the event. |
| `github.ref` | Fully qualified branch or tag ref, such as `refs/heads/main`. |
| `github.ref_name` | Short branch or tag name. |
| `github.ref_type` | `branch` or `tag` when applicable. |
| `github.sha` | Commit SHA associated with the event. Its meaning can depend on the event type. |
| `github.workflow` | Workflow name. |
| `github.run_id` | Unique ID of the workflow run in the repository. |
| `github.run_number` | Repository-level run number for that workflow. |
| `github.run_attempt` | Attempt number for this run, including reruns. |
| `github.server_url` | GitHub server URL. |
| `github.api_url` | GitHub REST API URL. |
| `github.workflow_ref` | Ref of the workflow file being run. |
| `github.workflow_sha` | Commit SHA of the workflow file. |
| `github.event_path` | Runner path to the event payload file. |

### Pull request caveat

For pull request events, `github.ref` and `github.sha` can refer to a merge ref and merge commit rather than directly to the source branch's head commit. Use event-specific properties such as `github.event.pull_request.head.sha` when the pull request head is specifically required, and review the event payload shape for the trigger being used.

### Inspecting an event payload

For troubleshooting, print selected fields rather than dumping all metadata:

```yaml
- name: Show event details
  run: |
    echo "Event: ${{ github.event_name }}"
    echo "Repository: ${{ github.repository }}"
```

Event payloads may contain user-provided text. Treat them as untrusted input, especially when interpolating them into shell commands. Prefer passing values through `env:` and quoting them in the shell instead of placing untrusted text directly into an inline script.

</details>

---

<details>
<summary><strong>4. `env`: workflow, job, and step variables</strong></summary>

<br>

The `env:` key defines environment variables for a workflow, job, or individual step. A narrower scope overrides the same name from a wider scope:

**step > job > workflow**

```yaml
name: CI

env:
  APP_MODE: test

jobs:
  check:
    runs-on: ubuntu-latest
    env:
      LOG_LEVEL: info
    steps:
      - name: Run checks
        env:
          LOG_LEVEL: debug
        run: |
          echo "Mode: $APP_MODE"
          echo "Log level: $LOG_LEVEL"
```

The step sees `APP_MODE=test` and `LOG_LEVEL=debug`. Use `env.NAME` in expressions where supported, and the runner's shell syntax inside `run:`.

An environment variable set by a shell command is normally local to that process. To make a value available to later steps in the same job, append it to the runner-provided `$GITHUB_ENV` file:

```yaml
- name: Set a value for later steps
  run: echo "BUILD_LABEL=nightly" >> "$GITHUB_ENV"

- name: Read the value
  run: echo "$BUILD_LABEL"
```

The step that writes to `GITHUB_ENV` does not receive the new value itself; later steps do. This mechanism does not pass values to another job.

</details>

---

<details>
<summary><strong>5. `vars` and `secrets`: configuration and sensitive values</strong></summary>

<br>

### Configuration variables: `vars`

Use configuration variables for non-sensitive values managed in GitHub settings, such as a deployment region or service URL. They can be defined at organization, repository, and environment scope.

```yaml
- name: Show deployment region
  run: echo "Region: $DEPLOY_REGION"
  env:
    DEPLOY_REGION: ${{ vars.DEPLOY_REGION }}
```

When the same configuration variable name is defined at multiple levels, the most specific applicable scope takes precedence: **environment > repository > organization**. An unset configuration variable evaluates to an empty string. Do not store credentials in `vars`.

### Secrets: `secrets`

Use secrets for credentials and other sensitive values. Secrets may be managed at organization, repository, or environment scope. Environment secrets are available to jobs that use that environment and satisfy its protection rules.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
        run: ./deploy.sh
```

Important behavior:

- Secrets are not automatically exposed as shell variables; map them through `env:` or an action's documented input.
- Secrets are generally unavailable to workflows triggered from forked pull requests, with a limited `GITHUB_TOKEN` exception subject to GitHub's restrictions.
- Secrets cannot be directly referenced in an `if:` condition. Map an appropriate value to an environment variable or use a job/step output for the conditional logic, without exposing the secret.
- GitHub masks recognized secret values in logs, but masking is not a reason to print secrets. Transformed or partial values may not be masked.
- `secrets.GITHUB_TOKEN` is a short-lived token for the current workflow run. Restrict its permissions with the workflow or job `permissions:` key.

</details>

---

<details>
<summary><strong>6. `runner`, `job`, `strategy`, and `matrix`</strong></summary>

<br>

### `runner`

Describes the current runner. Common properties:

| Property | Meaning |
| :--- | :--- |
| `runner.os` | Operating system label, such as `Linux`, `Windows`, or `macOS`. |
| `runner.arch` | Runner architecture, such as `X64` or `ARM64`. |
| `runner.name` | Runner name. |
| `runner.temp` | Temporary directory path for the runner. |
| `runner.tool_cache` | Tool cache directory on the runner. |
| `runner.debug` | Set when runner diagnostic logging is enabled. |

### `job`

Describes the current job. The commonly used `job.status` is `success`, `failure`, or `cancelled`. Container and service properties are relevant when the job uses a container or service containers.

### `strategy`

Describes job strategy execution. Common properties include:

- `strategy.fail-fast`: whether a matrix cancels remaining jobs after a failure.
- `strategy.job-index`: zero-based index of the current job among matrix jobs.
- `strategy.job-total`: total number of jobs in the matrix.
- `strategy.max-parallel`: maximum number of matrix jobs allowed to run concurrently.

### `matrix`

Contains the values for the current matrix combination. The properties are defined by your own matrix:

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: [20, 22]
steps:
  - run: echo "OS=${{ matrix.os }} Node=${{ matrix.node }}"
```

This produces four job combinations. `matrix.os` and `matrix.node` exist because they are declared in this workflow; they are not universal built-in property names.

</details>

---

<details>
<summary><strong>7. `steps`, `needs`, `inputs`, and reusable workflow outputs</strong></summary>

<br>

### `steps`

Access outcomes and outputs from steps that have already run in the same job. Give a step an `id` to reference its outputs:

```yaml
steps:
  - id: version
    run: echo "value=1.2.3" >> "$GITHUB_OUTPUT"

  - name: Use the step output
    run: echo "Version: ${{ steps.version.outputs.value }}"
```

`steps.<id>.outcome` records the result before `continue-on-error` is applied; `steps.<id>.conclusion` reflects the final result after it. Outputs are strings; convert or validate them where a numeric or boolean value is required.

### `needs`

Access the result and declared outputs of jobs listed in the current job's `needs:`. Jobs must be direct dependencies:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.value }}
    steps:
      - id: version
        run: echo "value=1.2.3" >> "$GITHUB_OUTPUT"

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying ${{ needs.build.outputs.version }}"
```

Common properties are `needs.<job_id>.result` (`success`, `failure`, `cancelled`, or `skipped`) and `needs.<job_id>.outputs.<name>`.

### `inputs`

For `workflow_dispatch`, inputs come from the **Run workflow** form. For a reusable workflow, they are declared under `on.workflow_call.inputs` and provided by the caller. The `inputs` context preserves Boolean types for Boolean inputs:

```yaml
on:
  workflow_dispatch:
    inputs:
      run_checks:
        description: Run the checks
        type: boolean
        default: true

jobs:
  check:
    if: ${{ inputs.run_checks }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Checks enabled"
```

For manual dispatch workflows, `github.event.inputs` is a legacy-compatible representation in which inputs are strings. Prefer `inputs` when you need the declared input types.

### `jobs` in reusable workflows

Within a reusable workflow, the `jobs` context is used to expose outputs from its jobs through the reusable workflow's `on.workflow_call.outputs` interface. It is not a general replacement for `needs` in ordinary workflows.

</details>

---

<details>
<summary><strong>8. Default environment variables</strong></summary>

<br>

GitHub sets default environment variables for each runner process. They are read from shell scripts as environment variables (such as `$GITHUB_SHA` on Bash), not through the `env` context. GitHub does not allow workflows to overwrite default `GITHUB_*` and `RUNNER_*` variables.

| Variable | Meaning |
| :--- | :--- |
| `GITHUB_ACTION` | Identifier for the currently running action or step. |
| `GITHUB_ACTION_PATH` | Path to an action's directory; intended for use inside an action. |
| `GITHUB_ACTION_REPOSITORY` | Repository containing the action currently running, when applicable. |
| `GITHUB_ACTIONS` | `true` when the process is running in GitHub Actions. |
| `GITHUB_ACTOR` | Username that initiated the workflow run. |
| `GITHUB_ACTOR_ID` | Numeric ID of the actor. |
| `GITHUB_API_URL` | GitHub API URL. |
| `GITHUB_BASE_REF` | Base branch of a pull request or target branch of a merge queue event, when applicable. |
| `GITHUB_ENV` | File path for setting environment variables for later steps in this job. |
| `GITHUB_EVENT_NAME` | Name of the event that triggered the workflow. |
| `GITHUB_EVENT_PATH` | Path to the event payload file on the runner. |
| `GITHUB_GRAPHQL_URL` | GitHub GraphQL API URL. |
| `GITHUB_HEAD_REF` | Source branch of a pull request or merge queue event, when applicable. |
| `GITHUB_JOB` | Job ID of the current job. |
| `GITHUB_OUTPUT` | File path for setting outputs from the current step. |
| `GITHUB_PATH` | File path for adding directories to `PATH` for later steps in the job. |
| `GITHUB_REF` | Fully qualified branch, tag, or pull request merge ref. |
| `GITHUB_REF_NAME` | Short branch or tag name. |
| `GITHUB_REF_PROTECTED` | `true` if branch protections or rulesets apply to the ref. |
| `GITHUB_REF_TYPE` | `branch` or `tag`. |
| `GITHUB_REPOSITORY` | Repository name in `owner/repository` form. |
| `GITHUB_REPOSITORY_ID` | Numeric repository ID. |
| `GITHUB_REPOSITORY_OWNER` | Repository owner's name. |
| `GITHUB_RUN_ATTEMPT` | Attempt number for this run, including reruns. |
| `GITHUB_RUN_ID` | Unique workflow run ID. |
| `GITHUB_RUN_NUMBER` | Repository-level run number for the workflow. |
| `GITHUB_SERVER_URL` | GitHub server URL. |
| `GITHUB_SHA` | Commit SHA associated with the event. |
| `GITHUB_STEP_SUMMARY` | File path for adding Markdown to the job summary. |
| `GITHUB_TRIGGERING_ACTOR` | Username that triggered the current run or rerun. |
| `GITHUB_WORKFLOW` | Workflow name. |
| `GITHUB_WORKFLOW_REF` | Ref of the workflow file being run. |
| `GITHUB_WORKFLOW_SHA` | Commit SHA of the workflow file. |
| `GITHUB_WORKSPACE` | Default working directory for steps. |
| `RUNNER_ARCH` | Runner architecture. |
| `RUNNER_DEBUG` | `1` when runner diagnostic logging is enabled. |
| `RUNNER_NAME` | Runner name. |
| `RUNNER_OS` | Runner operating system. |
| `RUNNER_TEMP` | Temporary directory path. |
| `RUNNER_TOOL_CACHE` | Tool cache directory. |

This table lists commonly used variables; GitHub may add or update defaults. See the [Default environment variables reference](https://docs.github.com/actions/reference/workflows-and-actions/variables#default-environment-variables) for the complete current list.

</details>

---

<details>
<summary><strong>9. Workflow command files: `GITHUB_ENV`, `GITHUB_OUTPUT`, `GITHUB_PATH`, and summary</strong></summary>

<br>

These special default variables contain file paths. Write a line to the file to ask the runner to apply a workflow command:

| File variable | Purpose | Scope |
| :--- | :--- | :--- |
| `GITHUB_ENV` | Set an environment variable for later steps. | Later steps in the same job. |
| `GITHUB_OUTPUT` | Set an output for the current step. | Referenced as `steps.<id>.outputs.<name>`; can be promoted to a job output. |
| `GITHUB_PATH` | Add a directory to `PATH`. | Later steps in the same job. |
| `GITHUB_STEP_SUMMARY` | Append Markdown to the run's job summary. | Summary content for the current job. |

Example:

```yaml
steps:
  - id: version
    run: |
      echo "APP_VERSION=1.2.3" >> "$GITHUB_ENV"
      echo "value=1.2.3" >> "$GITHUB_OUTPUT"
      echo "### Build complete" >> "$GITHUB_STEP_SUMMARY"
  - run: echo "$APP_VERSION"
```

Use the appropriate shell syntax on Windows. Do not write untrusted content as special workflow-command syntax without safely encoding it. To pass a value between jobs, declare a job output based on a step output and consume it through `needs`.

</details>

---

<details>
<summary><strong>10. Expression operators and functions</strong></summary>

<br>

Expressions can combine context properties with literals, comparisons, and functions. Use `${{ ... }}` in most expression-capable fields; `if:` conditions are a notable location where the delimiters can be omitted.

### Operators

Common operators include:

| Operator | Meaning |
| :--- | :--- |
| `==`, `!=` | Equality and inequality. |
| `<`, `<=`, `>`, `>=` | Numeric or string comparison. |
| `&&`, `||`, `!` | Logical AND, OR, and NOT. |
| `()` | Grouping. |

### Common functions

| Function | Use |
| :--- | :--- |
| `contains(search, item)` | Check whether a string or array contains a value. |
| `startsWith(string, search)` | Check a string prefix. |
| `endsWith(string, search)` | Check a string suffix. |
| `format(string, args...)` | Insert values into a string template. |
| `join(array, separator)` | Join array items into text. |
| `toJSON(value)` | Convert a value to JSON text, useful for limited debugging. |
| `fromJSON(value)` | Parse JSON text into an expression value. |
| `hashFiles(path...)` | Compute a hash for matching workspace files, commonly used in cache keys. |
| `success()` | Check that preceding steps or required jobs succeeded, depending on context. |
| `failure()` | Check whether a preceding step or required job failed. |
| `cancelled()` | Check whether the run was cancelled. |
| `always()` | Run a step or job regardless of success or failure; cancellation and timeout behavior should be considered. |

Example:

```yaml
- name: Run only for main branch pushes
  if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
  run: echo "Building main"

- name: Show package file hash
  run: echo "See cache key in workflow configuration"
```

Expressions and contexts are documented in the [Expressions reference](https://docs.github.com/actions/learn-github-actions/expressions).

</details>

---

<details>
<summary><strong>11. Where variables are configured and where they work</strong></summary>

<br>

| Value source | Configure it in | Read it as |
| :--- | :--- | :--- |
| Workflow `env` | Top-level `env:` in workflow YAML | `${{ env.NAME }}` in supported expression locations; `$NAME` or platform shell equivalent in `run:` |
| Job `env` | A job's `env:` block | Same, limited to that job and its steps |
| Step `env` | A step's `env:` block | Same, limited to that step |
| Configuration variable | Repository or organization **Settings → Secrets and variables → Actions → Variables**, or environment variables settings | `${{ vars.NAME }}`; map to `env:` for shell use |
| Secret | Repository, organization, or environment **Settings → Secrets and variables → Actions → Secrets** | `${{ secrets.NAME }}`; map to `env:` or a documented action input |
| Manual workflow input | `on.workflow_dispatch.inputs` | `${{ inputs.NAME }}` |
| Reusable workflow input | `on.workflow_call.inputs` | `${{ inputs.NAME }}` |
| Step output | `GITHUB_OUTPUT` in a step with an `id` | `${{ steps.ID.outputs.NAME }}` |
| Job output | Job `outputs:` mapped from a step output | `${{ needs.JOB_ID.outputs.NAME }}` in a dependent job |
| Matrix value | `strategy.matrix` in workflow YAML | `${{ matrix.NAME }}` |
| Default runner variable | Set by the runner | Shell environment syntax; many equivalents are also exposed through contexts |

</details>

---

<details>
<summary><strong>12. Safety and debugging guidance</strong></summary>

<br>

- Do not commit credentials. Use `secrets` for sensitive values and `vars` for non-sensitive configuration.
- Avoid printing secrets or entire contexts. Log only the specific fields needed to debug.
- Treat event payload fields, workflow inputs, and outputs as untrusted strings; validate them before using them in commands, paths, or API requests.
- Pass expression values into scripts using `env:` and quote shell variables, rather than interpolating untrusted text directly into a script body.
- Remember that `env`, step outputs, and shell processes have different scopes. Values do not automatically persist between jobs.
- If an expression is unexpectedly empty, check the event type, context availability for that YAML key, variable scope, spelling, and whether the value is defined.
- Use `toJSON` only for carefully selected non-sensitive data; event payloads and contexts may contain large or user-controlled values.

Official references:

- [Contexts](https://docs.github.com/actions/learn-github-actions/contexts)
- [Variables](https://docs.github.com/actions/reference/workflows-and-actions/variables)
- [Workflow commands](https://docs.github.com/actions/using-workflows/workflow-commands-for-github-actions)
- [Expressions](https://docs.github.com/actions/learn-github-actions/expressions)
- [Workflow syntax](https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions)

</details>
