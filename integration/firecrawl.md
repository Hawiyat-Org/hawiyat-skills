# Firecrawl Setup — Universal Harness Prompt

You are setting up Firecrawl from https://docs.firecrawl.dev on this machine. This prompt is designed to be fed to any AI coding agent on any OS (Linux, macOS, Windows/WSL). Follow every step exactly.

> **Agent instructions:** This document is a sequential setup guide. Work through each section in order. Do not skip steps. Do not assume tools are installed — verify first. After completing all steps, print the final summary table. **On Windows, if an automated install fails, provide the user a step-by-step manual guide and wait for them to confirm completion before proceeding.** You are the one who will use \`firecrawl\` to fulfill the user's web scraping and data extraction tasks — do not print CLI usage examples or "next steps" to the user; just confirm setup is complete.

> **Critical: Non-interactive shell limitation.** Most AI coding harnesses (Claude Code, Cursor, etc.) run commands in a **non-interactive shell**. The \`firecrawl login\` command is interactive and will block waiting for input. This guide handles that via piped input or PKCE browser auth — **do not run \`firecrawl login\` without a non-interactive workaround in a harness shell.**

> **Firecrawl overview:** Firecrawl gives AI agents fast, reliable web context with search, scraping, and interaction tools. It has three skill segments: **CLI skills** (teach the agent to drive the CLI live), **Build skills** (teach the agent to integrate Firecrawl into product code), and **Workflow skills** (teach the agent to produce finished deliverables from web data).

---

## 1. DETECT THE ENVIRONMENT

Determine the following before proceeding:

- **OS**: \`uname -a\` (Linux), \`sw_vers\` (macOS), \`ver\` (Windows). For Linux, also check the distro (\`cat /etc/os-release\`).
- **Home directory**: \`echo $HOME\`
- **Current working directory**: \`pwd\`
- **AI client/agent**: Check for config directories and report which ones are present:
  \`\`\`
  ~/.claude/       → Claude Code
  ~/.agents/       → Cursor / Windsurf / Zed / Amp
  ~/.openclaw/     → OpenClaw
  ~/.gemini/       → Gemini CLI
  \`\`\`
  If multiple are present, report all. The agent will configure all detected clients in later steps.

Report all findings before continuing.

---

## 2. CHECK AND INSTALL NODE.JS + NPM

Check first — do not reinstall if already present:

\`\`\`bash
node --version && npm --version
\`\`\`

**If already installed**, report the versions and move to Step 3.

**If not installed**, install using the appropriate method for the OS:

| OS | Method |
|----|--------|
| Linux (Debian/Ubuntu) | \`curl -fsSL https://deb.nodesource.com/setup_lts.x \\| sudo -E bash - && sudo apt-get install -y nodejs\` |
| Linux (Fedora/RHEL) | \`sudo dnf install nodejs npm\` |
| Linux (Arch) | \`sudo pacman -S nodejs npm\` |
| Linux (nvm — any distro) | \`curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh \\| bash\` then \`nvm install --lts\` |
| macOS (Homebrew) | \`brew install node\` |
| macOS (nvm) | Same as Linux nvm |
| Windows | \`winget install OpenJS.NodeJS.LTS\` |

If npm is missing but node is present, enable it: \`corepack enable && corepack prepare npm@latest --activate\`

#### Windows fallback — if winget fails or is unavailable

Provide the user the following step-by-step guide and **wait for them to confirm** before continuing:

1. Open **https://nodejs.org/en/download** in your browser
2. Download the **Windows Installer (.msi)** — choose the LTS version
3. Run the downloaded \`.msi\` file
4. Click **Next** through the installer, accept the license agreement
5. Click **Install**, then **Finish**
6. **Close and reopen your terminal** so PATH updates take effect
7. Run \`node --version && npm --version\` to verify — report the versions

Do not proceed until the user confirms they see valid version output.

Report the installed versions.

---

## 3. INSTALL FIRECRAWL CLI

Check first — do not reinstall if already present:

\`\`\`bash
firecrawl --version
\`\`\`

**If already installed**, report the version and move to Step 4.

**If not installed**:

\`\`\`bash
npm install -g firecrawl-cli
\`\`\`

Verify with \`firecrawl --version\`. If not on PATH, check \`npm root -g\` and common bin directories, then add to PATH.

**Note:** The CLI is now installed globally and available as \`firecrawl\` from any directory.

---

## 4. AUTHENTICATE WITH FIRECRAWL

Firecrawl requires an API key for full functionality (crawl, map, agent, extract, and higher rate limits). Keyless mode works for \`scrape\`, \`search\`, \`interact\`, and \`parse\` at lower rate limits.

### Step 4a — Check current auth status

\`\`\`bash
firecrawl --status
\`\`\`

If already authenticated (shows an active team and credits), skip to Step 5.

### Step 4b — Authenticate via browser

Use the \`--browser\` flag which opens an auth page for the user to sign in or create an account:

\`\`\`bash
firecrawl init --all --browser
\`\`\`

The CLI will print a URL if the browser cannot open automatically. **Present the full URL to the user** and ask them to open it in their browser. The command polls until the browser login completes. Wait for the user to confirm before continuing.

**Fallback** — If the browser flow isn't working, use piped login:

\`\`\`bash
echo "1" | firecrawl login
\`\`\`

### Step 4c — Verify authentication

\`\`\`bash
firecrawl --status
\`\`\`

Expected output: shows your team name, credits remaining, and plan tier.

If authentication fails, the browser flow might not have completed; re-run Step 4b.

---

## 5. INSTALL ALL AGENT SKILLS

> **Why three skill segments?** Firecrawl skills are organized by what the agent is doing: running commands live (**CLI skills**), writing product code (**Build skills**), or producing finished deliverables (**Workflow skills**). All three are installed together below.

Install the skills using the Firecrawl CLI:

\`\`\`bash
firecrawl init --all --browser
\`\`\`

This installs all three skill segments:
- **CLI skills** (\`firecrawl/cli\`) — teach the agent how to drive the Firecrawl CLI: which command to run, when to scrape vs search vs interact, how to chain results, and how to recover when a job fails
- **Build skills** (\`firecrawl/skills\`) — teach the agent how to add Firecrawl to product code: pick the right API endpoint, install the matching SDK, store \`FIRECRAWL_API_KEY\` safely, write the call site, and ship a smoke-tested integration
- **Workflow skills** (\`firecrawl/firecrawl-workflows\`) — turn Firecrawl web data into finished deliverables: research briefs, SEO audits, lead lists, QA reports, knowledge bases, and design clones

> **Note:** If not using \`--browser\`, authenticate first via Step 4 then run: \`firecrawl init --all\`

### Install skills globally

Skills must be available **globally** so the agent can use them from **any directory** — not just the project where they were installed. Do NOT symlink from \`~/.claude/skills/\` to the project-local \`.agents/skills/\` — those symlinks break when the user runs Claude Code from a different directory. Instead, **copy** the skill directories into the global config.

**For Claude Code** — copy into \`~/.claude/skills/\`:

\`\`\`bash
mkdir -p ~/.claude/skills
cp -r .agents/skills/firecrawl-* ~/.claude/skills/
\`\`\`

**For OpenClaw** — copy into \`~/.openclaw/skills/\`:

\`\`\`bash
mkdir -p ~/.openclaw/skills
cp -r .agents/skills/firecrawl-* ~/.openclaw/skills/
\`\`\`

> **Windows note:** Symlinks require elevated privileges on Windows. If \`ln -sf\` fails, **copy** instead: \`cp -r .agents/skills/firecrawl-* .claude/skills/\`

### Verify skills are globally accessible

\`\`\`bash
# Should show real files, NOT symlinks
ls -la ~/.claude/skills/firecrawl

# Should be readable and contain valid frontmatter
head -5 ~/.claude/skills/firecrawl/SKILL.md

# Count total firecrawl skills
ls ~/.claude/skills/ | grep ^firecrawl | wc -l
\`\`\`

If \`ls -la\` shows \`->\` (a symlink), the copy failed — remove the symlink and re-run the \`cp\` command. Report the total count of firecrawl skills installed (expected: 31).

---

## 6. CONFIGURE THE FIRECRAWL MCP SERVER

The Firecrawl MCP server gives AI assistants direct access to Firecrawl's web scraping and data extraction tools. It requires \`FIRECRAWL_API_KEY\` to be set in the environment.

There are two modes:

### Mode A — Local MCP server (via npx)

\`\`\`json
{
  "mcpServers": {
    "firecrawl": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_KEY": "fc-..."
      }
    }
  }
}
\`\`\`

### Mode B — Remote MCP server (no local process)

\`\`\`json
{
  "mcpServers": {
    "firecrawl": {
      "url": "https://mcp.firecrawl.dev/v2/mcp",
      "env": {
        "FIRECRAWL_API_KEY": "fc-..."
      }
    }
  }
}
\`\`\`

### Installation per client

Apply the MCP config to the detected AI client(s). Always use the **global** config path (not project-local) so the server is available from any directory.

| Client | Global config location | Add command |
|--------|----------------------|-------------|
| Claude Code | \`~/.claude.json\` (under \`"mcpServers"\`) | \`claude mcp add --scope user firecrawl npx -- -y firecrawl-mcp\` |
| Claude Desktop (macOS) | \`~/Library/Application Support/Claude/claude_desktop_config.json\` | Edit config |
| Claude Desktop (Windows) | \`%APPDATA%\\Claude\\claude_desktop_config.json\` | Edit config |
| Cursor | \`~/.cursor/mcp.json\` | Edit config |
| Windsurf | \`~/.codeium/windsurf/mcp_config.json\` | Edit config |
| VS Code Copilot | \`%APPDATA%/Code/User/settings.json\` (under \`"mcp"\`) | Edit settings |
| Cline (VS Code) | \`~/.cline/mcp_settings.json\` | Click MCP Servers icon → Configure |
| Zed | \`~/.config/zed/settings.json\` (under \`"context_servers"\`) | Edit settings |

For each detected client, either:
1. **If the global config file exists** — merge the \`firecrawl\` entry into the existing \`mcpServers\` object (do not overwrite other servers)
2. **If the global config file does not exist** — create it with the config block

> **Important:** Do NOT add the MCP server to project-local configs (e.g., \`.cursor/mcp.json\` in a project root). These only work inside that one directory.

### Verify MCP

For Claude Code:

\`\`\`bash
claude mcp list
\`\`\`

Expected: \`firecrawl: npx -y firecrawl-mcp - ✓ Connected\`

For other clients, restart the client and check the MCP server status. If the server shows as disconnected, verify that \`FIRECRAWL_API_KEY\` is set and the config file is in the global location.

---

## 7. VERIFY THE INSTALLATION

Run the official verification:

\`\`\`bash
mkdir -p .firecrawl
firecrawl --status
firecrawl scrape "https://firecrawl.dev" -o .firecrawl/install-check.md
\`\`\`

Then verify the scraped output:

\`\`\`bash
head -20 .firecrawl/install-check.md
\`\`\`

Also test search:

\`\`\`bash
firecrawl search "Firecrawl web scraping" --limit 3
\`\`\`

> **Note:** The search flag is \`--limit\`, not \`--max-results\`.

Expected results:
- \`firecrawl --status\` shows authenticated team and credits
- \`firecrawl scrape\` produces a markdown file with content from the Firecrawl homepage
- \`firecrawl search\` returns a list of search results with URLs and snippets

If any command fails, debug before proceeding:
- **Auth errors**: re-run Step 4
- **Network errors**: check internet connectivity
- **Command not found**: verify \`firecrawl\` is on PATH, or re-install

---

## 8. FETCH AND STORE REFERENCE DOCUMENTATION

Download these resources to the project root for future reference:

\`\`\`bash
mkdir -p docs
curl -sL "https://raw.githubusercontent.com/firecrawl/cli/main/README.md" -o FIRECRAWL_CLI.md
curl -sL "https://raw.githubusercontent.com/firecrawl/skills/main/README.md" -o FIRECRAWL_SKILLS.md
curl -sL "https://raw.githubusercontent.com/firecrawl/firecrawl-workflows/main/README.md" -o FIRECRAWL_WORKFLOWS.md
\`\`\`

Also fetch the main docs page:

\`\`\`bash
curl -sL "https://docs.firecrawl.dev/llms.txt" -o docs/firecrawl-llms.txt
\`\`\`

Verify each file is non-empty after download. If any curl fails, report which files are missing and continue with the ones that succeeded.

---

## 9. AGENT KNOWLEDGE: CHOOSE YOUR PATH

The following paths describe how Firecrawl is used. The agent should reference these when deciding how to fulfill the user's web data tasks. Do NOT print this section to the user.

### Path A — Live Web Tools

Use this when you need web data during your work: searching, scraping known URLs, interacting with live pages, crawling docs, or mapping a site.

Default flow:
1. Start with \`firecrawl search\` when you have discovery needs
2. Move to \`firecrawl scrape\` when you have a URL
3. Use \`firecrawl interact\` only when the page needs clicks, forms, or login
4. If any step fails or returns unexpected output, run \`firecrawl ask\` with the failing \`jobId\`

Available commands: \`firecrawl search\`, \`firecrawl scrape\`, \`firecrawl interact\`, \`firecrawl crawl\`, \`firecrawl map\`, \`firecrawl ask\`, \`firecrawl docs-search\`

### Path B — Integrate Firecrawl Into an App

Use this when building an application, agent, or workflow that calls the Firecrawl API **from code** — the integration runs inside the user's product, not from the agent's terminal session.

Choose the project mode:
- **Fresh project** — pick the stack, install the SDK, add env vars, run a smoke test
- **Existing project** — inspect the repo first, integrate where the project already handles APIs and secrets

Available build skills: \`firecrawl-build\`, \`firecrawl-build-onboarding\`, \`firecrawl-build-scrape\`, \`firecrawl-build-search\`, \`firecrawl-build-interact\`, \`firecrawl-build-parse\`

### Path C — Repeatable Deliverables

Use this when the goal is a finished artifact: research brief, SEO audit, QA report, lead list, knowledge base, competitive intel digest, or design clone.

Default flow:
1. Confirm the workflow and final artifact with the user
2. Collect web evidence with Firecrawl through the CLI
3. Save or cite source evidence so claims are traceable
4. Run independent research units in parallel when available
5. Synthesize findings into the requested deliverable

Start with the umbrella \`firecrawl-workflows\` skill which inspects the request and routes to the right workflow.

### Path D — Account Authorization

Reference for the auth flow in Step 4 above.

### Path E — Use Firecrawl Without Installing Anything

Use the REST API directly:

**Base URL:** \`https://api.firecrawl.dev/v2\`
**Auth header:** \`Authorization: Bearer fc-YOUR_API_KEY\`

Available endpoints: \`POST /search\`, \`POST /scrape\`, \`POST /interact\`, \`POST /support/ask\`, \`POST /support/docs-search\`

---

## 10. FINAL VERIFICATION AND SUMMARY

Run one final verification:

\`\`\`bash
firecrawl --status
\`\`\`

Then print the summary table:

| Field | Value |
|-------|-------|
| OS detected | ... |
| AI client(s) detected | ... |
| Node.js version | ... |
| npm version | ... |
| Firecrawl CLI version | ... |
| Auth status | Authenticated / Not authenticated |
| Team name | ... |
| Credits remaining | ... |
| Skills installed | ... (count) |
| Skills visible at | \`~/.claude/skills/\` (global) |
| MCP server configured | Yes / No |
| MCP config location | ... (path) |
| FIRECRAWL_API_KEY | Set / Not set |

### Confirm completion

Print a brief confirmation that setup is complete. Do **not** print CLI usage examples or "next steps" — you are the agent who will use Firecrawl to perform the user's web data tasks. If the user's original request was just to set up Firecrawl, confirm and ask what they'd like to scrape, search, or build.

---

> **Reference:** This setup guide incorporates material from https://docs.firecrawl.dev/ai-onboarding — the official Firecrawl AI agent onboarding documentation.
