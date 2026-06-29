# Academic Research Skills — Installation Prompt

You are installing the Academic Research Skills suite from https://github.com/imbad0202/academic-research-skills on this machine. Follow every step exactly.

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

\`\`\`bash
git clone https://github.com/imbad0202/academic-research-skills.git /tmp/academic-research-skill 2>/dev/null || (echo "Already cloned, pulling updates..." && cd /tmp/academic-research-skill && git pull)
\`\`\`

Verify:

\`\`\`bash
ls /tmp/academic-research-skill/
\`\`\`

---

## 3. INSTALL THE SKILL

Copy the skill files to the appropriate location for the detected AI client:

### Claude Code

\`\`\`bash
mkdir -p ~/.claude/skills/academic-research
cp -r /tmp/academic-research-skill/* ~/.claude/skills/academic-research/
\`\`\`

### Cursor / Windsurf / Zed / Amp

\`\`\`bash
mkdir -p ~/.agents/skills/academic-research
cp -r /tmp/academic-research-skill/* ~/.agents/skills/academic-research/
\`\`\`

### Gemini

\`\`\`bash
mkdir -p ~/.gemini/skills/academic-research
cp -r /tmp/academic-research-skill/* ~/.gemini/skills/academic-research/
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

> **Please quit and restart your AI coding client now** (Claude Code, Cursor, etc.). This ensures the new skill is loaded and available for use.

The skill files are stored at \`~/.claude/skills/academic-research/\` (or \`~/.agents/skills/academic-research/\`). After restarting, the Academic Research Skills suite will be available.

---

## 6. VERIFY INSTALLATION

After restart, verify the skill is available:

For Claude Code: \`claude skills list\` or ask "List my available skills."

Expected: \`academic-research\` (or its sub-skills \`deep-research\`, \`academic-paper\`, \`academic-paper-reviewer\`, \`academic-pipeline\`) should appear.

---

## 7. SKILL SUMMARY

| Field | Value |
|-------|-------|
| Skill name | Academic Research Skills |
| Source | https://github.com/imbad0202/academic-research-skills |
| Installed at | \`~/.claude/skills/academic-research/\` |
| Status | Installed — restart client to activate |

### Confirm completion

Print a brief confirmation that installation is complete. Ask the user what they'd like to research — start with \`/ars-plan\` for a Socratic paper planning dialogue or \`/ars-lit-review "topic"\` for a literature review.
\\