# Receiptify LC254 — License Control Panel

Private license authority for **Receiptify LAN**.  
Hosted at: `https://mulw.github.io/receiptifylc254`

---

## How it works

```
┌─────────────────────────────┐      ┌──────────────────────────────────────┐
│   LC254 Control Panel       │      │   Receiptify Backend (client PC)     │
│   (GitHub Pages)            │      │                                      │
│                             │      │  On startup + every 12h:             │
│  ┌─────────────────────┐    │      │    fetch licenses.json               │
│  │ Issue License       │    │      │    find entry matching fingerprint   │
│  │ Revoke License      │────┼──────▶   if revoked → block new logins     │
│  │ Extend/Reduce       │    │      │   if expired → block new logins      │
│  │ View all clients    │    │      │   if not found → check local .key    │
│  └─────────────────────┘    │      └──────────────────────────────────────┘
│          │                  │
│          ▼                  │
│  licenses.json (this repo)  │
└─────────────────────────────┘
```

## Setup

### 1. Create the GitHub repo
- Repo name: `receiptifylc254` (private or public — raw content is unauthenticated)
- Enable **GitHub Pages** → Settings → Pages → Source: `main` branch, root `/`

### 2. Configure the panel (`index.html`)
Edit the `CONFIG` block at the top of `index.html`:

```js
const CONFIG = {
  PANEL_PASSWORD:  'YourStrongPassword',          // panel login password
  GITHUB_OWNER:    'mulw',                         // your GitHub username
  GITHUB_REPO:     'receiptifylc254',
  GITHUB_FILE:     'licenses.json',
  GITHUB_BRANCH:   'main',
  GITHUB_TOKEN:    'github_pat_xxxx...',           // Fine-grained PAT (Contents: Read+Write)
  SHARED_SECRET:   'RECEIPTIFY_LC254_CLOUD_SECRET_2025', // must match license.js
};
```

### 3. Generate a GitHub PAT
- Go to: https://github.com/settings/tokens
- Click "Generate new token (fine-grained)"
- Repository access: `receiptifylc254`
- Permissions → Repository → Contents: **Read and Write**
- Copy the token into `GITHUB_TOKEN` in `index.html`

### 4. Push to GitHub
```bash
git init
git add .
git commit -m "init"
git remote add origin https://github.com/mulw/receiptifylc254.git
git push -u origin main
```

---

## Workflow for a new client

1. Client runs `node scripts/generateLicense.js` on their machine
2. It prints their **machine fingerprint** (32-char hex)
3. They give you the fingerprint
4. You open `mulw.github.io/receiptifylc254`, log in, click **"+ Issue License"**
5. Fill in business name, fingerprint, plan, expiry → **Issue License**
6. Click **Save to GitHub**
7. Click **Token** on the new license → copy the token
8. Client pastes it when prompted by `generateLicense.js` (or saves as `license.key`)
9. Client starts the server — it verifies against the cloud automatically

---

## Revoking a license

1. Open the panel, find the business
2. Click **✗ Revoke**
3. Click **Save to GitHub**
4. Within 12 hours (next check), the client server detects the revocation and blocks new logins
5. Existing logged-in sessions continue until they expire (typically 24h)

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | The password-gated control panel (runs entirely in browser) |
| `licenses.json` | The license registry — read by all Receiptify instances |

## Security notes

- `licenses.json` is **publicly readable** (needed for client verification without auth)
- The panel itself is password-protected
- Each license entry contains only the machine fingerprint (hash), not the raw machine ID
- Tokens are HMAC-signed with `SHARED_SECRET` — tampering is detectable
- The GitHub PAT is embedded in `index.html` — keep this repo **private** or rotate the PAT regularly
