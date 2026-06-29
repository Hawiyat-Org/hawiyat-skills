# gws CLI Setup — Universal Harness Prompt

You are setting up the Google Workspace CLI (\`gws\`) from https://github.com/googleworkspace/cli on this machine. This prompt is designed to be fed to any AI coding agent on any OS (Linux, macOS, Windows/WSL). Follow every step exactly.

> **Agent instructions:** This document is a sequential setup guide. Work through each section in order. Do not skip steps. Do not assume tools are installed — verify first. After completing all steps, print the final summary table. **On Windows, if an automated install fails, provide the user a step-by-step manual guide and wait for them to confirm completion before proceeding.** You are the one who will use \`gws\` to fulfill the user's Google Workspace tasks — do not print CLI usage examples or "next steps" to the user; just confirm setup is complete.

---

## 1. DETECT THE ENVIRONMENT

Determine the following before proceeding:

- **OS**: \`uname -a\` (Linux), \`sw_vers\` (macOS), \`ver\` (Windows). For Linux, also check the distro (\`cat /etc/os-release\`).
- **Home directory**: \`echo $HOME\`
- **Current working directory**: \`pwd\`
- **AI client/agent**: Check for config directories and report which one is present:
  \`\`\`
  ~/.claude/       → Claude Code
  ~/.agents/       → Cursor / Windsurf / Zed / Amp
  ~/.openclaw/     → OpenClaw
  ~/.gemini/       → Gemini
  \`\`\`
  If multiple are present, report all and default to the first for skill symlinking.

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
5. On the **Tools for Native Modules** page, optionally check "Automatically install the necessary tools" (not required for gws)
6. Click **Install**, then **Finish**
7. **Close and reopen your terminal** (or reload the harness shell) so PATH updates take effect
8. Run \`node --version && npm --version\` to verify — report the versions

Do not proceed until the user confirms they see valid version output.

Report the installed versions.

---

## 3. INSTALL GOOGLE CLOUD CLI (gcloud)

Check first — do not reinstall if already present:

\`\`\`bash
gcloud --version
\`\`\`

**If already installed**, report the version and move to Step 4.

**If not installed**, consult https://cloud.google.com/sdk/docs/install for OS-specific instructions:

| OS | Method |
|----|--------|
| Linux (Debian/Ubuntu) | Add the Cloud SDK apt repo, import the Google Cloud public key, then \`sudo apt-get install google-cloud-cli\` |
| Linux (RHEL/Fedora/CentOS) | Add the Cloud SDK yum/DNF repo, then \`sudo dnf install google-cloud-cli\` |
| Linux (Arch) | \`yay -S google-cloud-sdk\` or download the tar archive from the install page, extract, and run \`./google-cloud-sdk/install.sh\` |
| Linux (tar archive) | Download \`google-cloud-cli-linux-*.tar.gz\`, extract, run \`./google-cloud-sdk/install.sh\` |
| macOS (Homebrew) | \`brew install --cask google-cloud-sdk\` |
| macOS (tar archive) | Download \`google-cloud-cli-darwin-*.tar.gz\`, extract, run \`./google-cloud-sdk/install.sh\` |
| Windows | Download \`GoogleCloudSDKInstaller.exe\` from the install page |

#### Windows fallback — if the installer fails or is unavailable

Provide the user the following step-by-step guide and **wait for them to confirm** before continuing:

1. Open **https://cloud.google.com/sdk/docs/install#windows** in your browser
2. Download \`GoogleCloudSDKInstaller.exe\`
3. Run the downloaded \`.exe\` file
4. If Windows SmartScreen appears, click **More info → Run anyway**
5. Follow the installer wizard:
   - Accept the license agreement
   - Choose install location (default is fine)
   - **Important:** Check "Add gcloud to PATH" when prompted
   - Check "Start Google Cloud SDK Shell" if offered
6. Click **Install**, then **Finish**
7. **Close and reopen your terminal** so PATH updates take effect
8. Run \`gcloud --version\` to verify — report the output

Do not proceed until the user confirms \`gcloud --version\` produces output.

After installation, verify with \`gcloud --version\`. If not on PATH, add it (the installer usually offers to do this).

---

## 4. INSTALL THE GWS CLI

\`\`\`bash
gws --version
\`\`\`

**If already installed**, report the version and move to Step 5.

**If not installed**:

\`\`\`bash
npm install -g @googleworkspace/cli
\`\`\`

Verify with \`gws --version\`. If not on PATH, check \`npm root -g\` and common bin directories, then add to PATH.

---

## 5. SET UP GCP PROJECT AND AUTHENTICATION

This is a multi-step process. **Do not skip any sub-step.**

### Step 5a — Authenticate gcloud

\`\`\`bash
gcloud auth list
\`\`\`

If no credentialed accounts exist, run:

\`\`\`bash
gcloud auth login
\`\`\`

This will print a URL. **Present the full URL to the user** and ask them to open it in their browser. The command blocks until the browser login completes. Wait for the user to confirm before continuing.

### Step 5b — Select or Create a GCP Project

Ask the user for their **GCP Project Name** (not the ID). Then find the Project ID:

\`\`\`bash
gcloud projects list --format="value(projectId,name)"
\`\`\`

Set the project:

\`\`\`bash
gcloud config set project PROJECT_ID
\`\`\`

### Step 5c — Configure the OAuth Consent Screen and Test Users

> **⚠️ This step is mandatory.** OAuth client creation fails without a configured consent screen, and authentication fails if the user is not in the test users list.

Direct the user to:
**https://console.cloud.google.com/auth/overview?project=PROJECT_ID**

Guide them through:

1. Choose **External** user type (or **Internal** if using Google Workspace)
2. Fill in:
   - **App name**: \`gws CLI\`
   - **User support email**: their email
   - **Developer contact information**: their email
3. Accept the User Data Policy checkbox
4. Complete all remaining steps (Audience, Contact Information, Finish)

**After the consent screen is created, immediately add the user as a test user:**

Direct the user to:
**https://console.cloud.google.com/auth/audience?project=PROJECT_ID**

5. In the **Test users** section, click **Add users**
6. Enter their email address
7. Click **Save**

> **Do not skip this.** New OAuth apps default to **Testing** mode. Without being added as a test user, the browser authentication flow will be rejected — even with a valid Client ID and Secret. This is the most common blocker during setup.

Wait for the user to confirm they added their email before continuing.

### Step 5d — Create the OAuth Client ID

> **⚠️ This must be done AFTER the consent screen is configured.**

Direct the user to:
**https://console.cloud.google.com/apis/credentials?project=PROJECT_ID**

Guide them through:

1. Click **+ Create Credentials → OAuth client ID**
2. Application type: **Desktop app**
3. Name: \`gws CLI\`
4. Click **Create**
5. Copy the **Client ID** and **Client Secret** from the popup

### Step 5e — Collect and Validate Credentials

Ask the user to share both values:

\`\`\`
Client ID:     xxxxxxx.apps.googleusercontent.com
Client Secret: GOCSPX-xxxxxxxxxxxxxxxx
\`\`\`

**Validate before proceeding:**

- Client ID must end with \`.apps.googleusercontent.com\`
- Client Secret must start with \`GOCSPX-\`
- Neither value may be empty

If validation fails, ask the user to re-check and re-share.

### Step 5f — Run gws auth login

> **⚠️ Important:** \`export\` does not persist across shell calls in most harnesses. Always set env vars and run the command in the **same shell invocation**.

\`\`\`bash
export GOOGLE_WORKSPACE_CLI_CLIENT_ID="<client-id>.apps.googleusercontent.com" && \\
export GOOGLE_WORKSPACE_CLI_CLIENT_SECRET="<client-secret>" && \\
gws auth login -s drive,gmail,calendar,sheets,docs,tasks
\`\`\`

This command will print a URL. **Present the full URL to the user** and ask them to open it in their browser.

> If authentication fails with a "not a test user" error, the user was not added in Step 5c. Send them to **https://console.cloud.google.com/auth/audience?project=PROJECT_ID** to add their email, then re-run this command to regenerate the link.

Wait for the user to confirm the browser flow completed before continuing.

---

## 6. VERIFY AUTHENTICATION

Run \`gws auth status\` and check the JSON output. Confirm **all** of the following:

| Field | Expected |
|-------|----------|
| \`user\` | The user's email |
| \`client_config_exists\` | \`true\` |
| \`config_client_id\` | Populated (matches the Client ID) |
| \`has_refresh_token\` | \`true\` |
| \`token_valid\` | \`true\` |
| \`encrypted_credentials_exists\` | \`true\` |
| \`encryption_valid\` | \`true\` |

Also verify gcloud auth:

\`\`\`bash
gcloud auth list
\`\`\`

If any field is missing or \`false\`, debug before continuing. Common issues:
- \`has_refresh_token: false\` → the browser flow didn't complete; re-run \`gws auth login\`
- \`client_config_exists: false\` → env vars weren't set; ensure they're in the same shell command
- Test user error → see Step 5f warning above

---

## 7. INSTALL ALL AGENT SKILLS FROM THE REPO

\`\`\`bash
npx skills add https://github.com/googleworkspace/cli
\`\`\`

This installs skills into \`.agents/skills/\` in the project directory.

### Install skills globally

Skills must be available **globally** so the agent can use them from **any directory** — not just the project where they were installed. Do NOT symlink from \`~/.claude/skills/\` to the project-local \`.agents/skills/\` — those symlinks break when the user runs Claude Code from a different directory. Instead, **copy** the skill directories into the global config.

**For Claude Code** — copy into \`~/.claude/skills/\`:

\`\`\`bash
mkdir -p ~/.claude/skills
cp -r .agents/skills/gws-* .agents/skills/persona-* .agents/skills/recipe-* ~/.claude/skills/
\`\`\`

**For OpenClaw** — copy into \`~/.openclaw/skills/\`:

\`\`\`bash
mkdir -p ~/.openclaw/skills
cp -r .agents/skills/gws-* .agents/skills/persona-* .agents/skills/recipe-* ~/.openclaw/skills/
\`\`\`

> **Do NOT create project-local \`.claude/skills/\` symlinks.** The whole point is that skills work everywhere. A project-local symlink only works inside that one project directory — it defeats the purpose.

> **Windows note:** Symlinks require elevated privileges on Windows. If \`ln -sf\` fails, **copy** instead: \`cp -r .agents/skills/gws-* .agents/skills/persona-* .agents/skills/recipe-* ~/.claude/skills/\`

### Verify skills are globally accessible

\`\`\`bash
# Should show real files, NOT symlinks
ls -la ~/.claude/skills/gws-drive

# Should be readable and contain valid frontmatter
head -5 ~/.claude/skills/gws-drive/SKILL.md

# Count total skills in the global directory
ls ~/.claude/skills/ | wc -l
\`\`\`

If \`ls -la\` shows \`->\` (a symlink), the copy failed — remove the symlink and re-run the \`cp\` command. Report the total count of skills installed.

---

## 8. FETCH AND STORE REFERENCE DOCUMENTATION

Download these files from the repo to the project root for future reference:

\`\`\`bash
curl -sL https://raw.githubusercontent.com/googleworkspace/cli/main/README.md -o README.md
mkdir -p docs
curl -sL https://raw.githubusercontent.com/googleworkspace/cli/main/docs/skills.md -o docs/skills.md
curl -sL https://raw.githubusercontent.com/googleworkspace/cli/main/AGENTS.md -o AGENTS.md
curl -sL https://raw.githubusercontent.com/googleworkspace/cli/main/CONTEXT.md -o CONTEXT.md
curl -sL https://raw.githubusercontent.com/googleworkspace/cli/main/.env.example -o .env.example
\`\`\`

Verify each file is non-empty after download. If any curl fails (network issue, 404), report which files are missing and continue with the ones that succeeded.

---

## 9. FINAL VERIFICATION AND SUMMARY

Run \`gws auth status\` one final time. Then print the summary table:

| Field | Value |
|-------|-------|
| OS detected | ... |
| AI client detected | ... |
| Node.js version | ... |
| npm version | ... |
| gcloud version | ... |
| gws version | ... |
| Skills installed | ... (count) |
| Skills visible at | \`~/.claude/skills/\` (global) |
| Config directory | \`~/.config/gws/\` |
| Auth status | Authenticated / Not authenticated |
| User email | ... |
| Client ID | ... (masked) |
| Has refresh token | true / false |
| Token valid | true / false |

### Confirm completion

Print a brief confirmation that setup is complete. Do **not** print CLI usage examples or "next steps" — you are the agent who will use \`gws\` to perform the user's Google Workspace tasks. If the user's original request was just to set up gws, confirm and ask what they'd like to do with it.
