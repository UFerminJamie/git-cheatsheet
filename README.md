# Git Command Encyclopedia — Complete Practical Reference

[Add a description about this repo]

> **Companion:** [Git Project Lifecycle Guide](project-lifecycle/README.md) walks through a project from `mkdir` to shipping, reviewing, merging and recovery.

An extensive, searchable **Git + GitHub CLI** reference organized like [joshnh/Git-Commands](https://github.com/joshnh/Git-Commands), with grouped command tables, realistic examples, warnings, and explanations. All commands run in a terminal. Commands prefixed with `gh` require the separate [GitHub CLI](https://cli.github.com/) tool; `git` itself does **not** create pull requests, issues, or hosted GitHub repositories.

**Conventions:** replace `YOUR_USERNAME`, `my-project`, branch names, remote URLs, file paths, hashes and PR numbers with real values. Examples assume `main` as the default branch and `origin` as the primary remote. Run commands from the repository root unless otherwise stated. The shell snippets use Bash/Zsh; commands such as `mkdir`, `cat`, `touch`, `printf` and `cd` are **shell commands**, not Git commands.

**Safety legend:** ✅ read-only or normally safe · ◇ writes locally/changes work, review first · ⚠️ can discard or rewrite work; make a backup and inspect the exact target.

## Table of contents

- [Mental model](#mental-model)
- [Quickstart: a new GitHub repository](#quickstart-a-new-github-repository)
- [Setup and identity](#1-setup-and-identity)
- [Help and troubleshooting](#2-help-and-troubleshooting)
- [Create / clone repositories](#3-create--clone-repositories)
- [Repository status and tracked files](#4-repository-status-and-tracked-files)
- [Stage and unstage changes](#5-stage-and-unstage-changes)
- [Commit and edit history](#6-commit-and-edit-history)
- [Review diffs](#7-review-diffs)
- [Explore history](#8-explore-history)
- [Branch and switch](#9-branch-and-switch)
- [Merge branches](#10-merge-branches)
- [Rebase and edit commits](#11-rebase-and-edit-commits)
- [Resolve conflicts](#12-resolve-conflicts)
- [Remotes](#13-remotes)
- [Fetch / pull / push](#14-fetch--pull--push)
- [Stashes](#15-stashes)
- [Undo tracked file edits](#16-undo-tracked-file-edits)
- [Reset / revert / recovery](#17-reset--revert--recovery)
- [Clean untracked files](#18-clean-untracked-files)
- [Tags](#19-tags)
- [Search / blame / bisect](#20-search--blame--bisect)
- [Cherry-pick and patches](#21-cherry-pick-and-patches)
- [Ignore and attributes](#22-ignore-and-attributes)
- [Worktrees](#23-worktrees)
- [Submodules and large files](#24-submodules-and-large-files)
- [Sparse checkouts](#25-sparse-checkouts)
- [GitHub authentication and repositories](#26-github-authentication-and-repositories)
- [GitHub pull requests and issues](#27-github-pull-requests-and-issues)
- [GitHub Actions and releases](#28-github-actions-and-releases)
- [Backups and maintenance](#29-backups-and-maintenance)
- [Advanced inspection and plumbing](#30-advanced-inspection-and-plumbing)
- [Useful configurations and aliases](#useful-configurations-and-aliases)
- [Undo decision table](#undo-decision-table)
- [References](#official-references)

## Mental model

```text
Working directory    --git add-->       Index / staging area
        ^                                       |
        |                                git commit
   git restore                                  v
        |                               Local commit history
        |                                       |
        +------------------------------ git switch / restore
                                                |
                                          git push
                                                v
                                    Remote repository (GitHub)

Remote repository --git fetch--> remote-tracking refs (origin/main)
Remote-tracking refs --git merge or git rebase--> current local branch
```

- **Working tree:** actual files on disk. **Index/staging area:** precisely what the next commit will contain. **Commit:** an immutable snapshot identified by a hash. **Branch:** movable pointer to a commit. **HEAD:** what you have checked out. **Remote:** another repository; `origin/main` is a **locally stored remote-tracking ref**, not the live server.
- `git add` stages a copy of a file's current content. Editing the file again after staging leaves the previous version staged until you stage it again.
- `git fetch` downloads remote objects and updates remote-tracking refs without integrating them into your local branch. `git pull` fetches **and then** merges or rebases, depending on options/configuration. `git push` uploads refs/commits.
- **Git is not GitHub:** Git works offline. GitHub supplies hosting, pull requests, issues, actions, permissions and web UI. The `gh` tool accesses GitHub from the terminal.

## Quickstart: a new GitHub repository

```bash
# Once per machine (replace identity)
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
gh auth login

mkdir my-project && cd my-project
git init -b main
printf '# My Project\n' > README.md
printf '__pycache__/\n.env\n' > .gitignore
git add README.md .gitignore
git commit -m "chore: initialize project"

# Make a private GitHub repo and push the existing local history.
gh repo create my-project --private --source=. --remote=origin --push

git status -sb
git remote -v
```

For a public repository use `--public`. `gh auth login` is interactive, and access/push operations may be subject to organization policies. If you already created a nonempty remote, do not blindly push over it: fetch and inspect first.

## 1. Setup and identity

Identity, editor, defaults, credentials and host-independent settings. Prefer per-repo configuration when different email identities are required.

```bash
git --version
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --list --show-origin
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git --version` | Display installed Git version. | ✅ |
| `git config --global user.name "Your Name"` | Set author name across local repositories. | ◇ |
| `git config --global user.email "you@example.com"` | Set author email for future commits. | ◇ |
| `git config user.email "work@example.com"` | Override email in the current repository. | ◇ |
| `git config --global init.defaultBranch main` | Set initial branch name for future `git init`. | ◇ |
| `git config --global core.editor "code --wait"` | Use VS Code as Git editor, if installed. | ◇ |
| `git config --global core.editor nano` | Use nano as Git editor. | ◇ |
| `git config --global pull.ff only` | Make ordinary pulls fast-forward only unless overridden. | ◇ |
| `git config --global pull.rebase true` | Default pulls to rebase (alternative to `pull.ff only`). | ◇ |
| `git config --global fetch.prune true` | Prune deleted remote-tracking refs when fetching. | ◇ |
| `git config --global rerere.enabled true` | Remember recorded conflict resolutions for reuse. | ◇ |
| `git config --list` | Print effective configuration. | ✅ |
| `git config --list --show-origin` | Display values and which config file provided each. | ✅ |
| `git config --get user.email` | Inspect one effective setting. | ✅ |
| `git config --local --list` | Inspect repository-only settings. | ✅ |
| `git config --global --unset alias.st` | Remove a configured global alias. | ◇ |
| `git config --edit --global` | Edit global Git config interactively. | ◇ |
| `git config --show-origin --get-regexp '^remote\.'` | Inspect remote settings and their origins. | ✅ |

**Notes:** Do not commit API keys, passwords, `.env` secrets or private SSH keys. Git config email is commit metadata; GitHub email privacy can use a GitHub noreply email.

## 2. Help and troubleshooting

Built-in help is authoritative for your installed Git version.

```bash
git help -a
git help restore
git status --help
git -C /path/to/repo status
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git help` | Show common Git commands. | ✅ |
| `git help -a` | List all installed command names. | ✅ |
| `git help -g` | Show Git guides and concepts. | ✅ |
| `git help branch` | Open a manual page. | ✅ |
| `git branch -h` | Show concise command-specific help. | ✅ |
| `git -C /path/to/repo status` | Run in a different directory without changing shell directory. | ✅ |
| `git rev-parse --show-toplevel` | Print repository root. | ✅ |
| `git rev-parse --is-inside-work-tree` | Check whether current directory is in a working tree. | ✅ |
| `git version --build-options` | Inspect Git build details. | ✅ |
| `GIT_TRACE=1 git fetch` | Trace a Git operation for debugging (may expose sensitive paths). | ✅ |
| `GIT_TRACE_PACKET=1 git ls-remote origin` | Trace protocol packets; avoid publishing logs with sensitive data. | ✅ |

## 3. Create / clone repositories

Initialize a repository, clone existing history or migrate a folder. Bare repositories hold Git objects but no normal worktree.

```bash
mkdir my-project && cd my-project
git init -b main
# Alternative if project already exists: cd existing-project && git init -b main
# Clone another repository:
git clone https://github.com/OWNER/REPO.git
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git init` | Initialize Git in current directory. | ◇ |
| `git init -b main` | Initialize and explicitly name the initial branch. | ◇ |
| `git init /path/to/new-repo` | Initialize Git in a named directory. | ◇ |
| `git init --bare /srv/repo.git` | Make a bare repository (for hosting/mirroring). | ◇ |
| `git clone https://github.com/OWNER/REPO.git` | Clone a repository using HTTPS. | ◇ |
| `git clone git@github.com:OWNER/REPO.git` | Clone using SSH after SSH auth setup. | ◇ |
| `git clone URL directory-name` | Clone into a custom local folder. | ◇ |
| `git clone --branch feature/example URL` | Checkout a given branch after cloning. | ◇ |
| `git clone --depth 1 URL` | Shallow clone (limited commit history). | ◇ |
| `git clone --filter=blob:none URL` | Partial clone; download blobs on demand if remote supports it. | ◇ |
| `git clone --single-branch --branch main URL` | Clone a specified branch without tracking other branches initially. | ◇ |
| `git clone --recurse-submodules URL` | Clone and initialize nested submodules recursively. | ◇ |
| `git clone --mirror URL backup.git` | Mirror all refs into a bare repo; use with care. | ◇ |
| `git clone --bare URL bare-copy.git` | Clone without a working directory. | ◇ |
| `git remote add origin URL` | Set up an external repository for an existing local one. | ◇ |

## 4. Repository status and tracked files

Inspect current state before and after every significant command. `git ls-files` is useful for explaining why `.gitignore` appears not to work.

```bash
git status -sb
git status --ignored
git ls-files | head
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git status` | Show staged, unstaged and untracked changes. | ✅ |
| `git status -sb` | Compact status including branch/tracking info. | ✅ |
| `git status --porcelain=v1` | Stable machine-readable status. | ✅ |
| `git status --ignored` | Also show ignored paths. | ✅ |
| `git ls-files` | List tracked files in index. | ✅ |
| `git ls-files --others --exclude-standard` | List untracked, non-ignored files. | ✅ |
| `git ls-files --stage` | Show index object IDs and modes. | ✅ |
| `git ls-files --unmerged` | Show conflicted index entries. | ✅ |
| `git check-ignore -v path/to/file` | Explain a matching ignore rule. | ✅ |
| `git check-ignore -v --no-index path/to/file` | Test ignore rules regardless of index tracking. | ✅ |
| `git rev-parse --abbrev-ref HEAD` | Print current branch (or HEAD if detached). | ✅ |
| `git branch --show-current` | Print currently checked-out branch name. | ✅ |
| `git status --short` | One-line per changed path. | ✅ |

## 5. Stage and unstage changes

Stage a deliberately reviewed set of changes. Interactive staging keeps unrelated edits out of a commit.

```bash
git diff
git add README.md src/main.py
git diff --cached
git restore --staged src/main.py
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git add file.py` | Stage file changes. | ◇ |
| `git add folder/` | Stage changes beneath a directory. | ◇ |
| `git add .` | Stage additions/changes/deletions under current directory. | ◇ |
| `git add -A` | Stage all additions, modifications and deletions in repository. | ◇ |
| `git add -u` | Stage modified/deleted tracked files, not new files. | ◇ |
| `git add -p` | Interactively choose diff hunks to stage. | ◇ |
| `git add -i` | Interactive staging menu. | ◇ |
| `git add -N newfile.txt` | Mark intent to add; exposes new file to diffs without staging contents. | ◇ |
| `git restore --staged file.py` | Unstage a file, retaining working-directory changes. | ◇ |
| `git restore --staged .` | Unstage all staged content, retaining working changes. | ◇ |
| `git reset HEAD -- file.py` | Older-compatible way to unstage a path. | ◇ |
| `git diff --cached` | Review staged change contents before commit. | ✅ |
| `git diff --check` | Check for whitespace errors in unstaged changes. | ✅ |
| `git diff --cached --check` | Check staged patch for whitespace errors. | ✅ |

**Notes:** `git add .` considers the current directory and below; `git add -A` considers the whole working tree. Always run `git diff --cached` to verify the upcoming snapshot.

## 6. Commit and edit history

A commit records staged content, author/committer metadata and a message. Clear messages should explain *why* a change was made, not only *what* files changed.

```bash
git add -p
git diff --cached
git commit -m "feat: add patient export"
git show --stat --oneline HEAD
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git commit -m "feat: add export"` | Commit staged snapshot with a subject line. | ◇ |
| `git commit` | Open editor for subject and body. | ◇ |
| `git commit -m "fix: validate input" -m "Reject malformed IDs before processing."` | Write subject plus explanatory body. | ◇ |
| `git commit -am "fix: correct typo"` | Stage changes to tracked files then commit (NOT untracked files). | ◇ |
| `git commit --allow-empty -m "chore: trigger pipeline"` | Record empty commit, sometimes useful for CI. | ◇ |
| `git commit --amend` | Replace the last commit using current index/editor. | ⚠️ |
| `git commit --amend --no-edit` | Add staged changes to last commit without editing its message. | ⚠️ |
| `git commit --amend -m "better subject"` | Replace last commit message (rewrites its hash). | ⚠️ |
| `git commit --no-verify -m "message"` | Bypass pre-commit and commit-msg hooks; use only by policy/exception. | ⚠️ |
| `git commit --signoff -m "fix: issue"` | Append Signed-off-by trailer (not cryptographic signing). | ◇ |
| `git commit -S -m "fix: secure patch"` | Create cryptographically signed commit if signing configured. | ◇ |
| `git commit --fixup HASH` | Make a fixup commit intended for autosquash rebase. | ◇ |
| `git log -1 --format=fuller` | Inspect author/committer/time of latest commit. | ✅ |

**Notes:** Only amend commits that have not been shared, unless collaborating on a deliberate history rewrite.

## 7. Review diffs

Choose the right comparison: unstaged vs index, staged vs HEAD, commits vs commits, and three-dot PR-style changes.

```bash
git diff
git diff --cached
git diff main...feature/example --stat
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git diff` | Show unstaged tracked changes vs index. | ✅ |
| `git diff --cached` | Show staged changes vs HEAD. | ✅ |
| `git diff HEAD` | Show all tracked working tree differences vs HEAD (staged + unstaged). | ✅ |
| `git diff -- README.md` | Diff one file (use `--` to delimit path). | ✅ |
| `git diff --stat` | Summarize unstaged file changes. | ✅ |
| `git diff --name-only` | List paths with unstaged changes. | ✅ |
| `git diff --word-diff` | Compare changed words rather than only lines. | ✅ |
| `git diff --color-words` | Highlight changed words. | ✅ |
| `git diff --check` | Detect introduced whitespace errors. | ✅ |
| `git diff HEAD~1 HEAD` | Diff last two snapshots. | ✅ |
| `git diff main feature/example` | Compare tips of two branches directly. | ✅ |
| `git diff main...feature/example` | Show changes introduced on feature since common ancestor. | ✅ |
| `git diff --cached --stat` | Summarize staged changes. | ✅ |
| `git diff --diff-filter=AM --name-only HEAD~1 HEAD` | Show added/modified paths between commits. | ✅ |
| `git diff --no-index a.txt b.txt` | Compare two paths even outside a Git repository (exit 1 if different). | ✅ |
| `git difftool` | Open configured external diff tool. | ✅ |

**Notes:** `A..B` is used in revision-set contexts such as `git log` and means commits reachable from B but not A. In `git diff A B`, A and B are two endpoints. `git diff A...B` compares the merge base of A/B against B.

## 8. Explore history

Find the change that introduced a line, locate a commit or inspect a former version without modifying your tree.

```bash
git log --oneline --decorate --graph --all -20
git show HEAD -- README.md
git log -S 'needle' --all -- src/
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git log` | View chronological commits reachable from HEAD. | ✅ |
| `git log --oneline` | Print compact commit history. | ✅ |
| `git log --oneline --graph --decorate --all` | Display branch history as a graph. | ✅ |
| `git log -10` | Show last ten commits. | ✅ |
| `git log --stat` | Include per-commit file change summary. | ✅ |
| `git log -p` | Include actual patches in history. | ✅ |
| `git log -- README.md` | Follow history of a path. | ✅ |
| `git log --follow -- README.md` | Follow path across detected renames (one file). | ✅ |
| `git log --author="Your Name"` | Filter by author name/email pattern. | ✅ |
| `git log --since="2 weeks ago"` | Filter by commit dates. | ✅ |
| `git log --grep="export"` | Search commit messages. | ✅ |
| `git log -S "function_name" --all -- src/` | Find commits changing occurrence count of text. | ✅ |
| `git log -G "regex" -- src/` | Find commits whose diff matches a regex. | ✅ |
| `git log main..feature/example` | Show commits on feature not reachable from main. | ✅ |
| `git log --left-right --cherry-pick main...feature/example` | Compare unique commits across divergent histories. | ✅ |
| `git show HASH` | Inspect named commit content and metadata. | ✅ |
| `git show HASH:path/to/file` | Print file content from an older commit. | ✅ |
| `git show --stat HEAD` | Show latest commit's change summary. | ✅ |
| `git shortlog -sn` | Count commits per author. | ✅ |
| `git describe --tags --always` | Label a commit using reachable tags when available. | ✅ |

## 9. Branch and switch

Branches isolate features/experiments. Prefer `git switch` to move between branches and `git restore` for files; older projects may use `git checkout`.

```bash
git switch main
git switch -c feature/export
git branch -vv
git switch -
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git branch` | List local branches. | ✅ |
| `git branch -a` | List local and remote-tracking branches. | ✅ |
| `git branch -r` | List remote-tracking refs. | ✅ |
| `git branch -vv` | Show local branches, tip commits and upstream tracking. | ✅ |
| `git branch feature/export` | Create branch without switching. | ◇ |
| `git switch -c feature/export` | Create and switch to new branch. | ◇ |
| `git switch main` | Checkout local main branch. | ◇ |
| `git switch -` | Switch to previous branch. | ◇ |
| `git switch -c feature/export origin/main` | Branch from latest fetched origin/main. | ◇ |
| `git switch --track origin/feature/export` | Create local tracking branch from remote branch. | ◇ |
| `git switch --detach HASH` | Inspect a commit in detached HEAD state. | ◇ |
| `git branch -m old-name new-name` | Rename a local branch. | ◇ |
| `git branch -M main` | Force rename current branch to main (may replace local name). | ⚠️ |
| `git branch -d feature/export` | Delete a fully merged local branch. | ◇ |
| `git branch -D feature/export` | Force-delete local branch even if commits not merged. | ⚠️ |
| `git branch --merged main` | List branches already merged into main. | ✅ |
| `git branch --no-merged main` | List local branches not merged into main. | ✅ |
| `git branch --set-upstream-to=origin/main main` | Set upstream for local main. | ◇ |
| `git branch --unset-upstream` | Remove current branch upstream config. | ◇ |
| `git checkout -b feature/export` | Legacy equivalent to switch -c. | ◇ |

**Notes:** A checkout may refuse to proceed if it would overwrite uncommitted changes. Commit, stash or explicitly discard before switching.

## 10. Merge branches

A merge integrates another branch into the currently checked-out branch. The command is always `git merge SOURCE` **while on destination**.

```bash
git switch main
git pull --ff-only origin main
git merge --no-ff feature/export
git log --graph --oneline -8
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git merge feature/export` | Merge feature into current branch. | ◇ |
| `git merge --ff-only feature/export` | Merge only if fast-forward possible. | ◇ |
| `git merge --no-ff feature/export` | Always produce a merge commit when integration succeeds. | ◇ |
| `git merge --squash feature/export` | Stage aggregate effect without recording merge parent; commit yourself. | ◇ |
| `git merge --no-commit feature/export` | Stop before creating an automatic merge commit when possible. | ◇ |
| `git merge --abort` | Abort in-progress conflicted merge and try to restore pre-merge state. | ◇ |
| `git merge --continue` | Finish merge after conflict resolution if appropriate. | ◇ |
| `git merge-base main feature/export` | Identify common ancestor. | ✅ |
| `git merge-tree main feature/export` | Preview tree merge information without changing worktree (Git version-dependent output). | ✅ |
| `git log --first-parent --oneline main` | View mainline history through first parents. | ✅ |

**Notes:** A merge conflict is not corruption. Resolve marked files, stage them, then use `git merge --continue` or `git commit` as prompted. Squash merge changes ancestry; it does not record the merged branch as a parent.

## 11. Rebase and edit commits

Rebase replays commits onto a new base. Good for a local feature branch; avoid rebasing branches other people are concurrently using.

```bash
git switch feature/export
git fetch origin
git rebase origin/main
# If conflict: fix files; git add ...; git rebase --continue
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git rebase main` | Replay current branch commits atop local main. | ⚠️ |
| `git rebase origin/main` | Replay commits atop fetched remote main. | ⚠️ |
| `git rebase -i HEAD~3` | Interactively reorder/squash/edit the last three commits. | ⚠️ |
| `git rebase -i --autosquash main` | Apply fixup/squash hints while replaying against main. | ⚠️ |
| `git rebase --continue` | Proceed after resolving conflict and staging. | ◇ |
| `git rebase --abort` | Return to state before current rebase if possible. | ◇ |
| `git rebase --skip` | Drop current problematic patch; can lose that commit's intended change. | ⚠️ |
| `git rebase --onto new-base old-base feature/export` | Rebase selected feature commits onto new base. | ⚠️ |
| `git range-diff main...old-tip main...new-tip` | Compare two commit series before/after a rebase. | ✅ |
| `git pull --rebase` | Fetch then rebase current branch on configured upstream. | ⚠️ |

**Notes:** Rebased commits get new hashes. If a pushed feature branch must be updated after a rebase, coordinate and use `git push --force-with-lease`, never casually overwrite shared work.

## 12. Resolve conflicts

Stop, inspect `git status`, edit conflicted files, test, stage resolved paths, then continue the active operation.

```bash
git status
git diff --name-only --diff-filter=U
# Edit conflict markers <<<<<<<, =======, >>>>>>>
git add path/to/conflicted.py
git merge --continue
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git status` | Find unmerged paths and active merge/rebase state. | ✅ |
| `git diff --name-only --diff-filter=U` | List unresolved paths. | ✅ |
| `git ls-files -u` | Inspect index stages for conflicts. | ✅ |
| `git diff --cc` | Show combined conflict diff. | ✅ |
| `git show :1:path/to/file` | Inspect common-base version during conflict. | ✅ |
| `git show :2:path/to/file` | Inspect index stage 2 (ours in normal merge). | ✅ |
| `git show :3:path/to/file` | Inspect index stage 3 (theirs in normal merge). | ✅ |
| `git mergetool` | Launch configured merge conflict helper. | ◇ |
| `git add path/to/file` | Mark file resolved after choosing/editing desired content. | ◇ |
| `git merge --continue` | Complete a paused merge. | ◇ |
| `git merge --abort` | Abort pending merge. | ◇ |
| `git rebase --continue` | Continue paused rebase. | ◇ |
| `git rebase --abort` | Abort pending rebase. | ◇ |
| `git cherry-pick --continue` | Continue paused cherry-pick. | ◇ |
| `git cherry-pick --abort` | Abort pending cherry-pick. | ◇ |

**Notes:** During a **rebase**, the meaning of `ours` / `theirs` may appear reversed relative to the branch you originally started on. Inspect each version before using a one-sided resolution.

## 13. Remotes

A remote is a named fetch/push endpoint. `origin` is a convention, not a Git keyword. Fork workflows often have both `origin` and `upstream`.

```bash
git remote -v
git remote add upstream https://github.com/ORIGINAL/REPO.git
git fetch upstream
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git remote` | List remote names. | ✅ |
| `git remote -v` | List remote fetch and push URLs. | ✅ |
| `git remote show origin` | Show remote branches, tracking and URL details. | ✅ |
| `git remote add origin URL` | Add origin to existing local repo. | ◇ |
| `git remote add upstream URL` | Add original project as upstream on a fork. | ◇ |
| `git remote set-url origin URL` | Replace remote URL (e.g. HTTPS to SSH). | ◇ |
| `git remote rename origin old-origin` | Rename a remote. | ◇ |
| `git remote remove upstream` | Remove local remote configuration; doesn't delete remote repo. | ◇ |
| `git remote get-url origin` | Print origin fetch URL. | ✅ |
| `git remote set-url --push origin PUSH_URL` | Set a separate push endpoint. | ◇ |
| `git ls-remote origin` | Inspect server refs without updating local tracking refs. | ✅ |
| `git remote prune origin` | Remove stale local origin/* remote-tracking refs. | ◇ |

## 14. Fetch / pull / push

Prefer fetching and inspecting before integrating; use `--ff-only` when you want unexpected divergence to stop rather than trigger automatic merge/rebase.

```bash
git fetch origin
git log --oneline HEAD..origin/main
git pull --ff-only origin main
# From a feature branch:
git push -u origin feature/export
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git fetch origin` | Download updates and refresh origin tracking refs. | ◇ |
| `git fetch --all` | Fetch all configured remotes. | ◇ |
| `git fetch --prune origin` | Drop stale origin remote-tracking refs while fetching. | ◇ |
| `git fetch origin main` | Fetch specific branch and update FETCH_HEAD (tracking behavior depends on refspec). | ◇ |
| `git pull` | Fetch and merge/rebase configured upstream according to config. | ◇ |
| `git pull --ff-only` | Pull only when current branch can fast-forward. | ◇ |
| `git pull --rebase` | Fetch and rebase local commits onto upstream. | ⚠️ |
| `git pull origin main` | Fetch then integrate main from origin into current branch. | ◇ |
| `git push` | Push current branch using configured tracking/push rules. | ◇ |
| `git push -u origin feature/export` | Publish feature branch and set upstream. | ◇ |
| `git push origin main` | Push local main to origin/main. | ◇ |
| `git push --all origin` | Push all local branch heads (not tags). | ⚠️ |
| `git push --tags origin` | Push all local tags (audit before doing so). | ◇ |
| `git push origin v1.0.0` | Push a specific tag. | ◇ |
| `git push origin --delete feature/export` | Delete remote branch. | ⚠️ |
| `git push --force-with-lease origin feature/export` | Rewrite remote feature if its head matches expected tracked ref. | ⚠️ |
| `git push --dry-run origin main` | Show proposed push without updating remote. | ✅ |
| `git log --oneline HEAD..@{u}` | Show upstream commits missing locally (branch must track upstream). | ✅ |
| `git log --oneline @{u}..HEAD` | Show local commits not yet upstream. | ✅ |

**Notes:** `git pull origin main` while on **feature** integrates **main into feature**; it does not switch to main. `--force-with-lease` is safer than `--force` but can still replace published history; coordinate carefully.

## 15. Stashes

Stash for temporary interruptions. A stash is not a reliable long-term backup: it is a local reference and can be dropped.

```bash
git stash push -u -m "WIP before review"
git stash list
git stash pop
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git stash push` | Stash tracked staged/unstaged changes. | ◇ |
| `git stash push -u` | Include untracked but not ignored files. | ◇ |
| `git stash push -a` | Include untracked and ignored files. | ⚠️ |
| `git stash push -m "WIP"` | Name a stash entry. | ◇ |
| `git stash push -- src/main.py` | Stash only matched path(s). | ◇ |
| `git stash list` | List stash entries. | ✅ |
| `git stash show -p stash@{0}` | Inspect full patch stored in stash. | ✅ |
| `git stash apply stash@{0}` | Reapply a stash without deleting it. | ◇ |
| `git stash pop` | Apply most recent stash then drop it if successful. | ◇ |
| `git stash branch recovery stash@{0}` | Start a branch from original stash base and apply changes. | ◇ |
| `git stash drop stash@{0}` | Delete one stash reference. | ⚠️ |
| `git stash clear` | Delete all stash entries. | ⚠️ |

## 16. Undo tracked file edits

`git restore` is the modern file-focused command. First determine whether the bad change is *unstaged*, *staged*, or already *committed*.

```bash
git diff -- src/main.py
git restore --staged src/main.py  # unstage, keep file edits
git restore -- src/main.py         # discard unstaged tracked edits
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git restore -- file.py` | Replace unstaged tracked file content from index. | ⚠️ |
| `git restore -- .` | Discard unstaged tracked changes under current directory. | ⚠️ |
| `git restore --staged file.py` | Unstage file content, preserving working-file edits. | ◇ |
| `git restore --staged .` | Unstage all staged changes under current directory. | ◇ |
| `git restore --source=HEAD~1 -- file.py` | Copy older version to working tree (doesn't commit). | ⚠️ |
| `git restore --source=HASH --staged --worktree -- file.py` | Copy version into index and working tree. | ⚠️ |
| `git restore -p file.py` | Interactively discard selected unstaged hunks. | ⚠️ |
| `git checkout HASH -- file.py` | Legacy form that updates index and worktree from commit. | ⚠️ |
| `git rm file.py` | Delete tracked file from disk and stage deletion. | ◇ |
| `git rm --cached file.py` | Stop tracking without deleting local working file. | ◇ |
| `git mv old.py new.py` | Move tracked file, stage rename/deletion/additions. | ◇ |

**Notes:** A deleted tracked file can usually be restored from a commit. Uncommitted file edits discarded with `git restore` might not be recoverable.

## 17. Reset / revert / recovery

These commands have very different semantics: **restore** files, **reset** move branch/index pointers, **revert** create an inverse commit, **reflog** locate former HEAD states. Prefer `revert` on shared history.

```bash
git log --oneline -6
git revert BAD_COMMIT_HASH              # safe shared-history undo
# If local unpublished commit only:
git reset --soft HEAD~1               # remove commit, leave staged
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git reset --soft HEAD~1` | Move HEAD back one commit, keep index and worktree. | ⚠️ |
| `git reset --mixed HEAD~1` | Move HEAD back; keep working changes but unstage them (default). | ⚠️ |
| `git reset --hard HEAD~1` | Move HEAD back and discard tracked changes to match target. | ⚠️ |
| `git reset --hard origin/main` | Make branch/index/working tree match fetched origin/main. | ⚠️ |
| `git revert HASH` | Create new commit that reverses changes of one commit. | ◇ |
| `git revert --no-commit HASH` | Apply inverse changes to index/worktree but don't commit yet. | ◇ |
| `git revert -m 1 MERGE_HASH` | Revert a merge commit relative to parent 1 (understand ancestry first). | ⚠️ |
| `git revert --continue` | Continue a conflicted revert. | ◇ |
| `git revert --abort` | Abort a conflicted revert. | ◇ |
| `git reflog` | Show local movement of HEAD including old tips. | ✅ |
| `git reflog show main` | Show local reflog for named branch. | ✅ |
| `git branch rescue HASH` | Preserve a recovered commit under a new branch name. | ◇ |
| `git fsck --no-reflogs --unreachable` | Find dangling/unreachable objects for advanced recovery. | ✅ |
| `git show HEAD@{1}` | Inspect prior reflog entry; not necessarily prior commit. | ✅ |

**Notes:** Before `reset --hard`, create a safety branch `git branch rescue-before-reset` and stash/backup any uncommitted edits. A reset after a shared push rewrites history if force-pushed. Reflog data is local and expires; do not rely on it as a permanent backup.

## 18. Clean untracked files

Only `git clean` affects untracked files. Always preview first; ignored-file cleanup can delete environments, outputs and local data.

```bash
git clean -nd             # preview untracked paths
git clean -nfdx           # preview including ignored paths
# Do NOT run deletion flags without reviewing preview!
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git clean -n` | Dry-run list of removable untracked files. | ✅ |
| `git clean -nd` | Preview untracked files and directories. | ✅ |
| `git clean -fd` | Remove untracked files/directories (not ignored). | ⚠️ |
| `git clean -nfdx` | Preview deleting ignored AND untracked files/dirs. | ✅ |
| `git clean -fdx` | Delete ignored and untracked files/dirs. | ⚠️ |
| `git clean -i` | Interactive cleanup. | ⚠️ |
| `git clean -e '*.log' -nd` | Preview cleanup while excluding matching files. | ✅ |

**Notes:** `git reset --hard` does **not** generally erase all untracked files. `git clean -fdx` is often the dangerous half of a total reset.

## 19. Tags

Use annotated tags for releases. Unlike branches, tags normally should not move after publication.

```bash
git tag -a v1.0.0 -m "First stable release"
git push origin v1.0.0
git show v1.0.0
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git tag` | List local tags. | ✅ |
| `git tag -l 'v1.*'` | Filter tags by pattern. | ✅ |
| `git tag v1.0.0` | Create lightweight tag at HEAD. | ◇ |
| `git tag -a v1.0.0 -m "Release v1.0.0"` | Create annotated tag. | ◇ |
| `git tag -a v1.0.0 HASH -m "Release"` | Annotate a specified commit. | ◇ |
| `git tag -s v1.0.0 -m "Release"` | Create signed tag if signing is configured. | ◇ |
| `git show v1.0.0` | Inspect tagged object/commit. | ✅ |
| `git push origin v1.0.0` | Publish a single tag. | ◇ |
| `git push origin --tags` | Publish all local tags. | ◇ |
| `git tag -d v1.0.0` | Delete local tag. | ⚠️ |
| `git push origin :refs/tags/v1.0.0` | Delete remote tag. | ⚠️ |
| `git describe --tags --always` | Describe HEAD relative to tags. | ✅ |

## 20. Search / blame / bisect

Track why a particular line changed and pinpoint a regression using binary search. Bisect needs reproducible good/bad criteria.

```bash
git grep -n 'TODO'
git blame -L 10,30 src/main.py
git bisect start
git bisect bad
git bisect good v1.0.0
# test midpoint, then git bisect good/bad; finally git bisect reset
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git grep -n "TODO"` | Search text in tracked content with line numbers. | ✅ |
| `git grep -n -i "error" -- '*.py'` | Case-insensitive grep limited to Python pathspecs. | ✅ |
| `git grep -n "symbol" HASH -- src/` | Search tracked paths in another commit. | ✅ |
| `git blame src/main.py` | Attribute surviving lines to commits. | ✅ |
| `git blame -L 10,30 src/main.py` | Blame selected line interval. | ✅ |
| `git blame -w src/main.py` | Ignore whitespace-only changes for blame. | ✅ |
| `git bisect start` | Enter regression search mode. | ◇ |
| `git bisect bad` | Mark current checkout as faulty. | ◇ |
| `git bisect good v1.0.0` | Mark older revision working. | ◇ |
| `git bisect good` | Mark checked midpoint good. | ◇ |
| `git bisect bad HASH` | Mark a specified midpoint bad. | ◇ |
| `git bisect skip` | Skip untestable commit. | ◇ |
| `git bisect log` | Inspect bisect sequence. | ✅ |
| `git bisect reset` | Exit bisect and restore original checkout. | ◇ |
| `git bisect run ./test-regression.sh` | Automate test (script: 0 good, 1-127 except 125 bad, 125 skip). | ◇ |

## 21. Cherry-pick and patches

Move individual commits across branches or exchange patch files. Use only when the full merge is not desired.

```bash
git switch release/1.0
git cherry-pick FIX_COMMIT_HASH
# Or create a patch file:
git format-patch -1 FIX_COMMIT_HASH
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git cherry-pick HASH` | Replay another commit onto current branch. | ◇ |
| `git cherry-pick -x HASH` | Cherry-pick and record origin commit hash in message. | ◇ |
| `git cherry-pick A^..B` | Cherry-pick inclusive contiguous commit range (verify history). | ◇ |
| `git cherry-pick -n HASH` | Apply patch without committing. | ◇ |
| `git cherry-pick --continue` | Continue after resolving cherry-pick conflict. | ◇ |
| `git cherry-pick --abort` | Undo in-progress cherry-pick operation. | ◇ |
| `git format-patch -1 HASH` | Write mailbox patch for one commit. | ◇ |
| `git format-patch main..feature/export` | Export commit series as patch files. | ◇ |
| `git am 0001-change.patch` | Apply mailbox patch as a commit. | ◇ |
| `git am --abort` | Abort patch-series application. | ◇ |
| `git apply --check change.diff` | Check whether raw patch can apply cleanly. | ✅ |
| `git apply change.diff` | Apply raw diff to worktree without commit. | ◇ |
| `git apply --reverse change.diff` | Reverse effects of raw patch. | ⚠️ |

## 22. Ignore and attributes

`.gitignore` excludes *untracked* paths from normal staging; it does not untrack things already committed. `.gitattributes` handles text rules and filters.

```bash
printf '.env\n__pycache__/\n*.log\n' >> .gitignore
git check-ignore -v .env
git rm --cached .env   # if a sensitive file was already tracked: history still contains it
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git check-ignore -v .env` | Explain whether and why file is ignored. | ✅ |
| `git status --ignored` | Show ignored entries. | ✅ |
| `git rm --cached .env` | Stop tracking file, leaving working copy; commit this change. | ◇ |
| `git rm -r --cached output/` | Remove generated directory from index only. | ◇ |
| `git add -f special.log` | Force-add an ignored file (review carefully). | ◇ |
| `git config --global core.excludesFile ~/.gitignore_global` | Use global personal ignore file. | ◇ |
| `git check-attr -a -- README.md` | Inspect effective attributes for a path. | ✅ |
| `git add --renormalize .` | Restage normalized tracked contents after attributes/line-ending change. | ◇ |

**Notes:** Removing a secret in a new commit does **not** erase its earlier versions. Rotate leaked secrets and follow a coordinated history-cleanup procedure.

## 23. Worktrees

Worktrees let the same repository maintain several checked-out branches in separate directories (great for experiments and code-review).

```bash
git worktree add ../my-project-hotfix -b hotfix/urgent main
git worktree list
# Later, after committing/stashing in alternate worktree:
git worktree remove ../my-project-hotfix
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git worktree list` | List all linked working directories and branches. | ✅ |
| `git worktree add ../project-feature feature/export` | Checkout existing branch in another directory. | ◇ |
| `git worktree add -b feature/try ../project-try main` | Create a new branch and separate directory from main. | ◇ |
| `git worktree add --detach ../project-inspect HASH` | Inspect a historical commit in another directory. | ◇ |
| `git worktree remove ../project-try` | Remove clean linked worktree. | ◇ |
| `git worktree prune` | Remove stale linked-worktree metadata. | ◇ |
| `git worktree lock ../project-try` | Protect worktree metadata from pruning (e.g. portable disk). | ◇ |
| `git worktree unlock ../project-try` | Allow pruning worktree metadata again. | ◇ |

**Notes:** Do not manually delete `.git` metadata. Git may refuse to check out the same branch in two worktrees simultaneously.

## 24. Submodules and large files

Submodules embed a reference to another repository; Git LFS is a separate extension for large binary artifacts.

```bash
git submodule add https://github.com/OWNER/DEP.git vendor/dep
git submodule update --init --recursive
# If Git LFS is installed:
git lfs install
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git submodule add URL vendor/dep` | Add external repo at a path. | ◇ |
| `git submodule status` | Show submodule commit state. | ✅ |
| `git submodule update --init --recursive` | Initialize/checkout recorded submodule commits. | ◇ |
| `git submodule update --remote vendor/dep` | Advance submodule to configured remote branch. | ◇ |
| `git submodule foreach 'git status -sb'` | Run command in each initialized submodule. | ✅ |
| `git clone --recurse-submodules URL` | Clone project and nested submodules. | ◇ |
| `git lfs install` | Install local Git LFS hooks/config (requires git-lfs). | ◇ |
| `git lfs track '*.tif'` | Track matching large files with LFS (commit .gitattributes). | ◇ |
| `git lfs ls-files` | List LFS-managed files. | ✅ |
| `git lfs pull` | Download LFS content for checked-out pointers. | ◇ |
| `git lfs status` | Inspect local LFS working/staging state. | ✅ |

**Notes:** For WSI/TIFF, checkpoints, slide arrays and patient data: verify privacy/license/policy before uploading. `.gitignore` or external object storage is often safer than putting large datasets on GitHub. Git LFS quotas/hosting restrictions apply.

## 25. Sparse checkouts

Large monorepos can check out selected paths without discarding the full Git concept of history.

```bash
git sparse-checkout init --cone
git sparse-checkout set src docs
git sparse-checkout list
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git sparse-checkout init --cone` | Enable cone-mode sparse checkout. | ◇ |
| `git sparse-checkout set src docs` | Restrict populated working paths. | ◇ |
| `git sparse-checkout add tests` | Include another directory. | ◇ |
| `git sparse-checkout list` | List selected sparse paths. | ✅ |
| `git sparse-checkout disable` | Populate full working tree and disable sparse mode. | ◇ |
| `git clone --filter=blob:none --sparse URL` | Partial clone with sparse checkout enabled. | ◇ |

## 26. GitHub authentication and repositories

The following are **GitHub CLI (`gh`)** commands, not core Git; install `gh` separately. Login/permissions may be interactive.

```bash
gh auth login
gh auth status
gh repo create my-project --private --source=. --remote=origin --push
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `gh --version` | Show installed GitHub CLI version. | ✅ |
| `gh auth login` | Authenticate GitHub CLI interactively. | ◇ |
| `gh auth status` | Display authentication state. | ✅ |
| `gh auth setup-git` | Configure Git to use GitHub CLI credential helper. | ◇ |
| `gh auth logout` | Sign out of a stored GitHub CLI session. | ◇ |
| `gh repo create my-project --private --source=. --remote=origin --push` | Create private hosted repo for current project and push. | ◇ |
| `gh repo create my-project --public --source=. --remote=origin --push` | Create public hosted repo for current project and push. | ⚠️ |
| `gh repo create my-project --private --clone` | Create and clone a new empty/private repository. | ◇ |
| `gh repo clone OWNER/REPO` | Clone a GitHub project using gh. | ◇ |
| `gh repo view --web` | Open hosted repository in browser. | ✅ |
| `gh repo view` | Inspect repository metadata. | ✅ |
| `gh repo fork OWNER/REPO --clone` | Fork a repository then clone fork. | ◇ |
| `gh repo set-default OWNER/REPO` | Set default GitHub repo for CLI operations in current directory. | ◇ |
| `gh repo sync OWNER/FORK -b main` | Sync a fork branch with upstream via GitHub (remote write). | ◇ |
| `gh browse` | Open current repository in browser. | ✅ |

**Notes:** By default create a **private** repository for unfamiliar code or datasets. For `--source=. --push`, have a local Git repository with a commit already. `gh repo create` supports interactive creation too.

## 27. GitHub pull requests and issues

A pull request (PR) is a GitHub review/integration object, not a native Git object. Review, comments and merge policy live on GitHub.

```bash
git push -u origin feature/export
gh pr create --base main --head feature/export --title "Add export" --body "Summary and tests"
gh pr checks
gh pr view --comments
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `gh pr create --fill` | Open PR using commits to suggest title/body. | ◇ |
| `gh pr create --draft --fill` | Open draft PR for work in progress. | ◇ |
| `gh pr create --base main --title "Add export" --body "Summary and validation"` | Open PR with explicit target and content. | ◇ |
| `gh pr list` | Show repository pull requests. | ✅ |
| `gh pr status` | Show own/relevant PR status. | ✅ |
| `gh pr view 12` | Inspect PR number 12. | ✅ |
| `gh pr view 12 --web` | Open PR in browser. | ✅ |
| `gh pr diff 12` | Inspect PR patch in terminal. | ✅ |
| `gh pr checks 12` | Inspect CI check outcomes. | ✅ |
| `gh pr checkout 12` | Checkout PR branch locally for testing. | ◇ |
| `gh pr comment 12 --body "Looks good after tests"` | Add conversation comment to PR. | ◇ |
| `gh pr review 12 --approve` | Submit approving review. | ◇ |
| `gh pr review 12 --request-changes --body "Please handle empty inputs"` | Request changes with context. | ◇ |
| `gh pr ready 12` | Mark a draft PR ready for review. | ◇ |
| `gh pr merge 12 --squash --delete-branch` | Squash-merge and delete feature branch where possible. | ⚠️ |
| `gh pr merge 12 --merge` | Merge with a merge commit. | ◇ |
| `gh pr merge 12 --rebase` | Rebase-merge PR through GitHub. | ◇ |
| `gh pr close 12` | Close without merging. | ◇ |
| `gh pr reopen 12` | Reopen closed, unmerged PR. | ◇ |
| `gh issue create --title "Bug: export fails" --body "Reproduction steps..."` | Create GitHub issue. | ◇ |
| `gh issue list` | List GitHub issues. | ✅ |
| `gh issue view 23` | Inspect issue. | ✅ |
| `gh issue comment 23 --body "Working on this"` | Comment on issue. | ◇ |
| `gh issue close 23` | Close issue. | ◇ |

**Notes:** Only approve a PR after review. `gh pr merge` may be blocked by repository rules or checks. PR authors may not have permission to self-approve.

## 28. GitHub Actions and releases

Extra `gh` commands for CI/CD and release workflows. Test locally first and avoid putting credentials into commands or tracked files.

```bash
gh run list
gh run view --log-failed
gh release create v1.0.0 --generate-notes
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `gh run list` | List recent workflow runs. | ✅ |
| `gh run view RUN_ID` | Inspect workflow run status and jobs. | ✅ |
| `gh run view RUN_ID --log-failed` | Inspect failed job logs. | ✅ |
| `gh run watch RUN_ID` | Watch run until it completes. | ✅ |
| `gh run rerun RUN_ID --failed` | Re-run failed jobs from a run. | ◇ |
| `gh workflow list` | List GitHub Actions workflow definitions. | ✅ |
| `gh workflow run workflow.yml` | Trigger workflow_dispatch workflow (must be supported). | ◇ |
| `gh release list` | List GitHub releases. | ✅ |
| `gh release view v1.0.0` | Inspect release. | ✅ |
| `gh release create v1.0.0 --generate-notes` | Create release for an existing or new tag. | ◇ |
| `gh release upload v1.0.0 dist/my-app.whl` | Attach a build artifact to release. | ◇ |
| `gh release download v1.0.0` | Download published release assets. | ◇ |

## 29. Backups and maintenance

Use backups for major refactors and before destructive history surgery. A clone, bundle or separate worktree is better than reliance on reflog alone.

```bash
git branch backup/pre-refactor
# Create a self-contained backup of Git refs/objects:
git bundle create ../my-project-backup.bundle --all
git bundle verify ../my-project-backup.bundle
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git branch backup/pre-refactor` | Preserve current commit under a named local branch. | ◇ |
| `git tag -a before-v2 -m "Before migration"` | Mark a checkpoint with annotated tag. | ◇ |
| `git bundle create ../repo.bundle --all` | Export reachable commits/refs into a portable bundle. | ◇ |
| `git bundle verify ../repo.bundle` | Validate bundle prerequisites. | ✅ |
| `git clone ../repo.bundle restored-repo` | Restore repository from a bundle into new directory. | ◇ |
| `git count-objects -vH` | Inspect loose/packed object storage. | ✅ |
| `git fsck --full` | Check reachable objects and integrity. | ✅ |
| `git gc` | Run garbage collection/packing; generally automatic. | ◇ |
| `git maintenance run` | Run configured Git maintenance tasks. | ◇ |
| `git repack -ad` | Repack repository objects (advanced). | ◇ |
| `git reflog expire --expire=now --all` | Remove reflog protections early; avoid before recovery. | ⚠️ |
| `git gc --prune=now` | Immediately prune eligible objects; can destroy recovery options. | ⚠️ |

**Notes:** A Git bundle/clone backs up committed Git objects, **not untracked/uncommitted working files** by itself. Copy those files separately or commit to a temporary local branch (subject to privacy rules).

## 30. Advanced inspection and plumbing

Low-level commands are useful for diagnostics and automation; most day-to-day workflows should use porcelain commands above.

```bash
git rev-parse HEAD
git cat-file -t HEAD
git ls-tree -r --name-only HEAD
```

| Command | Purpose / when to use it | Safety |
|---|---|---|
| `git rev-parse HEAD` | Show full hash of HEAD commit. | ✅ |
| `git rev-parse --verify HEAD` | Verify named revision exists; print resolved SHA. | ✅ |
| `git rev-list --count HEAD` | Count reachable commits. | ✅ |
| `git rev-list --objects --all` | Enumerate reachable object IDs and paths. | ✅ |
| `git cat-file -t HEAD` | Inspect Git object type. | ✅ |
| `git cat-file -p HEAD` | Print raw commit object content. | ✅ |
| `git cat-file -s HEAD` | Show object size in bytes. | ✅ |
| `git ls-tree -r --name-only HEAD` | List file names in committed tree. | ✅ |
| `git show-ref` | Enumerate local reference names and hashes. | ✅ |
| `git for-each-ref --format='%(refname:short)' refs/heads/` | Enumerate branch refs with format. | ✅ |
| `git symbolic-ref --short HEAD` | Return current branch ref, errors if detached. | ✅ |
| `git merge-base --is-ancestor main feature/export` | Test ancestry (exit 0 if main ancestor). | ✅ |
| `git hash-object README.md` | Calculate Git blob hash for file bytes. | ✅ |
| `git update-index --refresh` | Refresh index stat information. | ◇ |
| `git archive --format=tar -o ../source.tar HEAD` | Export snapshot without `.git` history. | ◇ |
| `git notes add -m "Review note" HASH` | Attach Git note; not pushed by default. | ◇ |

**Notes:** Avoid scripting direct ref updates (`git update-ref`, etc.) until you understand Git's object model and concurrency.

## Useful configurations and aliases

Configurations below are optional and should reflect your team's conventions:

```bash
# Concise readable history
git config --global alias.lg 'log --graph --decorate --oneline --all'
# Safer fetch hygiene
git config --global fetch.prune true
# Common conflict representation (supported on current Git)
git config --global merge.conflictStyle zdiff3
# Keep pull integration deliberate (pick one project policy)
git config --global pull.ff only
# Or use rebase-oriented pulls instead:
# git config --global --unset pull.ff
# git config --global pull.rebase true

git lg -20
```

**Conventional Commits** are an optional commit convention: `feat: add heatmap toggle`, `fix: correct tile indexing`, `docs: clarify setup`, `refactor: simplify sampler`, `test: cover edge case`, `chore: update dependencies`. Git itself does not enforce these prefixes.

## Undo decision table

| Situation | Preferred command | What survives? |
|---|---|---|
| Staged accidentally | `git restore --staged file.py` | File edits remain in working tree. |
| Unstaged tracked edit you **truly** want gone | `git restore -- file.py` | Only staged version remains; unstaged edit discarded. |
| Want file from an old commit | `git restore --source=HASH -- file.py` | New working copy of old version; then stage/commit normally. |
| Just committed locally and want to edit message | `git commit --amend` | Last commit replaced with new hash. |
| Just committed locally; keep staged changes | `git reset --soft HEAD~1` | Commit removed from current branch; content remains staged. |
| Just committed locally; keep unstaged changes | `git reset HEAD~1` | Commit removed; content remains in working tree. |
| Commit already pushed/shared | `git revert HASH` | Creates an inverse commit without rewriting shared history. |
| Accidentally moved branch back | `git reflog` → `git branch rescue HASH` | Former commit recovered if still available locally. |
| Untracked generated files | `git clean -nd` **before** `git clean -fd` | Deletion can be permanent; carefully review dry run. |
| Merge conflict in progress | `git merge --abort` | Attempts to restore pre-merge state. |
| Rebase conflict in progress | `git rebase --abort` | Returns branch to pre-rebase state if possible. |
| Local branch diverged from remote | `git fetch`, inspect log/diff, decide `merge` or `rebase` | Preserve work until reconciled deliberately. |

### Common mistakes

- **Mistaking a commit for a push:** `git commit` stores locally; `git push` publishes.
- **Thinking all modifications are staged:** `git add` takes a snapshot at the time you run it; review with `git diff --cached`.
- **Using `git pull origin main` on the wrong branch:** it integrates origin's main into the branch you have checked out.
- **Assuming `.gitignore` removes committed files:** it only affects untracked file matching.
- **Assuming `git revert` returns the whole project to an old date:** revert undoes specific patches via new commits; restoring a whole tree is a separate workflow.
- **Forgetting `git fetch`:** remote-tracking refs can be stale.
- **Force-pushing a shared branch:** prefer immutable public history and a revert PR.
- **Blindly deleting WSI data, models, or environments:** use `git clean -n...` to inspect first; large artifacts may not be versioned.
- **Copying a command containing `HASH` or `YOUR_USERNAME` literally:** placeholders are not valid arguments.

## Official references

- [Git command reference](https://git-scm.com/docs) and [Pro Git book](https://git-scm.com/book/en/v2)
- [Git basics (GitHub Docs)](https://docs.github.com/en/get-started/using-git/about-git)
- [GitHub CLI manual](https://cli.github.com/manual/)
- [git-restore](https://git-scm.com/docs/git-restore), [git-reset](https://git-scm.com/docs/git-reset), [git-revert](https://git-scm.com/docs/git-revert), [git-reflog](https://git-scm.com/docs/git-reflog)
- Inspiration for the **grouped-table format**: [joshnh/Git-Commands](https://github.com/joshnh/Git-Commands). This guide is a substantially expanded, independently written reference.

_Updated October 2026. Availability of advanced options depends on your installed Git and GitHub CLI versions. Consult `git help COMMAND` or `gh COMMAND --help` for local details._

