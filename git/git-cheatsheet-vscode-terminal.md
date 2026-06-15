# Git Cheatsheet — VS Code Terminal

> A practical Git reference for **multi-branch workflows**: stage · commit · push · pull · merge · rebase · stash
> Designed for daily use from the **VS Code integrated terminal**.

---

## 🔍 Always Start Here — Status & Orientation

```bash
git status                              # Where am I? What changed?
git branch -a                           # List local and remote branches
git log --oneline --graph --all -15     # Compact commit history with branch graph
git remote -v                           # Show configured remotes
```

---

## 🌿 Branches

```bash
# Create and switch to a new branch
git checkout -b feature/branch-name     # Classic syntax
git switch -c feature/branch-name       # Modern syntax

# Switch branch
git checkout main
git switch develop

# Rename a branch
git branch -m old-name new-name

# Delete a local branch
git branch -d feature/branch-name       # Delete only if already merged
git branch -D feature/branch-name       # Force delete

# Delete a remote branch
git push origin --delete feature/branch-name

# Track a remote branch
git checkout --track origin/feature/remote-branch

# Modern alternative
git switch --track -c feature/remote-branch origin/feature/remote-branch
```

---

## ➕ Stage Changes

```bash
git add src/myfile.py                   # Stage a single file
git add .                               # Stage changes in the current directory
git add -A                              # Stage all changes in the repository
git add *.py                            # Stage files by pattern
git add -p src/myfile.py                # Interactive staging, hunk by hunk
```

### Unstage Without Losing Changes

```bash
git restore --staged src/myfile.py
git restore --staged .
```

### Compare Changes

```bash
git diff                                # Unstaged changes
git diff --staged                       # Staged changes vs last commit
```

---

## ✅ Commit

```bash
git commit -m "feat: add login endpoint"
```

### Stage Tracked Files and Commit in One Step

```bash
git commit -am "fix: correct null check"
```

> Note: `git commit -am` only stages **already tracked files**.
> It does not include new/untracked files.

### Open Editor for a Multi-line Commit Message

```bash
git commit
```

### Amend the Last Commit

Use this only before pushing, or only on personal branches if you know what you are doing.

```bash
git commit --amend -m "fix: correct commit message"
git commit --amend --no-edit            # Keep same message, add currently staged files
```

### Empty Commit

Useful to trigger CI/CD pipelines.

```bash
git commit --allow-empty -m "chore: trigger pipeline"
```

### Undo the Last Commit

```bash
git reset --soft HEAD~1                 # Undo commit, keep changes staged
git reset HEAD~1                        # Undo commit, keep changes unstaged
```

### Conventional Commit Prefixes

```text
feat:      New feature
fix:       Bug fix
chore:     Maintenance task
docs:      Documentation only
refactor:  Code change without behavior change
test:      Tests
style:     Formatting only, no logic change
perf:      Performance improvement
ci:        CI/CD changes
build:     Build system or dependency changes
```

---

## 🚀 Push

```bash
git push                                # Push current branch if already tracked
git push -u origin feature/branch-name  # First push of a new branch
git push origin feature/branch-name     # Explicit push
```

### Force Push Safely

Use only after operations like rebase, and preferably only on personal branches.

```bash
git push --force-with-lease
```

> Prefer `--force-with-lease` over `--force`: it protects you from overwriting someone else’s remote changes.

### Tags

```bash
git tag -a v1.2.0 -m "Release v1.2.0"   # Create annotated tag
git push origin v1.2.0                  # Push a specific tag
git push --tags                         # Push all local tags
```

### Rarely Recommended

```bash
git push --all origin                   # Push all local branches
```

> Use with care. In most team workflows, pushing only the branch you are working on is safer.

---

## ⬇️ Pull & Fetch

```bash
git pull                                # Fetch + merge current tracked branch
git pull --rebase                       # Fetch + rebase for cleaner history
git pull origin develop                 # Pull develop into the current branch
```

> Important: `git pull origin develop` does not switch to `develop`.
> It pulls `origin/develop` into your current branch.

### Fetch Without Merging

```bash
git fetch origin
git fetch --all --prune                 # Fetch all remotes and remove stale references
```

### Inspect Incoming Commits Before Merging/Rebasing

```bash
git log HEAD..origin/main --oneline
```

### Recommended for Feature Branches

```bash
git pull --rebase origin main
```

This updates your feature branch by replaying your commits on top of the latest `main`.

---

## 🔀 Merge

### Standard Merge

Switch to the target branch first, then merge.

```bash
git switch main
git pull origin main
git merge feature/branch-name
```

### Merge with Explicit Merge Commit

```bash
git merge --no-ff feature/branch-name -m "Merge: feature/branch-name into main"
```

### Squash Merge

Compress all commits from a branch into one staged change.

```bash
git switch main
git pull origin main
git merge --squash feature/branch-name
git commit -m "feat: add squashed feature"
```

### Abort a Merge in Progress

```bash
git merge --abort
```

### Resolve Merge Conflicts

After editing conflicted files:

```bash
git add src/conflicted-file.py          # Mark conflict as resolved
git commit                              # Complete the merge
```

---

## 🔁 Rebase

Rebase is often used to keep feature branch history clean.

```bash
git switch feature/branch-name
git fetch origin
git rebase origin/main
```

### During a Rebase Conflict

After fixing the conflicted files:

```bash
git add src/conflicted-file.py
git rebase --continue
```

### Abort a Rebase

```bash
git rebase --abort
```

### Push After Rebase

If the branch was already pushed before the rebase:

```bash
git push --force-with-lease
```

---

## 💾 Temporary Save — Stash

```bash
git stash                               # Save current changes
git stash push -m "WIP: login form validation"
git stash push -u -m "WIP: include untracked files"
```

### Manage Stashes

```bash
git stash list                          # List stashes
git stash apply                         # Apply latest stash, keep it in stash list
git stash pop                           # Apply latest stash and remove it from stash list
git stash apply stash@{2}               # Apply a specific stash
git stash drop stash@{0}                # Delete a specific stash
git stash clear                         # Delete all stashes
```

---

## 🔄 Daily Workflow — Feature Branch

```bash
# 1. Start of the day — sync main
git switch main
git pull origin main

# 2. Create or switch to your feature branch
git switch -c feature/my-feature        # New branch
git switch feature/my-feature           # Existing branch

# 3. Work, then stage and commit
git add .
git commit -m "feat: implement X"

# 4. Keep your branch updated with main
git fetch origin
git rebase origin/main

# 5a. First push
git push -u origin feature/my-feature

# 5b. Next pushes
git push

# 6. Open a Pull Request / Merge Request on GitHub or GitLab

# 7. After the PR is merged — cleanup
git switch main
git pull origin main
git branch -d feature/my-feature
```

Optional remote cleanup:

```bash
git push origin --delete feature/my-feature
```

---

## ↩️ Undo & Fix Mistakes

### Discard Changes in a File

```bash
git restore src/myfile.py
```

### Discard All Tracked Local Changes

```bash
git restore .
```

> This restores tracked files only.
> It does not remove untracked files.

### Remove Untracked Files

```bash
git clean -fd
```

> Warning: this is destructive. It permanently removes untracked files and directories.

### Revert a Commit Safely

Creates a new commit that undoes a previous commit.
Safe for shared branches.

```bash
git revert abc1234
```

### Reset to a Specific Commit

```bash
git reset --soft abc1234                # Move back, keep changes staged
git reset --mixed abc1234               # Move back, keep changes unstaged
git reset --hard abc1234                # Move back and discard changes
```

> Warning: `git reset --hard` is destructive.
> Avoid it on shared branches unless you know exactly what you are doing.

### Recover a Deleted Branch

```bash
git reflog                              # Find the last commit hash
git switch -c recovered-branch abc1234
```

---

## ⚙️ One-Time VS Code Setup

### Identity

For public GitHub repositories, consider using a GitHub noreply email instead of a company email.

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Example with GitHub noreply:

```bash
git config --global user.email "12345678+username@users.noreply.github.com"
```

### VS Code as Editor and Diff/Merge Tool

```bash
git config --global core.editor "code --wait"

git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'

git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
```

### Default Branch Name

```bash
git config --global init.defaultBranch main
```

### Rebase Automatically on Pull

```bash
git config --global pull.rebase true
```

### Credentials

Recommended:

```bash
git config --global credential.helper manager
```

Alternative, only on trusted machines:

```bash
git config --global credential.helper store
```

> Warning: `store` saves credentials in plain text.

### Verify Configuration

```bash
git config --global --list
```

---

## ⚡ Quick Reference

| Action                       | Command                                     |
| ---------------------------- | ------------------------------------------- |
| Check status                 | `git status`                                |
| New branch + switch          | `git switch -c feature/name`                |
| Switch branch                | `git switch branch-name`                    |
| Stage everything             | `git add -A`                                |
| Commit                       | `git commit -m "message"`                   |
| Stage tracked files + commit | `git commit -am "message"`                  |
| First push                   | `git push -u origin branch-name`            |
| Next pushes                  | `git push`                                  |
| Pull with rebase             | `git pull --rebase`                         |
| Merge branch into main       | `git switch main && git merge feature/name` |
| Save WIP                     | `git stash push -m "WIP"`                   |
| Restore stash                | `git stash pop`                             |
| Undo last commit             | `git reset --soft HEAD~1`                   |
| Discard file changes         | `git restore filename`                      |
| Compact branch graph         | `git log --oneline --graph --all -15`       |

---

## 💡 VS Code Tip

Open the integrated terminal with:

```text
Ctrl + `
```

Then run Git commands directly from the project folder.
