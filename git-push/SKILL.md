---
name: git-push
description: Safely push local Git commits to the remote repository after checking whether the upstream branch has unpulled commits. Use when the user asks to push code, upload code, submit code, sync commits, push local commits, or says phrases like "推送代码", "上传代码", "提交到仓库", "同步到远端", "push code", or "git push". The workflow fetches remote state, rebases local commits onto upstream when needed, resolves conflicts that are semantically clear, asks the user to choose among concrete resolution options when intent is ambiguous, and then pushes the local commits.
---

# Git Push

## Workflow

1. Confirm the current directory is inside a Git work tree:
   ```bash
   git rev-parse --is-inside-work-tree
   ```
   If it is not a Git work tree, stop and report the problem concisely.

2. Confirm there is no unfinished Git operation:
   ```bash
   git status --short
   ```
   If a rebase, merge, cherry-pick, or revert is already in progress, inspect `git status` and continue only when the next required Git action is unambiguous. Otherwise stop and report the blocking state.

3. Require a clean worktree before rebasing and pushing:
   ```bash
   git status --short
   ```
   If unstaged, staged, or untracked changes exist, stop and tell the user to commit or stash them first. Do not auto-commit, auto-stash, or discard worktree changes.

4. Resolve the current branch and upstream:
   ```bash
   git branch --show-current
   git rev-parse --abbrev-ref --symbolic-full-name @{u}
   ```
   If there is no upstream branch, stop and ask the user to set an upstream or specify the remote branch. Do not guess a remote branch.

5. Fetch remote state:
   ```bash
   git fetch --prune
   ```

6. Compare local branch and upstream:
   ```bash
   git rev-list --left-right --count @{u}...HEAD
   ```
   Interpret output as `<behind> <ahead>`.

7. Decide the next Git action:
   - If `<behind>` is `0`, do not pull. Push local commits directly.
   - If `<behind>` is greater than `0`, rebase local commits onto upstream:
     ```bash
     git pull --rebase
     ```
   - If `<ahead>` is `0` after the rebase or before push, report that there are no local commits to push.

8. Handle rebase conflicts by intent:
   - Inspect conflict files with:
     ```bash
     git status --short
     git diff --name-only --diff-filter=U
     ```
   - Read the conflict hunks and surrounding code to understand both sides' intent.
   - If the correct resolution is clear from code context and preserves both compatible intents, resolve it directly.
   - If the conflict is generated or mechanical and the repository has an obvious normal regeneration command, tell the user the command to run instead of running project build, compile, test, or syntax-check commands unless explicitly allowed.
   - If multiple valid resolutions exist, stop and ask the user to choose. Present 2-4 concrete options with short tradeoffs, such as keeping local behavior, keeping remote behavior, combining both paths, or manually editing a named file.
   - Do not choose one side blindly, delete code to make conflicts disappear, or invent business logic that is not supported by surrounding code.
   - After the user chooses or after a clear direct resolution, stage resolved files and continue:
     ```bash
     git add <resolved-files>
     git rebase --continue
     ```
   - Do not run `git rebase --abort` unless the user asks.

9. Push local commits:
   ```bash
   git push
   ```
   If the push is rejected because the remote advanced again, repeat from the fetch/compare step once. If it fails again, stop and report the reason.

10. Report the final result concisely, including whether a rebase happened and whether the push succeeded.

## Safety Rules

- Never run destructive commands such as `git reset --hard`, `git checkout -- <file>`, `git clean`, or `git rebase --abort` unless explicitly requested.
- Never force push unless the user explicitly asks and confirms the exact branch.
- Never create commits in this workflow. If local changes are uncommitted, stop and ask the user to commit or stash them first.
- Prefer correctness over completion. When conflict resolution is not objectively clear, explain the viable choices and wait for the user's decision.
- Do not run project builds, compilers, tests, or syntax checks unless the user explicitly requests them. If validation would normally be useful, tell the user the command to run.

## Trigger Examples

- `推送代码`
- `上传代码到仓库`
- `把本地提交同步到远端`
- `帮我 push 一下`
- `git push`
- `push current commits`
