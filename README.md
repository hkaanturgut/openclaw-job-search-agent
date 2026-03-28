# 🦞 OpenClaw Job Search Agent
### *You Click Apply. Your AI Agent Does Everything Else in Your Job Search Journey.*

> **Presented at [OpenClaw Toronto Meetup](https://www.meetup.com/openclaw-toronto/) — March 26, 2026**
> Built by [Kaan Turgut](https://www.linkedin.com/in/hkaanturgut/)

---

## 🎯 What This Does

A fully automated job search agent that runs 24/7 on your own server. Every night it:

1. 🔍 **Searches** JSearch API (Google for Jobs) for real job postings with direct apply links
2. 📊 **Scores** each job against your resume — only 80+/100 qualify
3. 👤 **Finds** the recruiter or hiring manager email via Hunter.io
4. ✍️ **Drafts** a personalized cold outreach email using your real experience
5. 📱 **Sends you a Telegram message** with all results + approval buttons
6. 📧 **Sends the email via Gmail** the moment you say approve
7. 📝 **Logs everything** to `job-applications.md` — no duplicates, ever

**You only do one thing: click Apply on the job portal.**

---

## 🏗️ Architecture

<img width="1681" height="938" alt="Screenshot 2026-03-28 at 3 50 32 PM" src="https://github.com/user-attachments/assets/81f7f025-abc3-407c-973a-0c0277fceaa1" />


## 📱 What You See on Telegram

```
━━━━━━━━━━━━━━━━━━━━━━
🔢 JOB 1/10
🏢 Company: [Target Company]
💼 Role: Senior Platform Engineer (Canada) | Score: 92/100
📍 Location: Remote — Canada
📅 Posted: March 20, 2026
🔗 APPLY: [https://company.com/careers/job/123456]
━━━━━━━━━━━━━━━━━━━━━━
👤 Contact: [Recruiter Name] | [Title]
📧 Email: [recruiter@company.com]
━━━━━━━━━━━━━━━━━━━━━━
📝 DRAFT EMAIL
Subject: Senior Platform Engineer — Azure + Terraform Expertise for [Target Company]

Hi [Recruiter Name],

[Target Company]'s platform engineering scope matches my background closely.
At [Previous Employer] I led a 15-person DevOps team through full
cloud transformation — Terraform IaC, AKS, GitHub Actions pipelines,
and DR testing across 8 cloud-native apps. Now at [Current Employer] as
Hybrid Cloud Solution Architect I design multi-cloud solutions daily.

Happy to discuss how I can contribute to your platform goals.

Best, [Your Name]
━━━━━━━━━━━━━━━━━━━━━━
✅ approve 1   |   ⏭️ skip 1
━━━━━━━━━━━━━━━━━━━━━━
```

---

## 🛠️ Tech Stack

| Component | Tool | Purpose |
|-----------|------|---------|
| AI Agent | [OpenClaw](https://github.com/openclaw/openclaw) | Self-hosted agent framework |
| AI Model | Claude Sonnet 4.6 (Anthropic API) | Intelligence layer |
| Messaging | Telegram Bot API | Approval interface |
| Job Search | [JSearch API — RapidAPI](https://rapidapi.com/letscrape-6bRBa3QguO5/api/jsearch) | Real job URLs via Google for Jobs |
| Job Scoring | `scorer.py` (custom script) | Profile-based scoring 0–100 |
| Contact Finding | [Hunter.io API](https://hunter.io) | Recruiter email lookup |
| Email Sending | Gmail API (via gog skill) | Outreach delivery |
| Email Writing | cold-email-writer skill | Personalized drafts |
| Server | Windows Server 2025 | Self-hosted runtime |
| Remote Access | [Tailscale](https://tailscale.com) | Secure RDP from Mac |

---

## 📋 Prerequisites

- [ ] Windows Server / Linux machine (or any always-on computer)
- [ ] [OpenClaw installed and running](https://docs.openclaw.ai/start/getting-started)
- [ ] Anthropic API key ([console.anthropic.com](https://console.anthropic.com)) — Tier 2 recommended ($40 deposit)
- [ ] Telegram account + Bot created via [@BotFather](https://t.me/botfather)
- [ ] Gmail account + Google Cloud project (Gmail API + Calendar API enabled)
- [ ] [RapidAPI account](https://rapidapi.com) — JSearch Basic plan ($10/month, 200 requests)
- [ ] [Hunter.io](https://hunter.io) free account (25 searches/month)

---

## 🚀 Setup Guide

> **⚠️ Follow these steps in order.** Each step creates folders and config that the next step depends on.

---

### Step 1 — Install OpenClaw

> **References:**
> - [Getting Started](https://docs.openclaw.ai/start/getting-started)
> - [Onboarding CLI](https://docs.openclaw.ai/start/wizard)
> - [Windows Setup](https://docs.openclaw.ai/platforms/windows)

```bash
npm install -g openclaw
openclaw onboard --install-daemon
openclaw gateway start
openclaw agents add main
```

> **Windows users — if `openclaw` is not recognized after install, fix the PATH first:**
> ```powershell
> npm config set prefix "$env:USERPROFILE\AppData\Roaming\npm"
> [System.Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";$env:USERPROFILE\AppData\Roaming\npm", "Machine")
> ```
> Close and reopen PowerShell, then run the commands above.

---
---

### Step 2 — Connect Telegram

1. Message [@BotFather](https://t.me/botfather) → `/newbot` → copy your token
2. Add to OpenClaw config:

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "YOUR_BOT_TOKEN",
      "dmPolicy": "pairing",
      "streaming": "partial"
    }
  }
}
```

3. Start and pair:
```bash
openclaw gateway start
openclaw pairing list telegram
openclaw pairing approve telegram YOUR_CODE
```

---

### Step 3 — Install Required Skills

> **Do this before any other setup.** The `gog` skill creates the `skills/gog/` folder that Gmail OAuth depends on in the next step.

Send these to your bot one by one:

```
Install the gog skill from ClawHub
```
```
Install the cold-email-writer skill from ClawHub
```

Verify the folder was created:
```powershell
dir "C:\Users\Administrator\.openclaw\workspace\skills"
```

You should see `gog/` and `cold-email-writer/` folders before continuing.

### Step 4 — Connect Gmail

> **Requires Step 2 to be completed first** — the `skills/gog/` folder must exist.

1. Go to [Google Cloud Console](https://console.cloud.google.com) → create project `OpenClaw`
2. Enable **Gmail API** and **Google Calendar API**
3. Go to **OAuth consent screen** → External → add your Gmail as a **Test User**
4. Create OAuth credentials → **Desktop app** → download `client_secret.json`
5. Place it at:
```
C:\Users\Administrator\.openclaw\workspace\skills\gog\client_secret.json
```
6. Tell your bot:
```
The client_secret.json is in skills/gog/ — run the Google OAuth authentication flow now
```
7. Click the OAuth URL, sign in with your Gmail, copy the redirect URL back to your bot

---

### Step 5 — Set Up JSearch (RapidAPI)

1. Sign up at [rapidapi.com](https://rapidapi.com)
2. Subscribe to [JSearch API](https://rapidapi.com/letscrape-6bRBa3QguO5/api/jsearch) — Basic plan ($10/month)
3. Copy your `X-RapidAPI-Key`
4. Tell your bot:
```
I have a JSearch API key from RapidAPI. Create jsearch_jobs.py in the workspace that calls:
GET https://jsearch.p.rapidapi.com/search
Headers: X-RapidAPI-Key: YOUR_KEY, X-RapidAPI-Host: jsearch.p.rapidapi.com
Test it with query "Senior DevOps Engineer remote Canada", num_pages=1
Show me the first 3 results with job_title, employer_name, job_apply_link
```

Confirm you see 3 real jobs with real apply links before continuing.

---

### Step 6 — Set Up Hunter.io

1. Sign up free at [hunter.io](https://hunter.io)
2. Copy your API key
3. Tell your bot:
```
Configure Hunter.io with API key: YOUR_KEY
For each company try these domain variations in order:
1. Domain extracted from job_apply_link URL
2. companyname.com
3. companyname.ca
Call: GET https://api.hunter.io/v2/domain-search?domain=DOMAIN&api_key=YOUR_KEY
```

4. Check your remaining quota:
```
Call GET https://api.hunter.io/v2/account?api_key=YOUR_KEY and tell me how many searches I have left this month
```

---

### Step 7 — Save Your Job Search Profile

```
Save my permanent job search profile as job-search-profile.md:

Name: [Your Name]
Current Role: [Your Role] at [Company]
Location: [City], Canada
Target: Remote Canada or Remote anywhere

Target roles:
- Senior DevOps Engineer
- Platform Engineer
- Site Reliability Engineer (SRE)
- Cloud Infrastructure Engineer

Skills: Azure, AWS, GCP, Terraform, Kubernetes, Docker, GitHub Actions,
        Jenkins, Python, Bash, PowerShell, Helm, ArgoCD

Key experience:
- [Company A]: [what you did, team size, impact]
- [Company B]: [what you did, team size, impact]

Preferences:
- Remote only, full-time
- Always require my confirmation before sending any email
```

---

### Step 8 — Deploy the Nightly Cron

Send this to your bot:

```
Create nightly_job_search_stable.py in the workspace.

STEP 1 — SEARCH (4 parallel queries, 10s timeout each, retry once on fail):
  "Senior DevOps Engineer remote Canada"
  "Platform Engineer remote Canada"
  "Site Reliability Engineer remote Canada"
  "Cloud Infrastructure Engineer remote Canada"
API: GET https://jsearch.p.rapidapi.com/search
Params: num_pages=3, remote_jobs_only=true, date_posted=month
Filter in Python: last 14 days, valid apply link, no Dice.com

STEP 2 — SCORE with scorer.py against job-search-profile.md:
  Title match: 30pts | Skills: 40pts | Remote: 20pts | <7 days: 10pts
Keep 80+. Skip companies in job-applications.md.
If fewer than 5 results, relax to 70/100.

STEP 3 — HUNTER.IO (parallel): try 4 domains per company.
Priority: Engineering Manager → VP Engineering → Technical Recruiter.

STEP 4 — DRAFT via cold-email-writer: unique subject, <120 words,
reference specific skills from the job description.

STEP 5 — TELEGRAM: one message per job in approval format.
Send final summary: X found → X filtered → X qualified → X contacts.

STEP 6 — approve N: send email via Gmail, log to job-applications.md.
        skip N: log as skipped.

STEP 7 — FALLBACK: after every successful run save to last_good_results.json.
If live search fails completely, load last_good_results.json and notify me.

Save as cron "nightly-job-search" at 9 PM Toronto (America/Toronto).
Run now as first test.
```

---

## 🔒 Security Considerations

| Risk | Mitigation |
|------|-----------|
| API keys exposed | Never paste keys in Telegram chat — use terminal/PowerShell |
| Unauthorized bot access | Use `dmPolicy: "pairing"` (default) |
| Gateway exposed publicly | Loopback only — use Tailscale for remote access |
| Malicious ClawHub skills | Check VirusTotal before installing any skill |
| API cost overrun | Set spend limits in Anthropic Console + RapidAPI dashboard |
| Sending spam | Always use approval flow — never auto-send without confirmation |
| Duplicate applications | Agent checks `job-applications.md` before every send |

> **Key principle:** This runs on YOUR server. Your resume, emails, and job data never leave your machine.

---

## 📁 File Structure

```
C:\Users\Administrator\.openclaw\
├── openclaw.json                        # Gateway config (model, Telegram)
└── workspace/
    ├── job-search-profile.md            # Your resume + target roles
    ├── job-applications.md              # All sent/skipped applications log
    ├── jsearch_results.json             # Raw JSearch API results
    ├── last_good_results.json           # Fallback — last successful search
    ├── workflow_results.json            # Scored + processed results
    ├── jsearch_jobs.py                  # JSearch API caller
    ├── scorer.py                        # Job scoring engine (0-100)
    ├── nightly_job_search_stable.py     # Main cron script
    └── skills/
        ├── gog/                         # Google Workspace (Gmail + Calendar)
        │   └── client_secret.json       # ← Place here AFTER installing gog skill
        └── cold-email-writer/           # Personalized email drafting
```

---

## 🤝 Contributing

Built for the [awesome-openclaw-usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases) repo. If you improve on this, please submit a PR there.

---

## 📚 References

- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw Docs](https://docs.openclaw.ai)
- [awesome-openclaw-usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases)
- [JSearch API — RapidAPI](https://rapidapi.com/letscrape-6bRBa3QguO5/api/jsearch)
- [Hunter.io API Docs](https://hunter.io/api-documentation)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Google OAuth Setup](https://developers.google.com/identity/protocols/oauth2)

---

## 👤 Author

**Kaan Turgut** — Hybrid Cloud Solution Architect

- 🐦 X: [@hkaanturgut](https://x.com/hkaanturgut)
- 💼 LinkedIn: [linkedin.com/in/hkaanturgut](https://linkedin.com/in/hkaanturgut)

---

*Built with 🦞 OpenClaw + 🤖 Claude Sonnet 4.6*

*Presented at OpenClaw Toronto Meetup — March 26, 2026*
