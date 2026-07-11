# Krynox status page (Upptime)

Public status page for Krynox Captcha at **https://status.krynox.net**, powered by
[Upptime](https://upptime.js.org). It runs **entirely on GitHub** — Actions cron
runs the checks, Issues track incidents, Pages serves the site. Nothing runs on
our own VPS, so the page stays up **even when `api.krynox.net` is down**, which is
exactly when a status page matters.

This folder holds the two things that are Krynox-specific: [`.upptimerc.yml`](./.upptimerc.yml)
(what to monitor + how the page looks) and this guide. Everything else (the
scheduled workflows, the status-site build) comes from the Upptime template.

## What it monitors

| Site                         | Endpoint                                          | Check                                          |
| ---------------------------- | ------------------------------------------------- | ---------------------------------------------- |
| Dashboard                    | `https://krynox.net`                              | 200, < 5 s                                     |
| API — Data plane             | `https://api.krynox.net/health`                   | 200, body contains `krynox-captcha-api`, < 3 s |
| Widget CDN                   | `https://cdn.krynox.net/widget/krynox-captcha.js` | 200, < 4 s                                     |
| Challenge issue _(optional)_ | `https://api.krynox.net/challenge?sitekey=kcpt_…` | end-to-end DB + HMAC path                      |

## One-time setup

1. **Create the repo from the template.** Go to <https://github.com/upptime/upptime>
   → **Use this template** → owner `krynox-security`, name **`status`**, visibility
   **Public** (public repos get free, unlimited Actions minutes + Pages).

2. **Drop in our config.** Replace the template's `.upptimerc.yml` with
   [`.upptimerc.yml`](./.upptimerc.yml) from this folder, and set `assignees` to
   the on-call maintainer's GitHub username.

3. **Add the logo.** Commit `assets/krynox-logo.png` (the light-mode mark) into the
   repo so the page renders its own logo without depending on krynox.net. It's
   already referenced by `logoUrl` in the config.

4. **Create the `GH_PAT` secret.** Upptime commits results and opens/closes
   incident issues, so it needs a token with more than the default.

   - Create a **fine-grained PAT** scoped to the `krynox-security/status` repo with
     **Contents: Read/Write**, **Issues: Read/Write**, **Pages: Read/Write**,
     **Administration: Read/Write** (classic PAT alternative: `repo` + `workflow`).
   - Repo → **Settings → Secrets and variables → Actions → New repository secret**,
     name **`GH_PAT`**, paste the token.

5. **Enable Issues + Actions.** Settings → General → Features → **Issues** on.
   Settings → Actions → **Allow all actions**. Then run **Setup CI** once from the
   Actions tab (or just push the `.upptimerc.yml` change — it triggers setup).

6. **Enable GitHub Pages.** Settings → **Pages** → Source = **Deploy from a branch**,
   branch **`gh-pages`** / `/ (root)`. Upptime's setup workflow publishes there.

7. **Wire the custom domain (DNS).** `cname: status.krynox.net` in the config writes
   the `CNAME` file; you still need the DNS record. In the krynox.net zone
   (Cloudflare) add:

   ```
   Type: CNAME   Name: status   Target: krynox-security.github.io
   ```

   Cloudflare-proxied (orange cloud) is fine. In GitHub **Settings → Pages → Custom
   domain**, enter `status.krynox.net` and tick **Enforce HTTPS** once the cert is
   issued. (If Cloudflare's proxy causes a cert loop, switch the record to **DNS
   only** / grey cloud until GitHub issues the cert, then re-proxy.)

8. **Done — no code change needed.** The footer link on the marketing site already
   points to `https://status.krynox.net` (`aegis/apps/web/src/routes/+page.svelte`).

## Tuning later

- **Notifications** (email/Slack/Telegram/Discord on incident) — add a `notifications:`
  block to `.upptimerc.yml`; Upptime supports many channels via Apprise.
- **Enable the end-to-end challenge check** — uncomment the `Challenge issue` site and
  paste a real public `kcpt_…` key (public by design; safe to commit).
- **Check cadence / thresholds** — `maxResponseTime`, `expectedStatusCodes`, and the
  cron interval are all in-repo config.

## Why not build our own UI?

A status page must be hosted **independently of the system it reports on**. Krynox is
currently a single Bun process on a single VPS (no HA). A bespoke page inside
`apps/web` would go down with the very outage it's supposed to announce. Upptime lives
on GitHub, off our infra, for free — the right trade-off at this stage. Revisit a
branded in-house page once the platform is multi-node (Horizon 2+).
