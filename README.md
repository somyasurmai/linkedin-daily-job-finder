# LinkedIn Daily Job Finder

An AI-powered job search automation agent that scrapes LinkedIn daily, scores job listings against your profile using Claude AI, and delivers a ranked digest to your inbox every morning.

Built with n8n, Apify, Anthropic Claude, Google Sheets, and Gmail.

## What It Does

Every morning at 9am, this agent:

1. Scrapes fresh LinkedIn job postings (last 24 hours) across your target roles and locations
2. Scores each job against your profile using Claude AI (0-100 score)
3. Classifies jobs as STRONG MATCH, ALSO WORTH CONSIDERING, or EXCLUDE
4. Logs all matches to Google Sheets for tracking
5. Sends a beautiful HTML email digest with ranked results sorted by score

## Sample Output

The daily email digest shows:
- Strong matches highlighted in green
- Worth considering in yellow
- Sorted by score, highest first
- One-click Apply button per job

## Prerequisites

You will need free accounts on the following services:

| Service | Purpose | Cost |
|---------|---------|------|
| n8n | Workflow automation | Free trial / ~€5/month self-hosted |
| Apify | LinkedIn scraping | Free tier ($5 credit/month) |
| Anthropic | AI scoring via Claude | Pay as you go (~$3-20/month) |
| Google Sheets | Job tracker | Free |
| Gmail | Daily digest | Free |

## Setup

### 1. Import the n8n Workflow

- Open your n8n instance
- Go to Workflows → Import
- Upload `workflow.json` from this repo

### 2. Set Up Apify

- Create an account at apify.com
- Find the actor `curious_coder/linkedin-jobs-scraper`
- Copy your API token from Apify Console → Settings → Integrations
- Add it to the HTTP Request nodes in n8n

### 3. Set Up Anthropic

- Create an account at anthropic.com
- Generate an API key
- Add credits ($10 recommended to start)
- Add it to the Anthropic Chat Model node in n8n

### 4. Set Up Google Sheets

Create a new Google Sheet with the following column headers:

DATE | TITLE | COMPANY | LOCATION | SCORE | CATEGORY | CONTRACT | LANGUAGE | POSTED | MATCH REASON | URL | STATUS

Connect your Google account in n8n and link the sheet to the Append Row nodes.

### 5. Set Up Gmail

Connect your Google account in n8n to the Gmail node for sending the daily digest.

### 6. Customise the Search URLs

In HTTP Request 1, update the Apify POST body with your target keywords and locations:

```json
{
  "urls": [
    "https://www.linkedin.com/jobs/search/?keywords=%22Learning+%26+Development%22&location=Germany&f_TPR=r86400&f_E=1,2,3&f_JT=F,I,C",
    "https://www.linkedin.com/jobs/search/?keywords=%22HR+Specialist%22&location=Germany&f_TPR=r86400&f_E=1,2,3&f_JT=F,I,C",
    "https://www.linkedin.com/jobs/search/?keywords=%22Learning+%26+Development%22&location=France&f_TPR=r86400&f_E=1,2,3&f_JT=F,I,C",
    "https://www.linkedin.com/jobs/search/?keywords=%22HR+Specialist%22&location=France&f_TPR=r86400&f_E=1,2,3&f_JT=F,I,C"
  ],
  "rows": 1,
  "scrapeCompany": false,
  "splitByLocation": false
}
```

URL parameter reference:

| Parameter | Meaning |
|-----------|---------|
| keywords | Job title to search (use %22 for exact phrase quotes) |
| location | Country or city name |
| f_TPR=r86400 | Posted in last 24 hours |
| f_E=1,2,3 | Entry, Associate, Mid level |
| f_JT=F,I,C | Full-time, Internship, Contract |

### 7. Customise the AI Scoring Prompt

Open the AI Agent node and update the prompt with:

- Your education and experience
- Your target roles and priorities
- Your language skills
- Your preferred locations

### 8. Activate

Set the Schedule Trigger to your preferred time (default: 9am daily) and toggle the workflow to Active.

## How the AI Scoring Works

Each job is scored out of 100 across 5 dimensions:

| Dimension | Max Points |
|-----------|-----------|
| Function and title fit | 40 |
| Experience match | 20 |
| Language fit | 20 |
| Location tier | 15 |
| Brand bonus | 5 |

Score classifications:

| Score | Classification |
|-------|---------------|
| 65-100 | STRONG MATCH |
| 45-64 | ALSO WORTH CONSIDERING |
| 0-44 | EXCLUDE |

## Workflow Architecture

```
Schedule Trigger (9am daily)
→ HTTP Request 1 (Apify — start scrape)
→ Wait (30 seconds)
→ HTTP Request 2 (Apify — fetch results)
→ Loop Over Items
    → Wait (10 seconds)
    → AI Agent (Claude — score each job)
    → Code Node (parse JSON)
    → If (score ≥ 45)
        → True: Google Sheets Job Tracker
        → False: Skip
→ Aggregate
→ Gmail (daily digest)
```

## Cost Estimate

Running daily with approximately 50 jobs per day:

| Service | Monthly Cost |
|---------|-------------|
| Apify | Free tier |
| Claude Haiku | ~$1-2/month |
| Claude Sonnet | ~$10-20/month |
| n8n self-hosted | ~€5/month |

## Customisation Tips

- Add more keywords by adding more URLs to the Apify POST body
- Change locations by replacing country names in the URLs
- Adjust scoring thresholds by editing the If node condition
- Switch AI model between claude-haiku-4-5 and claude-sonnet-4-6
- Add more target roles by editing the AI Agent prompt

## Ideas for Future Improvement

- CV tailoring per job using Claude
- LinkedIn contact finder for each company
- Deduplication across days to avoid seeing the same job twice
- Telegram or Slack notification option
- Support for other job boards such as Indeed and Glassdoor

## Article

Read the full story of how and why this was built:
Coming soon

## Disclaimer

This tool scrapes publicly available LinkedIn job listings. Use responsibly and in accordance with LinkedIn's Terms of Service.

## License

MIT License — free to use, modify and distribute.

---

Built by a 2026 HR/Tech Master's graduate who got tired of manually checking LinkedIn every day.
