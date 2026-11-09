# Lab 01: Git Basic Workflow & File Lifecycle

---

## 🎯 Lab Objectives
In this hands-on lab, you will:
1. Initialize a new Git repository from scratch (`git init`).
2. Move files through the 3 Git areas (**Workspace** &rarr; **Staging Area** &rarr; **Local Repository**).
3. Observe file state transitions using `git status`.
4. Inspect line-by-line file changes with `git diff`.
5. Record permanent snapshots using `git commit` and review history with `git log`.

---

## 📋 Prerequisites
- Git client installed on your computer ([Lab 00: Installation Guide](./lab00-installation.md)).
- Terminal access (Git Bash, PowerShell, macOS Terminal, or Linux terminal).

---

## 🚀 Step-by-Step Instructions

### Step 1: Create a Project Directory & Initialize Git
Open your terminal and create a new project directory called `my-web-app`, then navigate into it:

```bash
mkdir -p ~/git-practice/my-web-app
cd ~/git-practice/my-web-app
```

Now, initialize Git inside this folder:
```bash
git init
```

**Expected Output:**
```text
Initialized empty Git repository in /Users/username/git-practice/my-web-app/.git/
```

> [!NOTE]
> Run `ls -la` (or `dir /a` on Windows). You will see the hidden `.git/` directory. This is your **Local Repository**.

---

### Step 2: Check Initial Repository Status
Run `git status` to check the current state:

```bash
git status
```

**Expected Output:**
```text
On branch main

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

---

### Step 3: Create Files in the Workspace
Create two new files in your working directory:

```bash
echo "<h1>Welcome to DevOps Lab</h1>" > index.html
echo "body { font-family: Arial; }" > style.css
```

Now check `git status` again:
```bash
git status
```

**Expected Output:**
```text
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	index.html
	style.css

nothing added to commit but untracked files present (use "git add" to track)
```
> Notice that the files appear in **red** under `Untracked files`. Git sees them on your hard drive, but is not tracking them yet.

---

### Step 4: Stage Files (Workspace &rarr; Staging Area)
Stage only `index.html` first:

```bash
git add index.html
git status
```

**Expected Output:**
```text
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   index.html

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	style.css
```
> Notice the difference:
> - `index.html` is **green** under `Changes to be committed` (Staging Area).
> - `style.css` is still **red** under `Untracked files` (Workspace only).

Now stage `style.css` (or stage all files at once with `git add .`):
```bash
git add .
git status
```

---

### Step 5: Create Your First Commit (Staging Area &rarr; Local Repo)
Save the staged snapshot into the permanent `.git` history with a clear commit message:

```bash
git commit -m "feat: initial commit with index.html and style.css"
```

**Expected Output:**
```text
[main (root-commit) a1b2c3d] feat: initial commit with index.html and style.css
 2 files changed, 2 insertions(+)
 create mode 100644 index.html
 create mode 100644 style.css
```

Now verify the workspace is clean:
```bash
git status
```
```text
On branch main
nothing to commit, working tree clean
```

---

### Step 6: View Commit History
Inspect your newly created commit:

```bash
# Full detailed log
git log
```

```bash
# Concise one-line log (recommended for daily use)
git log --oneline
```

**Expected Output:**
```text
a1b2c3d (HEAD -> main) feat: initial commit with index.html and style.css
```

---

### Step 7: Modify a File & Inspect Differences (`git diff`)
Add a new paragraph to `index.html`:

```bash
echo "<p>Git makes tracking changes simple.</p>" >> index.html
```

Check `git status`:
```bash
git status
```
```text
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
	modified:   index.html
```

Run `git diff` to see the exact line-by-line changes in your workspace:
```bash
git diff
```

**Expected Output:**
```diff
diff --git a/index.html b/index.html
--- a/index.html
+++ b/index.html
@@ -1 +1,2 @@
 <h1>Welcome to DevOps Lab</h1>
+<p>Git makes tracking changes simple.</p>
```

---

### Step 8: Stage Modification and Inspect Staged Diff
Stage the updated `index.html`:

```bash
git add index.html
```

Now run `git diff --staged` to review what is about to be committed:
```bash
git diff --staged
```

---

### Step 9: Commit the Second Snapshot
Commit your second change:

```bash
git commit -m "docs: add description paragraph to index.html"
```

View your updated history:
```bash
git log --oneline
```

**Expected Output:**
```text
f4e5d6c (HEAD -> main) docs: add description paragraph to index.html
a1b2c3d feat: initial commit with index.html and style.css
```

---

## 🏆 Lab Challenge (Self-Check)
1. Create a third file called `README.md` containing `# My Web App`.
2. Stage and commit it with message `"docs: add README"`.
3. Run `git log --oneline` and verify you have **3 commits** in your history!

