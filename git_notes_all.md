# Git Notes

## 1. Git Basics

### What Git is  
Git is a **distributed version control system**. Every developer has a full local copy (all history + commits).  
- You can work and view history offline.  
- The remote repo is not a single point of failure.  

### Install + Setup  
- Download: https://git-scm.com (Downloads tab)  
- Check installation: **`git --version`**  
- Set identity:  
  - **`git config --global user.name "Your Name"`**  
  - **`git config --global user.email "you@example.com"`**  
- View config: **`git config --list`**  

👉 After installing, confirm with `git --version`. Name/email makes commits traceable to you.

### Help  
- **`git help <command>`**  
- **`git <command> --help`**

### Start tracking a project  
- Enter folder: **`cd your-project/`**  
- Init repo: **`git init`** → creates `.git/` folder  
- Status: **`git status`**  

### Ignore unwanted files  
Create `.gitignore`:  
```
.DS_Store
*.log
*.pyc
.env
```  
Now Git won’t track these.

### Git’s three areas  
- **Working directory**: actual files  
- **Staging area**: what you’ve queued for commit  
- **Repository**: permanent history  

### Add & Commit  
- Stage all: **`git add -A`**  
- Stage one: **`git add file`**  
- Commit staged: **`git commit -m "message"`**  

### Undo staging  
- Unstage one: **`git reset file`**  
- Unstage all: **`git reset`**  

### See changes & history  
- **`git diff`** → unstaged changes  
- **`git diff --staged`** → staged changes  
- **`git log`** → commit history  

### Work with remotes  
- Clone: **`git clone <URL> .`**  
- Show remotes: **`git remote -v`**  
- Show branches: **`git branch -a`**  

### Edit–Commit–Push cycle  
- Edit → **`git add -A`** → **`git commit -m "msg"`**  
- Update from remote: **`git pull`**  
- Push your commits: **`git push`**  

👉 Many repos now use **main** (not master). Example: `git push origin main`.

### Branch workflow  
- New branch: **`git branch feature`**  
- Switch: **`git checkout feature`** (or `git switch -c feature`)  
- Work & commit as usual.  
- First push: **`git push -u origin feature`**  
- Merge back:  
  - **`git checkout main`**  
  - **`git pull`**  
  - **`git merge feature`**  
  - **`git push origin main`**  
- Delete branch:  
  - Local: **`git branch -d feature`**  
  - Remote: **`git push origin --delete feature`**

---

## 2. Undo & Rollback

### Fix mistakes locally  
- Bad commit message: **`git commit --amend -m "Correct message"`**  
- Forgot file: **`git add file && git commit --amend`**  

### Wrong branch  
- Move commit: checkout correct branch → **`git cherry-pick <hash>`**  
- Remove from wrong branch: **`git reset`** (see below)  

### Reset types  
- **Soft** → keep changes staged: **`git reset --soft <hash>`**  
- **Mixed** (default) → keep changes unstaged: **`git reset <hash>`**  
- **Hard** → discard all changes, match commit: **`git reset --hard <hash>`**  

### Clean up junk  
- Remove untracked files/dirs: **`git clean -df`**

### Recover lost commits  
- View reflog: **`git reflog`**  
- Checkout old commit: **`git checkout <hash>`**  
- Save it: **`git branch rescue`**

### Safe undo when pushed  
- Create a new commit that reverses another: **`git revert <hash>`**  

---

### 🔑 Cheat Row (Undo & Rollback)  
- Wrong commit msg → `git commit --amend -m "Correct msg"`  
- Missed file → `git add file && git commit --amend`  
- Wrong branch → `git cherry-pick <hash>` then reset wrong branch  
- Undo commit keep staged → `git reset --soft HEAD~1`  
- Undo commit keep unstaged → `git reset HEAD~1`  
- Throw away commit + changes → `git reset --hard HEAD~1`  
- Remove untracked → `git clean -df`  
- Recover lost → `git reflog` → checkout hash → `git branch rescue`  
- Safe undo (already pushed) → `git revert <hash>`  

---

## 3. Git Stash

### What stash is  
Temporary storage for changes you don’t want to commit yet. Lets you switch branches or clean up without losing work.  

### Core commands  
- Save changes: **`git stash save "msg"`**  
- List: **`git stash list`**  
- Apply (keep stash): **`git stash apply stash@{0}`**  
- Apply (and drop): **`git stash pop`**  
- Drop one: **`git stash drop stash@{0}`**  
- Drop all: **`git stash clear`**

### Example  
1. Work on a file → `git diff` shows edits.  
2. Save work: `git stash save "add function"` → file resets clean.  
3. Switch branch safely.  
4. Restore changes: `git stash pop`.  
5. Multiple stashes stack with indexes (`stash@{0}`, `stash@{1}` …).  

### Common use case  
Made edits on wrong branch?  
- `git stash save "msg"`  
- `git checkout correct-branch`  
- `git stash pop` → now you can commit on correct branch.

---

### 🔑 Cheat Row (Stash)  
- Save work → `git stash save "msg"`  
- See list → `git stash list`  
- Apply only → `git stash apply stash@{id}`  
- Apply + drop → `git stash pop`  
- Drop one → `git stash drop stash@{id}`  
- Drop all → `git stash clear`  

---

**End of Notes**  
