# Notion Integration Setup — Universal Harness Prompt

You are setting up a Notion integration on this machine. This prompt is designed to be fed to any AI coding agent on any OS (Linux, macOS, Windows/WSL). Follow every step exactly.

> **Agent instructions:** This document is a sequential setup guide. Work through each section in order. Do not skip steps. Do not assume tools are installed — verify first. After completing all steps, inform the user of the final status.

---

## 1. DETECT THE ENVIRONMENT

Determine the following before proceeding:

- **OS**: \`uname -a\` (Linux), \`sw_vers\` (macOS), \`ver\` (Windows). For Linux, also check the distro (\`cat /etc/os-release\`).
- **Home directory**: \`echo $HOME\`
- **AI client/agent**: Check for config directories and report which one is present:
  \`\`\`
  ~/.claude/       → Claude Code
  ~/.agents/       → Cursor / Windsurf / Zed / Amp
  ~/.gemini/       → Gemini
  \`\`\`
  If multiple are present, report all and default to the first.

Report all findings before continuing.

---

## 2. CREATE A NOTION INTEGRATION TOKEN

⚠️ **STOP AND READ:** The agent **must stop here** and ask the user to create a Notion integration token.

Tell the user:

> **Go to https://app.notion.com/developers/tokens and create a new integration.**
> - Click **"+ New integration"**
> - Give it a name (e.g., "My AI Assistant")
> - Select the workspace
> - Under **Capabilities**, ensure at least \`Read content\`, \`Update content\`, \`Insert content\` are enabled
> - Click **"Submit"**
> - **Copy the "Internal Integration Secret"** (starts with \`ntn_\` or \`secret_\`)
> - Return here and paste the token

**Wait for the user to paste the token before continuing.**

Also tell the user they must **share a page or database** with this integration:
1. Open Notion
2. Go to the page/database they want the agent to access
3. Click **"..."** in the top-right → **"Add connections"** → select the integration name
4. Repeat for any other pages/databases the agent needs

---

## 3. STORE THE TOKEN

Save the token to \`~/.notion-token\` so the agent can read it later:

\`\`\`bash
echo "<paste-the-token-here>" > ~/.notion-token
chmod 600 ~/.notion-token
\`\`\`

Then set it for this session:

\`\`\`bash
export NOTION_TOKEN=$(cat ~/.notion-token)
\`\`\`

### Reload skills

Run the skills reload command so the Notion integration is recognized:

\`\`\`bash
claude skills reload 2>/dev/null || echo "Restart your client to pick up the new skill."
\`\`\`

---

## 4. VERIFY IT WORKS

\`\`\`bash
curl -s -o /dev/null -w "%{http_code}" -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" https://api.notion.com/v1/users/me
\`\`\`

Expected: \`200\`. If \`401\`, the token is invalid — ask the user to re-create it.

---

## 5. HOW TO USE THE NOTION API

Now that the token is stored at \`~/.notion-token\`, use these direct Notion API calls whenever the user needs Notion operations. Read the token from the file each time. Do NOT use any MCP tools.

### Headers for every request

\`\`\`
Authorization: Bearer $(cat ~/.notion-token)
Notion-Version: 2022-06-28
Content-Type: application/json
\`\`\`

### List users

\`\`\`bash
curl -s -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" https://api.notion.com/v1/users
\`\`\`

### Search pages / databases

\`\`\`bash
curl -s -X POST -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" -H "Content-Type: application/json" https://api.notion.com/v1/search -d '{"query":"<search-term>","page_size":10}'
\`\`\`

### Retrieve a page

\`\`\`bash
curl -s -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" https://api.notion.com/v1/pages/<page-id>
\`\`\`

### Create a page

\`\`\`bash
curl -s -X POST -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" -H "Content-Type: application/json" https://api.notion.com/v1/pages -d '{
  "parent": { "type": "page_id", "page_id": "<parent-id>" },
  "properties": {
    "title": { "title": [{ "text": { "content": "My New Page" } }] }
  }
}'
\`\`\`

### Query a database

\`\`\`bash
curl -s -X POST -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" -H "Content-Type: application/json" https://api.notion.com/v1/databases/<database-id>/query -d '{"page_size":10}'
\`\`\`

### Get block children (page content)

\`\`\`bash
curl -s -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" https://api.notion.com/v1/blocks/<block-id>/children
\`\`\`

### Append blocks

\`\`\`bash
curl -s -X PATCH -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" -H "Content-Type: application/json" https://api.notion.com/v1/blocks/<block-id>/children -d '{
  "children": [
    { "object": "block", "type": "paragraph", "paragraph": { "rich_text": [{ "type": "text", "text": { "content": "Hello from AI!" } }] } }
  ]
}'
\`\`\`

---

## 6. FINAL VERIFICATION AND SUMMARY

Run final verification:

\`\`\`bash
TOKEN=$(cat ~/.notion-token)
echo "$(echo $TOKEN | head -c 10)... (token set)"
curl -s -o /dev/null -w "API status: %{http_code}\\n" -H "Authorization: Bearer $TOKEN" -H "Notion-Version: 2022-06-28" https://api.notion.com/v1/users/me
\`\`\`

Then print the summary table:

| Field | Value |
|-------|-------|
| OS detected | ... |
| AI client(s) detected | ... |
| Notion token | Stored in \`~/.notion-token\` |
| Connection method | Direct Notion API via curl |
| API version | 2022-06-28 |
| Integration docs | https://app.notion.com/developers/tokens |

### Confirm completion

Print a brief confirmation that setup is complete.

---

## 7. CREATE THE CLAUDE CODE SKILL

This is the critical step. Create a skill directory and SKILL.md so Claude Code recognizes the integration:

\`\`\`bash
mkdir -p ~/.claude/skills/notion
\`\`\`

Write the following content to \`~/.claude/skills/notion/SKILL.md\`:

\`\`\`
cat > ~/.claude/skills/notion/SKILL.md << 'SKILLEOF'
---
name: notion
description: "Complete Notion API integration — create/read/update/delete pages, query databases, manage blocks, search, comments, media, templates, and automation. Token stored at ~/.notion-token. Activates on /notion or intent like 'create a page in Notion', 'query my database', 'search Notion for X'"
---
# Notion Integration
Full programmatic access to Notion via the official API. Read/write pages, databases, blocks, comments, and more.
## Setup
Token is stored at \`~/.notion-token\`. Verify it works:
\`\`\`bash
curl -s -o /dev/null -w "%{http_code}" -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" https://api.notion.com/v1/users/me
\`\`\`
Expected: \`200\`
## Auth Headers (use on every request)
\`\`\`
Authorization: Bearer $(cat ~/.notion-token)
Notion-Version: 2022-06-28
Content-Type: application/json
\`\`\`
## Available Operations
### Pages
- **List all pages**: \`POST /search\` with \`filter: {value: "page", property: "object"}\`
- **Get a page**: \`GET /pages/{id}\`
- **Create a page**: \`POST /pages\` with parent + properties + optional children
- **Update page**: \`PATCH /pages/{id}\` with properties to change
- **Archive page**: \`PATCH /pages/{id}\` with \`{"archived": true}\`
- **Restore page**: \`PATCH /pages/{id}\` with \`{"archived": false}\`
### Databases
- **List databases**: \`POST /search\` with \`filter: {value: "database", property: "object"}\`
- **Get database**: \`GET /databases/{id}\`
- **Create database**: \`POST /databases\` with parent + properties schema
- **Query database**: \`POST /databases/{id}/query\` with optional filter + sort
- **Update schema**: \`PATCH /databases/{id}\` with properties changes
### Blocks (Page Content)
- **Get children**: \`GET /blocks/{id}/children\`
- **Append blocks**: \`PATCH /blocks/{id}/children\` with children array
- **Delete block**: \`DELETE /blocks/{id}\`
**Block types**: heading_1, heading_2, heading_3, paragraph, bulleted_list_item, numbered_list_item, to_do, toggle, quote, callout, code, divider, table_of_contents, bookmark, image, video, audio, embed, file, equation, column_list, column, table, table_row, synced_block, link_preview, breadcrumb
### Search
- **Global search**: \`POST /search\` with query + filter
- **Sort results**: Add \`sort: {direction: "descending", timestamp: "last_edited_time"}\`
### Comments
- **List comments**: \`POST /comments\` with block_id
- **Add comment**: \`POST /comments\` with parent + rich_text
### Users
- **List users**: \`GET /users\`
- **Current bot**: \`GET /users/me\`
## Property Types
| Type | Set Format | Read Path |
|------|-----------|-----------|
| title | \`{"title": [{"text": {"content": "X"}}]}\` | \`.title[0].text.content\` |
| rich_text | \`{"rich_text": [{"text": {"content": "X"}}]}\` | \`.rich_text[0].text.content\` |
| number | \`{"number": 42}\` | \`.number\` |
| select | \`{"select": {"name": "X"}}\` | \`.select.name\` |
| multi_select | \`{"multi_select": [{"name": "A"}, {"name": "B"}]}\` | \`.multi_select[].name\` |
| status | \`{"status": {"name": "X"}}\` | \`.status.name\` |
| date | \`{"date": {"start": "2026-06-26"}}\` | \`.date.start\` |
| checkbox | \`{"checkbox": true}\` | \`.checkbox\` |
| url | \`{"url": "https://..."}\` | \`.url\` |
| people | \`{"people": [{"object":"user","id":"ID"}]}\` | \`.people[0].name\` |
| relation | \`{"relation": [{"id": "PAGE_ID"}]}\` | \`.relation[0].id\` |
## Filter Examples
\`\`\`json
{"property": "Status", "status": {"equals": "Done"}}
{"property": "Name", "title": {"contains": "search"}}
{"property": "Date", "date": {"on_or_after": "2026-06-01"}}
{"and": [{"property": "Status", "status": {"equals": "In progress"}}, {"property": "Priority", "select": {"equals": "High"}}]}
\`\`\`
## Rich Text Formatting
\`\`\`json
{"type": "text", "text": {"content": "bold"}, "annotations": {"bold": true}}
{"type": "text", "text": {"content": "click", "link": {"url": "https://..."}}}
\`\`\`
## Tips
- Always verify token with \`/users/me\` first
- Use \`jq\` to parse JSON responses
- Page IDs and database IDs are UUIDs (32 hex chars with dashes)
- The \`title\` property name varies by database — check with \`GET /databases/{id}\` first
- Use \`filter\` + \`page_size\` to avoid fetching entire databases
- Archive before delete — Notion has no hard delete via API
SKILLEOF
\`\`\`

### Verify the skill file exists

\`\`\`bash
ls -la ~/.claude/skills/notion/SKILL.md
\`\`\`

---

## 8. CREATE THE /NOTION COMMAND FILE

Create a slash command so users can type \`/notion\` in Claude Code:

\`\`\`bash
cat > ~/.claude/commands/notion.md << 'CMDEOF'
---
description: "Notion API integration — search, create, read, update pages/databases/blocks. Usage: /notion [task]"
---
Load the Notion skill from \`~/.claude/skills/notion/SKILL.md\` and execute the user's request using the Notion API.
**Arguments**: $ARGUMENTS
If no arguments provided, show a summary of available operations and ask what the user wants to do.

## Quick Reference
- Token: \`~/.notion-token\`
- API: \`https://api.notion.com/v1\`
- Version: \`2022-06-28\`
- Auth: \`Authorization: Bearer $(cat ~/.notion-token)\`

## Common Tasks
| Task | Command |
|------|---------|
| Search all pages | \`POST /search\` with \`filter: {value: "page", property: "object"}\` |
| Search databases | \`POST /search\` with \`filter: {value: "database", property: "object"}\` |
| Get page | \`GET /pages/{id}\` |
| Create page | \`POST /pages\` |
| Query database | \`POST /databases/{id}/query\` |
| Get page content | \`GET /blocks/{id}/children\` |
| Add content | \`PATCH /blocks/{id}/children\` |
| List users | \`GET /users\` |

Read the full skill file at \`~/.claude/skills/notion/SKILL.md\` for complete reference.
CMDEOF
\`\`\`

### Verify

\`\`\`bash
ls -la ~/.claude/commands/notion.md
\`\`\`

---

## 9. CREATE MEMORY FILES (for cross-session persistence)

Create memory files so the integration works in future sessions:

\`\`\`bash
MEMORY_DIR=$(find ~/.claude/projects -name "MEMORY.md" -path "*/memory/*" -exec dirname {} \\; 2>/dev/null | head -1)
if [ -z "$MEMORY_DIR" ]; then
  MEMORY_DIR="$HOME/.claude/memory"
  mkdir -p "$MEMORY_DIR"
fi
echo "Memory directory: $MEMORY_DIR"
\`\`\`

Write the index file:

\`\`\`
cat > "$MEMORY_DIR/notion-skills-index.md" << 'MEMEOF'
---
name: notion-skills-index
description: Master index of all Notion API skills
metadata:
  type: reference
---
# Notion Skills Master Index
Token: \`~/.notion-token\` (API version: 2022-06-28)
## Quick Reference
Authorization: Bearer $(cat ~/.notion-token)
Notion-Version: 2022-06-28
Content-Type: application/json
MEMEOF
\`\`\`

Write the remaining memory files:

\`\`\`bash
# notion-pages.md
cat > "$MEMORY_DIR/notion-pages.md" << 'MEMEOF'
---
name: notion-pages
description: Notion page CRUD operations
---
curl -s -X POST -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" -H "Content-Type: application/json" https://api.notion.com/v1/search -d '{"filter":{"value":"page","property":"object"},"page_size":100}'
curl -s -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" https://api.notion.com/v1/pages/{page_id}
MEMEOF

# notion-databases.md
cat > "$MEMORY_DIR/notion-databases.md" << 'MEMEOF'
---
name: notion-databases
description: Notion database operations — create, query, filter, sort
---
curl -s -X POST -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" -H "Content-Type: application/json" https://api.notion.com/v1/databases/{database_id}/query -d '{"page_size": 100}'
MEMEOF

# notion-blocks.md
cat > "$MEMORY_DIR/notion-blocks.md" << 'MEMEOF'
---
name: notion-blocks
description: Notion block types
---
curl -s -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" https://api.notion.com/v1/blocks/{block_id}/children
curl -s -X PATCH -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" -H "Content-Type: application/json" https://api.notion.com/v1/blocks/{block_id}/children -d '{"children":[...]}'
MEMEOF

# notion-search.md
cat > "$MEMORY_DIR/notion-search.md" << 'MEMEOF'
---
name: notion-search
description: Notion search operations
---
curl -s -X POST -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" -H "Content-Type: application/json" https://api.notion.com/v1/search -d '{"query":"meeting notes","page_size":20}'
MEMEOF

# notion-comments.md
cat > "$MEMORY_DIR/notion-comments.md" << 'MEMEOF'
---
name: notion-comments
description: Notion comment operations
---
curl -s -X POST -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" -H "Content-Type: application/json" https://api.notion.com/v1/comments -d '{"parent":{"type":"page_id","page_id":"PAGE_ID"},"rich_text":[{"type":"text","text":{"content":"This is a comment"}}]}'
MEMEOF

# notion-templates.md
cat > "$MEMORY_DIR/notion-templates.md" << 'MEMEOF'
---
name: notion-templates
description: Notion page and database templates
---
# Meeting Notes, Task Tracker, Project Wiki
POST /pages with children: heading sections + bulleted lists + to_do items
POST /databases with Task, Status, Priority, Assignee, Due Date
MEMEOF

# notion-automation.md
cat > "$MEMORY_DIR/notion-automation.md" << 'MEMEOF'
---
name: notion-automation
description: Notion batch operations and automation
---
curl -s -X POST -H "Authorization: Bearer $(cat ~/.notion-token)" -H "Notion-Version: 2022-06-28" -H "Content-Type: application/json" https://api.notion.com/v1/search -d '{"filter":{"value":"page","property":"object"},"page_size":100}' > notion_pages.json
MEMEOF

# notion-properties.md
cat > "$MEMORY_DIR/notion-properties.md" << 'MEMEOF'
---
name: notion-properties
description: Notion property types reference
---
title: {"title":[{"text":{"content":"X"}}]} → .title[0].text.content
select: {"select":{"name":"X"}} → .select.name
status: {"status":{"name":"X"}} → .status.name
date: {"date":{"start":"2026-06-26"}} → .date.start
checkbox: {"checkbox":true} → .checkbox
url: {"url":"https://..."} → .url
people: {"people":[{"object":"user","id":"ID"}]} → .people[0].name
relation: {"relation":[{"id":"PAGE_ID"}]} → .relation[0].id
MEMEOF

# notion-advanced.md
cat > "$MEMORY_DIR/notion-advanced.md" << 'MEMEOF'
---
name: notion-advanced
description: Notion advanced operations
---
# Synced blocks, columns, complex filters
{"and":[{"property":"Status","status":{"does_not_equal":"Done"}},{"or":[{"property":"Priority","select":{"equals":"High"}},{"property":"Priority","select":{"equals":"Critical"}}]}]}
MEMEOF
\`\`\`

### Update MEMORY.md index

\`\`\`bash
if ! grep -q "notion-skills-index" "$MEMORY_DIR/../MEMORY.md" 2>/dev/null; then
  cat >> "$MEMORY_DIR/../MEMORY.md" << 'IDXEOF'
- [Notion Skills Index](notion-skills/notion-skills-index.md) — Master index
- [Notion Pages](notion-skills/notion-pages.md) — CRUD pages
- [Notion Databases](notion-skills/notion-databases.md) — Query and filter
- [Notion Blocks](notion-skills/notion-blocks.md) — Page content
- [Notion Search](notion-skills/notion-search.md) — Search
- [Notion Comments](notion-skills/notion-comments.md) — Comments
- [Notion Templates](notion-skills/notion-templates.md) — Templates
- [Notion Automation](notion-skills/notion-automation.md) — Automation
- [Notion Properties](notion-skills/notion-properties.md) — Property types
- [Notion Advanced](notion-skills/notion-advanced.md) — Advanced
IDXEOF
fi
\`\`\`

---

## 10. RESTART INSTRUCTIONS

⚠️ **\`claude skills reload\` does NOT work reliably.** The user MUST restart Claude Code for \`/notion\` to appear.

Tell the user:

> **Setup is complete! To activate \`/notion\`:**
>
> 1. **Close this Claude Code session** (type \`/exit\` or close the window)
> 2. **Reopen Claude Code** in the same directory
> 3. **Type \`/notion\`** — it should now appear in the command list
>
> If \`/notion\` still doesn't appear after restart, re-run this setup.

---

## 11. HOW TO USE IN NEW SESSIONS

After restart, the user can:

**Slash command:**
\`\`\`
/notion search for meeting notes
/notion create a task database
/notion query my project board for high priority items
\`\`\`

**Natural language:**
\`\`\`
"Use my Notion skills to create a page"
"Search Notion for pages about machine learning"
\`\`\`

The agent will:
1. Load the skill from \`~/.claude/skills/notion/SKILL.md\`
2. Read the token from \`~/.notion-token\`
3. Execute the Notion API call via curl
4. Return the results
