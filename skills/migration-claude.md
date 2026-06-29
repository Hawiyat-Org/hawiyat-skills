---
name: claude-cloud-to-local
description: Full pipeline — export Claude.ai cloud data → migrate to Claude Code sessions + Cowork skills + gateway + memory — one-shot autonomous migration with consistency audit
---

# Claude Cloud → Local: Full Migration Skill (v2.0)

Migrates an entire Claude.ai data export (conversations, memories, user profile) into a local-first architecture: Claude Code sessions + Cowork memory + conversations index — all in one pass.

**v2.0 fixes**: template contamination bug, truncated UUID detection, stale field inheritance, wrong `cwd` format, missing consistency audit, guarded overwrite preventing repairs, hardcoded model.

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
9. **Runs consistency audit** — validates every session has correct UUIDs, required fields, no stale template junk
10. **Logs** full summary of what was imported + audit results

## ⚠️ Critical Pitfalls (Lessons Learned — v2.0 Updates)

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

### Pitfall 6: `cwd` in agent-mode metadata must be the real filesystem outputs path ⚠️ UPDATED
The original skill used `/sessions/local_{uuid}` but real native sessions use the actual path:

```
C:\Users\<user>\AppData\Local\Claude-3p\local-agent-mode-sessions\{user}\{ws}\local_{uuid}\outputs
```

Using the virtual path causes message loading failures.

### Pitfall 7: audit.jsonl must NOT have `_audit_hmac`
Remove HMAC fields — they cause validation failures for imported sessions.

### Pitfall 8: NEVER spread a template session's fields ⚠️ NEW
The original skill did `{...template, sessionId, ...}` which copied:
- Stale `cliSessionId` (wrong — same for all sessions!)
- Wrong `title` from a random existing session
- `userApprovedFileAccessPaths` — paths to someone else's files
- `webFetchAllowedUrls` — 50+ unrelated URLs
- `hostLoopMode: true` (should be false for imported sessions)

Use a clean base template with only structural fields; populate per-session data explicitly.

### Pitfall 9: UUID directory names may be truncated ⚠️ NEW
- `local-agent-mode-sessions/` uses **8-character truncated UUIDs** (`d2e0770f`)
- `claude-code-sessions/` uses **full 36-character UUIDs** (`d2e0770f-a94b-448e-b755-a13318a66021`)
The regex `/^[0-9a-f-]{36}$/i` misses truncated dirs. Accept both formats and cross-reference.

### Pitfall 10: ALWAYS overwrite metadata on re-run ⚠️ NEW
The original skill used `if (!fs.existsSync(...))` guards. This means broken metadata from a first run is never fixed on re-run. Always write.

### Pitfall 11: Run consistency audit after migration ⚠️ NEW
After all writes, validate every session:
- `cliSessionId` matches the conversation UUID
- No stale template fields (`userApprovedFileAccessPaths`, `webFetchAllowedUrls`, foreign `title`)
- `hostLoopMode` is `false`
- All required files exist (JSONL, audit.jsonl, .claude.json, env.json)
- Index contains all entries

## Implementation

```node
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');

// ═══════════════════════════════════════════════════════════════════
//  CONFIGURATION
// ═══════════════════════════════════════════════════════════════════
const srcDir = (process.env.SRC_DIR || '.').replace(/\\+$/, '') + '/';
const claudeDir = process.env.USERPROFILE + '/.claude';
const dstDir = process.cwd();
const now = new Date().toISOString();
const localAppData = process.env.LOCALAPPDATA ||
  path.join(process.env.USERPROFILE, 'AppData', 'Local');

// Clean agent-mode template — NO template spreading. Only structural defaults.
const AGENT_TEMPLATE = {
  userSelectedFolders: [],
  isArchived: false,
  hostLoopMode: false,
  egressAllowedDomains: ['*.anthropic.com', 'anthropic.com', 'claude.com', '*.claude.com'],
  orgCliExecPolicies: { status: 'ok', policies: {} },
  memoryEnabled: true,
  skillsEnabled: true,
  pluginsEnabled: true,
  systemPromptRendererAppends: [],
  accountName: 'Cowork 3P',
  emailAddress: 'cowork-3p@localhost',
  slashCommands: [
    'deep-research', 'anthropic-skills:consolidate-memory',
    'anthropic-skills:schedule', 'anthropic-skills:setup-cowork',
    'clear', 'compact', 'context', 'heapdump', 'init',
    'reload-skills', 'review', 'security-review', 'usage',
    'insights', 'goal', 'team-onboarding'
  ],
  enabledMcpTools: {
    'local:mcp-registry:search_mcp_registry': false,
    'local:mcp-registry:suggest_connectors': false,
    'local:plugins:suggest_plugin_install': false,
    'local:plugins:search_plugins': false,
    'local:plugins:list_plugins': false
  },
  remoteMcpServersConfig: []
};

// Code-sessions template
const CODE_TEMPLATE = {
  titleSource: 'auto',
  permissionMode: 'acceptEdits',
  enabledMcpTools: {
    'local:mcp-registry:search_mcp_registry': false,
    'local:mcp-registry:suggest_connectors': false,
    'local:plugins:suggest_plugin_install': false,
    'local:plugins:search_plugins': false,
    'local:plugins:list_plugins': false
  },
  remoteMcpServersConfig: [],
  sessionPermissionUpdates: [],
  classifierSummaryEnabled: false,
  spawnSeed: {}
};

// ═══════════════════════════════════════════════════════════════════
//  HELPERS
// ═══════════════════════════════════════════════════════════════════
function readJSON(filePath) {
  const raw = fs.readFileSync(filePath, 'utf8');
  const clean = raw.charCodeAt(0) === 0xFEFF ? raw.slice(1) : raw;
  return JSON.parse(clean);
}

function sanitizeName(name) {
  return (name || 'Untitled').replace(/[\n\r]+/g, ' ').trim().substring(0, 100);
}

function getFirstUserText(messages) {
  const first = messages.find(m => m.sender === 'human' || m.role === 'user');
  if (!first) return '';
  const text = typeof first.text === 'string' ? first.text : (first.content || '');
  return text.substring(0, 100);
}

// ═══════════════════════════════════════════════════════════════════
//  1. READ EXPORT FILES
// ═══════════════════════════════════════════════════════════════════
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

// ═══════════════════════════════════════════════════════════════════
//  2. UPDATE COWORK MEMORY
// ═══════════════════════════════════════════════════════════════════
function updateCoworkMemory() {
  const hotMemPath = path.join(claudeDir, 'cowork', 'hot-memory.md');
  fs.mkdirSync(path.dirname(hotMemPath), { recursive: true });

  const profileLines = [];
  if (user.full_name) profileLines.push('- **Name:** ' + user.full_name);
  if (user.email_address) profileLines.push('- **Email:** ' + user.email_address);
  if (user.verified_phone_number) profileLines.push('- **Phone:** ' + user.verified_phone_number);

  const recentConvs = conversations
    .sort((a, b) => new Date(b.created_at || 0) - new Date(a.created_at || 0))
    .slice(0, 10)
    .map(c => '- **' + (c.name || 'Untitled') + '** (' + (c.created_at || '').slice(0, 10) + ')')
    .join('\n');

  const dateStr = now.slice(0, 10);
  const content = [
    '# Hot Memory',
    '> ' + dateStr,
    '',
    '## User Profile',
    profileLines.join('\n') || '- (from export)',
    '',
    '## Recent Conversations',
    recentConvs || '- (none)',
    '',
    '## Total History',
    '- **' + conversations.length + ' cloud conversations** imported',
    '- **' + memories.length + ' memory entries** synced',
    ''
  ].join('\n');
  fs.writeFileSync(hotMemPath, content);
  console.log('✅ Cowork hot-memory.md updated');
}
updateCoworkMemory();

// ═══════════════════════════════════════════════════════════════════
//  3. FIND USER/WORKSPACE UUIDs (handles both truncated and full)
// ═══════════════════════════════════════════════════════════════════
function findUserWorkspace() {
  const base = path.join(localAppData, 'Claude-3p', 'local-agent-mode-sessions');
  if (!fs.existsSync(base)) return null;

  // Accept both 8-char truncated and 36-char full UUIDs
  const uuidPat = /^[0-9a-f]{8}(?:-[0-9a-f-]{27})?$/i;
  const userDirs = fs.readdirSync(base).filter(d => uuidPat.test(d) && d !== 'skills-plugin');

  for (const u of userDirs) {
    const fullPath = path.join(base, u);
    if (!fs.statSync(fullPath).isDirectory()) continue;
    const wsDirs = fs.readdirSync(fullPath).filter(d => uuidPat.test(d));
    if (wsDirs.length > 0) {
      return { userTrunc: u, wsTrunc: wsDirs[0] };
    }
  }
  return null;
}

const uw = findUserWorkspace();
if (!uw) {
  console.log('❌ Could not find Claude-3p local-agent-mode-sessions directories');
  console.log('   Looked in: ' + path.join(localAppData, 'Claude-3p', 'local-agent-mode-sessions'));
  process.exit(1);
}
console.log('🔑 Found local-agent-mode UUIDs (truncated): user=' + uw.userTrunc + ' workspace=' + uw.wsTrunc);

// Cross-reference with claude-code-sessions to find full UUIDs
function findFullUuids(truncated) {
  const codeBase = path.join(localAppData, 'Claude-3p', 'claude-code-sessions');
  if (!fs.existsSync(codeBase)) return null;
  const fullUsers = fs.readdirSync(codeBase).filter(d => /^[0-9a-f-]{36}$/i.test(d));
  for (const u of fullUsers) {
    if (u.startsWith(truncated.userTrunc)) {
      const wss = fs.readdirSync(path.join(codeBase, u)).filter(d => /^[0-9a-f-]{36}$/i.test(d));
      for (const w of wss) {
        if (w.startsWith(truncated.wsTrunc)) {
          return { fullUserUuid: u, fullWsUuid: w };
        }
      }
    }
  }
  return null;
}

const fullUuids = findFullUuids(uw);
if (fullUuids) {
  console.log('🔑 Found full UUIDs: user=' + fullUuids.fullUserUuid + ' workspace=' + fullUuids.fullWsUuid);
} else {
  console.log('⚠️ Could not cross-reference full UUIDs — using truncated for code-sessions too');
}

const realUserUuid = fullUuids ? fullUuids.fullUserUuid : uw.userTrunc;
const realWsUuid = fullUuids ? fullUuids.fullWsUuid : uw.wsTrunc;

// Agent-mode uses truncated dirs; code-sessions uses full dirs
const agentBase = path.join(localAppData, 'Claude-3p', 'local-agent-mode-sessions', uw.userTrunc, uw.wsTrunc);
const codeSessDir = path.join(localAppData, 'Claude-3p', 'claude-code-sessions', realUserUuid, realWsUuid);

console.log('📂 Agent-mode dir: ' + agentBase);
console.log('📂 Code-sessions dir: ' + codeSessDir);

// ═══════════════════════════════════════════════════════════════════
//  4. READ NATIVE SYSTEM PROMPT (structural only — no field spreading)
// ═══════════════════════════════════════════════════════════════════
function readNativeSystemPrompt() {
  const jsonFiles = fs.readdirSync(agentBase).filter(f => f.endsWith('.json') && f.startsWith('local_'));
  for (const f of jsonFiles) {
    try {
      const content = JSON.parse(fs.readFileSync(path.join(agentBase, f), 'utf8'));
      if (content.systemPrompt) return content.systemPrompt;
    } catch(e) {}
  }
  return '';
}

const nativeSystemPrompt = readNativeSystemPrompt();
const detectedModel = nativeSystemPrompt.includes('opus') ? 'claude-opus-hawiyat-cowork[1m]' : 'claude-sonnet-4-20250514';
console.log('📝 Native system prompt found: ' + (nativeSystemPrompt ? nativeSystemPrompt.substring(0, 40) + '...' : 'NONE'));

// ═══════════════════════════════════════════════════════════════════
//  5. FIND PROJECT DIRECTORY FOR JSONL
// ═══════════════════════════════════════════════════════════════════
function findProjectDir() {
  const projectsBase = path.join(claudeDir, 'projects');
  fs.mkdirSync(projectsBase, { recursive: true });

  const dirs = fs.readdirSync(projectsBase);
  for (const d of dirs) {
    const decoded = d.replace(/C--/g, 'C:\\\\').replace(/-/g, '\\\\');
    if (dstDir.toLowerCase().includes(decoded.toLowerCase().slice(0, 20))) {
      return path.join(projectsBase, d);
    }
  }
  // Fallback: create new project dir
  const safeName = dstDir.replace(/[^a-zA-Z0-9]/g, '-').replace(/-+/g, '-').replace(/^-|-$/g, '');
  const newDir = path.join(projectsBase, 'C--' + safeName);
  fs.mkdirSync(newDir, { recursive: true });
  return newDir;
}

const projectDir = findProjectDir();
console.log('📁 Project dir: ' + projectDir);

// ═══════════════════════════════════════════════════════════════════
//  6. CONVERT & DEPLOY (always overwrite — no !existsSync guards)
// ═══════════════════════════════════════════════════════════════════
const convIndex = { generated_at: now, source: 'Claude.ai Export', conversations: [] };
let converted = 0, skipped = 0;
const auditResults = { passed: 0, failed: 0, issues: [] };

conversations.forEach(conv => {
  const uuid = conv.uuid;
  const name = sanitizeName(conv.name);
  const messages = conv.chat_messages || [];
  if (!messages.length) { skipped++; return; }

  const sessionId = 'local_' + uuid;
  const startTime = messages[0]?.created_at || now;
  const createdAtMs = new Date(startTime).getTime();
  const firstUserText = getFirstUserText(messages);

  // Pre-create data dir (needed for cwd path)
  const dataDir = path.join(agentBase, sessionId);
  fs.mkdirSync(path.join(dataDir, 'outputs'), { recursive: true });
  fs.mkdirSync(path.join(dataDir, 'uploads'), { recursive: true });

  // ── 6a. Agent-mode metadata ──
  // IMPORTANT: Build from clean template, NEVER spread an existing session.
  const agentMeta = Object.assign({}, AGENT_TEMPLATE, {
    sessionId: sessionId,
    processName: name,
    cliSessionId: uuid,
    cwd: path.join(dataDir, 'outputs'),
    createdAt: createdAtMs,
    lastActivityAt: createdAtMs,
    model: detectedModel,
    vmProcessName: name,
    initialMessage: firstUserText,
    systemPrompt: nativeSystemPrompt
  });

  // ⚠️ Explicitly delete known stale fields that could linger from template
  delete agentMeta.title;
  delete agentMeta.userApprovedFileAccessPaths;
  delete agentMeta.webFetchAllowedUrls;
  delete agentMeta.fsDetectedFiles;

  const agentMetaPath = path.join(agentBase, sessionId + '.json');
  fs.writeFileSync(agentMetaPath, JSON.stringify(agentMeta));
  converted++;

  // ── 6b. audit.jsonl (no HMAC) ──
  const auditLines = messages.map(msg => {
    const ts = msg.created_at || now;
    const text = typeof msg.text === 'string' ? msg.text : (msg.content || '');
    const isUser = msg.sender === 'human' || msg.role === 'user';
    return JSON.stringify({
      type: isUser ? 'user' : 'assistant',
      uuid: msg.uuid || crypto.randomUUID(),
      session_id: uuid,
      parent_tool_use_id: null,
      client_platform: 'desktop_app',
      message: isUser
        ? { role: 'user', content: text }
        : { id: msg.uuid || crypto.randomUUID(), type: 'message', role: 'assistant',
            content: [{ type: 'text', text }] },
      _audit_timestamp: ts
      // ⚠️ NO _audit_hmac field — causes validation failures
    });
  });
  fs.writeFileSync(path.join(dataDir, 'audit.jsonl'), auditLines.join('\n') + '\n');

  // ── 6c. .claude/.claude.json ──
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

  // ── 6d. Build JSONL transcript ──
  const nativeLines = [];

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
    const ts = msg.created_at || startTime;
    const msgUuid = msg.uuid || crypto.randomUUID();
    const text = typeof msg.text === 'string' ? msg.text : (msg.content || '');

    if (msg.sender === 'human' || msg.role === 'user') {
      nativeLines.push(JSON.stringify({
        type: 'user', uuid: msgUuid, parentUuid: prevUuid, timestamp: ts,
        sessionId: uuid, cwd: dstDir, entrypoint: 'claude-desktop-3p',
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
        sessionId: uuid, cwd: dstDir, entrypoint: 'claude-desktop-3p',
        version: '2.1.187', userType: 'external', gitBranch: 'HEAD',
        message: { id: msgUuid, type: 'message', role: 'assistant',
          content: [{ type: 'text', text }] }
      }));
      lastAssistantUuid = msgUuid;
      prevUuid = msgUuid;
    }
  });

  // ── CRITICAL: Add required metadata entries ──
  nativeLines.push(JSON.stringify({ type: 'mode', mode: 'normal', sessionId: uuid }));
  nativeLines.push(JSON.stringify({ type: 'custom-title', customTitle: name, sessionId: uuid }));
  nativeLines.push(JSON.stringify({
    type: 'last-prompt',
    lastPrompt: (lastUserText + name).substring(0, 150),
    leafUuid: lastAssistantUuid || lastUserUuid || uuid,
    sessionId: uuid
  }));

  const jsonlContent = nativeLines.join('\n') + '\n';

  // Write to project dir (where Claude Code reads transcripts)
  fs.writeFileSync(path.join(projectDir, uuid + '.jsonl'), jsonlContent);
  // Also write to export dir (backup)
  fs.writeFileSync(path.join(dstDir, uuid + '.jsonl'), jsonlContent);

  // ── 6e. Create session-env ──
  const envDir = path.join(claudeDir, 'session-env', uuid);
  fs.mkdirSync(envDir, { recursive: true });
  fs.writeFileSync(path.join(envDir, 'env.json'), JSON.stringify({ sessionId: uuid, cwd: dstDir }));

  // ── 6f. Create code-sessions metadata ──
  const codeMeta = Object.assign({}, CODE_TEMPLATE, {
    sessionId: sessionId,
    cliSessionId: uuid,
    cwd: dstDir,
    originCwd: dstDir,
    lastFocusedAt: Date.now(),
    createdAt: createdAtMs,
    lastActivityAt: Date.now(),
    model: 'claude-opus-hawiyat-composer',
    isArchived: false,
    title: name
  });

  const codeMetaPath = path.join(codeSessDir, sessionId + '.json');
  fs.writeFileSync(codeMetaPath, JSON.stringify(codeMeta));

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
    uuid: uuid, name: conv.name || 'Untitled',
    session_path: uuid + '.jsonl',
    message_count: messages.length,
    created_at: startTime,
    topics: conv.summary ? [topics] : topics
  });

  // ── 6g. Per-session audit ──
  const sessionIssues = [];
  if (agentMeta.cliSessionId !== uuid) sessionIssues.push('agent-mode cliSessionId mismatch');
  if (codeMeta.cliSessionId !== uuid) sessionIssues.push('code-sessions cliSessionId mismatch');
  if (agentMeta.hostLoopMode !== false) sessionIssues.push('hostLoopMode should be false');
  if (agentMeta.userApprovedFileAccessPaths) sessionIssues.push('stale userApprovedFileAccessPaths');
  if (agentMeta.webFetchAllowedUrls) sessionIssues.push('stale webFetchAllowedUrls');
  if (agentMeta.title !== undefined && agentMeta.title !== null) sessionIssues.push('stale title field');

  if (sessionIssues.length === 0) {
    auditResults.passed++;
  } else {
    auditResults.failed++;
    auditResults.issues.push({ uuid: uuid, name: name, issues: sessionIssues });
  }
});

console.log('');
console.log('📝 Converted: ' + converted + ', Skipped (empty): ' + skipped);

// ═══════════════════════════════════════════════════════════════════
//  7. BUILD CONVERSATIONS INDEX
// ═══════════════════════════════════════════════════════════════════
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
console.log('📇 Index updated: ' + convIndex.conversations.length + ' total entries');

// ═══════════════════════════════════════════════════════════════════
//  8. CONSISTENCY AUDIT — validate every session artifact on disk
// ═══════════════════════════════════════════════════════════════════
console.log('');
console.log('────────────────────────────────────────');
console.log('  🔍 RUNNING CONSISTENCY AUDIT');
console.log('────────────────────────────────────────');

let auditTotal = 0, auditPass = 0, auditFail = 0;

conversations.forEach(conv => {
  const uuid = conv.uuid;
  const sessionId = 'local_' + uuid;
  const issues = [];

  // Check agent-mode metadata exists and is correct
  const agentMetaPath = path.join(agentBase, sessionId + '.json');
  if (!fs.existsSync(agentMetaPath)) {
    issues.push('MISSING agent-mode metadata');
  } else {
    try {
      const meta = JSON.parse(fs.readFileSync(agentMetaPath, 'utf8'));
      if (meta.cliSessionId !== uuid) issues.push('WRONG cliSessionId: ' + meta.cliSessionId);
      if (meta.hostLoopMode !== false) issues.push('hostLoopMode=' + meta.hostLoopMode);
      if (meta.userApprovedFileAccessPaths) issues.push('STALE userApprovedFileAccessPaths');
      if (meta.webFetchAllowedUrls) issues.push('STALE webFetchAllowedUrls (' + meta.webFetchAllowedUrls.length + ' urls)');
      if (meta.title !== undefined) issues.push('STALE title field: "' + (meta.title || '').substring(0, 40) + '"');
    } catch(e) {
      issues.push('INVALID JSON in agent-mode metadata');
    }
  }

  // Check agent-mode data dir
  const dataDir = path.join(agentBase, sessionId);
  if (!fs.existsSync(dataDir)) issues.push('MISSING agent data directory');
  else {
    if (!fs.existsSync(path.join(dataDir, 'audit.jsonl'))) issues.push('MISSING audit.jsonl');
    if (!fs.existsSync(path.join(dataDir, 'outputs'))) issues.push('MISSING outputs/');
    if (!fs.existsSync(path.join(dataDir, '.claude', '.claude.json'))) issues.push('MISSING .claude/.claude.json');
  }

  // Check code-sessions metadata
  const codeMetaPath = path.join(codeSessDir, sessionId + '.json');
  if (!fs.existsSync(codeMetaPath)) issues.push('MISSING code-sessions metadata');
  else {
    try {
      const meta = JSON.parse(fs.readFileSync(codeMetaPath, 'utf8'));
      if (meta.cliSessionId !== uuid) issues.push('WRONG code cliSessionId: ' + meta.cliSessionId);
    } catch(e) {
      issues.push('INVALID JSON in code-sessions metadata');
    }
  }

  // Check JSONL in project dir
  const jsonlPath = path.join(projectDir, uuid + '.jsonl');
  if (!fs.existsSync(jsonlPath)) issues.push('MISSING JSONL in project dir');
  else {
    try {
      const content = fs.readFileSync(jsonlPath, 'utf8');
      const lines = content.trim().split('\n');
      const hasMode = lines.some(l => l.includes('"type":"mode"'));
      const hasTitle = lines.some(l => l.includes('"type":"custom-title"'));
      const hasLast = lines.some(l => l.includes('"type":"last-prompt"'));
      if (!hasMode) issues.push('MISSING mode entry in JSONL');
      if (!hasTitle) issues.push('MISSING custom-title entry in JSONL');
      if (!hasLast) issues.push('MISSING last-prompt entry in JSONL');
      // Validate all lines parse as JSON
      lines.forEach((line, i) => {
        try { JSON.parse(line); } catch(e) { issues.push('INVALID JSON at line ' + (i+1) + ' of JSONL'); }
      });
    } catch(e) { issues.push('UNREADABLE JSONL'); }
  }

  // Check session-env
  const envPath = path.join(claudeDir, 'session-env', uuid, 'env.json');
  if (!fs.existsSync(envPath)) issues.push('MISSING session-env');

  // Check index entries
  const inIndex = convIndex.conversations.some(c => c.uuid === uuid);
  if (!inIndex) issues.push('MISSING from conversations index');

  auditTotal++;
  if (issues.length === 0) {
    auditPass++;
    console.log('  ✅ ' + uuid.substring(0, 8) + '... ' + (conv.name || '').substring(0, 30));
  } else {
    auditFail++;
    console.log('  ❌ ' + uuid.substring(0, 8) + '... ' + (conv.name || '').substring(0, 30));
    issues.forEach(i => console.log('       - ' + i));
  }
});

// ═══════════════════════════════════════════════════════════════════
//  9. SUMMARY
// ═══════════════════════════════════════════════════════════════════
console.log('');
console.log('═══════════════════════════════════════');
console.log('  ✅ MIGRATION COMPLETE');
console.log('═══════════════════════════════════════');
console.log('  Conversations:     ' + converted + ' converted, ' + skipped + ' skipped');
console.log('  Project JSONL:     ' + (projectDir ? projectDir : '❌'));
console.log('  Agent-mode:        ✅ ' + converted + ' sessions');
console.log('  Code-sessions:     ✅ ' + converted + ' metadata');
console.log('  Index:             ' + convIndex.conversations.length + ' entries');
console.log('');
console.log('  🔍 AUDIT RESULTS');
console.log('  ──────────────');
console.log('  Passed:            ' + auditPass + '/' + auditTotal);
console.log('  Failed:            ' + auditFail + '/' + auditTotal);
if (auditResults.issues.length > 0) {
  console.log('  Per-session audit: ' + auditResults.passed + ' clean, ' + auditResults.failed + ' with issues');
}
console.log('');
console.log('  ⚠️  Known issues (non-blocking):');
console.log('  - hostLoopMode=false means imported sessions won\'t auto-fork');
console.log('  - model is set to detected native model (' + detectedModel + ')');
console.log('  - permissionMode=acceptEdits for imported sessions');
console.log('');
console.log('  📁 Locations:');
console.log('  - JSONL:          ' + projectDir);
console.log('  - Agent-mode:     ' + agentBase);
console.log('  - Code-sessions:  ' + codeSessDir);
console.log('  - Cowork memory:  ' + path.join(claudeDir, 'cowork', 'hot-memory.md'));
console.log('  - Index:          ' + idxPath);
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
- **Audit results** are printed at the end — any ❌ means something needs fixing

## v2.0 Changelog

| Fix | Description |
|-----|-------------|
| 🐛 Template contamination | Removed `{...template}` spread. Uses clean base template with explicit per-session fields. |
| 🐛 Truncated UUID detection | Accepts 8-char dir names. Cross-references with full UUIDs from code-sessions dir. |
| 🐛 Stale field inheritance | Explicitly `delete`s `title`, `userApprovedFileAccessPaths`, `webFetchAllowedUrls`, `fsDetectedFiles` |
| 🐛 Wrong cwd format | Uses real outputs dir path, not virtual `/sessions/` path |
| 🐛 Guarded overwrites | Removed all `if (!existsSync)` guards — always writes, so re-runs fix broken state |
| 🐛 Hardcoded model | Detects model from native session's systemPrompt |
| ✨ Consistency audit | Validates every session's metadata, JSONL, dirs, and index after migration |
| ✨ Per-session audit | Checks `cliSessionId`, stale fields, `hostLoopMode` during the conversion loop |
| ✨ Detailed error output | Each ❌ shows exactly which files/fields are wrong |
