# UI/UX Pro Max Skill — Installation Prompt

You are installing the UI/UX Pro Max design intelligence skill from https://github.com/nextlevelbuilder/ui-ux-pro-max-skill on this machine. Follow every step exactly.

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
- **Python**: \`python3 --version\` (required for the search engine)

Report findings before continuing.

---

## 2. INSTALL THE UI/UX PRO MAX CLI

The skill is distributed as an npm package:

\`\`\`bash
npm install -g ui-ux-pro-max-cli
\`\`\`

Verify:

\`\`\`bash
uipro --version
\`\`\`

---

## 3. INITIALIZE THE SKILL

\`\`\`bash
uipro init --ai opencode
\`\`\`

This installs the SKILL.md, data CSVs, and Python scripts into your project's skills directory.

If using Claude Code instead:

\`\`\`bash
uipro init --ai claude
\`\`\`

---

## 4. INSTALL FOR OTHER CLIENTS (MANUAL)

If the \`uipro init\` command does not support your client, fetch from GitHub and copy:

\`\`\`bash
git clone https://github.com/nextlevelbuilder/ui-ux-pro-max-skill.git /tmp/ui-ux-pro-max 2>/dev/null || (echo "Already cloned, pulling updates..." && cd /tmp/ui-ux-pro-max && git pull)
\`\`\`

\`\`\`bash
# For Claude Code:
mkdir -p ~/.claude/skills/ui-ux-pro-max
cp -r /tmp/ui-ux-pro-max/src/* ~/.claude/skills/ui-ux-pro-max/

# For Cursor / Windsurf / Zed / Amp:
mkdir -p ~/.agents/skills/ui-ux-pro-max
cp -r /tmp/ui-ux-pro-max/src/* ~/.agents/skills/ui-ux-pro-max/

# For Gemini:
mkdir -p ~/.gemini/skills/ui-ux-pro-max
cp -r /tmp/ui-ux-pro-max/src/* ~/.gemini/skills/ui-ux-pro-max/
\`\`\`

---

## 5. VERIFY PYTHON DEPENDENCIES

The search engine requires Python with BM25 support:

\`\`\`bash
python3 -c "from rank_bm25 import BM25Okapi; print('BM25 OK')" 2>/dev/null || pip install rank-bm25
\`\`\`

---

## 6. TEST THE SKILL

Run a test search to confirm everything works:

\`\`\`bash
python3 ~/.claude/skills/ui-ux-pro-max/scripts/search.py "beauty spa wellness" --domain style
\`\`\`

Expected output: a design system recommendation with UI style, colors, typography, and effects.

---

## 7. RELOAD SKILLS

### Claude Code

\`\`\`bash
claude skills reload 2>/dev/null || echo "Skills directory updated. Skills will load on next restart."
\`\`\`

### Cursor / Windsurf

Skills are loaded from the \`~/.agents/skills/\` directory on startup. No reload command available — restart required.

---

## 8. RESTART YOUR CLIENT

⛔ **STOP AND RESTART:** Tell the user:

> **Please quit and restart your AI coding client now** (Claude Code, Cursor, etc.). This ensures the new skill is loaded and available for use.

The skill files are stored at \`~/.claude/skills/ui-ux-pro-max/\` (or \`~/.agents/skills/ui-ux-pro-max/\`). After restarting, the UI/UX Pro Max skill will be available for generating design systems.

---

## 9. VERIFY INSTALLATION

After restart, verify the skill is available:

For Claude Code: \`claude skills list\` or ask "List my available skills."

---

## 10. HOW TO USE

**Generate a design system:**
\`\`\`
ui-ux-pro-max design-system "e-commerce fashion" --name "StyleVault"
\`\`\`

**Search specific domains:**
\`\`\`
ui-ux-pro-max search "banking dashboard" --domain ux
ui-ux-pro-max search "fintech app" --domain color
\`\`\`

**Get stack-specific guidelines:**
\`\`\`
ui-ux-pro-max stack nextjs "landing page design system"
\`\`\`

---

## 11. SKILL SUMMARY

| Field | Value |
|-------|-------|
| Skill name | UI/UX Pro Max |
| Source | https://github.com/nextlevelbuilder/ui-ux-pro-max-skill |
| Installed at | \`~/.claude/skills/ui-ux-pro-max/\` |
| Status | Installed — restart client to activate |

### Confirm completion

Print a brief confirmation that installation is complete. Ask the user what they'd like to design — they have 67 UI styles and 161 color palettes ready.
