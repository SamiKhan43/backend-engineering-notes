# Git and GitHub Explained: Everything a Backend Engineer Needs

A beginner-to-engineer reference for version control and collaboration: how Git works, the daily workflow, branching, merging and rebasing, undoing mistakes, remotes, GitHub (issues, pull requests, code review, branch protection), CI/CD with GitHub Actions, security (secrets, signed commits, Dependabot), team workflows, releases, and a troubleshooting table for the problems everyone hits.

---

## Quick summary

- **Git** is a **distributed version control system**: it records snapshots (commits) of your project so you can go back, compare, branch, and collaborate.
- **GitHub** is a **hosting platform** around Git: remote repositories, pull requests, issues, code review, CI/CD (Actions), and security tooling. (GitLab and Bitbucket are alternatives.)
- Daily loop: **branch → edit → `add` → `commit` → `push` → pull request → review → merge**.
- Commit **small, focused changes** with **clear messages**. Never commit **secrets**.
- **Merge** keeps history as it happened. **Rebase** rewrites it to be linear. **Never rebase shared branches.**
- Almost everything is **recoverable** in Git (`reflog` is your safety net), as long as it was committed.
- Protect `main` with **branch protection**, **required reviews**, and **CI checks**.

---

## Table of contents

1. [What are Git and GitHub](#1-what-are-git-and-github)
2. [Installation and first-time setup](#2-installation-and-first-time-setup)
3. [Core concepts](#3-core-concepts)
4. [Everyday workflow](#4-everyday-workflow)
5. [Good commits and commit messages](#5-good-commits-and-commit-messages)
6. [.gitignore and files you must never commit](#6-gitignore-and-files-you-must-never-commit)
7. [Branching and merging](#7-branching-and-merging)
8. [Rebase and rewriting history](#8-rebase-and-rewriting-history)
9. [Undoing things](#9-undoing-things)
10. [Remotes: fetch, pull and push](#10-remotes-fetch-pull-and-push)
11. [Branching strategies](#11-branching-strategies)
12. [GitHub essentials](#12-github-essentials)
13. [Pull requests and code review](#13-pull-requests-and-code-review)
14. [GitHub Actions and CI/CD](#14-github-actions-and-cicd)
15. [Security on Git and GitHub](#15-security-on-git-and-github)
16. [Useful advanced commands](#16-useful-advanced-commands)
17. [How Git works inside](#17-how-git-works-inside)
18. [Team workflow, releases and versioning](#18-team-workflow-releases-and-versioning)
19. [Productivity setup](#19-productivity-setup)
20. [Common problems and fixes](#20-common-problems-and-fixes)
21. [Learning roadmap](#21-learning-roadmap)
22. [Checklist: things to remember](#22-checklist-things-to-remember)
23. [Common pitfalls and interview questions](#23-common-pitfalls-and-interview-questions)

Related guides: [Introduction to Backend Development](backend-development-intro.md), [APIs](api-guide.md), [Databases](database-guide.md), [Authentication and Authorization](authentication-guide.md), [HTTP and HTTPS](http-https-guide.md).

---

## 1. What are Git and GitHub

**Version control** records the history of your files so you can:

- See **what changed, when, and who changed it** (and why).
- **Go back** to any earlier version.
- Work on **features in parallel** (branches) and combine them.
- **Collaborate** without overwriting each other's work.
- Keep a **backup** of your code on a remote server.

**Git vs GitHub**

| | Git | GitHub |
|---|---|---|
| What | A tool installed on your computer | A website/service that hosts Git repositories |
| Works offline? | Yes (almost everything) | No |
| Provides | Commits, branches, merges, history | Remote repos, pull requests, issues, code review, Actions, security scanning |
| Alternatives | (Mercurial, SVN, older) | GitLab, Bitbucket, Azure DevOps, Gitea |

**Distributed** means every clone contains the **entire history**. You can commit, branch, and inspect history with no network. A remote (like GitHub) is just another copy that everyone agrees to sync with.

---

## 2. Installation and first-time setup

**Install**

- Windows: Git for Windows (includes Git Bash). macOS: `brew install git` (or Xcode tools). Linux: `sudo apt install git`.
- Check: `git --version`.

**One-time configuration**

```bash
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"      # match your GitHub email
git config --global init.defaultBranch main
git config --global core.editor "code --wait"          # or nano, vim
git config --global pull.rebase false                  # choose a pull strategy (see section 10)
git config --global push.autoSetupRemote true          # first push sets upstream automatically
git config --global core.autocrlf input                # Mac/Linux; use "true" on Windows

git config --list --show-origin                        # see all settings
```

Config levels: `--system` (all users), `--global` (you), and `--local` (this repo only, the default inside a repo).

**Authenticate with GitHub (SSH is the common choice)**

```bash
ssh-keygen -t ed25519 -C "you@example.com"      # press Enter for defaults, set a passphrase
cat ~/.ssh/id_ed25519.pub                        # copy the PUBLIC key

# GitHub: Settings → SSH and GPG keys → New SSH key → paste
ssh -T git@github.com                            # test: "Hi <user>! You've successfully authenticated"
```

Alternatives: **HTTPS + a credential helper or personal access token**, or the **GitHub CLI** (`gh auth login`). Never share your **private** key (the file without `.pub`).

---

## 3. Core concepts

**The three areas (and the remote)**

```mermaid
flowchart LR
  A["Working directory (your files)"] -->|"git add"| B["Staging area (index)"]
  B -->|"git commit"| C["Local repository (.git)"]
  C -->|"git push"| D["Remote (GitHub)"]
  D -->|"git fetch"| C
  C -->|"git switch / restore"| A
```

| Term | Meaning |
|---|---|
| **Repository (repo)** | A project folder plus its full history (stored in `.git/`) |
| **Working directory** | The files you see and edit |
| **Staging area (index)** | The "shopping cart" of changes that will go into the next commit |
| **Commit** | A **snapshot** of the staged files, with a message, author, time, and parent commit(s) |
| **Hash (SHA)** | The unique ID of a commit, e.g. `a3f9c2d` |
| **Branch** | A lightweight **movable pointer** to a commit |
| **HEAD** | Pointer to "where you are now" (usually the current branch's latest commit) |
| **Remote** | A copy of the repo elsewhere (default name: `origin`) |
| **Tag** | A fixed label on a commit (e.g. a release `v1.2.0`) |
| **Merge / Rebase** | Two ways to combine work from different branches |
| **Clone / Fork** | Clone = copy a repo to your machine. Fork = your own server-side copy of someone else's repo |

**Commits form a graph.** Each commit points to its parent, and a branch is just a name pointing at one commit.

```mermaid
gitGraph
  commit id: "A"
  commit id: "B"
  branch feature
  checkout feature
  commit id: "C"
  commit id: "D"
  checkout main
  commit id: "E"
  merge feature id: "M"
```

**File states**: *untracked* (new, Git ignores it), *modified* (changed since last commit), *staged* (will be in the next commit), *committed* (safe in history).

---

## 4. Everyday workflow

**Start a repository**

```bash
git init                                # new repo in the current folder
git clone git@github.com:user/repo.git  # copy an existing repo (with all history)
git clone --depth 1 <url>               # shallow clone: latest snapshot only (fast, e.g. in CI)
```

**The core loop**

```bash
git status                       # what changed? (use it constantly)
git diff                         # unstaged changes
git add file.py                  # stage one file
git add -p                       # stage piece by piece (interactive, great for clean commits)
git add .                        # stage everything (check git status first!)
git diff --staged                # what will go into the commit
git commit -m "Add rate limiting to login endpoint"
git log --oneline --graph --decorate -10    # compact history
git push                         # send commits to the remote
git pull                         # get and integrate remote changes
```

**Looking around**

```bash
git log                          # full history
git log -p -- path/to/file       # history of one file with diffs
git show a3f9c2d                 # one commit in detail
git blame file.py                # who last changed each line
git diff main...feature          # changes on feature since it branched from main
git branch -vv                   # local branches and their upstreams
```

**Tips**

- Run `git status` **before** `git add .` and `git commit`. It prevents committing junk.
- `git commit -am "msg"` stages and commits **tracked** files only (new files are not included).
- Commit **often** locally (small commits). You can tidy them before sharing.
- Pull **before** you start working and before you push, to reduce conflicts.

---

## 5. Good commits and commit messages

**What makes a good commit**

- **One logical change** per commit (atomic): a bug fix, or a refactor, or a feature step. Not "misc changes".
- It should **build and pass tests** on its own, so `git bisect` and reverts work.
- Do not mix **formatting changes** with **behavior changes**.
- Do not commit commented-out code, debug prints, or large generated files.

**Commit message format**

```
Short summary in imperative mood (50 chars or fewer)

Longer explanation if needed, wrapped at about 72 characters. Explain
WHAT changed and especially WHY, not how (the diff shows how).

Closes #123
```

- **Imperative mood**: "Add", "Fix", "Remove" (it completes "If applied, this commit will ...").
- Bad: `fix`, `update`, `changes`, `wip`, `asdf`. Good: `Fix N+1 query in order listing`.
- Reference issues: `Closes #123`, `Fixes #123`, `Refs #99`. GitHub links them and auto-closes the issue on merge to the default branch.

**Conventional Commits** (a popular convention, helps changelogs and automated versioning)

```
feat: add refresh token rotation
fix: return 404 instead of 500 for unknown task id
docs: explain idempotency keys in API guide
refactor: extract repository layer from router
test: add ownership check tests
chore: bump fastapi to 0.115
feat!: remove v1 login endpoint        # "!" or a BREAKING CHANGE footer marks a breaking change
```

---

## 6. .gitignore and files you must never commit

A **`.gitignore`** file lists patterns of files Git should not track.

```gitignore
# Python
__pycache__/
*.pyc
.venv/
venv/
.pytest_cache/
.ruff_cache/
dist/
build/
*.egg-info/

# Node
node_modules/

# Secrets and local config
.env
.env.*
!.env.example          # an example file WITHOUT real values is fine to commit
*.pem
*.key
secrets/

# Editors and OS
.vscode/
.idea/
.DS_Store
Thumbs.db

# Local data
*.sqlite3
*.log
dump.sql
```

**Never commit**

- **Secrets**: passwords, API keys, tokens, private keys, `.env` files, cloud credentials.
- **Dependencies** (`node_modules`, virtual environments). Commit `requirements.txt` / `package-lock.json` instead.
- **Build output and caches.**
- **Large binaries**, database dumps, or personal data (use Git LFS or object storage if you truly need versions of big files).
- Editor and OS junk.

**Rules and tricks**

- Create `.gitignore` **before** the first commit. Patterns only affect **untracked** files.
- If a file is already tracked, ignoring it is not enough:

```bash
git rm --cached .env             # stop tracking, but keep the file on disk
echo ".env" >> .gitignore
git commit -m "Stop tracking .env"
```

- Commit a `.env.example` listing variable **names** (no secrets) so teammates know what to set.
- Check why something is ignored: `git check-ignore -v path/to/file`.
- **If you committed a secret**: treat it as **leaked**. **Rotate/revoke it first**, then (optionally) remove it from history (section 15). Deleting it in a new commit does **not** remove it from history.

---

## 7. Branching and merging

**Branches** let you work on something without disturbing `main`. They are cheap (just a pointer), so use them for everything.

```bash
git branch                           # list local branches
git switch -c feature/rate-limiting  # create and switch (older: git checkout -b)
git switch main                      # switch back
git branch -d feature/rate-limiting  # delete (safe: refuses if unmerged)
git branch -D feature/old            # force delete
git push origin --delete feature/old # delete on the remote
git fetch --prune                    # forget remote branches that were deleted
```

**Naming**: `feature/short-description`, `fix/login-timeout`, `chore/update-deps`, or `username/topic`. Include a ticket id if you use one (`feature/PROJ-123-rate-limiting`).

### Merging

```bash
git switch main
git pull
git merge feature/rate-limiting
```

Two outcomes:

| Type | When | Result |
|---|---|---|
| **Fast-forward** | `main` has no new commits since the branch started | Pointer just moves forward, linear history |
| **Merge commit** | Both branches have new commits | A new commit with two parents ties them together |

Force a merge commit with `git merge --no-ff`. Cancel a merge in progress with `git merge --abort`.

### Merge conflicts

A **conflict** happens when two branches change the **same lines** (or one deletes a file the other edits). Git stops and marks the files:

```
<<<<<<< HEAD
timeout = 30
=======
timeout = 60
>>>>>>> feature/timeouts
```

**Resolve step by step**

1. `git status` shows "both modified" files.
2. Open each file, **decide the correct final content**, and delete the markers (`<<<<<<<`, `=======`, `>>>>>>>`). Often it is a combination, not just one side.
3. Run the **tests**.
4. `git add <file>` then `git commit` (for a merge) or `git rebase --continue` (for a rebase).
5. Changed your mind? `git merge --abort` / `git rebase --abort` returns you to before.

Tips: conflicts are normal, not a failure. **Small, short-lived branches** and **frequent syncing with `main`** keep them rare. Use an editor's merge tool (VS Code, `git mergetool`) if helpful. Enable `git config --global rerere.enabled true` so Git remembers how you resolved a repeated conflict.

---

## 8. Rebase and rewriting history

**Rebase** replays your branch's commits **on top of** another branch, producing a **linear** history.

```mermaid
gitGraph
  commit id: "A"
  commit id: "B"
  branch feature
  checkout feature
  commit id: "C"
  checkout main
  commit id: "D"
  checkout feature
  commit id: "C-prime (rebased)"
```

```bash
git switch feature/x
git fetch origin
git rebase origin/main        # replay my commits on top of the latest main
# resolve conflicts if any, git add ..., git rebase --continue
git push --force-with-lease   # needed because the branch history changed
```

**Merge vs rebase**

| | Merge | Rebase |
|---|---|---|
| History | Preserved exactly as it happened (branches visible) | Rewritten into a straight line |
| Safe on shared branches? | **Yes** | **No**, it changes commit hashes |
| Conflicts | Resolved once | May be resolved per commit |
| Good for | Integrating into `main`, shared branches | Cleaning up **your own** unpushed or private branch before a PR |

> **Golden rule: never rebase (or force-push) commits that other people have based work on**, such as `main`.

**Always use `--force-with-lease`, not `--force`.** It refuses to overwrite remote commits you have not seen.

### Interactive rebase: clean up before a pull request

```bash
git rebase -i HEAD~4          # edit the last 4 commits
```

In the editor, change the word before each commit:

| Command | Effect |
|---|---|
| `pick` | Keep the commit |
| `reword` | Keep, edit the message |
| `squash` | Merge into the previous commit, combine messages |
| `fixup` | Merge into the previous commit, discard this message |
| `drop` | Delete the commit |
| (reorder lines) | Reorder commits |

Related helpers:

```bash
git commit --amend                     # modify the LAST commit (message and/or add staged changes)
git commit --amend --no-edit
git commit --fixup a3f9c2d             # mark a commit as a fix for an earlier one
git rebase -i --autosquash main        # automatically folds fixup commits into place
```

**Squash merge** (a GitHub option) does this at merge time: all PR commits become one commit on `main`.

---

## 9. Undoing things

First, find the situation, then pick the tool.

| Situation | Command |
|---|---|
| Discard changes in a file (unstaged) | `git restore file.py` |
| Unstage a file (keep the changes) | `git restore --staged file.py` |
| Fix the last commit message / add a forgotten file | `git commit --amend` (only if **not pushed**) |
| Undo the last commit, keep changes **staged** | `git reset --soft HEAD~1` |
| Undo the last commit, keep changes **unstaged** | `git reset HEAD~1` (same as `--mixed`) |
| Undo the last commit and **throw changes away** | `git reset --hard HEAD~1` (dangerous) |
| Undo a commit that is **already pushed/shared** | `git revert a3f9c2d` (adds a new "undo" commit, safe) |
| Undo a merge commit | `git revert -m 1 <merge-sha>` |
| Save work-in-progress without committing | `git stash` |
| Remove untracked files | `git clean -n` (preview), then `git clean -fd` |
| Recover "lost" commits | `git reflog` then `git reset --hard <sha>` or `git switch -c rescue <sha>` |
| Get one file from another commit/branch | `git restore --source=main -- path/file.py` |

**`reset` vs `revert`**

- **`reset`** moves the branch pointer back (rewrites history). Use it for **local, unpushed** work.
- **`revert`** creates a **new commit** that reverses an older one. Use it for **anything shared**.

**Reset modes**

| Mode | Moves branch | Staging area | Working files |
|---|---|---|---|
| `--soft` | Yes | Kept | Kept |
| `--mixed` (default) | Yes | Reset | Kept |
| `--hard` | Yes | Reset | **Overwritten (lost)** |

**Stash**

```bash
git stash push -m "half-done pagination"   # shelve tracked changes (add -u to include untracked)
git stash list
git stash pop                              # apply the latest and remove it
git stash apply stash@{1}                  # apply without removing
git stash drop stash@{0}
```

**The safety net: `git reflog`**

```bash
git reflog              # a log of everywhere HEAD has been, including "deleted" commits
git reset --hard HEAD@{3}
```

Committed work is almost always recoverable for weeks. **Uncommitted** changes thrown away with `--hard` or `clean` are **not**. Commit or stash first when unsure.

---

## 10. Remotes: fetch, pull and push

```bash
git remote -v                                  # list remotes
git remote add origin git@github.com:user/repo.git
git push -u origin main                        # push and set the upstream (tracking) branch
git push                                       # later pushes need no arguments
git fetch                                      # download remote changes, do NOT change your files
git pull                                       # = fetch + integrate (merge or rebase)
git push origin v1.2.0                         # push a tag (or --tags for all)
```

**Fetch vs pull**

- **`git fetch`**: safe. Updates `origin/main` (your copy of the remote branch) only. Inspect with `git log main..origin/main` or `git diff main origin/main`.
- **`git pull`**: fetch **plus** merge/rebase into your current branch.
- Prefer `git pull --rebase` (or `git config --global pull.rebase true`) for a tidy linear history on your own feature branches. `pull.ff only` is the strictest and safest, since it refuses anything that is not a fast-forward.

**Rejected pushes**

```
! [rejected] main -> main (non-fast-forward)
```

Someone pushed before you. Run `git pull --rebase` (or `git pull`), fix conflicts, then push again. **Do not force push** to fix this.

**Forks and `upstream`** (open source / contributing to repos you do not own)

```bash
git clone git@github.com:YOU/project.git        # your fork
git remote add upstream git@github.com:ORIGINAL/project.git
git fetch upstream
git switch main && git merge upstream/main      # sync your fork's main
```

---

## 11. Branching strategies

| Strategy | How it works | Best for |
|---|---|---|
| **GitHub Flow** | `main` is always deployable. Short-lived feature branches → pull request → review + CI → merge → deploy | Most web/backend teams, continuous delivery |
| **Trunk-based development** | Everyone commits to `main` (or very short branches, under a day), unfinished work hidden behind **feature flags** | High-performing teams with strong CI and testing |
| **Git Flow** | Long-lived `develop` and `main`, plus `feature/*`, `release/*`, `hotfix/*` branches | Versioned/packaged software with scheduled releases (heavier) |
| **Release branches** | `main` + branches like `release/2.4` for maintenance | Supporting multiple released versions |

**Recommendation for most backend work: GitHub Flow or trunk-based.**

```mermaid
flowchart LR
  A["Branch from main"] --> B["Commit small changes"]
  B --> C["Open pull request"]
  C --> D["Review and CI checks"]
  D --> E["Merge to main"]
  E --> F["Deploy"]
  F --> A
```

**Principles**

- **Keep branches short-lived** (days, not weeks). Long branches mean painful merges.
- Keep `main` **always releasable**. Protect it.
- Use **feature flags** to merge unfinished features safely.
- Make **small PRs** (see next sections).
- Database migrations and API changes must be **backward compatible** so you can deploy and roll back safely (see the Databases and APIs guides).

---

## 12. GitHub essentials

**Repository basics**

- **README.md**: what the project is, how to run, test, and deploy it. The most important file.
- **LICENSE**: tells others what they may do (MIT, Apache-2.0, GPL...). No license = all rights reserved.
- **`.gitignore`**, **CONTRIBUTING.md**, **CODE_OF_CONDUCT.md**, **SECURITY.md** (how to report vulnerabilities).
- **Visibility**: public or private. Organizations own team repos with roles (read, triage, write, maintain, admin).
- **Default branch**: usually `main`.

**Features you will use**

| Feature | Purpose |
|---|---|
| **Issues** | Track bugs, tasks, and ideas. Labels, assignees, milestones |
| **Pull requests (PRs)** | Propose changes, discuss, review, run CI, then merge |
| **Projects** | Kanban/board planning on top of issues and PRs |
| **Actions** | CI/CD automation (section 14) |
| **Releases and tags** | Package and announce versions |
| **Discussions** | Q&A and community conversations |
| **Wiki / Pages** | Docs and static websites |
| **Security tab** | Dependabot, secret scanning, code scanning, advisories |
| **Codespaces** | Cloud development environments |
| **Packages / GHCR** | Host Docker images and packages |

**Useful files in `.github/`**

```
.github/
├── workflows/ci.yml                 # GitHub Actions workflows
├── pull_request_template.md         # default PR description
├── ISSUE_TEMPLATE/bug_report.md     # issue templates
├── CODEOWNERS                       # who must review which paths
└── dependabot.yml                   # automatic dependency update PRs
```

**CODEOWNERS example**

```
# Everything needs one of these teams by default
*                   @my-org/backend
# Database migrations need a DBA review
/migrations/        @my-org/dba
# Auth code needs the security team
/app/auth/          @my-org/security
```

**Branch protection / rulesets (set these on `main`)**

- Require a **pull request** before merging, with **at least 1 approval** (and re-approval after new pushes).
- Require **status checks** (tests, lint) to pass, and branches to be up to date.
- Require **code owner** review where relevant.
- **Block force pushes** and deletion of `main`.
- Require **linear history** or **signed commits** if your team wants it.
- Restrict who can push or bypass.

**Merge methods for PRs**

| Method | Result | Good when |
|---|---|---|
| **Merge commit** | All commits kept + a merge commit | You want full history |
| **Squash and merge** | All PR commits become one commit | PRs have messy WIP commits, clean `main` history |
| **Rebase and merge** | Commits replayed linearly, no merge commit | You want linear history and clean individual commits |

**GitHub CLI (`gh`)**

```bash
gh auth login
gh repo create my-app --private --source=. --push
gh pr create --fill                 # open a PR from the current branch
gh pr checkout 123                  # check out someone's PR locally
gh pr list
gh pr merge --squash --delete-branch
gh issue create --title "Login returns 500" --label bug
gh run list                         # see CI runs
gh run watch
```

**Linking work**: writing `Closes #123` in a PR description or commit message auto-closes that issue when it merges into the default branch. Mention people with `@username`.

---

## 13. Pull requests and code review

### Writing a good PR

- **Small and focused**: one purpose, ideally under about 400 changed lines. Large PRs get shallow reviews and hide bugs.
- **Clear title** (what) and **description** (why, how, how it was tested).
- Link the issue (`Closes #123`), add screenshots or example requests/responses for API changes.
- Mark **breaking changes**, migrations, and deploy-order requirements.
- Open a **draft PR** early for feedback on work in progress.
- **Self-review** your diff first. You will catch typos, debug prints, and leftover TODOs.
- Make sure **CI is green** before requesting review.

**PR template**

```markdown
## What
Short description of the change.

## Why
Link to the issue / the problem being solved. Closes #

## How to test
1. Steps or commands
2. Expected result

## Checklist
- [ ] Tests added or updated
- [ ] Docs / API spec updated
- [ ] Migration is backward compatible
- [ ] No secrets or debug code
```

### Reviewing code (backend checklist)

| Area | Questions to ask |
|---|---|
| **Correctness** | Does it do what the issue asks? Edge cases, empty/null inputs, error paths? |
| **Tests** | Are there tests for new behavior and failure cases? Do they fail without the change? |
| **Security** | Input validation? **Authorization/ownership check** on every object? SQL injection, secrets, logging of sensitive data? |
| **Data and migrations** | Backward compatible? Locks big tables? Indexes for new queries? Rollback plan? |
| **API contract** | Breaking change for clients? Status codes, error format, versioning? |
| **Performance** | N+1 queries, unbounded results, missing pagination, slow loops, missing timeouts? |
| **Reliability** | Retries, idempotency, timeouts, transactions, race conditions? |
| **Readability** | Clear names, small functions, no duplication, comments explain *why*? |
| **Observability** | Useful logs and metrics, no sensitive data in logs? |

### Review etiquette

- Review the **code, not the person**. Be kind and specific ("Consider X because Y").
- Separate **must-fix** from **nits/suggestions** (prefix with `nit:` or `suggestion:`).
- Ask questions instead of assuming ("What happens if `user` is None here?").
- Approve when it is **good enough and safe**, not perfect. Do not block on style the linter should enforce.
- Respond to every comment (fix it, or explain why not), and re-request review after changes.
- Review quickly. Waiting PRs block teammates.
- Automate the boring parts (formatting, linting, type checks) in CI so reviews focus on design and logic.

---

## 14. GitHub Actions and CI/CD

**CI** (Continuous Integration): on every push/PR, automatically **build, lint, and test**. **CD** (Continuous Delivery/Deployment): automatically **deliver** passing builds to an environment.

**GitHub Actions concepts**

| Term | Meaning |
|---|---|
| **Workflow** | A YAML file in `.github/workflows/` |
| **Event / trigger** | `push`, `pull_request`, `schedule` (cron), `workflow_dispatch` (manual), `release` |
| **Job** | A group of steps running on one **runner** (VM) |
| **Step** | A shell command or a reusable **action** (`uses:`) |
| **Runner** | The machine (`ubuntu-latest`, self-hosted) |
| **Secrets** | Encrypted values (`${{ secrets.NAME }}`), set in repo/org/environment settings |
| **Environment** | Named target (staging, production) with its own secrets and optional approvals |

**A CI workflow for a Python backend with PostgreSQL**

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read            # least privilege for the built-in token

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true  # cancel older runs of the same branch

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        ports: ["5432:5432"]
        options: >-
          --health-cmd "pg_isready -U test"
          --health-interval 5s
          --health-timeout 5s
          --health-retries 10
    env:
      DATABASE_URL: postgresql+psycopg://test:test@localhost:5432/testdb
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip
      - run: pip install -r requirements.txt
      - run: ruff check .          # lint
      - run: pytest -q             # tests
```

(Action versions move on. Check each action's page for the latest major version.)

**A deploy job that runs only after tests pass on `main`**

```yaml
  deploy:
    needs: test                               # wait for the test job
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production                   # can require manual approval
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}   # never hardcode secrets
        run: ./scripts/deploy.sh
```

**Typical CI/CD pipeline stages**

```mermaid
flowchart LR
  A["Push or PR"] --> B["Lint and format check"]
  B --> C["Unit and integration tests"]
  C --> D["Build Docker image"]
  D --> E["Security scan"]
  E --> F["Push image to registry"]
  F --> G["Deploy to staging"]
  G --> H["Approval"]
  H --> I["Deploy to production"]
```

**Best practices**

- Keep CI **fast** (cache dependencies, run tests in parallel, split slow suites).
- Make CI **required** on PRs through branch protection.
- Set **`permissions:`** to the minimum, and keep secrets in **GitHub Secrets**, never in the YAML.
- **Pin third-party actions** to a full commit SHA for supply-chain safety (or at least a trusted major version), and review what you use.
- Secrets are **not** available to workflows from forks' PRs by default. That is intentional.
- Use **environments** with required reviewers for production.
- Use a **matrix** to test multiple Python versions: `strategy: matrix: python-version: ["3.11", "3.12"]`.
- Make deployments **repeatable and reversible** (versioned images, rollback script), with database migrations that are backward compatible.
- Add a **`schedule:`** job for nightly dependency or security scans.

---

## 15. Security on Git and GitHub

**Protect your account**

- Enable **two-factor authentication** (a hardware key or authenticator app, or passkeys).
- Use **SSH keys with passphrases**. Remove old keys you no longer use.
- Prefer **fine-grained personal access tokens** with minimal scope and an **expiry date**. Treat tokens like passwords.
- Use SSO and least-privilege roles in organizations.

**Keep secrets out of Git**

- Use `.gitignore`, `.env.example`, and a **secrets manager** or **GitHub Actions secrets** for real values.
- Add a **pre-commit hook** or scanner (`gitleaks`, `detect-secrets`) to block secrets locally.
- Enable GitHub **secret scanning** and **push protection** (blocks pushes containing known secret formats).

**If a secret was committed or pushed**

1. **Revoke/rotate the secret immediately.** Assume it is compromised. This is the only step that really fixes the problem.
2. Remove it from history if needed, using `git filter-repo` (or BFG Repo-Cleaner), then force-push, and ask collaborators to re-clone or reset:

```bash
git filter-repo --path .env --invert-paths      # remove a file from ALL history
```

3. Check logs for misuse of that credential.
4. Add the file to `.gitignore` and add a scanner so it does not happen again.

Removing a secret from history does not help if someone already copied it, or if forks and clones exist. That is why **rotation comes first**.

**Dependencies and code scanning**

- **Dependabot**: opens PRs for vulnerable or outdated dependencies, and sends alerts.

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

- **Code scanning (CodeQL)** finds vulnerabilities in your code, and **dependency review** flags risky new dependencies in PRs.
- Commit **lock files** for reproducible builds.
- Review third-party **Actions** and packages before using them (typosquatting and supply-chain attacks are real).

**Signed commits** (prove that a commit really came from you)

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
# Add the same public key on GitHub as a "Signing key"; commits show a "Verified" badge
```

**Repo settings to enable**: branch protection on `main`, required reviews and checks, restricted force pushes, `SECURITY.md`, private vulnerability reporting, and limited admin access.

---

## 16. Useful advanced commands

```bash
# Cherry-pick: copy one commit onto the current branch (e.g. backport a hotfix)
git cherry-pick a3f9c2d

# Bisect: binary-search history for the commit that introduced a bug
git bisect start
git bisect bad                     # current version is broken
git bisect good v1.4.0             # this older version worked
# Git checks out middle commits; test each, then mark: git bisect good / git bisect bad
git bisect run pytest tests/test_login.py     # or automate it
git bisect reset

# Search history
git log -S "rate_limit" --oneline              # commits that added/removed this text
git log --author="Ana" --since="2 weeks ago"
git log --follow -p -- app/auth.py             # file history across renames
git grep "TODO"                                # search tracked files

# Tags and releases
git tag -a v1.2.0 -m "Release 1.2.0"
git push origin v1.2.0

# Worktrees: check out another branch in a separate folder, no stashing
git worktree add ../hotfix-dir hotfix/login

# Compare and inspect
git diff --stat main...feature
git show --stat HEAD
git shortlog -sn                               # commits per author

# Partial restore and interactive tools
git restore -p file.py                         # discard changes piece by piece
git add -p                                     # stage piece by piece
```

**Git hooks and the `pre-commit` framework**

Hooks run scripts at events (before commit, before push). The hooks in `.git/hooks` are **not shared** through the repo, so use the **pre-commit** tool, which reads a committed config:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.9            # pin a version; update with `pre-commit autoupdate`
    hooks:
      - id: ruff
      - id: ruff-format
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.4
    hooks:
      - id: gitleaks
```

```bash
pip install pre-commit
pre-commit install          # run the hooks on every commit
```

(Always run the same checks in CI too, because local hooks can be skipped.)

**Other things to know**

- **Git LFS**: stores large binaries (models, media) outside normal history. Prefer object storage for big data, and avoid committing large files at all.
- **Submodules**: embed another repo inside yours. Powerful but error-prone, so avoid them unless you have a clear need.
- **Monorepo vs polyrepo**: one repo for many services (shared tooling, atomic changes, needs good CI) vs one repo per service (clear boundaries, more coordination).
- **`git commit --amend`, `rebase -i`, `reset`** rewrite history. Use them only on **unshared** commits.

---

## 17. How Git works inside

Understanding the model removes the "magic".

- Git is a **content-addressable key-value store**: every object is stored under the **SHA hash of its content**. Same content = same hash; any change = a different hash. This is also what makes history tamper-evident.
- Four object types:

| Object | Holds |
|---|---|
| **Blob** | The contents of one file |
| **Tree** | A directory listing (names → blobs and other trees) |
| **Commit** | Pointer to a tree (the snapshot), parent commit(s), author, message |
| **Tag** (annotated) | A named pointer to a commit with a message |

- A commit stores a **snapshot**, not a diff. (Diffs are calculated on demand, and storage is compressed into **packfiles**.)
- **Branches and tags are just files** containing a commit hash (`.git/refs/heads/main`). That is why creating a branch is instant.
- **HEAD** (`.git/HEAD`) says which branch (or commit) you have checked out. A **detached HEAD** means HEAD points straight at a commit, not a branch (commits made there are easy to lose, so create a branch: `git switch -c rescue`).
- **`origin/main`** is a **remote-tracking branch**: your local record of where `main` was on the remote at the last fetch.
- **Rewriting history** (rebase, amend) creates **new commits** with new hashes and moves the branch pointer. The old commits still exist for a while (see `reflog`) until garbage collected.
- The `.git` folder **is** the repository. Delete it and the history is gone, so back up (push!) accordingly.

---

## 18. Team workflow, releases and versioning

**A typical day on a team (GitHub Flow)**

```bash
git switch main && git pull                 # start from the latest main
git switch -c feature/PROJ-42-cursor-pagination
# ... code, test, commit in small steps ...
git fetch origin && git rebase origin/main  # stay up to date (own branch only)
git push -u origin feature/PROJ-42-cursor-pagination
gh pr create --fill                         # open PR, request reviewers
# address review comments, push more commits
# CI green + approvals → squash and merge → branch auto-deleted
git switch main && git pull && git branch -d feature/PROJ-42-cursor-pagination
```

**Hotfix flow**: branch from `main` (or from the release tag), fix, PR with expedited review, merge, deploy, then make sure the fix also reaches any release branches (`cherry-pick`).

**Rolling back a bad deploy**

- Fastest: redeploy the **previous version** (image tag or release).
- Then `git revert <bad-commit>` (or the merge commit with `-m 1`) via a PR so `main` matches reality.
- Never rewrite or force-push `main` to "remove" a bad commit.

**Semantic Versioning (SemVer)**: `MAJOR.MINOR.PATCH`, e.g. `2.4.1`.

| Part | Bump when |
|---|---|
| **MAJOR** | Breaking changes (API incompatible) |
| **MINOR** | New backward-compatible features |
| **PATCH** | Backward-compatible bug fixes |

**Releases**

1. Merge to `main`, make sure CI is green.
2. Tag: `git tag -a v1.3.0 -m "Release 1.3.0"` and `git push origin v1.3.0`.
3. Create a **GitHub Release** with notes (`gh release create v1.3.0 --generate-notes`).
4. A workflow triggered by the tag builds and deploys (or publishes the package/image).
5. Keep a **CHANGELOG.md** (the "Keep a Changelog" format) listing added, changed, fixed, removed, and security items. Conventional Commits can generate it.

**Collaboration habits**

- Sync with `main` **often**. Small PRs merge easily.
- Communicate before big refactors or file moves, to avoid conflicts.
- Do not commit to `main` directly on team projects. Use PRs.
- Write PR descriptions for the reviewer who has no context.
- Delete merged branches (enable "automatically delete head branches").

---

## 19. Productivity setup

```bash
# Handy aliases
git config --global alias.st "status -sb"
git config --global alias.co "checkout"
git config --global alias.sw "switch"
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.last "log -1 --stat"
git config --global alias.undo "reset --soft HEAD~1"

# Quality-of-life settings
git config --global rerere.enabled true          # remember conflict resolutions
git config --global fetch.prune true             # auto-clean deleted remote branches
git config --global diff.colorMoved zebra        # show moved lines distinctly
git config --global help.autocorrect 10          # auto-run the corrected command after 1 second
git config --global merge.conflictstyle zdiff3   # shows the base version in conflict markers
```

**Other tips**

- Use a good **terminal prompt** that shows the current branch (e.g. starship, oh-my-zsh git plugin).
- Learn your editor's **Git integration** (VS Code Source Control, JetBrains), and a GUI such as GitHub Desktop, GitKraken, or `lazygit` for visual history.
- Use `git add -p` and `git diff --staged` before every commit.
- Keep a **global gitignore** for editor/OS files: `git config --global core.excludesfile ~/.gitignore_global`.
- Learn to read `git log --graph`. It makes branches click.

---

## 20. Common problems and fixes

| Problem | Fix |
|---|---|
| **"I committed to the wrong branch"** (not pushed) | `git switch -c correct-branch` (keeps the commit), then on the wrong branch `git reset --hard origin/<branch>` |
| **"Undo my last commit but keep the code"** | `git reset --soft HEAD~1` |
| **"I pushed a bad commit"** | `git revert <sha>`, push. Do not force-push shared branches |
| **"Push rejected (non-fast-forward)"** | `git pull --rebase`, resolve conflicts, push again |
| **Merge conflict panic** | `git status`, fix markers, `git add`, continue. Or `git merge --abort` / `git rebase --abort` to start over |
| **Detached HEAD** | `git switch -c my-work` to keep commits there, or `git switch main` to leave |
| **"I lost commits"** | `git reflog`, find the hash, `git switch -c rescue <hash>` |
| **Wrong commit message (unpushed)** | `git commit --amend` |
| **Committed a secret** | **Rotate it first**, then `git filter-repo`/BFG to scrub history, force-push |
| **Committed a huge file** | Remove with `git filter-repo --strip-blobs-bigger-than 50M`, use LFS or object storage |
| **File still tracked after adding to `.gitignore`** | `git rm --cached <file>`, commit |
| **Need to temporarily switch tasks** | `git stash push -m "wip"`, switch, later `git stash pop` |
| **Changes disappear after `reset --hard`/`clean`** | Uncommitted work is not recoverable. Commit or stash first next time |
| **`Permission denied (publickey)`** | Check `ssh -T git@github.com`, make sure the key is added to GitHub and the agent (`ssh-add ~/.ssh/id_ed25519`) |
| **Line-ending warnings (CRLF/LF)** | Set `core.autocrlf`, or add a `.gitattributes` with `* text=auto` |
| **Case-only file rename not detected (Windows/macOS)** | `git mv Foo.py foo.py` |
| **Pull overwrote or conflicts with local changes** | Commit or stash before pulling |
| **PR shows unrelated commits** | Branch was cut from the wrong base. Rebase onto the right branch: `git rebase --onto main <old-base> feature` |
| **Repo is slow or huge** | Look for big files in history, use shallow clones (`--depth 1`), `git gc`, LFS |
| **"fatal: refusing to merge unrelated histories"** | The two repos have no common ancestor. Use `--allow-unrelated-histories` only if intended |

---

## 21. Learning roadmap

1. **Basics**: `init`, `clone`, `status`, `add`, `commit`, `log`, `diff`, `push`, `pull`.
2. **Branching and merging**: create branches, merge, **resolve a conflict on purpose** in a practice repo.
3. **`.gitignore`, good commit messages**, and keeping secrets out.
4. **GitHub workflow**: fork, branch, PR, review, merge. Contribute a small fix to an open-source project.
5. **Undoing**: `restore`, `reset`, `revert`, `stash`, **`reflog`**. Break things in a sandbox and recover them.
6. **Rebase and interactive rebase**, `--force-with-lease`.
7. **GitHub Actions**: run tests on every PR, then add a deploy job.
8. **Branch protection, CODEOWNERS, Dependabot, secret scanning.**
9. **Advanced tools**: `bisect`, `cherry-pick`, `worktree`, hooks, signed commits.
10. **Release management**: tags, SemVer, changelogs, GitHub Releases.

**Practice exercises**

1. Create a repo, make 5 commits, view them with `git log --graph`.
2. Make two branches that edit the same line, merge, resolve the conflict.
3. Make 4 messy commits, then squash them into 1 with `rebase -i`.
4. Commit a fake secret, then fix it properly (rotate, scrub, add a hook).
5. Delete a branch, then recover it with `reflog`.
6. Set up a CI workflow that runs `pytest` and block merging when it fails.
7. Use `git bisect run` to find a deliberately introduced bug.
8. Tag a release and generate release notes.

Good resources: the free **Pro Git** book (git-scm.com), `git help <command>`, and the interactive site "Learn Git Branching".

---

## 22. Checklist: things to remember

1. **Git** = local version control. **GitHub** = hosting and collaboration around it.
2. Three areas: **working directory → staging → repository**, then **push** to the remote.
3. Run **`git status`** often, and check **`git diff --staged`** before committing.
4. Make **small, atomic commits** with **clear imperative messages** that explain *why*.
5. Create **`.gitignore`** first. **Never commit secrets**, `.env`, dependencies, or big binaries.
6. Use **branches for everything**, keep them **short-lived**, and **never commit directly to `main`** on a team.
7. **Merge** preserves history. **Rebase** tidies **your own** branch. **Never rebase shared history.**
8. Use **`--force-with-lease`**, never plain `--force`, and never on `main`.
9. Shared mistake? **`git revert`**. Local mistake? **`reset`/`amend`**. Lost something? **`git reflog`**.
10. **`git stash`** to park unfinished work.
11. **Pull/fetch before you push**, and resolve conflicts carefully, then run tests.
12. **Pull requests**: small, well described, self-reviewed, CI green, reviewed kindly and thoroughly.
13. Protect `main`: **required reviews, required checks, no force pushes.**
14. **CI** on every PR (lint + tests). **CD** from `main`, with approvals for production.
15. Least-privilege **Actions permissions**, secrets in **GitHub Secrets**, pinned third-party actions.
16. Turn on **2FA**, **secret scanning**, **push protection**, **Dependabot**.
17. Leaked secret = **rotate first**, scrub history second.
18. Tag releases with **SemVer**, keep a **changelog**, and roll back by **redeploying** the previous version.

---

## 23. Common pitfalls and interview questions

**Pitfalls**

- Committing `.env` files, API keys, or private keys (and thinking deleting the file fixes it).
- Huge "do everything" commits and PRs, or meaningless messages like `fix` and `wip`.
- Force-pushing to shared branches, or rebasing commits others already use.
- Long-lived branches that drift far from `main` and cause giant conflicts.
- Using `git add .` blindly and committing debug files or generated output.
- Using `reset --hard` or `clean -fd` on uncommitted work without checking.
- Resolving conflicts by blindly accepting one side without running tests.
- No branch protection, no required CI, merging your own unreviewed PRs.
- Putting secrets in GitHub Actions YAML instead of Secrets, or using unpinned third-party actions.
- Committing `node_modules`, virtual environments, database dumps, or large binaries.
- Deploying from a developer's laptop instead of CI.
- Mixing formatting changes and logic changes in one commit.
- Rewriting history after pushing, then wondering why teammates have conflicts.

**Interview questions**

- **What is Git, and how is it different from GitHub?**
  Git is a distributed version control tool that runs locally. GitHub is a hosting service built around Git that adds pull requests, issues, code review, CI, and security features.
- **What is the staging area?**
  A buffer between your working directory and the repository where you choose exactly which changes go into the next commit.
- **`git fetch` vs `git pull`?**
  Fetch downloads remote changes without touching your files. Pull is fetch plus merge (or rebase) into your current branch.
- **`git merge` vs `git rebase`?**
  Merge joins histories with a merge commit and preserves what happened. Rebase replays your commits on a new base for a linear history, but rewrites commits, so never do it on shared branches.
- **`git reset` vs `git revert`?**
  Reset moves the branch pointer back (rewrites history, for local work). Revert adds a new commit that undoes an earlier one (safe for shared history).
- **Explain `reset --soft`, `--mixed`, and `--hard`.**
  Soft keeps changes staged, mixed (default) keeps them unstaged, hard discards them from the working directory too.
- **How do you resolve a merge conflict?**
  Open the conflicted files, choose or combine the right content, remove the markers, run tests, `git add`, then commit (or `rebase --continue`).
- **How do you undo a pushed commit?**
  `git revert <sha>` and push. Avoid rewriting shared history.
- **What does `git stash` do?**
  Temporarily shelves uncommitted changes so you can switch tasks, then restores them with `stash pop`.
- **What is `git reflog` and when is it useful?**
  A local log of where HEAD has been. It lets you recover "lost" commits after a bad reset, rebase, or deleted branch.
- **What is a detached HEAD?**
  HEAD points directly at a commit instead of a branch. New commits there can be lost unless you create a branch.
- **What is cherry-pick and when do you use it?**
  Applying a single commit from another branch onto the current one, for example to backport a hotfix.
- **Squash merge vs rebase merge vs merge commit?**
  Squash combines all PR commits into one. Rebase-merge replays commits linearly. A merge commit keeps all commits plus a merge node.
- **How do you find which commit introduced a bug?**
  `git bisect` (optionally `bisect run` with a test) binary-searches history to find the first bad commit.
- **How would you handle a secret committed to a repo?**
  Revoke and rotate it immediately, remove it from history with `git filter-repo` or BFG and force-push, check for misuse, then add `.gitignore` entries and secret scanning to prevent a repeat.
- **What is a good branching strategy for a backend team?**
  GitHub Flow or trunk-based: short-lived branches, PRs with review and CI, `main` always deployable, feature flags for unfinished work.
- **What is CI/CD and how do you set it up with GitHub Actions?**
  CI automatically lints and tests every change, and CD deploys passing builds. A YAML workflow in `.github/workflows/` triggered by push/PR runs jobs on runners, using secrets and environments for deployment.
- **How do you protect the `main` branch?**
  Branch protection or rulesets: required PR reviews, required status checks, CODEOWNERS, no force pushes or deletions, and optionally signed commits and linear history.
- **What is semantic versioning?**
  `MAJOR.MINOR.PATCH`: major for breaking changes, minor for backward-compatible features, patch for backward-compatible fixes.
- **How do you keep a long-running feature branch in sync?**
  Regularly rebase it on (or merge in) `main`, resolve conflicts early, and prefer breaking the work into smaller PRs behind a feature flag.
- **Why are commits in Git cheap to branch and merge?**
  Commits are immutable snapshots identified by hashes, and branches are just small pointers to a commit, so creating a branch only writes a tiny file.
