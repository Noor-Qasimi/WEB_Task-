# Git Exercise — Answers

This README contains my answers to the six Git questions, with commands and explanations for each scenario.

## Question 1: `git merge` vs. `git rebase`

Both commands integrate changes from one branch into another, but they handle history differently.

### Merge

`git merge` combines the histories of two branches. If the branches have diverged, Git may create a merge commit. This preserves the branch structure and is useful when I want to keep a record of how shared work was integrated.

```bash
git switch main
git merge feature-login
```

### Rebase

`git rebase` reapplies the commits from the current branch on top of another base. This usually creates a more linear history, but the rebased commits get new commit IDs.

```bash
git switch feature-login
git rebase main
```

I would use rebase to update a private feature branch and keep its history tidy. I would avoid rebasing commits that teammates already depend on unless we have coordinated it, because rebase rewrites history. In short, merge preserves branch history and may add a merge commit; rebase replays commits onto a new base.

## Question 2: Reviewing and Staging Selected Changes

A **hunk** is a section of a diff that groups nearby changes. A file can contain multiple hunks, and Git lets me stage only the parts I want.

```bash
git add -p path/to/file
```

Common choices include:

- `y`: stage this hunk.
- `n`: skip this hunk.
- `s`: split the hunk into smaller hunks, if possible.
- `e`: edit the patch manually.
- `q`: quit interactive staging.

For example, if a file contains a bug fix and unrelated formatting changes, I can stage the bug-fix hunk and leave the formatting changes unstaged. I can review the result with:

```bash
git diff --staged
git diff
git status
```

This helps me make a focused commit containing one logical change instead of mixing unrelated edits.

## Question 3: Finding a Bug-Inducing Commit with `git bisect`

`git bisect` uses binary search to identify the commit that introduced a bug. I need one known-bad commit and one known-good commit.

```bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
```

Git checks out an intermediate commit. I test that version and mark the result:

```bash
git bisect good
```

or:

```bash
git bisect bad
```

I repeat this until Git identifies the first bad commit in the search range, then finish and return to the original checkout:

```bash
git bisect reset
```

This is faster than checking every commit manually because each test narrows the range of possible commits.

## Question 4: Inspecting Changes, Committing, and Ignoring Temporary Files

In this scenario, `README.md` is tracked. Some changes were staged, and then further edits were made without staging them. The `tmp/` directory contains only untracked temporary files.

### 4.1 Inspect staged and unstaged changes

```bash
git status
git diff
git diff --staged
```

- `git status` summarizes staged changes, unstaged changes, and untracked files.
- `git diff` shows working-tree changes that have not been staged.
- `git diff --staged` shows changes in the staging area that are ready for the next commit.

The staged and unstaged diffs can show different sets of edits to the same file.

### 4.2 What does a normal commit include?

```bash
git commit -m "Update README"
```

A normal commit includes only the changes already in the staging area. It includes the earlier staged edits, but not the later unstaged edits. Those later edits remain in the working tree.

### 4.3 Ignore the `tmp/` directory

Add this line to the repository's `.gitignore` file:

```gitignore
tmp/
```

This ignores untracked files in that directory. However, `.gitignore` does not stop tracking files Git already tracks. If the files had already been committed, I would remove them from the index while keeping local copies:

```bash
git rm -r --cached tmp/
```

In the given scenario, the files are untracked, so adding `tmp/` to `.gitignore` is sufficient.

## Question 5: Resolving a Merge Conflict

In this scenario, my teammate and I started from the same commit. I worked on `feature-description`, while my teammate worked on `main`. We changed the same line in `README.md` in different ways and committed our changes.

### 5.1 Why did the conflict happen?

Git reports a conflict because both branches changed the same line in incompatible ways, so it cannot automatically decide which version is correct. Not every change to the same file causes a conflict: edits to separate lines or changes that can be combined safely often merge automatically.

### 5.2 Find and understand the conflict

```bash
git status
```

Git lists files that need resolution. The conflicted section may look like this:

```text
<<<<<<< HEAD
Text from the current branch (main)
=======
Text from the incoming feature-description branch
>>>>>>> feature-description
```

The text between `<<<<<<< HEAD` and `=======` is from the current branch (`main` here). The text between `=======` and `>>>>>>> feature-description` is from the incoming branch. These marker lines must be removed when I resolve the file.

### 5.3 Resolve and finish the merge

I would edit the section into the correct final wording, combining both versions if both contain important information. I would save the file and make sure no conflict markers remain. Then:

```bash
git add README.md
git status
git commit
```

The final `git commit` completes the merge if Git has not already completed it automatically.

If I decide to cancel the merge instead, I can run:

```bash
git merge --abort
```

This attempts to restore the working tree and index to their pre-merge state. I should protect unrelated work before starting a merge.

## Question 6: Choosing the Right Way to Undo Changes

### 6.1 Discard unstaged edits to `notes.txt`

If I edited `notes.txt`, have not staged the changes, and want to discard them, I would run:

```bash
git restore notes.txt
```

This restores the working-tree version from the staging area. In this scenario, that version matches the last commit, so the unstaged edits are discarded. I should use this only when I am sure I no longer need those edits.

### 6.2 Unstage changes but keep the edits

If I staged changes to `notes.txt` by mistake but have not committed them, I would run:

```bash
git restore --staged notes.txt
```

This removes the changes from the staging area but keeps the edits in the working tree, so I can continue reviewing or editing them.

### 6.3 Reverse a wrong commit already pushed to shared `main`

If an incorrect ordinary commit has already been pushed and teammates have received it, I would use:

```bash
git revert <commit-hash>
```

This creates a new commit that reverses the changes introduced by the specified commit. The original commit remains in history. I would choose `git revert` because rewriting shared history and force-pushing can disrupt teammates' local histories. A revert records the correction without deleting the original shared commit.

---

## Note on AI Assistance

AI assistance was used to help organize and clarify the explanations and Git commands in this README. I reviewed the answers against the assignment scenarios and am responsible for understanding and explaining the commands and their effects.
