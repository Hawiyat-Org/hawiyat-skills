# GitHub CLI Setup — Universal Harness Prompt

You are setting up the GitHub CLI (\`gh\`) from https://cli.github.com on this machine. This prompt is designed to be fed to any AI coding agent on any OS (Linux, macOS, Windows/WSL). Follow every step exactly.

> **Agent instructions:** This document is a sequential setup guide. Work through each section in order. Do not skip steps. Do not assume tools are installed — verify first. After completing all steps, inform the user of the final status. **On Windows, if an automated install fails, provide the user a step-by-step manual guide and wait for them to confirm completion before proceeding.**

---

## 1. DETECT THE ENVIRONMENT

Determine the following before proceeding:

- **OS**: \`uname -a\` (Linux), \`sw_vers\` (macOS), \`ver\` (Windows). For Linux, also check the distro (\`cat /etc/os-release\`).
- **Home directory**: \`echo $HOME\`
- **Current working directory**: \`pwd\`
- **AI client/agent**: Check for config directories and report which one is present:
  \`\`\`
  ~/.claude/       → Claude Code
  ~/.agents/       → Cursor / Windsurf / Zed / Amp
  ~/.openclaw/     → OpenClaw
  ~/.gemini/       → Gemini
  \`\`\`
  If multiple are present, report all and default to the first.

Report all findings before continuing.

---

## 2. CHECK AND INSTALL GITHUB CLI

Check if \`gh\` is already installed:

\`\`\`bash
gh --version
\`\`\`

**If already installed**, report the version and skip to Step 3.

**If not installed**, install using the appropriate method:

| OS | Method |
|----|--------|
| **Windows** | First check: \`winget --version\`. If winget is available, run \`winget install --id GitHub.cli\`. If winget is NOT installed, tell the user to install winget from the Microsoft Store or download the GitHub CLI MSI installer directly from **https://cli.github.com** |
| **macOS** | \`brew install gh\` (requires [Homebrew](https://brew.sh)) |
| **Linux (Debian/Ubuntu)** | \`curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo gpg --dearmor -o /usr/share/keyrings/githubcli-archive-keyring.gpg && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null && sudo apt update && sudo apt install gh\` |
| **Linux (Fedora/RHEL)** | \`sudo dnf install 'dnf-command(config-manager)' && sudo dnf config-manager --add-repo https://cli.github.com/packages/rpm/gh-cli.repo && sudo dnf install gh\` |
| **Linux (Arch)** | \`sudo pacman -S github-cli\` |
| **Linux (any distro via tar)** | Download from **https://github.com/cli/cli/releases/latest**, extract and place \`gh\` binary on PATH |
| **macOS (without Homebrew)** | Download the .pkg or .tar.gz from **https://github.com/cli/cli/releases/latest** |

After installing, verify:

\`\`\`bash
gh --version
\`\`\`

---

## 3. AUTHENTICATE WITH GITHUB (PERSONAL ACCESS TOKEN)

⚠️ **STOP AND READ:** The agent **must stop here** and ask the user to provide a GitHub Personal Access Token.

Tell the user:

> **Go to https://github.com/settings/tokens and generate a new token (classic or fine-grained).**
> - For fine-grained: select the repos you need, set at least \`Contents: Read\`, \`Issues: Read & Write\`, \`Pull requests: Read & Write\`
> - For classic: enable \`repo\` scope
> - Copy the token and paste it here

**Wait for the user to paste the token before continuing.**

Once the user provides the token, store it:

\`\`\`bash
# Store the token
echo "export GITHUB_TOKEN=your_token_here" >> ~/.bashrc
export GITHUB_TOKEN=your_token_here
\`\`\`

Then authenticate gh with the token:

\`\`\`bash
echo "$GITHUB_TOKEN" | gh auth login --with-token
\`\`\`

Verify:

\`\`\`bash
gh auth status
\`\`\`

Expected output: \`✓ Logged in to github.com as YOUR_USERNAME (...)\`

---

## 4. RESTART YOUR CLIENT

After authentication, the user must **restart their AI coding client** (Claude Code, Cursor, etc.) for the GitHub CLI environment variables and session to be picked up.

> **Tell the user:** Please quit and restart your AI coding client now. This ensures the \`gh\` command and its authentication are fully available in the agent's environment.

---

## 5. INSTALL AND LOAD GH SKILLS

GitHub CLI supports agent skills via \`gh skill\` (docs: https://cli.github.com/manual/gh_skill). These skills extend what your AI agent can do with GitHub.

### List available skills

\`\`\`bash
gh skill search
\`\`\`

### Install skills

\`\`\`bash
# Install specific skills (examples)
gh skill install github/awesome-copilot documentation-writer
gh skill install github/awesome-copilot pr-reviewer
\`\`\`

### List installed skills

\`\`\`bash
gh skill list
\`\`\`

### Update all installed skills

\`\`\`bash
gh skill update --all
\`\`\`

### GH Skill Commands Reference

GitHub CLI provides the following skill management commands:

| Command | Description |
|---------|-------------|
| \`gh skill install <owner/repo> <name>\` | Install a skill from a GitHub repository |
| \`gh skill list\` | List installed skills |
| \`gh skill preview <owner/repo> <name>\` | Preview a skill before installing |
| \`gh skill publish [--dry-run]\` | Publish a skill |
| \`gh skill search <query>\` | Search for available skills |
| \`gh skill update [--all]\` | Update installed skills |

For full documentation: **https://cli.github.com/manual/gh_skill**

---

## 6. FINAL VERIFICATION AND SUMMARY

Run final verification:

\`\`\`bash
gh --version && gh auth status && gh skill list
\`\`\`

Then print the summary table:

| Field | Value |
|-------|-------|
| OS detected | ... |
| AI client(s) detected | ... |
| GitHub CLI version | ... |
| Auth status | Authenticated / Not authenticated |
| Logged in as | ... |
| Installed skills | ... (list) |
| Skills documented at | https://cli.github.com/manual/gh_skill |

### Confirm completion

Print a brief confirmation that setup is complete.

---

## 7. GENERATE SKILL MANIFEST

Now that everything is working, fetch all available GitHub CLI skills and write them to a \`SKILL.md\` manifest so the user can reference them later:

\`\`\`bash
cat > SKILL.md << 'SKILL_EOF'
# GitHub CLI Skills

$(gh auth status 2>&1 | head -2)
$(gh --version | head -1)

## Available commands

$(gh help | grep -E "^\\s+\\w" | head -40)

## Available skills

$(gh skill list 2>/dev/null || echo "No skills found")

## Authentication

$(gh auth status 2>&1 | tail -5)
SKILL_EOF

echo "SKILL.md generated"
\`\`\`

Tell the user the manifest is ready at \`SKILL.md\` in the current directory and they can use it to see all available GitHub capabilities.

---

## 8. CREATE THE CLAUDE CODE SKILL

Create a skill directory and SKILL.md so Claude Code recognizes the GitHub integration:

\`\`\`bash
mkdir -p ~/.claude/skills/github
cat > ~/.claude/skills/github/SKILL.md << 'SKILLEOF'
---
name: github
description: "GitHub integration via gh CLI — PRs, issues, repos, search, workflows, gists, releases, actions. Token stored as GITHUB_TOKEN or gh auth. Activates on /github or intent like 'create a PR', 'list open issues', 'clone a repo'"
---
# GitHub Integration
Full access to GitHub via the gh CLI. Manage repositories, pull requests, issues, workflows, and more.
## Setup
Authenticated via \`gh auth status\`:
\`\`\`bash
gh auth status && gh --version
\`\`\`
## Available Operations
### Pull Requests
- **List PRs**: \`gh pr list --state open --limit 20\`
- **View PR**: \`gh pr view <number>\`
- **Create PR**: \`gh pr create --title "X" --body "Y" --base main\`
- **Review PR**: \`gh pr review <number> --approve\`
- **Merge PR**: \`gh pr merge <number> --squash\`
### Issues
- **List issues**: \`gh issue list --state open --limit 20\`
- **View issue**: \`gh issue view <number>\`
- **Create issue**: \`gh issue create --title "X" --body "Y"\`
- **Close issue**: \`gh issue close <number>\`
### Repos
- **List repos**: \`gh repo list <owner> --limit 30\`
- **View repo**: \`gh repo view <owner>/<repo>\`
- **Clone repo**: \`gh repo clone <owner>/<repo>\`
- **Create repo**: \`gh repo create <name> --public --clone\`
### Workflows / Actions
- **List workflows**: \`gh workflow list\`
- **Run workflow**: \`gh workflow run <name>\`
- **List runs**: \`gh run list --limit 10\`
- **View run**: \`gh run view <id>\`
### Search
- **Code**: \`gh search code "pattern" --repo <owner>/<repo>\`
- **Issues/PRs**: \`gh search issues "text" --state open\`
- **Repos**: \`gh search repos "keyword"\`
### Gists
- **List gists**: \`gh gist list --limit 20\`
- **Create gist**: \`gh gist create <file>\`
- **View gist**: \`gh gist view <id>\`
### Releases
- **List releases**: \`gh release list --limit 10\`
- **Create release**: \`gh release create <tag> --title "X"\`
- **Download asset**: \`gh release download <tag> --pattern "*.zip"\`
## Tips
- Use \`--json\` flag for machine-readable output
- Pipe through \`jq\` for complex queries
- Use \`gh api\` for endpoints not covered by CLI
- Always specify \`--repo owner/repo\` when outside the repo directory
SKILLEOF
echo "GitHub skill created"
\`\`\`

### Verify

\`\`\`bash
ls -la ~/.claude/skills/github/SKILL.md
\`\`\`

---

## 9. CREATE THE /GITHUB COMMAND FILE

\`\`\`bash
cat > ~/.claude/commands/github.md << 'CMDEOF'
---
description: "GitHub integration via gh CLI — PRs, issues, repos, search. Usage: /github [task]"
---
Load the GitHub skill from \`~/.claude/skills/github/SKILL.md\` and execute the user's request using gh CLI.
**Arguments**: $ARGUMENTS
If no arguments provided, show a summary of available operations.

## Quick Reference
- Auth: \`gh auth status\`
- API: \`gh api\` for REST/GraphQL

## Common Tasks
| Task | Command |
|------|---------|
| List PRs | \`gh pr list --state open\` |
| Create PR | \`gh pr create --title "X" --body "Y"\` |
| List issues | \`gh issue list --state open\` |
| Create issue | \`gh issue create --title "X" --body "Y"\` |
| List repos | \`gh repo list <owner>\` |
| Clone repo | \`gh repo clone <owner>/<repo>\` |
| Run workflow | \`gh workflow run <name>\` |
| Search code | \`gh search code "pattern"\` |
CMDEOF
echo "GitHub command created"
\`\`\`

### Verify

\`\`\`bash
ls -la ~/.claude/commands/github.md
\`\`\`

---

## 10. CREATE MEMORY FILES (for cross-session persistence)

\`\`\`bash
MEMORY_DIR=$(find ~/.claude/projects -name "MEMORY.md" -path "*/memory/*" -exec dirname {} \\; 2>/dev/null | head -1)
if [ -z "$MEMORY_DIR" ]; then
  MEMORY_DIR="$HOME/.claude/memory"
  mkdir -p "$MEMORY_DIR"
fi

cat > "$MEMORY_DIR/github-skills-index.md" << 'MEMEOF'
---
name: github-skills-index
description: Master index of GitHub CLI skills
---
# GitHub Skills Index
Auth: gh auth status
## Files
- github-pull-requests.md — PR operations
- github-issues.md — Issue management
- github-repos.md — Repository management
- github-actions.md — Workflows and Actions
- github-search.md — Code and content search
MEMEOF

cat > "$MEMORY_DIR/github-pull-requests.md" << 'MEMEOF'
---
name: github-pull-requests
description: GH PR operations
---
gh pr list --state open --json number,title,author,headRefName,createdAt
gh pr view <number> --json body,reviews,comments,additions,deletions,files
gh pr create --title "X" --body "Y" --base main --head branch
gh pr review <number> --approve --body "Looks good"
gh pr merge <number> --squash --delete-branch
gh pr checkout <number>
MEMEOF

cat > "$MEMORY_DIR/github-issues.md" << 'MEMEOF'
---
name: github-issues
description: GH issue management
---
gh issue list --state open --json number,title,labels,assignees,createdAt
gh issue view <number> --json body,comments,reactions
gh issue create --title "X" --body "Y" --label "bug" --assignee @me
gh issue close <number>
gh issue reopen <number>
gh issue comment <number> --body "Fixed in #123"
MEMEOF

cat > "$MEMORY_DIR/github-repos.md" << 'MEMEOF'
---
name: github-repos
description: GH repository management
---
gh repo list <owner> --limit 30 --json name,description,updatedAt,primaryLanguage
gh repo view <owner>/<repo> --json description,readme,languages,license
gh repo create <name> --public --clone --description "X"
gh repo fork <owner>/<repo> --clone
gh repo archive <owner>/<repo> -y
MEMEOF

cat > "$MEMORY_DIR/github-actions.md" << 'MEMEOF'
---
name: github-actions
description: GH Actions and workflows
---
gh workflow list --json name,id,state
gh workflow run <name> --ref main -f key=value
gh run list --limit 10 --json name,status,conclusion,createdAt
gh run view <id> --log
gh run watch <id>
gh run rerun <id>
MEMEOF

cat > "$MEMORY_DIR/github-search.md" << 'MEMEOF'
---
name: github-search
description: GH code and content search
---
gh search code "pattern" --repo owner/repo --extension ts --limit 20
gh search issues "text" --state open --repo owner/repo --json title,url
gh search repos "topic:react stars:>1000" --limit 20 --json name,url,stargazerCount
gh search prs "label:bug" --state open --json title,number,url
gh search commits "fix" --repo owner/repo --limit 10
MEMEOF

if ! grep -q "github-skills-index" "$MEMORY_DIR/../MEMORY.md" 2>/dev/null; then
  cat >> "$MEMORY_DIR/../MEMORY.md" << 'IDXEOF'
- [GitHub Skills Index](notion-skills/github-skills-index.md) — Master index
- [GitHub PRs](notion-skills/github-pull-requests.md) — Pull requests
- [GitHub Issues](notion-skills/github-issues.md) — Issues
- [GitHub Repos](notion-skills/github-repos.md) — Repositories
- [GitHub Actions](notion-skills/github-actions.md) — Workflows
- [GitHub Search](notion-skills/github-search.md) — Search
IDXEOF
fi
echo "GitHub memory files created"
\`\`\`

---

## 11. RESTART INSTRUCTIONS

⚠️ **\`claude skills reload\` does NOT work reliably.** The user MUST restart Claude Code for \`/github\` to appear.

Tell the user:

> **Setup is complete! To activate \`/github\`:**
>
> 1. **Close this Claude Code session** (type \`/exit\` or close the window)
> 2. **Reopen Claude Code** in the same directory
> 3. **Type \`/github\`** — it should now appear in the command list

---

## 12. HOW TO USE IN NEW SESSIONS

**Slash command:**
\`\`\`
/github list open PRs in owner/repo
/github create an issue for bug in login flow
/github search for code using deprecated API
\`\`\`

**Natural language:**
\`\`\`
"Use my GitHub skills to clone a repo"
"Search GitHub for React components using TypeScript"
\`\`\`

The agent will:
1. Load the skill from \`~/.claude/skills/github/SKILL.md\`
2. Execute the GitHub CLI command via \`gh\`
3. Return the results
