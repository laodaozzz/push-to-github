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

### Step 3: Check GitHub CLI Auth and Verify Username

```bash
gh auth status
```

If not logged in, instruct the user to run `! gh auth login` and wait for confirmation.

Then get the **actual GitHub username** from the API (the keyring label may have typos):

```bash
gh api user --jq '.login'
```

Store this as `{owner}` — it is the source of truth for all subsequent steps.

Also verify the git global config matches:

```bash
git config --global user.name
git config --global user.email
```

If they are placeholders or don't match, update them:

```bash
git config --global user.name "{owner}"
git config --global user.email "{owner}@users.noreply.github.com"
```

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

### Step 5: Scan for Sensitive Information

Before committing, scan all staged files for secrets and private data. Run these checks in parallel:

**Check 1: Pattern scan** — grep for common secret patterns:

```bash
grep -rniE '(api[_-]?key|api[_-]?secret|access[_-]?token|secret[_-]?key|private[_-]?key|auth[_-]?token|password|passwd|bearer\s+[a-zA-Z0-9_\-\.]+|sk-[a-zA-Z0-9]{20,}|ghp_[a-zA-Z0-9]{36}|gho_[a-zA-Z0-9]{36}|AKIA[0-9A-Z]{16})' --include='*.*' .
```

**Check 2: Sensitive files** — look for files that should not be committed:

```bash
ls -a .env .env.* *.pem *.key *.p12 *.pfx credentials.json service-account.json 2>/dev/null
```

**If secrets found:**
1. Report each finding to the user (file path and matched pattern)
2. Add the offending files/patterns to `.gitignore`
3. Ask the user to confirm whether to proceed, skip those files, or abort
4. If secrets are in tracked content (not just files), ask the user to remove them before proceeding

**If .env or credential files exist but contain no real secrets (e.g., `.env.example` with placeholders):**
- Still add them to `.gitignore` as a precaution
- Note this to the user

Only proceed to Step 6 after the user confirms the scan results are acceptable.

### Step 6: Initialize Git and Commit

```bash
git init
git add .
git commit -m "Initial commit: {brief description}"
```

If there are files that should not be committed (e.g., `.env`, `node_modules`, large binaries), create a `.gitignore` first.

### Step 7: Create Repo and Push

Use the `{owner}` from Step 3:

```bash
gh repo create {repo-name} --{public/private} --source=. --push
```

If the repo already exists on GitHub, use the verified `{owner}` from Step 3:

```bash
git remote add origin https://github.com/{owner}/{repo}.git
git push -u origin main
```

**Never construct URLs from `gh auth status` output** — always use the `{owner}` obtained from `gh api user`.

### Step 8: Set GitHub About

Use the `{owner}` verified in Step 3 to set the bilingual About description:

```bash
gh repo edit {owner}/{repo} --description "{English description} | {Chinese description}"
```

And add relevant topics:

```bash
gh repo edit {owner}/{repo} --add-topic {topic1},{topic2}
```

Choose topics based on the project's tech stack and purpose (e.g., `markdown`, `converter`, `python`, `document`).

### Step 9: Verify and Report

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
