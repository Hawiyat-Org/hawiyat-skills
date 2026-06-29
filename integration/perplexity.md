# Perplexity Web MCP Setup — Universal Harness Prompt

You are setting up Perplexity Web MCP (\`pwm\`) from https://github.com/jacob-bd/perplexity-web-mcp on this machine. This prompt is designed to be fed to any AI coding agent on any OS (Linux, macOS, Windows/WSL). Follow every step exactly.

> **Agent instructions:** This document is a sequential setup guide. Work through each section in order. Do not skip steps. Do not assume tools are installed — verify first. After completing all steps, print the final summary table. **On Windows, if an automated install fails, provide the user a step-by-step manual guide and wait for them to confirm completion before proceeding.** You are the one who will use \`pwm\` to fulfill the user's Perplexity tasks — do not print CLI usage examples or "next steps" to the user; just confirm setup is complete.

> **⚠️ Unofficial & Unsupported:** This tool is **not affiliated with, endorsed by, or supported by Perplexity AI**. It interacts with Perplexity's web interface through unofficial methods that may break at any time. Use at your own risk. Inform the user of this before proceeding.

> **Critical: Non-interactive shell limitation.** Most AI coding harnesses (Claude Code, Cursor, etc.) run commands in a **non-interactive shell**. The \`pwm login\` command supports \`--email\` and \`--code\` flags specifically for non-interactive environments — this is the recommended approach. Do **not** run \`pwm login\` without flags in a harness shell — it may block waiting for interactive input.

---

## 1. DETECT THE ENVIRONMENT

Determine the following before proceeding:

- **OS**: \`uname -a\` (Linux/macOS), \`ver\` (Windows). For Linux, also check the distro (\`cat /etc/os-release\`).
- **Home directory**: \`echo $HOME\`
- **Current working directory**: \`pwd\`
- **Shell**: \`echo $SHELL\`
- **AI client/agent**: Check for config directories and report which ones are present:
  \`\`\`
  ~/.claude/       → Claude Code
  ~/.agents/       → Cursor / Windsurf / Zed / Amp
  ~/.openclaw/     → OpenClaw
  ~/.gemini/       → Gemini CLI
  \`\`\`
  If multiple are present, report all. The agent will use \`pwm setup add all\` and \`pwm skill install all\` to configure all detected clients automatically.

Report all findings before continuing.

---

## 2. CHECK AND INSTALL PYTHON

\`pwm\` requires Python 3.10–3.13. However, if installing via \`uv\` (recommended), you do **not** need a compatible system Python — \`uv\` manages its own Python versions automatically. Only worry about the system Python version if installing via \`pip\` or \`pipx\`.

Check first — do not reinstall if already present:

\`\`\`bash
python3 --version 2>/dev/null || python --version 2>/dev/null
\`\`\`

**If already installed**, report the version. If the version is between 3.10 and 3.13, move to Step 3. If the version is below 3.10 or above 3.13, **do not reinstall Python** — instead, ensure \`uv\` is available (Step 3) and let it handle the Python version.

**If Python is not installed**, install using the appropriate method:

| OS | Method |
|----|--------|
| Linux (Debian/Ubuntu) | \`sudo apt-get install python3 python3-pip python3-venv\` |
| Linux (Fedora/RHEL) | \`sudo dnf install python3 python3-pip\` |
| Linux (Arch) | \`sudo pacman -S python python-pip\` |
| macOS (Homebrew) | \`brew install python@3.12\` |
| Windows | See Windows guide below |

#### Windows — install Python

**Option A — winget (recommended):**

\`\`\`bash
winget install Python.Python.3.12
\`\`\`

If winget is unavailable or fails, use the manual guide below.

**Option B — manual installer:**

Provide the user the following step-by-step guide and **wait for them to confirm** before continuing:

1. Open **https://www.python.org/downloads/** in your browser
2. Download the **Windows installer (.exe)** for Python 3.12 (look for "Latest Python 3.12 Release")
3. Run the downloaded \`.exe\` file
4. **Important:** Check **"Add python.exe to PATH"** at the bottom of the installer window
5. Click **Install Now** (or "Customize installation" if you need specific options)
6. After install completes, click **Disable path length limit** if offered, then **Close**
7. **Close and reopen your terminal** so PATH updates take effect
8. Run \`python --version\` to verify — report the version

Do not proceed until the user confirms valid Python version output (3.10–3.13) or confirms \`uv\` is available.

---

## 3. INSTALL PERPLEXITY WEB MCP

Check first — do not reinstall if already present:

\`\`\`bash
pwm --version 2>/dev/null
\`\`\`

**If already installed**, report the version and move to Step 4.

**If not installed**, use the recommended installer. **\`uv\` is strongly preferred** because it isolates the tool in its own environment and manages Python versions automatically — this is critical on systems where the system Python is outside the 3.10–3.13 range (e.g., Arch Linux with Python 3.14+).

### Option A — Install with uv (recommended)

Check if \`uv\` is already installed before installing it:

\`\`\`bash
which uv 2>/dev/null && uv --version
\`\`\`

**If \`uv\` is not present**, install it:

| OS | Method |
|----|--------|
| Linux / macOS | \`curl -fsSL https://astral.sh/uv/install.sh \\| bash\` |
| Windows | \`winget install astral-sh.uv\` |

#### Windows — install uv

**Option A — winget (recommended):**

\`\`\`bash
winget install astral-sh.uv
\`\`\`

**Option B — PowerShell installer:**

\`\`\`powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
\`\`\`

**Option C — manual download:**

Provide the user the following step-by-step guide and **wait for them to confirm** before continuing:

1. Open **https://github.com/astral-sh/uv/releases** in your browser
2. Download the latest \`uv-x86_64-pc-windows-msvc.zip\` file
3. Extract the zip file — you will get a \`uv.exe\` file
4. Move \`uv.exe\` to a directory on your PATH, such as \`C:\\Users\\<username>\\.local\\bin\\\`
   - If \`~/.local/bin/\` does not exist, create it: \`mkdir %USERPROFILE%\\.local\\bin\`
5. **Close and reopen your terminal** so PATH updates take effect
6. Run \`uv --version\` to verify — report the version

Do not proceed until the user confirms \`uv\` is available.

Then install \`pwm\`:

\`\`\`bash
uv tool install perplexity-web-mcp-cli
\`\`\`

> **Note:** \`uv tool install\` automatically downloads a compatible Python version (3.10–3.13) if the system Python is outside this range. No manual Python management needed.

### Option B — Install with pipx

\`\`\`bash
pip install pipx 2>/dev/null
pipx install perplexity-web-mcp-cli
\`\`\`

### Option C — Install with pip (fallback)

\`\`\`bash
pip install perplexity-web-mcp-cli
\`\`\`

> **Note:** \`uv\` and \`pipx\` are preferred because they isolate the tool in its own environment. \`pip\` installs into the current Python environment, which may cause conflicts.

### Verify

\`\`\`bash
pwm --version
\`\`\`

If \`pwm\` is not on PATH, report where it was installed and add it. Common locations:
- \`uv\` installs to \`~/.local/bin/\` (Linux/macOS) or \`%USERPROFILE%\\.local\\bin\\\` (Windows)
- \`pipx\` installs to \`~/.local/bin/\`
- \`pip\` installs to the Python environment's \`bin/\` or \`Scripts/\` directory

Report the installed version.

---

## 4. AUTHENTICATE WITH PERPLEXITY

\`pwm\` requires authentication via email + one-time code. This is a two-step process.

> **Non-interactive shell:** The commands below use \`--email\` and \`--code\` flags, which work in non-interactive shells. Do **not** run \`pwm login\` without flags — it may block waiting for input.

### Step 4a — Send Verification Code

\`\`\`bash
pwm login --email user@example.com
\`\`\`

Replace \`user@example.com\` with the user's email. This sends a 6-digit verification code to their inbox.

**If the command does not accept \`--email\`** (older version), fall back to running:

\`\`\`bash
pwm login
\`\`\`

This will print a URL or prompt. **Present the URL to the user** and ask them to open it in their browser to sign in. The command blocks until complete. Wait for the user to confirm.

### Step 4b — Complete Login with Code

Ask the user to check their email for the 6-digit code from Perplexity, then run:

\`\`\`bash
pwm login --email user@example.com --code 123456
\`\`\`

Replace \`123456\` with the code the user received.

### Step 4c — Verify Authentication

\`\`\`bash
pwm login --check
\`\`\`

Expected output: confirmation that the user is authenticated, including their email, username, and subscription tier.

If authentication fails, common issues:
- **"Invalid code"** → The code expired (they expire quickly). Re-run Step 4a to send a new code.
- **"User not found"** → The email is not associated with a Perplexity account. The user may need to sign up at https://perplexity.ai first.

---

## 5. CHECK USAGE AND RATE LIMITS

Verify the account's remaining quota:

\`\`\`bash
pwm usage
\`\`\`

Perplexity subscription tiers:

| Tier | Pro Searches | Deep Research | Files & Apps | Browser Agent | Labs |
|------|-------------|---------------|-------------|---------------|------|
| **Free** | 3/day | 0 | 0 | 0 | 0 |
| **Pro** ($20/mo) | Weekly pool | Monthly pool | Monthly pool | Monthly pool | Monthly pool |
| **Max** ($200/mo) | Weekly pool (larger) | Monthly pool (larger) | Monthly pool (larger) | Monthly pool (larger) | Monthly pool (larger) |

Report the remaining quotas to the user. Explain that:
- Free tier users get only **3 Pro Searches per day** — each \`pwm ask\` with default model uses one.
- Running out of Pro Searches will block further queries until the daily reset.
- \`pwm ask "question" --no-citations\` is a lightweight query that still uses a Pro Search.
- Deep Research (\`pwm research\`) is not available on the Free tier.

---

## 6. TEST THE CLI

Run a test query to confirm everything works:

\`\`\`bash
pwm ask "What is 2+2?" --no-citations
\`\`\`

Expected output: a concise answer like \`2+2 equals 4.\` without citations.

If this fails with an auth error, re-run Step 4. If it fails with a rate limit error, report the limits to the user.

---

## 7. CONFIGURE THE MCP SERVER (GLOBALLY)

The Perplexity MCP server (\`pwm-mcp\`) gives AI assistants direct access to Perplexity's models. **The MCP server must be configured globally** (user-level, not project-level) so it is available from any directory.

> **⚠️ Important:** Do NOT add the MCP server to project-local configs (e.g., \`.cursor/mcp.json\` in a project root, or \`.claude.json\` in a project). These only work inside that one project. Always use the **global/user-level** config path.

### Step 7a — Use pwm's built-in setup (primary method)

\`pwm setup add\` automatically detects installed clients and configures the MCP server globally for each. It writes the correct config to the correct **global** location for every supported client.

**Configure all detected clients at once:**

\`\`\`bash
pwm setup add all
\`\`\`

This interactively detects which clients are installed and configures each one at the user level. If the non-interactive shell prevents interactive prompts, configure per client below.

**Or configure specific clients:**

| Client | Command |
|--------|---------|
| Claude Code | \`pwm setup add claude-code\` |
| Claude Desktop | \`pwm setup add claude-desktop\` |
| Cursor | \`pwm setup add cursor\` |
| Windsurf | \`pwm setup add windsurf\` |
| Cline | \`pwm setup add cline\` |
| Gemini CLI | \`pwm setup add gemini\` |
| Codex CLI | \`pwm setup add codex\` |
| Antigravity | \`pwm setup add antigravity\` |
| OpenCode | \`pwm setup add opencode\` |

### Step 7b — Manual configuration (for unsupported clients or if pwm setup fails)

If \`pwm setup add\` does not support the detected client, or if the automated setup fails, configure the MCP server manually. All clients use the same underlying command — only the config file location differs.

> **⚠️ Always use the GLOBAL config path, not a project-local one.** A project-local config only works inside that directory.

**The MCP config block:**

\`\`\`json
{
  "mcpServers": {
    "perplexity": {
      "command": "pwm-mcp"
    }
  }
}
\`\`\`

**Global config file locations by client:**

| Client | Global config file location |
|--------|----------------------------|
| Claude Code | \`~/.claude.json\` (add under \`"mcpServers"\` key) |
| Claude Desktop (macOS) | \`~/Library/Application Support/Claude/claude_desktop_config.json\` |
| Claude Desktop (Windows) | \`%APPDATA%\\Claude\\claude_desktop_config.json\` |
| Cursor | \`~/.cursor/mcp.json\` |
| Windsurf | \`~/.codeium/windsurf/mcp_config.json\` |
| Cline (VS Code) | \`~/.cline/mcp_settings.json\` (or open via MCP Servers icon → Configure) |
| VS Code Copilot | \`%APPDATA%/Code/User/settings.json\` (under \`"mcp"\` key) |
| Zed | \`~/.config/zed/settings.json\` (under \`"context_servers"\`) |

For each detected client, either:

1. **If the global config file exists** — merge the \`perplexity\` entry into the existing \`mcpServers\` object (do not overwrite other servers).
2. **If the global config file does not exist** — create it with the full JSON block above.

> **Windows note:** Config paths use \`%APPDATA%\` or \`%USERPROFILE%\` instead of \`~\`. Expand them correctly: \`%APPDATA%\` = \`C:\\Users\\<username>\\AppData\\Roaming\`.

> **Claude Code shortcut:** For Claude Code specifically, the CLI provides a global add command:
> \`\`\`bash
> claude mcp add --scope user perplexity pwm-mcp
> \`\`\`
> The \`--scope user\` flag ensures the server is available in all projects, not just the current one.

### Step 7c — Remove any project-local MCP configs

Check for and remove any project-local MCP configurations that would override or conflict with the global setup:

\`\`\`bash
# Check for project-local configs (these should NOT exist for perplexity)
find . -name "mcp.json" -path "*/.cursor/*" 2>/dev/null
find . -name ".claude.json" -maxdepth 2 2>/dev/null
\`\`\`

If any of these files contain a \`perplexity\` entry, remove just that entry (keep other servers). If the file only contains perplexity, delete the file.

### Step 7d — Verify MCP (globally)

Run the verification command for the primary client:

**Claude Code:**

\`\`\`bash
claude mcp list
\`\`\`

Expected: \`perplexity: pwm-mcp - ✓ Connected\` (should appear regardless of which directory you're in)

**Claude Code — verify it's global (not project-local):**

\`\`\`bash
claude mcp list --scope user
\`\`\`

Expected: perplexity should appear in the user-scope list.

**Other clients:** Restart the client and check the MCP server status in settings. If the server shows as disconnected, verify that \`pwm-mcp\` is on PATH by running \`which pwm-mcp\` (Linux/macOS) or \`where pwm-mcp\` (Windows). Then confirm the config file is in the **global** location, not a project-local one.

---

## 8. INSTALL THE AGENT SKILL (GLOBALLY)

The Perplexity Web MCP project includes a portable Agent Skill (SKILL.md) that teaches AI agents how to use the CLI and MCP tools effectively. **Skills must be installed globally** (in the user's home config directory, not in any project directory) so they are available from any working directory.

> **⚠️ Important:** Do NOT install skills into project-local directories (e.g., \`.claude/skills/\` inside a project root). These only work inside that one project. Always install to the **global** config directory (e.g., \`~/.claude/skills/\`).

### Step 8a — Use pwm's built-in installer (primary method)

\`pwm skill install\` automatically detects installed clients and installs the skill globally for each.

**Install for all detected clients:**

\`\`\`bash
pwm skill install all
\`\`\`

**Or install for a specific client:**

| Client | Command |
|--------|---------|
| Claude Code | \`pwm skill install claude-code\` |
| Cursor | \`pwm skill install cursor\` |
| Codex CLI | \`pwm skill install codex\` |
| Gemini CLI | \`pwm skill install gemini-cli\` |
| Antigravity | \`pwm skill install antigravity\` |
| Cline | \`pwm skill install cline\` |
| OpenCode | \`pwm skill install opencode\` |
| OpenClaw | \`pwm skill install openclaw\` |

### Step 8b — Verify skills are globally accessible

Skills must be in the **global config directory** — not symlinked from a project, not in a project-local \`.claude/skills/\` folder. Verify the installation for the detected clients:

**Claude Code:**

\`\`\`bash
# Must be a real directory, NOT a symlink
ls -la ~/.claude/skills/perplexity-web-mcp

# Must show valid frontmatter
head -10 ~/.claude/skills/perplexity-web-mcp/SKILL.md
grep -E '^(name|description):' ~/.claude/skills/perplexity-web-mcp/SKILL.md
\`\`\`

**Cursor / Windsurf / Zed / Amp:**

\`\`\`bash
ls -la ~/.agents/skills/perplexity-web-mcp
head -10 ~/.agents/skills/perplexity-web-mcp/SKILL.md
\`\`\`

**OpenClaw:**

\`\`\`bash
ls -la ~/.openclaw/skills/perplexity-web-mcp
head -10 ~/.openclaw/skills/perplexity-web-mcp/SKILL.md
\`\`\`

**Gemini CLI:**

\`\`\`bash
ls -la ~/.gemini/skills/perplexity-web-mcp
head -10 ~/.gemini/skills/perplexity-web-mcp/SKILL.md
\`\`\`

**If the skill directory is missing or is a broken symlink**, manually copy it:

\`\`\`bash
# Find where pwm installed the skill source:
find ~/.local -name "SKILL.md" -path "*/perplexity-web-mcp/*" 2>/dev/null
find ~/.cargo -name "SKILL.md" -path "*/perplexity-web-mcp/*" 2>/dev/null

# Copy to the appropriate GLOBAL location (not project-local):
mkdir -p ~/.claude/skills
cp -r <source-path>/perplexity-web-mcp ~/.claude/skills/
\`\`\`

### Step 8c — Clean up any project-local skill installations

Check for and remove any project-local skill directories that could cause confusion:

\`\`\`bash
# Check for project-local skill dirs (these should NOT exist)
find . -name "perplexity-web-mcp" -path "*/skills/*" -maxdepth 4 2>/dev/null
\`\`\`

If any are found inside a project directory (not \`~/.claude/skills/\`), remove them. The global installation is the only one that should exist.

> **Windows note:** Symlinks require elevated privileges on Windows. If \`ln -sf\` fails, **copy** instead: \`cp -r <source> ~/.claude/skills/\`

### Step 8d — Verify the skill is usable from any directory

\`\`\`bash
# Change to a different directory and verify the skill is still visible
cd /tmp && head -5 ~/.claude/skills/perplexity-web-mcp/SKILL.md
cd -  # return to original directory
\`\`\`

If the skill is readable from any directory, it is correctly installed globally.

---

## 9. RUN DOCTOR

Run the built-in diagnostic tool to verify everything is correct:

\`\`\`bash
pwm doctor
\`\`\`

This checks:
- **Installation** — whether \`pwm\` and \`pwm-mcp\` are on PATH
- **Authentication** — whether a valid token exists and the account is accessible
- **Connectivity** — whether the Perplexity search endpoint is reachable
- **Rate Limits** — remaining quotas for each feature
- **MCP Configuration** — which clients have the MCP server configured
- **Skill Installation** — whether agent skills are installed and up to date

If any issues are reported, address them before continuing. The doctor output includes fix suggestions for every issue found.

> **Note:** The doctor may report clients as "not configured" (e.g., \`claude-desktop: not configured\`). This is expected if that client is not installed on the machine — only the detected clients need to be configured.

---

## 10. FINAL VERIFICATION AND SUMMARY

Run \`pwm --version\` and \`pwm login --check\`. Then print the summary table:

| Field | Value |
|-------|-------|
| OS detected | ... |
| AI client(s) detected | ... |
| Python version | ... (system) |
| uv version | ... (if installed) |
| pwm version | ... |
| Auth status | Authenticated / Not authenticated |
| Account email | ... |
| Subscription tier | Free / Pro / Max |
| Pro Searches remaining | ... |
| Deep Research remaining | ... |
| MCP server configured (global) | Yes / No (list clients) |
| MCP config location | ... (global paths) |
| Agent skill installed (global) | Yes / No (list clients) |
| Skill visible at | ... (global paths) |
| Doctor status | All OK / Issues found |

### Confirm completion

Print a brief confirmation that setup is complete. Do **not** print CLI usage examples or "next steps" — you are the agent who will use \`pwm\` to perform the user's Perplexity tasks. If the user's original request was just to set up Perplexity Web MCP, confirm and ask what they'd like to research or query.
