# Shopify Blog Automation - Technical Setup Guide

This comprehensive guide walks you through setting up the AI powered Shopify blog content generator from scratch. Follow each section carefully to ensure everything works correctly.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Workflow Import & Configuration](#workflow-import--configuration)
3. [n8n Setup](#n8n-setup)
4. [MongoDB Atlas Setup](#mongodb-atlas-setup)
5. [Google Sheets Configuration](#google-sheets-configuration)
6. [API Keys & Credentials](#api-keys--credentials)
7. [Shopify API Setup](#shopify-api-setup)
8. [MCP Server Configuration](#mcp-server-configuration)
9. [Testing Your Workflow](#testing-your-workflow)
10. [Cost Breakdown](#cost-breakdown)
11. [Customization Guide](#customization-guide)
12. [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before starting, ensure you have:

- [ ] Active n8n instance (cloud or self hosted)
- [ ] Shopify store with blog enabled
- [ ] MongoDB Atlas account (free tier available)
- [ ] Google Gemini API key (free tier available)
- [ ] OpenAI API key (for embeddings)
- [ ] Google account for Sheets
- [ ] Twitter Developer account (optional)

**Estimated Setup Time**: 60 to 90 minutes

---

## Workflow Import & Configuration

### Step 1: Get the Workflow Files

First, obtain all 4 workflow JSON files:

**Option 1: Purchase the workflow**
- Visit [0emp0.gumroad.com/l/shopify-blog-automation](https://0emp0.gumroad.com/l/shopify-blog-automation)
- Complete your purchase
- Download all workflow files:
  - `Shopify Blog Automation.json` (main workflow)
  - `tool - gemini.json` (Gemini image generator tool)
  - `tool - web search.json` (Web search tool)
  - `tool - check link status.json` (Link validator tool)

**Option 2: Join the community**
- Join our [Skool community](https://www.skool.com/aia-ai-automation-2762)
- Access workflow files in Classroom → Expert Automation → Shopify Content Generator

### Step 2: Import All Workflows

Import each workflow file to your n8n instance:

1. In n8n, click **"Workflows"** in the sidebar
2. Click **"+ Add Workflow"**
3. Click the **three dots** menu (⋮) → **"Import from File"**
4. Select `Shopify Blog Automation.json`
5. Repeat for the 3 tool workflows:
   - `tool - gemini.json`
   - `tool - web search.json`
   - `tool - check link status.json`

**Important**: Import all 4 workflows before proceeding with configuration.

---

## n8n Setup

1. **Sign up for n8n Cloud**
   - Visit [n8n.cloud](https://n8n.partnerlinks.io/emp0)
   - Choose a plan (Starter plan at $20/month recommended)
   - Complete account registration

2. **Access your n8n instance**
   - Log into your cloud dashboard
   - Click "Open n8n" to access the workflow editor

---

## MongoDB Atlas Setup

MongoDB Atlas stores article embeddings and prevents duplicate content through vector search.

### Step 1: Create MongoDB Atlas Account

1. Visit [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
2. Sign up for free account
3. Create new project: `n8n-content-generator`

### Step 2: Create a Cluster

1. Click **"Build a Database"**
2. Choose **"M0 FREE"** tier (512MB storage, perfect for starting)
3. Select cloud provider and region (choose closest to your n8n instance)
4. Cluster name: `ContentGenerator`
5. Click **"Create"**

### Step 3: Configure Database Access

1. In Atlas dashboard, go to **"Database Access"**
2. Click **"Add New Database User"**
3. Authentication Method: **Password**
4. Username: `n8n_content_bot`
5. Generate secure password and **save it**
6. Database User Privileges: **"Read and write to any database"**
7. Click **"Add User"**

### Step 4: Configure Network Access

1. Go to **"Network Access"**
2. Click **"Add IP Address"**
3. For testing: Click **"Allow Access from Anywhere"** (0.0.0.0/0)
4. For production: Add your n8n instance's IP address
5. Click **"Confirm"**

### Step 5: Create Database and Collections

1. Go to **"Database"** → **"Browse Collections"**
2. Click **"Add My Own Data"**
3. Create the following collections in database `n8n`:

**Collection 1: news_articles**
![MongoDB News Articles](https://articles.emp0.com/wp-content/uploads/2025/12/MongoDB-news-articles.png)
- Stores original RSS feed articles
- Used for content aggregation and deduplication

**Collection 2: news_chunks**
![MongoDB News Chunks](https://articles.emp0.com/wp-content/uploads/2025/12/MongoDB-news-chunks.png)
- Stores processed article chunks with embeddings
- Used for semantic search and topic matching

**Collection 3: shopify blog**
![MongoDB Shopify Blog Database](https://articles.emp0.com/wp-content/uploads/2025/12/MongoDB-shopify-blog.png)
- Stores generated blog posts
- Tracks publication status and metadata

### Step 6: Create Vector Search Index

**Step 1: Navigate to Search & Vector Search**

![MongoDB Vector Search Index Step 1](https://articles.emp0.com/wp-content/uploads/2025/12/MongoDB-vector-search-index-step-1.png)

1. On the left sidebar, click on **"Search & Vector Search"** under DATABASE
2. Click **"Create Search Index"**
3. Choose **"Vector Search"** (for semantic search and AI embeddings)
4. Under **"Index Name and Data Source"**:
   - Index Name: `shopify_news_chunks_vector_index`
   - Database and Collection: Select `news_chunks`
5. Under **"Configuration Method"**, select **"Visual Editor"**
6. Click **"Next"**

**Step 2: Configure Vector Field**

![MongoDB Vector Search Index Step 2](https://articles.emp0.com/wp-content/uploads/2025/12/MongoDB-vector-search-index-step-2.png)

7. In the **"Vector Field"** section, configure:
   - **Path**: `embedding` (the field name where embeddings are stored)
   - **Number of Dimensions**: `1536` (for OpenAI text-embedding-3-small)
   - **Similarity Method**: `Dot Product` (or `cosine` - both work with OpenAI embeddings)

8. In the **"Filter Field"** section:
   - **Path**: `createdDate` (allows filtering by date in searches)

9. Click **"Next"**

**Step 3: Review and Create**

![MongoDB Vector Search Index Step 3](https://articles.emp0.com/wp-content/uploads/2025/12/MongoDB-vector-search-index-step-3.png)

10. Review your configuration:
    - **Vector Path**: embedding
    - **Number of Dimensions**: 1536
    - **Similarity Method**: dotProduct
    - **Filter Path**: createdDate

11. Click **"Create Vector Search Index"**

**Important Notes:**
- Create this index on the `news_chunks` collection, NOT the `articles` collection
- The index will take a few minutes to build
- You'll see "Initial Sync" status, then "Active" when ready
- The `embedding` field must contain 1536-dimensional vectors from OpenAI's text-embedding-3-small model

### Step 7: Get Connection String

1. Go to **"Database"** → Click **"Connect"**
2. Choose **"Connect your application"**
3. Driver: **Node.js** version 4.1 or later
4. Copy the connection string
5. Replace `<password>` with your database user password
6. Replace `<dbname>` with `shopify_blog`
7. **Save this connection string** - you'll need it for n8n

Example:
```
mongodb+srv://YOUR_USERNAME:YOUR_PASSWORD@contentgenerator.xxxxx.mongodb.net/shopify_blog?retryWrites=true&w=majority
```

---

## Google Sheets Configuration

Google Sheets manages RSS feeds and blog categories.

### Step 1: Enable Google APIs

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create new project or select existing one
3. Navigate to **"APIs & Services"** → **"Library"**
4. Enable:
   - **Google Sheets API**
   - **Google Drive API**

### Step 2: Create Configuration Spreadsheet

![Google Sheets Copy Template](https://articles.emp0.com/wp-content/uploads/2025/12/gsheet-copy-document.png)

1. **[Click here to copy our pre-built template](https://docs.google.com/spreadsheets/d/1JGOz37XqWQXFOw7GGQkd6CUXouolSnUzqDk0IPM1Wd4/copy)**
2. Click **"Make a copy"** to create your own version
3. This will create a copy in your Google Drive with:
   - Pre-configured columns (RSS Feeds, Categories, details)
   - Example RSS feeds
   - Sample categories
4. **Update the `details` sheet** with your Shopify store information
5. Customize with your own RSS feeds and categories

**Option 2: Create from Scratch**

Create a new Google Sheet named `Shopify Blog Config` with three sheets (tabs):

### Sheet 1: RSS Feeds

![RSS Feeds Configuration](https://articles.emp0.com/wp-content/uploads/2025/12/ghseet-rss.png)

Create columns:

| Feed URL | Category | Active |
|----------|----------|--------|
| https://example.com/fitness/feed | Fitness | TRUE |
| https://example.com/nutrition/feed | Nutrition | TRUE |
| https://example.com/wellness/feed | Self-Care | TRUE |

**Example RSS Feeds by Niche:**

**Wellness/Fitness:**
- `https://www.mindbodygreen.com/rss`
- `https://www.wellandgood.com/feed/`
- `https://greatist.com/feed`

**Tech/Startup:**
- `https://techcrunch.com/feed/`
- `https://www.theverge.com/rss/index.xml`
- `https://news.ycombinator.com/rss`

**Marketing:**
- `https://neilpatel.com/feed/`
- `https://blog.hubspot.com/rss.xml`
- `https://moz.com/blog/feed`

#### Sheet 2: Categories

![Categories Configuration](https://articles.emp0.com/wp-content/uploads/2025/12/ghseet-categories.png)

Create columns:

| Category Name | Shopify Handle | Description |
|---------------|----------------|-------------|
| Fitness | fitness | Workout tips and exercise routines |
| Nutrition | nutrition | Healthy eating and recipes |
| Self-Care | self-care | Wellness and mental health |
| Personal Growth | personal-growth | Productivity and habits |

**How to Find Category IDs in Shopify:**

![Shopify Category ID](https://articles.emp0.com/wp-content/uploads/2025/12/Shopify-category-id.png)

1. In Shopify Admin, go to **Content** → **Blog posts**
2. Click on **"Manage blogs"** at the top
3. Click on a blog category (e.g., "Nutrition")
4. Look at the URL in your browser - the category ID is at the end:
   ```
   https://admin.shopify.com/store/yourstore/content/blogs/[CATEGORY_ID]
   ```
   Example: `https://admin.shopify.com/store/pinkmatchaco/content/blogs/120191779097`

5. The category ID in this example is: `120191779097`
6. The category handle is visible in the "Search engine listing" section (e.g., `nutrition`)

**Important Notes:**
- The **Shopify Handle** must match exactly what appears in your Shopify blog category settings
- Category handles are lowercase and use hyphens (e.g., `self-care`, not `Self Care`)
- You can find all your blog categories in Shopify Admin → Content → Blog posts → Manage blogs

### Sheet 3: details

![Details Sheet Configuration](https://articles.emp0.com/wp-content/uploads/2025/12/ghseet-details.png)

**IMPORTANT**: This sheet contains your Shopify store configuration and branding information.

Create 8 columns with the following structure:

| Column Name | Description | Example |
|-------------|-------------|---------|
| **domain** | Your store domain (without https://) | www.pinkmatcha.co |
| **Brand Name** | Your store/brand name | Pink Matcha |
| **Shopify Store Subdomain** | Your .myshopify.com subdomain | pinkmatchaco |
| **Shopify Link** | Your full store URL | https://pinkmatcha.co/ |
| **Theme Id** | Your Shopify theme ID | 183750230297 |
| **Branding Logo** | URL to your brand logo/product image | https://cdn.shopify.com/s/files/1/0968/4280/9625/files/Pink_Matcha_Drink.png?v=1766335545 |
| **Branding Style** | Style instructions for AI image generation | Image generated should contain a girl holding this drink |
| **Image Style** | Additional image generation guidelines | Generate a wide, natural-looking scene with no square canvas... |

**How to fill in each field:**

1. **domain**: Your primary domain without https:// or www (if applicable)
2. **Brand Name**: The name of your store as you want it referenced in content
3. **Shopify Store Subdomain**: Found in Shopify Admin → Settings → Store details (yourstore.myshopify.com)
4. **Shopify Link**: Your full public-facing store URL with https://
5. **Theme Id**: See instructions below on how to find your theme ID
6. **Branding Logo**: CDN URL to your logo or signature product image
7. **Branding Style**: Instructions for AI when generating branded images (optional)
8. **Image Style**: Technical requirements for image generation (aspect ratio, composition, etc.)

**How to Find Your Shopify Theme ID:**

![Shopify Theme Page](https://articles.emp0.com/wp-content/uploads/2025/12/Shopify-theme-page.png)

**Step 1: Navigate to Themes**
1. In Shopify Admin, go to **Online Store** → **Themes**
2. Find your current active theme (marked with "Current theme")

![Shopify Theme Edit](https://articles.emp0.com/wp-content/uploads/2025/12/Shopify-theme-edit.png)

**Step 2: Access Theme Editor**
3. Click the **"Customize"** button on your current theme
4. Look at the URL in your browser's address bar
5. The Theme ID is the long number in the URL after `/themes/`

**Example URL:**
```
https://admin.shopify.com/store/pinkmatchaco/themes/183750230297/editor
```

In this example, the Theme ID is: `183750230297`

**Alternative Method:**
1. Click the **three dots (⋯)** button on your theme
2. Select **"Edit code"**
3. Look at the URL - the Theme ID appears after `/themes/`:
   ```
   https://admin.shopify.com/store/yourstore/themes/[THEME_ID]/
   ```

**Example row:**

```
domain: www.pinkmatcha.co
Brand Name: Pink Matcha
Shopify Store Subdomain: pinkmatchaco
Shopify Link: https://pinkmatcha.co/
Theme Id: 183750230297
Branding Logo: https://cdn.shopify.com/s/files/1/0968/4280/9625/files/Pink_Matcha_Drink.png?v=1766335545
Branding Style: Image generated should contain a girl holding this drink
Image Style: Generate a wide, natural-looking scene with no square canvas, no centered square background, and no boxed framing, and avoid any designs that resemble posters, album covers, book covers, or social media tiles.
```

**Note**: The workflow uses this information for branding consistency and AI-generated featured images. Fill in all fields for best results.

### Step 3: Share Sheet and Get ID

1. Click **"Share"** → Set to **"Anyone with the link can view"**
2. Copy the spreadsheet ID from URL:
   ```
   https://docs.google.com/spreadsheets/d/SPREADSHEET_ID_HERE/edit
   ```
3. Save this ID for n8n configuration

### Step 4: Configure Google Sheets Credentials in n8n

1. In n8n, go to **"Credentials"**
2. Click **"+ Add Credential"**
3. Search for **"Google Sheets OAuth2 API"**
4. Click **"Connect my account"**
5. Follow OAuth flow and grant permissions
6. Name it: `Google Sheets - Blog Config`
7. Click **"Save"**

---

## API Keys & Credentials

### Google Gemini API (Content Generation & Image Generation)

1. Visit [Google AI Studio](https://ai.google.dev/)
2. Click **"Get API Key"**
3. Create new API key or use existing
4. Copy the key (starts with `AIza...`)
5. **Save securely**

**Pricing**: Free tier includes generous quota for both text and image generation. Pay-as-you-go pricing after.

**Used for:**
- Blog content generation (Gemini Pro)
- Featured image generation (Gemini 3 Pro Image Preview)
- AI-powered branding overlay on images

### OpenAI API (Embeddings & Web Search)

1. Visit [OpenAI Platform](https://platform.openai.com/)
2. Go to **"API Keys"**
3. Click **"Create new secret key"**
4. Name it: `n8n-shopify-workflow`
5. Copy the key (starts with `sk-...`)
6. **Save securely**

**Pricing**:
- Embeddings: ~$0.0001 per 1000 tokens (very cheap)
- GPT-4O-SEARCH-PREVIEW: ~$0.01-0.03 per search query
- Expected total: $10-20/month

**Used for:**
- Text embeddings (text-embedding-3-small) for semantic search
- Web search tool (GPT-4O-SEARCH-PREVIEW) for real-time research

### Configure AI Credentials in n8n

**1. Google Gemini (PaLM) API Credential**

1. In n8n **Credentials**, click **"+ Add Credential"**
2. Search for **"Google PaLM API"** or **"Google Gemini"**
3. Enter your Gemini API key from Google AI Studio
4. Name it exactly: `Google Gemini(PaLM) Api account 3`
5. Click **"Save"**

**Important**: The credential type must be **"Google PaLM API"** (googlePalmApi), not the standard Gemini Chat Model credential. This is used for direct HTTP API calls to Gemini.

**2. OpenAI API Credential**

1. Click **"+ Add Credential"**
2. Search for **"OpenAI API"**
3. Enter your OpenAI API key
4. Name it exactly: `OpenAi - shopify`
5. Click **"Save"**

**Used in:**
- Embeddings nodes (text-embedding-3-small)
- Web search tool workflow (GPT-4O-SEARCH-PREVIEW)

**3. MongoDB Credential**

1. Click **"+ Add Credential"**
2. Search for **"MongoDB"**
3. Enter your MongoDB Atlas connection string (from earlier setup)
4. Name it exactly: `MongoDB account`
5. Test connection
6. Click **"Save"**

---

## Shopify API Setup

### Step 1: Create Custom App in Shopify

1. Log into your Shopify admin
2. Go to **"Settings"** → **"Apps and sales channels"**
3. Click **"Develop apps"**
4. Click **"Build Apps in Dev Dashboard"**
5. App name: `n8n Content Generator`
6. Click **"Create app"**

![Shopify Build App](https://articles.emp0.com/wp-content/uploads/2025/12/Shopify-build-app.png)

### Step 2: Configure API Scopes and Redirect URL

1. Click **"Configure Admin API scopes"**
2. Enable the following scopes:

**Content & Blog Management:**
   - `read_content` - Read blog posts
   - `write_content` - Create blog posts

**Product Management:**
   - `read_all_orders` - Read all orders
   - `read_orders` - Read orders
   - `write_orders` - Write orders
   - `read_product_feeds` - Read product feeds
   - `write_product_feeds` - Write product feeds
   - `read_product_listings` - Read product listings
   - `write_product_listings` - Write product listings
   - `read_products` - Read products for product integration
   - `write_products` - Write products

**Theme & Asset Management:**
   - `write_theme_code` - Write theme code
   - `read_themes` - Read theme files
   - `write_themes` - Upload generated images to theme assets

**Unauthenticated Access (for public data):**
   - `unauthenticated_read_product_pickup_locations` - Read pickup locations
   - `unauthenticated_read_product_inventory` - Read product inventory
   - `unauthenticated_read_product_listings` - Read product listings
   - `unauthenticated_read_product_tags` - Read product tags
   - `unauthenticated_read_content` - Read content

3. Click **"Save"**

4. Under **"App setup"**, add your **Allowed redirection URL(s)**:
   ```
   https://n8n.emp0.com/rest/oauth2-credential/callback
   ```

   **Note:** Replace `n8n.emp0.com` with your actual n8n instance URL. If using n8n locally or self-hosted:
   - Local: `http://localhost:5678/rest/oauth2-credential/callback`
   - Self-hosted: `https://your-n8n-domain.com/rest/oauth2-credential/callback`

5. Click **"Save"** again to apply redirect URL changes

![Shopify App Create](https://articles.emp0.com/wp-content/uploads/2025/12/Shopify-app-create.png)

### Step 3: Install App and Get API Credentials

1. Click **"Install app"**
2. Confirm installation
3. Click **"Reveal token once"** and copy:
   - **Admin API access token**
   - **API key** (Client ID)
   - **API secret key** (Client Secret)
4. **Save these securely**

![Shopify App Client ID and Secret](https://articles.emp0.com/wp-content/uploads/2025/12/Shopify-app-client-id-and-secret.png)

### Step 4: Get Your Shopify Store Details

You'll need:
- **Store URL**: `your-store.myshopify.com`
- **Blog Handle/ID**: Found in Shopify Admin → Online Store → Blog Posts → Click blog name
  - The blog handle appears in the URL: `/admin/blogs/[BLOG_ID]`
  - Or you can find the blog handle in the blog settings
- **Category Handles**: Found in the same section, used to categorize your blog posts

![Shopify Category ID](https://articles.emp0.com/wp-content/uploads/2025/12/Shopify-category-id.png)


### Step 5: Configure Shopify Credentials in n8n

![n8n Shopify OAuth2 Setup](https://articles.emp0.com/wp-content/uploads/2025/12/n8n-setup-shopify.png)

1. In n8n **Credentials**, click **"+ Add Credential"**
2. Search for **"Shopify OAuth2 API"**
3. **Copy the OAuth Callback URL** shown at the top of the credential form
4. Go back to your Shopify Admin → **Apps** → **App you created in Step 3**
5. In the app settings, paste the OAuth Callback URL into the **Allowed redirection URL(s)** field
6. Back in n8n, enter your Shopify details:
   - **Shop Subdomain URL**: Your full .myshopify.com URL (e.g., `https://your-store.myshopify.com`)
   - **Client ID**: The API Key from Step 3
   - **Client Secret**: The API Secret Key from Step 3
   - **Access Scopes**: `write_themes,read_themes,write_content,read_content,write_products,read_products`
7. Name it exactly: `Shopify account`
8. Click **"Connect my account"** to complete OAuth2 authentication
9. You'll be redirected to Shopify to authorize the connection
10. After authorization, you'll be redirected back to n8n with the credential saved

---

## Tool Workflow Configuration

The workflow uses 3 sub-workflows as tools for enhanced AI capabilities.

### Tool 1: Gemini Image Generator

**File**: `tool - gemini.json`

![Gemini Tool Workflow](https://articles.emp0.com/wp-content/uploads/2025/12/n8n-workflow-tool-gemini-image-generator.png)

This tool generates AI-powered featured images for blog posts using **Google Gemini 3 Pro Image Preview**.

**What it does:**
- Generates custom branded images based on article topics
- Creates 16:9 aspect ratio images perfect for blog headers
- Supports image editing with branding overlay
- Uploads images directly to Shopify theme assets
- Incorporates your brand logo and style guidelines

**Required Inputs:**
- `prompt` - The image generation prompt based on article topic
- `slug` - Article slug for tracking
- `edit` - Boolean flag for image editing mode
- `shopify_subdomain` - Your .myshopify.com subdomain
- `theme_id` - Your Shopify theme ID
- `branding_logo` - URL to your brand logo/product image (optional if edit is false)
- `branding_style` - Style instructions for AI (e.g., "Image should contain a girl holding this drink") (optional if edit is false)
- `image_style` - Additional generation guidelines (e.g., "Wide, natural scene with no square framing")

**Configuration**:
1. Open the `tool - gemini` workflow
2. Locate the **HTTP Request** nodes calling Gemini API
3. Assign your **Google Gemini (PaLM) API** credential to both HTTP Request nodes
4. Locate the **Upload image1** node
5. Assign your **Shopify OAuth2** credential
6. **Activate the workflow**

**Important**: This workflow uses direct Gemini API calls via HTTP Request (not the Gemini Chat Model node).

### Tool 2: Web Search Tool

**File**: `tool - web search.json`

![Web Search Tool Workflow](https://articles.emp0.com/wp-content/uploads/2025/12/n8n-workflow-tool-web-search.png)

Enables the AI to search the web for current information and trending topics using **OpenAI GPT-4O-SEARCH-PREVIEW**.

**What it does:**
- Conducts targeted web searches based on article topics
- Filters for reliable and recent sources (news, blogs, academic sources)
- Returns top 3-5 summaries with titles, URLs, dates, and content
- Prioritizes statistics, quotes, and specific examples
- Returns structured JSON data

**Required Inputs:**
- `prompt` - The search query

**Configuration**:
1. Open the `tool - web search` workflow
2. Locate the **Message a model** node (OpenAI node)
3. Assign your **OpenAI API** credential
4. The model is pre-configured to use `gpt-4o-search-preview`
5. **Activate the workflow**

**Important**: This uses OpenAI's search-enabled model, NOT Brave Search or SerpAPI.

### Tool 3: Link Validator

**File**: `tool - check link status.json`

![Link Validator Tool Workflow](https://articles.emp0.com/wp-content/uploads/2025/12/n8n-workflow-tool-link-validator.png)

Validates URLs before including them in blog posts to ensure all links are working.

**What it does:**
- Sends HEAD request to check link status
- Returns "VALID_LINK" or "INVALID_LINK" status
- Prevents broken links in published articles
- Handles errors gracefully

**Required Inputs:**
- `link` - The URL to validate

**Configuration**:
1. Open the `tool - check link status` workflow
2. Review the **HTTP Request** node configuration
3. Adjust timeout settings if needed (default: standard timeout)
4. **Activate the workflow**

**Important**: This is a simple HTTP HEAD request with no external API required.

### Connecting Tools to Main Workflow

1. In the main `Shopify Blog Automation` workflow (previously named `Content Generator Shopify Blog`)
2. Locate **AI Agent** nodes throughout the workflow
3. The tools are called using **Execute Workflow** nodes
4. Ensure all 3 tool workflows are **activated** before running the main workflow
5. The main workflow will automatically call these tools when needed

---

## Configure the Main Workflow

### Assign All Credentials

Go through each node and assign credentials:

**Google Sheets Nodes**:
- `Read RSS News Feeds` → Assign `Google Sheets - Blog Config`
- Any other Sheets nodes → Same credential

**MongoDB Nodes**:
- `MongoDB Atlas Vector Store` → Assign `MongoDB Atlas - Content`
- `Insert documents` → Same credential

**AI Nodes**:
- `Google Gemini` nodes → Assign `Google Gemini`
- `OpenAI Embeddings` nodes → Assign `OpenAI - Embeddings`

**Shopify Nodes**:
- `HTTP Request` nodes to Shopify → Assign `Shopify - Blog API`

**Twitter Nodes** (if enabled):
- Twitter API nodes → Assign Twitter credentials

### Step 3: Configure Schedule Trigger

1. Locate the **Schedule Trigger** node
2. Set schedule (default: daily at 9 AM)
3. Adjust timezone if needed

**Recommended Schedules**:
- **Daily**: `0 9 * * *` (9 AM every day)
- **Weekdays only**: `0 9 * * 1-5` (9 AM Monday-Friday)
- **Twice daily**: `0 9,17 * * *` (9 AM and 5 PM)

---

## Testing Your Workflow

### Step 1: Test RSS Feed Reading

1. Disable all nodes except:
   - Schedule Trigger
   - Read RSS News Feeds
   - Set and Normalize Fields
2. Click **"Execute Workflow"**
3. Verify RSS feeds are being read correctly
4. Check output data structure

### Step 2: Test Vector Search

1. Enable MongoDB Vector Store nodes
2. Run workflow with sample article
3. Check MongoDB Atlas for stored embeddings
4. Verify vector search is working

### Step 3: Test Content Generation

1. Enable AI Agent and Gemini nodes
2. Run with a single RSS article
3. Review generated content quality
4. Adjust prompts if needed

### Step 4: Test Shopify Publishing

**Important**: Set to "draft" mode for testing!

1. Enable Shopify publishing nodes
2. Run full workflow
3. Check Shopify Admin for draft blog post
4. Verify formatting, category, and metadata

### Step 5: Full End-to-End Test

1. Enable all nodes
2. Run complete workflow
3. Verify:
   - RSS feeds read ✓
   - Content generated ✓
   - Duplicates prevented ✓
   - Published to Shopify ✓
   - Posted to Twitter (if enabled) ✓
4. Set the Workflow to "Active"

---

## Cost Breakdown

### Monthly Cost Estimate

Based on generating 10 articles per day:

| Service | Cost | Notes |
|---------|------|-------|
| **Google Gemini API** | $30 | Content generation  |
| **OpenAI Embeddings** | $10 | Vector embeddings for duplicate prevention |
| **MongoDB Atlas** | $0 | Free M0 tier (512MB) |
| **n8n Cloud** | $20 | Or $0 if self-hosted |
| **Shopify** | $25 | Basic Plan for API access |
| **Twitter API** | $0 | Free tier for 500 posts / month |
| **TOTAL** | **$85/month** | **~$0.29 per article** |

### Cost Optimization Tips

1. **Self-host n8n**: Save $10/month → [Railway](https://railway.com/deploy/Hx5aTY?referralCode=jay) or [Hostinger](https://www.hostinger.com/vps/n8n-hosting?REFERRALCODE=jayemp0)
2. **Use MongoDB Free Tier**: $0 for up to 512MB (sufficient for thousands of articles)
3. **Start with 1 article/day**: Reduce API costs while testing
4. **Twitter integration**: Dont upgrade if your usage is below 500 posts per month
5. **Use Google Gemini Flash**: Cheaper model for basic content


**Compare to Traditional Content Creation**:
- Freelance writers: $20-30 per article
- 90 articles/month: **$1,800-2,700** vs **$55 with automation**

---

## Advanced Features

### Configure Multiple Blogs

![Multiple Blogs Setup](https://articles.emp0.com/wp-content/uploads/2025/12/Pink-Matcha-Blog-List.png)

You can run multiple blogs within the same Shopify store, each with its own content strategy.

**Step 1: Create Additional Blogs in Shopify**

1. In Shopify Admin, go to **Online Store** → **Blog Posts**
2. Click **"Manage blogs"**
3. Click **"Add blog"**
4. Create blogs for different purposes:
   - `Men's Fashion Blog`
   - `Women's Fashion Blog`
   - `Kids Fashion Blog`
   - etc.
5. Note each blog's **Blog ID** from the URL

**Step 2: Duplicate the Workflow**

1. In n8n, open your main workflow
2. Click the **three dots** menu → **"Duplicate"**
3. Rename: `Shopify Blog - Men's Fashion`
4. Repeat for each blog you want to automate

**Step 3: Configure Each Workflow**

For each duplicated workflow:

1. Update **Build Config** with the specific blog ID
2. Create separate Google Sheets for RSS feeds (or use different sheet tabs)
3. Configure different RSS feeds relevant to that blog's niche
4. Set unique categories for each blog
5. Adjust AI prompts to match the target audience

**Step 4: Schedule Independently**

Each workflow can run on its own schedule:
- Men's blog: Daily at 9 AM
- Women's blog: Daily at 11 AM
- Kids blog: 3x per week

**Benefits:**
- Target different audiences with tailored content
- Separate product lines get dedicated blogs
- Test different content strategies simultaneously
- Scale content production exponentially

### Configure Product Integration

![Product Integration](https://articles.emp0.com/wp-content/uploads/2025/12/Pink-Matcha-Digital-Products.png)

Automatically embed your Shopify products within blog articles.

**Step 1: Fetch Your Product Catalog**

1. Add **HTTP Request** node to workflow
2. Configure Shopify API call:
   ```
   GET https://YOUR-STORE.myshopify.com/admin/api/2024-01/products.json
   ```
3. Use your Shopify Admin API token
4. Store products in workflow variable or MongoDB

**Step 2: Configure Product Matching**

Update the AI Agent prompt to include product references:

```javascript
{
  "systemPrompt": `You are a content writer for [YOUR STORE].

  When writing articles:
  1. Analyze the article topic
  2. Reference relevant products from our catalog
  3. Include product links using this format: [Product Name](https://store.com/products/handle)
  4. Add product cards in HTML:
     <div class="product-card">
       <img src="product-image-url" alt="Product Name">
       <h3>Product Name</h3>
       <p>Product description</p>
       <a href="product-url">Shop Now</a>
     </div>

  Available Products:
  {{ $json.products }}

  Naturally weave 2-3 product mentions into every article.`
}
```

**Step 3: Dynamic Product Selection**

![Multiple Products](https://articles.emp0.com/wp-content/uploads/2025/12/Pink-Matcha-Product-List.png)

Use AI to intelligently match products:

```javascript
// In Code node before AI Agent
const article_topic = $input.item.json.category;
const all_products = $('Get Products').all();

// Filter products by category/tags
const relevant_products = all_products.filter(product => {
  return product.tags.includes(article_topic) ||
         product.product_type === article_topic;
});

// Pass to AI Agent
return {
  json: {
    ...$input.item.json,
    relevant_products: relevant_products.slice(0, 5) // Top 5 products
  }
};
```

**Step 4: Format Product Embeds**

The AI will automatically:
- Create product cards with images
- Add contextual product mentions
- Include "Shop Now" CTAs
- Link to product pages

**Example Output:**

```html
<h2>Top 5 Yoga Mats for Beginners</h2>
<p>Starting your yoga journey? Here are the best mats...</p>

<div class="product-card">
  <img src="yoga-mat.jpg" alt="Premium Yoga Mat">
  <h3>Premium Eco-Friendly Yoga Mat</h3>
  <p>Non-slip, extra cushioning, sustainable materials</p>
  <a href="/products/premium-yoga-mat">Shop Now - $49.99</a>
</div>

<p>We especially love the Premium Eco-Friendly Yoga Mat for its...</p>
```

**Benefits:**
- Turn blog readers into buyers
- Increase average order value
- Cross-sell related products
- Track which products drive traffic

---

## Customization Guide

### Customize Content Style

Edit the AI prompts in the Gemini agent to match your brand:

```
Write in a [conversational/professional/casual] tone.
Target audience: [demographics and interests]
Brand voice: [adjectives describing your brand]
Key themes: [topics to emphasize]
Avoid: [topics or language to avoid]
```

### Customize Article Structure

Modify the structured output parser to change article format:

```json
{
  "title": "string",
  "metaDescription": "string (max 160 chars)",
  "category": "string",
  "content": {
    "introduction": "string",
    "sections": [
      {
        "heading": "string",
        "content": "string"
      }
    ],
    "conclusion": "string"
  },
  "tags": ["array of strings"],
  "seoKeywords": ["array of strings"]
}
```

### Add Featured Images

**Example Blog Post with Featured Image:**

![Blog Post Detail](https://articles.emp0.com/wp-content/uploads/2025/12/Pink-Matcha-Blog-Post-Screenshot-Vertical-2.png)

### Add Internal Linking

1. Query Shopify for existing blog posts
2. Use AI to identify relevant internal links
3. Insert links into generated content
4. Improve SEO and user engagement

### Multi-Language Support

1. Duplicate workflow for each language
2. Configure language-specific RSS feeds
3. Update AI prompts with target language
4. Create separate Shopify blogs per language

### Social Media Distribution

![Twitter Auto-Posting](https://articles.emp0.com/wp-content/uploads/2025/12/twitter.png)

Automatically share your blog posts on social media platforms.

**Twitter Integration:**

1. Add **Twitter** node after Shopify publishing
2. Configure Twitter API credentials
3. Extract key points from article using AI
4. Format tweet with:
   - Engaging hook (first 100 characters)
   - 2-3 key takeaways
   - Link to blog post
   - Relevant hashtags

---

## Troubleshooting

### Issue: RSS Feeds Not Loading

**Symptoms**: Workflow runs but no articles are fetched

**Solutions**:

1. **Check RSS feed URLs**:
   - Verify URLs are valid and accessible
   - Test in browser or RSS reader first
   - Some sites block automated access

2. **Google Sheets permissions**:
   - Ensure sheet is shared ("Anyone with link can view")
   - Verify n8n can access the sheet
   - Check OAuth2 token hasn't expired

3. **RSS Feed node configuration**:
   - Verify column names match exactly
   - Check "Active" column is TRUE
   - Ensure no extra spaces in URLs

### Issue: Duplicate Content Being Generated

**Symptoms**: Same topics appear multiple times

**Solutions**:

1. **Check vector search threshold**:
   - Lower similarity threshold (e.g., 0.7 → 0.6)
   - Increase required matches for duplicate detection

2. **Verify MongoDB vector index**:
   - Check index is created and active
   - Ensure embeddings are being stored
   - Review similarity search query

3. **Clear old embeddings**:
   - Delete old test articles from MongoDB
   - Re-run with fresh data

### Issue: Poor Content Quality

**Symptoms**: Generated articles are generic or off-topic

**Solutions**:

1. **Improve AI prompts**:
   - Add more specific instructions
   - Include example articles
   - Define tone, style, and structure clearly

2. **Increase context**:
   - Feed more RSS content to AI
   - Include category descriptions
   - Add brand voice guidelines

3. **Use better AI model**:
   - Switch from Gemini Flash to Gemini Pro
   - Experiment with different temperature settings
   - Increase max token output

### Issue: Shopify Publishing Fails

**Symptoms**: Articles generate but don't appear in Shopify

**Solutions**:

1. **Verify Shopify API credentials**:
   - Check access token is valid
   - Ensure correct scopes are enabled
   - Test API connection manually

2. **Check blog ID**:
   - Verify blog ID is correct
   - Test with Shopify API directly
   - Ensure blog exists and is accessible

3. **Review API request format**:
   - Check JSON payload structure
   - Verify required fields are present
   - Review Shopify API documentation

4. **Check error messages**:
   - Review workflow execution logs
   - Look for specific API error codes
   - Fix validation errors in content

### Issue: High API Costs

**Symptoms**: Bills higher than expected

**Solutions**:

1. **Monitor token usage**:
   - Check Google Gemini dashboard
   - Review OpenAI usage stats
   - Track tokens per article

2. **Optimize content length**:
   - Reduce max word count
   - Use more efficient prompts
   - Batch API requests when possible

3. **Reduce generation frequency**:
   - Change from daily to 3x/week
   - Lower articles per run
   - Only generate during peak traffic times

4. **Use cheaper models**:
   - Gemini Flash instead of Pro
   - OpenAI text-embedding-3-small instead of ada-002

### Issue: MongoDB Connection Errors

**Symptoms**: Vector store nodes fail

**Solutions**:

1. **Check connection string**:
   - Verify password is correct
   - Ensure database name is included
   - Check for URL encoding issues

2. **Network access**:
   - Verify IP whitelist in MongoDB Atlas
   - Add n8n instance IP if changed
   - Use "Allow from anywhere" for testing

3. **Database permissions**:
   - Ensure user has read/write access
   - Check database name matches
   - Verify collection exists

---

## Advanced Configuration

### Webhook Triggers

Add webhook endpoint to trigger content generation on-demand:

1. Add **Webhook** node to workflow
2. Set path: `/generate-content`
3. Configure POST method
4. Pass category or topic as parameter
5. Generate content immediately

### A/B Testing Headlines

1. Generate multiple title variations
2. Store in MongoDB with metrics
3. Track click-through rates in Shopify
4. Use best-performing headlines

### Content Calendar Integration

1. Connect to Google Calendar or Notion
2. Read planned content topics
3. Generate articles based on calendar
4. Schedule publication dates

### SEO Optimization

1. Add keyword research tool (Ahrefs, SEMrush API)
2. Identify trending keywords in niche
3. Include in content generation prompts
4. Track rankings and adjust

---

## Support Resources

- **n8n Documentation**: [docs.n8n.io](https://docs.n8n.io)
- **MongoDB Atlas Docs**: [docs.mongodb.com](https://docs.mongodb.com)
- **Shopify API Docs**: [shopify.dev/docs/api](https://shopify.dev/docs/api)
- **Google Gemini Docs**: [ai.google.dev/docs](https://ai.google.dev/docs)
- **emp0 Community**: [Skool Community](https://www.skool.com/aia-ai-automation-2762)
- **Purchase Workflow**: [Gumroad](https://0emp0.gumroad.com/l/shopify-blog-automation)
- **Email Support**: [jay@emp0.com](mailto:jay@emp0.com)

---

## Changelog

**Version 1.0** (December 2024)
- Initial release
- Google Gemini integration
- MongoDB Atlas vector store
- Shopify blog publishing
- RSS feed aggregation
- Twitter auto-posting
- Multi-category support
- Duplicate prevention

---

**Need help?** Join our [Skool community](https://www.skool.com/aia-ai-automation-2762) for live support and connect with other workflow users.

Happy content generating! 🚀
