
# AI-Powered Blog Automation with n8n — Technical Setup Guide

This document provides a step-by-step technical guide to set up the complete AI-powered blog automation system using n8n and third-party APIs. It is intended for developers, automation specialists, and technical users who want to deploy, modify, and run the system independently.

- ✨ This workflow is published on the [n8n official website](https://n8n.io/workflows/6734-ai-powered-blog-automation-generate-and-publish-seo-articles-with-gpt-4-to-wordpress-and-twitter/)
- 📚 Checkout the articles [generated with this workflow](https://articles.emp0.com/tag/v3/)
- 🛒 [Buy Now for only $29](https://0emp0.gumroad.com/l/content-farming-v3) and get instant traffic to your blog / wordpress site

## Contents

- [Basics](#basics)
- [Setup](#setup)
- [API Configs](#4-configure-api-credentials)
- [Customization](#customize-the-workflow)
- [Cost Summary](#cost-summary)
- [Final Notes](#final-notes)

---

# Basics

## How It Works

### Step 1: Ingest Recent News via RSS + Vectorization  
![Step 1](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-step-1.png)
We scrape articles from predefined RSS feeds and chunk them using a content splitting algorithm, saving them into a vector database (e.g., MongoDB with embedding support or Supabase pgvector). Each chunk includes a SHA256 fingerprint, timestamp, category, source, and embedding vector for similarity search.

```json
[
  "https://feeds.bbci.co.uk/news/technology/rss.xml",
  "https://www.wired.com/feed/rss",
  "https://www.technologyreview.com/topnews.rss",
  "https://techcrunch.com/feed/",
  "https://www.marktechpost.com/feed/",
  "https://hackernoon.com/feed",
  "https://www.artificialintelligence-news.com/feed/"
]
```

### Step 2: Topic Generation with Semantic Clustering  
![Step 2](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-step-2.png)
A topic extraction agent queries the vector store and groups article clusters by semantic similarity. This enables the workflow to determine what topic has recurring relevance improving freshness and relevance.

### Step 3: Research + Data Enrichment  
![Step 3](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-step-3.png)
For the selected topic:
- The workflow extracts quotes, facts, entities, source credibility, outbound links.
- It generates related and main keywords using LSI (Latent Semantic Indexing).
- It checks for existing SEO competition to tailor a unique angle.

### Step 4: Content Generation (3 Agent Loop)  
![Step 4](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-step-4.png)
The article is generated in a loop involving:
- **Task Definition Agent**: plans the structure, section breakdown, tone, and goals.
- **Content Generation Agent**: writes each block of the article with access to tools like:
  - Internet search to generate outbound links (via MCP tool)

![Outbound links](https://articles.emp0.com/wp-content/uploads/2025/08/mcp-tool-search-the-web.png)

  - Image generation (Leonardo AI or DALL·E via MCP)

![Image generation](https://articles.emp0.com/wp-content/uploads/2025/07/generate-and-upload-blog-images-with-leonardo-ai-and-wordpress.png)

  - Chart/graph generation (via QuickChart )

![Chart generation](https://articles.emp0.com/wp-content/uploads/2025/07/AI-Powered-Chart-Generation-from-Web-Data-with-GPT-4o-and-WordPress-Upload.png)

- **Quality Control Agent**: validates article length, readability, Flesch Kincaid score, SEO score, keyword density, uniqueness, narrative flow, and backlink count.

The agents operate in a looped chain of thought format using n8n loops and memory iterating until the article crosses an 80% quality threshold or max loop limit.

### Step 5: Blog Title Optimization  
![Step 5](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-step-5.png)
We generate 5 SEO optimized blog titles using GPT-4 and rank them by CTR potential using simulated headline testing techniques. The best one is selected.

### Step 6: Metadata Generation  
![Step 6](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-step-6.png)
This includes:
- Meta Title
- Meta Description
- URL Slug
- Alt Tags
- SEO Report
- Estimated Read Time
- Category + Tags

### Step 7: Featured Image Generation  
![Step 7](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-step-7.png)
A separate MCP (Model Context Protocol) tool generates a custom hero image based on the blog content using prompts tailored to your site branding.

### Step 8: WordPress Draft Creation  
![Step 8](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-step-8.png)
The article is sent to WordPress using its REST API with:
- HTML content blocks
- Featured image
- Meta tags
- Internal/external links
- Slug and publishing date

### Step 9: Distribution  
![Step 9](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-step-9.png)
We stagger publishing based on platform analytics:
- **Twitter** (X): publish image+caption+link using the Twitter API
![Step 9 twitter](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-step-9-twitter.png)
- **Dev.to**: repurpose into Markdown format and post using Dev.to’s API
![Step 9 dev.to](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-step-9-dev.to_.png)
- Optional: schedule via n8n timers for different timezones and platform peak engagement windows

---

## What makes this flow unique ? 

### 3 AI Agent Architecture 
This isn’t just automation. It’s AI workflow orchestration. You get a full scale, modular content team simulated by multi agent communication using the following architecture:

![3 AI Agents talking to each other](https://articles.emp0.com/wp-content/uploads/2025/07/content-generator-v3-3-ai-agents-talking-to-each-other-workflow-black.png)

### Save progress

All progress is saved step by step using a completedStep field so if the process fails midway, you can restart it from the failed node without regenerating the entire flow.

Completed step 5
![Completed step 5](https://articles.emp0.com/wp-content/uploads/2025/07/content-generator-v3-completed-step-5.png)

Completed step 9
![Completed step 9](https://articles.emp0.com/wp-content/uploads/2025/07/content-generator-v3-completed-step-9.png)

---

# Setup

## 1. Deploy Your n8n Instance

You can either:

### Option A: Use n8n Cloud

1. Register here: [n8n Cloud Signup](https://n8n.partnerlinks.io/emp0)
2. Login and go to **Workflows** > **New Workflow**

Cost: $20 per month

### Option B: Deploy on Railway (Recommended)

1. Go to this Railway template: [Deploy n8n on Railway](https://railway.com/deploy/n8n-with-workers?referralCode=jay)
2. Create a custom domain and connect it to your n8n on Railway (optional)
3. Deploy and access your n8n instance.
4. Create a username and password for your admin user

Cost is $5 per month but will scale with usage. I realistically spend around $20 per month. You can alternatively [deploy n8n on hostinger](https://www.hostinger.com/vps/n8n-hosting) which is cheaper in my opinion

![Successful Railway n8n Deployment](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-n8n-on-railway.png)

---

## 2. Download the Workflows

Download the 4 workflows from Gumroad after purchase:

1. Content Generator v3
2. Chart generation MCP tool
3. Image Generation MCP tool
4. Web Search MCP tool

![Gumroad download interface](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-gumroad-after-purchase.png)

---

## 3. Import to n8n

1. Open your n8n instance.
2. Go to **Workflows > Import Workflow**.
3. Upload each workflow `.json` file.
4. Rename and organize them appropriately (e.g., `01_content_ingestion`, `02_generate_article`, etc).

![Import to n8n](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-tutorial-import-from-file.png)

---

## 4. Configure API Credentials

### 4.1 OpenAI API Key

1. Get your API key from: [OpenAI Platform](https://platform.openai.com/account/api-keys)
2. In n8n, go to **Credentials** > **OpenAI API** > Add new.
3. Paste your key and save.

Cost: Free to register. Need to top up around $10, The monthly cost will be around $100

### 4.2 MongoDB Atlas (Vector DB + Document Storage)

1. Register at [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
2. Create a cluster and add a database user.
3. Whitelist your IP or set to allow all (`0.0.0.0/0`).
4. Get the connection string (replace `<password>` and `<dbname>`).
5. Create a **MongoDB** credential in n8n.

Cost: Free for an M0 cluster with 500mb of memory, its more than enough for the first 3 months

### 4.3 Leonardo AI (Image Generation)

1. Sign up at [Leonardo AI](https://app.leonardo.ai)
2. Navigate to **API Access** and copy your API key.
3. Add it to your environment or as a credential in n8n via HTTP Request node headers.

Cost: $10 but realistically will cost $20 per month

### 4.4 WordPress API Setup

#### On WordPress:
1. Go to your Wordrpess admin dashboard. Usually its [yourwordpress.com/wp-admin](yourwordpress.com/wp-admin)
2. Go to **Users > Your Profile > Application Passwords** and generate one.
3. Note your WordPress username, site URL, and app password.

#### On n8n:

1. Go to **Credentials** > **HTTP Basic Auth**
2. Enter:
   - Username: your WordPress username
   - Password: the application password
3. Use this credential in WordPress-related nodes.

### 4.5 Twitter API Setup

1. Apply for a Twitter Developer account at [Twitter Developer Portal](https://developer.twitter.com)
2. Create an App under your project.
3. Go to **Keys & Tokens** tab:
   - Consumer Key & Secret
   - Access Token & Secret
4. In n8n, go to **Credentials > Twitter OAuth1**
5. Enter these four values.

### 4.6 Dev.to API Setup

1. Go to [Dev.to Account Settings](https://dev.to/settings/account)
2. Copy your **DEV API Key**.
3. Add to n8n as a **Header** in HTTP Request node:
   - Header name: `api-key`
   - Value: `<your_dev_api_key>`

---

# Customize the workflow

## 5. Adjusting the Workflow

### Set Timezone

1. Go to each schedule node.
2. Set timezone to your local timezone (e.g., `Asia/Jakarta`).

![Set Timezone](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-tutorial-adjust-timezone.png)

### Timing Control

- **Step 1 (Ingestion)**: Run at 8 AM
- **Step 2 (Topic Selection)**: Run at 9 AM
- **Step 3–9**: Run hourly from 10 AM to 4 PM (or customize as needed)

![Customize schedulers](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-tutorial-adjust-schedule-time.png)

### Signature Customization

Edit the Markdown signature block in the last step before publishing to Dev.to or WordPress. Example:

``` markdown
    **Written by [Your Company Name](https://yourdomain.com)**  
    📬 Reach us at: [hello@yourdomain.com](hello@yourdomain.com)
    🐦 Twitter: [@yourhandle](https://twitter.com/yourhandle)
```

![Customize signature](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-tutorial-adjust-content-signature.png)

---

### SEO Quality Threshold

- Inside the **QC Agent** node, adjust the success condition (default `>= 8`).
- Recommended minimum: 7%. Avoid setting to 10 (100%), as it's extremely rare to reach and causes loops.

![Customize quality threshold](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-tutorial-adjust-quality-threshold.png)

### Max Iterations (AI Loop Control)

- Navigate to the `Do While` loop.
- Set maximum iterations to `1` if optimizing for token costs.
- You can raise it to `2–3` for better quality.

![Customize max iterations](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-tutorial-adjust-iterations.png)

### Define Your Content Pillars + RSS Feeds

At the beginning of the ingestion flow, edit the `Set` node to:

- Add content pillars, e.g., `AI`, `Automation`, `Business Ideas`
- Match pillar → WordPress Category ID map
- Add relevant RSS feed URLs

Example rss feed:

``` json
[
  "https://feeds.bbci.co.uk/news/technology/rss.xml",
  "https://www.wired.com/feed/rss",
  "https://www.technologyreview.com/topnews.rss",
  "https://techcrunch.com/feed/",
  "https://www.marktechpost.com/feed/",
  "https://hackernoon.com/feed",
  "https://www.artificialintelligence-news.com/feed/"
]

```
![Customize rss feed](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-tutorial-adjust-rss-feed.png)

Example category list
```json
[ 
    {
        "name": "AI",     
        "description": "Articles on AI and artificial intelligence, including machine learning, neural networks, and AI trends.",     
        "id":8
    },{
        "name": "Automation",     
        "description": "Articles on automation technologies, workflow automations, integrations, and process optimization.", 
        "id":9
    },{
        "name": "Business Ideas",     
        "description": "Articles on innovation, startup concepts, monetization strategies, and business model generation.",         
        "id":7
    } 
]
```

![Customize Content Pillars](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-tutorial-adjust-categories.png)

---

# Cost Summary

| Service           | Monthly Cost Estimate  |
|-------------------|------------------------|
| OpenAI            | $100                   |
| Leonardo AI       | $20                    |
| MongoDB Atlas     | Free (M0)              |
| Railway (n8n)     | $20                    |
| Monthly Estimate  | $150                   |

---

# Final Notes

- Each workflow saves state in `completedStep` so that you can recover if it fails midway.
- Make sure all credentials are active and environment variables set.
- Test each step manually before activating timers.

You're now fully equipped to automate your blog with cutting-edge AI, auto-publishing, and multi-platform distribution.

Need help with advanced customization, platform migration, or custom agents? Contact us for enterprise setup assistance.
[Email](tools@emp0.com)
[Discord](https://discord.com/users/jym.god)
[Official website](https://emp0.com)
