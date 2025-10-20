# 🤖 Reddit Auto-Comment Assistant (AI-Driven Marketing Workflow)

Automate how you reply to Reddit posts using **AI-generated, first-person comments** that sound human, follow subreddit rules, and (optionally) promote your own links or products.

---

## 🧩 Overview

This workflow monitors Reddit mentions (via **F5Bot Gmail alerts**) and automatically:
1. Fetches the relevant Reddit post.  
2. Checks the subreddit’s rules for self-promotion.  
3. Generates a comment using GPT-5 style prompting (human-like tone, &lt;255 chars).  
4. Optionally promotes your chosen product from Google Sheets.  
5. Posts the comment automatically

It’s ideal for creators, marketers, or founders who want to grow awareness **organically and authentically** on Reddit — without sounding like a bot.

---

## 🧠 Workflow Diagram

![Workflow Diagram](https://articles.emp0.com/wp-content/uploads/2025/10/reddit-automation-workflow-1.png)

---

## 🚀 Key Features

| Feature | Description |
|----------|--------------|
| **AI-Generated Reddit Replies** | Uses GPT-powered reasoning and prompt structure that mimics a senior marketing pro typing casually. |
| **Rule-Aware Posting** | Reads subreddit rules and adapts tone — no promo where it’s not allowed. |
| **Product Integration** | Pulls product name + URL from your Google Sheet automatically. |
| **Full Automation Loop** | From Gmail → Gsheet → Reddit |
| **Evaluation Metrics** | Logs tool usage, link presence, and formatting to ensure output quality. |

---

## 🧰 Setup Guide

### 1️⃣ Prerequisites

| Tool | Purpose |
|------|----------|
| **n8n Cloud or Self-Host** | Workflow automation environment |
| **OpenAI API key** | For comment generation |
| **Reddit OAuth2 credentials** | To post comments |
| **Google Sheets API** | To fetch and evaluate products |
| **Gmail API** | To read F5Bot alerts |

---

### 2️⃣ Import the Workflow

1. Download `Reddit Assistant.json`  
2. In n8n, click **Import Workflow → From File**  
3. Paste your credentials in the corresponding nodes:
   - `Reddit account`
   - `Gmail account`
   - `Gsheet account`
   - `OpenAI API`

---

### 3️⃣ Connect Your Google Sheets

You’ll need **two Google Sheets**:

| Sheet | Purpose | Example Tab |
|--------|----------|-------------|
| **Product List** | Contains all your product names, URLs, goals, and CTAs | `promo` |
| **Reddit Evaluations** | Logs AI performance metrics and tool usage | `reddit evaluations` |

![Google Sheet Example](https://articles.emp0.com/wp-content/uploads/2025/10/reddit-automation-gsheet-promo.png)

---

### 4️⃣ Set Up Gmail Trigger (F5Bot)

1. Subscribe to [F5Bot](https://f5bot.com) alerts for keywords like `"blog automation"` or your brand name.  

![F5Bot Setup](https://articles.emp0.com/wp-content/uploads/2025/10/reddit-automation-f5bot-setup.png)

2. Configure Gmail Trigger to only pull from sender: `admin@f5bot.com`.

![F5Bot Email Example](https://articles.emp0.com/wp-content/uploads/2025/10/reddit-automation-email-f5bot.png)
---

### 5️⃣ Configure AI Agent Prompt

The built-in prompt follows a **GPT-5-style structured reasoning chain**:

- Reads the Reddit post + rules.  
- Determines if promotion is allowed.  
- Fetches product data from Google Sheets.  
- Writes a short, human comment (&lt;255 chars).  
- Avoids buzzwords and fake enthusiasm.

---

## 📊 Workflow Evaluations

The workflow includes **automatic evaluation nodes** to track:

| Metric | Description |
|--------|--------------|
| `contains link` | Checks if comment includes a URL |
| `contains dash` | Detects format breaks |
| `Tools Used` | Logs which AI tools were used in reasoning |
| `executionTime` | Monitors average latency |

![Workflow Evaluations](https://articles.emp0.com/wp-content/uploads/2025/10/reddit-automation-evaluations.png)

---

## 💡 Why This Workflow Has Value

| Value | Explanation |
|--------|--------------|
| **Saves time** | Automates Reddit marketing without manual engagement. |
| **Feels human** | AI comments use a fast-typing, casual tone (e.g., “u,” “ur,” “idk”). |
| **Follows rules** | Respects subreddits where promo is banned. |
| **Data-driven** | Logs performance across 10 test cases for validation. |
| **Monetizable** | Can promote Gumroad, YouTube, or SaaS products safely. |

---

## ⚙️ Example Use Case

&gt; “I used this automation to pull $1.4k by replying to Reddit posts about blog automation.  
&gt; Each comment felt natural and directed users to my n8n workflow.”

![Reddit Success Example](https://articles.emp0.com/wp-content/uploads/2025/10/reddit-automation-reddit-comment.png)