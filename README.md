# QuantMan Auto-Login

Automatically log in to your broker account every weekday morning — no servers to run, no manual effort.

Supports **AliceBlue** and **Flattrade**. Other brokers on request.

One-time set up, takes 5 minutes, works entirely on mobile

---

## How your credentials are handled

This is the most important thing to understand before you set this up.

**Where they live.** Your broker password and TOTP secret are stored as **GitHub Secrets**, inside your own forked repository — not in a database, spreadsheet, or file the admin controls. Once you save a secret, GitHub never displays its value again to anyone, for any reason — not to another app, not to an API call, and not even to you or the admin looking at your own repo's Settings page. That's a permanent platform guarantee, not something either of you can turn off or bypass.

**When they're used.** Once a day, at the time you choose, **your own** GitHub Actions workflow (running under your account, not the admin's) reads those secret values and sends them once, over an encrypted connection, to the login server — for the single purpose of typing them into your broker's login page on your behalf. That's the only moment they're ever used. The server doesn't write them to a database or a log file; they exist in memory for the few seconds the login takes, then are gone once that run finishes.

**What the admin can't do.** The admin cannot retrieve or view your stored secret values, ever — GitHub blocks that for everyone, including the admin, permanently. The automation also can't do anything beyond that one login step: it cannot place trades, transfer funds, change your broker settings, or take any action on your account other than establishing your daily session.

Two other pieces are involved, worth knowing what each one can and can't do:

* **Your GitHub OIDC token** proves to the server that a specific workflow run belongs to your account. It's short-lived (valid for a single run) and generated automatically by GitHub — you never see or handle it.
* **The admin's GitHub App**, which you'll install in Step 4 below, is only ever used to send your repository a "please run your workflow now" signal on schedule. It cannot read your GitHub Secrets under any circumstance — that's not a configuration choice, it's the same GitHub platform guarantee mentioned above. Secret values are write-only once set; no app, token, or even the repo owner can read them back through any API.

---

## Setup (one-time, ~5 minutes, works entirely on mobile)

> **Mobile tip:** GitHub's Settings pages (where you add secrets) don't always render well inside the GitHub app itself. If a step below looks broken or missing, open the link in your **mobile browser** instead, and turn on **"Desktop site"** in your browser's menu for that page only.

### Step 1 — Fork this repository

Tap **Fork** at the top right of this page. This creates your own copy of the workflow under your GitHub account.

### Step 2 — Enable Actions on your fork

GitHub disables Actions by default on new forks. In **your forked repo**, tap the **Actions** tab and tap **"I understand my workflows, go ahead and enable them."** Nothing below will run without this.

### Step 3 — Add your secrets

In your forked repository: **Settings → Secrets and variables → Actions → New repository secret**. Add these four, one at a time. The names are kept to a single letter on purpose — nothing to copy or get exactly right, just type the letter and paste the value:

| Secret name | Value |
|---|---|
| `U` | Your broker client ID / user ID |
| `P` | Your broker login password |
| `S` | Your TOTP secret key (the one you used to set up your authenticator app) |
| `B` | Either `aliceblue` or `flattrade` |

> **Where do I find my TOTP secret?**
> It's the alphanumeric key shown when you set up two-factor authentication on your broker account — usually displayed as a QR code alongside a text key. If you no longer have it, you'll need to reset 2FA on your broker account to get it again.

> **Keep these up to date.** If you ever change your broker password or reset your TOTP/2FA, update the matching secret here (`P` or `S`) too — GitHub won't know about the change on its own, and the automation will keep using the old value until you update it.

### Step 4 — Install the GitHub App

This is what lets the admin trigger your login automatically every day, at the time you choose.

1. Open **https://github.com/apps/quantman-auto-login-trigger** and tap **Install**.
2. Choose your own personal GitHub account.
3. Select **"Only select repositories"** and pick your forked repo.
4. Review the requested permission (**Contents: Read and write** — used only to send the trigger signal, never to read your secrets) and tap **Install**.

### Step 5 — Get whitelisted and set your schedule

Open Telegram, search for **@DailyAutoLogin_bot**, and send it one message with:

- Your GitHub username
- Your User ID (broker client ID)
- Your Broker Name

You'll get an instant reply confirming your message was received.

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
2. GitHub issues a short-lived OIDC token proving this run belongs to your account.
3. The server verifies the token, confirms your client ID is linked to your GitHub username, and performs the browser-based login on your behalf.
4. The session is established on the broker's end — QuantMan picks it up automatically.
5. You get a ✅ or ❌ Telegram message confirming the result.

The entire process usually finishes in under a minute.

**If it fails.** A failed attempt retries automatically, up to 3 times, about a minute apart — so a single ❌ doesn't need any action from you right away; give it a few minutes to see if a retry follows up with a ✅. If it still fails after all 3 attempts, try logging in manually on the QuantMan website, the way you normally would. If that works fine, the issue is on the automation side — message the admin. If it also fails there, it's likely your password or TOTP — something only you can fix, by updating the relevant secret (Step 3) with the correct value.

---

## Troubleshooting

* **Unauthorized Error:** Make sure you completed **Step 5** — your GitHub username and Client ID need to be whitelisted on the backend server.
* **Not getting Telegram alerts:** Confirm you messaged **@DailyAutoLogin_bot** as described in Step 5 — you should have gotten an instant reply confirming it went through. If you didn't, try sending it again.
* **Automatic runs never start:** Confirm you completed **Step 4** (installing the GitHub App) — without it, the admin's server has no way to trigger your workflow, even if you're fully whitelisted.
* **Want to change your daily time?** Just message the admin — no need to redo any setup steps.

---

## Privacy

* Your secrets live only in your own GitHub account, encrypted, and unreadable by anyone (including the admin) once saved — see [How your credentials are handled](#how-your-credentials-are-handled) above.
* The login server doesn't write your password or TOTP secret to a database or log file — they're only ever held in memory for the seconds it takes to complete your daily login, then discarded.
* Your credentials are used for exactly one thing: logging into your broker account, and only at the time you scheduled (or when you manually run the workflow yourself). They're never used for anything else, and never on any other schedule.
* The admin's server only keeps your GitHub username, broker name, and client ID — needed for access control, nothing more.
* The GitHub App can only trigger your workflow — it cannot read your repository's secrets, under any permission level, by GitHub's own design.
* OIDC tokens are short-lived (expire after the workflow run) and are masked in all logs.

---

## Questions / Access Requests

* **Telegram / WhatsApp:** 9704291506
* **Email:** hemugarlapati@gmail.com
