# Git Docs: Everything to know about GIT

---

## Index

1. [One Time Setup per machine](#one-time-setup-per-machine)

2. [New Repo Workflow: Fork → Clone → Stage → Commit → Push → Pull](#new-repo-workflow-fork--clone--stage--commit--push--pull)
   - [1. Fork](#1-fork)
   - [2. Clone](#2-clone)
        - [Adding Remote](#adding-remote)
   - [3. Staging](#3-staging)
   - [4. Commit](#4-commit)
        - [Tag](#tag)
   - [5. Pushing](#5-pushing)
        - [Push tags](#push-tags)
   - [6. Pulling from remote or other branches (Fetch and Merge)](#6-pulling-from-remote-or-other-branches-fetch-and-merge)

3. [Merging](#merging)
   - [Merge outcomes](#merge-outcomes)
   - [Merge Conflicts](#merge-conflicts)

4. [Branches](#branches)
   - [Anecdote for understanding branches](#anecdote-for-understanding-branches)
   - [Branch commands](#branch-commands)
   - [What happens when you switch](#what-happens-when-you-switch)

5. [Tracking and Status](#tracking-and-status)
   - [1. Status](#1-status)
   - [2. Log](#2-log)
   - [3. Show](#3-show)
   - [4. Commit references: ^ and ~](#4-commit-references--and-)
   - [5. Reflog](#5-reflog)

6. [Helpful Git Commands](#helpful-git-commands)
   - [1. Stash](#1-stash)
   - [2. Restore](#2-restore)
   - [3. rm (Remove files)](#3-rm-remove-files)
   - [4. Reset](#4-reset)
   - [5. git diff](#5-git-diff)

---

## One Time Setup per machine

- **Set your identity** (required for commits):

  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "you@example.com"
  ```

- **Default branch name** (optional; Git now uses `main` by default):

  ```bash
  git config --global init.defaultBranch main
  ```

- **View all config**: `git config --list`

---

## New Repo Workflow: Fork → Clone → Stage → Commit → Push → Pull

### 1. Fork

- Go to someone else's repository
- It either needs to be public or you need to have access to it.
- Click the `Fork` icon and create your copy of the current repo in your github account.

### 2. Clone

- Go to your repository on github and click the `Code` icon to reveal and copy the https link to the repo.(Also called the repo-url)
- Navigate to the directory where you want to clone the repo

```bash
cd <path-to-proj-dir/>
```

```bash
git clone <repo-url> .
```

- Initializes Git in the proj-dir/, sets `origin` to the repo-url and sets default upstream to origin.
- `.` signifies that git should clone the repo's content in the current directory(proj-dir/).

**Alternatively**

```bash
git clone <repo-url> 
```

- Creates a folder by the name of the repo and copies all the content of the repo in that folder.

<br>

Now open these the proj-dir/ in an editor like VS code and create the following files:

1. `.gitignore`: For adding all files/folders in the proj-dir/ that should be ignored by git, so they don't show up in git status as untracked files.
2. `README.md`: Used to describe your project in a markdown file. Rendered on the github webpage of your repo.
3. `.gitattributes`: Used to describe things like which files will be used interpreted by git as binary(raw bytes to be used by a program) vs text(shown as human readable content)

#### Adding Remote

From your local git repo you can access remote repositories. By default the repository you cloned is set to `origin`.

Another common remote to add is the original repo you forked from and set it is as `upstream`:

```bash
git remote add upstream <parent-repo-url>
```

### 3. Staging

Once you are done making changes to your project in an editor you need to add the files in their current state to the staging area(index) of the local repo. This process is called staging a file.

Once an untracked file is staged with `git add`, it becomes tracked. After that, Git will notice later edits to that file and show them as `Changes not staged for commit` until you stage those edits too.

It can be done in the following ways:

```bash
git add <file-path>        # stage one file
git add .             # stage all changes in current directory 
git add -A            # stage all changes in the entire local repo no matter which level of the directory you call it from
```

**Staging Area/Index** is originally a copy of the commit that HEAD points to. When you add file to it, you are changing that copy. So you could have 3 different version of the same file:

```text
HEAD                 = Version 1 # last commit
Staging Area / Index = Version 2 # staged version of the file
Working Directory    = Version 3 # most recently edited version of the file.
```

### 4. Commit

When you are done staging the files you want in the next commit, you take a new snapshot of the staging area.

A commit only requires the staging area to be different from the current commit (`HEAD`) in at least one way. You can stage some edited files and leave other edited files unstaged to exclude them from this commit.

A commit requires a commit message, and is made like this:

```bash
git commit -m "Your commit message"  
```

Every commmit has a unique ***commit hash code*** associated with it that acts as its unique identifier.

Get more context about [Branches](#branches) of commits in its section below.

#### Tag

You could mark a specific commit (e.g. specific release, version of your software, etc) by using a tag like this:

  ```bash
  git tag v1.0.0                # lightweight tag at current commit
  git tag v1.0.0  <commit-name> # <commit-name> could be hash or pointer name
  git tag -a v1.0.0 -m "msg"    # annotated tag (recommended)
  ```

These tags are readily visible on github and can be used by the general user of the github repo to easily identify and access a specific commit.

### 5. Pushing

Pushing usually means sending your line of commits(commit history) to a remote repo.

```bash
git push -u origin main   # first time: set upstream and push
git push                 # later: push to the tracking branch
```

`-u` sets the default upstream to `origin/main` for successive push, pull, fetch commands on the main branch. If you create a different branch locally you have to set a different upstream for that branch

#### Push tags

Pushing a commit does not automatically push the tag to remote, you need to push tags by:

```bash
git push origin v1.0.0   # pushes this one tag to origin
git push origin --tags   # pushes all tags to origin
```

### 6. Pulling from remote or other branches (Fetch and Merge)

**Fetch**: Download updates from remote without changing your working files:

```bash
git fetch origin           # all branches from origin
git fetch origin other-branch   # only a specific branch
```

**Merge**: Combine another branch (or fetched ref) into your current branch:

```bash
git merge origin/main      # merge fetched origin/main
git merge other-branch     # merge another local branch 
```

**Pull**: Fetch from remote + merge into your current branch.

```bash
git pull                    # fetch + merge from upstream (if set)
git pull origin main        # fetch origin and merge origin/main into current branch
git pull --rebase           # fetch + rebase current branch on top of upstream
```

## Merging

Merging means combining the commits of two branches so one branch includes the other’s history. It can most commonly be carried out using these commands, which differ in how the final history turns out:

| Method | Command | Result |
|--------|--------|--------|
| **Merge** | `git merge other-branch` | One of the [Merge Outcomes](#merge-outcomes) depending on the relation between branches|
| **Squash** | `git merge --squash other-branch` | Combines all changes from the other branch into one set of changes, applies them to the current branch’s working tree and stages them.<br> Does not create a commit.<br> You need to run `git commit` to create the single squash commit. |
| **Rebase** | `git rebase <target-branch>` (while on curr-branch)  | curr-branch = The branch that gets moved.<br>The target-branch = The new base for the replayed commits of curr-branch <br> It takes all the commits from curr-branch and replays(same metadata, new commit) them onto tip of the target-branch as the new base for the curr-branch, so history stays linear and no merge commit is created.  <br>**Flags (full commands):**<br>• `git rebase --abort` — Cancels the rebase and restores your branch to its state before the rebase; use when you want to give up.<br>• `git rebase --continue` — Resumes the rebase after you’ve resolved conflicts and staged the files; use when you’re ready to finish.<br>• `git rebase --skip` — Drops the current commit being replayed and continues; use when that commit is redundant or you no longer want it.<br>• `git rebase -i <target-upstream>` — Opens an editor to pick, reorder, squash, or edit commits before they’re replayed on the target-upstream commit ; use when you want to clean up or rearrange history.<br>• `git rebase --update-refs <target-branch>` — After replaying commits, automatically moves any local branch pointers that pointed to the old commits so they point to the rewritten commits instead; useful when you have stacked branches.|
| **Cherry-pick** | `git cherry-pick <commit-hash>` or `git cherry-pick <commit1> <commit2>...` (while on target-branch) | Applies one or more specific commits (by hash) from another branch onto the current branch, creating new commits with the same changes. Use when you want only selected commits, not a full merge or rebase.<br>**Flags (full commands):**<br>• `git cherry-pick --abort` — Cancels the cherry-pick and restores the branch to its state before the operation.<br>• `git cherry-pick --continue` — Resumes after resolving conflicts and staging the files.<br>• `git cherry-pick --skip` — Skips the current commit and continues with the rest.<br>• `git cherry-pick -n` (or `--no-commit`) — Applies the changes without committing; lets you stage and commit manually (e.g. to squash several cherry-picked commits into one). |

### Merge outcomes

Executing a merge could go one of three ways:

| Outcome | Meaning |
|--------|--------|
| **Fast forward** | - In case there is no separate line of history which means technically the other branch is just a few commits ahead of the current branch and the current branch has no commits that aren't on the other branch.  <br> - The current branch pointer just moves forward to the tip of the other branch.<br>- This doesn't create any kind of new commit in log history |
| **Three-way Merge, no conflict** | - There are two separate lines of history meaning current branch has commits that do not exist on the other branch. <br>- Git performs a 3 way merge commit with two parents and a base(the common ancestor commit between two branches) and writes an automatic merge commit message|
| **Three-way Merge with conflict** | - Same as above but Git can’t auto-merge;<br> - Merge needs to be resolved as discussed in [Merge Conflict](#merge-conflicts) |

### Merge Conflicts

A conflict happens when two branches with different lines of history change the same part of the same file and Git can’t decide which version to keep. Conflicts can be from merge, squash or rebase.

**Anecdote to understand cause of conflicts**

- You make commits on a feature branch; meanwhile, others commit on main. The histories split: each branch has new commits that the other doesn’t.
- You try to merge the feature branch into main. Git looks at both branch tips. When the same lines in the same file were changed differently on each branch, Git cannot decide which version to pick automatically since the choice is not linear as it was in the case of fast-forward where git could just choose the latest commit. Now we are dealing with separate lines of history and none of the tips could be said to be an update of the other.
- Git writes conflict markers around those overlapping sections of each of these conflict files. You then manually have to deal with the conflict as described below.

**How to handle conflicts**:

1. You are on curr-branch and try to merge other-branch into it. You can preemtively tell Git to prefer one side for conflicting regions by using `-X ours` (prefer current branch) or `-X theirs` (prefer the branch being merged):

   ```bash
   git merge other-branch            # No Flag for merge strategy
   git merge -X ours other-branch    # Merge Strategy: prefer curr-branch (ours)version in conflicts
   git merge -X theirs other-branch  # Merge strategy prefer other-branch(theirs) version in conflicts
   ```

`-X` is used to set merge strategy preemptively if you are sure which branch's version you would prefer for each case of conflict during this merge.

2. In case you didn't use `-X`, If Git reports a conflict, run `git status` to see which files are conflicted. For each file you can:

    - **Take our version** (curr-branch):

   ```bash
   git restore --ours <file>
   git add <file>
   ```

   - **Take their version** (other-branch):

   ```bash
   git restore --theirs <file>
   git add <file>
   ````

   - **Resolve manually** (next step) if you need to decide for each section of conflict in a file.

3. To resolve manually: open each conflicted file in VS Code (or your editor). You’ll see conflict markers:

   ```
   <<<<<<< HEAD
   our version
   =======
   their version
   >>>>>>> other-branch
   ```

   Edit the file to remove the markers(<<<====>>>) and keep whatever content you want.

4. Once you are sure that you handled the conflict in each file that appeared in the `git status` after the mergem. <br>
You can stage the resolved files: `git add <file>` (or `git add .` if all are resolved). But be aware that even if the conflict is not resolved, git is not going to alert you and can commit broken code. <br>One easy way to avoid this is to search for the conflict markers using VS codes find(⌘⇧F) across the entire directory

5. Finish the resolution:
   - **Merge and Squash conflict:** Stage the resolved files, then run `git commit` (Git will use the default merge message).
   - **Rebase conflict:** Stage the resolved files, then run `git rebase --continue` — do **not** run `git commit`. In a rebase, Git creates the commit for you when you continue; using `git commit` would add an extra unwanted commit.

6. In case the conflict is too much to handle, `git merge --abort` or `git rebase --abort`: all conflict markers are removed and files are restored to the state of the current branch before the merge or rebase started.

---

## Branches

To understand the concept of branches, remember that each commit is recognized by its unique hash code. <br>
Each successive commit is linked to its previous commit by a parent–child relationship. Now, just to make it easier to traverse, we use named pointers to point to the most recent commits on a branch.

**HEAD**: A pointer to the commit you are currently on (and usually to the current branch). It moves to whichever commit you checkout or switch to.

**Branch name**: A pointer that moves to the latest commit on that branch when you commit. Creating a branch just creates a new pointer.

**Detached HEAD:** Normally HEAD points to a branch name (e.g. `main`), and that branch points to a commit. So, Detached HEAD means HEAD points directly to a commit, not to a branch. <br> If you make new commits while in detached HEAD, they are not on any branch. Once you switch away (e.g. back to `main`), that new commit may become unreachable unless you have its hash or the reflog. Git may eventually remove it during garbage collection.

### Anecdote for understanding branches

- Say you are on `main` at commit A, so HEAD also points to A.

- You create a new-branch. Creating a branch just means the pointer `new-branch` now points to A; you have not switched yet.

- Say you commit right now, you would be committing to `main` branch, so that would add a commit B to `main` with parent A. It causes `main` and `HEAD` to move to B.

- After that you switch to `new-branch` which still points at A and hence `HEAD` also points to A.  `main` keeps on pointing to commit B.

- The next commit you make will be commit C on `new-branch` also with parent A. This is how separate branches are linked using branch pointers.

### Branch commands

Creating a branch and switching to it

```bash
git branch new-branch   # create a new branch at current commit
git switch new-branch   # switch to new-branch
git switch -c new-branch   # create and switch to new-branch
git switch --detach <commit>   # Switch to a specific commit, <commit> could a hash or something like HEAD~2
```

Deleting a branch

```bash
git branch -d branch-name   # delete if merged
git branch -D branch-name   # force delete even if not merged
git push origin --delete branch-name   # delete on remote
```

### What happens when you switch

When you run `git switch <branch>` or `git switch --detach <commit-hash>`, Git does three things:

1. **HEAD** moves to the target branch (or directly to the commit, if detached).
2. **The index (staging area)** is updated to match the target commit.
3. **The working directory** is updated so your files on disk match that commit.

So after a switch, your project folder reflects the snapshot of the branch or commit you switched to. Any uncommitted changes in the way are either carried over (if possible), or Git will refuse the switch and ask you to commit or stash first.

---

## Tracking and Status

### 1. Status

```bash
git status
```

Shows:

- Current branch name.
- Whether you’re ahead/behind the remote after fetch.
- **Staged changes to be committed.**
- **Changes not staged for commit**(modified files).
- **Untracked files:** Files not in the stagign area.

### 2. Log

You can see the entire history of your current branch from the HEAD back through to its earliest ancestor.
<br>Easiest way to get hash of a specific commit.

`git log` could be used with a variety of Flags irrespective of their order:

| Flag | What it does |
|--------|----------------|
| `git log` | Full history with messages, authors, dates. |
| `git log --oneline` | One line per commit (short hash + message). |
| `git log --graph` | shows branch/merge structure with ASCII graph. |
| `git log --stat` | Per-commit summary of files and add/delete line counts. |
| `git log --name-status` | Lists files changed per commit (Added/Modified/Deleted). |

### 3. Show

Inspect a **single commit** in detail (message, author, date, and the full diff):

```bash
git show <commit>
git show <commit> --stat    # summary of changed files only
git show <commit>:<path>   # show file contents at that commit
```

Useful when you have a commit hash from `git log` and want to see exactly what changed.

### 4. Commit references: `^` and `~`

Both refer to ancestors of a commit, but `~` walks back along the first-parent chain (good for “main line” history). `^` lets you pick which parent at merge points.:

| Notation | Meaning |
|--------|---------|
| `~` (tilde) | Takes steps back along the current branch. `HEAD~1` = one step back along the current branch. `HEAD~2` = two steps back along current branch. |
| `^` (caret) | Chooses which parent. `HEAD^1` = first parent, `HEAD^2` = second parent(in case of merge commit).  |

**Example of you standing on a Merge commit (2 parents):**  

- `HEAD~1` One step back along the curr branch = `HEAD^1`(first parent of current commit(the branch you merged *into*, e.g. `main`)).  
- `HEAD^2` = second parent (the branch you merged *in*, e.g. a feature branch).  
- `HEAD~2` = first parent of the first parent (two steps back along the first-parent line).  
- `HEAD^2~1` = one step back from the second parent (along the merged-in branch).

### 5. Reflog

Shows a log of **where HEAD has been** (checkouts, resets, merges, etc.). Useful to recover “lost” commits or branches after a reset or mistaken switch to a commit:

```bash
git reflog
git switch --detach <commit-hash>   # or git reset --hard <commit-hash> to restore
```

---

## Helpful Git Commands

### 1. Stash

Temporarily save uncommitted changes and clean the working tree:

```bash
git stash                    # stash changes (including staged)
git stash -u                 # include untracked files
git stash list               # shows a stacked view(LIFO) of your stashed changes
git stash pop                # apply most recent stash and remove it
git stash apply              # apply most recent stash, keep it
git stash drop               # remove most recent stash
```

### 2. Restore

Restore files in the working tree (and optionally the index) to a given state:

| Command | Effect |
|--------|--------|
| `git restore <file>` | Restore file in working directory from the Staging Area/Index; Index ─► Working Directory |
| `git restore --staged <file>` | Restores staged version of the file from HEAD and unstages the file since INDEX ==HEAD for that file; HEAD ─► Index; Working Directory version of the file does not change;  |
| `git restore --source=<tree> <file>` | Restore file in the working directory from another commit(e.g. `--source=main`). |

### 3. `rm` (Remove files)

Remove from both the working directory and Git’s tracking:

```bash
git rm <file>                # delete file and stage the deletion
git rm -r <directory>        # recursive: remove directory and contents
git rm --cached <file>       # remove the file from Git's index/staging area, but keep it as is on disk
                             # this stages a deletion for the next commit
                             # File shows as untracked from now on
                             # Needs to be followed with an addition to .igitignore for the file to be completely ignored and not pop up as untracked from now on
```

### 4. Reset

Moving the current branch pointer(and optionally the index and working tree) to another commit

| Option | Effect |
|--------|--------|
| `git reset <commit>` (` default = `--mixed`) | Move the branch pointer to the commit and HEAD follows with it; Staging area reset to commit; working tree remains as is |
| `git reset --soft <commit>` | Move branch pointer to commit; keep index and working tree (changes stay staged) as is. |
| `git reset --hard <commit>` | Move branch to commit; reset index and working tree (all local changes lost) to the commmit. |

Example: `git reset --soft HEAD~1` undoes the last commit but keeps changes staged.

### 5. `git diff`

Show differences between trees (working tree, index, commits):

| Command | Compares |
|--------|----------|
| `git diff` | Working tree vs index (unstaged changes). |
| `git diff --staged` | Index vs HEAD (staged changes). |
| `git diff HEAD` | Working tree vs HEAD (staged + unstaged). |
| `git diff commit1 commit2` | Two commits (or branch tips) line by line. |
| `git diff --stat` | Summary: files and add/delete counts. |
| `git diff --name-status` | Only file names and status (Added/Modified/Deleted). |

---

*Guide covers: one-time setup, repo workflow (fork → clone → stage → commit → push → pull), merge conflicts, status/log/reflog, branches, and essential commands (stash, restore, rm, reset, diff).*
