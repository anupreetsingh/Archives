# Git Docs: Everything to know about GIT

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

## HEAD (Current Pointer)

**HEAD** is the pointer to the commit you are currently on. Most of the time, `HEAD` is attached to a branch name, and the branch name resolves to a commit hash:

```text
HEAD -> main -> commit A
```

In that normal attached state, `HEAD` pointer follows branch name pointer so actions like making a commit moves the current branch pointer forward, and `HEAD` along with it.

**Detached HEAD:** means HEAD points directly to a commit, not to a branch:

```text
HEAD -> commit A
main -> commit B
```

If you make new commits while in detached HEAD, they are not on any branch. Once you switch away (e.g. back to `main`), that new commit may become unreachable unless you have its hash, you tagged it or can locate it in the reflog. Git may eventually remove such unreachable commits during garbage collection.

### Switch

When you run `git switch <branch>` or `git switch --detach <commit-hash>`, Git changes where `HEAD` points:

1. **HEAD** moves to the target branch (or directly to the commit, if detached).
2. **The index (staging area)** is updated to match the target commit.
3. **The working directory** is updated so your files on disk match that commit.

So after a switch, your project folder reflects the snapshot of the branch or commit you switched to. Any uncommitted changes in the way are either carried over (if possible), or Git will refuse the switch and ask you to commit or stash first.

## Branches

To understand the concept of branches, remember that each commit is recognized by its unique hash code. <br>
Each successive commit is linked to its previous commit by a parent–child relationship. Now, just to make it easier to traverse, we use named pointers to point to the most recent commits on a branch.

**Branch name**: A pointer that moves to the latest commit on that branch when you commit. Creating a branch just creates a new pointer.

The current branch is the branch that `HEAD` is attached to. When you commit while `HEAD` is attached to a branch, Git creates the new commit and moves that branch pointer forward to it.

### Example of how branch pointers move

- Say you are on `main` at commit A, so `HEAD` is attached to `main`, and `main` points to A.

- You create a new-branch. Creating a branch just means the pointer `new-branch` now points to A; you have not switched yet.

- Say you commit right now, you would be committing to `main` branch, so that would add a commit B to `main` with parent A. It causes `main` to move to B, and `HEAD` follows because it is attached to `main`.

- After that you switch to `new-branch`, which still points at A. `HEAD` is now attached to `new-branch`, and `main` keeps on pointing to commit B.

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

### Reset

Reset and move the current branch pointer (and optionally the index and working tree) to another commit.

| Option | Effect |
|--------|--------|
| `git reset <commit>` (default = `--mixed`) | Move the current branch pointer to the commit and HEAD follows with it; Staging area reset to commit; working tree remains as is |
| `git reset --soft <commit>` | Move current branch pointer to commit; keep index and working tree (changes stay staged) as is. |
| `git reset --hard <commit>` | Move current branch pointer to commit; reset index and working tree (all local changes lost) to the commit. |

Example: `git reset --soft HEAD~2` moves the branch pointer to HEAD~2 but keeps changes from the previous tip staged.

## Merge

If two branches share a common ancestor commit in their history, Git can combine the changes from those branches using a merge operation.

- **Command:** Run `git merge <incoming-branch>` while `HEAD` is on the `<current-branch>`.
- **Intent:** Bring in and apply all combined changes from the `<incoming-branch>` that are introduced in commits not part of the `<current branch>` while preserving the relationship between the two branch histories.

Executing a merge goes in 4 directions depending on whether the branch histories have split:

### No split history

- The two branch histories have not split into two separate lines.
- One branch is simply ahead of the other branch.

#### Fast-forward merge

Example:

```text
A---B main
     \
      C---D feature

After:

A---B
     \
      C---D main, feature
```

- Run `git merge feature` while on `main`.
- In this example, `main` is the current branch and `feature` is the incoming branch.
- `feature` has commits `C` and `D` that are missing from `main`.
- Instead of creating a new commit, Git only moves the `main` pointer forward to the tip of `feature`, bringing the changes from the incoming branch into the current branch.

#### Already up to date

Example:

```text
A---B main
     \
      C---D feature

After:

A---B main
     \
      C---D feature
```

- Run `git merge main` while on `feature`.
- In this example, `feature` is the current branch and `main` is the incoming branch.
- `main` has no commits that are missing from `feature` so there is no change to be brought in.
- Git reports `Already up to date`.
- Git does not create a new commit.
- Git does not move either branch pointer.

### Split history

#### Three-way merge applies cleanly

Example:

```text
A---B---C main
 \
  D---E feature

After:

A---B---C---M main
 \         /
  D---E feature
```

- There are two separate lines of history.
- Run `git merge feature` while on `main`.
- In this example, `main` is the current branch and `feature` is the incoming branch.
- `main` has commits `B` and `C` that do not exist on `feature`.
- `feature` has commits `D` and `E` that do not exist on `main`.
- Both branches have moved ahead from the common base ancestor, so their histories have split apart.
- Git uses three snapshots to perform the merge: the tip of `main`, the tip of `feature`, and their common base ancestor.
- Git creates a merge commit with 2 parents: the old tip of `main` and the tip of `feature`.
- Git writes an automatic merge commit message.
- The `main` pointer moves to the new merge commit.
- The `feature` pointer stays where it was.

**Why no Conflict?**

In this case, both branches have their own changes in separate commits exclusive to the branches, but they do not change the same part of the same file compared to the base ancestors version.

File-content example:

```text
Base version:
line 1: title
line 2: description

main changes line 1:
line 1: better title
line 2: description

feature changes line 2:
line 1: title
line 2: better description
```

#### Three-way merge raises conflict

- Git tries to do a regular three-way merge, but both branches changed the same part of the same file compared to the base ancestor's version.
- Git cannot automatically decide which branch's change to favor, so it marks that part of the file as conflicted.

Example:

```text
Base version:
line 1: title
line 2: description

Current branch changes line 1:
line 1: better title
line 2: description

Incoming branch also changes line 1:
line 1: alternate title
line 2: description
```

- Both branches moved ahead from the same base version.
- Both branches changed `line 1`, but they changed it in different ways.
- Neither branch tip is simply a newer version of the other, so Git cannot pick one automatically.
- You need to manually resolve the conflicting changes as discussed in [Resolving Conflicts](#resolving-conflicts).

### Squash merge

- **Command:** `git merge --squash <incoming-branch>` while on the `<current-branch>`.
- **Intent:** Bring the changes from `<incoming-branch>` into the current branch's working tree and staging area as one combined set, without bringing in any commits.
- **Result:**
  - Combines all changes from the incoming branch into one set of changes.
  - Applies those changes to the current branch's working tree and stages them.
  - Does not create a commit.
  - Can produce conflicts by the same mechanism as a regular merge: Git compares the merge base, the current branch, and the incoming branch. If both sides changed the same part differently, Git pauses and the conflict needs to be dealt with as discussed in [Resolving Conflicts](#resolving-conflicts).
  - You need to run `git commit` to create the single squash commit.

After you commit the staged squash result, the `<current-branch>` has one new normal commit containing the combined changes.

## Patch Replay

A **diff** is the comparison between two snapshots. It describes what lines/files were added, removed, or changed between those snapshots.

A **patch** is a diff represented as something Git can try to apply somewhere else.

Because a commit stores a snapshot, Git can compare that commit's snapshot with its parent's snapshot and produce a **diff introduced by that commit** and that diff could then be applied somewhere else as a **diff-patch introduced by the commit**.

**cherry-pick** and **rebase** are patch replay operations that apply diff-patches introduced by individual commits onto a different starting commit, creating new commit objects from the results.

In everyday language, "replaying commits" is simpler way of saying "replaying the diff-patch introduced by that commit."

### Cherry-pick

A cherry-pick applies the diff-patch of one or more selected commits onto the current branch or HEAD pointer.

- **Command:** `git cherry-pick <selected-commit>` while on the `<current branch>`.
- **Intent:** Take the diff patch introduced by one specific commit and replay that patch onto the tip of `<current branch>`.

Cherry-pick has no concern with `<selected-commit>` and `<current-branch>` having a common base ancestor. It works regardless. Just to prove that point our example discusses the case of not having a common ancestor because the mechanism of cherry-picking would be the same.

#### Applies cleanly

Example:

```text
A---B---C main

W---X---Y---Z---P---Q other-history

After:

A---B---C---Z' main

W---X---Y---Z---P---Q other-history
```

- Run `git cherry-pick Z` while on `main`.
- Git produces the diff-patch introduced by `Z` by comparing `Z` with its parent `Y`.
- Git tries to apply that patch onto the current `HEAD`, which is at the tip of `main` i.e. `C`.
- If the patch applies cleanly, Git creates a new commit `Z'` on `main`.
- `Z'` contains the same change introduced in `Z`, but it is a new commit with a different hash.
- The `main` pointer moves to `Z'`.
- The `other-history` pointer stays at `Q`.
- Changes from commits `W`, `X`, `Y`, `P`, and `Q` are not brought into `main` just because `Z` was cherry-picked.

**Why no Conflict?**

In this case, the diff-patch introduced by `Z` fits cleanly onto the file state at `C`. This is because `C` was in a state that the diff-patch expects it to be in to introduce that change.

File-content example:

```text
Y = parent of cherry-picked commit:
line 1: title
line 2: description

Z = commit being cherry-picked:
line 1: title
line 2: better description

C = current HEAD on main:
line 1: better title
line 2: description

Z' = new commit created on main:
line 1: better title
line 2: better description
```

The diff-patch introduced by `Z` is effectively:

```diff
-line 2: description
+line 2: better description
```

When Git applies that patch onto `C`, it can still find `line 2: description`, so the patch applies cleanly. Git keeps the unrelated change already present in `C` on line 1 and adds the cherry-picked change on line 2.

#### Raises conflict

Cherry-pick tries to apply the diff-patch introduced by `Z` onto `C` but `C` is not in the state that the diff-patch of `Z` expected it to be.

Example:

```text
Y = parent of cherry-picked commit:
line 1: title
line 2: description

Z = commit being cherry-picked:
line 1: title
line 2: better description

C = current HEAD on main:
line 1: title
line 2: alternate description
```

The diff-patch introduced by `Z` expects to replace:

```diff
-line 2: description
+line 2: better description
```

But the `C` already has `line 2: alternate description`, so Git cannot confidently apply the patch. Git pauses the cherry-pick and marks that area of the file as conflicted.

You need to manually resolve the conflicting changes as discussed in [Resolving Conflicts](#resolving-conflicts).

#### Multiple commits

```bash
git cherry-pick <commit1> <commit2> <commit3>
```

Git applies them in the order listed. Each selected commit is still handled separately: Git computes a diff-patch for that commit, applies that patch, and normally creates one new commit for each cleanly applied diff-patch.

You can also cherry-pick a range of commits:

```bash
git cherry-pick B^..D
```

This includes commits `B`, `C`, and `D`. The `^` is used because ranges like `B..D` exclude `B`, while `B^..D` starts from `B`'s parent and therefore includes `B`. Git still replays the commits one at a time, with a separate diff patch for each commit.

#### No Commit

By default, a clean cherry-pick creates a new commit immediately after applying the selected commit's diff-patch.

`git cherry-pick -n <commit>` or `git cherry-pick --no-commit <commit>` applies the diff-patch to your working tree and staging area without creating a commit.

This is useful when you want to cherry-pick multiple commits but combine their changes into one manual commit.

### Rebase

Rebase means putting the current branch onto a new base ancestor.

- **Command:** `git rebase <target-branch>` while on `<curr-branch>`.
- **Intent:** Make `<curr-branch>` rebase onto the tip of `<target-branch>` by replaying the commits from `<curr-branch>` that are missing from `<target-branch>`. Usually done to get a linear history where `<target-branch>` ends up being included in `<curr-branch>` ancestry.

#### Applies cleanly

Example:

```text
A---B---C main
     \
      D---E feature

After:

A---B---C main
         \
          D'---E' feature
```

- Run `git rebase main` while on `feature`.
- Git figures out the commits on `feature` that are missing from `main`: `D` and `E`.
- Git applies the diff-patch introduced by `D` onto the tip of `main`, i.e. `C`.
- If the patch applies cleanly, Git creates a new commit `D'` on top of `C`.
- `D'` contains the same change introduced by `D`, but it is a new commit with a different hash.
- Git then applies the diff-patch introduced by `E` onto `D'`.
- If that patch applies cleanly, Git creates a new commit `E'` on top of `D'`.
- `E'` contains the same change introduced by `E`, but it is a new commit with a different hash. This process continues until the tip of the feature branch which in this case is just `E`.
- After the replay succeeds, the `feature` branch pointer moves from `E` to `E'`.
- The `main` branch pointer stays at `C`.
- The original commits `D` and `E` are not moved but are unreachable using a branch reference.
- Hence, `main` has become part of the linear history of `feature`.

**Why no Conflict?**

In this case, each diff-patch introduced by the commits on `feature` that were missing from `main` fits cleanly onto the new base.

File-content example for replaying `D`:

```text
B = parent of D:
line 1: title
line 2: description

D = first feature commit being replayed:
line 1: title
line 2: better description

C = tip of target branch main:
line 1: better title
line 2: description

D' = rewritten commit created on top of C:
line 1: better title
line 2: better description
```

The diff-patch introduced by `D` is effectively:

```diff
-line 2: description
+line 2: better description
```

When Git applies that patch onto `C`, it can still find `line 2: description`, so the patch applies cleanly. Git keeps the change already present in `C` on line 1 and adds the replayed change from `D` on line 2.

#### Raises conflict

Rebase tries to replay each commit from `feature` missing from `target-branch` at the tip of `main`. If one replayed commit introduces a diff-patch that does not fit cleanly onto the new base, Git pauses at that commit and marks a conflict.

Example:

```text
B = parent of D:
line 1: title
line 2: description

D = first feature commit being replayed:
line 1: title
line 2: better description

C = tip of target branch main:
line 1: title
line 2: alternate description
```

The diff-patch introduced by `D` expects to replace:

```diff
-line 2: description
+line 2: better description
```

But `C` already has `line 2: alternate description`, so Git cannot confidently apply the patch. Git pauses while replaying commit `D`; `D'` has not been created yet.

You need to manually resolve the conflicting changes as discussed in [Resolving Conflicts](#resolving-conflicts).

If you resolve the conflict and continue, Git creates `D'` and then tries to replay the next commit from `feature`, i.e. `E`. A rebase conflict is the same kind of patch-replay problem as a cherry-pick conflict, but rebase has more possible pause points because it replays multiple commits from a branch instead of replaying just one selected commit.

#### Interactive Rebase

Interactive rebase lets you edit the list of commits before Git replays them.

Run `git rebase -i <target-branch>` while on `<curr-branch>`.

Git opens an editor with the commits in `<curr-branch>` missing from `<target-branch>` that are going to be replayed. From there you can choose actions like:

- `pick` - keep the commit as-is.
- `reword` - keep the commit's changes but edit its commit message.
- `squash` - combine the commit into the previous commit.
- `drop` - remove the commit from the replay.

Use interactive rebase when you want to clean up local commit history before sharing it.

#### Update Refs

- **Command:** `git rebase --update-refs <target-branch>` while on `<curr-branch>`.
- **Intent:** Rebase `<curr-branch>` onto `<target-branch>` and also update any related local branch pointers that point directly to one of the commits being replayed.

Example:

```text
Before:

A---B---C main
     \
      D---E feature
           \
            F topic

After:

A---B---C main
         \
          D'---E' feature
                \
                 F' topic
```

- Run `git rebase --update-refs main` while on `topic`.
- Git rebases `topic` onto `main`, so it rewrites `D`, `E`, and `F` as `D'`, `E'`, and `F'`. Since `feature` pointed to one of the rewritten commits, Git also updates `feature` from `E` to `E'`; `topic` moves from `F` to `F'`, and `main` stays at `C`.

## Resolving Conflicts

A conflict means Git cannot automatically produce one final file from the versions involved in the current operation. No matter whether the conflict came from a merge, squash merge, cherry-pick, or rebase, the handling pattern is the same:

1. Inspect which files are conflicted.
2. Decide what final content each conflicted file should contain.
3. Stage the resolved files.
4. Continue or finish the original Git operation.

When Git marks a file as conflicted, it writes conflict markers around the unresolved section:

```text
 <<<<<<< HEAD
 line 1: better title
 =======
 line 1: alternate title
 >>>>>>> incoming-branch
 line 2: description
```

You then manually choose or rewrite the final version as described below.

### `ours` and `theirs`

The meaning of `ours` and `theirs` depends on the operation:

| Operation | `ours` means | `theirs` means |
|-----------|--------------|----------------|
| merge | current branch | branch being merged in |
| squash merge | current branch | branch being squashed in |
| cherry-pick | current branch | commit being cherry-picked |
| rebase | target branch plus commits already replayed | commit currently being replayed |

During a rebase conflict, `ours` can feel backwards because Git has temporarily moved onto the target branch and is replaying your branch's commits onto it.

### How to handle conflicts

1. If you already know which side should win for conflicting regions, you can start the operation with a strategy option:

   ```bash
   git merge -X ours other-branch
   git merge -X theirs other-branch
   git merge --squash -X ours other-branch
   git cherry-pick -X theirs <commit>
   git rebase -X theirs main
   ```

   `-X ours` and `-X theirs` tell Git how to auto-resolve conflicting regions when possible. The meaning of `ours` and `theirs` follows the table above, so be especially careful with rebase.

2. If Git reports a conflict, run `git status` to see which files are conflicted. For each file you can:

   - **Take our version**:

     ```bash
     git restore --ours <file>
     git add <file>
     ```

   - **Take their version**:

     ```bash
     git restore --theirs <file>
     git add <file>
     ```

   - **Resolve manually** (next step) if you need to decide for each section of conflict in a file.

3. To resolve manually: open each conflicted file in VS Code (or your editor). You’ll see conflict markers:

   ```
   <<<<<<< HEAD
   our version
   =======
   their version
   >>>>>>> other-branch
   ```

   Edit the file to remove the markers (`<<<<<<<`, `=======`, `>>>>>>>`) and keep whatever content you want.

4. Once you are sure that you handled the conflict in each file that appeared in the `git status` after the merge, rebase, squash, or cherry-pick, you can stage the resolved files: `git add <file>` (or `git add .` if all are resolved). But be aware that even if the conflict is not resolved correctly, Git is not going to alert you and can commit broken code. One easy way to avoid this is to search for the conflict markers using VS Code's find across the entire directory.

5. Finish the resolution:
   - **Merge and Squash conflict:** Stage the resolved files, then run `git commit` (Git will use the default merge message).
   - **Rebase conflict:** Stage the resolved files, then run `git rebase --continue` - do **not** run `git commit`. In a rebase, Git creates the commit for you when you continue; using `git commit` would add an extra unwanted commit.
   - **Cherry-pick conflict:** Stage the resolved files, then run `git cherry-pick --continue`.
   - **Skip current replayed commit:** During a rebase or multi-commit cherry-pick, use `git rebase --skip` or `git cherry-pick --skip` if the current commit is redundant or you no longer want to apply it.

6. In case the conflict is too much to handle, use the matching abort command:

   ```bash
   git merge --abort          # abort a regular merge
   git rebase --abort         # abort a rebase
   git cherry-pick --abort    # abort a cherry-pick
   git reset --merge          # abort a conflicted squash merge
   ```

   For merge, rebase, cherry-pick, and conflicted squash merge, aborting removes the conflict markers and restores files to the state before the operation started. A conflicted squash merge uses `git reset --merge` because `git merge --squash` does not create `MERGE_HEAD`, so `git merge --abort` has no merge state to abort.

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

### 4. `git diff`

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

*Guide covers: one-time setup, repo workflow (fork → clone → stage → commit → push → pull), merge conflicts, current pointer/HEAD behavior, branches, reset, status/log/reflog, and essential commands (stash, restore, rm, diff).*
