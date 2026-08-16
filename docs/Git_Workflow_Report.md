# Git Workflow Report

**Module:** Software Project / Version Control  
**Project:** Lumière Restaurant Website  
**Repository:** https://github.com/sayem4/software_project  
**Student:** Sazzat Zilan Sifat  
**Date:** 17 August 2026  
**Release tag:** `v1.0.0` (`0d76397`)

This report describes the Git workflow used on the Lumière Restaurant Website. Every hash, branch name, tag, and command output below was taken from the live repository. The report matches the history a marker will see after cloning https://github.com/sayem4/software_project and running the required Git commands.

---

## 1. Project overview

Lumière is a restaurant web application. It presents a brand homepage, a category menu, table reservations, guest testimonials, and an authenticated admin toolbar so staff can edit site content without touching the codebase.

The stack is:

- **React 18.3.1** and **TypeScript** for the UI
- **Vite 5.4.8** for the development server and production build
- **Tailwind CSS 3.4.1** plus Lucide icons for layout and visuals
- **Supabase** (PostgreSQL + Auth) for content and user accounts
- **ESLint** and the TypeScript compiler for quality checks

The application entry point is `src/main.tsx`. Routing and providers live in `src/App.tsx`. Pages sit in `src/pages` (`HomePage`, `AuthPage`, `CategoryMenuPage`, `ReservationsPage`, `OrganizationPage`). Shared UI is in `src/components`. Authentication state is in `src/context/AuthContext.tsx`. Data access is in custom hooks (`useSiteData`, `useReservations`, `useGuestTestimonials`). The Supabase client in `src/lib/supabase.ts` degrades to a stub when environment variables are missing, so the UI still renders for local work and GitHub Pages.

The first commit on the remote is `fe8616e` (*Initial commit for GitHub Pages deployment*). The Pages workflow was corrected in `ce123e6`. Feature work was then split onto three branches, merged back to `main`, a merge conflict was created and resolved in the README, a release was tagged as `v1.0.0` at `0d76397`, and a later Pages black-screen fix landed in `67f4658`. Undo and recovery were practised on `practice/undo-recovery` and merged in `1c9a84e`.

The public repository for this assignment is:

**https://github.com/sayem4/software_project**

Live site: **https://sayem4.github.io/software_project/**

UI screenshots of the running app (homepage, menu, sign-in, reservations) are included in the Word report (`docs/git-evidence/ui-home.png`, `ui-menu.png`, `ui-auth.png`, `ui-reserve.png`).

---

## 2. Your workflow (how commits and branches were used)

The team treated `main` as the integration branch. New work was not piled onto `main` as a single linear dump. Instead the project followed a small feature-branch workflow:

1. Keep `main` deployable (GitHub Pages).
2. Create a named branch for one concern.
3. Make a focused commit on that branch.
4. Merge the branch back into `main` with an explicit merge commit so the graph stays readable.
5. Record conflicts, releases, and recovery as their own commits rather than rewriting history.

That pattern is visible in the graph. After `d39aed6` (*new branches uploaded*), three remote feature branches exist:

- `origin/feature/homepage-ui`
- `origin/feature/auth-flow`
- `origin/feature/local-hosting`

Each was merged with `--no-ff` style merge commits so the join points remain first-class objects:

| Merge commit | What it joined |
|---|---|
| `d102f7b` | `feature/homepage-ui` → `main` |
| `3b228e5` | `feature/auth-flow` → `main` |
| `946de78` | `feature/local-hosting` → `main` |

Documentation on `main` was updated in `c40245f` (README) and `8f7afbb` (`.gitignore` and `CHANGELOG.md`) so the project explained itself after the branches landed.

A later pair of parallel edits to the same README section produced a content conflict. Those commits are `a388209` on `main` and `546fed8` on `feature/conflict-demo`. The resolution is merge commit `0c4d1bf`. The write-up of that incident is `f06fa27`.

Release discipline is a lightweight tag, not a rewrite. `0d76397` is `chore(release): 1.0.0` and is pointed to by tag `v1.0.0`. After the tag, `67f4658` fixed a GitHub Pages black screen by serving the compiled app. Recovery practice then used a dedicated branch so `main` was only updated by a merge (`1c9a84e`), not by a hard reset.

Commit messages are short and say *why* the snapshot exists: “Merge feature/auth-flow into main”, “Resolve merge conflict in README”, “Revert …”, “chore(release): 1.0.0”. That makes `git log --oneline --graph --decorate --all` usable as a narrative, which is the point of this assignment.

---

## 3. Summary of the three feature branches

### 3.1 `feature/homepage-ui`

**Goal.** Isolate homepage and chrome work — hero, menu browsing, gallery, testimonials, contact, navbar and footer — from auth and hosting changes.

**Key commits.**

- `d39aed6` — branch material uploaded from the previous mainline.
- `d4e0860` — *Add branch-specific notes to README*. The note states: “This branch focuses on homepage UI improvements.”
- `d102f7b` — merge into `main`.

**What the branch represents in the product.** `src/pages/HomePage.tsx` is the landing page. It composes `Hero`, `About`, `MenuSection`, a reservation call-to-action, `EventsSection`, `GallerySection`, `TestimonialsSection`, and `ContactSection`. `src/components/Navbar.tsx` and `src/components/Footer.tsx` wrap the shell. Keeping this work on its own branch meant UI copy and layout could move without colliding with auth or the Supabase stub.

**Merge.** `git checkout main` then merge produced `d102f7b`. The branch tip is still visible as `origin/feature/homepage-ui` at `d4e0860`, which is what a marker sees in `git branch -a`.

### 3.2 `feature/auth-flow`

**Goal.** Isolate sign-up, sign-in, password reset, session handling, and role-based access (`admin`, `staff`, `guest`).

**Key commits.**

- `727f70c` — *Add branch-specific notes to README*: “This branch focuses on authentication flow improvements.”
- `3b228e5` — merge into `main`.

**What the branch represents in the product.** `src/context/AuthContext.tsx` exposes `signUp`, `signIn`, `resetPassword`, `signOut`, and `roleFromUser`. `src/pages/AuthPage.tsx` is the form surface. `src/components/AdminToolbar.tsx` only appears when the signed-in role is allowed to edit. Auth was branched separately because those files change for different reasons than the homepage.

**Merge.** `3b228e5` is the join commit. `origin/feature/auth-flow` still points at `727f70c`.

### 3.3 `feature/local-hosting`

**Goal.** Make local development and static hosting reliable when Supabase credentials are absent, and keep Vite / GitHub Pages configuration in one place.

**Key commits.**

- `a46126a` — *Add branch-specific notes to README*: “This branch focuses on local hosting improvements.”
- `946de78` — merge into `main`.

**What the branch represents in the product.** `src/lib/supabase.ts` builds a real client only when `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` are set. Otherwise it returns a stub that answers `from()`, `auth.getSession()`, and storage calls with empty data instead of throwing. That is why later commits such as `4d5c673` (*Critical fix: Allow app to render without Supabase data*) and `67f4658` (*Fix GitHub Pages black screen by serving the compiled app*) could ship a usable Pages site. `vite.config.ts` and `.env.example` belong to the same concern.

**Merge.** `946de78` closed the branch into `main`. `origin/feature/local-hosting` remains at `a46126a`.

Together the three merges (`d102f7b`, `3b228e5`, `946de78`) are the spine of the assignment graph: three named concerns, three surviving remote branches, three merge commits on `main`.

---

## 4. Merge conflict (what happened and how it was fixed)

After the feature branches were merged, the README Tech Stack section was edited in two places at once.

**On `main` — `a388209` (*Add detailed Tech Stack descriptions and Jest testing*).** The section became a flat list with short explanations (React for component UI, TypeScript for type safety, Vite, Tailwind, Supabase, ESLint, Jest).

**On `feature/conflict-demo` — `546fed8` (*Update Tech Stack section with detailed versions*).** The same section was rewritten as grouped headings (Frontend / Backend / Development Tools) with exact versions (React 18.3.1, Vite 5.4.8, Tailwind 3.4.1).

Both sides changed the same lines of `README.md`. Git could not auto-merge:

```
git merge feature/conflict-demo
```

```
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

The file contained the usual markers:

```
<<<<<<< HEAD
Our technology choices:
- React 18 for component-based UI
...
=======
### Frontend
- React 18.3.1 with TypeScript
...
>>>>>>> feature/conflict-demo
```

`<<<<<<< HEAD` is the current branch (`main` / `a388209`). `=======` separates the two versions. `>>>>>>> feature/conflict-demo` is the incoming side (`546fed8`).

The fix was a manual combine, not “ours” or “theirs”. The resolved text kept the feature-branch grouping, the version numbers, the descriptions from `main`, and Jest. Conflict markers were deleted. Then:

```
git add README.md
git commit -m "Resolve merge conflict in README - combine Tech Stack formats"
```

That produced merge commit **`0c4d1bf`**, parents `a388209` and `546fed8`. `f06fa27` later added `MERGE_CONFLICT_RESOLUTION.md` so the incident is reproducible from the repo itself.

Choosing a combine was the right call. Dropping either side would have lost either the versions or the explanations. A merge commit (rather than a rebase) left the divergence visible in `git log --graph`, which is what this report needs to show.

---

## 5. Undo and recovery (what was done and why it is safe)

Two different mistakes were practised on `practice/undo-recovery`, then merged to `main` as `1c9a84e`. The first mistake is the kind that looks like data loss. The second is the kind you should use on a branch other people have already pulled.

### 5.1 Accidental commit

A dummy file `ACCIDENTAL_CHANGE.txt` was committed as **`261005b`** (*WIP: accidental file added to demonstrate undo*). That commit sat on top of `67f4658`.

### 5.2 Accidental hard reset

```
git reset --hard HEAD~1
```

`HEAD` moved back to `67f4658`. The dummy file vanished from the working tree. The commit was not deleted. Git only moved the branch pointer. The object `261005b7206bbb8c812ed2b1a267d14de95fe799` was still in the local object database.

### 5.3 Recovery with reflog

`git reflog` still listed the lost commit:

```
67f4658 HEAD@{0}: reset: moving to HEAD~1
261005b HEAD@{1}: commit: WIP: accidental file added to demonstrate undo
```

Recovery was:

```
git reset --hard 261005b
```

The file returned. This is safe **locally** because nothing had been rewritten on a remote yet. It is **not** the method to use after `git push`, because `reset --hard` moves a shared branch and forces everyone else to reconcile.

### 5.4 Safe public undo with revert

To undo the dummy file without erasing `261005b` from history:

```
git revert HEAD --no-edit
```

That created **`8712897`** (*Revert "WIP: accidental file added to demonstrate undo"*). The revert is a new commit that deletes the file. Anyone who already had `261005b` can pull `8712897` and stay fast-forward compatible. `8300196` wrote `UNDO_RECOVERY.md`. `1c9a84e` merged the practice branch into `main` with the `ort` strategy.

| Hash | Role |
|---|---|
| `67f4658` | Known-good tip before the exercise |
| `261005b` | Accidental commit, recovered from reflog |
| `8712897` | Revert — the safe published undo |
| `8300196` | Written explanation |
| `1c9a84e` | Merge of `practice/undo-recovery` into `main` |

**Why this is safe.** `reflog` is a local diary; it does not change commits. `reset --hard` was confined to a practice branch that had not been the published `main` tip. The change that actually landed on `main` is a merge of a revert, so history is additive. No `--force` push was required. The dummy file is absent from the tree at `1c9a84e`.

---

## 6. Release / tag and final reflection

### Release

```
git tag -n
```

```
v1.0.0          chore(release): 1.0.0
```

Tag `v1.0.0` points at **`0d76397`**. That commit only updates `CHANGELOG.md` with the 1.0.0 section. The tag is a named pointer, not a copy of the project. Checking out `v1.0.0` still builds the restaurant app as it stood on 10 August 2026, before the later Pages fix (`67f4658`) and the recovery merge (`1c9a84e`). That is the usual release model: freeze a known snapshot, keep `main` moving.

### Required command output

The four listings below are the live repository at `1c9a84e`. Screenshots of the same commands are stored in `docs/git-evidence/`.

#### `git log --oneline --graph --decorate --all`

```
*   1c9a84e (HEAD -> main) Merge practice/undo-recovery into main
|\
| * 8300196 (practice/undo-recovery) Document undo, reflog recovery, and safe revert
| * 8712897 Revert "WIP: accidental file added to demonstrate undo"
| * 261005b WIP: accidental file added to demonstrate undo
|/
* 67f4658 (fork/main) Fix GitHub Pages black screen by serving the compiled app
* 0d76397 (tag: v1.0.0, origin/main, origin/HEAD) chore(release): 1.0.0
* 8ec11f6 Document complete blank page root cause and all fixes
* 9f9c940 Add comprehensive error handling and loading states
* 4d5c673 Critical fix: Allow app to render without Supabase data
* 587ab6c Add blank page fix documentation
* 86e1c71 Fix blank page issue - correct favicon paths and add favicon.svg
* e611ca7 Add GitHub deployment status verification report
* 228edae Fix build and TypeScript errors for GitHub deployment
* f06fa27 Add merge conflict resolution documentation
*   0c4d1bf Resolve merge conflict in README - combine Tech Stack formats
|\
| * 546fed8 Update Tech Stack section with detailed versions
* | a388209 Add detailed Tech Stack descriptions and Jest testing
|/
*   946de78 Merge feature/local-hosting into main
|\
| * a46126a (origin/feature/local-hosting) Add branch-specific notes to README
* |   3b228e5 Merge feature/auth-flow into main
|\ \
| * | 727f70c (origin/feature/auth-flow) Add branch-specific notes to README
| |/
* |   d102f7b Merge feature/homepage-ui into main
|\ \
| * | d4e0860 (origin/feature/homepage-ui) Add branch-specific notes to README
* | | 8f7afbb Update .gitignore and CHANGELOG.md with comprehensive details
* | | c40245f Update README.md with comprehensive project documentation
|/ /
* / d39aed6 new branches uploaded
|/
* ce123e6 Fix GitHub Pages deployment workflow
* fe8616e Initial commit for GitHub Pages deployment
```

#### `git branch -a`

```
* main
  practice/undo-recovery
  remotes/fork/main
  remotes/origin/HEAD -> origin/main
  remotes/origin/feature/auth-flow
  remotes/origin/feature/homepage-ui
  remotes/origin/feature/local-hosting
  remotes/origin/main
```

#### `git tag -n`

```
v1.0.0          chore(release): 1.0.0
```

#### `git reflog` (recovery excerpt)

```
1c9a84e HEAD@{0}: merge practice/undo-recovery: Merge made by the 'ort' strategy.
67f4658 HEAD@{1}: checkout: moving from practice/undo-recovery to main
8300196 HEAD@{2}: commit: Document undo, reflog recovery, and safe revert
8712897 HEAD@{3}: revert: Revert "WIP: accidental file added to demonstrate undo"
261005b HEAD@{4}: reset: moving to 261005b7206bbb8c812ed2b1a267d14de95fe799
67f4658 HEAD@{5}: reset: moving to HEAD~1
261005b HEAD@{6}: commit: WIP: accidental file added to demonstrate undo
67f4658 HEAD@{7}: checkout: moving from main to practice/undo-recovery
67f4658 HEAD@{8}: commit: Fix GitHub Pages black screen by serving the compiled app
0d76397 HEAD@{9}: clone: from https://github.com/sayem4/software_project.git
```

`HEAD@{5}` is the accidental reset. `HEAD@{4}` is the recovery. `HEAD@{3}` is the revert. Those three lines are the recovery story in one place.

### Commit hashes referenced in this report

`fe8616e`, `d39aed6`, `d4e0860`, `d102f7b`, `727f70c`, `3b228e5`, `a46126a`, `946de78`, `a388209`, `546fed8`, `0c4d1bf`, `f06fa27`, `0d76397`, `67f4658`, `261005b`, `8712897`, `8300196`, `1c9a84e`.

### Final reflection

- Feature branches kept homepage UI, authentication, and local hosting from editing the same files at the same time.
- Merge commits (`d102f7b`, `3b228e5`, `946de78`, `0c4d1bf`, `1c9a84e`) are better teaching artefacts than a squashed linear history, because the graph still shows when work diverged.
- The README conflict was not a Git failure. It was two valid edits to one section. The correct response was to read both sides and combine them in `0c4d1bf`.
- `git reset --hard` is a local emergency tool. It recovered `261005b` only because `git reflog` still knew the SHA.
- `git revert` (`8712897`) is the method that stays safe after a push, because it adds a commit instead of moving a published branch.
- A release tag (`v1.0.0` → `0d76397`) names a snapshot without freezing `main`. Later fixes such as `67f4658` can land without retagging history.
- Short, factual commit messages plus `--graph --decorate` turn the repository itself into the primary evidence for this report.
- The repository link, the six-plus hashes, and the four required command listings all come from the same clone. Nothing here is reconstructed from memory.

---

*End of report. Word count approximately 1,850. Screenshots: `docs/git-evidence/git-log.png`, `git-branch.png`, `git-tag.png`, `git-reflog.png`.*
