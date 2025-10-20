Automatically turns trending Reddit posts into **punchy, first-person tweets** powered by **Google Gemini AI**, **Reddit**, and **Twitter API**, with Google Sheets logging.

---

## 🧩 Overview

This workflow repurposes Reddit content into original tweets every few hours.  
It’s perfect for creators, marketers, or founders who want to **automate content inspiration** while keeping tweets sounding **human, edgy, and fresh**.

**Core automation loop:**

1. Fetch trending Reddit posts from selected subreddits.  
2. Use **Gemini AI** to write a short, first-person tweet.  
3. Check your Google Sheet to avoid reusing the same Reddit post.  
4. Publish to Twitter automatically.  
5. Log tweet + Reddit reference in Google Sheets.  

---

## 🧠 Workflow Diagram

![Workflow Diagram](https://articles.emp0.com/wp-content/uploads/2025/10/reddit-twitter-workflow-1.png)

---

## 🪄 How It Works

1️⃣ **Every 2 hours** → the workflow triggers automatically.  
2️⃣ It picks a subreddit (like `r/automation`, `r/n8n`, `r/SaaS`).  
3️⃣ Gemini AI analyzes a rising Reddit post and writes a **fresh, short tweet**.  
4️⃣ The system checks your **Google Sheet** to ensure it hasn’t used that Reddit post before.  
5️⃣ Once validated, the tweet is **published via Twitter API** and **logged**. 

---

## 🧠 Example Tweet Output

![Example Tweet](https://articles.emp0.com/wp-content/uploads/2025/10/reddit-twitter-post.png)

---

## 📊 Logged Data (Google Sheets)

Each tweet is automatically logged for version control and duplication checks.

| Date | Subreddit | Post ID | Tweet Text |
|------|------------|----------|-------------|
| 08/10/2025 | n8n_ai_agents | 1o16ome | Just saw a wild n8n workflow on Reddit... |

---

## ⚙️ Key Components

| Node | Function |
|------|-----------|
| **Schedule Trigger** | Runs every 2 hours to generate a new tweet. |
| **Code (Randomly Decide Subreddit)** | Picks one subreddit randomly from your preset list. |
| **Gemini Chat Model** | Generates tweet text in first person tone using custom prompt rules. |
| **Reddit Tool** | Fetches top or rising posts from the chosen subreddit. |
| **Google Sheets (read database)** | Keeps a record of already-used Reddit posts. |
| **Structured Output Parser** | Ensures consistent tweet formatting (tweet text, subreddit, post ID). |
| **Twitter Node** | Publishes the AI-generated tweet. |
| **Append Row in Sheet** | Logs the tweet with date, subreddit, and post ID. |

---

## 🧩 Setup Tutorial

### 1️⃣ Prerequisites

| Tool | Purpose |
|------|----------|
| **n8n Cloud or Self-Host** | Workflow execution |
| **Google Gemini API Key** | For tweet generation |
| **Reddit OAuth2 API** | To fetch posts |
| **Twitter (X) API OAuth2** | To publish tweets |
| **Google Sheets API** | For logging and duplication tracking |

---

### 2️⃣ Import the Workflow

1. Download `Reddit Twitter Automation.json`.  
2. In n8n, click **Import Workflow → From File**.  
3. Connect your credentials:  
   - Gemini → `Gemini`  
   - Reddit → `Reddit account`  
   - Twitter → `X`  
   - Google Sheets → `Gsheet`  

---

### 3️⃣ Configure Google Sheet

Your sheet must include these columns:

| Column | Description |
|--------|--------------|
| `PAST TWEETS` | The tweet text |
| `Date` | Auto-generated date |
| `subreddit` | Reddit source |
| `post_id` | Reddit post reference |

![Google Sheet Log](https://articles.emp0.com/wp-content/uploads/2025/10/reddit-twitter-gsheet.png)

---

### 4️⃣ Customize Subreddits

In the **Code Node**, update this array to choose which subreddits to monitor:

```javascript
const subreddits = [
  "n8n",
  "microsaas",
  "SaaS",
  "automation",
  "n8n_ai_agents"
];