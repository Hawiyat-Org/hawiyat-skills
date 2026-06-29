# Claude Skills (Jeffallan) — Installation Prompt

You are installing the Claude Skills plugin from https://github.com/Jeffallan/claude-skills on this machine. Follow every step exactly.

> **Agent instructions:** This document is a sequential installation guide. Work through each section in order. Do not skip steps. Do not assume anything — verify first.

---

## 1. DETECT THE ENVIRONMENT

Determine the following:

- **OS**: \`uname -a\` (Linux), \`sw_vers\` (macOS), \`ver\` (Windows)
- **Home directory**: \`echo $HOME\`
- **Current working directory**: \`pwd\`
- **AI client/agent**: Check for config directories:
  \`\`\`
  ~/.claude/       → Claude Code
  ~/.agents/       → Cursor / Windsurf / Zed / Amp
  ~/.gemini/       → Gemini
  \`\`\`

Report findings before continuing.

---

## 2. INSTALL THE SKILLS PLUGIN

Use the Claude Code plugin marketplace to install:

\`\`\`bash
/plugin marketplace add jeffallan/claude-skills
\`\`\`

If the plugin marketplace command is not available, install via the alternative method:

\`\`\`bash
/plugin install fullstack-dev-skills@jeffallan
\`\`\`

Verify installation:

\`\`\`bash
ls ~/.claude/plugins/jeffallan-claude-skills/ 2>/dev/null && echo "Plugin installed" || echo "Plugin directory not found"
\`\`\`

---

## 3. INSTALL FOR OTHER CLIENTS (MANUAL)

If using Cursor / Windsurf / Zed / Amp or other AI coding assistants, clone the repository and copy the skills:

\`\`\`bash
git clone https://github.com/Jeffallan/claude-skills.git /tmp/jeffallan-skills 2>/dev/null || (echo "Already cloned, pulling updates..." && cd /tmp/jeffallan-skills && git pull)
\`\`\`

List available skill directories:

\`\`\`bash
ls /tmp/jeffallan-skills/skills/
\`\`\`

Choose the skills you want to install:

\`\`\`bash
# For Claude Code:
mkdir -p ~/.claude/skills/jeffallan
cp -r /tmp/jeffallan-skills/skills/* ~/.claude/skills/jeffallan/

# For Cursor / Windsurf / Zed / Amp:
mkdir -p ~/.agents/skills/jeffallan
cp -r /tmp/jeffallan-skills/skills/* ~/.agents/skills/jeffallan/

# For Gemini:
mkdir -p ~/.gemini/skills/jeffallan
cp -r /tmp/jeffallan-skills/skills/* ~/.gemini/skills/jeffallan/
\`\`\`

---

## 4. RELOAD SKILLS

### Claude Code

\`\`\`bash
claude skills reload 2>/dev/null || echo "Skills directory updated. Skills will load on next restart."
\`\`\`

### Cursor / Windsurf

Skills are loaded from the \`~/.agents/skills/\` directory on startup. No reload command available — restart required.

---

## 5. RESTART YOUR CLIENT

⛔ **STOP AND RESTART:** Tell the user:

> **Please quit and restart your AI coding client now** (Claude Code, Cursor, etc.). This ensures the new skills are loaded and available for use.

The skills are stored at \`~/.claude/skills/jeffallan/\` (or \`~/.agents/skills/jeffallan/\`). After restarting, all 66 specialized skills will be available across 12 categories.

---

## 6. VERIFY INSTALLATION

After restart, verify the skills are available:

For Claude Code: \`claude skills list\` or ask "List my available skills."

Test a specific skill:
\`\`\`
/nextjs-dev "Create a Next.js App Router page with server components and data fetching"
\`\`\`

Expected: The \`/nextjs-dev\` command should respond with guidance.

---

## 7. SKILL SUMMARY

| Field | Value |
|-------|-------|
| Skill name | Claude Skills (Jeffallan) |
| Source | https://github.com/Jeffallan/claude-skills |
| Total skills | 66 across 12 categories |
| Installed at | \`~/.claude/skills/jeffallan/\` |
| Status | Installed — restart client to activate |

### Confirm completion

Print a brief confirmation that installation is complete. Ask the user what they'd like to build — they have 66 specialized skills ready to help.
