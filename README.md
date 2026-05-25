# push-to-github

[English](#english) | [中文](#中文)

---

## English

A Claude Code skill that automates pushing local projects to GitHub. Handles the entire workflow from git initialization to repo creation, including bilingual README generation and repo metadata setup.

### What It Does

1. Gathers repo info (name, visibility, description)
2. Configures v2ray proxy for git
3. Checks GitHub CLI authentication
4. Generates a bilingual (English/Chinese) README.md
5. Initializes git and creates the initial commit
6. Creates the GitHub repo and pushes code
7. Sets the About description and topics via `gh repo edit`
8. Verifies and reports the result

### Installation

Copy the `skill.md` file to your Claude Code skills directory:

```
~/.claude/skills/push-to-github/skill.md
```

### Usage

In Claude Code, say any of:

- "push to github"
- "推送到github"
- "publish to github"
- "上传到github"

Claude Code will trigger this skill and guide you through the process.

---

## 中文

一个 Claude Code 技能，用于自动化将本地项目推送到 GitHub。覆盖从 git 初始化到仓库创建的完整流程，包括双语 README 生成和仓库元数据设置。

### 功能

1. 收集仓库信息（名称、可见性、描述）
2. 配置 v2ray 代理
3. 检查 GitHub CLI 认证状态
4. 生成中英双语 README.md
5. 初始化 git 并创建首次提交
6. 创建 GitHub 仓库并推送代码
7. 通过 `gh repo edit` 设置仓库描述和 Topics
8. 验证并报告结果

### 安装

将 `skill.md` 文件复制到 Claude Code 的 skills 目录：

```
~/.claude/skills/push-to-github/skill.md
```

### 使用

在 Claude Code 中输入以下任意指令：

- "push to github"
- "推送到github"
- "publish to github"
- "上传到github"

Claude Code 会自动触发此技能并引导你完成流程。

---

## License

MIT
