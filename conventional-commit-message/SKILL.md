---
name: conventional-commit-message
description: Generate a single Chinese Conventional Commits git commit message from the current staged and unstaged working tree changes. Use when the user expresses an intent to commit code or current changes, such as "提交代码", "提交一下", "帮我提交", "commit code", "commit changes", "git commit", "生成提交信息", or asks to create, write, generate, or suggest a git commit message. The workflow stages unstaged and untracked worktree changes before analyzing the staged diff, then outputs only the final commit message.
---

# Conventional Commit Message

## Workflow

1. Confirm the current directory is inside a Git work tree:
   ```bash
   git rev-parse --is-inside-work-tree
   ```
   If it is not a Git work tree, stop and report the problem concisely.

2. Inspect current changes:
   ```bash
   git status --short
   ```
   If there are no staged, unstaged, or untracked changes, stop and report that there are no changes to commit.

3. Stage all worktree changes before generating the message:
   ```bash
   git add -A
   ```
   This is required when unstaged, deleted, renamed, or untracked files exist. Do not run `git commit` unless the user explicitly asks to create the commit.

4. Analyze only the staged changes:
   ```bash
   git diff --cached --stat
   git diff --cached --name-status
   git diff --cached
   ```
   For very large diffs, use the stat and name-status output first, then inspect the most relevant file diffs needed to identify the primary intent.

5. Output exactly one final commit message line and nothing else. Treat natural-language requests like `提交代码`, `提交一下`, or `帮我提交` as requests to produce the commit message unless the user explicitly asks to run `git commit`.

## Message Rules

Use exactly this format:

```text
<type>: <description>
```

- `<type>` must be exactly one of: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, `ci`, `build`
- Include one space after the colon.
- Write `<description>` in concise Chinese.
- Do not end with punctuation.
- Do not include a scope, body, footer, bullet list, markdown, quotes, or explanations.

## Type Selection

- `feat`: 新增功能、页面、接口、用户可见能力
- `fix`: 修复缺陷、错误行为、异常、兼容性问题
- `docs`: 仅文档、注释说明、示例文本修改
- `style`: 仅格式、空白、排版、lint 自动格式化，且不影响行为
- `refactor`: 重构实现、拆分结构、调整命名或组织方式，且无功能变化
- `perf`: 性能优化、加载速度、缓存、渲染或查询效率改善
- `test`: 新增、修改或修复测试用例、测试工具、mock
- `chore`: 辅助脚本、维护性任务、配置杂项、仓库清理
- `ci`: CI/CD 工作流、流水线、发布自动化配置
- `build`: 构建系统、打包配置、依赖版本、锁文件修改

When multiple types apply, choose the type that best describes the primary intent. Prefer user-visible intent over incidental file changes.

## Description Guidance

- Use a short verb-object Chinese phrase, such as `新增用户登录页面` or `修复验证码校验失败的问题`.
- Mention the most important changed behavior or artifact, not implementation trivia.
- Keep it specific enough to distinguish the commit from nearby changes.
- Avoid vague descriptions such as `更新代码`, `修改文件`, `调整逻辑`, or `优化项目`.
