---
name: push-to-github
description: Push a local project to GitHub with auto-generated bilingual README and repo setup. TRIGGER when: user says "push to github", "推送到github", "publish to github", "上传到github", or asks to create a GitHub repo from local code. Also handles git init, proxy config, and bilingual documentation generation.
---

# Push to GitHub

Automate the workflow of pushing a local project to GitHub, including git initialization, proxy configuration, bilingual README generation, and repo creation.

## Workflow

Follow these steps in order. Ask the user for missing information before proceeding.

### Step 1: Gather Information

Ask the user for:
- **Repo name** — default to the current directory name
- **Visibility** — public or private (default: public)
- **Project description** — a one-line description of what the project does

If the project has multiple files, quickly scan them to understand the project purpose. Use this to auto-generate descriptions if the user doesn't provide one.

### Step 2: Configure v2ray Proxy

Before any GitHub operation, configure git proxy for v2ray:

```bash
git config --global http.proxy socks5://127.0.0.1:10808
git config --global https.proxy socks5://127.0.0.1:10808
```

If the user uses a different proxy port, ask them. Common v2ray ports:
- SOCKS5: `10808`
- HTTP: `10809`

### Step 3: Check GitHub CLI Auth

```bash
gh auth status
```

If not logged in, instruct the user to run `! gh auth login` and wait for confirmation.

**Important**: Do NOT use the account name from `gh auth status` as the GitHub owner. The display name may differ from the actual GitHub username. The correct owner will be determined in Step 7 after creating the repo.

### Step 4: Generate Bilingual README.md

Generate a README.md with both English and Chinese sections. Use this template structure:

```markdown
# {project-name}

[English](#english) | [中文](#中文)

---

## English

{English description of the project}

### Supported Formats / Features

{Relevant details in English}

### Installation

```bash
{installation commands}
```

### Usage

{usage instructions in English}

### Example

{example in English}

---

## 中文

{Chinese description of the project}

### 支持格式 / 功能特点

{Relevant details in Chinese}

### 安装

```bash
{installation commands}
```

### 使用

{usage instructions in Chinese}

### 示例

{example in Chinese}

---

## License

{License type, default MIT}
```

Adapt the sections (Supported Formats, Features, etc.) based on what the project actually does. Remove irrelevant sections.

### Step 5: Initialize Git and Commit

```bash
git init
git add .
git commit -m "Initial commit: {brief description}"
```

If there are files that should not be committed (e.g., `.env`, `node_modules`, large binaries), create a `.gitignore` first.

### Step 6: Create Repo and Push

```bash
gh repo create {repo-name} --{public/private} --source=. --push
```

If the repo already exists on GitHub:

```bash
git remote add origin https://github.com/{owner}/{repo}.git
git push -u origin main
```

### Step 7: Determine Repo Owner and Set GitHub About

After the repo is created and pushed, determine the actual owner from the git remote — do NOT use the `gh auth status` account name:

```bash
git remote -v
```

Parse the owner from the remote URL (e.g., `https://github.com/{owner}/{repo}.git`).

Then set the bilingual About description:

```bash
gh repo edit {owner}/{repo} --description "{English description} | {Chinese description}"
```

And add relevant topics:

```bash
gh repo edit {owner}/{repo} --add-topic {topic1},{topic2}
```

Choose topics based on the project's tech stack and purpose (e.g., `markdown`, `converter`, `python`, `document`).

### Step 8: Verify and Report

After pushing:
1. Confirm the push succeeded
2. Show the user the repo URL
3. Confirm the About description and topics are set

## Error Handling

- **Network failure**: Remind the user to check v2ray is running. Offer to retry.
- **Repo already exists**: Ask if the user wants to overwrite or add a remote to the existing repo.
- **Auth failure**: Guide the user through `gh auth login`.
- **Push rejected**: Check if remote has content, offer force push or merge.

## Notes

- Always use `main` as the default branch name.
- Prefer `gh` CLI over manual GitHub API calls.
- The proxy config is global (`--global`), so it applies to all git operations.
- If the project has a `requirements.txt` or `package.json`, mention key dependencies in the README.
