# Git PAT Setup Guide

How to create a Personal Access Token (PAT) and configure it for claudeit
on macOS and Windows. claudeit uses HTTPS + PAT only — no SSH required.

---

## Step 1 — Create Your Token

### GitHub

1. Go to **github.com** → click your profile photo (top right) → **Settings**
2. Scroll down the left sidebar → **Developer settings** (at the bottom)
3. **Personal access tokens** → **Tokens (classic)**
4. Click **Generate new token** → **Generate new token (classic)**
5. Fill in:
   - **Note:** `claudeit`
   - **Expiration:** 90 days (or "No expiration" for convenience)
   - **Scopes:** check ✅ `repo` — this covers everything claudeit needs
6. Click **Generate token**
7. **Copy the token immediately** — GitHub shows it only once

### GitLab

1. Go to **gitlab.com** → click your avatar (top right) → **Preferences**
2. Left sidebar → **Access Tokens**
3. Click **Add new token**
4. Fill in:
   - **Token name:** `claudeit`
   - **Expiration date:** set or leave blank
   - **Scopes:** check ✅ `read_repository` and ✅ `write_repository`
5. Click **Create personal access token**
6. **Copy the token immediately** — GitLab shows it only once

---

## Step 2 — Store Token on Your Machine

claudeit stores your token in the OS credential manager so it is used automatically
on every push and pull. You only do this once per machine.

---

### macOS

macOS has a built-in secure Keychain. Git uses it natively.

**1. Configure git to use macOS Keychain:**
```bash
git config --global credential.helper osxkeychain
```

**2. Store your token — run this in Terminal:**
```bash
printf "protocol=https\nhost=github.com\nusername=YOUR_GITHUB_USERNAME\npassword=YOUR_TOKEN_HERE\n" | git credential approve
```
Replace `github.com` with `gitlab.com` if using GitLab.
Replace `YOUR_GITHUB_USERNAME` and `YOUR_TOKEN_HERE` with your actual values.

**3. Verify it works:**
```bash
git ls-remote https://github.com/YOUR_USERNAME/YOUR_REPO.git HEAD
```
If it returns a hash with no password prompt — you are done.

**To view what is stored:**
Open **Keychain Access** app → search for `github` → you will see the claudeit entry.

**To delete and re-enter when token expires:**
```bash
printf "protocol=https\nhost=github.com\n" | git credential reject
```
Then repeat Step 2 with your new token.

---

### Windows — Git Bash

Git for Windows includes Git Credential Manager (GCM) which stores tokens
in Windows Credential Manager (encrypted).

**1. Verify GCM is active:**
```bash
git config --global credential.helper
```
Should return `manager` or `manager-core`. If it does, GCM is already set up.

If not, configure it:
```bash
git config --global credential.helper manager
```

**2. Store your token — run this in Git Bash:**
```bash
printf "protocol=https\nhost=github.com\nusername=YOUR_GITHUB_USERNAME\npassword=YOUR_TOKEN_HERE\n" | git credential approve
```

**3. Verify:**
```bash
git ls-remote https://github.com/YOUR_USERNAME/YOUR_REPO.git HEAD
```
No prompt = working.

**To view stored credentials:**
Open **Windows Credential Manager** → **Windows Credentials** → look for `git:https://github.com`

**To delete and re-enter:**
```bash
printf "protocol=https\nhost=github.com\n" | git credential reject
```

---

### Windows — WSL (Windows Subsystem for Linux)

WSL can share credentials with Windows Git Credential Manager so you only
store the token once and both environments use it.

**1. Find your Windows GCM path inside WSL:**
```bash
ls "/mnt/c/Program Files/Git/mingw64/bin/git-credential-manager.exe" 2>/dev/null \
  && echo "Found" || echo "Not found — check your Git for Windows installation"
```

**2. Configure WSL git to use Windows GCM:**
```bash
git config --global credential.helper \
  "/mnt/c/Program Files/Git/mingw64/bin/git-credential-manager.exe"
```

**3. Store your token — run this inside WSL:**
```bash
printf "protocol=https\nhost=github.com\nusername=YOUR_GITHUB_USERNAME\npassword=YOUR_TOKEN_HERE\n" | git credential approve
```

**4. Verify:**
```bash
git ls-remote https://github.com/YOUR_USERNAME/YOUR_REPO.git HEAD
```

**If GCM path is not found**, fall back to WSL plaintext store:
```bash
git config --global credential.helper store
printf "protocol=https\nhost=github.com\nusername=YOUR_GITHUB_USERNAME\npassword=YOUR_TOKEN_HERE\n" | git credential approve
```
Note: `store` saves to `~/.git-credentials` in plaintext — less secure but functional.

---

## Step 3 — Tell claudeit Your Remote URL

When you run `claudeit init` and choose DISTRIBUTED, claudeit asks:
```
What is the git remote URL for your context repo?
```

Always use the **HTTPS URL**, not the SSH URL:
```
✅  https://github.com/your-org/your-context-repo.git
❌  git@github.com:your-org/your-context-repo.git   ← SSH, do not use
```

claudeit tests the connection immediately using the token stored in Step 2.
If it works, setup continues. If not, it tells you exactly what is wrong.

---

## Quick Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `Authentication failed` | Wrong token or expired | Generate new token, redo Step 2 |
| `Repository not found` | Token missing `repo` scope | Regenerate with `repo` checked |
| `Permission denied (403)` | Token is read-only | Regenerate with write scope |
| `Could not resolve host` | Wrong URL or no internet | Check URL and connection |
| Password prompt keeps appearing | Credential helper not configured | Re-run the git config line in Step 2 |

**Token expired?** Run `claudeit login reset` inside a Claude Code session.
claudeit clears the old token and walks you through entering a new one.

---

## Security Rules claudeit Enforces

- Token stored in OS credential manager only
- Git uses the token directly — claudeit never handles it after initial setup
- Token never written to `.contextit/` or any project file
- Token never committed to any git repo
- Token never logged or echoed after you paste it

If a token is ever detected inside a `.contextit/` file, claudeit refuses
to commit and removes it before any push.
