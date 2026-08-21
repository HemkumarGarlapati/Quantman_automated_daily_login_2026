# QuantMan Auto-Login

Automatically log in to your broker account every weekday morning — no servers to run, no manual effort, and your password gets saved securely on your own GitHub account.

Supports **AliceBlue** and **Flattrade**. Other brokers on request.

This entire setup can be done from your phone.

---

## How your credentials stay safe

This is the most important thing to understand before you set this up.

Your broker password and TOTP secret never leave your own GitHub account. They're stored as GitHub Secrets — encrypted, never visible in logs, and only accessible to workflow runs inside your own forked repository. The login server never stores them either; it receives them only for the moment it takes to log in, then discards them.

Two separate credentials are involved, and it's worth knowing what each one can and can't do:

* **Your GitHub OIDC token** proves to the server that a specific workflow run belongs to your account. It's short-lived (valid for a single run) and generated automatically by GitHub — you never see or handle it.
* **The admin's GitHub App**, which you'll install in Step 4 below, is only ever used to send your repository a "please run your workflow now" signal on schedule. It cannot read your GitHub Secrets under any circumstance — that's not a configuration choice, it's a GitHub platform guarantee. Secret values are write-only once set; no app, token, or even the repo owner can read them back through any API.

---

## Setup (one-time, ~5 minutes, works entirely on mobile)

> **Mobile tip:** GitHub's Settings pages (where you add secrets) don't always render well inside the GitHub app itself. If a step below looks broken or missing, open the link in your **mobile browser** instead, and turn on **"Desktop site"** in your browser's menu for that page only.

### Step 1 — Fork this repository

Tap **Fork** at the top right of this page. This creates your own copy of the workflow under your GitHub account.

### Step 2 — Enable Actions on your fork

GitHub disables Actions by default on new forks. In **your forked repo**, tap the **Actions** tab and tap **"I understand my workflows, go ahead and enable them."** Nothing below will run without this.

### Step 3 — Add your secrets

In your forked repository: **Settings → Secrets and variables → Actions → New repository secret**. Add these four, one at a time:

| Secret name | Value |
|---|---|
| `QUANTMAN_CLIENT_ID` | Your broker client ID / user ID |
| `QUANTMAN_PASSWORD` | Your broker login password |
| `QUANTMAN_TOTP_SECRET` | Your TOTP secret key (the one you used to set up your authenticator app) |
| `BROKER` | Either `aliceblue` or `flattrade` |

> **Where do I find my TOTP secret?**
> It's the alphanumeric key shown when you set up two-factor authentication on your broker account — usually displayed as a QR code alongside a text key. If you no longer have it, you'll need to reset 2FA on your broker account to get it again.

### Step 4 — Install the GitHub App

This is what lets the admin trigger your login automatically every day, on the time you choose — no Personal Access Token, no third-party cron site to configure.

1. Open **https://github.com/apps/quantman-auto-login-trigger** and tap **Install**.
2. Choose your own personal GitHub account.
3. Select **"Only select repositories"** and pick your forked repo.
4. Review the requested permission (**Contents: Read and write** — used only to send the trigger signal, never to read your secrets) and tap **Install**.

### Step 5 — Get whitelisted and set your schedule

Two things, sent together:

1. **Message the notification bot first:** open Telegram, search for **@DailyAutoLogin_bot**, and send it `/start`. Bots can't message you first — this has to happen before anything else in this step will actually reach you.
2. **Send your details to the admin.** Use the setup form (ask the admin for the link) or send this directly via WhatsApp/Telegram to **9704291506** or email **hemugarlapati@gmail.com**:
   - Your name
   - GitHub username
   - Broker
   - Client ID
   - Your Telegram Chat ID (message **@userinfobot** on Telegram — it instantly replies with your numeric ID)
   - Your preferred daily login time (IST)

Without this step, the workflow will run if manually triggered, but the server will reject the login request, and you won't get alerts.

Once the admin confirms, you're done.

---

## Running the login

### Automatically (default)

Once whitelisted, this runs on its own every weekday at the time you requested (IST). The admin's server triggers it directly through the GitHub App — nothing on your end to configure or maintain.

### Manually, any time

Go to **Actions → Trigger QuantMan API Login → Run workflow**.

---

## What happens when it runs

1. The admin's server (or your manual tap) triggers the workflow.
2. It sends a quick ping to wake up the login server.
3. GitHub issues a short-lived OIDC token proving this run belongs to your account.
4. The server verifies the token, confirms your client ID is linked to your GitHub username, and performs the browser-based login on your behalf.
5. The session is established on the broker's end — QuantMan picks it up automatically.
6. You get a ✅ or ❌ Telegram message confirming the result.

The entire process takes roughly 30–45 seconds.

---

## Troubleshooting

* **502 Bad Gateway on wake-up:** Render's free-tier instance sleeps after inactivity. The workflow retries automatically, so this usually resolves within a minute or two.
* **Unauthorized Error:** Make sure you completed **Step 5** — your GitHub username and Client ID need to be whitelisted on the backend server.
* **Not getting Telegram alerts:** Confirm you sent `/start` to **@DailyAutoLoginBot** (Step 5.1), and that the numeric Chat ID you gave the admin is correct — get it again from **@userinfobot** if unsure.
* **Automatic runs never start:** Confirm you completed **Step 4** (installing the GitHub App) — without it, the admin's server has no way to trigger your workflow, even if you're fully whitelisted.
* **Want to change your daily time?** Just message the admin — no need to redo any setup steps.

---

## Privacy

* Your secrets are stored only in your own GitHub account.
* The login server does not log or store your password or TOTP secret.
* The server only records your GitHub username, broker name, and client ID for access control.
* The GitHub App can only trigger your workflow — it cannot read your repository's secrets, under any permission level, by GitHub's own design.
* OIDC tokens are short-lived (expire after the workflow run) and are masked in all logs.

---

## Questions / Access Requests

* **Telegram / WhatsApp:** 9704291506
* **Email:** hemugarlapati@gmail.com
