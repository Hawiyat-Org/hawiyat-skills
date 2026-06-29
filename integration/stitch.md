---
name: google-stitch
description: Stitch MCP integration — design with AI, fetch design DNA directly in your editor via MCP
---

# Google Stitch MCP — Setup Prompt

You are setting up the Google Stitch MCP server from https://stitch.withgoogle.com/docs/mcp/setup on this machine. This prompt is designed to be fed to any AI coding agent on any OS (Linux, macOS, Windows/WSL). Follow every step exactly.

> **Agent instructions:** This document is a sequential setup guide. Work through each section in order. Do not skip steps. Do not assume tools are installed — verify first. After completing all steps, inform the user of the final status.

---

## 1. DETECT THE ENVIRONMENT

Determine the following before proceeding:

- **OS**: `uname -a` (Linux), `sw_vers` (macOS), `ver` (Windows)
- **Home directory**: `echo $HOME`
- **AI client/agent**: Check for config directories:
  ```
  ~/.claude/       → Claude Code
  ~/.agents/       → Cursor / Windsurf / Zed / Amp
  ~/.gemini/       → Gemini
  ```
- **Node.js**: `node --version` (v18+ required for npx)

Report all findings before continuing.

---

## 2. GENERATE A STITCH API KEY

⚠️ **STOP AND READ:** The agent must stop here and ask the user to generate a Stitch API key.

Tell the user:

> **Go to https://stitch.withgoogle.com and:**
> 1. Sign in with your Google account
> 2. Click your **Profile Picture** in the top right corner
> 3. Select **Stitch settings** from the dropdown
> 4. Navigate to the **API key** section
> 5. Click **Create key**
> 6. **Copy the key immediately** — it will not be shown again
>
> Paste the key here when ready.

**Wait for the user to paste the key before continuing.**

Once the user provides the key, store it securely:

```bash
# Store the key in shell config
echo 'export STITCH_API_KEY=your_key_here' >> ~/.bashrc
export STITCH_API_KEY=your_key_here
```

> The API key is the **only** authentication method needed. No Google Cloud project, no billing, no OAuth required.

---

## 3. CONFIGURE THE STITCH MCP SERVER

### Claude Code

```bash
claude mcp add stitch --transport http https://stitch.googleapis.com/mcp --header "X-Goog-Api-Key: $STITCH_API_KEY" -s user
```

Verify:

```bash
claude mcp get stitch --json
```

Expected output — `transport.type` is `streamable_http`, `transport.url` is the Stitch endpoint, and `X-Goog-Api-Key` header is set.

### Cursor / Windsurf / Zed / Amp

Add to your MCP config file (`~/.cursor/mcp.json`, `~/.windsurf/mcp_config.json`, etc.):

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["-y", "@_davideast/stitch-mcp", "proxy"],
      "env": {
        "STITCH_API_KEY": "<your-api-key>"
      }
    }
  }
}
```

### Any client via npx proxy (fallback)

If the client does not support HTTP MCP natively, use the npx proxy:

```bash
npx -y @_davideast/stitch-mcp proxy
```

This starts a local stdio MCP server that proxies to the Stitch API. Set `STITCH_API_KEY` in the environment before launching.

---

## 4. VERIFY THE CONNECTION

Restart your AI coding client, then verify:

```bash
/mcp
```

Expected: `stitch · ✔ connected`

Test with a tool call:

```
List my Stitch projects.
```

Expected: returns your Stitch projects (or empty list if none created yet).

---

## 5. CREATE THE SKILL MANIFEST

Create a skill directory and SKILL.md so the AI client recognizes the Stitch MCP integration:

```bash
mkdir -p ~/.claude/skills/stitch
cat > ~/.claude/skills/stitch/SKILL.md << 'SKILLEOF'
---
name: stitch
description: "Google Stitch MCP — design with AI. Fetch design DNA, read projects, apply design tokens. API key stored as STITCH_API_KEY."
---
# Google Stitch MCP
Design with AI. Stitch MCP gives your AI agent direct access to your Stitch designs — read project metadata, fetch design tokens, and apply UI decisions without manual exports.

## Setup
- **MCP endpoint**: `https://stitch.googleapis.com/mcp`
- **Auth**: `X-Goog-Api-Key` header via `STITCH_API_KEY`
- **Status**: `/mcp` should show `stitch · ✔ connected`

## Available Tools

### Projects
- **List projects**: List my Stitch projects.
- **Get project**: Show details for project [name].
- **Read design**: Read the design from project [name].

### Design Tokens
- **Get colors**: What colors are used in [project]?
- **Get typography**: Show the typography system for [project].
- **Get layout**: Describe the layout of [project].

### Export
- **Export to code**: Export [project] as React/Tailwind.
- **Get CSS**: Show the CSS for [project].

## Tips
- Use the MCP server directly — no need to copy hex codes or spacing values manually.
- Design DNA includes colors, typography, spacing, and layout tokens.
- The API key does not expire but can be revoked from Stitch settings.
SKILLEOF
echo "Stitch skill created"
```

### Verify

```bash
ls -la ~/.claude/skills/stitch/SKILL.md
```

---

## 6. CREATE THE /STITCH COMMAND FILE

```bash
mkdir -p ~/.claude/commands
cat > ~/.claude/commands/stitch.md << 'CMDEOF'
---
description: "Google Stitch MCP — design with AI. Usage: /stitch [task]"
---
Load the Stitch skill from `~/.claude/skills/stitch/SKILL.md` and execute the user's request using Stitch MCP tools.
**Arguments**: $ARGUMENTS
If no arguments provided, show a summary of available operations.

## Quick Reference
- **MCP**: `stitch` via `https://stitch.googleapis.com/mcp`
- **Auth**: `X-Goog-Api-Key` header (env: `$STITCH_API_KEY`)

## Common Tasks
| Task | Prompt |
|------|--------|
| List projects | List my Stitch projects |
| Read design | Read the design from [project] |
| Get colors | Show the color palette for [project] |
| Get typography | What fonts and text styles in [project]? |
| Export | Export [project] as React code |
CMDEOF
echo "Stitch command created"
```

### Verify

```bash
ls -la ~/.claude/commands/stitch.md
```

---

## 7. CREATE MEMORY FILES (for cross-session persistence)

```bash
MEMORY_DIR=$(find ~/.claude/projects -name "MEMORY.md" -path "*/memory/*" -exec dirname {} \; 2>/dev/null | head -1)
if [ -z "$MEMORY_DIR" ]; then
  MEMORY_DIR="$HOME/.claude/memory"
  mkdir -p "$MEMORY_DIR"
fi

cat > "$MEMORY_DIR/stitch-mcp.md" << 'MEMEOF'
---
name: stitch-mcp
description: Google Stitch MCP reference
---
# Google Stitch MCP
- **Endpoint**: `https://stitch.googleapis.com/mcp`
- **Auth header**: `X-Goog-Api-Key`
- **Env var**: `$STITCH_API_KEY`
- **Config**: `claude mcp add stitch --transport http https://stitch.googleapis.com/mcp --header "X-Goog-Api-Key: $STITCH_API_KEY" -s user`
- **Verify**: `/mcp` → `stitch · ✔ connected`
- **Test**: "List my Stitch projects."

## Quick commands
| Action | Command |
|--------|---------|
| List projects | List my Stitch projects |
| Read design | Read the design from [name] |
| Get tokens | Get design tokens for [name] |
| Export code | Export [name] as React |
MEMEOF

if ! grep -q "stitch-mcp" "$MEMORY_DIR/../MEMORY.md" 2>/dev/null; then
  cat >> "$MEMORY_DIR/../MEMORY.md" << 'IDXEOF'
- [Stitch MCP](stitch-mcp.md) — Google Stitch design MCP integration
IDXEOF
fi
echo "Stitch memory files created"
```

---

## 8. RESTART INSTRUCTIONS

⚠️ **The client must be restarted** for the MCP server and slash command to be picked up.

Tell the user:

> **Setup is complete! To activate:**
>
> 1. **Close this AI coding session** (type `/exit` or close the window)
> 2. **Reopen the client** in the same directory
> 3. **Type `/mcp`** — `stitch` should show `✔ connected`
> 4. **Try**: "List my Stitch projects" or `/stitch list projects`

---

## 9. HOW TO USE IN NEW SESSIONS

**Slash command:**
```
/stitch list my projects
/stitch read design from LaunchPad
/stitch export LaunchPad as React
```

**Natural language:**
```
"Use Stitch MCP to read my current project's design tokens"
"Fetch the color palette from my Stitch project and apply it to this component"
"Export my LaunchPad design as React code"
```

The agent will:
1. Load the skill from `~/.claude/skills/stitch/SKILL.md`
2. Call the Stitch MCP tools via the configured server
3. Return the design data or apply it to the current project

---

## 10. SKILL SUMMARY

| Field | Value |
|-------|-------|
| Integration | Google Stitch MCP |
| Website | https://stitch.withgoogle.com |
| MCP endpoint | `https://stitch.googleapis.com/mcp` |
| Auth method | API key (`X-Goog-Api-Key` header) |
| Env variable | `STITCH_API_KEY` |
| Skill location | `~/.claude/skills/stitch/SKILL.md` |
| Command location | `~/.claude/commands/stitch.md` |
| Config file | `~/.claude.json` (Claude Code MCP entry) |
| Status | Installed — restart client to activate |

### Confirm completion

Print a brief confirmation that setup is complete. Remind the user they can always type `/stitch` or ask the agent to use Stitch MCP in any session.
