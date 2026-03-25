# 🦞 OpenClaw Job Search Agent
### *You Click Apply. Your AI Agent Does Everything Else in Your Job Search Journey.*

> **Presented at [OpenClaw Toronto Meetup](https://www.meetup.com/openclaw-toronto/) — March 26, 2026**
> Built by [Kaan Turgut](https://www.linkedin.com/in/hkaanturgut/)

---

## 🎯 What This Does

A fully automated job search agent that runs 24/7 on your own server. Every night it:

1. 🔍 **Searches** LinkedIn & Indeed for matching roles
2. 📊 **Scores** each job against your resume (only 80+/100 qualify)
3. 👤 **Finds** the recruiter or hiring manager email via Hunter.io
4. ✍️ **Drafts** a personalized cold outreach email using your real experience
5. 📱 **Sends you a Telegram message** with all results + approval buttons
6. 📧 **Sends the email via Gmail** the moment you say approve
7. 📝 **Logs everything** to `job-applications.md` — no duplicates, ever

**You only do one thing: click Apply on the job portal.**

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR OPENCLAW SERVER                       │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              OpenClaw Gateway (Port 18789)           │   │
│  │                                                     │   │
│  │  ┌──────────┐  ┌───────────┐  ┌─────────────────┐  │   │
│  │  │  Claude  │  │   Cron    │  │    Skills       │  │   │
│  │  │Sonnet 4.6│  │  9 PM ET  │  │ job-auto-apply  │  │   │
│  │  │(Anthropic│  │  Nightly  │  │ cold-email-     │  │   │
│  │  │   API)   │  │  Search   │  │   writer        │  │   │
│  │  └──────────┘  └───────────┘  │ apollo-api      │  │   │
│  │                               └─────────────────┘  │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                  │
└───────────────────────────┼──────────────────────────────────┘
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
   ┌─────────────┐  ┌──────────────┐  ┌─────────────┐
   │  Hunter.io  │  │   Gmail API  │  │  Telegram   │
   │  (Find      │  │  (Send cold  │  │  (Your      │
   │  recruiter  │  │   emails)    │  │  approval   │
   │  emails)    │  │              │  │  interface) │
   └─────────────┘  └──────────────┘  └─────────────┘
```

---

## 🔄 Nightly Job Search Flow

```
Every night at 9 PM Toronto time
           │
           ▼
   ┌───────────────┐
   │  Search Jobs  │ ← LinkedIn + Indeed
   │  Senior DevOps│   Remote Canada/Global
   │  Platform Eng │   Posted last 7 days
   │  SRE roles    │
   └───────┬───────┘
           │
           ▼
   ┌───────────────┐
   │ Score Against │ ← Checks job-search-profile.md
   │  Your Profile │   Min score: 80/100
   │               │   Skips already-applied companies
   └───────┬───────┘
           │
           ▼
   ┌───────────────┐
   │  Find Contact │ ← Hunter.io API
   │  Recruiter or │   Domain search by company
   │  Hiring Mgr   │   Gets name + verified email
   └───────┬───────┘
           │
           ▼
   ┌───────────────┐
   │  Draft Email  │ ← cold-email-writer skill
   │  Personalized │   Mentions Canadian Tire + Lenovo
   │  < 150 words  │   Tailored to each role
   └───────┬───────┘
           │
           ▼
   ┌───────────────┐
   │ Telegram Msg  │ ← Sent to your chat
   │  10 jobs with │   With full email preview
   │  approve/skip │   Approval buttons
   └───────┬───────┘
           │
     ┌─────┴──────┐
     │            │
     ▼            ▼
┌─────────┐  ┌─────────┐
│ APPROVE │  │  SKIP   │
│         │  │         │
│ ✅ Send  │  │ ⏭️ Log  │
│ email   │  │ as      │
│ via     │  │ skipped │
│ Gmail   │  │         │
│         │  │         │
│ 📝 Log  │  └─────────┘
│ to MD   │
└─────────┘
```

---

## 📱 What You See on Telegram

```
━━━━━━━━━━━━━━━━━━━━━━
🔢 JOB 1/10
🏢 Company: Hopper
💼 Role: Senior SRE / FinOps Engineer
📊 Match Score: 91/100
📍 Location: 100% Remote — Toronto base
🔗 Apply: https://hopper.com/careers
━━━━━━━━━━━━━━━━━━━━━━
👤 Contact: Jonathan Ostrander | Engineering Manager
📧 Email: jostrander@hopper.com
━━━━━━━━━━━━━━━━━━━━━━
📝 DRAFT EMAIL:
Subject: Senior SRE Opportunity at Hopper - 6 Years DevOps Experience

Hi Jonathan,

I'm excited about Hopper's growth and the Senior SRE/FinOps role.
My background directly matches: 6 years of DevOps engineering at
Canadian Tire (led 15-person team) and Lenovo (Hybrid Cloud Architect).

Specialized in Azure, Terraform, and GitHub Actions CI/CD pipelines.
I thrive in high-reliability environments where cost optimization and
operational excellence intersect.

Would love to discuss how I can contribute to Hopper's infrastructure goals.

Best, Kaan
━━━━━━━━━━━━━━━━━━━━━━
Reply "approve 1" → sends email + logs
Reply "skip 1" → skips and notes reason
━━━━━━━━━━━━━━━━━━━━━━
```

---

## 🛠️ Tech Stack

| Component | Tool | Purpose |
|-----------|------|---------|
| AI Agent | [OpenClaw](https://github.com/openclaw/openclaw) | Self-hosted agent framework |
| AI Model | Claude Sonnet 4.6 (Anthropic API) | Intelligence layer |
| Messaging | Telegram Bot API | Approval interface |
| Job Search | JobSpy (via job-auto-apply skill) | Multi-board job scraping |
| Contact Finding | [Hunter.io API](https://hunter.io) | Recruiter email lookup |
| Email Sending | Gmail API (via gog skill) | Outreach delivery |
| Email Writing | cold-email-writer skill | Personalized drafts |
| Server | Windows Server 2025 | Self-hosted runtime |
| Remote Access | [Tailscale](https://tailscale.com) | Secure remote access |

---

## 📋 Prerequisites

Before starting, you need:

- [ ] Windows Server / Linux machine (or any always-on computer)
- [ ] [OpenClaw installed and running](https://docs.openclaw.ai/start/getting-started)
- [ ] Anthropic API key ([console.anthropic.com](https://console.anthropic.com)) — Tier 2 recommended ($40 deposit)
- [ ] Telegram account + Bot created via [@BotFather](https://t.me/botfather)
- [ ] Gmail account
- [ ] [Hunter.io](https://hunter.io) free account (25 searches/month free)
- [ ] [Google Cloud project](https://console.cloud.google.com) with Calendar + Gmail APIs enabled

---

## 🚀 Setup Guide

### Step 1 — OpenClaw Installation & Configuration

> **References:**
> - [Getting Started](https://docs.openclaw.ai/start/getting-started)
> - [Onboarding CLI](https://docs.openclaw.ai/start/wizard)
> - [Windows Setup](https://docs.openclaw.ai/platforms/windows)

Run the onboarding wizard:
```bash
openclaw onboard --install-daemon
```

Configure your Anthropic API key:
```bash
openclaw agents add main
```

---

### Step 2 — Connect Telegram

1. Open Telegram and message [@BotFather](https://t.me/botfather)
2. Send `/newbot` and follow prompts
3. Copy your bot token
4. Add to OpenClaw config:

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

5. Start the gateway and approve the pairing code:
```bash
openclaw gateway start
openclaw pairing list telegram
openclaw pairing approve telegram YOUR_CODE
```

---

### Step 3 — Connect Gmail + Google Calendar

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project named `OpenClaw`
3. Enable **Gmail API** and **Google Calendar API**
4. Create OAuth credentials → Desktop app → Download `client_secret.json`
5. Copy `client_secret.json` to your OpenClaw workspace:
```
~/.openclaw/workspace/skills/gog/client_secret.json
```

6. Tell your bot to authenticate:
```
The client_secret.json is in skills/gog/ — please run the Google OAuth authentication flow now
```

7. Click the OAuth URL, sign in, copy the redirect URL back to your bot

---

### Step 4 — Set Up Hunter.io

1. Sign up free at [hunter.io](https://hunter.io)
2. Get your API key from the dashboard
3. Tell your bot:
```
Configure Hunter.io with this API key: YOUR_KEY
Use it to find recruiter emails by calling:
GET https://api.hunter.io/v2/domain-search?domain=COMPANY_DOMAIN&api_key=YOUR_KEY
```

---

### Step 5 — Install Required Skills

Send these messages to your bot one by one:

```
Install the job-auto-apply skill from ClawHub
```
```
Install the cold-email-writer skill from ClawHub
```
```
Install the gog skill from ClawHub
```

---

### Step 6 — Save Your Job Search Profile

Send this to your bot (customize with your details):

```
You are my personal job search agent. Here is my profile, save it permanently:

Name: [Your Name]
Current Role: [Your Role] at [Company]
Location: [City], Canada
Target: Remote Canada or Remote anywhere

Skills:
- [Your skills list]

Experience:
- [X years] in [your field]
- Key clients/companies: [list]

Target roles:
- [Role 1]
- [Role 2]
- [Role 3]

Preferences:
- Remote only
- Full-time
- Always require my confirmation before applying or sending any message
- Max 5 applications per day
- Start in dry-run mode first

Save this as my permanent job search profile in job-search-profile.md
```

---

### Step 7 — Set Up the Nightly Cron Job

Send this to your bot:

```
Set up a nightly job search cron job that runs every day at 9 PM Toronto time.

Every night:
1. Search LinkedIn and Indeed for [YOUR TARGET ROLES] - remote Canada or remote anywhere
2. Score each job against my saved profile - only show scores above 80/100
3. Skip any company already in job-applications.md
4. Find top 10 NEW matches only
5. For each match, use Hunter.io to find recruiter/hiring manager email
6. Use cold-email-writer to draft personalized outreach mentioning [YOUR KEY EXPERIENCE]
7. Send results to THIS Telegram chat in approval format
8. When I approve, send email via Gmail and log to job-applications.md
9. Never contact same person twice

Save as cron job called "nightly-job-search". Run now as test.
```

---

### Step 8 — Organize with Telegram Topics (Optional but Recommended)

1. Create a Telegram group called `Your AI Hub`
2. Add your bot as **Admin**
3. Enable **Topics** in group settings
4. Create topics:
   - `🔍 Job Search`
   - `☀️ Morning Brief`
   - `📺 YouTube Digest`
   - `⚙️ Config`

5. In each topic, tell the bot:
```
You are now in the [Topic Name] topic. Always send [relevant content] to this topic only.
```

---

## 🔒 Security Considerations

This agent has access to your email and job portals. Keep it safe:

| Risk | Mitigation |
|------|-----------|
| API keys exposed | Never paste keys in chat — use terminal input |
| Unauthorized bot access | Use `dmPolicy: "pairing"` (default) |
| Gateway exposed publicly | Keep bound to loopback, use Tailscale for remote access |
| Malicious ClawHub skills | Check VirusTotal score before installing any skill |
| Token bill explosion | Set monthly spend limit in Anthropic Console |
| Sending spam | Always use approval flow — never auto-send without confirmation |
| Duplicate applications | Agent checks `job-applications.md` before every send |

> **Key principle:** This runs on YOUR server. Your resume, emails, and job data never leave your machine. Unlike cloud AI tools, you own everything.

---

## 📁 File Structure

```
~/.openclaw/
├── openclaw.json              # Main config (model, Telegram, browser)
├── workspace/
│   ├── job-search-profile.md  # Your resume + preferences
│   ├── job-applications.md    # Log of all applications sent
│   ├── seen-videos.txt        # YouTube digest dedup list
│   └── skills/
│       ├── gog/               # Google Workspace (Gmail, Calendar)
│       │   └── client_secret.json
│       ├── job-auto-apply/    # Job search across platforms
│       ├── cold-email-writer/ # Personalized email drafting
│       └── apollo-api/        # Contact enrichment
└── agents/
    └── main/
        └── agent/
            └── auth-profiles.json  # API keys (Anthropic)
```

---

## 💡 Extending the Agent

Once the job search agent is working, here are natural next steps:

### Morning Briefing
```
Every morning at 8am Toronto time, send me:
1. Today's weather in Toronto
2. My Google Calendar events for today
3. Top 3 AI and tech news - browse the web
4. One build idea of the day
5. What you recommend I focus on today
```

### YouTube Digest
```
Every morning at 9am, search YouTube for new videos about OpenClaw,
Microsoft AI Foundry, and GitHub Copilot. For each new video get the
transcript, give me 3 bullet summary, and note anything relevant to my work.
```

### Interview Prep Agent
```
I have a meeting at [company] tomorrow. Research them, pull our email history,
find 3 talking points, and send me a prep brief tonight.
```

---

## 🤝 Contributing

This use case was built for the [awesome-openclaw-usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases) repo.

If you build on this or improve it, please submit a PR there. The repo has 26k+ stars and your contribution will help thousands of people.

---

## 📚 References

- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw Docs](https://docs.openclaw.ai)
- [awesome-openclaw-usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases)
- [ClawHub Skills Registry](https://clawhub.com)
- [Hunter.io API Docs](https://hunter.io/api-documentation)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Google OAuth Setup](https://developers.google.com/identity/protocols/oauth2)

---

## 👤 Author

**Kaan Turgut** — Hybrid Cloud Solution Architect

- 🐦 X: [@hkaanturgut](https://x.com/hkaanturgut)
- 💼 LinkedIn: [linkedin.com/in/hkaanturgut](https://linkedin.com/in/hkaanturgut)
- 🏢 Currently: Lenovo North America

---

*Built with 🦞 OpenClaw + ☁️ Azure + 🤖 Claude Sonnet 4.6*

*Presented at OpenClaw Toronto Meetup — March 26, 2026*
