# Firebase Setup — Universal Harness Prompt

You are setting up Firebase CLI from https://github.com/firebase/firebase-tools on this machine. Follow every step exactly.

> **Agent instructions:** This document is a sequential setup guide. Work through each section in order. Do not skip steps. Do not assume anything — verify first.

---

## 1. CHECK IF FIREBASE CLI IS INSTALLED

\`\`\`bash
firebase --version
\`\`\`

**If already installed**, report the version and go to Step 3.

**If not installed**, continue to Step 2.

---

## 2. INSTALL FIREBASE CLI

Install from the official repo using npm:

\`\`\`bash
npm install -g firebase-tools
\`\`\`

Verify:

\`\`\`bash
firebase --version
\`\`\`

---

## 3. LOG IN TO FIREBASE

Run the login command:

\`\`\`bash
firebase login
\`\`\`

This will print a URL. **Present the full URL to the user** and ask them to open it in their browser to sign in with their Google account. Wait for the user to confirm they completed the login before continuing.

Verify login:

\`\`\`bash
firebase projects:list
\`\`\`

If projects are listed, the login was successful.

---

## 4. SUMMARY

| Field | Value |
|-------|-------|
| Firebase CLI | Installed |
| Auth status | Logged in |

### Confirm completion

Print a brief confirmation that Firebase is ready. Ask the user what they'd like to do (init a project, deploy hosting, etc.).
