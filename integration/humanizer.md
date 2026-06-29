# Humanizer Skill — Installation Prompt

You are installing the Humanizer skill from https://github.com/blader/humanizer on this machine. Follow every step exactly.

> **Agent instructions:** This document is a sequential installation guide. Work through each section in order. Do not skip steps. Do not assume anything — verify first.

---

## 1. DETECT THE ENVIRONMENT

Determine the following:

- **OS**: \`uname -a\` (Linux), \`sw_vers\` (macOS), \`ver\` (Windows)
- **Home directory**: \`echo $HOME\`
- **AI client/agent**: Check for config directories:
  \`\`\`
  ~/.claude/       → Claude Code
  ~/.agents/       → Cursor / Windsurf / Zed / Amp
  ~/.gemini/       → Gemini
  \`\`\`

Report findings before continuing.

---

## 2. FETCH THE SKILL FROM GITHUB

Fetch the skill from the repository:

\`\`\`bash
# Clone or fetch the skill repository
git clone https://github.com/blader/humanizer.git /tmp/humanizer-skill 2>/dev/null || (echo "Already cloned, pulling updates..." && cd /tmp/humanizer-skill && git pull)
\`\`\`

Verify the fetch succeeded:

\`\`\`bash
ls /tmp/humanizer-skill/
\`\`\`

---

## 3. INSTALL THE SKILL

Copy the skill files to the appropriate location for the detected AI client:

### Claude Code

\`\`\`bash
mkdir -p ~/.claude/skills/humanizer
cp -r /tmp/humanizer-skill/* ~/.claude/skills/humanizer/
\`\`\`

### Cursor / Windsurf / Zed / Amp

\`\`\`bash
mkdir -p ~/.agents/skills/humanizer
cp -r /tmp/humanizer-skill/* ~/.agents/skills/humanizer/
\`\`\`

### Gemini

\`\`\`bash
mkdir -p ~/.gemini/skills/humanizer
cp -r /tmp/humanizer-skill/* ~/.gemini/skills/humanizer/
\`\`\`

---

## 4. RELOAD SKILLS

Reload the skills configuration for the detected client:

### Claude Code

\`\`\`bash
claude skills reload 2>/dev/null || echo "Skills directory updated. Skills will load on next restart."
\`\`\`

### Cursor / Windsurf

Skills are loaded from the \`~/.agents/skills/\` directory on startup. No reload command available — restart required.

---

## 5. RESTART YOUR CLIENT

⛔ **STOP AND RESTART:** Tell the user:

> **Please quit and restart your AI coding client now** (Claude Code, Cursor, etc.). This ensures the new skill is loaded and available for use.

The skill files are stored at \`~/.claude/skills/humanizer/\` (or \`~/.agents/skills/humanizer/\`). After restarting, the Humanizer skill will be available.

---

## 6. VERIFY INSTALLATION

After restart, verify the skill is available:

For Claude Code: \`claude skills list\` or ask "List my available skills."

Expected: \`humanizer\` should appear in the list.

---

## 7. SKILL SUMMARY

| Field | Value |
|-------|-------|
| Skill name | Humanizer |
| Source | https://github.com/blader/humanizer |
| Installed at | \`~/.claude/skills/humanizer/\` |
| Status | Installed — restart client to activate |

### Confirm completion

Print a brief confirmation that installation is complete. Ask the user what they'd like to humanize first.
