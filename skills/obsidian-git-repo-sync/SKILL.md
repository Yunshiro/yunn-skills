---
name: obsidian-git-repo-sync
description: Sync a local directory to a GitHub remote repository. This skill should be used when the user wants to push/sync/upload their current project directory to a GitHub repository. Triggered by requests like "同步到远程仓库", "推送到GitHub", "上传到仓库", "sync to GitHub", or "push to remote".
agent_created: true
---

# Obsidian Git Repo Sync

## Overview

Initialize a local directory as a git repository and push all contents to a specified GitHub remote. The skill guides the process step by step, starting with an environment check and then collecting the remote URL from the user.

## Workflow

### Step 1: Check Git Installation

Check whether git is installed on the system by running:

```bash
git --version
```

- **If the command succeeds** → proceed to Step 2.
- **If the command fails** (git not found) → tell the user: "Git is not installed on this machine. Please install Git first from https://git-scm.com/ or use your system package manager (e.g., `brew install git` on macOS). Once installed, let me know and I'll continue." **Stop here and wait for the user.**

### Step 2: Ask for the Remote Repository URL

Once git is confirmed installed, ask the user:

> "What is your GitHub remote repository URL? (e.g., `https://github.com/username/repo.git`)"

Wait for the user to provide the URL before proceeding.

### Step 3: Sync the Directory to the Remote

After receiving the URL, execute the following steps using the current workspace directory:

1. **Initialize git** (if not already a repository):

   ```bash
   git init
   ```

   If the directory is already a git repository (`.git` exists), skip this step.

2. **Add the remote** (if not already configured):

   Check existing remotes first:
   ```bash
   git remote -v
   ```

   If `origin` does not exist, add it:
   ```bash
   git remote add origin <REMOTE_URL>
   ```

   If `origin` already exists but points to a different URL, update it:
   ```bash
   git remote set-url origin <REMOTE_URL>
   ```

3. **Stage and commit all files:**

   ```bash
   git add -A
   git commit -m "Initial commit"
   ```

   If there are no changes to commit, skip this step.

4. **Push to remote:**

   ```bash
   git branch -M main
   git push -u origin main
   ```

   If the push fails, check the error message:
   - **Authentication failure**: Tell the user to configure git credentials (e.g., using a Personal Access Token or SSH key) and retry.
   - **Remote not found**: Verify the URL is correct with the user.
   - **Remote has existing commits**: Use `git pull origin main --allow-unrelated-histories --no-rebase` first to merge, then commit and push again.

### Step 4: Report the Result

Tell the user what was pushed:
- The remote URL
- The branch name (main)
- How many files were committed
- A reminder: "Your local branch is now tracking `origin/main`. Future pushes can be done with `git push` directly."

## Notes

- Always use the workspace root directory (`.`) as the working directory for all git commands.
- If there's a `.gitignore` file already present, it will be respected. If not, suggest creating one if there are files the user might not want to sync (e.g., `.obsidian/`, `node_modules/`, `.env`).
