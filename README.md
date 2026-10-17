# Git Project Lifecycle — From Zero to Production and Recovery

> **Companion reference:** [Git Command Encyclopedia](../command-reference/README.md) contains 400+ command examples organized into 30 grouped tables. This guide instead explains **when, why, and in what order to run them**.

A hands-on terminal guide following a project from its first directory, initial commit and GitHub repository to feature development, code review, PR comments, merges, releases, hotfixes and multiple kinds of rollback. Written for one developer and small teams; the same concepts apply to research, Python packages, websites and larger applications.

**Assumptions:** a Bash/Zsh shell on macOS/Linux (Windows users can use Git Bash or WSL); Git and [GitHub CLI](https://cli.github.com/) installed; the hosted repository is on GitHub; primary branch is called `main`; examples use `my-project` and `feature/add-greeting`. `git` runs locally; `gh` talks to GitHub. Replace placeholders such as `YOUR_USERNAME`, `HASH`, `12` (a PR number) and `URL` before running commands.

**Safety:** this guide deliberately separates *inspect* commands from potentially destructive *change* commands. Never paste an entire guide indiscriminately into a terminal. In particular, `git restore`, `git reset --hard`, `git clean -fdx`, `git branch -D` and force-pushes can remove work.

## Contents

1. [Understand the complete lifecycle](#1-understand-the-complete-lifecycle)
2. [Install and configure your tools](#2-install-and-configure-your-tools)
3. [Create a project from nothing](#3-create-a-project-from-nothing)
4. [Create the first commit](#4-create-the-first-commit)
5. [Publish it as a new GitHub repository](#5-publish-it-as-a-new-github-repository)
6. [Alternative: start from an existing GitHub repository](#6-alternative-start-from-an-existing-github-repository)
7. [Start a normal day: update main safely](#7-start-a-normal-day-update-main-safely)
8. [Create a feature branch](#8-create-a-feature-branch)
9. [Change code and inspect exactly what changed](#9-change-code-and-inspect-exactly-what-changed)
10. [Stage only the changes you intend](#10-stage-only-the-changes-you-intend)
11. [Write meaningful commit messages](#11-write-meaningful-commit-messages)
12. [Push your feature branch to GitHub](#12-push-your-feature-branch-to-github)
13. [Open a pull request and describe your work](#13-open-a-pull-request-and-describe-your-work)
14. [Comment on the PR and respond to review](#14-comment-on-the-pr-and-respond-to-review)
15. [When main changes while you are working](#15-when-main-changes-while-you-are-working)
16. [Handle merge conflicts](#16-handle-merge-conflicts)
17. [Merge a pull request](#17-merge-a-pull-request)
18. [Update your local project and clean up branches](#18-update-your-local-project-and-clean-up-branches)
19. [Direct local merge, without GitHub PRs](#19-direct-local-merge-without-github-prs)
20. [Releases and tags](#20-releases-and-tags)
21. [Hotfixes and cherry-picks](#21-hotfixes-and-cherry-picks)
22. [Pause work using a stash](#22-pause-work-using-a-stash)
23. [Recover from almost every Git mistake](#23-recover-from-almost-every-git-mistake)
24. [Inspect who changed what, when, and why](#24-inspect-who-changed-what-when-and-why)
25. [Back up code before a major refactor](#25-back-up-code-before-a-major-refactor)
26. [Collaboration: forks, teammates and upstream](#26-collaboration-forks-teammates-and-upstream)
27. [Large files, research data, privacy and secrets](#27-large-files-research-data-privacy-and-secrets)
28. [A practical Git workflow for each workday](#28-a-practical-git-workflow-for-each-workday)
29. [One-page command decision matrix](#29-one-page-command-decision-matrix)
30. [Glossary and further reading](#30-glossary-and-further-reading)

---

## 1. Understand the complete lifecycle

```text
Prepare machine
   |
   v
Create local project ----or---- clone an existing repository
   |
   v
Initialize Git / inspect existing history
   |
   v
Add .gitignore, README, source and tests
   |
   v
Review changes -> stage -> commit
   |
   v
Create remote GitHub repository -> first push
   |
   v
Update main -> new feature branch -> implement/test
   |
   v
Review diff -> stage -> commit -> push feature branch
   |
   v
Create pull request -> comments and reviews -> fix problems
   |
   v
Resolve integration conflicts -> pass tests -> merge PR
   |
   v
Pull main -> delete obsolete branches -> tag/release
   |
   v
Repeat / hotfix / revert / recover / back up
```

Git maintains three important states before publishing:

| State | What it contains | Typical command |
|---|---|---|
| Working tree | Files you are currently editing | `git status`, `git diff` |
| Staging area (index) | The exact file content selected for the *next commit* | `git add`, `git diff --cached` |
| Local history | Immutable snapshots plus parent relationships | `git commit`, `git log` |
| Remote-tracking refs | Your last fetched view of a remote branch | `git fetch`, `git log origin/main` |
| GitHub repository | Hosted copies of pushed commits and hosted collaboration tools | `git push`, `gh pr create` |

A commit is **not** a push. A push is **not** a merge. A GitHub pull request is **not** a Git command. Branch names are pointers; committing on one branch does not automatically modify another.

## 2. Install and configure your tools

### 2.1 Check installation

```bash
git --version
gh --version
```

**Explanation:** `git` handles local revision control. `gh` is optional for Git itself but used here to create a hosted repository and conduct GitHub PR review entirely from the command line. If `gh` is not present, install [GitHub CLI](https://cli.github.com/) first.

Typical macOS/Homebrew installation:

```bash
brew install git gh
```

Typical Ubuntu/Debian Git installation:

```bash
sudo apt update
sudo apt install git
```

For GitHub CLI on Linux, use the [official installation instructions](https://github.com/cli/cli/blob/trunk/docs/install_linux.md) appropriate to your distribution rather than assuming an arbitrary package mirror.

### 2.2 Set author identity once

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global core.editor nano

git config --global --list
```

**Explanation:** Git writes author details into commit metadata. Use an email you want associated with commits. The `--global` flag applies to your user account; omit it inside one repository to set project-specific identity. Change `nano` to `"code --wait"` if you have the VS Code CLI and prefer it.

**If using a different identity in a work repository:**

```bash
cd /path/to/work-project
git config user.name "Your Work Name"
git config user.email "work@example.com"
git config --get user.email
```

### 2.3 Authenticate GitHub CLI

```bash
gh auth login
gh auth status
```

Choose GitHub.com and a protocol during interactive login. The browser-based sign-in is generally easiest. On private repositories, permissions and organization policies may restrict repository creation or pushing.

### 2.4 Optional SSH access

```bash
# Only create a key if you do not already have an appropriate SSH key.
ssh-keygen -t ed25519 -C "you@example.com"

# Ask GitHub CLI to add the public key (after gh auth login):
gh ssh-key add ~/.ssh/id_ed25519.pub --title "My laptop"

# Test GitHub SSH access:
ssh -T git@github.com
```

**Explanation:** HTTPS authentication through GitHub CLI is also fine. Never share or commit `~/.ssh/id_ed25519` (the **private** key). When SSH testing, GitHub may acknowledge authentication but report that shell access is unavailable; this is expected.

---

## 3. Create a project from nothing

The next commands construct a small **Python** example so that later commits and tests are meaningful. Replace these files with your actual project structure for a web application, package, research pipeline or another language.

````bash
mkdir my-project
cd my-project
mkdir -p src tests

# Create a basic project README.
cat > README.md <<'MARKDOWN'
# My Project

A tiny example of a version-controlled Python project.

## Run

```bash
python src/hello.py
```

## Test

```bash
python -m unittest discover -s tests -v
```
MARKDOWN

# Create a Python file.
cat > src/hello.py <<'PY'
def greet(name: str) -> str:
    return f"Hello, {name}!"


if __name__ == "__main__":
    print(greet("world"))
PY

# Write a small unit test.
cat > tests/test_hello.py <<'PY'
import sys
import unittest
from pathlib import Path

sys.path.insert(0, str(Path(__file__).resolve().parents[1] / "src"))
from hello import greet


class GreetingTests(unittest.TestCase):
    def test_greeting(self):
        self.assertEqual(greet("world"), "Hello, world!")


if __name__ == "__main__":
    unittest.main()
PY
````

**Explanation:** `mkdir`, `cat` and `cd` are **shell commands**, not Git. This is a concrete program and test that you can run before making the first commit.

### 3.1 Create `.gitignore` early

```bash
cat > .gitignore <<'IGNORE'
# Python
__pycache__/
*.py[cod]
.pytest_cache/
.venv/
venv/

# Environment variables / private credentials
.env
.env.*
!.env.example
*.pem

# Editors / OS
.DS_Store
.vscode/

# Large/generated data artifacts (adapt to project)
data/
results/
checkpoints/
*.svs
*.tif
*.tiff
*.h5
*.hdf5
*.pt
*.pth
IGNORE
```

**Explanation:** `.gitignore` controls whether Git normally offers matching **untracked** files to be added. It does not remove files already tracked in past commits. Decide data policies for your specific project: sometimes small checked-in fixtures are appropriate, but real patient datasets, WSI files and large checkpoints need special handling.

### 3.2 Run a test before Git initialization

```bash
python src/hello.py
python -m unittest discover -s tests -v
```

Expect `Hello, world!` and a passing test. A good first commit should represent a working state whenever possible.

---

## 4. Create the first commit

### 4.1 Initialize the Git repository

```bash
git init -b main
git branch --show-current
git status
```

**Explanation:** `git init -b main` creates a hidden `.git/` database and names the initial branch `main`. The branch does not yet point to a commit. Do not edit `.git` by hand.

### 4.2 Review and stage initial files

```bash
git status --short
git add README.md .gitignore src/ tests/
git diff --cached --stat
git diff --cached
```

**Explanation:** `git add` moves the content to the index. `git diff --cached` is the actual change set that the upcoming commit will capture. Check for accidental credentials, `.env` files, huge binaries or temporary data. Review this every time, not just on day one.

### 4.3 Write the initial commit

```bash
git commit -m "chore: initialize Python project"
git status -sb
git log --oneline --decorate -5
```

**Explanation:** `git commit` stores a new snapshot in the local history. A clean `git status` means no tracked working changes or untracked non-ignored files are currently pending. You can make multiple local commits without internet access.

### What these five everyday commands do

| Command | Role in the first commit |
|---|---|
| `git init -b main` | Make a local repository and name the initial branch. |
| `git status` | Check which files are new/modified/staged. |
| `git add ...` | Choose the content of the next snapshot. |
| `git diff --cached` | Inspect that exact staged snapshot. |
| `git commit -m "..."` | Save the selected snapshot into local history. |

---

## 5. Publish it as a new GitHub repository

### 5.1 Preferred: one command with GitHub CLI

```bash
# Still inside the local my-project/ folder:
gh repo create my-project --private --source=. --remote=origin --push
```

**Explanation:** Creates a **private** GitHub repository, adds its URL as the `origin` remote and pushes your existing local branch/commit. Confirm the selected owner/account in interactive prompts. For an intentionally public codebase, replace `--private` with `--public` **after verifying you have no private data**. To create under an organization you can administer, use `gh repo create ORG_NAME/my-project --private --source=. --remote=origin --push` instead.

Confirm:

```bash
git remote -v
git branch -vv
git status -sb
gh repo view --web
```

### 5.2 Alternative: manually connect to an empty GitHub repository

If you already created a completely empty GitHub repository, without a generated README, LICENSE or `.gitignore`:

```bash
# Use the remote URL shown by GitHub, not this literal placeholder:
git remote add origin https://github.com/YOUR_USERNAME/my-project.git

git remote -v
git push -u origin main
```

**Explanation:** `remote add` only creates local URL configuration. `git push -u` sends `main` and remembers which upstream branch to track. If the remote **already has a commit**, first `git fetch origin`, inspect ancestry and integrate intentionally; don't assume you can fast-forward or overwrite the remote.

### 5.3 Alternative: let GitHub CLI create an empty remote first

```bash
# Creates an empty private remote project (may prompt for details).
gh repo create my-project --private

# If it did NOT add a remote to your local repository, connect it:
git remote -v
git remote add origin https://github.com/YOUR_USERNAME/my-project.git
git push -u origin main
```

Only run `git remote add origin ...` if `git remote -v` confirms `origin` does not already exist. For an existing remote, use `git remote set-url origin URL` if the URL needs correction.

### 5.4 HTTPS vs SSH

```bash
# Inspect current origin.
git remote get-url origin

# Optional change from HTTPS to SSH after SSH key setup:
git remote set-url origin git@github.com:YOUR_USERNAME/my-project.git
```

Changing the remote URL doesn't change any commits or create a repository.

---

## 6. Alternative: start from an existing GitHub repository

**Skip Steps 3–5** when the GitHub repository already exists and has content you want to work with.

```bash
cd ~/Projects  # use an existing directory you own, or choose another path
git clone https://github.com/YOUR_USERNAME/my-project.git
cd my-project

git remote -v
git status -sb
git log --oneline --decorate -8
```

**Explanation:** `git clone` copies the Git history, creates a working tree, configures `origin` and normally checks out the remote's default branch. If the repository is private, the account needs read permission.

To clone by SSH:

```bash
git clone git@github.com:YOUR_USERNAME/my-project.git
```

To clone into a chosen directory:

```bash
git clone https://github.com/YOUR_USERNAME/my-project.git local-project-folder
```

**Do not** run `git init` again inside an existing clone. It is already a Git repository.

---

## 7. Start a normal day: update main safely

Start from a clean tree before syncing a long-running branch:

```bash
git status -sb
git switch main
git fetch origin
git log --oneline --left-right main...origin/main
git pull --ff-only origin main
git status -sb
```

**What happens:**

1. `git status` reveals local edits before branch switching.
2. `git switch main` puts you on your integration branch.
3. `git fetch origin` downloads changes without automatically applying them to `main`.
4. `git log --left-right main...origin/main` shows commits unique to either side of the divergence.
5. `git pull --ff-only origin main` updates `main` **only if a fast-forward is possible**. Divergence stops with an error rather than silently merging unrelated commits.

Do not attempt `git pull` from a dirty tree without understanding how the merge/rebase would interact with your edits. Commit, stash or back up work first.

### Fetch vs pull vs push

| Command | Main effect | Updates your checked-out files? |
|---|---|---|
| `git fetch origin` | Download remote objects/update remote-tracking refs | **No** (normally) |
| `git pull --ff-only origin main` | Fetch then fast-forward your current branch to origin/main if possible | **Yes** |
| `git pull --rebase` | Fetch and replay your local commits over the upstream | **Yes**, rewrites local commit IDs |
| `git push origin main` | Upload commits/ref to origin/main if accepted | Not normally |

---

## 8. Create a feature branch

Never start a significant change directly on `main` if you expect peer review.

```bash
git switch main
git pull --ff-only origin main
git switch -c feature/add-greeting

git branch --show-current
git branch -vv
```

**Explanation:** Creates a new branch pointing at the current `main` commit and checks it out. Future commits will advance only `feature/add-greeting` until you merge or otherwise integrate them.

Branch naming examples:

| Purpose | Example branch |
|---|---|
| Feature | `feature/add-heatmap-layer` |
| Bug fix | `fix/tile-coordinate-offset` |
| Documentation | `docs/install-instructions` |
| Experiment | `experiment/new-sampler` |
| Emergency repair | `hotfix/export-crash` |
| Code cleanup | `refactor/deduplicate-parser` |

For an experimental change you may want a checkpoint as well:

```bash
git branch backup/before-greeting-experiment
```

This branch preserves **committed** state. It does not save uncommitted or untracked files.

---

## 9. Change code and inspect exactly what changed

Let's make a small update. This command overwrites the example `src/hello.py` intentionally, so use it **only** in the toy project from this guide.

```bash
cat > src/hello.py <<'PY'
def greet(name: str, excited: bool = False) -> str:
    message = f"Hello, {name}!"
    return message.upper() if excited else message


if __name__ == "__main__":
    print(greet("world"))
PY
```

Add a test for the new behavior:

```bash
cat > tests/test_hello.py <<'PY'
import sys
import unittest
from pathlib import Path

sys.path.insert(0, str(Path(__file__).resolve().parents[1] / "src"))
from hello import greet


class GreetingTests(unittest.TestCase):
    def test_default_greeting(self):
        self.assertEqual(greet("world"), "Hello, world!")

    def test_excited_greeting(self):
        self.assertEqual(greet("world", excited=True), "HELLO, WORLD!")


if __name__ == "__main__":
    unittest.main()
PY
```

Inspect and test:

```bash
git status -sb
git diff
git diff --stat
git diff --check
python -m unittest discover -s tests -v
```

**Explanation:** `git diff` shows tracked, unstaged changes against the index. `git diff --stat` summarizes changed line counts and paths; `git diff --check` flags whitespace problems. A passing unit test provides evidence the branch still works.

To compare your entire current work to the last commit, including staged changes:

```bash
git diff HEAD
```

To see only filenames:

```bash
git diff --name-only
```

---

## 10. Stage only the changes you intend

```bash
git add src/hello.py tests/test_hello.py
git diff --cached --stat
git diff --cached
git status -sb
```

**Explanation:** `git add` freezes the current versions of those paths into the index. If you edit the same file **again** after staging, the new content will not be staged until you call `git add` again.

For files containing both wanted and unwanted hunks:

```bash
git add -p src/hello.py
```

**Explanation:** Interactive patch staging offers to stage or skip individual hunks. Use this to make logically focused commits.

If you accidentally staged a file:

```bash
git restore --staged tests/test_hello.py
```

This **unstages**, but does **not** erase the file's edits. To stage it again:

```bash
git add tests/test_hello.py
```

**Before every commit:** ask whether all staged files belong to one understandable change, and verify no credentials/large files are staged.

---

## 11. Write meaningful commit messages

```bash
git commit -m "feat: support excited greetings"
git log -1 --format=fuller
git show --stat HEAD
```

**Explanation:** A commit message is a permanent description attached to the snapshot. It's different from a PR review **comment**; GitHub PR comments are online discussion.

### Commit message pattern

```text
<type>: <short imperative summary>

Why this change was needed, what constraints matter,
and what reviewer should look for.

Refs: #123
```

Use an editor for a longer message:

```bash
git commit
```

Or supply a subject plus body at the command line:

```bash
git commit -m "fix: validate empty names" \
  -m "Avoid passing blank names into the greeting formatter. Add a regression test."
```

### Conventional message examples

| Prefix | Example | Intended meaning |
|---|---|---|
| `feat:` | `feat: add export endpoint` | New user-facing capability |
| `fix:` | `fix: correct indexing error` | Bug repair |
| `docs:` | `docs: document Docker setup` | Documentation change |
| `test:` | `test: cover missing metadata` | Tests |
| `refactor:` | `refactor: simplify parser` | Structural cleanup without intended behavior change |
| `perf:` | `perf: cache tile embeddings` | Performance improvement |
| `chore:` | `chore: update dependencies` | Maintenance |
| `ci:` | `ci: run tests on PRs` | Build/test automation |

These prefixes are a community convention, **not a Git requirement**. Use your team's policy if different.

### Fix the *last local, unpublished* commit

```bash
# Edit files first, then:
git add src/hello.py
git commit --amend --no-edit
```

**Important:** amend replaces the commit and changes its hash. Avoid amending after others have based their work on it; prefer another commit in a shared branch.

---

## 12. Push your feature branch to GitHub

```bash
git branch --show-current
git push -u origin feature/add-greeting
git status -sb
git branch -vv
```

**Explanation:** `-u` sets the upstream branch so later feature pushes can simply be:

```bash
git push
```

If you have already pushed and made another normal commit, Git can fast-forward the remote branch with another `git push`.

**If a push is rejected:** somebody may have pushed to that branch first, or branch protections may apply. Fetch, compare, and resolve the divergence. Do **not** respond by blindly using `git push --force`.

To see unpushed commits once an upstream exists:

```bash
git log --oneline @{u}..HEAD
```

To see remote commits you haven't integrated:

```bash
git fetch origin
git log --oneline HEAD..@{u}
```

---

## 13. Open a pull request and describe your work

You now have a hosted feature branch. Open a **pull request** targeting `main`:

```bash
gh pr create \
  --base main \
  --head feature/add-greeting \
  --title "feat: support excited greetings" \
  --body "Adds an optional excited greeting mode and unit tests."
```

**Explanation:** A PR asks for feature-branch changes to be reviewed and integrated into `main`. It does not merge them immediately. The CLI prints a URL and may prompt for missing details; some organizations require reviewers, statuses, approvals and protected branches.

For an early work-in-progress PR:

```bash
gh pr create --draft --base main --fill
```

For a finished draft PR:

```bash
gh pr ready
```

To inspect the PR:

```bash
gh pr status
gh pr view
gh pr diff
gh pr checks
gh pr view --web
```

### Suggested PR description template

```markdown
## Summary
- Add an optional excited greeting mode.
- Preserve existing behavior by default.

## Why
Users need an explicit way to emphasize greetings without breaking old calls.

## Tests
- [x] `python -m unittest discover -s tests -v`

## Notes for reviewers
Please check that the default argument preserves backward compatibility.
```

**Issue linking:** Include `Fixes #123` in a PR body when you intend GitHub to close the issue upon merging the PR. Use real issue numbers, and confirm the link is appropriate.

---

## 14. Comment on the PR and respond to review

A **commit message** goes in Git history. A **PR comment** is a GitHub conversation entry. Both matter in real projects.

### 14.1 Add a general PR conversation comment

```bash
# Replace 12 with the actual PR number.
gh pr comment 12 --body "Implemented the requested validation and added tests."
```

### 14.2 Review somebody else's PR

```bash
gh pr view 12
gh pr diff 12
gh pr checks 12

# Only after you have reviewed and are authorized to approve:
gh pr review 12 --approve
```

Request changes with an explanation:

```bash
gh pr review 12 --request-changes \
  --body "Please add a test for a missing or empty input."
```

Leave a non-blocking review comment:

```bash
gh pr review 12 --comment \
  --body "The interface is clear; consider adding a short usage example."
```

### 14.3 Fix review feedback on your feature branch

```bash
git switch feature/add-greeting

# Edit source/tests, then check:
git diff
python -m unittest discover -s tests -v

git add src/ tests/
git diff --cached
git commit -m "test: cover greeting edge cases"
git push
```

**Explanation:** New commits pushed to the same feature branch are automatically included in its existing PR. Normally you do **not** open another PR for each review fix.

### 14.4 Inline comments

For comments on a particular line in the GitHub Files changed UI, use:

```bash
gh pr view 12 --web
```

Then open **Files changed**, select the changed line, and submit the review comment. GitHub's line-level conversation is more context-specific than the CLI's general PR comment. This guide deliberately avoids pretending a plain Git command supports inline PR comments.

---

## 15. When main changes while you are working

Your teammate may merge other work before your feature is ready. You have two mainstream choices; pick your team's policy.

### Option A: merge latest main into your feature (doesn't rewrite feature commits)

```bash
git switch feature/add-greeting
git fetch origin
git merge origin/main

# If needed, resolve conflicts and stage files; then:
# git merge --continue

git push
```

**Explanation:** Preserves feature commit hashes and adds a merge commit if histories diverge. Good for shared feature branches, though history may be busier.

### Option B: rebase feature on latest main (cleaner linear history, rewrites commits)

```bash
git switch feature/add-greeting
git fetch origin
git rebase origin/main

# If needed, resolve conflicts and stage files; then:
# git rebase --continue
```

**Explanation:** Replays your feature commits on top of latest `origin/main` and gives them new hashes. Ideal for a local unpublished feature, but needs coordination if the feature was already pushed.

If the feature was already pushed, and you consciously rebased it:

```bash
# ONLY after ensuring no one else is relying on the remote feature branch:
git push --force-with-lease origin feature/add-greeting
```

`--force-with-lease` checks the remote tip against your expected version, but still rewrites remote history. Never force-push protected `main` or somebody else's shared branch as a shortcut.

### Which to use?

| Topic | Merge main into feature | Rebase feature onto main |
|---|---|---|
| Preserves existing feature SHA hashes | Yes | No |
| History shape | Merge commit can appear | Linear replay |
| Feature already shared with collaborators | Usually safer | Needs coordination |
| Push after successful operation | Ordinary `git push` | May require `--force-with-lease` |
| Can have conflicts | Yes | Yes, potentially during several replayed commits |

---

## 16. Handle merge conflicts

Conflicts happen when Git cannot safely combine overlapping changes. A typical conflict marker looks like:

```text
<<<<<<< HEAD
return "Hello"
=======
return "Hi"
>>>>>>> feature/add-greeting
```

The markers are **not** valid program code. Edit the file to the final intended version, removing the marker lines.

### 16.1 Merge conflict recovery sequence

```bash
git status
git diff --name-only --diff-filter=U

# Open/edit every conflicted file, remove markers, choose final content.
# Then test and stage:
python -m unittest discover -s tests -v
git add src/hello.py
git status

git merge --continue
```

**Explanation:** `git add` tells Git the conflict in the staged file is resolved. When all conflicts are resolved, `git merge --continue` completes the operation. If Git asks for a commit message, save and close the editor.

If you want to abandon the merge:

```bash
git merge --abort
```

### 16.2 Rebase conflict recovery sequence

```bash
git status
# Edit conflict markers and run tests.
git add src/hello.py
git rebase --continue
```

Repeat until all replayed commits apply. To return to the state before rebase:

```bash
git rebase --abort
```

**Important:** In a rebase, Git's “ours”/“theirs” labels may refer to different sides than you expect. Inspect actual content rather than blindly taking `--ours` or `--theirs`.

### 16.3 Inspect the three stages of a conflict

```bash
git show :1:src/hello.py  # merge base
git show :2:src/hello.py  # stage 2
git show :3:src/hello.py  # stage 3
```

Use these while the conflict is still unresolved. They are a useful way to reason about competing changes and avoid accidentally dropping one side.

---

## 17. Merge a pull request

After reviews and required CI checks pass:

```bash
gh pr view 12
gh pr checks 12
gh pr merge 12 --squash --delete-branch
```

**Explanation:** A squash merge integrates all PR changes as **one new commit** on `main`. `--delete-branch` requests feature-branch cleanup; exact local/remote deletion depends on current branch and permissions. If branch deletion fails, the merge may still have succeeded; inspect PR state and branch list.

Other merge strategies:

```bash
# Alternative: preserve individual commits and merge ancestry:
gh pr merge 12 --merge

# Alternative: GitHub rebase-merge:
gh pr merge 12 --rebase
```

Choose **one** merge strategy for a particular PR, not all three. Repository policy determines which strategies are allowed.

| Strategy | Results on main | Common reason to choose it |
|---|---|---|
| Squash (`--squash`) | One combined commit | Clean history; many review-fix commits |
| Merge commit (`--merge`) | Branch commits plus explicit merge commit | Preserve branch relationships |
| Rebase merge (`--rebase`) | Replay feature commits without merge commit | Linear sequence while keeping multiple commits |

**Caution:** Squash merging means Git may **not** consider your local original feature branch ancestrally merged, even though GitHub reports the PR merged. `git branch -d feature/...` can consequently refuse deletion. Verify the GitHub PR was merged, then use local `-D` **only** if you no longer need the original branch commits.

---

## 18. Update your local project and clean up branches

```bash
git switch main
git fetch --prune origin
git pull --ff-only origin main

git status -sb
git log --oneline --decorate --graph -10
git branch -a
```

**Explanation:** This gets the merged result into local `main`. `fetch --prune` deletes obsolete **remote-tracking references**, not actual local feature branches.

When you know your feature branch has been integrated:

```bash
# Safe if Git sees its commit ancestry as merged:
git branch -d feature/add-greeting
```

If the PR was **squash-merged** and Git refuses that deletion, verify merge completion first:

```bash
gh pr view 12

# ONLY after confirming integration and that the local branch is no longer needed:
git branch -D feature/add-greeting
```

To remove a remote feature branch that was not automatically deleted:

```bash
git push origin --delete feature/add-greeting
```

Remote deletions need permissions and may be restricted by branch protection. Avoid deleting another developer's in-progress branch.

---

## 19. Direct local merge, without GitHub PRs

Git does not require GitHub PRs. You can merge local branches, then push the resulting `main` branch where permissions allow:

```bash
git switch main
git pull --ff-only origin main

git merge --no-ff feature/add-greeting
python -m unittest discover -s tests -v

git push origin main
git branch -d feature/add-greeting
```

**Explanation:** `git merge feature/add-greeting` takes the **source branch** as input and merges **into the branch currently checked out** (`main` here). There is no normal `git merge source target` command that silently switches branches for you.

On a protected GitHub main branch, direct pushing may be forbidden. In that case use a PR, as above.

---

## 20. Releases and tags

Tag stable, tested versions so team members can reproduce particular snapshots.

```bash
git switch main
git pull --ff-only origin main
python -m unittest discover -s tests -v

git tag -a v1.0.0 -m "First stable release"
git push origin v1.0.0

git tag -l
git show v1.0.0
```

**Explanation:** An annotated tag stores metadata and a human-readable message; the tag typically stays tied to one commit permanently. Publishing a tag is separate from pushing a branch.

Create a GitHub release from it:

```bash
gh release create v1.0.0 --generate-notes
gh release view v1.0.0
```

This creates a hosted release page; it does not automatically deploy your application or build your binaries unless your project has automation for that.

To inspect a released snapshot without changing your feature/main branches:

```bash
git switch --detach v1.0.0
# Inspect, run tests, compare; avoid committing important work detached.
git switch main
```

---

## 21. Hotfixes and cherry-picks

A hotfix starts from the currently maintained base, fixes a specific urgent bug, gets reviewed, then merges quickly.

```bash
git switch main
git pull --ff-only origin main
git switch -c hotfix/fix-startup

# Edit and test.
git status
git add src/ tests/
git diff --cached
git commit -m "fix: handle missing startup config"
git push -u origin hotfix/fix-startup
gh pr create --base main --fill
```

If another release branch needs only a particular fix from main, use `cherry-pick` rather than merging every newer change:

```bash
git switch release/1.0
git cherry-pick -x FIX_COMMIT_HASH
```

**Explanation:** Cherry-pick reproduces the patch on another branch and creates a different commit hash; `-x` records its original hash in the message, useful for auditability. Resolve any cherry-pick conflict and test.

Abort a problematic cherry-pick:

```bash
git cherry-pick --abort
```

---

## 22. Pause work using a stash

Suppose you have half-finished changes but need to inspect a clean main branch. A stash temporarily saves a snapshot of working changes.

```bash
git status -sb
git stash push -u -m "WIP: greeting experiment"
git stash list

git switch main
# Do other work...

git switch feature/add-greeting
git stash show -p stash@{0}
git stash apply stash@{0}
```

**Explanation:** `-u` includes untracked **non-ignored** files. `apply` keeps the stash entry for safety. After confirming everything is restored as intended, you may remove that stash:

```bash
git stash drop stash@{0}
```

`git stash pop` is a shorthand for apply-then-drop (when successful), but it is not ideal as your only copy of important, long-running work. Avoid `git stash clear` unless you truly intend to remove all stash records.

**If several stashes exist:** inspect `git stash list` and select the correct `stash@{n}`. Stashes are local and may not transfer with clones or survive indefinite cleanup.

---

## 23. Recover from almost every Git mistake

**First:** inspect state and protect what is still good. Do not run a cleanup or reset merely to quiet `git status`.

```bash
git status -sb
git log --oneline --graph --decorate -12
git reflog -12
```

### Scenario A — You staged a file that should not be in the next commit

```bash
git restore --staged src/hello.py
```

**Result:** file changes remain on disk; they are simply no longer staged. This is the common and safe correction.

### Scenario B — You edited a tracked file and want to discard **only its unstaged changes**

```bash
git diff -- src/hello.py  # inspect what would be lost
git restore -- src/hello.py
```

**Result:** restores from the current index, discarding unstaged tracked changes to that path. An uncommitted edit may be unrecoverable. Staged content is not necessarily affected.

### Scenario C — You want a file's version from an older commit

```bash
git log --oneline -- src/hello.py
# Replace HASH with the exact old commit found above.
git restore --source=HASH -- src/hello.py

git diff
git add src/hello.py
git commit -m "fix: restore earlier greeting implementation"
```

**Result:** keeps branch history and creates a new regular commit capturing the chosen previous version. The `restore` command itself does **not** create a commit.

### Scenario D — You made the last commit locally but forgot to stage a file

```bash
git add tests/test_hello.py
git commit --amend --no-edit
```

**Result:** replaces the last commit with a corrected snapshot, giving it a new SHA. Use only before publishing or with explicit team coordination.

### Scenario E — You made the last local commit but want to uncommit it, keeping changes staged

```bash
git status -sb
git reset --soft HEAD~1
git status -sb
```

**Result:** moves the branch pointer back one commit; its file changes remain staged (subject to other changes and the starting index). Good for rebuilding local commits.

### Scenario F — You made a local commit but want the changes unstaged

```bash
git reset --mixed HEAD~1
```

**Result:** moves the current branch back, updates the index to the target, and retains the working-file changes. This is typically equivalent to `git reset HEAD~1`.

### Scenario G — A bad commit is already pushed to a shared branch

```bash
git switch main
git pull --ff-only origin main
git log --oneline -12

git switch -c revert/bad-change
git revert BAD_COMMIT_HASH
python -m unittest discover -s tests -v

git push -u origin revert/bad-change
gh pr create --base main --title "revert: undo regression" \
  --body "Reverts BAD_COMMIT_HASH because it introduced a regression."
```

**Result:** `git revert` makes a *new inverse commit* without rewriting the history that collaborators may have already pulled. For shared history, this is generally safer than reset + force-push. If later commits depend on that change, the revert may conflict or need additional adaptations.

### Scenario H — You need to revert an already merged PR

If the PR was **squash-merged**, find its one resulting commit on `main` and revert that commit as in Scenario G. If it was merged using a **merge commit**, reverting it requires choosing the proper mainline parent:

```bash
git show --summary MERGE_COMMIT_HASH
# Only if you have verified parent 1 is the correct mainline:
git revert -m 1 MERGE_COMMIT_HASH
```

**Important:** `-m 1` does not mean “revert merge #1”; it chooses **parent 1** as the mainline. Reverting a merge can influence later re-merges; inspect ancestry carefully, work on a revert branch and test. Some GitHub CLI versions also offer `gh pr revert PR_NUMBER` to open a revert PR.

### Scenario I — You ran `git reset --hard` and lost the tip of a previous **commit**

```bash
git reflog --date=local
# Identify the former commit SHA by looking at the sequence, then:
git show OLD_COMMIT_HASH
git branch rescue/lost-work OLD_COMMIT_HASH
```

**Result:** creates a recovery branch pointing to a previously recorded commit **if Git still has the object**. The reflog normally tracks local HEAD movements, including many resets and rebases, but may expire; it does **not** guarantee recovery of uncommitted edits.

If you want to switch to that recovered branch:

```bash
git switch rescue/lost-work
```

### Scenario J — You started a merge/rebase/cherry-pick and want to stop

```bash
# Choose ONE command matching the operation in git status:
git merge --abort
git rebase --abort
git cherry-pick --abort
```

**Result:** attempts to restore the state from before that operation. Do not run all three as a batch; choose the active operation.

### Scenario K — You deleted an untracked file with Git cleanup

```bash
# Always preview before any cleanup:
git clean -nd
# If you explicitly intend to remove the listed untracked files:
# git clean -fd
```

Untracked files deleted by `git clean` generally have **no Git history** to recover from. You may need filesystem backups. `git clean -fdx` additionally removes ignored files and can destroy local virtual environments, datasets, build outputs and checkpoints.

### Scenario L — You need your local checkout to match remote main *exactly*

This is highly destructive for local tracked edits and unpushed commits. **Do not run on a branch you want to keep as-is.** First make a safety reference and backup any uncommitted files.

```bash
git fetch origin
git status -sb
git log --oneline HEAD..origin/main
git log --oneline origin/main..HEAD

git branch rescue/before-hard-reset
# Only after carefully confirming the destructive operation:
# git reset --hard origin/main
```

**Result if uncommented:** resets the **current branch** to fetched `origin/main`, replaces index and tracked working tree content and removes uncommitted tracked edits. Some untracked files remain; inspect `git clean -nd` separately if you must clean those too. The rescue branch stores only committed history.

### Recovery summary

| What you want to undo | Usually choose | Rewrites published history? |
|---|---|---|
| Stage selection | `git restore --staged FILE` | No |
| Unstaged tracked edit | `git restore -- FILE` (destructive) | No |
| Last unpublished commit (keep staged edits) | `git reset --soft HEAD~1` | Only local branch moves |
| Last unpublished commit (keep unstaged edits) | `git reset HEAD~1` | Only local branch moves |
| Pushed commit | `git revert HASH` | **No** |
| Old file version | `git restore --source=HASH -- FILE` then commit | No |
| Accidental hard-reset commit tip | `git reflog` + `git branch rescue HASH` | No |
| In-progress merge | `git merge --abort` | No |
| In-progress rebase | `git rebase --abort` | Not by itself |
| Untracked cleanup | Prefer preview/backup first; Git may not recover it | N/A |

---

## 24. Inspect who changed what, when, and why

Use these in any phase, especially code review and incident investigation:

```bash
# Compact graph showing all local and fetched refs:
git log --oneline --graph --decorate --all -30

# Show latest change's file list and counts:
git show --stat HEAD

# Show exactly what a commit changed:
git show COMMIT_HASH

# List commits affecting a specific file:
git log --follow -- src/hello.py

# Attribute surviving source lines:
git blame src/hello.py

# Search history for when text was introduced/removed:
git log -S 'greet(' -- src/hello.py

# Search tracked source files:
git grep -n 'greet'
```

### Compare different things correctly

| Question | Command |
|---|---|
| What changed in my tracked files since the index? | `git diff` |
| What did I stage? | `git diff --cached` |
| Everything changed vs last commit, including staged edits? | `git diff HEAD` |
| What changed between two recorded commits? | `git diff OLD_HASH NEW_HASH` |
| What does feature add relative to its branch point from main? | `git diff main...feature/add-greeting` |
| Which commits are on feature but not main? | `git log main..feature/add-greeting --oneline` |
| What changed in a GitHub PR? | `gh pr diff 12` |
| Which remote changes haven't landed on local main? | `git fetch origin` then `git log main..origin/main --oneline` |

**Important:** `git diff` generally does not display content of completely untracked files until those files are added (or marked with intent-to-add). Inspect `git status` too.

### Find a regression with `git bisect`

```bash
git bisect start
git bisect bad HEAD
git bisect good v1.0.0

# Test each selected midpoint and mark as good or bad:
# git bisect good
# git bisect bad

# Exit at the end:
git bisect reset
```

**Explanation:** Git binary-searches the commit ancestry for the introduction of a reproducible failing behavior. This is particularly useful when your test suite was green last month and is failing now.

---

## 25. Back up code before a major refactor

Git is a powerful version control system, but **Git history by itself is not a backup of uncommitted files or ignored datasets**. For a serious refactor, use more than one protection layer.

### Layer A: save a branch checkpoint

```bash
git status -sb
# Make sure the current committed state is a good checkpoint.
git branch backup/pre-refactor
```

**Explanation:** Fast, very little extra disk space, recoverable with `git switch backup/pre-refactor` later. Saves **commits**, not unstaged/untracked edits.

### Layer B: make a portable Git history bundle

```bash
git bundle create ../my-project-before-refactor.bundle --all
git bundle verify ../my-project-before-refactor.bundle
```

**Explanation:** Backs up reachable Git objects/refs. You can recreate a separate clone later with:

```bash
git clone ../my-project-before-refactor.bundle ../restored-my-project
```

### Layer C: copy a selected working-code snapshot into a temporary directory

The example below is intended to run **from the root of your existing repository**, before making any changes. Excludes avoid copying large/binary/generated data. `rsync` is an external tool, not Git. Review the exclusion list for your own project; other large folders may need adding. Avoid storing private patient data in broad temporary paths.

```bash
# Run from repository root. Creates a uniquely named directory.
BACKUP_ROOT="${TMPDIR:-/tmp}/git-project-backups"
BACKUP_DIR="$BACKUP_ROOT/my-project-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"

rsync -a \
  --exclude='.git/' \
  --exclude='.env' \
  --exclude='.env.*' \
  --exclude='*.pem' \
  --exclude='*.key' \
  --exclude='.venv/' \
  --exclude='venv/' \
  --exclude='__pycache__/' \
  --exclude='node_modules/' \
  --exclude='data/' \
  --exclude='datasets/' \
  --exclude='results/' \
  --exclude='outputs/' \
  --exclude='checkpoints/' \
  --exclude='*.svs' \
  --exclude='*.tif' \
  --exclude='*.tiff' \
  --exclude='*.h5' \
  --exclude='*.pt' \
  --exclude='*.pth' \
  ./ "$BACKUP_DIR/"

printf 'Backup path: %s\n' "$BACKUP_DIR"
du -sh "$BACKUP_DIR"
```

**Explanation:** The snapshot captures currently present included files, **including working-file edits**, but does not preserve `.git` history. A temporary directory can be cleaned automatically by the OS; for durable backups, choose a trusted persistent location instead. The example excludes common secret-file patterns, but secrets stored under other names may still be copied; audit the backup contents and choose safe filesystem permissions.

Before overwriting your current files during a restore, compare both versions and make another backup if the current work might be useful:

```bash
# Review differences between saved snapshot and working repo:
diff -qr -x .git "$BACKUP_DIR" . || true

# Preview rsync restore. Do not use --delete casually:
rsync -an "$BACKUP_DIR/" ./

# Only after inspecting the preview:
# rsync -a "$BACKUP_DIR/" ./
```

Restoring this snapshot does not delete files created later unless you explicitly do additional cleanup. `rsync` won't automatically restore history because `.git/` was excluded; use the bundle/branch independently for committed history.

### What can I restore from each layer?

| Backup | Includes committed history | Includes uncommitted code | Includes excluded data | Typical size |
|---|---|---|---|---|
| Branch pointer | Yes, via existing Git objects | No | No | Tiny ref |
| Git bundle | Reachable committed Git refs/objects | No | No | Depends on Git history |
| Selective `rsync` working snapshot | No | Yes, if not excluded | No | Depends on included code |
| Full separate filesystem backup | Depends on what copied | Usually yes | Only if included | Potentially large |

**Project completion reporting:** show checkpoint branch name, bundle path/size, selective snapshot path/size, and which directories/patterns were excluded. This makes it much easier to decide which recovery option to use.

---

## 26. Collaboration: forks, teammates and upstream

### 26.1 Working on a repository you do not own

```bash
# GitHub CLI can fork and clone when permitted:
gh repo fork ORIGINAL_OWNER/PROJECT --clone
cd PROJECT

git remote -v
```

**Explanation:** Typically your fork is `origin`; the original project is `upstream`. Verify the exact names using `git remote -v`, because workflows/CLI versions may differ.

If the original project remote is missing:

```bash
git remote add upstream https://github.com/ORIGINAL_OWNER/PROJECT.git
```

Get changes from the original:

```bash
git switch main
git fetch upstream
git merge --ff-only upstream/main

git push origin main
```

This assumes your fork's main branch has not diverged; otherwise inspect history before integrating.

### 26.2 Publish your contribution

```bash
git switch -c feature/contribution
# Edit, test, stage, commit.
git push -u origin feature/contribution

gh pr create --base main --title "feat: contribution" --body "Summary and tests"
```

Check the created PR's **target repository**. A fork PR often targets the original upstream repository rather than your fork; the CLI may ask you to select it.

### 26.3 Shared branch etiquette

- Keep `main` stable; review work through PRs when repository policy requires it.
- Fetch before starting significant changes and before integrating your PR.
- Avoid rebasing or force-pushing any branch others are already building upon.
- Keep commits focused and descriptive; do not commit generated datasets or credentials.
- Review CI results and resolve reviewer comments before merging.
- Prefer `git revert` for already-published bad commits on shared branches.

---

## 27. Large files, research data, privacy and secrets

Code and documentation fit naturally in Git. Large Whole Slide Images, checkpoints, extensive caches and confidential clinical datasets frequently do **not**.

### 27.1 Check whether Git already tracks a large file

```bash
git ls-files | grep -E '\.(svs|tif|tiff|h5|hdf5|pt|pth)$' || true
```

If a generated path was staged but not committed:

```bash
git restore --staged data/large-slide.svs
```

If a file is **already tracked** and should stop being tracked:

```bash
git rm --cached path/to/file
printf '\npath/to/file\n' >> .gitignore

git add .gitignore
git commit -m "chore: stop tracking generated data"
```

**Important:** stopping tracking now does **not** remove the file from past Git commits. For data already shared, assess confidentiality and hosting exposure; scrub/rotate secrets using an audited, coordinated procedure if appropriate.

### 27.2 When Git LFS makes sense

If the repository is allowed to track particular large binary files and Git LFS is installed:

```bash
git lfs install
git lfs track '*.tif'
git add .gitattributes
git commit -m "chore: track TIFF assets with Git LFS"
```

LFS is a separate tool, not a solution to privacy policy or cloud hosting quotas. For protected health information, follow your institution's security and data-governance rules.

### 27.3 Protect secrets

Never commit personal access tokens, private SSH keys, API keys or production secrets. Use environment variables, secret managers or your organization's approved tooling. A `.gitignore` entry only prevents accidental **future** additions of untracked secrets; it cannot protect content that was previously committed or uploaded.

---

## 28. A practical Git workflow for each workday

### Morning: update your base and create a branch

```bash
git status -sb
git switch main
git fetch origin
git pull --ff-only origin main
git switch -c feature/your-change
```

### During development: inspect, test, commit and push

```bash
# Do your normal coding work.
git status -sb
git diff
git diff --check

# Run project-specific tests here.
git add -p
git diff --cached
git commit -m "feat: explain the change"
git push -u origin feature/your-change
```

### When ready: PR, review, revisions, merge

```bash
gh pr create --base main --fill
gh pr view --web
gh pr checks

# If reviewers request changes, edit/commit/push on the same branch:
# git add ...
# git commit -m "fix: address review feedback"
# git push

# Once approved and checks pass, substitute actual PR number:
# gh pr merge 12 --squash --delete-branch
```

### After merge: bring main up to date

```bash
git switch main
git fetch --prune origin
git pull --ff-only origin main
git status -sb
```

Do not execute this whole playbook blindly: if there are existing changes, a protected branch, a different default branch, a currently open PR or a failed check, handle that state first.

---

## 29. One-page command decision matrix

| Your immediate goal | Command | Notes |
|---|---|---|
| Start Git tracking | `git init -b main` | New local project only. |
| Clone hosted project | `git clone URL` | Existing repo. |
| See my branch and working state | `git status -sb` | Start here for safety. |
| Create feature branch | `git switch -c feature/name` | Based on current commit. |
| Change active branch | `git switch main` | Needs a non-conflicting worktree. |
| Inspect unstaged edits | `git diff` | Does not include content of normal untracked files. |
| Inspect staged edits | `git diff --cached` | Exact next commit patch. |
| Stage one file | `git add FILE` | Snapshot from working tree to index. |
| Stage parts of a file | `git add -p` | Interactive. |
| Remove staged changes, keep edits | `git restore --staged FILE` | Safe unstage. |
| Commit locally | `git commit -m "message"` | Does not publish. |
| Publish and track branch | `git push -u origin BRANCH` | Remote write. |
| Fetch current remote state | `git fetch origin` | No automatic integration. |
| Update main safely | `git pull --ff-only origin main` | Run while on main. |
| Create GitHub PR | `gh pr create --base main --fill` | Requires gh auth/remote. |
| Comment on PR | `gh pr comment 12 --body "text"` | General PR conversation. |
| Inspect PR changes | `gh pr diff 12` | Hosted change set. |
| Check review/CI state | `gh pr checks 12` | CI-specific. |
| Squash merge PR | `gh pr merge 12 --squash --delete-branch` | After approvals/tests. |
| Merge local branch | `git merge feature/name` | From the *destination* branch. |
| Put current edits aside | `git stash push -u -m "WIP"` | Temporary local stash. |
| Unstage but keep file | `git restore --staged FILE` | Does not erase work. |
| Discard unstaged edit | `git restore -- FILE` | May erase uncommitted work. |
| Undo published bad commit | `git revert HASH` | Adds inverse commit. |
| Remove unpublished commit, keep staged content | `git reset --soft HEAD~1` | Rewrites local branch pointer. |
| Recover old commit tip | `git reflog` | Then `git branch rescue HASH`. |
| See who changed a line | `git blame FILE` | Historical attribution. |
| Snapshot stable version | `git tag -a v1.0.0 -m "Release"` | Tag, then push separately. |
| Back up Git committed history | `git bundle create ../backup.bundle --all` | Doesn't include uncommitted working edits. |
| Preview deleting untracked files | `git clean -nd` | Never skip preview. |

---

## 30. Glossary and further reading

| Term | Meaning |
|---|---|
| **Repository** | Git's objects/refs and optionally a working directory containing project files. |
| **Working tree** | Actual checked-out files you edit. |
| **Index / staging area** | Candidate snapshot for next commit. |
| **Commit** | Immutable snapshot/metadata with parent(s), identified by a hash. |
| **HEAD** | Reference to current checkout, usually via current branch. |
| **Branch** | Movable label pointing to a commit. |
| **Remote** | Configured external repository, e.g. `origin`. |
| **Upstream** | Branch your local branch is configured to track. |
| **Fetch** | Update local view of remote data, without merging into checked-out branch. |
| **Pull** | Fetch plus configured merge/rebase integration. |
| **Push** | Publish accepted refs/commits to remote. |
| **PR / pull request** | GitHub-hosted code-review and merge proposal. |
| **Fast-forward** | Move branch pointer forward without creating a merge commit. |
| **Merge commit** | Commit with two or more parents, combining histories. |
| **Rebase** | Replay commits on a new base, creating new commit hashes. |
| **Revert** | Create an inverse commit without rewriting history. |
| **Reset** | Reposition HEAD/branch and possibly index/worktree depending on mode. |
| **Reflog** | Local log of recent reference movements; expires. |
| **Stash** | Local saved work-in-progress state. |
| **Tag** | Named reference often used to label a release commit. |

**Official docs:** [Git reference](https://git-scm.com/docs) · [Pro Git](https://git-scm.com/book/en/v2) · [GitHub CLI manual](https://cli.github.com/manual/) · [GitHub pull request guide](https://docs.github.com/en/pull-requests).

**Related table encyclopedia:** [command-reference/README.md](../command-reference/README.md), with a separate `git`/`gh` command glossary grouped by task. The grouped-table design was inspired by [joshnh/Git-Commands](https://github.com/joshnh/Git-Commands), without copying its content.

_Updated October 2026. Some commands are destructive and some options are Git/GitHub-CLI-version-dependent; inspect your own repository and use local `git help COMMAND` or `gh COMMAND --help` when uncertain._
