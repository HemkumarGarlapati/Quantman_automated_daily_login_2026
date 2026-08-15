# QuantMan Auto-Login

Automatically log in to your broker account every day with a single GitHub Actions workflow — no servers, no manual effort, no third-party apps holding your password.

Supports **AliceBlue** and **Flattrade**. Supports other brokers on request.

---

## How your credentials stay safe

This is the most important thing to understand before you set this up.

Your broker password and TOTP secret never leave your own GitHub account. They are stored as GitHub Secrets — encrypted, never visible in logs, and only accessible to workflow runs in your own forked repository. The login automation server never stores them either; it receives them only during the login request and discards them immediately after.

The server verifies your identity using a GitHub OIDC token — a short-lived cryptographic token issued by GitHub itself, valid for a single workflow run. No API keys, no passwords, no shared secrets between you and the server.

---

## Setup (one-time, ~5 minutes)

### Step 1 — Fork this repository

Click **Fork** at the top right of this page. This creates your own private copy of the workflow under your GitHub account.

### Step 2 — Add your secrets

In your forked repository, go to **Settings → Secrets and variables → Actions → New repository secret** and add the following:

| Secret name | Value |
|---|---|
| `QUANTMAN_CLIENT_ID` | Your broker client ID / user ID |
| `QUANTMAN_PASSWORD` | Your broker login password |
| `QUANTMAN_TOTP_SECRET` | Your TOTP secret key (the one you use to set up your authenticator app) |
| `BROKER` | Either `aliceblue` or `flattrade` |

> **Where do I find my TOTP secret?**
> It is the alphanumeric key shown when you set up two-factor authentication on your broker account — usually displayed as a QR code alongside a text key. If you set up TOTP using an app like Google Authenticator, you would have seen this key during initial setup. If you no longer have it, you will need to reset 2FA on your broker account to get it again.

### Step 3 — Request access

Send your **GitHub username**, **Broker Name**, and **User ID** to **9704291506** via WhatsApp or Telegram, or email **hemugarlapati@gmail.com** to be whitelisted on the server. Without this step, the workflow will run, but the server will reject the request.

Once confirmed, you are ready to go.

---

## Running the login

### Manually

Go to **Actions → Trigger QuantMan API Login → Run workflow**.

### Automatically (daily)

To run the login automatically at set times, cron-job.org needs permission to trigger your GitHub workflow. That permission comes from a Personal Access Token (PAT) — a secure key you generate once on GitHub and give to cron-job.org.

#### A · Create a Personal Access Token (PAT)
A PAT is a password-like key that lets cron-job.org trigger your workflow on your behalf without sharing your GitHub password.

1. **Open GitHub Settings:** Click your profile picture (top-right on github.com) → **Settings**.
2. **Go to Developer Settings:** Scroll to the very bottom of the left sidebar → click **Developer settings**.
3. **Open Fine-grained Tokens:** **Personal access tokens** → **Fine-grained tokens** → click **Generate new token**.
4. **Fill in the Token Form:** Use the details from the table below.
5. **Set Permissions:** Under **Repository permissions**, set **Actions** to **Read and write**, and **Contents** to **Read-only**. Leave everything else as **No access**.
6. **Generate and Copy:** Click **Generate token**. Copy the token immediately — GitHub shows it only once (it will look like `github_pat_11ABCDE...`).

| Token Form Field | What to Enter |
|---|---|
| **Token name** | `QuantMan Cron Trigger` |
| **Expiration** | Custom → select 1 year from today *(set a calendar reminder to renew before it expires)* |
| **Repository access** | Only select repositories → choose your QuantMan repo |
| **Actions permission** | Read and write |
| **Contents permission** | Read-only |

> **⚠️ Save your token now!** GitHub shows the token only once. If you lose it before saving it in cron-job.org, you will need to regenerate a new one.

---

## What happens when it runs

1. The cron job sends a quick ping request to wake up the login server.
2. The GitHub Actions workflow triggers automatically.
3. GitHub issues a short-lived OIDC token proving this run belongs to your account.
4. The server verifies the token, confirms your client ID is linked to your GitHub username, and performs the browser-based login on your behalf.
5. The session is established on the broker's end — QuantMan picks it up automatically.

The entire process takes roughly 30–45 seconds.

---

## Troubleshooting

* **502 Bad Gateway on Wake-up:** Render free-tier instances sleep after inactivity. Ensure your cron-job is scheduled a few minutes *before* your login automation to give the server time to wake up.
* **Unauthorized Error:** Ensure you completed **Step 3** to whitelist your GitHub username and Client ID on the backend server.

---

## Privacy

* Your secrets are stored only in your own GitHub account.
* The login server does not log or store your password or TOTP secret.
* The server only records your GitHub username, broker name, and client ID for access control.
* OIDC tokens are short-lived (expire after the workflow run) and are masked in all logs.

---

## Questions / Access Requests

* **Telegram / WhatsApp:** 9704291506
* **Email:** hemugarlapati@gmail.com
