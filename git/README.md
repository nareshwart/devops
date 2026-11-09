# Git & Version Control

Welcome to the **Git** module. This module is divided into two focused sections:
- **`concepts/`**: Core architecture, mental models, and visual diagrams (Theory)
- **`labs/`**: Step-by-step hands-on exercises with terminal commands & expected output (Practice)
- **`img/`**: Shared diagrams, charts, and architectural assets

---

## 📚 Concepts (Architecture & Mental Models)

| # | Concept Topic | Description |
|---|---|---|
| 01 | [VCS Introduction](./concepts/vcs-intro.md) | What is version control? Local vs. Centralized vs. Distributed VCS |
| 02 | [Git Local Architecture](./concepts/git-local.md) | The 3 local areas: Workspace, Staging Area (Index), and Local Repository (.git) |
| 03 | [Git Branching Concepts](./concepts/git-branching.md) | What is a branch? Why branch? `HEAD` mechanics, code isolation, and parallel workflows |
| 04 | [Git Merge Strategies](./concepts/git-merge-strategies.md) | Fast-Forward (`--ff`), 3-Way merge (`ort`), and Squash merge (`--squash`) |
| 05 | [Git Merge Conflicts](./concepts/git-merge-conflicts.md) | Anatomy of conflicts, conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`), resolution lifecycle |
| 06 | [Git Branching Workflows](./concepts/git-workflows.md) | Comparison of GitHub Flow, Gitflow, Trunk-Based Development, and GitLab Flow |
| 07 | [GitHub Overview & Mental Model](./concepts/github-intro.md) | What is GitHub? Git vs GitHub, cloud hosting, Pull Requests, Actions, and repo anatomy |

---

## 🛠️ Labs (Hands-On Practice)

| # | Lab | Focus Area |
|---|---|---|
| 00 | [Git Installation & Setup](./labs/lab00-installation.md) | Install Git client on Windows, macOS, or Linux + global identity setup |
| 01 | [Git Basic Workflow & File Lifecycle](./labs/lab01-basic-workflow.md) | Practice `git init`, `status`, `add`, `commit`, `diff`, and `log` from scratch |
| 02 | [Creating, Switching & Managing Local Branches](./labs/lab02-branching-local.md) | Create, list, switch, isolate changes, and delete local branches |
| 03 | [Practicing Git Merge Strategies](./labs/lab03-merge-strategies.md) | Hands-on Fast-Forward, No-FF (`--no-ff`), 3-Way merge, and Squash merge (`--squash`) |
| 04 | [Git Merge Conflicts & Abort Strategy](./labs/lab04-merge-conflicts.md) | Deliberately trigger, inspect, resolve conflicts, and test `git merge --abort` |
| 05 | [GitHub Basics: Create, Clone, Push & Pull](./labs/lab05-github-basics-clone-push-pull.md) | Create repo on GitHub, clone, push local code, simulate cloud edit, and pull |
| 06 | [Remote Collaboration & GitHub Flow](./labs/lab06-pull-requests-and-github-flow.md) | SSH keys, upstream tracking (`-u`), branch pushes, Pull Requests, and code reviews |
