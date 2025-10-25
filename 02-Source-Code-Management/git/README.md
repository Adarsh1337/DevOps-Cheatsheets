# 🧩 Git Cheatsheet

[← Back to Main ToC](../../README.md)

## Core Concepts
- Repositories: local vs remote, clone, fork, origin/upstream
- Commits: snapshots, messages, hashes
- Branching model: main, feature, release, hotfix
- Staging area: working tree → index → commit
- Remotes: add, fetch, push, pull

## Setup
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase false
```

## Create & Clone
```bash
git init                    # Initialize local repo
git clone URL               # Clone
```

## Status & Diff
```bash
git status                  # What changed
git diff                    # Unstaged diff
git diff --staged           # Staged diff
```

## Add, Commit, Amend
```bash
git add file1 dir/          # Stage changes
git add -p                  # Interactive stage
git commit -m "message"     # Commit
git commit --amend          # Amend last commit (message or content)
```

## Branching
```bash
git branch                  # List branches
git branch -m old new       # Rename branch
git switch -c feature/x     # Create and switch
git switch main             # Switch branches
git merge feature/x         # Merge into current
```

## Syncing Remotes
```bash
git remote -v               # List remotes
git remote add origin URL   # Add remote

git fetch origin            # Update refs
git pull --ff-only          # Fast-forward only
# or
git pull --rebase           # Rebase local commits on top

git push -u origin main     # First push set upstream
git push                    # Push current branch
```

## Reset, Restore, Revert
```bash
# Discard unstaged file changes
git restore --source=HEAD -- path/to/file
# Unstage
git restore --staged path/to/file
# Move branch pointer (keep or drop work)
git reset --soft HEAD~1     # Keep index and working tree
git reset --mixed HEAD~1    # Keep working tree, reset index
git reset --hard HEAD~1     # Discard everything
# Undo a public commit safely
git revert <commit>
```

## Stash
```bash
git stash push -m "wip"     # Stash tracked changes
git stash list              # Show stashes
git stash apply             # Apply latest
git stash pop               # Apply and drop
```

## Logs and Inspect
```bash
git log --oneline --graph --decorate --all
git show <commit>
git blame path/file         # Who changed which line
```

## Ignore
```
# .gitignore examples
*.log
.env
__pycache__/
node_modules/
```

## Tags & Releases
```bash
git tag v1.0.0              # Lightweight tag
git tag -a v1.0.0 -m "v1"   # Annotated tag
git push origin v1.0.0
```

## Rebasing & Conflicts
```bash
git rebase main             # Rebase current onto main
git rebase --continue       # After resolving conflicts
git rebase --abort          # Abort rebase

# Conflict resolution
# 1) Edit files to resolve
# 2) git add <resolved files>
# 3) Continue rebase/merge
```

## Submodules
```bash
git submodule add URL path
git submodule update --init --recursive
```

## Hooks (client-side)
- .git/hooks/pre-commit: lint/tests
- .git/hooks/commit-msg: enforce message format

## Best Practices & Gotchas
- Write clear, imperative commit messages
- Keep commits small and logical
- Rebase feature branches before merging
- Avoid force-pushing shared branches
- Use protected branches and PR reviews
- Always pull with rebase or fast-forward to maintain linear history
- Use .gitignore early; do not commit secrets
