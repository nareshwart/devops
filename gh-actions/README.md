# GitHub Actions

Learn how GitHub Actions automates CI/CD, then practice building and running workflows. The concepts explain the main ideas; the labs provide hands-on exercises.

## Concepts

| # | Topic | What you'll learn |
| :---: | :--- | :--- |
| 1 | [CI/CD Introduction and GitHub Actions Overview](./concepts/cicd-intro-gha-overview.md) | CI, Continuous Delivery, Continuous Deployment, and how GitHub Actions fits in |
| 2 | [The `.github/workflows/` Directory](./concepts/github-actions-workflows-directory.md) | Where workflow files belong and how to organize them |
| 3 | [Jobs, Steps, Matrix, and Artifacts](./concepts/jobs-steps-matrix-artifacts.md) | Job and step structure, dependencies, matrix jobs, caching, and artifacts |
| 4 | [Secrets, Environments, and Contexts](./concepts/secrets-environments-contexts.md) | Store sensitive values, configure environments, and use contexts |
| 5 | [Runners](./concepts/runners.md) | How runners execute jobs, hosted and self-hosted options, and troubleshooting |
| 6 | [Pre-Built Actions](./concepts/prebuilt-actions.md) | What actions do, where to find them, and how to use them safely |
| 7 | [Variables and Contexts](./concepts/variables-and-contexts.md) | Built-in contexts, default variables, inputs, outputs, and expressions |

## Hands-on labs

| Lab | Topic | What you'll do |
| :---: | :--- | :--- |
| [Lab 01: First Workflow](./labs/lab01-first-workflow.md) | Workflow basics | Run a workflow on push or manually, then inspect its steps and logs |
| [Lab 02: Job Dependencies and Artifacts](./labs/lab02-job-dependencies-artifacts.md) | Multiple jobs | Connect jobs and pass files between them |
| [Lab 03: Secrets and Environments](./labs/lab03-secrets-and-environments.md) | Deployment configuration | Use secrets and protect a deployment environment |
| [Lab 04: Pre-Built Actions](./labs/lab04-prebuilt-actions.md) | Reusable actions | Check out code, set up Python, and upload an artifact using actions |
| [Lab 05: Custom Actions and Reusable Workflows](./labs/lab05-custom-composite-action.md) | Workflow reuse | Create a composite action and call a reusable workflow |
| [Lab 06: Built-In Variables](./labs/lab06-built-in-variables.md) | Runtime information | Read repository, event, run, and runner values in a workflow |

## Suggested learning path

Read the concepts in order, then complete Labs 01–06. Labs 04–06 build on the workflow basics from Lab 01.

## Repository folders

- [`concepts/`](./concepts/) — explanations and reference guides
- [`labs/`](./labs/) — step-by-step exercises
- [`img/`](./img/) — diagrams used by the guides
