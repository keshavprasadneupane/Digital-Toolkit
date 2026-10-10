# Git Cheat Sheet

A practical reference for everyday Git commands, version control workflows, branching, remote repositories, and troubleshooting.

## 1. Initial Configuration

| Command                                            | Description                                                              |
| -------------------------------------------------- | ------------------------------------------------------------------------ |
| `git --version`                                    | Display the installed Git version.                                       |
| `git config --global user.name "Your Name"`        | Set your global commit author name.                                      |
| `git config --global user.email "you@example.com"` | Set your global commit author email.                                     |
| `git config --global --list`                       | Display global Git configuration.                                        |
| `git config --list`                                | Display effective configuration values.                                  |
| `git config --global init.defaultBranch main`      | Set `main` as the default branch for new repositories.                   |
| `git config --global core.editor "code --wait"`    | Configure VS Code as the Git editor, if the `code` command is available. |

## 2. Creating a Repository

| Command                                  | Description                                           |
| ---------------------------------------- | ----------------------------------------------------- |
| `git init`                               | Initialize a repository in the current directory.     |
| `git init project-name`                  | Create a directory and initialize a repository in it. |
| `git clone <repository-url>`             | Clone a remote repository.                            |
| `git clone <repository-url> <directory>` | Clone into a specified directory.                     |
| `git status`                             | Show the working tree and staging status.             |
| `git status -sb`                         | Display a concise branch and working-tree summary.    |

## 3. Staging and Committing

| Command                   | Description                                                                                       |
| ------------------------- | ------------------------------------------------------------------------------------------------- |
| `git add <file>`          | Stage a specific file.                                                                            |
| `git add .`               | Stage changes under the current directory.                                                        |
| `git add -A`              | Stage additions, modifications, and deletions throughout the working tree.                        |
| `git add -p`              | Interactively stage selected parts of changes.                                                    |
| `git diff`                | Show unstaged changes.                                                                            |
| `git diff --staged`       | Show staged changes that will go into the next commit.                                            |
| `git commit -m "message"` | Commit staged changes with a message.                                                             |
| `git commit`              | Open the configured editor to write a commit message.                                             |
| `git commit --amend`      | Replace the latest commit with a new commit containing the staged changes and/or updated message. |
| `git log`                 | Show commit history.                                                                              |
| `git log --oneline`       | Show a compact commit history.                                                                    |
| `git show <commit>`       | Display a commit and its changes.                                                                 |

**Tip:** Review `git status` and `git diff --staged` before committing.

## 4. Branching and Merging

| Command                       | Description                                                         |
| ----------------------------- | ------------------------------------------------------------------- |
| `git branch`                  | List local branches.                                                |
| `git branch -a`               | List local and remote-tracking branches.                            |
| `git branch <branch-name>`    | Create a branch.                                                    |
| `git switch <branch-name>`    | Switch to an existing branch.                                       |
| `git switch -c <branch-name>` | Create and switch to a new branch.                                  |
| `git merge <branch-name>`     | Merge the specified branch into the current branch.                 |
| `git branch -d <branch-name>` | Delete a branch that Git considers safely merged.                   |
| `git branch -D <branch-name>` | Force-delete a local branch; potentially discards unmerged work.    |
| `git merge --abort`           | Abort a merge in progress when Git can restore the pre-merge state. |

Prefer `git switch` for changing branches. Older commands such as `git checkout` can also switch branches, but combine several different operations.

## 5. Remote Repositories

| Command                                      | Description                                                           |
| -------------------------------------------- | --------------------------------------------------------------------- |
| `git remote -v`                              | List configured remotes and their URLs.                               |
| `git remote add origin <repository-url>`     | Add a remote named `origin`.                                          |
| `git remote set-url origin <repository-url>` | Change the URL of an existing remote.                                 |
| `git fetch`                                  | Download remote updates without merging them into the current branch. |
| `git fetch --all`                            | Fetch updates from all configured remotes.                            |
| `git pull`                                   | Fetch and integrate updates from the configured upstream branch.      |
| `git pull --rebase`                          | Fetch and replay local commits on top of the updated upstream branch. |
| `git push`                                   | Push the current branch to its configured upstream, when one exists.  |
| `git push -u origin <branch-name>`           | Push a branch and set its upstream tracking branch.                   |
| `git push origin --delete <branch-name>`     | Delete a branch from the remote repository.                           |
| `git ls-remote origin`                       | List references advertised by the remote.                             |

**Note:** `git pull` integrates changes, whereas `git fetch` only downloads them. Review your working tree before pulling.

## 6. Undoing Changes

Choose the command that matches what you want to undo. Some operations permanently discard work.

| Command                       | Description                                                                                             |
| ----------------------------- | ------------------------------------------------------------------------------------------------------- |
| `git restore <file>`          | Discard unstaged changes to a tracked file.                                                             |
| `git restore --staged <file>` | Unstage a file while preserving its working-tree changes.                                               |
| `git reset --soft HEAD~1`     | Move the current branch back one commit while keeping changes staged.                                   |
| `git reset HEAD~1`            | Move the current branch back one commit and leave the changes unstaged in the working tree.             |
| `git reset --hard HEAD~1`     | Move the current branch back one commit and discard affected tracked working-tree changes. Destructive. |
| `git revert <commit>`         | Create a new commit that reverses the specified commit's changes.                                       |
| `git reflog`                  | Show recent local reference movements, useful for recovering from mistakes.                             |
| `git stash`                   | Temporarily save tracked working-tree changes and the index state.                                      |
| `git stash -u`                | Stash changes, including untracked files.                                                               |
| `git stash list`              | List saved stashes.                                                                                     |
| `git stash pop`               | Apply the latest stash and remove it from the stash list if successful.                                 |
| `git stash apply`             | Apply the latest stash while keeping it in the stash list.                                              |

**Important:** Avoid `git reset --hard`, force-pushing, and similar destructive operations unless you understand their impact. Prefer `git revert` when undoing a commit that has already been shared.

## 7. Tags and Releases

| Command                                 | Description                                     |
| --------------------------------------- | ----------------------------------------------- |
| `git tag`                               | List local tags.                                |
| `git tag <tag-name>`                    | Create a lightweight tag at the current commit. |
| `git tag -a v1.0.0 -m "Release v1.0.0"` | Create an annotated release tag.                |
| `git show <tag-name>`                   | Inspect a tag and its associated commit.        |
| `git push origin <tag-name>`            | Push a specific tag to the remote.              |
| `git push origin --tags`                | Push local tags to the remote.                  |

Annotated tags are generally preferable for release versions because they store tag metadata and a message.

## 8. Inspecting History and Changes

| Command                                | Description                                                                                      |
| -------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `git log --oneline --graph --decorate` | Display a compact, graph-oriented commit history.                                                |
| `git log -n 10`                        | Show the latest 10 commits.                                                                      |
| `git diff <branch-1>..<branch-2>`      | Compare the changes between two branch tips.                                                     |
| `git diff HEAD`                        | Compare the working tree with the current commit, including staged and unstaged tracked changes. |
| `git blame <file>`                     | Show the last-commit attribution for each line.                                                  |
| `git show --stat <commit>`             | Summarize the files changed in a commit.                                                         |
| `git log -- <file>`                    | Show commits affecting a particular file.                                                        |
| `git diff --check`                     | Check for whitespace errors in unstaged changes.                                                 |

## 9. `.gitignore`

Use a `.gitignore` file to prevent selected untracked files from being added to Git.

Example:

```
# Python
__pycache__/
*.py[cod]
.venv/
venv/

# Environment variables and secrets
.env
.env.*
!.env.example

# Editors
.vscode/
.idea/

# Logs
*.log

# Operating system files
.DS_Store
Thumbs.db
```

| Command                          | Description                                                             |
| -------------------------------- | ----------------------------------------------------------------------- |
| `git check-ignore -v <file>`     | Show which ignore rule matches a file.                                  |
| `git rm --cached <file>`         | Stop tracking a file while retaining it on disk.                        |
| `git rm -r --cached <directory>` | Stop tracking a directory and its contents while retaining local files. |

**Note:** `.gitignore` does not automatically untrack files that have already been committed. Remove them from the index when appropriate, then commit the change. If a secret was committed, revoke or rotate it; deleting the file alone does not remove it from Git history.

## 10. Useful Diagnostics

| Command                 | Description                                                     |
| ----------------------- | --------------------------------------------------------------- |
| `git status`            | Inspect repository state.                                       |
| `git remote -v`         | Verify remote URLs.                                             |
| `git branch -vv`        | Show local branches and their upstream tracking status.         |
| `git diff --check`      | Find whitespace errors in unstaged changes.                     |
| `git fsck`              | Check repository object connectivity and validity.              |
| `git count-objects -vH` | Inspect loose objects and repository object-storage statistics. |
| `git reflog --all`      | Inspect recent reference movements across references.           |

## 11. Recommended Workflows

### Start a New Feature

```
git switch main
git pull --ff-only
git switch -c feat/my-feature
```

Work on your changes, then stage and commit them:

```
git status
git diff
git add <file>
git diff --staged
git commit -m "feat: describe the new feature"
git push -u origin feat/my-feature
```

Replace `main` with the appropriate base branch if your project uses a different one.

### Update an Existing Branch

```
git status
git fetch
git pull
```

Resolve any conflicts if necessary, inspect the changes, and continue working.

### Commit a Bug Fix

```
git status
git diff
git add <file>
git diff --staged
git commit -m "fix: describe the bug fix"
```

### Inspect Before Pushing

```
git status -sb
git log --oneline -5
git diff @{upstream}...HEAD
git push
```

The comparison command assumes the current branch has an upstream configured. If it does not, inspect the relevant remote branch explicitly.

## 12. Commit Message Conventions

A common convention is:

```
<type>: <short description>
```

| Type        | Purpose                                              |
| ----------- | ---------------------------------------------------- |
| `feat:`     | Add a feature.                                       |
| `fix:`      | Fix a bug.                                           |
| `docs:`     | Update documentation.                                |
| `refactor:` | Restructure code without changing intended behavior. |
| `perf:`     | Improve performance.                                 |
| `test:`     | Add or update tests.                                 |
| `build:`    | Change build systems or dependencies.                |
| `ci:`       | Change continuous integration workflows.             |
| `chore:`    | Perform routine maintenance.                         |

Examples:

```
git commit -m "feat: add user authentication"
git commit -m "fix: handle empty configuration"
git commit -m "docs: update installation guide"
git commit -m "refactor: simplify data processing"
```

These prefixes are conventions, not requirements enforced by Git.

## 13. Most Frequently Used Commands

These are the commands to remember for everyday Git work.

| Command                       | When to use it                                    |
| ----------------------------- | ------------------------------------------------- |
| `git status`                  | Check what's changed.                             |
| `git add .`                   | Stage changes under the current directory.        |
| `git add -A`                  | Stage all changes across the working tree.        |
| `git commit -m "message"`     | Save staged changes in a commit.                  |
| `git pull`                    | Fetch and integrate upstream changes.             |
| `git push`                    | Upload local commits to the configured remote.    |
| `git log --oneline -5`        | Inspect recent commits.                           |
| `git diff`                    | Review unstaged changes.                          |
| `git diff --staged`           | Review staged changes.                            |
| `git switch <branch>`         | Switch branches.                                  |
| `git switch -c <branch>`      | Create and switch to a new branch.                |
| `git fetch`                   | Download remote updates without integrating them. |
| `git stash`                   | Temporarily save tracked changes.                 |
| `git restore --staged <file>` | Unstage a file without discarding its edits.      |
| `git remote -v`               | Check remote repository URLs.                     |

### The Everyday Cycle

```
git status
git add .
git diff --staged
git commit -m "type: describe changes"
git push
```

If you are collaborating with others, check for upstream changes before starting or pushing work.

---

**Git Cheat Sheet** — A practical reference for everyday version control.

Add a conflict-resolution workflowClarify pull and rebase choices