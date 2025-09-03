# Git Command-Line Basics (Plain English Notes)

## What Git is
Git is a **distributed** version control system.  
Every developer has a full local copy (history + commits).  
You can work and view history offline, and the remote is not a single point of failure.

---

## Install + First-time Setup

```bash
# Download
# https://git-scm.com (Downloads tab)

# Check installation
git --version

# Identify yourself (stored in global config)
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# See config
git config --list
```

Plain English: After installing, confirm with `git --version`.  
Set your name/email so teammates see who made each commit.  
`git config --list` shows what’s set.

**Help anytime:**

```bash
git help <command>
# or
git <command> --help
```

---

## Start tracking an existing local folder

```bash
cd your-project/
git init
git status
```

Meaning: `git init` creates a hidden `.git/` folder (that’s the repo).  
`git status` shows what’s untracked/changed.

Ignore files you don’t want in the repo:  

Create `.gitignore` and add patterns (examples):

```
.DS_Store
*.log
*.pyc
.env
```

Then run:

```bash
git status
```

---

## The three areas (simple mental model)

- **Working directory**: your files.  
- **Staging area**: changes you intend to commit.  
- **Repository (committed)**: history.  

Move changes through the stages:

```bash
# Add everything changed/untracked to staging
git add -A

# Or add one file
git add path/to/file

# See what’s staged / not staged
git status

# Commit staged changes
git commit -m "Describe what changed"
```

Undo staging if needed:

```bash
# Unstage one file (keep edits)
git reset path/to/file

# Unstage everything (keep edits)
git reset
```

See what changed:

```bash
git diff           # unstaged changes
git diff --staged  # staged changes
git log            # commit history
```

---

## Clone and work with a remote

```bash
# Clone a remote repo into current folder
git clone <REMOTE_URL> .

# See remotes
git remote -v

# See branches (local + remote)
git branch -a
```

Meaning: `git clone` copies the project + history locally.  
`git remote -v` shows URLs of remotes (e.g., origin).  
`git branch -a` lists branches.

Typical edit-commit-push cycle:

```bash
# after editing files...
git add -A
git commit -m "Fix multiply()"
git pull            # get others' changes first to avoid conflicts
git push            # send your commits to the remote
```

Note: Many repos use `main` (not `master`) as the default branch:  
`git push origin main`

---

## Branching workflow (feature branches)

```bash
# Create a feature branch
git branch calc-divide

# Switch to it
git checkout calc-divide
# (newer Git: git switch -c calc-divide)

# Make changes, then:
git add -A
git commit -m "Implement divide()"

# First push of a new branch (sets upstream)
git push -u origin calc-divide
```

Meaning: You develop on a separate branch, push it to the remote, and (in teams) open a PR.  
`-u` remembers the upstream so later you can just `git push` / `git pull`.

Merge back to main:

```bash
git checkout main
git pull                      # stay current
git merge calc-divide         # fast-forward or merge commit
git push origin main
```

Delete the finished branch:

```bash
# Local
git branch -d calc-divide

# Remote
git push origin --delete calc-divide
```

---

## Quick “from scratch” example (all together)

```bash
# 1) init + first commit
git init
echo ".env" > .gitignore
git add -A
git commit -m "Initial commit"

# 2) feature work on a branch
git switch -c subtract
# (edit files...)
git add -A
git commit -m "Add subtract()"
git push -u origin subtract

# 3) merge to main
git switch main
git pull
git merge subtract
git push

# 4) cleanup
git branch -d subtract
git push origin --delete subtract
```

Why these steps: feature branches keep main stable.  
Upstream tracking simplifies push/pull.  
Pulling before merging prevents surprises.

---

## Tiny corrections to the transcript

- Website is **https://git-scm.com** (not “g-s cm”).  
- Help forms are `git help <verb>` or `git <verb> --help`.  
- Many repos now use **main** as default branch.  
