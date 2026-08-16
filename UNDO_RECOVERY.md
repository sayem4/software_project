# Undo and Recovery Documentation

This document records a controlled undo/recovery exercise on branch `practice/undo-recovery`.
The goal was to show two complementary Git techniques:

1. **Local recovery with `git reflog`** after an accidental `git reset --hard`
2. **Safe public undo with `git revert`**, which does not rewrite published history

---

## Why this is safe

| Method | Rewrites history? | Safe on a shared branch? | What it is for |
|---|---|---|---|
| `git reset --hard` | Yes (moves the branch pointer) | No, if the commit was already pushed | Local cleanup only |
| `git reflog` | No | Yes (read-only) | Finding a "lost" commit SHA |
| `git revert` | No (adds a new commit) | Yes | Undoing a published mistake |

`git reset --hard` was used only on a local practice branch. The published undo is a revert commit, so teammates who already pulled the accidental commit can pull the revert and stay in sync.

---

## Step 1 — Accidental commit

A dummy file `ACCIDENTAL_CHANGE.txt` was committed on purpose.

```
git checkout -b practice/undo-recovery
git add ACCIDENTAL_CHANGE.txt
git commit -m "WIP: accidental file added to demonstrate undo"
```

**Commit:** `261005b` — *WIP: accidental file added to demonstrate undo*

---

## Step 2 — Accidental hard reset

```
git reset --hard HEAD~1
```

Result:

- `HEAD` moved back to `67f4658`
- `ACCIDENTAL_CHANGE.txt` disappeared from the working tree
- The commit was **not deleted**. Git still had it; only the branch pointer moved.

---

## Step 3 — Recover the commit from reflog

```
git reflog
git reset --hard 261005b
```

Reflog excerpt immediately after the accidental reset:

```
67f4658 HEAD@{0}: reset: moving to HEAD~1
261005b HEAD@{1}: commit: WIP: accidental file added to demonstrate undo
67f4658 HEAD@{2}: checkout: moving from main to practice/undo-recovery
```

`HEAD@{1}` still pointed at `261005b`, so the commit was recovered with:

```
git reset --hard 261005b
```

The dummy file returned. Recovery is possible because reflog keeps a local diary of every HEAD movement.

---

## Step 4 — Safe undo with revert

Resetting again would have been unsafe if the accidental commit had already been pushed. Instead:

```
git revert HEAD --no-edit
```

**Commit:** `8712897` — *Revert "WIP: accidental file added to demonstrate undo"*

This created a new commit that deleted `ACCIDENTAL_CHANGE.txt`. History is still linear and honest:

```
8712897 Revert "WIP: accidental file added to demonstrate undo"
261005b WIP: accidental file added to demonstrate undo
67f4658 Fix GitHub Pages black screen by serving the compiled app
```

---

## Final reflog snippet

```
8712897 HEAD@{0}: revert: Revert "WIP: accidental file added to demonstrate undo"
261005b HEAD@{1}: reset: moving to 261005b7206bbb8c812ed2b1a267d14de95fe799
67f4658 HEAD@{2}: reset: moving to HEAD~1
261005b HEAD@{3}: commit: WIP: accidental file added to demonstrate undo
67f4658 HEAD@{4}: checkout: moving from main to practice/undo-recovery
```

---

## Key hashes used in this exercise

| Hash | Meaning |
|---|---|
| `67f4658` | Known-good main tip before the exercise |
| `261005b` | Accidental commit (recovered via reflog) |
| `8712897` | Revert commit (safe public undo) |

---

## Commands used (copy-paste)

```bash
git checkout -b practice/undo-recovery
git add ACCIDENTAL_CHANGE.txt
git commit -m "WIP: accidental file added to demonstrate undo"
git reset --hard HEAD~1
git reflog
git reset --hard 261005b
git revert HEAD --no-edit
```
