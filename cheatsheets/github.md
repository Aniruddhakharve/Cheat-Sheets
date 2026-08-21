# Git & GitHub Cheat Sheet
## 90 Days of DevOps — Day 22 to Day 26

A practical revision cheat sheet covering Git branching, merging, rebase, stash, cherry-pick, reset, reflog, revert, Git workflows, and GitHub CLI.

---

# Table of Contents

1. Day 22 — Branching, Merging & Remote Basics
2. Day 23 — Rebase & Stash
3. Day 24 — Cherry-Pick, Reset, Reflog & Revert
4. Day 25 — Git Workflows
5. Day 26 — GitHub CLI
6. Important Git Differences
7. Git Mental Models
8. Interview Quick Revision
9. Golden Rules
10. Final Revision Map

---

# Day 22 — Branching, Merging & Remote Basics

## 1. Git Branch

A branch is a movable pointer to a commit.

Example:

    A --- B --- C
              ^
              |
             main

### Create a branch

    git branch feature-login

### Switch branch

    git switch feature-login

### Create and switch

    git switch -c feature-login

### List branches

    git branch

### Show current branch

    git branch --show-current

### Delete branch

    git branch -d feature-login

### Force delete branch

    git branch -D feature-login

---

# 2. HEAD

HEAD tells Git where you are currently working.

Example:

    A --- B --- C
              ^
              |
             main
              ^
              |
             HEAD

If you switch branches:

    A --- B --- C
              ^
             main

    A --- B --- D
              ^
           feature
              ^
             HEAD

### Remember

HEAD = Where am I currently?

---

# 3. Git Status

    git status

Shows:

- Current branch
- Staged changes
- Unstaged changes
- Untracked files
- Merge/rebase/revert state

---

# 4. Git Add

    git add file.txt

Add everything:

    git add .

Moves changes:

    Working Directory
           |
        git add
           ↓
      Staging Area

---

# 5. Git Commit

    git commit -m "Add login feature"

Moves staged changes into the Git repository.

    Working Directory
           |
        git add
           ↓
      Staging Area
           |
       git commit
           ↓
     Local Repository

---

# 6. Git Log

Basic:

    git log

Compact:

    git log --oneline

Graph:

    git log --oneline --graph --decorate --all

Last 10 commits:

    git log --oneline --graph --decorate --all -10

Show a commit:

    git show <commit-id>

Example:

    git show 6f618cb

---

# 7. Merge

Merge combines changes from another branch into the current branch.

Example:

        D --- E
       /
    A --- B --- C
                 ^
                main

Run:

    git switch main
    git merge feature

---

# 8. Fast-Forward Merge

A fast-forward merge happens when the current branch has not diverged.

Before:

    A --- B
          ^
         main

    A --- B --- C --- D
                      ^
                    feature

Run:

    git switch main
    git merge feature

After:

    A --- B --- C --- D
                      ^
                     main

No new merge commit is created.

### Remember

Fast-forward = Git only moves the branch pointer forward.

---

# 9. Merge Commit

A merge commit happens when branches have diverged.

Example:

        D --- E
       /       \
    A --- B --- M
           \   /
            C

M is the merge commit.

A normal merge commit usually has two parents:

- Parent 1 → current branch history
- Parent 2 → branch being merged

Force a merge commit:

    git merge --no-ff feature

### Remember

Merge commit = combines two divergent histories.

---

# 10. Merge Conflict

A conflict happens when Git cannot automatically combine changes.

Example:

    <<<<<<< HEAD
    Environment: production
    =======
    Environment: development
    >>>>>>> feature

Resolve manually.

Then:

    git add file.txt
    git commit

General flow:

    git merge
        ↓
    Conflict
        ↓
    Edit file
        ↓
    Remove conflict markers
        ↓
    git add
        ↓
    git commit

---

# 11. Fetch vs Pull

## git fetch

    git fetch origin

Downloads remote changes but does not integrate them into your current branch.

Think:

    Remote Repository
           |
       git fetch
           ↓
       origin/main

Your local main does not automatically move.

---

## git pull

    git pull

Generally:

    git fetch
    +
    integrate changes

### Easy memory

    fetch = download

    pull = download + integrate

---

# Day 23 — Rebase & Stash

# 12. Rebase

Rebase moves your commits onto another base.

Before:

    A --- B --- C
           \
            D --- E

Run:

    git switch feature
    git rebase main

After:

    A --- B --- C --- D' --- E'

The feature commits are replayed on top of main.

The replayed commits get new IDs.

### Remember

Rebase = replay my commits on top of another base.

---

# 13. Rebase Conflict

Check:

    git status

Resolve the conflict.

Then:

    git add file.txt
    git rebase --continue

Skip:

    git rebase --skip

Abort:

    git rebase --abort

---

# 14. Merge vs Rebase

## Merge

        D --- E
       /       \
    A --- B --- C --- M

- Preserves branch history
- Does not rewrite existing commits
- Can create a merge commit

## Rebase

    A --- B --- C --- D' --- E'

- Creates a linear history
- Replays commits
- Changes commit IDs
- Rewrites history

### Important

Do not casually rebase commits that other people are already using.

---

# 15. Git Stash

Stash temporarily stores uncommitted work.

Example:

    You are working on feature A
              ↓
    Urgent task arrives
              ↓
    Need clean working tree
              ↓
          git stash

### Stash changes

    git stash

### Named stash

    git stash push -m "WIP dashboard"

### List stashes

    git stash list

Example:

    stash@{0}: On main: WIP dashboard
    stash@{1}: On feature-login: WIP login

### Apply stash

    git stash apply

Specific stash:

    git stash apply stash@{1}

### Pop stash

    git stash pop

### Drop stash

    git stash drop stash@{0}

### Delete all stashes

    git stash clear

---

# 16. Stash Apply vs Pop

    git stash apply
           ↓
    Restore changes
           ↓
    Keep stash

    git stash pop
           ↓
    Restore changes
           ↓
    Remove stash

### Remember

stash = temporarily put unfinished work aside

---

# Day 24 — Cherry-Pick, Reset, Reflog & Revert

# 17. Cherry-Pick

Cherry-pick applies one specific commit to the current branch.

Example:

    feature:

    A --- B --- C --- D
                      ^
                  wanted commit

    main:

    A --- B --- X

Run:

    git switch main
    git cherry-pick D

Result:

    A --- B --- X --- D'

The changes from D are applied to main.

D' gets a new commit ID.

### Remember

Cherry-pick = take one specific commit.

---

# 18. Cherry-Pick Conflict

Resolve conflict:

    git add file.txt
    git cherry-pick --continue

Skip:

    git cherry-pick --skip

Abort:

    git cherry-pick --abort

---

# 19. Git Reset

Reset moves the current branch pointer.

There are three important modes:

- --soft
- --mixed
- --hard

---

# 20. Reset --soft

    git reset --soft HEAD~1

Moves HEAD backward but keeps changes staged.

Example:

    Before:

    A --- B --- C
                ^
               HEAD

    After:

    A --- B
          ^
         HEAD

    Changes from C
          ↓
    Staging Area

Use when:

Undo the commit, but keep the changes staged.

---

# 21. Reset --mixed

    git reset HEAD~1

`--mixed` is the default.

HEAD moves backward.

Changes remain in the working directory but become unstaged.

Example:

    A --- B
          ^
         HEAD

    Changes from old commit
              ↓
       Working Directory

Use when:

Undo the commit, but keep the changes as unstaged modifications.

---

# 22. Reset --hard

    git reset --hard HEAD~1

HEAD moves backward.

The staging area and working directory are also reset.

Example:

    Before:

    A --- B --- C
                ^
               HEAD

    After:

    A --- B
          ^
         HEAD

WARNING:

`--hard` can remove uncommitted changes.

---

# 23. Reset Comparison

| Command | HEAD | Staging Area | Working Directory |
|---|---|---|---|
| `git reset --soft HEAD~1` | Moves | Changes staged | Changes remain |
| `git reset HEAD~1` | Moves | Changes unstaged | Changes remain |
| `git reset --hard HEAD~1` | Moves | Reset | Reset |

### Easy memory

    soft  → keep staged

    mixed → keep unstaged

    hard  → discard changes

---

# 24. Reflog

Reflog records local movements of Git references such as HEAD.

Run:

    git reflog

Example:

    cb8b58b HEAD@{0}: reset: moving to HEAD~1
    92cd083 HEAD@{1}: commit: Reset practice commit C
    cb8b58b HEAD@{2}: reset: moving to HEAD~1

Reflog is useful when you:

- Accidentally reset
- Rebase incorrectly
- Move a branch
- Lose track of a commit

---

# 25. Recover Using Reflog

Find the old commit:

    git reflog

Suppose:

    92cd083

Recover:

    git reset --hard 92cd083

### Remember

    git log
    → shows reachable commit history

    git reflog
    → shows local movements of HEAD/references

---

# 26. Revert

Revert creates a new commit that undoes an earlier commit.

Before:

    A --- B --- C

Run:

    git revert C

After:

    A --- B --- C --- C'

C' reverses the changes introduced by C.

### Remember

Revert = create a new commit that undoes an old commit.

---

# 27. Reset vs Revert

## Reset

    A --- B --- C

           ↓ reset

    A --- B

History is moved backward.

## Revert

    A --- B --- C --- C'

History stays intact.

A new commit undoes the old commit.

### Interview answer

Reset changes/moves history.

Revert preserves history and creates a new commit that reverses an earlier commit.

For shared branches:

Revert is generally safer than reset.

---

# 28. Revert Conflict

Resolve:

    git add file.txt
    git revert --continue

Skip:

    git revert --skip

Abort:

    git revert --abort

---

# Day 25 — Git Workflows

# 29. GitHub Flow

GitHub Flow is a simple workflow based on short-lived feature branches and Pull Requests.

Typical flow:

    main
      ↓
    Create feature branch
      ↓
    Make changes
      ↓
    Commit
      ↓
    Push
      ↓
    Pull Request
      ↓
    Code Review
      ↓
    CI/CD
      ↓
    Merge into main
      ↓
    Deploy

Example:

    git switch main
    git pull
    git switch -c feature-login

    # Make changes

    git add .
    git commit -m "Add login feature"
    git push -u origin feature-login

Then create a Pull Request.

### Advantages

- Simple
- Short-lived branches
- Easy code review
- Works well with CI/CD
- Good for continuous delivery
- Fast development

---

# 30. GitFlow

GitFlow uses multiple branches and a more structured release process.

Common branches:

    main
    develop
    feature/*
    release/*
    hotfix/*

Conceptually:

    feature/*
        ↓
    develop
        ↓
    release/*
        ↓
    main

### Characteristics

- More branches
- Longer-lived branches
- Structured releases
- Useful for scheduled releases
- Separate integration branch

---

# 31. Trunk-Based Development

Trunk-Based Development revolves around the main/trunk branch.

Developers use very short-lived branches or integrate directly into the trunk.

    main ────────────────────────────
          ↑       ↑       ↑
        commit  commit  commit

### Key idea

Keep branches short-lived and integrate frequently into main.

---

# 32. Workflow Comparison

| Workflow | Main Idea | Branch Lifetime | Typical Use |
|---|---|---|---|
| GitHub Flow | Feature branch + PR | Short | Startups / CI/CD |
| GitFlow | Structured branches | Longer | Scheduled releases |
| Trunk-Based | Main/trunk centered | Very short | Frequent integration |

### Easy memory

GitHub Flow
→ Simple + PR + short-lived branches

GitFlow
→ Many branches + structured releases

Trunk-Based
→ Main/trunk centered + frequent integration

---

# Day 26 — GitHub CLI

# 33. What is GitHub CLI?

GitHub CLI allows you to interact with GitHub directly from the terminal.

Command:

    gh

Check version:

    gh --version

Check authentication:

    gh auth status

---

# 34. Repository Commands

View repository:

    gh repo view owner/repository

Example:

    gh repo view Aniruddhakharve/devops-git-practice

View repository information:

    gh repo view Aniruddhakharve/devops-git-practice \
      --json name,owner,isPrivate,defaultBranchRef,url

List repositories:

    gh repo list Aniruddhakharve --limit 10

Open repository:

    gh repo view --web

---

# 35. GitHub Issues

Create issue:

    gh issue create \
      --repo owner/repository \
      --title "Practice GitHub CLI issue" \
      --body "Created this issue while revising Day 26." \
      --label "documentation"

List issues:

    gh issue list \
      --repo owner/repository

View issue:

    gh issue view 3 \
      --repo owner/repository

Close issue:

    gh issue close 3 \
      --repo owner/repository

List closed issues:

    gh issue list \
      --repo owner/repository \
      --state closed

---

# 36. GitHub Pull Requests

Create branch:

    git switch -c gh-cli-practice

Create file:

    echo "GitHub CLI PR practice" > gh-cli-practice.txt

Stage:

    git add gh-cli-practice.txt

Commit:

    git commit -m "Practice GitHub CLI pull request"

Push:

    git push -u origin gh-cli-practice

Create PR:

    gh pr create \
      --repo owner/repository \
      --base main \
      --head gh-cli-practice \
      --title "Practice GitHub CLI pull request" \
      --body "Created this pull request using GitHub CLI."

---

# 37. List Pull Requests

    gh pr list \
      --repo owner/repository

---

# 38. View Pull Request

    gh pr view 6 \
      --repo owner/repository

---

# 39. View PR Diff

    gh pr diff 6 \
      --repo owner/repository

This shows the changes introduced by the Pull Request.

---

# 40. Check PR Status

    gh pr checks 6 \
      --repo owner/repository

If no GitHub Actions workflows/checks exist:

    no checks reported

---

# 41. Merge Pull Request

Example:

    gh pr merge 6 --squash --delete-branch

This can:

    Pull Request
         ↓
    Squash commits
         ↓
    Merge into main
         ↓
    Delete branch

---

# 42. Local Main Can Diverge After PR Merge

Suppose GitHub merged a PR into remote main.

Remote:

    A --- B --- C --- D
                      ^
                 origin/main

Local:

    A --- B --- C
                ^
               main

Check:

    git status
    git branch -vv

You may see:

    Your branch and 'origin/main' have diverged

Fetch:

    git fetch origin

Check remote history:

    git log --oneline origin/main -5

If you intentionally want local main to exactly match remote:

    git reset --hard origin/main

WARNING:

This can discard local commits that are not present on `origin/main`.

---

# 43. GitHub Actions

List workflows:

    gh workflow list \
      --repo owner/repository

List workflow runs:

    gh run list \
      --repo owner/repository

View a run:

    gh run view <run-id> \
      --repo owner/repository

View failed logs:

    gh run view <run-id> \
      --repo owner/repository \
      --log-failed

Useful for:

- CI troubleshooting
- Build failures
- Deployment failures
- Inspecting GitHub Actions

---

# 44. GitHub API

`gh api` allows GitHub API requests from the terminal.

Example:

    gh api repos/owner/repository \
      --jq '.name,.default_branch'

Example:

    gh api repos/Aniruddhakharve/devops-git-practice \
      --jq '.name,.default_branch'

Output:

    devops-git-practice
    main

### --jq

Extracts specific fields from JSON output.

---

# 45. GitHub Search

Search repositories:

    gh search repos "kubernetes" --limit 5

Example results:

    kubernetes/kubernetes
    kubernetes/minikube
    helm/helm
    k3s-io/k3s
    kubernetes/community

Useful for discovering GitHub repositories and open-source projects.

---

# 46. GitHub Aliases

List aliases:

    gh alias list

Example:

    co: pr checkout
    prs: pr list

Aliases provide shortcuts for frequently used GitHub CLI commands.

---

# 47. GitHub Gists

Create a file:

    echo "GitHub CLI Gist practice" > /tmp/gh-gist-practice.txt

Create a gist:

    gh gist create \
      /tmp/gh-gist-practice.txt \
      --desc "GitHub CLI practice"

List gists:

    gh gist list

View a gist:

    gh gist view <gist-id>

Gists can be:

- Public
- Secret

A secret gist is not publicly listed, but anyone with its URL can access it.

---

# Important Git Differences

# 48. Fetch vs Pull

    git fetch
        ↓
    Download remote changes
        ↓
    Do not integrate automatically

    git pull
        ↓
    Fetch
    +
    Integrate

### Interview answer

`git fetch` downloads remote changes without integrating them into the current branch, while `git pull` fetches and then integrates those changes.

---

# 49. Merge vs Rebase

Merge:

- Combines histories
- Preserves existing commits
- May create a merge commit

Rebase:

- Replays commits
- Creates new commit IDs
- Produces linear history
- Rewrites history

---

# 50. Reset vs Revert

Reset:

- Moves branch/history
- Can rewrite history

Revert:

- Creates a new commit
- Undoes previous changes
- Preserves history

---

# 51. Reset Modes

    --soft
    → HEAD moves
    → Changes remain staged

    --mixed
    → HEAD moves
    → Changes become unstaged

    --hard
    → HEAD moves
    → Staging + working directory reset

---

# 52. Stash Apply vs Pop

    apply
    → Restore changes
    → Keep stash

    pop
    → Restore changes
    → Remove stash

---

# 53. Merge vs Cherry-Pick

Merge:

    Merge = branch-level operation

It combines another branch's history.

Cherry-pick:

    Cherry-pick = commit-level operation

It applies one specific commit.

---

# 54. Reset vs Reflog

Reset changes where the branch points:

    git reset --hard HEAD~1

Reflog records previous local positions:

    git reflog

Recovery:

    git reflog
    git reset --hard <old-commit>

---

# Git Mental Models

# 55. Git's Main Areas

                     GIT
                      |
                      v
            +-------------------+
            | Working Directory |
            +---------+---------+
                      |
                   git add
                      |
                      v
            +-------------------+
            |   Staging Area    |
            +---------+---------+
                      |
                  git commit
                      |
                      v
            +-------------------+
            | Local Repository  |
            +---------+---------+
                      |
                   git push
                      |
                      v
            +-------------------+
            | Remote Repository |
            |      GitHub       |
            +-------------------+

---

# 56. Complete Git Flow

    Create/modify file
           ↓
       git status
           ↓
        git add
           ↓
      Staging Area
           ↓
       git commit
           ↓
     Local Repository
           ↓
        git push
           ↓
         GitHub
           ↓
     Pull Request
           ↓
      Code Review
           ↓
         CI/CD
           ↓
         Merge

---

# 57. Git Branch Mental Model

                    main
                     |
                     v
A ---- B ---- C ---- D
       \
        E ---- F
             ^
             |
          feature

---

# 58. Merge Mental Model

Before:

        E ---- F
       /
A ---- B ---- C ---- D
                      ^
                     main

After:

        E ---- F
       /         \
A ---- B ---- C -- M
                  ^
                 main

M is the merge commit.

---

# 59. Rebase Mental Model

Before:

    A ---- B ---- C
           \
            D ---- E

After:

    A ---- B ---- C ---- D' ---- E'

---

# 60. Cherry-Pick Mental Model

feature:

    A ---- B ---- C ---- D
                          ^
                      wanted commit

main:

    A ---- B ---- X

After:

    A ---- B ---- X ---- D'

Only the changes from D were applied.

---

# 61. Reset Mental Model

Before:

    A ---- B ---- C
                 ^
                HEAD

After:

    A ---- B
          ^
         HEAD

---

# 62. Revert Mental Model

Before:

    A ---- B ---- C

After:

    A ---- B ---- C ---- C'

C' undoes the changes introduced by C.

---

# 63. Git Lifecycle of a Change

              Working Directory
                      |
                   git add
                      ↓
               Staging Area
                      |
                 git commit
                      ↓
               Local Repository
                      |
                  git push
                      ↓
               Remote Repository
                      |
               Pull Request
                      ↓
                   Review
                      ↓
                   Merge

---

# Interview Quick Revision

## What is Git?

Git is a distributed version control system used to track changes and collaborate on source code.

---

## What is a Git branch?

A branch is a movable pointer/reference to a commit.

---

## What is HEAD?

HEAD points to the currently checked-out branch or commit.

---

## What is a fast-forward merge?

A fast-forward merge occurs when the target branch has not diverged, so Git simply moves the branch pointer forward without creating a merge commit.

---

## What is a merge commit?

A merge commit combines two divergent histories and normally has two parents.

---

## What is rebase?

Rebase replays commits onto a new base and creates new commit IDs.

---

## What is cherry-pick?

Cherry-pick applies a specific commit from another branch to the current branch.

---

## What is stash?

Stash temporarily stores uncommitted changes so you can work with a clean working directory.

---

## What is reflog?

Reflog records local movements of Git references and can help recover commits after reset or rebase.

---

## What is reset?

Reset moves the current branch pointer and, depending on the mode, can also modify the staging area and working directory.

---

## What is revert?

Revert creates a new commit that reverses the changes introduced by an earlier commit.

---

## Why is revert safer than reset on shared branches?

Revert preserves existing history and creates a new commit instead of rewriting the shared branch history.

---

## What is GitHub Flow?

GitHub Flow is a simple workflow based on short-lived feature branches, Pull Requests, code review, CI/CD, and merging into main.

---

## What is GitFlow?

GitFlow is a structured branching model using branches such as main, develop, feature, release, and hotfix.

---

## What is Trunk-Based Development?

Trunk-Based Development keeps development centered around the main/trunk branch and encourages short-lived branches and frequent integration.

---

## What is GitHub CLI?

GitHub CLI is a command-line tool for interacting with GitHub directly from the terminal.

---

# Golden Rules

1. `git add`
   → Working Directory → Staging Area

2. `git commit`
   → Staging Area → Local Repository

3. `git push`
   → Local Repository → Remote Repository

4. `git fetch`
   → Download remote changes

5. `git pull`
   → Fetch + integrate

6. `git merge`
   → Combine branch histories

7. Fast-forward
   → Move branch pointer forward

8. Merge commit
   → Combine divergent histories
   → Normally has two parents

9. `git rebase`
   → Replay commits onto another base

10. `git stash`
    → Temporarily store unfinished work

11. `git cherry-pick`
    → Apply one specific commit

12. `git reset`
    → Move branch/history

13. `git reflog`
    → Find previous local HEAD/reference positions

14. `git revert`
    → Create an undo commit

15. GitHub Flow
    → Short-lived branch + PR + CI/CD

16. GitFlow
    → Structured branches + release process

17. Trunk-Based
    → Main/trunk centered + frequent integration

18. GitHub CLI
    → Manage GitHub from the terminal

---

# Day 22 → Day 26 Summary

    DAY 22
    ├── Git branches
    ├── HEAD
    ├── git log
    ├── Merge
    ├── Fast-forward merge
    ├── Merge commit
    ├── Merge conflicts
    └── Fetch vs Pull

    DAY 23
    ├── Rebase
    ├── Rebase conflicts
    ├── Stash
    ├── Stash apply
    └── Stash pop

    DAY 24
    ├── Cherry-pick
    ├── Reset
    │   ├── soft
    │   ├── mixed
    │   └── hard
    ├── Reflog
    └── Revert

    DAY 25
    ├── GitHub Flow
    ├── GitFlow
    └── Trunk-Based Development

    DAY 26
    ├── GitHub CLI
    ├── Repository commands
    ├── GitHub Issues
    ├── Pull Requests
    ├── PR diff
    ├── PR checks
    ├── PR merge
    ├── GitHub Actions
    ├── GitHub API
    ├── GitHub Search
    ├── GitHub Aliases
    └── GitHub Gists

---

# Final Revision Strategy

When revising Git, do not only memorize commands.

For every important command, ask:

    What does it do?
           ↓
    What happens to HEAD?
           ↓
    What happens to the branch?
           ↓
    What happens to the staging area?
           ↓
    What happens to the working directory?
           ↓
    Does it rewrite history?
           ↓
    Can I recover from it?

If you understand these questions, you understand Git rather than simply memorizing Git commands.

---

# One-Line Memory Map

    Branch
    → Create a separate line of work

    Merge
    → Combine branches

    Rebase
    → Replay commits onto a new base

    Stash
    → Temporarily save unfinished work

    Cherry-pick
    → Take one specific commit

    Reset
    → Move history/branch pointer

    Reflog
    → Find where HEAD used to be

    Revert
    → Undo with a new commit

    GitHub Flow
    → Short-lived branch + PR

    GitFlow
    → Structured multi-branch workflow

    Trunk-Based
    → Frequent integration into main

    GitHub CLI
    → Control GitHub from the terminal

---

# Day 22–26 Complete

    Git Basics
         ↓
    Branching
         ↓
    Merging
         ↓
    Rebase
         ↓
    Stash
         ↓
    Cherry-Pick
         ↓
    Reset
         ↓
    Reflog
         ↓
    Revert
         ↓
    Git Workflows
         ↓
    GitHub CLI
         ↓
    GitHub Issues + PRs + Actions + API

---

# 🎯 Final Goal

Git is not just a list of commands.

You should be able to visualize what Git is doing to:

- Commits
- Branches
- HEAD
- Working Directory
- Staging Area
- Local Repository
- Remote Repository

Once you understand those relationships, Git becomes much easier to remember and troubleshoot.
