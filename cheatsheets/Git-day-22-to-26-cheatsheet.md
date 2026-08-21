# Git & GitHub Master Cheat Sheet
## 90 Days of DevOps — Day 22 to Day 26

> Complete revision guide for everything covered and practiced from Day 22 to Day 26.
>
> This sheet is designed for **daily revision, hands-on practice, and Git/GitHub interview preparation**.

---

# Table of Contents

- [Day 22 — Git Branching, Merging & Remote Operations](#day-22--git-branching-merging--remote-operations)
- [Day 23 — Git Rebase & Stash](#day-23--git-rebase--stash)
- [Day 24 — Cherry-Pick, Reset, Reflog & Revert](#day-24--cherry-pick-reset-reflog--revert)
- [Day 25 — Git Workflows](#day-25--git-workflows)
- [Day 26 — GitHub CLI](#day-26--github-cli)
- [Git Command Comparison](#git-command-comparison)
- [Git Mental Models](#git-mental-models)
- [Common Conflict Recovery Commands](#common-conflict-recovery-commands)
- [Interview Questions & Answers](#interview-questions--answers)
- [Daily Git Practice Checklist](#daily-git-practice-checklist)
- [Final Git Memory Map](#final-git-memory-map)

---

# Day 22 — Git Branching, Merging & Remote Operations

---

# 1. Git Branch

## What is a branch?

A branch is a movable pointer/reference to a commit.

Branches allow developers to work on different features without directly modifying the main branch.

Example:

    A --- B --- C
              ^
              |
             main

Create a feature branch:

    git branch feature-login

Switch to it:

    git switch feature-login

Now:

    A --- B --- C
              ^
              |
         feature-login

---

## Commands

### List branches

    git branch

Example:

    git branch

Output:

    feature-login
    * main

`*` indicates the current branch.

---

### Create a branch

    git branch feature-login

---

### Switch branch

    git switch feature-login

---

### Create and switch

    git switch -c feature-login

This is the command you will commonly use in real projects.

---

### Show current branch

    git branch --show-current

Example:

    git branch --show-current

Output:

    feature-login

---

### Delete a merged branch

    git branch -d feature-login

---

### Force delete a branch

    git branch -D feature-login

Use `-D` carefully because Git will delete the branch even if it has unmerged commits.

---

## Remember

    branch = separate line of development

---

# 2. HEAD

## What is HEAD?

`HEAD` tells Git where you are currently checked out.

Example:

    A --- B --- C
              ^
              |
             main
              ^
              |
             HEAD

If you switch to another branch:

    A --- B --- C
              ^
             main

    A --- B --- D
              ^
           feature
              ^
             HEAD

---

## Check HEAD/current branch

    git branch --show-current

    git status

    git log --oneline --decorate -1

---

## Example

    git switch feature-login

Now:

    git branch --show-current

Output:

    feature-login

---

## Remember

    HEAD = Where am I currently?

---

# 3. Git Status

## What does it do?

Shows the current state of your working directory and staging area.

Command:

    git status

Example:

    On branch feature-login

    Changes not staged for commit:
      modified: app.py

---

## It tells you about

- Current branch
- Untracked files
- Modified files
- Staged files
- Merge conflicts
- Rebase state
- Revert state
- Cherry-pick state

---

## Example

    git status

If everything is clean:

    On branch main
    nothing to commit, working tree clean

---

# 4. Git Add

## What does it do?

Moves changes from the working directory into the staging area.

Command:

    git add file.txt

Example:

    echo "Login feature" > login.txt
    git add login.txt

Flow:

    Working Directory
           |
        git add
           ↓
      Staging Area

---

## Add multiple files

    git add file1.txt file2.txt

---

## Add everything

    git add .

---

## Check staged changes

    git diff --staged

---

## Remember

    git add = prepare changes for commit

---

# 5. Git Commit

## What does it do?

Creates a new commit from staged changes.

Command:

    git commit -m "Add login feature"

Flow:

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

## Example

    echo "Login feature" > login.txt
    git add login.txt
    git commit -m "Add login feature"

---

## View commit

    git log --oneline -1

---

# 6. Git Diff

## Working directory changes

    git diff

Shows changes that are not staged.

---

## Staged changes

    git diff --staged

Shows changes that are already staged.

Example:

    git add app.py
    git diff --staged

---

## Compare two branches

    git diff main..feature-login

---

## Compare using three-dot notation

    git diff main...feature-login

Useful for seeing the changes introduced by the feature branch relative to the common ancestor.

---

# 7. Git Log

## Normal log

    git log

---

## Compact log

    git log --oneline

Example:

    6e6c3b0 Hotfix change 1
    7eedbc3 Add dashboard part 2
    8b5ec61 Add dashboard part 1
    5d67fa5 Add main branch update

---

## Last 5 commits

    git log --oneline -5

---

## Graph

    git log --oneline --graph --decorate --all

Example:

    * 6e6c3b0 (HEAD -> feature) Hotfix
    * 7eedbc3 Dashboard part 2
    * 8b5ec61 Dashboard part 1
    * 5d67fa5 (main) Main update

---

## Show all branches

    git log --oneline --decorate --graph --all

---

## Show a specific commit

    git show <commit-id>

Example:

    git show 6e6c3b0

---

# 8. Git Merge

## What is merge?

Merge combines the history of another branch into the current branch.

Example:

    main:

    A --- B --- C

    feature:

    A --- B --- C --- D --- E

Merge:

    git switch main
    git merge feature

Result:

    A --- B --- C --- D --- E
                          ^
                         main

If there is no divergence, this can be a fast-forward merge.

---

## Important

You merge the branch **into the branch you are currently on**.

Correct:

    git switch main
    git merge feature-login

This means:

    feature-login → main

---

# 9. Fast-Forward Merge

## What is it?

A fast-forward merge occurs when the target branch has no new commits since the feature branch was created.

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

No merge commit is created.

Git simply moves the `main` pointer forward.

---

## Remember

    Fast-forward merge
    =
    move branch pointer forward

---

# 10. Merge Commit

## What is a merge commit?

A merge commit is created when two branches have diverged and Git combines both histories.

Before:

            D --- E
           /       \
    A --- B         ?
           \
            C --- F
             ^
            main

After:

            D --- E
           /       \
    A --- B --- C --- M
                     ^
                    main

`M` is the merge commit.

A merge commit normally has two parents.

---

## Force a merge commit

Even if fast-forward is possible:

    git merge --no-ff feature-login

Example:

    git switch main
    git merge --no-ff feature-login -m "Merge login feature"

---

## Why use `--no-ff`?

It preserves the fact that a feature branch existed.

Without `--no-ff`:

    A --- B --- C --- D

With `--no-ff`:

            C --- D
           /       \
    A --- B -------- M

---

## Remember

    Merge commit
    =
    combines divergent histories
    =
    normally has two parents

---

# 11. Merge Conflict

## What is a conflict?

A conflict occurs when Git cannot automatically determine which changes should be kept.

Example:

    <<<<<<< HEAD
    Environment: production
    =======
    Environment: development
    >>>>>>> feature

---

## Conflict markers

    <<<<<<< HEAD
    Current branch changes
    =======
    Incoming branch changes
    >>>>>>> feature

---

## Resolve conflict

Edit the file and remove the conflict markers.

Then:

    git add file.txt
    git commit

---

## Check conflicts

    git status

---

## Example flow

    git merge feature

    CONFLICT (content): Merge conflict in app.txt

Then:

    cat app.txt

Edit the file.

Then:

    git add app.txt
    git commit -m "Resolve merge conflict"

---

# 12. Remote Repository

A remote repository is a Git repository hosted somewhere such as GitHub.

Check remotes:

    git remote -v

Example:

    origin  git@github.com:username/project.git (fetch)
    origin  git@github.com:username/project.git (push)

---

## Add remote

    git remote add origin <repository-url>

---

## Change remote URL

    git remote set-url origin <repository-url>

---

## Show remote details

    git remote show origin

---

# 13. Git Push

Push local commits to a remote repository.

    git push origin main

---

## Push current branch and set upstream

    git push -u origin feature-login

After setting upstream, you can usually use:

    git push

---

## Example

    git switch -c feature-login
    echo "Login" > login.txt
    git add .
    git commit -m "Add login feature"
    git push -u origin feature-login

---

# 14. Git Fetch

## What does it do?

Downloads changes from the remote repository but does not automatically merge them into your current branch.

Command:

    git fetch origin

Think:

    GitHub
       |
    git fetch
       ↓
    origin/main

Your local `main` does not automatically move.

---

## Example

    git fetch origin

Then inspect:

    git log --oneline origin/main -5

Compare:

    git diff main..origin/main

---

# 15. Git Pull

## What does it do?

`git pull` generally performs:

    git fetch
    +
    integration

Command:

    git pull

---

## Example

    git switch main
    git pull

---

## Fetch vs Pull

    git fetch
    → Download remote changes
    → Do not integrate automatically

    git pull
    → Fetch
    → Integrate

---

## Interview answer

`git fetch` downloads changes from the remote repository without changing the current branch, while `git pull` fetches and then integrates the remote changes into the current branch.

---

# 16. Remote Tracking Branch

Example:

    main
        [local]

    origin/main
        [remote-tracking reference]

Check:

    git branch -vv

Example:

    * main  4f39ed5 [origin/main] Latest commit

This shows which remote branch the local branch tracks.

---

# 17. Branch Tracking

Create a branch and set upstream:

    git push -u origin feature-login

The `-u` sets the upstream branch.

After that:

    git push
    git pull

can usually be used without specifying the remote and branch.

---

# Day 23 — Git Rebase & Stash

---

# 18. Git Rebase

## What is rebase?

Rebase takes commits from your branch and replays them on top of another base.

Before:

    A --- B --- C
           \
            D --- E

Suppose `main` is at C and feature contains D and E.

Run:

    git switch feature
    git rebase main

After:

    A --- B --- C --- D' --- E'

D and E are replayed.

Because they are recreated, their commit IDs change.

---

## Basic rebase command

    git rebase main

---

## Practical example

    git switch feature-login
    git rebase main

---

## Why rebase?

It creates a cleaner, more linear history.

---

## Remember

    Rebase
    =
    replay my commits on top of another base

---

# 19. Rebase Conflict

A conflict can occur during rebase.

Check:

    git status

Resolve the conflicting file.

Then:

    git add file.txt
    git rebase --continue

---

## Skip the conflicting commit

    git rebase --skip

---

## Abort the rebase

    git rebase --abort

This returns the branch to the state before the rebase started.

---

## Complete flow

    git rebase main
           ↓
        Conflict
           ↓
      Fix the file
           ↓
       git add .
           ↓
    git rebase --continue

---

# 20. Merge vs Rebase

## Merge

    A --- B --- C
           \     \
            D --- E --- M

Characteristics:

- Preserves existing history
- Does not rewrite existing commits
- May create merge commits
- Shows branch topology

---

## Rebase

    A --- B --- C --- D' --- E'

Characteristics:

- Creates linear history
- Replays commits
- Creates new commit IDs
- Rewrites history

---

## Important rule

Avoid rebasing shared/public commits unless you understand the consequences.

---

# 21. Git Stash

## What is stash?

Stash temporarily stores uncommitted changes.

Example situation:

    You are working on feature-login
             ↓
    Your manager asks for urgent work
             ↓
    You need a clean working tree
             ↓
         git stash
             ↓
    Switch branch and work
             ↓
    Return later
             ↓
    Restore changes

---

## Stash current changes

    git stash

---

## Named stash

    git stash push -m "WIP login feature"

This is better when you have multiple stashes.

---

## List stashes

    git stash list

Example:

    stash@{0}: On feature-login: WIP login feature
    stash@{1}: On main: WIP dashboard

---

## Show stash

    git stash show stash@{0}

Show full patch:

    git stash show -p stash@{0}

---

# 22. Git Stash Apply

Apply a stash without removing it.

    git stash apply

Specific stash:

    git stash apply stash@{1}

After applying:

    git stash list

The stash is still present.

---

# 23. Git Stash Pop

Apply the stash and remove it from the stash list.

    git stash pop

Specific stash:

    git stash pop stash@{1}

---

# 24. Git Stash Drop

Remove one stash:

    git stash drop stash@{0}

---

# 25. Git Stash Clear

Delete all stashes:

    git stash clear

Use carefully.

---

# 26. Stash Apply vs Pop

    git stash apply
    → Restore changes
    → Keep stash

    git stash pop
    → Restore changes
    → Remove stash

---

# Day 24 — Cherry-Pick, Reset, Reflog & Revert

---

# 27. Git Cherry-Pick

## What is cherry-pick?

Cherry-pick applies the changes from a specific commit onto the current branch.

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

The changes from D are copied into a new commit D'.

---

## Basic command

    git cherry-pick <commit-id>

Example:

    git cherry-pick 6e6c3b0

---

## When is cherry-pick useful?

Example:

A bug fix exists on a feature branch:

    feature-hotfix
          |
          H  ← bug fix

You want only that fix on main.

    git switch main
    git cherry-pick H

You do not need to merge the entire feature branch.

---

# 28. Cherry-Pick Conflict

If a conflict occurs:

    git status

Resolve the file.

Then:

    git add file.txt
    git cherry-pick --continue

---

## Skip the commit

    git cherry-pick --skip

---

## Abort cherry-pick

    git cherry-pick --abort

---

# 29. Git Reset

## What does reset do?

Reset moves the current branch pointer to another commit.

The effect on staging and working files depends on the reset mode.

Three important modes:

- `--soft`
- `--mixed`
- `--hard`

---

# 30. Reset --soft

Command:

    git reset --soft HEAD~1

Moves HEAD backward.

Changes from the removed commit remain staged.

Example:

Before:

    A --- B --- C
                ^
               HEAD

Run:

    git reset --soft HEAD~1

After:

    A --- B
          ^
         HEAD

Changes introduced by C:

    Staging Area
         ↑
    Changes from C

---

## Use case

You committed too early and want to modify the commit.

Example:

    git commit -m "Wrong commit message"

Then:

    git reset --soft HEAD~1

Modify or add files.

Then:

    git commit -m "Correct commit message"

---

# 31. Reset --mixed

`--mixed` is the default mode.

Command:

    git reset --mixed HEAD~1

or simply:

    git reset HEAD~1

HEAD moves backward.

Changes remain in the working directory but are unstaged.

Example:

    A --- B --- C
                ^
               HEAD

After:

    A --- B
          ^
         HEAD

Changes from C:

    Working Directory
         ↑
    Unstaged changes

---

## Use case

You want to undo a commit but keep the changes so you can modify them before committing again.

---

# 32. Reset --hard

Command:

    git reset --hard HEAD~1

Moves HEAD backward and resets the staging area and working directory.

Before:

    A --- B --- C
                ^
               HEAD

After:

    A --- B
          ^
         HEAD

Changes from C are removed from the working directory.

---

## WARNING

`git reset --hard` can destroy uncommitted work.

Use carefully.

---

# 33. Reset Comparison

| Command | HEAD | Staging Area | Working Directory |
|---|---|---|---|
| `git reset --soft HEAD~1` | Moves | Changes staged | Changes remain |
| `git reset HEAD~1` | Moves | Changes unstaged | Changes remain |
| `git reset --hard HEAD~1` | Moves | Reset | Reset |

### Easy memory

    soft
    → keep staged

    mixed
    → keep unstaged

    hard
    → discard changes

---

# 34. Git Reflog

## What is reflog?

Reflog records local movements of Git references, especially `HEAD`.

Command:

    git reflog

Example:

    cb8b58b HEAD@{0}: reset: moving to HEAD~1
    92cd083 HEAD@{1}: commit: Reset practice commit C
    cb8b58b HEAD@{2}: reset: moving to HEAD~1
    d60c8f3 HEAD@{3}: commit: Reset practice commit C

---

## Why is reflog important?

Suppose you run:

    git reset --hard HEAD~1

and realize you removed the wrong commit.

Run:

    git reflog

Find the old commit:

    92cd083

Recover it:

    git reset --hard 92cd083

---

## Log vs Reflog

`git log`:

    git log --oneline

Shows reachable commit history.

`git reflog`:

    git reflog

Shows local movements of HEAD/references.

---

## Remember

    reflog
    =
    Git's local recovery map

---

# 35. Git Revert

## What is revert?

Revert creates a new commit that reverses the changes introduced by an earlier commit.

Before:

    A --- B --- C

Run:

    git revert C

After:

    A --- B --- C --- C'

C' undoes C.

---

## Command

    git revert <commit-id>

Example:

    git revert 4958ee7

Git may open an editor for the revert commit message.

---

## One-line example

    git revert --no-edit 4958ee7

---

# 36. Reset vs Revert

## Reset

    A --- B --- C

    git reset --hard B

    A --- B

The branch pointer moves backward.

---

## Revert

    A --- B --- C --- C'

C' reverses C.

History remains intact.

---

## Best rule

For a private/local branch:

    reset can be useful

For a shared branch:

    revert is generally safer

---

# 37. Revert Conflict

If revert causes a conflict:

    git status

Resolve the file.

Then:

    git add file.txt
    git revert --continue

---

## Skip the revert

    git revert --skip

---

## Abort the revert

    git revert --abort

---

## Example flow

    git revert 4958ee7

    CONFLICT (content): Merge conflict in revert-practice.txt

Then:

    cat revert-practice.txt

Resolve:

    git add revert-practice.txt
    git revert --continue

---

# Day 25 — Git Workflows

---

# 38. GitHub Flow

## What is GitHub Flow?

GitHub Flow is a simple workflow centered around:

- Main branch
- Short-lived feature branches
- Pull Requests
- Code review
- CI/CD
- Frequent merging

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
    Merge
      ↓
    Deploy

---

## Practical GitHub Flow

Start from updated main:

    git switch main
    git pull

Create feature branch:

    git switch -c feature-login

Make changes:

    echo "Login feature" > login.txt

Stage:

    git add login.txt

Commit:

    git commit -m "Add login feature"

Push:

    git push -u origin feature-login

Then create a Pull Request.

---

## Why GitHub Flow?

Good for:

- Startups
- Fast shipping
- Continuous Delivery
- CI/CD
- Short-lived feature branches
- Simple branching strategy

---

# 39. GitFlow

## What is GitFlow?

GitFlow is a more structured branching model.

Common branches:

    main
    develop
    feature/*
    release/*
    hotfix/*

Typical flow:

    feature/*
        ↓
    develop
        ↓
    release/*
        ↓
    main

---

## Feature branch example

    git switch develop
    git switch -c feature-login

Work:

    git add .
    git commit -m "Add login feature"

Merge:

    git switch develop
    git merge --no-ff feature-login

---

## Release branch

    git switch develop
    git switch -c release/1.0

---

## Hotfix branch

    git switch main
    git switch -c hotfix/critical-bug

---

## Characteristics

- More branches
- Longer-lived branches
- Structured release process
- Useful for scheduled releases
- Separate integration branch

---

# 40. Trunk-Based Development

## What is it?

Trunk-Based Development centers development around the main/trunk branch.

Developers integrate changes frequently.

Example:

    main ─────────────────────────────
          ↑       ↑       ↑       ↑
        commit  commit  commit  commit

Branches, if used, are usually very short-lived.

---

## Practical example

    git switch main
    git pull

Create short-lived branch:

    git switch -c feature-login

Make a small change:

    git add .
    git commit -m "Add login validation"

Push:

    git push -u origin feature-login

Create PR.

Merge quickly.

---

## Main idea

    Small changes
        ↓
    Short-lived branches
        ↓
    Frequent integration
        ↓
    Main stays healthy

---

# 41. Workflow Comparison

| Workflow | Main Idea | Branch Lifetime | Release Style |
|---|---|---|---|
| GitHub Flow | Feature branch + PR | Short | Continuous |
| GitFlow | Multiple structured branches | Longer | Scheduled |
| Trunk-Based | Main/trunk centered | Very short | Continuous |

---

## Easy memory

GitHub Flow:

    Simple + PR + short-lived branches

GitFlow:

    Many branches + structured releases

Trunk-Based:

    Main-centered + frequent integration

---

# 42. Which Workflow Should You Choose?

## Startup shipping quickly

GitHub Flow is often a practical choice because:

- Simple
- Easy to understand
- Short-lived branches
- Pull Request based
- Works well with CI/CD
- Supports continuous delivery

---

## Large release-oriented organization

GitFlow can be useful when:

- Releases are scheduled
- Multiple environments/releases need separation
- Teams need structured release branches
- Hotfix/release management is important

---

## Large engineering organization with strong CI/CD

Trunk-Based Development can work well when:

- Developers integrate frequently
- CI is strong
- Changes are small
- Main is kept stable
- Releases happen frequently

---

# Day 26 — GitHub CLI

---

# 43. What is GitHub CLI?

GitHub CLI (`gh`) allows you to interact with GitHub directly from the terminal.

Instead of opening the GitHub website for many tasks, you can use:

    gh

---

# 44. Check GitHub CLI Version

Command:

    gh --version

Example:

    gh version 2.97.0

---

# 45. GitHub CLI Authentication

Check authentication:

    gh auth status

Example:

    github.com
      ✓ Logged in to github.com account Aniruddhakharve
      - Active account: true
      - Git operations protocol: ssh

---

## Why authentication matters

It allows GitHub CLI to perform actions such as:

- Create issues
- Create Pull Requests
- Merge Pull Requests
- View repositories
- View Actions
- Create Gists

---

# 46. Repository Commands

## View repository

    gh repo view owner/repository

Example:

    gh repo view Aniruddhakharve/devops-git-practice

---

## View repository as JSON

    gh repo view Aniruddhakharve/devops-git-practice \
      --json name,owner,isPrivate,defaultBranchRef,url

---

## List repositories

    gh repo list Aniruddhakharve --limit 10

---

## Open repository in browser

    gh repo view --web

---

# 47. GitHub Issues

## Create issue

    gh issue create \
      --repo owner/repository \
      --title "Practice GitHub CLI issue" \
      --body "Created this issue while revising Day 26." \
      --label "documentation"

Example:

    gh issue create \
      --repo Aniruddhakharve/devops-git-practice \
      --title "Practice GitHub CLI issue" \
      --body "Created this issue while revising Day 26." \
      --label "documentation"

---

## List issues

    gh issue list \
      --repo owner/repository

---

## View issue

    gh issue view 3 \
      --repo owner/repository

---

## Close issue

    gh issue close 3 \
      --repo owner/repository

---

## List closed issues

    gh issue list \
      --repo owner/repository \
      --state closed

---

# 48. GitHub Pull Request Workflow

## Step 1 — Create branch

    git switch -c gh-cli-practice

---

## Step 2 — Create file

    echo "GitHub CLI PR practice" > gh-cli-practice.txt

---

## Step 3 — Stage

    git add gh-cli-practice.txt

---

## Step 4 — Commit

    git commit -m "Practice GitHub CLI pull request"

---

## Step 5 — Push

    git push -u origin gh-cli-practice

---

## Step 6 — Create PR

    gh pr create \
      --repo owner/repository \
      --base main \
      --head gh-cli-practice \
      --title "Practice GitHub CLI pull request" \
      --body "Created this pull request using GitHub CLI."

---

# 49. List Pull Requests

    gh pr list \
      --repo owner/repository

---

## List merged PRs

    gh pr list \
      --repo owner/repository \
      --state merged

---

## List closed PRs

    gh pr list \
      --repo owner/repository \
      --state closed

---

# 50. View Pull Request

    gh pr view 6 \
      --repo owner/repository

Example output:

    Practice clean GitHub CLI pull request
    owner/repository#6

    Merged

---

# 51. View Pull Request Diff

    gh pr diff 6 \
      --repo owner/repository

This shows the exact changes introduced by the PR.

---

## Compare local branch manually

    git log --oneline --decorate --graph main..gh-cli-clean

---

## Compare branch changes

    git diff main...gh-cli-clean

---

# 52. Check Pull Request CI Status

    gh pr checks 6 \
      --repo owner/repository

If there are no GitHub Actions checks:

    no checks reported

---

# 53. Merge Pull Request

Example:

    gh pr merge 6 --squash --delete-branch

This performs:

    Pull Request
         ↓
    Squash commits
         ↓
    Merge into main
         ↓
    Delete branch

---

## What does `--squash` mean?

If a PR contains multiple commits:

    A
    B
    C

Squash merge creates one commit on main:

    A
    |
    S

Where S contains the combined changes.

---

## What does `--delete-branch` mean?

After merging, GitHub CLI deletes the feature branch.

---

# 54. PR Merge and Local Divergence

A very important practical situation we experienced:

GitHub merged PR #6.

Remote:

    origin/main:
    
    A --- B --- C --- D
                      ^
                 origin/main

Local:

    main:
    
    A --- B --- C
                ^
               main

If local main also has commits that are not on remote:

    Local:
    
    A --- B --- C --- L1 --- L2 --- L3 --- L4
                                  ^
                                 main

    Remote:
    
    A --- B --- C --- D
                      ^
                 origin/main

Now the branches have diverged.

Check:

    git status

    git branch -vv

You may see:

    Your branch and 'origin/main' have diverged,
    and have 4 and 1 different commits each, respectively.

---

## Inspect remote main

    git fetch origin

    git log --oneline --decorate origin/main -5

---

## If you intentionally want local main to exactly match remote

    git reset --hard origin/main

Then:

    git status

Output:

    On branch main
    Your branch is up to date with 'origin/main'.

    nothing to commit, working tree clean

---

## WARNING

Do not blindly use:

    git reset --hard origin/main

If your local commits contain work you still need.

---

# 55. GitHub Actions

GitHub CLI can manage GitHub Actions from the terminal.

---

## List workflows

    gh workflow list \
      --repo owner/repository

Example:

    gh workflow list \
      --repo Aniruddhakharve/devops-git-practice

If there are no workflows:

    no workflows found

---

## List workflow runs

    gh run list \
      --repo owner/repository

If there are no runs:

    no runs found

---

## View a workflow run

    gh run view <run-id> \
      --repo owner/repository

Example:

    gh run view 31839765489 \
      --repo cli/cli

---

## View failed logs

    gh run view 31839765489 \
      --repo cli/cli \
      --log-failed

Useful for troubleshooting CI failures.

---

## Example failed output

    bash: /home/runner/work/_temp/gh-aw/actions/create_prompt_first.sh: No such file or directory

    ##[error]Process completed with exit code 127.

This demonstrates how `gh run view --log-failed` can help identify the exact failing command.

---

# 56. GitHub API with `gh api`

`gh api` allows you to call GitHub's REST API from the terminal.

Example:

    gh api repos/Aniruddhakharve/devops-git-practice

This returns repository information as JSON.

---

## Extract specific fields using `--jq`

    gh api repos/Aniruddhakharve/devops-git-practice \
      --jq '.name,.default_branch'

Output:

    devops-git-practice
    main

---

## Remember

    gh api
    =
    GitHub API from the command line

---

# 57. GitHub Search

Search repositories:

    gh search repos "kubernetes" --limit 5

Example:

    kubernetes/kubernetes
    kubernetes/minikube
    helm/helm
    k3s-io/k3s
    kubernetes/community

---

## Useful for

- Finding open-source projects
- Discovering repositories
- Exploring technologies
- Finding projects to study

---

# 58. GitHub Aliases

Aliases provide shortcuts for GitHub CLI commands.

List aliases:

    gh alias list

Example:

    co: pr checkout
    prs: pr list

Now instead of:

    gh pr list

You can use:

    gh prs

---

# 59. GitHub Releases

List releases:

    gh release list \
      --repo owner/repository

If there are no releases:

    no releases found

---

# 60. GitHub Gists

A Gist is useful for quickly sharing snippets or small files.

---

## Create a file

    echo "GitHub CLI Gist practice" > /tmp/gh-gist-practice.txt

---

## Create a secret gist

    gh gist create /tmp/gh-gist-practice.txt \
      --desc "GitHub CLI practice"

Example output:

    ✓ Created secret gist gh-gist-practice.txt

---

## List gists

    gh gist list

---

## View a gist

    gh gist view <gist-id>

---

## Public vs Secret Gist

Public:

- Publicly discoverable

Secret:

- Not publicly listed
- Anyone with the URL can access it

Secret does NOT mean password-protected.

---

# 61. GitHub CLI Complete Workflow

A complete CLI Pull Request workflow:

    gh auth status

    gh repo view owner/repository

    git switch main
    git pull

    git switch -c feature-demo

    echo "New feature" > feature.txt

    git add feature.txt

    git commit -m "Add new feature"

    git push -u origin feature-demo

    gh pr create \
      --base main \
      --head feature-demo \
      --title "Add new feature" \
      --body "Added new feature."

    gh pr view <pr-number>

    gh pr diff <pr-number>

    gh pr checks <pr-number>

    gh pr merge <pr-number> --squash --delete-branch

    git switch main

    git fetch origin

    git reset --hard origin/main

---

# Git Command Comparison

---

# 62. `git fetch` vs `git pull`

| Command | What it does |
|---|---|
| `git fetch` | Downloads remote changes |
| `git pull` | Fetches + integrates changes |

Memory:

    fetch = download

    pull = download + integrate

---

# 63. `git merge` vs `git rebase`

| Merge | Rebase |
|---|---|
| Combines histories | Replays commits |
| Preserves existing commits | Creates new commit IDs |
| May create merge commit | Usually linear history |
| Does not rewrite existing history | Rewrites commit history |

Memory:

    merge = combine

    rebase = replay

---

# 64. `git reset` vs `git revert`

| Reset | Revert |
|---|---|
| Moves branch pointer | Creates a new commit |
| Can rewrite history | Preserves history |
| Useful for local/private work | Safer for shared branches |
| Can discard changes with `--hard` | Reverses changes through a new commit |

Memory:

    reset = move history

    revert = undo with a new commit

---

# 65. `git reset --soft` vs `--mixed` vs `--hard`

| Mode | HEAD | Staging | Working Directory |
|---|---|---|---|
| `--soft` | Moves | Changes remain staged | Changes remain |
| `--mixed` | Moves | Changes become unstaged | Changes remain |
| `--hard` | Moves | Reset | Reset |

Memory:

    soft  → staged

    mixed → unstaged

    hard  → discard

---

# 66. `git stash apply` vs `git stash pop`

| Command | Restore changes | Remove stash |
|---|---|---|
| `git stash apply` | Yes | No |
| `git stash pop` | Yes | Yes |

---

# 67. Merge vs Cherry-Pick

Merge:

    git merge feature

Works at the branch/history level.

Cherry-pick:

    git cherry-pick <commit>

Works at the individual commit level.

Memory:

    merge = branch

    cherry-pick = commit

---

# 68. Reset vs Reflog

Reset:

    git reset --hard HEAD~1

Changes the current branch position.

Reflog:

    git reflog

Shows where HEAD/reference pointers were previously.

Memory:

    reset = change position

    reflog = find old position

---

# Git Mental Models

---

# 69. Working Directory → Staging → Repository

The most important Git flow:

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

---

# 70. Complete Developer Workflow

    Create/modify files
            ↓
       git status
            ↓
         git add
            ↓
       git diff --staged
            ↓
        git commit
            ↓
        git log
            ↓
        git push
            ↓
         GitHub
            ↓
      Pull Request
            ↓
       Code Review
            ↓
           CI
            ↓
          Merge
            ↓
         Deploy

---

# 71. Branch Mental Model

Example:

                    main
                     |
                     v
    A ---- B ---- C ---- D
           \
            E ---- F
                 ^
                 |
              feature

`main` and `feature` point to different commits.

---

# 72. Fast-Forward Mental Model

Before:

    A --- B
          ^
         main

    A --- B --- C --- D
                      ^
                    feature

After:

    A --- B --- C --- D
                      ^
                     main

No merge commit.

---

# 73. Merge Commit Mental Model

Before:

            C --- D
           /
    A --- B --- E --- F
                     ^
                    main

After:

            C --- D
           /       \
    A --- B --- E -- M
                     ^
                    main

`M` has two parents.

---

# 74. Rebase Mental Model

Before:

    A --- B --- C
           \
            D --- E

After:

    A --- B --- C --- D' --- E'

D and E are recreated.

Therefore:

    D != D'
    E != E'

Their commit IDs change.

---

# 75. Cherry-Pick Mental Model

Feature:

    A --- B --- C --- D

Main:

    A --- B --- X

Want only D:

    git switch main
    git cherry-pick D

Result:

    A --- B --- X --- D'

---

# 76. Reset Mental Model

Before:

    A --- B --- C
                ^
               HEAD

Command:

    git reset --hard HEAD~1

After:

    A --- B
          ^
         HEAD

---

# 77. Revert Mental Model

Before:

    A --- B --- C

Command:

    git revert C

After:

    A --- B --- C --- C'

C' reverses C.

---

# 78. Stash Mental Model

Before:

    Working Directory
    Modified files
         |
      git stash
         ↓
    Clean Working Directory

Later:

    git stash pop
         ↓
    Changes restored

---

# Common Conflict Recovery Commands

---

# 79. Merge Conflict

Start:

    git merge feature

Conflict:

    git status

Resolve file.

Then:

    git add .

    git commit

---

# 80. Rebase Conflict

Start:

    git rebase main

Conflict:

    git status

Resolve file.

Then:

    git add .

    git rebase --continue

Abort:

    git rebase --abort

---

# 81. Cherry-Pick Conflict

Start:

    git cherry-pick <commit>

Conflict:

    git status

Resolve.

Then:

    git add .

    git cherry-pick --continue

Abort:

    git cherry-pick --abort

---

# 82. Revert Conflict

Start:

    git revert <commit>

Conflict:

    git status

Resolve.

Then:

    git add .

    git revert --continue

Abort:

    git revert --abort

---

# 83. General Conflict Pattern

Most Git conflicts follow this pattern:

    Operation
        ↓
    Conflict
        ↓
    git status
        ↓
    Open conflicted file
        ↓
    Resolve manually
        ↓
    git add <file>
        ↓
    Continue operation

Depending on the operation:

    git commit
    git rebase --continue
    git cherry-pick --continue
    git revert --continue

---

# Interview Questions & Answers

---

# 84. What is Git?

Git is a distributed version control system used to track changes in source code and enable collaboration between developers.

---

# 85. What is GitHub?

GitHub is a platform for hosting Git repositories and providing collaboration features such as Pull Requests, Issues, Actions, and code review.

---

# 86. What is a Git branch?

A branch is a movable pointer to a commit that provides an independent line of development.

---

# 87. What is HEAD?

HEAD is the reference to the currently checked-out branch or commit.

---

# 88. What is a fast-forward merge?

A fast-forward merge happens when the target branch has not diverged, allowing Git to simply move the branch pointer forward without creating a merge commit.

---

# 89. What is a merge commit?

A merge commit combines two divergent histories and normally has two parent commits.

---

# 90. What is `--no-ff`?

`--no-ff` forces Git to create a merge commit even when a fast-forward merge would otherwise be possible.

Example:

    git merge --no-ff feature-login

---

# 91. What is rebase?

Rebase replays commits from one branch on top of another base commit.

Example:

    git switch feature
    git rebase main

---

# 92. Why does rebase change commit IDs?

Because Git recreates/replays the commits on a new base, resulting in new commit objects and therefore new commit IDs.

---

# 93. What is cherry-pick?

Cherry-pick applies the changes from a specific commit onto the current branch.

Example:

    git cherry-pick abc1234

---

# 94. What is Git stash?

Git stash temporarily stores uncommitted changes so that the working directory can be made clean.

Example:

    git stash push -m "WIP feature"

---

# 95. Difference between stash apply and stash pop?

`git stash apply` restores the stash but keeps it.

`git stash pop` restores the stash and removes it.

---

# 96. What is Git reset?

Reset moves the current branch pointer to another commit.

Its effect on the staging area and working directory depends on the mode.

---

# 97. Explain reset modes.

`--soft`:

    HEAD moves
    Changes remain staged

`--mixed`:

    HEAD moves
    Changes become unstaged

`--hard`:

    HEAD moves
    Staging and working directory are reset

---

# 98. What is Git reflog?

Reflog records local movements of Git references and is extremely useful for recovering commits after operations such as reset or rebase.

Example:

    git reflog

---

# 99. What is Git revert?

Revert creates a new commit that reverses the changes introduced by an earlier commit.

Example:

    git revert abc1234

---

# 100. Reset vs Revert?

Reset:

    Moves branch history.

Revert:

    Creates a new commit that undoes previous changes.

For shared branches, revert is generally safer.

---

# 101. Fetch vs Pull?

Fetch:

    Downloads remote changes.

Pull:

    Fetches and integrates remote changes.

---

# 102. Merge vs Rebase?

Merge:

    Combines histories.

Rebase:

    Replays commits onto another base.

---

# 103. Merge vs Cherry-Pick?

Merge operates on branches.

Cherry-pick operates on individual commits.

---

# 104. What is GitHub Flow?

A lightweight workflow using:

    main
      ↓
    feature branch
      ↓
    Pull Request
      ↓
    Review
      ↓
    CI/CD
      ↓
    Merge

---

# 105. What is GitFlow?

A structured branching model using branches such as:

    main
    develop
    feature/*
    release/*
    hotfix/*

---

# 106. What is Trunk-Based Development?

A development approach where teams integrate frequently into the main/trunk branch using very short-lived branches or direct integration.

---

# 107. What is GitHub CLI?

GitHub CLI is a command-line interface for interacting with GitHub.

Command:

    gh

---

# 108. How do you create a GitHub Issue using CLI?

    gh issue create \
      --repo owner/repository \
      --title "Bug report" \
      --body "Application is failing"

---

# 109. How do you create a Pull Request using CLI?

    gh pr create \
      --base main \
      --head feature-login \
      --title "Add login" \
      --body "Adds login functionality."

---

# 110. How do you inspect a Pull Request?

    gh pr view 5

    gh pr diff 5

    gh pr checks 5

---

# 111. How do you merge a Pull Request using CLI?

    gh pr merge 5 --squash --delete-branch

---

# 112. How do you inspect GitHub Actions?

    gh workflow list

    gh run list

    gh run view <run-id>

    gh run view <run-id> --log-failed

---

# 113. What is `gh api`?

`gh api` allows you to interact with the GitHub API directly from the terminal.

Example:

    gh api repos/owner/repository \
      --jq '.name,.default_branch'

---

# 114. What is a GitHub Gist?

A Gist is a simple way to share snippets or small files.

Example:

    gh gist create file.txt --desc "My practice"

---

# Daily Git Practice Checklist

## Day 22

- [ ] `git status`
- [ ] `git branch`
- [ ] `git branch --show-current`
- [ ] `git switch`
- [ ] `git switch -c`
- [ ] `git add`
- [ ] `git commit`
- [ ] `git log --oneline`
- [ ] `git log --graph --decorate --all`
- [ ] `git show`
- [ ] `git diff`
- [ ] `git diff --staged`
- [ ] `git merge`
- [ ] Fast-forward merge
- [ ] Merge commit
- [ ] `git merge --no-ff`
- [ ] Resolve merge conflict
- [ ] `git remote -v`
- [ ] `git fetch`
- [ ] `git pull`
- [ ] `git push`
- [ ] Remote tracking branches
- [ ] `git branch -vv`

---

## Day 23

- [ ] `git rebase`
- [ ] Understand replayed commits
- [ ] Resolve rebase conflict
- [ ] `git rebase --continue`
- [ ] `git rebase --skip`
- [ ] `git rebase --abort`
- [ ] `git stash`
- [ ] `git stash push -m`
- [ ] `git stash list`
- [ ] `git stash show`
- [ ] `git stash show -p`
- [ ] `git stash apply`
- [ ] `git stash pop`
- [ ] `git stash drop`
- [ ] `git stash clear`

---

## Day 24

- [ ] `git cherry-pick`
- [ ] Resolve cherry-pick conflict
- [ ] `git cherry-pick --continue`
- [ ] `git cherry-pick --skip`
- [ ] `git cherry-pick --abort`
- [ ] `git reset --soft`
- [ ] `git reset --mixed`
- [ ] `git reset --hard`
- [ ] Understand HEAD movement
- [ ] `git reflog`
- [ ] Recover a commit using reflog
- [ ] `git revert`
- [ ] Resolve revert conflict
- [ ] `git revert --continue`
- [ ] `git revert --skip`
- [ ] `git revert --abort`

---

## Day 25

- [ ] Understand GitHub Flow
- [ ] Practice short-lived feature branch
- [ ] Understand GitFlow
- [ ] Understand `main`
- [ ] Understand `develop`
- [ ] Understand `feature/*`
- [ ] Understand `release/*`
- [ ] Understand `hotfix/*`
- [ ] Understand Trunk-Based Development
- [ ] Compare all three workflows
- [ ] Know when each workflow is useful

---

## Day 26

- [ ] `gh --version`
- [ ] `gh auth status`
- [ ] `gh repo view`
- [ ] `gh repo list`
- [ ] `gh issue create`
- [ ] `gh issue list`
- [ ] `gh issue view`
- [ ] `gh issue close`
- [ ] `gh pr create`
- [ ] `gh pr list`
- [ ] `gh pr view`
- [ ] `gh pr diff`
- [ ] `gh pr checks`
- [ ] `gh pr merge`
- [ ] `gh workflow list`
- [ ] `gh run list`
- [ ] `gh run view`
- [ ] `gh run view --log-failed`
- [ ] `gh api`
- [ ] `--jq`
- [ ] `gh search repos`
- [ ] `gh alias list`
- [ ] `gh release list`
- [ ] `gh gist create`
- [ ] `gh gist list`
- [ ] `gh gist view`

---

# Final Git Memory Map

    git status
    ↓
    What is happening right now?

    git add
    ↓
    Put changes into staging

    git commit
    ↓
    Save staged changes as a commit

    git log
    ↓
    See commit history

    git branch
    ↓
    Manage branches

    git switch
    ↓
    Move between branches

    git merge
    ↓
    Combine branch histories

    git merge --no-ff
    ↓
    Force a merge commit

    git fetch
    ↓
    Download remote changes

    git pull
    ↓
    Fetch + integrate

    git push
    ↓
    Upload local commits

    git rebase
    ↓
    Replay commits onto another base

    git stash
    ↓
    Temporarily save unfinished work

    git cherry-pick
    ↓
    Apply one specific commit

    git reset
    ↓
    Move branch pointer

    git reflog
    ↓
    Find previous local HEAD positions

    git revert
    ↓
    Undo changes with a new commit

    gh
    ↓
    Manage GitHub from terminal

---

# The Most Important Git Concepts to Remember

## 1. Working Directory

Where you actually modify files.

---

## 2. Staging Area

Where you prepare changes for the next commit.

---

## 3. Commit

A saved snapshot of staged changes.

---

## 4. Branch

A pointer to a commit representing a line of development.

---

## 5. HEAD

Where you are currently checked out.

---

## 6. Merge

Combines branch histories.

---

## 7. Merge Commit

A commit that combines divergent histories and normally has two parents.

---

## 8. Rebase

Replays commits onto another base.

---

## 9. Stash

Temporarily stores unfinished work.

---

## 10. Cherry-Pick

Copies the changes from a specific commit into the current branch.

---

## 11. Reset

Moves the current branch pointer.

---

## 12. Reflog

Shows previous local reference movements and can help recover lost commits.

---

## 13. Revert

Creates a new commit that reverses an earlier commit.

---

## 14. GitHub Flow

Short-lived feature branch → Pull Request → Review → CI/CD → Merge.

---

## 15. GitFlow

Structured branch model → feature → develop → release → main.

---

## 16. Trunk-Based Development

Short-lived branches and frequent integration into main/trunk.

---

## 17. GitHub CLI

Use `gh` to interact with GitHub directly from the terminal.

---

# One-Line Interview Memory

    branch
    → separate line of development

    HEAD
    → where I am

    add
    → stage changes

    commit
    → save staged changes

    merge
    → combine branches

    merge commit
    → combine divergent histories

    fetch
    → download

    pull
    → download + integrate

    push
    → upload

    rebase
    → replay commits

    stash
    → temporarily save work

    cherry-pick
    → take one commit

    reset
    → move history/branch pointer

    reflog
    → recover old positions

    revert
    → undo with a new commit

    GitHub Flow
    → short-lived branch + PR

    GitFlow
    → structured release branches

    Trunk-Based
    → frequent integration into main

    GitHub CLI
    → GitHub from terminal

---

# Final Day 22–26 Roadmap

    DAY 22
    │
    ├── Branches
    ├── HEAD
    ├── Status
    ├── Add
    ├── Commit
    ├── Log
    ├── Diff
    ├── Merge
    ├── Fast-Forward Merge
    ├── Merge Commit
    ├── Merge Conflict
    ├── Remote
    ├── Push
    ├── Fetch
    ├── Pull
    └── Tracking Branches
         │
         ▼
    DAY 23
    │
    ├── Rebase
    ├── Rebase Conflict
    ├── Rebase Continue
    ├── Rebase Skip
    ├── Rebase Abort
    ├── Stash
    ├── Named Stash
    ├── Stash List
    ├── Stash Apply
    ├── Stash Pop
    ├── Stash Drop
    └── Stash Clear
         │
         ▼
    DAY 24
    │
    ├── Cherry-Pick
    ├── Cherry-Pick Conflict
    ├── Reset Soft
    ├── Reset Mixed
    ├── Reset Hard
    ├── Reflog
    ├── Recovery
    ├── Revert
    └── Revert Conflict
         │
         ▼
    DAY 25
    │
    ├── GitHub Flow
    ├── GitFlow
    └── Trunk-Based Development
         │
         ▼
    DAY 26
    │
    ├── GitHub CLI
    ├── Authentication
    ├── Repositories
    ├── Issues
    ├── Pull Requests
    ├── PR Diff
    ├── PR Checks
    ├── PR Merge
    ├── Squash Merge
    ├── Branch Deletion
    ├── GitHub Actions
    ├── Workflow Runs
    ├── Failed Logs
    ├── GitHub API
    ├── jq
    ├── Repository Search
    ├── Aliases
    ├── Releases
    └── Gists

---

# Git Mastery Goal

You do not need to memorize Git as hundreds of unrelated commands.

Understand the flow:

    Working Directory
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
      Remote Repository
            ↓
       Pull Request
            ↓
        Code Review
            ↓
           CI/CD
            ↓
          Merge
            ↓
          Deploy

And understand the recovery tools:

    Something went wrong
            ↓
       git status
            ↓
    What operation am I in?
            ↓
    merge / rebase / cherry-pick / revert
            ↓
    Resolve or abort
            ↓
    If history was moved accidentally
            ↓
       git reflog
            ↓
       Recover commit

---

# Final Rule

Do not just memorize:

    git reset
    git rebase
    git revert
    git cherry-pick

Always ask:

    What happens to HEAD?

    What happens to the branch pointer?

    What happens to the staging area?

    What happens to the working directory?

    Does this rewrite history?

    Is this safe on a shared branch?

    Can I recover if I make a mistake?

If you can answer those questions, you understand Git instead of simply memorizing Git commands.

---

# Day 22–26 Complete

Git fundamentals → Branching → Merging → Remote operations → Rebase → Stash → Cherry-Pick → Reset → Reflog → Revert → Git Workflows → GitHub CLI → Issues → Pull Requests → GitHub Actions → GitHub API → Gists

**Git is now a tool you can reason about, not just a list of commands.**
