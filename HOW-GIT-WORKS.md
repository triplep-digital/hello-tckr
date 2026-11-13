# How Git Works

## What git is

Git is a distributed version control system. It records the history of a project so you can see what changed, go back to earlier states, and work on several lines of development at once.

"Distributed" means every clone is a full copy of the repository, including its entire history. You can commit, browse history and create branches offline. A server such as GitHub is just another copy that people agree to share.

## Snapshots, not diffs

Many people picture git as storing a list of changes. It doesn't. Each commit records a snapshot of the whole project tree at that moment. Files that did not change are not copied again: the new snapshot simply points at the same stored content as the previous one.

Tools like `git diff` compute differences on demand by comparing two snapshots. (Git also packs objects into compressed "packfiles" for storage, but that is an optimization and does not change the model.)

## The object database

Everything git stores lives in `.git/objects` as objects of four kinds:

- **Blob**: the contents of a single file, with no name.
- **Tree**: a directory listing. Each entry has a name, a mode, and the hash of a blob (file) or another tree (subdirectory).
- **Commit**: points to one top-level tree, zero or more parent commits, an author, a committer, timestamps and a message.
- **Annotated tag**: a named, optionally signed object that points to another object (usually a commit), with a tagger and message.

### Content addressing

Each object is stored under a hash of its content. The default hash is SHA-1; SHA-256 is supported for newer repositories. This has useful consequences:

- Identical content is stored once, however many files or commits use it.
- An object's name is determined by its content, so it can't change silently.
- A commit's hash covers its tree and its parents' hashes, so changing any past commit changes every commit after it. History is tamper-evident.

You can look inside a commit with `git cat-file`:

```bash
git cat-file -p HEAD
```

```text
tree 9f1c2e...
parent 4b825d...
author Ada <ada@example.com> 1731484800 +0000
committer Ada <ada@example.com> 1731484800 +0000

Add greeting
```

Running `git cat-file -p` on the tree hash lists its blobs and subtrees.

## The three areas

Git works with three places where your files can live:

1. **Working tree**: the files you see and edit on disk.
2. **Index** (or staging area): a proposal for your next commit.
3. **Repository**: the committed history in `.git`.

The flow is:

- `git add <file>` copies the current content of a file from the working tree into the index.
- `git commit` turns the index into a new commit in the repository.

Staging lets you build a commit deliberately, including only some of your changes.

Three commands show where you stand:

```bash
git status          # what is modified, staged, or untracked
git diff            # working tree vs. index (unstaged changes)
git diff --staged   # index vs. last commit (what you would commit)
```

## References

Hashes are hard to remember, so git uses references: names that point at commits.

- A **branch** is a movable pointer to a commit. It is stored as a small file such as `.git/refs/heads/main` containing a commit hash.
- **HEAD** says where you are. Normally it points to the current branch (for example `ref: refs/heads/main`).
- When you commit, the new commit's parent is the current commit, and the current branch is moved forward to the new commit. HEAD follows because it points at the branch.
- A **tag** is a name for a commit that normally doesn't move, used for releases like `v1.0`. Lightweight tags are just refs; annotated tags are objects, as above.

Creating a branch is cheap because it only creates a pointer:

```bash
git branch feature      # create a pointer at the current commit
git switch feature      # point HEAD at it
git switch -c other     # create and switch in one step
```

### Detached HEAD

If you check out a commit directly, such as `git switch --detach <hash>` or `git checkout <tag>`, HEAD points to a commit instead of a branch. This is a detached HEAD. You can look around and even commit, but no branch follows those commits, so they can become hard to find once you switch away. To keep them, create a branch: `git switch -c my-branch`.

## History is a graph

Every commit lists its parent commits, so history forms a directed acyclic graph (DAG):

- A normal commit has one parent.
- The first commit has none.
- A merge commit has two or more parents.

Branches are simply different paths through this graph. `git log --graph --oneline --all` draws it.

## Combining work

Suppose `main` and `feature` have both moved on since they diverged.

### Fast-forward

If `main` has not moved since you branched, merging `feature` just moves the `main` pointer forward along the existing line. No new commit is created.

### Merge commit

If both branches have new commits, `git merge feature` (run while on `main`) creates a merge commit with two parents, joining the histories. Existing commits are untouched.

### Rebase

`git rebase main` (run while on `feature`) replays your commits on top of the tip of `main`. The result is a linear history, but the replayed commits are new commits with new hashes; the originals are abandoned. Because of that, don't rebase commits that others have already based work on (shared history).

### Conflicts

If both sides changed the same lines, git can't decide and stops with a conflict. The affected files contain markers:

```text
<<<<<<< HEAD
your version
=======
their version
>>>>>>> feature
```

To resolve:

1. Edit the file to the content you want and remove the markers.
2. `git add <file>` to mark it resolved.
3. Finish with `git commit` (merge) or `git rebase --continue` (rebase).

To back out instead, use `git merge --abort` or `git rebase --abort`.

## Remotes

A remote is another copy of the repository, identified by a name and a URL. `git clone <url>` copies the whole repository and names the source `origin`.

Git keeps **remote-tracking branches** such as `origin/main`. They record where the remote's branches were the last time you communicated with it. You don't commit to them directly; they update when you fetch, pull or push.

- `git fetch` downloads new commits and updates `origin/*`. It doesn't change your branches or working tree.
- `git pull` is `git fetch` followed by a merge (or a rebase, with `git pull --rebase`) into your current branch.
- `git push` uploads your branch's commits and moves the branch on the remote. It is refused if that would lose commits that the remote has and you don't, so fetch and integrate first.

## Undoing things

- `git restore <file>` discards uncommitted changes in a file in the working tree.
- `git restore --staged <file>` unstages a file, keeping your edits.
- `git revert <commit>` creates a new commit that reverses an earlier one. It is safe for shared history.
- `git reset <commit>` moves the current branch to another commit. The mode decides what happens to the index and working tree:
  - `--soft`: only the branch moves; changes stay staged.
  - `--mixed` (default): the branch moves and the index is reset; changes stay in the working tree.
  - `--hard`: the branch, index and working tree all reset; uncommitted work is lost.

### The reflog

Git records every movement of HEAD and branch tips in the reflog. If you reset or rebase and regret it, find the earlier state and return to it:

```bash
git reflog
git reset --hard HEAD@{1}
```

Reflog entries are local and expire eventually (by default after months), but they rescue many mistakes.

## A typical workflow

```bash
git clone https://github.com/example/project.git
cd project
git switch -c fix-typo          # new branch
# ...edit files...
git status
git diff
git add README.md
git commit -m "Fix typo in README"
git fetch origin                # see what changed upstream
git rebase origin/main          # or: git merge origin/main
git push -u origin fix-typo     # publish, then open a pull request
```

## What's inside `.git`

The `.git` directory is the repository. Your working tree is just a checkout of one commit from it.

- `HEAD`: the current branch (or commit, if detached).
- `config`: this repository's settings, including remotes.
- `objects/`: the object database of blobs, trees, commits and tags.
- `refs/`: branches (`refs/heads/`), remote-tracking branches (`refs/remotes/`) and tags (`refs/tags/`).
- `index`: the staging area.

Once you see that a repository is a graph of immutable objects plus a few movable pointers, most git commands become simple operations on that structure.
