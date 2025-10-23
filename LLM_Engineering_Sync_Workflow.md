# 🧭 LLM Engineering Sync Workflow  
### Keeping `MAIN` and `PRODUCTION` in Sync with Upstream

This document explains how to keep repositories synchronized:

```
┌────────────────────────┐
│ ed-donner/llm_engineering │  ← Source repository (upstream)
└──────────────┬─────────┘
               │
               ▼
┌────────────────────────┐
│ joshphillipssr/llm_engineering │  ← Personal fork (MAIN)
└──────────────┬─────────┘
               │
               ▼
┌────────────────────────┐
│ Local clone: PRODUCTION │  ← Personal working repo with edits
└────────────────────────┘
```

---

## 🗂 Folder Layout
```
/Users/josh/Projects/llm_engineering/
├── main/         ← clone of personal fork (joshphillipssr/llm_engineering)
└── production/   ← clone of main for personal edits
```

---

## ⚙️ 1. Sync MAIN with ed-donner’s Source Repo

### Step 1. Open the `main` folder
```bash
cd /Users/josh/Projects/llm_engineering/main
```

### Step 2. Add the original repo as “upstream” (only once)
```bash
git remote add upstream https://github.com/ed-donner/llm_engineering.git
```

Check remotes:
```bash
git remote -v
```
Expected:
```
origin    https://github.com/joshphillipssr/llm_engineering.git (fetch)
upstream  https://github.com/ed-donner/llm_engineering.git (fetch)
```

### Step 3. Fetch and merge upstream changes
```bash
git fetch upstream
git switch main
git merge --ff-only upstream/main
```

If Git reports conflicts, remove `--ff-only`, resolve them, then:
```bash
git add .
git commit -m "Merge upstream/main into main"
```

### Step 4. Push updated MAIN to personal fork
```bash
git push origin main
```

✅ Personal fork (`joshphillipssr/llm_engineering`) is now fully up to date with `ed-donner/llm_engineering`.

---

## 🧩 2. Merge MAIN into PRODUCTION

### Step 1. Open the `production` folder
```bash
cd /Users/josh/Projects/llm_engineering/production
```

### Step 2. Confirm you’re on the correct branch
```bash
git switch production
```

### Step 3. Fetch the latest MAIN (from your GitHub fork)
```bash
git fetch origin
```

### Step 4. Merge MAIN into PRODUCTION
```bash
git merge origin/main
```

If there are **no conflicts**, skip ahead to Step 6.  
If there **are conflicts**, see Section 3 below.

---

## 🧮 3. Resolving Conflicts (especially `.ipynb` files)

### Option A — Easy way (accept all MAIN changes)
```bash
git checkout --theirs .
git add .
git commit -m "Take all MAIN updates into PRODUCTION"
```

### Option B — Use `nbdime` for notebook merges (recommended)

#### 1. Install via pipx (one-time)
```bash
brew install pipx
pipx install nbdime
pipx ensurepath
export PATH="$HOME/.local/bin:$PATH"
```

#### 2. Enable nbdime for Git
```bash
nbdime config-git --enable
git config --global merge.tool nbdime
git config --global mergetool.prompt false
```

#### 3. Launch the merge editor
```bash
git mergetool
```
A browser window opens for each conflicted notebook.  
Use:
- **Use Local** → keep PRODUCTION version  
- **Use Remote** → take MAIN version  

After resolving all conflicts:
```bash
git add -A
git commit -m "Resolve .ipynb merge conflicts via nbdime"
```

---

## 🚀 4. Push Final PRODUCTION Updates
```bash
git push origin production
```

✅ `production` now contains all upstream changes from `ed-donner` via `main`, plus your local edits.

---

## 🧹 5. Post-Merge Cleanup

Remove any nbdime backup files (optional but tidy):
```bash
find . -name "*_BACKUP_*" -delete
echo '*_BACKUP_*' >> .gitignore
git add .gitignore
git commit -m "Ignore nbdime backup files"
git push origin production
```

---

## 🔄 Full Workflow Summary

```bash
# --- MAIN ---
cd ~/Projects/llm_engineering/main
git fetch upstream
git merge --ff-only upstream/main
git push origin main

# --- PRODUCTION ---
cd ~/Projects/llm_engineering/production
git fetch origin
git merge origin/main
git mergetool    # if needed (nbdime)
git add -A
git commit -m "Merge main into production"
git push origin production
```

---

## ✅ Quick Tips

- Always use `git merge origin/main` (not `git merge origin main`)
- For notebooks, prefer **nbdime** over VS Code merge editor.
- If conflicts are overwhelming, reset PRODUCTION to MAIN:
  ```bash
  git reset --hard origin/main
  git push -f origin production
  ```
- Configure `.gitattributes` for nbdime integration:
  ```bash
  echo "*.ipynb merge=nbdime diff=nbdime" >> .gitattributes
  git add .gitattributes
  git commit -m "Configure nbdime for .ipynb merges"
  git push origin production
  ```

---

**Last Updated:** October 2025  
**Maintainer:** Josh Phillips (`@joshphillipssr`)
