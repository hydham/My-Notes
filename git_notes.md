# Git Command-Line Basics

## What Git is
Git is a **distributed** version control system. Every developer has a full local copy (history + commits).  
You can work and view history offline, and the remote is not a single point of failure.

---

## Install and First-time Setup

```bash
# Download Git from:
https://git-scm.com

# Check installation
git --version

# Identify yourself (stored in global config)
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# See all config values
git config --list
```

**Help anytime:**

```bash
git help <command>
git <command> --help
```

---

## Start Tracking an Existing Local Folder

```bash
cd your-project/
git init
git status
```

- `git init` creates a hidden `.git/` folder (the repository).  
- `git status` shows what’s untracked or changed.

Ignore files you don’t want in the repo:

Create `.gitignore` and add patterns:

```
.DS_Store
*.log
*.pyc
.env
```

Then check with:

```bash
git status
```

---

## The Three Areas in Git

- **Working directory**: your actual files  
- **Staging area**: changes prepared for commit  
- **Repository**: committed history

```bash
# Add everything to staging
git add -A

# Add one file
git add file.txt

# Commit with a message
git commit -m "Initial commit"

# Unstage one file (keep edits)
git reset file.txt

# See differences
git diff           # unstaged changes
git diff --staged  # staged changes
git log            # commit history
```

---

## Clone and Work with a Remote

```bash
# Clone a repository
git clone <REMOTE_URL> .

# See remotes
git remote -v

# See branches (local + remote)
git branch -a
```

Typical workflow:

```bash
# Make changes, then
git add -A
git commit -m "Fix multiply()"
git pull     # update from remote first
git push     # push your commits
```

---

## Branching Workflow

```bash
# Create a new branch
git branch feature-branch

# Switch to it
git checkout feature-branch
# (or: git switch -c feature-branch)

# Commit changes
git add -A
git commit -m "Add new feature"

# Push the branch to remote
git push -u origin feature-branch
```

Merging back to main:

```bash
git checkout main
git pull
git merge feature-branch
git push origin main
```

Delete the branch when done:

```bash
git branch -d feature-branch
git push origin --delete feature-branch
```

---

## Quick Example (all together)

```bash
# Init repo
git init
echo ".env" > .gitignore
git add -A
git commit -m "Initial commit"

# Work on feature branch
git switch -c subtract
# edit files...
git add -A
git commit -m "Add subtract()"
git push -u origin subtract

# Merge back
git switch main
git pull
git merge subtract
git push

# Cleanup
git branch -d subtract
git push origin --delete subtract
```

---

### Notes
- Default branch may be **main** instead of **master** in modern Git.  
- Use `git help` or `git <command> --help` for manuals.  
- Distributed = every developer has a full backup.  
