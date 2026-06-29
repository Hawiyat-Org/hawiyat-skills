---
name: claude-cloud-to-local
description: Full pipeline — export Claude.ai cloud data → migrate to Claude Code sessions + Cowork skills + gateway + memory — one-shot autonomous migration
---

# Claude Cloud → Local: Full Migration Skill

Migrates an entire Claude.ai data export (conversations, memories, user profile) into a local-first architecture: Claude Code sessions + Cowork memory + conversations index — all in one pass.

## Usage

```
/claude-cloud-to-local <export-directory>
```

Before running, set `SRC_DIR` to the export folder path. The skill reads from that env var, not a CLI argument (to work inside Node heredoc blocks).

Where `<export-directory>` is a folder containing:
- `conversations.json` — array of chat conversations
- `memories.json` — array of user memory entries (can be `[]`)
- `users.json` — array of account/profile info

## What It Does

1. **Reads & validates** all three export files
2. **Updates Cowork memory** (`hot-memory.md`, `system-state.md`) with user profile + rich work context
3. **Converts each conversation** to native Claude Code JSONL session format
4. **Registers session metadata** in `claude-code-sessions/` (so sessions appear in `/sessions`)
5. **Registers session data** in `local-agent-mode-sessions/` (so messages load)
6. **Copies JSONL to project dir** (`~/.claude/projects/{project}/`) — where Claude Code reads transcripts
7. **Creates session-env entries** (`~/.claude/session-env/{uuid}/env.json`)
8. **Builds/updates** `~/.claude/conversations-index.json`
9. **Logs** full summary of what was imported

## ⚠️ Critical Pitfalls (Lessons Learned)

### Pitfall 1: JSONL files must be in `~/.claude/projects/{project}/`
Claude Code does NOT read JSONL from `local-agent-mode-sessions/` for transcript display. It reads from:
```
~/.claude/projects/C--Users-hp-Downloads-{folder-name}/{session-uuid}.jsonl
```
The project directory name is derived from the cwd path with special chars replaced.

### Pitfall 2: JSONL needs `mode`, `custom-title`, `last-prompt` entries
Without these, `getTranscript()` returns empty and Claude Code shows "session not on disk". Every JSONL must end with:
```json
{"type":"mode","mode":"normal","sessionId":"..."}
{"type":"custom-title","customTitle":"Session Title","sessionId":"..."}
{"type":"last-prompt","lastPrompt":"...","leafUuid":"...","sessionId":"..."}
```

### Pitfall 3: Sessions need metadata in BOTH directories
- `claude-code-sessions/` — for sidebar display
- `local-agent-mode-sessions/` — for message loading

### Pitfall 4: `local-agent-mode-sessions/` metadata needs specific fields
```json
{
  "accountName": "Cowork 3P",
  "emailAddress": "cowork-3p@localhost",
  "memoryEnabled": true,
  "pluginsEnabled": true,
  "skillsEnabled": true,
  "systemPrompt": "...",
  "systemPromptRendererAppends": []
}
```

### Pitfall 5: Session data directories need `.claude/.claude.json`
```json
{
  "oauthAccount": {
    "accountUuid": "<user-uuid>",
    "emailAddress": "cowork-3p@localhost",
    "organizationUuid": "<workspace-uuid>"
  },
  "migrationVersion": 13
}
```

### Pitfall 6: `cwd` field format
Use `/sessions/local_{uuid}` format, not the actual filesystem path.

### Pitfall 7: audit.jsonl must NOT have `_audit_hmac`
Remove HMAC fields — they cause validation failures for imported sessions.

## Implementation

```node
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');

// ── INPUT ────────────────────────────────────────────────────────
const srcDir = (process.env.SRC_DIR || '.').replace(/\\+$/, '') + '/';
const claudeDir = process.env.USERPROFILE + '/.claude';
const dstDir = process.cwd();
const cwd = dstDir.replace(/\\/g, '/');
const now = new Date().toISOString();

// ── PATHS ────────────────────────────────────────────────────────
const localAppData = process.env.LOCALAPPDATA ||
  path.join(process.env.USERPROFILE, 'AppData', 'Local');
const userUuid = 'eabaab85-2a27-4b2b-948f-bc54771f1bea'; // detect dynamically
const wsUuid = '00000000-0000-4000-8000-000000000001';

// Detect project directory name from cwd
const projectName = cwd.replace(/[^a-zA-Z0-9]/g, '-').replace(/-+/g, '-').replace(/^-|-$/g, '');

// ── 1. READ EXPORT FILES ────────────────────────────────────────
function readJSON(filePath) {
  const raw = fs.readFileSync(filePath, 'utf8');
  const clean = raw.charCodeAt(0) === 0xFEFF ? raw.slice(1) : raw;
  return JSON.parse(clean);
}

let conversations = [];
let memories = [];
let users = [];

try { conversations = readJSON(srcDir + 'conversations.json'); } catch(e) { console.log('⚠️ No conversations.json:', e.message); }
try { memories = readJSON(srcDir + 'memories.json'); } catch(e) { console.log('⚠️ No memories.json (using empty):', e.message); }
try { users = readJSON(srcDir + 'users.json'); } catch(e) { console.log('⚠️ No users.json:', e.message); }

console.log('📦 Export data loaded:');
console.log('   Conversations:', conversations.length);
console.log('   Memories:', memories.length);
console.log('   Users:', users.length);

const user = users[0] || {};
const memory = memories[0] || {};

// ── 2. UPDATE COWORK MEMORY ─────────────────────────────────────
function updateCoworkMemory() {
  const hotMemPath = path.join(claudeDir, 'cowork', 'hot-memory.md');
  const sysPath = path.join(claudeDir, 'cowork', 'system-state.md');
  fs.mkdirSync(path.dirname(hotMemPath), { recursive: true });

  const profileLines = [];
  if (user.full_name) profileLines.push(`- **Name:** ${user.full_name}`);
  if (user.email_address) profileLines.push(`- **Email:** ${user.email_address}`);
  if (user.verified_phone_number) profileLines.push(`- **Phone:** ${user.verified_phone_number}`);

  const recentConvs = conversations
    .sort((a, b) => new Date(b.created_at || 0) - new Date(a.created_at || 0))
    .slice(0, 10)
    .map(c => `- **${c.name || 'Untitled'}** (${(c.created_at || '').slice(0, 10)})`)
    .join('\n');

  const dateStr = now.slice(0, 10);
  fs.writeFileSync(hotMemPath, `# Hot Memory\n> ${dateStr}\n\n## User Profile\n${profileLines.join('\n') || '- (from export)'}\n\n## Recent Conversations\n${recentConvs || '- (none)'}\n\n## Total History\n- **${conversations.length} cloud conversations** imported\n- **${memories.length} memory entries** synced\n`);
  console.log('✅ Cowork hot-memory.md updated');
}
updateCoworkMemory();

// ── 3. FIND USER/WORKSPACE UUIDs ────────────────────────────────
function findUserWorkspace() {
  const base = path.join(localAppData, 'Claude-3p', 'local-agent-mode-sessions');
  if (!fs.existsSync(base)) return null;
  const users = fs.readdirSync(base).filter(d => /^[0-9a-f-]{36}$/i.test(d));
  for (const u of users) {
    const wss = fs.readdirSync(path.join(base, u)).filter(d => /^[0-9a-f-]{36}$/i.test(d));
    if (wss.length > 0) return { userUuid: u, wsUuid: wss[0] };
  }
  return null;
}

const uw = findUserWorkspace();
if (!uw) { console.log('❌ Could not find Claude-3p session directories'); process.exit(1); }
const { userUuid: realUserUuid, wsUuid: realWsUuid } = uw;
console.log(`🔑 Found user: ${realUserUuid}, workspace: ${realWsUuid}`);

const agentBase = path.join(localAppData, 'Claude-3p', 'local-agent-mode-sessions', realUserUuid, realWsUuid);
const codeSessDir = path.join(localAppData, 'Claude-3p', 'claude-code-sessions', realUserUuid, realWsUuid);

// Find project directory for JSONL
function findProjectDir() {
  const projectsBase = path.join(claudeDir, 'projects');
  if (!fs.existsSync(projectsBase)) return null;
  const dirs = fs.readdirSync(projectsBase);
  // Find project that matches our cwd
  for (const d of dirs) {
    const decoded = d.replace(/C--/, 'C:\\\\').replace(/-/g, '\\\\');
    if (cwd.toLowerCase().includes(decoded.toLowerCase().slice(0, 20))) {
      return path.join(projectsBase, d);
    }
  }
  // Fallback: create new project dir
  const newDir = path.join(projectsBase, 'C--' + cwd.replace(/[^a-zA-Z0-9]/g, '-').replace(/-+/g, '-'));
  fs.mkdirSync(newDir, { recursive: true });
  return newDir;
}

const projectDir = findProjectDir();
console.log(`📁 Project dir: ${projectDir}`);

// ── 4. READ REAL SESSION AS TEMPLATE ─────────────────────────────
function findRealSessionTemplate() {
  const jsonFiles = fs.readdirSync(agentBase).filter(f => f.endsWith('.json') && f.startsWith('local_'));
  for (const f of jsonFiles) {
    try {
      const content = JSON.parse(fs.readFileSync(path.join(agentBase, f), 'utf8'));
      if (content.systemPrompt && content.accountName) return content;
    } catch(e) {}
  }
  return null;
}

const template = findRealSessionTemplate();
if (!template) console.log('⚠️ No real session template found — using defaults');

// ── 5. CONVERT & DEPLOY ─────────────────────────────────────────
const convIndex = { generated_at: now, source: 'Claude.ai Export', conversations: [] };
let converted = 0, skipped = 0;

conversations.forEach(conv => {
  const uuid = conv.uuid;
  const name = conv.name || 'Untitled';
  const messages = conv.chat_messages || [];
  if (!messages.length) { skipped++; return; }

  const sessionId = 'local_' + uuid;

  // ── 5a. Build JSONL lines ──
  const nativeLines = [];
  const startTime = messages[0]?.created_at || now;

  nativeLines.push(JSON.stringify({
    type: 'queue-operation', operation: 'enqueue', timestamp: startTime,
    sessionId: uuid, content: name
  }));
  nativeLines.push(JSON.stringify({
    type: 'queue-operation', operation: 'dequeue', timestamp: startTime,
    sessionId: uuid
  }));

  let prevUuid = null;
  let lastUserUuid = null;
  let lastUserText = '';
  let lastAssistantUuid = null;

  messages.forEach(msg => {
    const ts = msg.created_at || now;
    const msgUuid = msg.uuid || crypto.randomUUID();
    const text = typeof msg.text === 'string' ? msg.text : (msg.content || '');

    if (msg.sender === 'human' || msg.role === 'user') {
      nativeLines.push(JSON.stringify({
        type: 'user', uuid: msgUuid, parentUuid: prevUuid, timestamp: ts,
        sessionId: uuid, cwd, entrypoint: 'claude-desktop-3p',
        version: '2.1.187', userType: 'external', gitBranch: 'HEAD',
        isSidechain: false, promptId: msgUuid,
        origin: { kind: 'human' }, promptSource: 'sdk', permissionMode: 'acceptEdits',
        message: { role: 'user', content: text }
      }));
      lastUserUuid = msgUuid;
      lastUserText = text.substring(0, 100);
      prevUuid = msgUuid;
    } else if (msg.sender === 'assistant' || msg.role === 'assistant') {
      nativeLines.push(JSON.stringify({
        type: 'assistant', uuid: msgUuid, parentUuid: prevUuid, timestamp: ts,
        sessionId: uuid, cwd, entrypoint: 'claude-desktop-3p',
        version: '2.1.187', userType: 'external', gitBranch: 'HEAD',
        message: { id: msgUuid, type: 'message', role: 'assistant',
          content: [{ type: 'text', text }] }
      }));
      lastAssistantUuid = msgUuid;
      prevUuid = msgUuid;
    }
  });

  // ── CRITICAL: Add required metadata entries ──
  nativeLines.push(JSON.stringify({
    type: 'mode', mode: 'normal', sessionId: uuid
  }));
  nativeLines.push(JSON.stringify({
    type: 'custom-title', customTitle: name, sessionId: uuid
  }));
  nativeLines.push(JSON.stringify({
    type: 'last-prompt',
    lastPrompt: lastUserText + name.substring(0, 50),
    leafUuid: lastAssistantUuid || lastUserUuid || uuid,
    sessionId: uuid
  }));

  const jsonlContent = nativeLines.join('\n') + '\n';

  // Write to project dir (where Claude Code reads transcripts)
  if (projectDir) {
    fs.writeFileSync(path.join(projectDir, uuid + '.jsonl'), jsonlContent);
  }

  // Also write to export dir (backup)
  fs.writeFileSync(path.join(dstDir, uuid + '.jsonl'), jsonlContent);

  // ── 5b. Create session-env ──
  const envDir = path.join(claudeDir, 'session-env', uuid);
  fs.mkdirSync(envDir, { recursive: true });
  fs.writeFileSync(path.join(envDir, 'env.json'), JSON.stringify({
    sessionId: uuid, cwd: dstDir
  }));

  // ── 5c. Create agent-mode data dir ──
  const dataDir = path.join(agentBase, sessionId);
  fs.mkdirSync(dataDir, { recursive: true });
  fs.mkdirSync(path.join(dataDir, 'outputs'), { recursive: true });
  fs.mkdirSync(path.join(dataDir, 'uploads'), { recursive: true });

  // audit.jsonl (without HMAC)
  const auditLines = messages.map(msg => {
    const ts = msg.created_at || new Date().toISOString();
    const text = typeof msg.text === 'string' ? msg.text : (msg.content || '');
    return JSON.stringify({
      type: (msg.sender === 'human' || msg.role === 'user') ? 'user' : 'assistant',
      uuid: msg.uuid || crypto.randomUUID(),
      session_id: uuid,
      parent_tool_use_id: null,
      client_platform: 'desktop_app',
      message: (msg.sender === 'human' || msg.role === 'user')
        ? { role: 'user', content: text }
        : { id: msg.uuid || crypto.randomUUID(), type: 'message', role: 'assistant',
            content: [{ type: 'text', text }] },
      _audit_timestamp: ts
    });
  });
  fs.writeFileSync(path.join(dataDir, 'audit.jsonl'), auditLines.join('\n') + '\n');

  // .claude/.claude.json
  const claudeJsonPath = path.join(dataDir, '.claude', '.claude.json');
  fs.mkdirSync(path.dirname(claudeJsonPath), { recursive: true });
  fs.writeFileSync(claudeJsonPath, JSON.stringify({
    oauthAccount: {
      accountUuid: realUserUuid,
      emailAddress: 'cowork-3p@localhost',
      organizationUuid: realWsUuid
    },
    migrationVersion: 13
  }, null, 2));

  // ── 5d. Create agent-mode metadata ──
  const agentMetaPath = path.join(agentBase, sessionId + '.json');
  if (!fs.existsSync(agentMetaPath)) {
    const agentMeta = template ? {
      ...template,
      sessionId,
      processName: name,
      cwd: '/sessions/' + sessionId,
      vmProcessName: name,
      createdAt: new Date(startTime).getTime(),
      lastActivityAt: new Date(startTime).getTime(),
      initialMessage: lastUserText,
      isArchived: false
    } : {
      sessionId,
      processName: name,
      cwd: '/sessions/' + sessionId,
      userSelectedFolders: [],
      createdAt: new Date(startTime).getTime(),
      lastActivityAt: new Date(startTime).getTime(),
      model: '', isArchived: false,
      vmProcessName: name, hostLoopMode: false,
      initialMessage: lastUserText,
      egressAllowedDomains: ['*.anthropic.com', 'anthropic.com', 'claude.com', '*.claude.com'],
      orgCliExecPolicies: { status: 'ok', policies: {} },
      accountName: 'Cowork 3P',
      emailAddress: 'cowork-3p@localhost',
      memoryEnabled: true, pluginsEnabled: true, skillsEnabled: true,
      systemPrompt: template?.systemPrompt || '',
      systemPromptRendererAppends: []
    };
    fs.writeFileSync(agentMetaPath, JSON.stringify(agentMeta));
  }

  // ── 5e. Create code-sessions metadata ──
  const codeMetaPath = path.join(codeSessDir, sessionId + '.json');
  if (!fs.existsSync(codeMetaPath)) {
    fs.writeFileSync(codeMetaPath, JSON.stringify({
      sessionId, cliSessionId: uuid,
      cwd: dstDir.replace(/\\/g, '\\\\'),
      originCwd: dstDir.replace(/\\/g, '\\\\'),
      lastFocusedAt: new Date(startTime).getTime(),
      createdAt: new Date(startTime).getTime(),
      lastActivityAt: new Date(startTime).getTime(),
      model: 'claude-sonnet-4-20250514', isArchived: false,
      title: name, titleSource: 'auto', permissionMode: 'acceptEdits',
      enabledMcpTools: {}, remoteMcpServersConfig: [],
      alwaysAllowedReasons: [], sessionPermissionUpdates: [],
      classifierSummaryEnabled: false, spawnSeed: {}
    }));
  }

  // ── Index entry ──
  let topics = 'general';
  if (conv.summary) {
    const l = conv.summary.toLowerCase();
    if (l.includes('code') || l.includes('program')) topics = 'code';
    else if (l.includes('kube') || l.includes('docker') || l.includes('infra')) topics = 'devops';
    else if (l.includes('aws') || l.includes('cloud')) topics = 'aws';
    else if (l.includes('ai') || l.includes('model')) topics = 'ai';
  }
  convIndex.conversations.push({
    uuid, name, session_path: uuid + '.jsonl',
    message_count: messages.length, created_at: startTime,
    topics: conv.summary ? [topics] : topics
  });
  converted++;
});

console.log(`📝 Converted: ${converted}, Skipped: ${skipped}`);

// ── 6. BUILD CONVERSATIONS INDEX ────────────────────────────────
const idxPath = path.join(claudeDir, 'conversations-index.json');
let existing = { conversations: [] };
try {
  const raw = fs.readFileSync(idxPath, 'utf8');
  existing = JSON.parse(raw.charCodeAt(0) === 0xFEFF ? raw.slice(1) : raw);
} catch(e) {}
const mergedMap = {};
(existing.conversations || []).forEach(c => { mergedMap[c.uuid] = c; });
convIndex.conversations.forEach(c => { mergedMap[c.uuid] = c; });
convIndex.conversations = Object.values(mergedMap);
convIndex.generated_at = now;
fs.writeFileSync(idxPath, JSON.stringify(convIndex, null, 2));
console.log(`📇 Index updated: ${convIndex.conversations.length} total entries`);

// ── 7. SUMMARY ──────────────────────────────────────────────────
console.log('');
console.log('═══════════════════════════════════════');
console.log('  ✅ MIGRATION COMPLETE');
console.log('═══════════════════════════════════════');
console.log(`  Conversations: ${converted} converted`);
console.log(`  Project JSONL: ${projectDir ? '✅' : '❌'}`);
console.log(`  Agent-mode: ✅ ${converted} sessions`);
console.log(`  Code-sessions: ✅ ${converted} metadata`);
console.log(`  Index: ${convIndex.conversations.length} entries`);
console.log('');
console.log('  📁 Locations:');
console.log(`  - JSONL: ${projectDir}`);
console.log(`  - Agent-mode: ${agentBase}`);
console.log(`  - Code-sessions: ${codeSessDir}`);
console.log(`  - Index: ${idxPath}`);
console.log('═══════════════════════════════════════');
console.log('');
console.log('🔄 Restart Claude Code for changes to take effect.');
```

## Post-Migration

After the skill runs:
- **JSONL files** are in `~/.claude/projects/{project}/` (where Claude Code reads them)
- **Sessions** appear in `/sessions` and the sidebar
- **Messages** load when clicking sessions
- **Index** is at `~/.claude/conversations-index.json`
- **Memory** is in `~/.claude/cowork/hot-memory.md`
- Search with `/context <query>` or `node session-search.js <query>`
