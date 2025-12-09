# AI-Powered Blog Automation with n8n — V4 Technical Setup Guide

This document provides a comprehensive step-by-step technical guide to set up the Content Generator V4 system using n8n and third-party APIs. It is intended for developers, automation specialists, and technical users who want to deploy, modify, and run the system independently.

**What's New in V4**: This guide includes setup for GPT-5 models, Gemini-based nano banana pro image generation (via Gemini HTTP request and API), dual vector database architecture, MongoDB memory system, internal linking configuration, and Yoast SEO integration.

---

## Table of Contents

- [What's New in V4](#whats-new-in-v4-setup-changes)
- [Architecture Overview](#architecture-overview)
- [Prerequisites](#prerequisites)
- [Step 1: Deploy n8n](#step-1-deploy-your-n8n-instance)
- [Step 2: Import Workflows](#step-2-download-and-import-workflows)
- [Step 3: Configure Credentials](#step-3-configure-api-credentials)
  - [3.1 OpenAI API](#31-openai-api-gpt-5-models)
  - [3.2 MongoDB Atlas](#32-mongodb-atlas-dual-vector-database)
  - [3.3 MongoDB Vector Search](#33-mongodb-vector-search-setup-critical-for-v4)
  - [3.4 Google Gemini (Image Generation)](#34-google-gemini-image-generation-via-node--api)
  - [3.5 WordPress API + Yoast SEO Setup](#35-wordpress-api-setup)
  - [3.6 Twitter API](#36-twitter-api-setup-x-platform)
  - [3.7 Dev.to API](#37-devto-api-setup)
  - [3.8 Google Sheets Integration](#38-google-sheets-integration-centralized-configuration---new-in-v4)
- [Step 4: Database Setup](#step-4-dual-vector-database-setup)
- [Step 5: Customize Workflow](#step-5-customize-the-workflow)
- [Step 6: Test and Deploy](#step-6-test-and-deploy)
- [Cost Analysis](#cost-analysis-v4)
- [Troubleshooting](#troubleshooting)
- [Advanced Configuration](#advanced-configuration)

---

## What's New in V4 Setup Changes

### New Services to Configure
1. **Google Gemini for Image Generation** (via Gemini node and API)
2. **MongoDB Memory Store** (new for agent coordination and link tracking)
3. **Google Sheets Integration** (for centralized category management)
4. **WordPress Internal Link Fetching** (new feature for intelligent inbound linking)
5. **WordPress Yoast SEO REST API Extension** (requires adding PHP code to functions.php)

### Simplified Configurations
1. **GPT-5 Models** (automatic via OpenAI - no extra config needed)
2. **Dual Vector Collections** (automatic creation on first run)
3. **Link Validator MCP Tool** (included in workflow)

---

## Architecture Overview

### V4 Content Generation Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│ STEP 1: Topic Discovery \+ Web Search                      │
│ - Discover topics via web search and clustering                │
│ - Store in 'news articles' collection (full metadata)           │
│ - Chunk and vectorize into 'news chunks' (semantic search)      │
│ - Deduplication via SHA256 content fingerprints                 │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 2: Topic Generation (GPT-5-mini)                           │
│ - Semantic clustering from vector database                      │
│ - Match to content pillars from Google Sheets                   │
│ - Score topics by freshness and relevance                       │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 3: Article Intelligence Agent (GPT-4o)                     │
│ - Extract: entities, keywords, facts, quotes, tone              │
│ - Generate LSI keywords for SEO                                 │
│ - Build foundation data structure                               │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 4: Multi-Agent Content Loop                                │
│                                                                 │
│ ┌─────────────────────────────────────────────────────────┐     │
│ │ Task Definition Agent (GPT-4.1-mini)                    │     │
│ │ - Plans 7 tasks (5 body + conclusion + FAQ)             │     │
│ │ - Assigns tone, goals, structure                        │     │
│ └──────────────────┬──────────────────────────────────────┘     │
│                    │                                            │
│                    ▼                                            │
│ ┌─────────────────────────────────────────────────────────┐     │
│ │ Content Writer Agent (GPT-4o) with MCP Tools            │     │
│ │ ┌─────────────────────────────────────────────────┐     │     │
│ │ │ - Web Search (outbound links)                   │     │     │
│ │ │ - Image Generator (Gemini)                      │     │     │
│ │ │ - Chart Generator (QuickChart)                  │     │     │
│ │ │ - Link Validator (health check)                 │     │     │
│ │ │ - Memory Tool (track inbound links)             │     │     │
│ │ │ - Think Tool (reasoning)                        │     │     │
│ │ └─────────────────────────────────────────────────┘     │     │
│ │ - Fetches WordPress posts for internal linking          │     │
│ │ - Writes sections with 1200 words target                │     │
│ │ - Generates FAQ with custom HTML/CSS                    │     │
│ └──────────────────┬──────────────────────────────────────┘     │
│                    │                                            │
│                    ▼                                            │
│ ┌─────────────────────────────────────────────────────────┐     │
│ │ Quality Check Agent (GPT-4o-mini)                       │     │
│ │ - Validates: Flesch score, SEO, links, structure        │     │
│ │ - Threshold: 70% quality score                          │     │
│ │ - Provides feedback for iteration (if needed)           │     │
│ └─────────────────────────────────────────────────────────┘     │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 5: Title Optimization (GPT-4.1-mini)                       │
│ - Generate 5 candidate titles                                   │
│ - Score on: emotion, clarity, SEO, clickability                 │
│ - Select highest scoring title                                  │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 6: Yoast SEO Metadata (GPT-4o-mini)                        │
│ - Generate: meta title, description, keywords                   │
│ - Create optimized slug (max 5 words)                           │
│ - Inject via WordPress REST API PUT request                     │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 7: Featured Image Generation                               │
│ - Gemini image generation                                       │
│ - Brand mascot (cat-themed) for engagement                      │
│ - Upload to WordPress media library                             │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 8: WordPress Draft Creation                                │
│ - Convert Markdown to HTML (GPT-4o-mini)                        │
│ - Insert featured image                                         │
│ - Apply category from Google Sheets mapping                     │
│ - Set status: draft                                             │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 9: Multi-Platform Distribution                             │
│ - Twitter: Image + caption + link                               │
│ - Dev.to: Markdown with your brand signature                    │
│ - Schedule via n8n timers (optional)                            │
└─────────────────────────────────────────────────────────────────┘
```

### V4 Database Architecture

```
MongoDB Atlas Cluster
│
├── Database: n8n
│   │
│   ├── Collection: blog v4
│   │
│   ├── Collection: news articles
│   │
│   ├── Collection: news chunks
│   │   └── Indexes: Vector search index (news_chunks_vector_index)
│   │
│   └── Collection: n8n_chat_histories
│       └── Context Window: 25 messages
```

---

## Prerequisites

Before starting, ensure you have:

- [ ] Basic understanding of n8n workflows
- [ ] Domain or subdomain for n8n (if self-hosting)
- [ ] Credit card for API services (OpenAI, Google AI Studio, etc.)
- [ ] WordPress site with admin access
- [ ] 1-2 hours for complete setup

---

## Step 1: Deploy Your n8n Instance

### Option A: n8n Cloud (Easiest)

1. Register at: [n8n Cloud Signup](https://n8n.partnerlinks.io/emp0)
2. Login and go to **Workflows** > **New Workflow**
3. Choose the standard plan ($20/month). V4 removed multiple scheduler nodes and reduces execution frequency, so the normal plan is sufficient.
4. Skip to [Step 2](#step-2-download-and-import-workflows)

**Cost**: $20/month (sufficient due to fewer scheduled executions)

### Option B: Railway Deployment (Recommended for Cost)

1. Go to: [Deploy n8n on Railway](https://railway.com/deploy/n8n-with-workers?referralCode=jay)
2. Click **Deploy Now**
3. Railway will provision:
   - n8n main instance
   - PostgreSQL database for n8n
   - Redis for queue management
4. Wait 3-5 minutes for deployment
5. Click on your n8n service and find the public URL
6. (Optional) Add a custom domain:
   - Go to **Settings** > **Domains**
   - Click **Generate Domain** or add custom
7. Visit your n8n URL and create admin credentials

**Cost**: $20-30/month (scales with usage)

![Railway Deployment Success](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-n8n-on-railway.png)

### Option C: Hostinger VPS (Budget Option)

1. Visit: [Hostinger VPS](https://www.hostinger.com/vps/n8n-hosting)
2. Choose VPS plan (minimum 2GB RAM recommended)
3. Install n8n using their 1-click installer or Docker
4. Follow [official n8n self-hosting guide](https://docs.n8n.io/hosting/installation/docker/)

**Cost**: $10-15/month

---

## Step 2: Download and Import Workflows

### Download the Workflow Package

After purchasing V4, you'll receive:

1. **Content Generator v4.1.json** (main workflow)
2. **mcp - chart generation.json** (QuickChart integration)
3. **mcp - image generation gemini nano banana pro.json** (Gemini node/API)
4. **mcp - web search.json** (internet search)
5. **mcp - check link status.json** (link validator)

### MCP Tools — Screenshots

Web Search MCP
![Web Search MCP](https://articles.emp0.com/wp-content/uploads/2025/10/content-generator-v4-mcp-web-search.png)

Image Generation (Gemini Nano banana pro) MCP
![Image Generation Gemini MCP](https://articles.emp0.com/wp-content/uploads/2025/12/content-generator-v4-mcp-image-generation-gemini-nano-banana-pro.png)

Check Link Status MCP
![Check Link Status MCP](https://articles.emp0.com/wp-content/uploads/2025/10/content-generator-v4-mcp-check-link-status.png)

Chart Generation MCP
![Chart Generation MCP](https://articles.emp0.com/wp-content/uploads/2025/10/content-generator-v4-mcp-chart-generation.png)

Download the files on [Gumroad](https://0emp0.gumroad.com/l/content-farming-v4)
![Gumroad Download](https://articles.emp0.com/wp-content/uploads/2025/10/content-generator-v4-download-gumroad.png)

OR Download the files from [AIA Skool Community](https://www.skool.com/aia-ai-automation-2762)
![Skool Download](https://articles.emp0.com/wp-content/uploads/2025/10/content-generator-v4-download-skool.png)

### Import to n8n

1. Open your n8n instance
2. Go to **Workflows** (left sidebar)
3. Click **Import Workflow** (top right)
4. Upload **Content Generator V4.json**
5. Repeat for all MCP tool workflows
6. Rename workflows for clarity:
   - `Content Generator V4`
   - `MCP - Chart Generator`
   - `MCP - Image Generator (Gemini)`
   - `MCP - Web Search`
   - `MCP - Link Validator`

![Import Workflow](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-tutorial-import-from-file.png)

---

## Step 3: Configure API Credentials

### 3.1 OpenAI API (GPT-5 Models)

**V4 uses GPT-5 models which may require waitlist access initially**

#### Get API Key

1. Visit: [OpenAI Platform](https://platform.openai.com/account/api-keys)
2. Sign up or log in
3. Click **Create new secret key**
4. Name it `n8n-content-generator-v4`
5. Copy the key (starts with `sk-proj-...`)

#### Add to n8n

1. In n8n, go to **Credentials** (left sidebar)
2. Click **Add Credential**
3. Search for **OpenAI**
4. Paste your API key
5. Test connection
6. Save as `OpenAI - GPT5 Content Gen`

#### Model Access Verification

V4 uses these models:
- `gpt-5-nano` (ultra-fast reasoning)
- `gpt-5-mini` (standard content)
- `gpt-4.1-mini` (title optimization)
- `gpt-4o` (complex reasoning)
- `gpt-4o-mini` (conversion tasks)

**Note**: If GPT-5 models are not yet available in your account, the workflow will fall back to GPT-4o. Update model names in workflow nodes once GPT-5 access is granted.

**Cost**:
- Initial credit: $5 free trial (for testing)
- Top-up: $50-100 recommended
- Monthly usage: $60-80 for ~300 articles

---

### 3.2 MongoDB Atlas (Dual Vector Database)

#### Create MongoDB Atlas Account

1. Register at: [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
2. Verify email and login

#### Create Cluster

1. Click **Build a Database**
2. Select **M0 Sandbox** (Free tier - good for 3-6 months)
3. Choose **AWS** as provider
4. Select region closest to your n8n server (e.g., `us-east-1`)
5. Name cluster: `content-generator-v4-cluster`
6. Click **Create**

#### Configure Database Access

1. Go to **Security** > **Database Access**
2. Click **Add New Database User**
3. Authentication: **Password**
4. Username: `n8n-user`
5. Password: Generate strong password (save it!)
6. Database User Privileges: **Read and write to any database**
7. Click **Add User**

#### Configure Network Access

1. Go to **Security** > **Network Access**
2. Click **Add IP Address**
3. For development: **Allow Access from Anywhere** (`0.0.0.0/0`)
4. For production: Add your specific n8n server IP
5. Click **Confirm**

#### Get Connection String

1. Go to **Deployment** > **Database**
2. Click **Connect** on your cluster
3. Choose **Connect your application**
4. Driver: **Node.js** version **4.1 or later**
5. Copy connection string

Example:
```
mongodb+srv://n8n-user:YOUR_PASSWORD@content-generator-v4-cluster.abc123.mongodb.net/contentgen?retryWrites=true&w=majority
```

Replace:
- `YOUR_PASSWORD` with your database user password
- `contentgen` with your database name (keep as `contentgen` for simplicity)

#### Add to n8n

1. In n8n, go to **Credentials**
2. Click **Add Credential**
3. Search for **MongoDB**
4. Paste connection string
5. Test connection (should show "Connected")
6. Save as `MongoDB - ContentGen V4`

**Cost**: Free (M0 cluster with 512MB storage)

---

### 3.3 MongoDB Vector Search Setup (CRITICAL for V4)

V4 uses a dual-collection architecture. You need to create **two vector indexes**.

#### Index 1: news chunks (Primary Vector Search)

1. In MongoDB Atlas, go to your cluster
2. Click **Search** tab

![Vector Index Setup](https://articles.emp0.com/wp-content/uploads/2025/08/setup-mongodb-vector-index-1.png)

3. Click **Create Search Index**
4. Choose **JSON Editor**
5. Database: `contentgen`
6. Collection: `news chunks`
7. Index Name: `news_chunks_vector_index`
8. Paste this JSON:

```json
{
  "fields": [
    {
      "numDimensions": 1536,
      "path": "embedding",
      "similarity": "dotProduct",
      "type": "vector"
    },
    {
      "path": "articleId",
      "type": "filter"
    },
    {
      "path": "createdDate",
      "type": "filter"
    }
  ]
}
```

9. Click **Create Search Index**
10. Wait 2-5 minutes for **Active** status

**⚠️ IMPORTANT**: The workflow will fail without these vector indexes. Do not skip this step.

---

### 3.4 Google Gemini (Image Generation via Node & API)

V4 now uses Google Gemini for image generation. You can generate images either with the n8n Gemini node or via HTTP API calls to Google AI Studio.

#### Get a Google AI Studio API Key

1. Visit Google AI Studio and create an API key.
2. Restrict the key to your use case as needed.

#### Use the n8n Gemini Node

1. In n8n, go to **Credentials** > **Add Credential**.
2. Select the Gemini/Google AI credential type and paste your API key.
3. In the workflow, open the image generation node and switch it to Gemini.
4. Choose an image-capable Gemini model and provide a concise prompt (e.g., "flat illustration of a tech blog hero image about {topic}").
5. Configure output to return a base64 image; downstream nodes decode and upload to WordPress.

---

### 3.5 WordPress API Setup

#### Enable WordPress REST API

1. Go to your WordPress admin: `yoursite.com/wp-admin`
2. Ensure REST API is enabled (default in WordPress 5.0+)
3. Test by visiting: `yoursite.com/wp-json/wp/v2/posts`
4. You should see JSON response (not 404)

#### Generate Application Password

1. Go to **Users** > **Profile**
2. Scroll to **Application Passwords**
3. Name: `n8n Content Generator V4`
4. Click **Add New Application Password**
5. Copy the generated password (format: `xxxx xxxx xxxx xxxx xxxx xxxx`)
6. Save this password (you can't view it again)

#### Add to n8n

1. In n8n, go to **Credentials**
2. Click **Add Credential**
3. Search for **HTTP Basic Auth**
4. Configure:
   - **Username**: Your WordPress username
   - **Password**: The application password (remove spaces: `xxxxxxxxxxxxxxxxxxxxxxxx`)
5. Save as `WordPress - Basic Auth`

#### Configure WordPress Nodes

The V4 workflow has multiple WordPress-related nodes:
1. **Fetch WordPress Posts** (for internal linking)
2. **Create Draft Post**
3. **Upload Featured Image**
4. **Set Yoast Metadata**

Update each node:
1. Set **Base URL**: `https://yoursite.com`
2. Select credential: `WordPress - Basic Auth`
3. Test connection

#### Enable Yoast SEO Meta Fields in WordPress REST API (CRITICAL for V4)

**NEW in V4**: To allow the workflow to update Yoast SEO meta description and focus keywords via REST API, you must add custom code to your WordPress theme.

**⚠️ IMPORTANT**: Without this step, Yoast SEO metadata will not be updated automatically.

##### Step 1: Access WordPress Theme Editor

1. Log in to WordPress Admin (`yoursite.com/wp-admin`)
2. Go to **Appearance** > **Theme File Editor**
3. If you see a warning about editing files directly, click **I understand**
4. In the right sidebar, find and click **Theme Functions** (`functions.php`)

![WordPress Theme Editor](https://articles.emp0.com/wp-content/uploads/2025/10/content-generator-v4-edit-theme-functions.png)

##### Step 2: Add Yoast SEO REST API Function

Scroll to the **bottom** of the `functions.php` file and paste this code:

```php
// Enable Yoast SEO fields in WordPress REST API for n8n Content Generator V4
function register_yoast_meta_in_rest() {
    register_rest_field('post', 'yoast_description', array(
        'get_callback' => function($post) {
            return get_post_meta($post->ID, '_yoast_wpseo_metadesc', true);
        },
        'update_callback' => function($value, $post) {
            return update_post_meta($post->ID, '_yoast_wpseo_metadesc', sanitize_text_field($value));
        },
        'schema' => array(
            'type' => 'string',
            'description' => 'Meta description for Yoast SEO',
        ),
    ));

    register_rest_field('post', 'yoast_keyword', array(
        'get_callback' => function($post) {
            return get_post_meta($post->ID, '_yoast_wpseo_focuskw', true);
        },
        'update_callback' => function($value, $post) {
            return update_post_meta($post->ID, '_yoast_wpseo_focuskw', sanitize_text_field($value));
        },
        'schema' => array(
            'type' => 'string',
            'description' => 'Focus keyword for Yoast SEO',
        ),
    ));
}
add_action('rest_api_init', 'register_yoast_meta_in_rest');
```

##### Step 3: Save Changes

1. Click **Update File** button at the bottom
2. You should see a success message: "File edited successfully"

##### Step 4: Verify REST API Access

Test that Yoast fields are now accessible via REST API:

1. Visit: `https://yoursite.com/wp-json/wp/v2/posts?per_page=1`
2. Look for `yoast_description` and `yoast_keyword` fields in the JSON response
3. If you see these fields, the setup is successful

**Example response snippet**:
```json
{
  "id": 123,
  "title": {"rendered": "Sample Post"},
  "yoast_description": "This is the meta description",
  "yoast_keyword": "focus keyword",
  ...
}
```

##### What This Code Does

- **Registers custom REST API fields** for Yoast SEO meta description and focus keyword
- **Allows n8n to read and write** these fields via WordPress REST API
- **Sanitizes input** to prevent security issues
- **Uses WordPress hooks** (`rest_api_init`) to integrate cleanly with REST API

##### Troubleshooting

**If fields don't appear in REST API:**

1. **Clear WordPress cache** (if using caching plugin)
2. **Verify Yoast SEO is installed and active**
3. **Check functions.php syntax**: Use [PHP Syntax Checker](https://phpcodechecker.com/)
4. **Test with a different post**: `yoursite.com/wp-json/wp/v2/posts/POST_ID`
5. **Check WordPress error logs** in cPanel or hosting dashboard

**If you get a white screen after saving:**

1. Don't panic - this means PHP syntax error
2. Access your site via FTP or cPanel File Manager
3. Edit `wp-content/themes/YOUR_THEME/functions.php`
4. Remove the code you just added
5. Check for missing semicolons or brackets
6. Try again with corrected code

**Alternative: Use Child Theme**

For better practice (especially if your theme updates frequently):

1. Create a child theme following [WordPress Child Theme Guide](https://developer.wordpress.org/themes/advanced-topics/child-themes/)
2. Add the code to the child theme's `functions.php`
3. This prevents your changes from being overwritten during theme updates

**Cost**: Free

---

### 3.6 Twitter API Setup (X Platform)

#### Apply for Twitter Developer Account

1. Visit: [Twitter Developer Portal](https://developer.twitter.com)
2. Sign up with your Twitter account
3. Apply for **Elevated Access** (required for posting)
4. Fill out the use case form (mention "automated blog distribution")
5. Wait for approval (usually 1-2 days)

#### Create Twitter App

1. In Developer Portal, go to **Projects & Apps**
2. Click **Create App**
3. Name: `n8n Content Generator V4`
4. Save your **API Key** and **API Secret Key**
5. Add your OAuth Redirect URL to the project settings


#### Add to n8n

1. In n8n, go to **Credentials**
2. Click **Add Credential**
3. Search for **Twitter OAuth2**
4. Fill in all four values:
   - **Client ID**
   - **Client Secret**
5. Save as `Twitter - OAuth2`
6. Press Connect and Authorize the connection

---

### 3.7 Dev.to API Setup

#### Get Dev.to API Key

1. Visit: [Dev.to Settings](https://dev.to/settings/account)
2. Scroll to **DEV API Keys**
3. Click **Generate API Key**
4. Name: `n8n Content Generator V4`
5. Copy the key

#### Add to n8n

Dev.to uses header-based authentication:

1. In your V4 workflow, find the **HTTP Request** node
2. Go to **Headers** section
3. Add header:
   - **Name**: `api-key`
   - **Value**: `Bearer api-key` (where api-key is the value from Dev.to)
4. Save workflow

---

### 3.8 Google Sheets Integration (Centralized Configuration - NEW in V4)

**Major V4 Feature**: Google Sheets provides centralized configuration for categories, company profile, and RSS sources — no workflow editing required for content strategy changes.

#### Step 1: Copy the Configuration Template

1. Visit: [Google Sheets Configuration Template](https://docs.google.com/spreadsheets/d/13WpGs3XpKyu2Hamq2-5VsgAkfcozo2Lje0xCqWb921Y/edit?usp=sharing)
2. Click **File** > **Make a copy**
3. Name it: `Content Generator V4 - [Your Blog Name]`
4. Save to your Google Drive

The template contains **3 sheets**: `category`, `data` (company profile), and `rss`

#### Step 2: Configure Sheet 1 - Categories

**Purpose**: Map your content pillars to WordPress categories

**Columns**:
- `name` - Category display name (e.g., "AI")
- `description` - Full description of what this category covers
- `id` - WordPress category ID number

**Example configuration:**
| name           | description                                                      | id |
|----------------|------------------------------------------------------------------|----|
| AI             | Articles on AI and artificial intelligence, ML, and AI trends    | 8  |
| Automation     | Articles on automation technologies and workflow optimization    | 9  |
| Business Ideas | Articles on innovation, startup concepts, and business models    | 7  |

![Gsheet category mapping](https://articles.emp0.com/wp-content/uploads/2025/10/content-generator-v4-gsheet-settings-category.png)

**How to find WordPress category ID**:
1. Go to WordPress Admin → **Posts** → **Categories**
2. Hover over your category name
3. Note the `tag_ID` parameter in the URL (e.g., `tag_ID=8`)

#### Step 3: Configure Sheet 2 - Company Profile

**Purpose**: Store brand/company profile details and workflow settings

**Columns**:
- `key` - Setting name (predefined)
- `value` - Your setting value

**Available settings (examples)**:
| key | description | example |
|-----|-------------|---------|
| COMPANY NAME | Your company name  | EMP0 |
| COMPANY PROFILE | Your company details | EMP0 is an AI Agency |
| ONLINE PROFILES | Your social URLs | https://emp0.com |
| MASCOT URL | Your mascot image URLs | https://emp0.com/logo.png |
| MASCOT DESCRIPTION | Describe your mascot | Green one eye octopus |
| BANNER IMAGE STYLE | Describe your banner image | Hyper realistic |

![Gsheet company profile and links](https://articles.emp0.com/wp-content/uploads/2025/12/content-gen-excel-data.png)


#### Step 4: Configure Sheet 3 - RSS Feeds

**Purpose**: Define content sources for ingestion and clustering

**Columns**:
- `url` - Full RSS feed URL
- `source` - Source name for attribution
- `category` - Content pillar (must match the `category` sheet)

**Example configuration:**
| url                                              | source       | category       |
|--------------------------------------------------|-------------|----------------|
| https://techcrunch.com/feed/                     | TechCrunch  | AI             |
| https://www.marktechpost.com/feed/               | MarkTechPost| AI             |
| https://feeds.bbci.co.uk/news/technology/rss.xml | BBC Tech    | Automation     |
| https://hackernoon.com/feed                      | HackerNoon  | Business Ideas |

Best practices:
- Add 5–10 feeds per category for diversity
- Mix authoritative sources (BBC, Wired) with niche sites
- Test RSS feeds before adding (open in browser to verify XML)

![Gsheet rss feed](https://articles.emp0.com/wp-content/uploads/2025/10/content-generator-v4-gsheet-settings-rss.png)

#### Step 5: Create Google Sheets Credential in n8n

1. Follow: [n8n Google Sheets OAuth Setup](https://docs.n8n.io/integrations/builtin/credentials/google/oauth-single-service/)

**Quick steps**:
1. Create Google Cloud Project at [Google Cloud Console](https://console.cloud.google.com/)
2. Enable **Google Sheets API**
3. Create **OAuth 2.0 credentials**:
   - Authorized redirect URI: `https://your-n8n-instance.com/rest/oauth2-credential/callback`
4. In n8n, add **Google Sheets OAuth2** credential
5. Connect and authorize
6. Save as `Google Sheets - Content Gen V4`

#### Step 6: Update Workflow to Use Your Sheet

1. Open **Content Generator V4** workflow
2. Find node: **Get categories**
3. Update **Document ID**:
   - Copy your Google Sheet URL
   - Extract ID between `/d/` and `/edit`
   - Paste into Document ID field
4. Select credential: `Google Sheets - Content Gen V4`
5. Test the node

Repeat for:
- **Get categories** node 
- **Get company profile** node
- **Get RSS feeds** node

#### Benefits of Google Sheets Configuration

- ✅ **No workflow editing** - Update categories without touching n8n
- ✅ **Multi-site management** - One sheet per blog, switch by changing Document ID
- ✅ **Team collaboration** - Share with content team for easy updates
- ✅ **Version control** - Google Sheets tracks all changes
- ✅ **Dynamic scaling** - Add new categories/settings without redeployment

**Cost**: Free

---

## Step 4: Initialize Collections

1. Open the V4 workflow
2. Find the first node: **Scheduler / Orchestrator Start**
3. Manually execute **Test workflow** (to the left of the clock node in step 1)
4. Wait for completion (5-10 minutes for first run)
5. Verify in MongoDB Atlas:
   - Collections created
   - Data populated in `news articles` and `news chunks`

---

## Step 5: Customize the Workflow

### 5.1 Set Timezone

1. Open the V4 workflow
2. Find all **Schedule Trigger** nodes (Steps 1-9)
3. For each scheduler:
   - Click the node
   - Find **Timezone** field
   - Set to your timezone (e.g., `America/New_York`, `Asia/Tokyo`)
4. Save workflow

![Set Timezone](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-tutorial-adjust-timezone.png)

**Recommended Schedule**:
- **Step 1 (Ingestion)**: 7:00 AM daily
- **Steps 3-8**: Every 2 hours from 9 AM to 5 PM (5 articles/day)

### 5.2 Configure Content Pillars (via Google Sheets)

All content pillars are managed in Google Sheets (no editing inside the workflow).

1. Open your copied Google Sheet
2. Update category sheet (columns: 
ame, description, id)
3. In n8n, ensure nodes use this range:
   - Get categories: category!A:C`r
4. Verify WordPress category IDs in the sheet match your site

### 5.3 Configure Internal Linking

V4's intelligent internal linking fetches your WordPress posts.

1. Find node: **Fetch WordPress Posts**
2. Verify settings:
   - **Method**: GET
   - **URL**: `{{$parameter.baseUrl}}/wp-json/wp/v2/posts?_embed&per_page=100`
   - **Authentication**: WordPress - Basic Auth
3. Adjust `per_page` if you have more than 100 posts (max: 100 per request)
4. The Content Writer Agent will automatically:
   - Parse Yoast SEO metadata (`yoast_head_json.og_url`, `og_title`)
   - Insert maximum 3 internal links per article
   - Track link usage via MongoDB Memory

**Memory Configuration**:
1. Find node: **Memory**
2. Verify settings:
   - **Type**: MongoDB Chat Memory
   - **Database**: `contentgen`
   - **Collection**: `chat_memory`
   - **Context Window Length**: 25

### 5.4 Adjust Quality Threshold

1. Find node: **Quality Check Agent**
2. Locate the success condition expression
3. Default: `>= 7` (70% quality score)
4. Recommended range: `7-9` (70-90%)

![Quality Threshold](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v3-tutorial-adjust-quality-threshold.png)

### 5.5 Configure Max Iterations

V4 uses a task-based iteration system.

1. Find node: **Content Writer Agent**
2. Settings:
   - **Max Iterations**: 25 (per task)
   - **Max Tries**: 5 (retry on failure)
   - **Retry on Fail**: true
3. Recommended for cost optimization:
   - Reduce **Max Tries** to 3
   - Quality threshold to 6-7

---

## Step 6: Test and Deploy

### 6.1 Test MCP Tools Individually

Before running the main workflow, test each MCP tool:

#### Test Image Generator
1. Open **MCP - Image Generator (Gemini)**
2. Click **Test workflow**
3. Provide a sample prompt: "A professional blog header image about artificial intelligence with a cat mascot"
4. Verify image URL is returned
5. Confirm the Gemini node/API returned an image and it uploaded successfully

#### Test Web Search
1. Open **MCP - Web Search**
2. Click **Test workflow**
3. Search query: "latest AI trends 2025"
4. Verify search results returned

#### Test Link Validator
1. Open **MCP - Link Validator**
2. Click **Test workflow**
3. Test URL: `https://www.google.com`
4. Verify status: `200` (valid)
5. Test invalid URL: `https://thisdoesnotexist123456.com`
6. Verify status: `404` or timeout

#### Test Chart Generator
1. Open **MCP - Chart Generator**
2. Click **Test workflow**
3. Provide sample data
4. Verify chart URL returned from QuickChart

### 6.2 Test Main Workflow (Step by Step)

#### Test Step 1: Topic Discovery & Web Search

1. Open **Content Generator V4**
2. Find node: **Scheduler / Orchestrator Start (Step 1)**
3. Click **Execute Node** (right-click menu)
4. Wait 5-10 minutes
5. Verify in MongoDB Atlas:
   - `news articles` collection has documents
   - `news chunks` collection has documents with embeddings
6. Check for duplicate prevention via `contentFingerprint`

#### Test Step 2: Topic Generation

1. Find node: **Schedule - Topic Selection (Step 2)**
2. Click **Execute Node**
3. Wait 2-3 minutes
4. Verify in MongoDB Atlas:
   - `generated_articles` collection has new document
   - Document has `completedStep: 2`
   - Topic is selected from content pillars

#### Test Step 3: Article Intelligence

1. Find node: **Schedule - Article Intelligence (Step 3)**
2. Click **Execute Node**
3. Wait 1-2 minutes
4. Verify output includes:
   - Main keywords
   - Related keywords
   - Named entities
   - Facts and quotes
   - Tone analysis

#### Test Step 4: Content Generation (CRITICAL)

1. Find node: **Schedule - Content Loop (Step 4)**
2. Click **Execute Node**
3. **Warning**: This is the longest step (10-30 minutes)
4. Monitor execution:
   - Task Definition Agent plans 7 tasks
   - Content Writer Agent processes each task
   - Memory Tool tracks internal links
   - Quality Check Agent validates output
5. Verify in MongoDB Atlas:
   - Article document updated with content sections
   - `completedStep: 4`
   - FAQ section generated
   - Internal links inserted (max 3)

**Troubleshooting**:
- If it fails, check OpenAI API quota
- Verify WordPress posts are fetched correctly
- Check MongoDB Memory collection for link tracking
- Ensure vector index is active

#### Test Step 5-9: Post-Processing

1. Execute nodes sequentially:
   - Step 5: Title Optimization
   - Step 6: Metadata Generation
   - Step 7: Image Generation
   - Step 8: WordPress Draft Creation
   - Step 9: Distribution
2. Verify after each step:
   - MongoDB `completedStep` increments
   - WordPress draft appears in admin
   - Twitter post published
   - Dev.to article created

### 6.3 End-to-End Test

1. Activate all schedule triggers
2. Wait for the full 24-hour cycle
3. Monitor:
   - n8n execution logs
   - MongoDB collections
   - WordPress drafts
   - Twitter feed
   - Dev.to profile
4. Check for errors in execution history

### 6.4 Deploy to Production

1. Verify all tests pass
2. Activate workflow (toggle in top right)
3. Set up monitoring:
   - n8n error notifications (email or Slack)
   - MongoDB Atlas alerts (storage, connections)
   - OpenAI API usage alerts
4. Schedule daily backups of MongoDB collections

---

## Cost Analysis V4

### Monthly Cost Breakdown

| Service | V3 Cost | V4 Cost | Savings | Notes |
|---------|---------|---------|---------|-------|
| **OpenAI API** | $100 | $60-80 | $20-40 | GPT-5 models more efficient |
| **Image Generation** | $20-30 | $10-15 | $10-15 | Gemini (node/API) |
| **MongoDB Atlas** | Free | Free | - | M0 cluster (512MB) |
| **n8n Hosting** | $20 | $20-30 | - | Railway or n8n Cloud |
| **Google AI (Gemini)** | - | Included above | - | Image generation API |
| **Google Sheets** | - | Free | - | New in V4 |
| **Twitter API** | Free | Free | - | Unchanged |
| **Dev.to** | Free | Free | - | Unchanged |
| **TOTAL** | ~$150 | ~$100-110 | **$40-50** | **35% reduction** |

### Cost per Article

- **V3**: $150 / 300 articles = **$0.50 per article**
- **V4**: $100 / 300 articles = **$0.33 per article**
- **Savings**: **34% per article**

### ROI Analysis

Assuming market rate of $50 per 1200-word article:

- **Monthly value generated**: 300 articles × $50 = **$15,000**
- **Monthly cost**: **$100**
- **Gross margin**: **99.3%**
- **Break-even**: **2 articles/month** (achieved in first day)

### Scaling Costs

| Articles/Day | Monthly Total | V4 Cost | Cost/Article | Notes |
|--------------|---------------|---------|--------------|-------|
| 5 | 150 | $55 | $0.37 | Low volume |
| 10 | 300 | $100 | $0.33 | Standard (default) |
| 20 | 600 | $180 | $0.30 | High volume |
| 50 | 1500 | $420 | $0.28 | Enterprise |

**Note**: At higher volumes, MongoDB may require M10 cluster ($0.08/hr = $57/month) for better vector search performance.

---

## Troubleshooting

### Common Issues and Solutions

#### 1. Vector Search Not Working

**Symptoms**:
- "Vector index not found" error
- No results from semantic search

**Solutions**:
1. Verify vector index is **Active** in MongoDB Atlas (Search tab)
2. Check index name matches: `news_chunks_vector_index`
3. Ensure embeddings have 1536 dimensions
4. Verify cosine similarity is configured
5. Re-run Step 1 to populate embeddings

#### 2. GPT-5 Models Not Available

**Symptoms**:
- "Model not found" error from OpenAI
- Workflow uses GPT-4o instead

**Solutions**:
1. Check OpenAI account for GPT-5 access (may be waitlisted)
2. Temporarily update workflow nodes to use GPT-4o
3. Replace model names:
   - `gpt-5-nano` → `gpt-4o-mini`
   - `gpt-5-mini` → `gpt-4o-mini`
   - Keep `gpt-4o` and `gpt-4.1-mini` unchanged
4. Wait for GPT-5 access and update back

#### 3. Internal Links Not Inserted

**Symptoms**:
- Articles have no internal links
- Memory tool errors

**Solutions**:
1. Verify WordPress REST API is accessible: `yoursite.com/wp-json/wp/v2/posts`
2. Check Application Password is correct (no spaces)
3. Ensure Yoast SEO plugin is installed (for `yoast_head_json`)
4. Verify MongoDB Memory collection exists: `n8n_chat_histories`
5. Check Content Writer Agent memory tool configuration

#### 4. Image Generation Fails

**Symptoms**:
- No featured images on WordPress
- Gemini API errors

**Solutions**:
1. Verify Gemini API key is valid and not restricted
2. Check Google AI Studio usage/quota limits
3. Ensure Gemini credentials are configured on the node (or API key query param is set)
4. Test the image generation node independently with a simple prompt
5. Review execution logs for response errors

#### 5. Broken Links in Content

**Symptoms**:
- WordPress draft has 404 links
- Link validator didn't catch them

**Solutions**:
1. Verify Link Validator MCP workflow is active
2. Check Content Writer Agent instructions include link validation
3. Ensure link validator has proper timeout settings (10-15 seconds)
4. Review link validator logic for edge cases
5. Manually review links before publishing

#### 6. Articles Too Short or Too Long

**Symptoms**:
- Articles consistently under 1000 words
- Articles exceed 2000 words

**Solutions**:
1. Adjust Task Definition Agent instructions
2. Update word count target in system prompt
3. Modify Quality Check Agent scoring for word count
4. Check if Content Writer Agent is hitting token limits
5. Adjust max iterations if quality threshold is too high

#### 7. MongoDB Connection Errors

**Symptoms**:
- "Failed to connect to MongoDB" error
- Timeout errors

**Solutions**:
1. Verify connection string has correct password (no special chars unescaped)
2. Check Network Access whitelist includes your n8n server IP
3. Ensure database name is correct: `contentgen`
4. Test connection in n8n credential settings
5. Check MongoDB Atlas cluster status (not paused)

#### 8. WordPress Draft Not Created

**Symptoms**:
- Article generated but not in WordPress
- REST API errors

**Solutions**:
1. Test WordPress REST API: `yoursite.com/wp-json/wp/v2/posts`
2. Verify Application Password permissions (read/write)
3. Check if WordPress site requires HTTPS
4. Ensure category IDs exist in WordPress
5. Review WordPress error logs in admin

#### 9. Out of Memory Errors (n8n)

**Symptoms**:
- Workflow crashes mid-execution
- "JavaScript heap out of memory" error

**Solutions**:
1. Upgrade n8n instance (Railway: scale to 2GB+ RAM)
2. Reduce batch size in splitInBatches nodes
3. Lower context window in Memory configuration (25 → 15)
4. Process fewer articles per run
5. Enable garbage collection in n8n environment variables

#### 10. High OpenAI Costs

**Symptoms**:
- Costs exceed $100/month
- Token usage spikes

**Solutions**:
1. Reduce max iterations (25 → 15)
2. Lower quality threshold (8 → 7)
3. Decrease word count target (1200 → 1000)
4. Limit internal links (3 → 2)
5. Use GPT-5-nano for more tasks
6. Reduce context window in Memory (25 → 15)
7. Disable image generation for some articles

---

## Advanced Configuration

### Fine-Tuning Content Quality

#### Readability Settings

Edit the Content Writer Agent prompt:

```
Target Flesch Reading Ease: 60+ (college level)
Average Sentence Length: Under 20 words
Transition Words: Minimum 30% of sentences
Active Voice: Mandatory preference
Paragraph Length: 3-5 sentences maximum
```

Adjust to your audience:
- **Technical audience**: Flesch 50-60, longer sentences
- **General audience**: Flesch 60-70, shorter sentences
- **Youth audience**: Flesch 70-80, very short sentences

#### SEO Optimization

Modify the Quality Check Agent scoring:

```javascript
// Current: 80% threshold
$json.seoScore >= 8

// More lenient: 70% threshold
$json.seoScore >= 7

// Strict: 90% threshold (may cause infinite loops)
$json.seoScore >= 9
```

### Scaling to Multiple Sites

#### Multi-Site Architecture

To manage multiple WordPress sites:

1. Duplicate the main workflow
2. Create separate credentials for each site:
   - WordPress Basic Auth
   - MongoDB collections (add site prefix)
3. Use different Google Sheets for categories per site
4. Set different schedules to avoid overlap
5. Tag MongoDB documents with site ID

#### WordPress Multisite Support

For WordPress Multisite:

1. Update WordPress API URLs to include site path:
   ```
   https://yoursite.com/site1/wp-json/wp/v2/posts
   ```
2. Use separate Application Passwords per subsite
3. Configure category mappings per subsite

### Custom Agent Behaviors

#### Modify Content Writer Agent

To change writing style:

1. Open workflow node: **Content Writer Agent**
2. Find the system prompt
3. Add custom instructions:

```
Writing Style: [Professional / Casual / Technical / Humorous]
Tone: [Authoritative / Friendly / Educational / Conversational]
POV: [First person (I, we) / Second person (you) / Third person]
Brand Voice: [Your brand voice guidelines]
Prohibited Words: [List of words to avoid]
Required Phrases: [Brand taglines, CTAs]
```

Example for casual tech blog:
```
Writing Style: Casual and approachable
Tone: Friendly and educational, like explaining to a smart friend
POV: Second person (you) with occasional first person (we)
Avoid: Corporate jargon, excessive buzzwords
Include: Practical examples, relatable analogies
```

#### Modify Quality Check Agent

To adjust scoring criteria:

1. Open workflow node: **Quality Check Agent**
2. Update scoring weights:

```javascript
// Default scoring
{
  "seoScore": 0.30,      // 30% weight on SEO
  "readability": 0.25,   // 25% weight on readability
  "structure": 0.20,     // 20% weight on structure
  "linkQuality": 0.15,   // 15% weight on links
  "engagement": 0.10     // 10% weight on engagement
}

// SEO-focused (e-commerce, affiliate)
{
  "seoScore": 0.40,
  "readability": 0.20,
  "structure": 0.15,
  "linkQuality": 0.20,
  "engagement": 0.05
}

// Engagement-focused (content marketing)
{
  "seoScore": 0.20,
  "readability": 0.30,
  "structure": 0.15,
  "linkQuality": 0.10,
  "engagement": 0.25
}
```

---

## Performance Optimization

### Reduce Execution Time

1. **Parallel Processing**: Enable for independent tasks
2. **Reduce API Calls**: Cache WordPress posts for 24 hours
3. **Lower Max Iterations**: 25 → 15 for Content Writer Agent
4. **Use GPT-5-nano**: For all non-creative tasks
5. **Optimize Vector Search**: Reduce `topK` from 10 to 5

### Reduce Token Usage

1. **Shorter System Prompts**: Remove verbose instructions
2. **Lower Context Window**: Memory 25 → 15 messages
3. **Reduce Word Count**: 1200 → 1000 words
4. **Fewer Examples**: In agent prompts
5. **Use Streaming**: Enable for GPT models (reduces waiting)

### Reduce Storage Costs

1. **Auto-Cleanup**: Delete `news chunks` older than 30 days
2. **Compression**: Enable MongoDB compression (M10+ clusters)
3. **Selective Vectorization**: Only vectorize relevant content
4. **Deduplication**: Aggressive content fingerprinting

---

## Security Best Practices

1. **API Keys**: Store in n8n credentials, never in workflow JSON
2. **WordPress**: Use Application Passwords, not main password
3. **MongoDB**: Whitelist specific IPs, not `0.0.0.0/0` in production
4. **n8n**: Enable 2FA, use strong admin password
5. **Backups**: Export workflows weekly, backup MongoDB monthly
6. **Rate Limits**: Respect API rate limits to avoid bans
7. **Monitoring**: Set up alerts for failed executions

---

## Getting Help

### Documentation Resources

- [n8n Official Docs](https://docs.n8n.io/)
- [OpenAI API Reference](https://platform.openai.com/docs)
- [MongoDB Atlas Docs](https://docs.atlas.mongodb.com/)
- [WordPress REST API Handbook](https://developer.wordpress.org/rest-api/)
- [Google AI Studio (Gemini) Docs](https://ai.google.dev/)

### Community Support

- **Discord**: [Join AI + Automation Discord](https://discord.gg/qg3qVfFchV)
- **Email**: tools@emp0.com
- **GitHub Issues**: [Report bugs](https://github.com/Jharilela/n8n-workflows/issues)
- **n8n Community**: [n8n Forum](https://community.n8n.io/)

### Professional Services

Need custom modifications or white-label deployment?

- **Setup Service**: $500 - Full installation and configuration
- **Custom Integration**: $25/hour - Add custom tools or APIs
- **White-Label**: $2000 - Fully branded, multi-site deployment
- **Consulting**: $150/hour - Strategy and optimization

Contact: tools@emp0.com

---

## Conclusion

You now have a complete, production-ready AI content generation system powered by GPT-5, intelligent internal linking, FAQ schema, and enterprise-grade validation.

**Next Steps**:

1. Complete all credential configurations
2. Test each step thoroughly
3. Customize for your niche and brand voice
4. Activate the workflow
5. Monitor performance for first week
6. Optimize based on results
7. Scale to multiple content pillars

**Remember**: Content quality > quantity. Start with 5-10 articles/day and optimize before scaling.

---

**Questions? Need help?**

📧 Email: tools@emp0.com
💬 Discord: @jym.god
🌐 Website: emp0.com
📚 GitHub: github.com/Jharilela

---

**Made with ❤️ by the EMP0 team**

*Automating content creation, one workflow at a time.*






