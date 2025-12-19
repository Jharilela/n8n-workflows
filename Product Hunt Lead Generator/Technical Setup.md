# Product Hunt Scraper - Technical Setup Guide

This comprehensive guide walks you through setting up the Product Hunt Scraper workflow from scratch. Follow each section carefully to ensure everything works correctly.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Workflow Import & Configuration](#workflow-import--configuration)
3. [n8n Setup](#n8n-setup)
4. [Creating n8n Data Tables](#creating-n8n-data-tables)
5. [Apify Configuration](#apify-configuration)
6. [Google API Setup](#google-api-setup)
7. [Testing Your Workflow](#testing-your-workflow)
8. [Cost Breakdown](#cost-breakdown)
9. [Optional Enhancements](#optional-enhancements)
10. [Troubleshooting](#troubleshooting)
11. [Important Warnings](#important-warnings)

---

## Prerequisites

Before starting, ensure you have:

- [ ] Active n8n instance (cloud or self-hosted)
- [ ] Apify account (sign up at [apify.com](https://www.apify.com/?fpr=99h7ds) - use referral code: **99h7ds**)
- [ ] Google account for Sheets, Drive and Gmail API integration
- [ ] Basic understanding of n8n workflows

**Estimated Setup Time**: 45-60 minutes

---

## Workflow Import & Configuration

### Step 1: Get the Workflow

First, you'll need to obtain the workflow JSON file:

**Option 1: Purchase the workflow**
- Visit [0emp0.gumroad.com/l/product-hunt-lead-generator](https://0emp0.gumroad.com/l/product-hunt-lead-generator)
- Complete your purchase
- Download the `Product Hunt Scraper.json` file

**Option 2: Join the community**
- Join our [Skool community](https://www.skool.com/aia-ai-automation-2762)
- Access the workflow files in the Classroom section -> Expert Automation -> Product Hunt Scraper

### Step 2: Import the Workflow

1. Once you have the `Product Hunt Scraper.json` file
2. In n8n, click **"Workflows"** in the sidebar
3. Click **"+ Add Workflow"**
4. Click the **three dots** menu (⋮) → **"Import from File"**
5. Select the downloaded JSON file
6. The workflow will appear in your canvas

You'll configure specific nodes later after setting up the required services.

---

## n8n Setup

### Option 1: n8n Cloud (Recommended for Beginners)

1. **Sign up for n8n Cloud**
   - Visit [n8n.cloud](https://n8n.partnerlinks.io/emp0)
   - Choose a plan (Starter plan at $20/month is sufficient)
   - Complete account registration

2. **Access your n8n instance**
   - Log into your cloud dashboard
   - Click "Open n8n" to access the workflow editor

### Option 2: Self-Hosted n8n (Cost-Effective)

For self-hosting, we recommend two excellent options:

#### Railway (Easiest Self-Hosted Option)

1. **Deploy n8n to Railway**
   - Click this deployment link: [Deploy n8n on Railway](https://railway.com/deploy/Hx5aTY?referralCode=jay)
   - Sign up/login to Railway
   - Click "Deploy Now"
   - Railway will automatically configure n8n with a public HTTPS URL
   - Cost: Pay-as-you-go, typically $5-15/month

2. **Access your n8n instance**
   - Railway provides a public URL automatically (e.g., `your-app.railway.app`)
   - No SSL or domain configuration needed
   - Click the URL to access n8n

#### Hostinger VPS (Best for Production)

1. **Get Hostinger VPS with n8n**
   - Visit [Hostinger n8n VPS Hosting](https://www.hostinger.com/vps/n8n-hosting?REFERRALCODE=jayemp0)
   - Choose a VPS plan (KVM 1 or higher recommended)
   - Select n8n as your pre-installed application
   - Complete purchase

2. **Access your n8n instance**
   - Hostinger provides a configured n8n installation with domain and SSL
   - Log in with credentials provided in your dashboard
   - Cost: Starting at ~$5-10/month

**Important**: For production use, webhooks require a publicly accessible HTTPS URL. Both Railway and Hostinger provide this automatically.

---

## Creating n8n Data Tables

![n8n Data Tables](https://articles.emp0.com/wp-content/uploads/2025/12/product-hunt-scraper-n8n-datatables.png)

The workflow uses n8n Data Tables to store configuration and runtime data.

### Step 1: Create a New Data Table

1. In your n8n instance, click **"Data Tables"** in the left sidebar
2. Click **"+ New Data Table"**
3. Name it: `product hunt` (or any name you prefer)
4. Set columns:
   - **Column 1**: `key` (type: String)
   - **Column 2**: `value` (type: String)

### Step 2: Note Your Data Table ID

1. After creating the table, open it
2. Copy the **Data Table ID** from the URL
   - Example: `https://app.n8n.cloud/projects/LAHZc7FnP0ywn2o3/datatables/3mOJkgv9zEcXiZP3`
   - The ID is: `3mOJkgv9zEcXiZP3`
3. **Save this ID** - you'll need it when configuring the workflow

### What This Table Stores

The Data Table keeps track of:
- `IS_WEBHOOK_CREATED`: Whether the Apify webhook has been registered (prevents duplicates)
- `RUNS`: JSON object tracking Apify actor run statuses
- `SHEET`: Google Sheets ID for storing leads

---

## Apify Configuration

Apify powers the scraping functionality. You'll need two actors:

### Step 1: Create an Apify Account

1. Visit [apify.com](https://www.apify.com/?fpr=99h7ds)
2. Sign up for a free account and use referral code: **99h7ds**
3. You get **$5 free credits** monthly (covers ~10 days of usage)
4. For production, upgrade to a paid plan

### Step 2: Subscribe to Required Actors

#### Actor 1: Advanced Product Hunt Scraper

1. Visit: [Advanced Product Hunt Scraper](https://www.apify.com/danpoletaev/product-hunt-scraper?fpr=99h7ds)
2. Click **"Try for free"** or **"Subscribe"**
3. Note the **Actor ID**: `danpoletaev/product-hunt-scraper`
4. This actor costs **$15/month rental** + compute costs

**What it does**: Scrapes Product Hunt for top products, including maker info, descriptions, upvotes, and social links.

#### Actor 2: Contact Info Scraper

1. Visit: [Contact Info Scraper](https://www.apify.com/vdrmota/contact-info-scraper?fpr=99h7ds)
2. Click **"Try for free"** or **"Subscribe"**
3. Note the **Actor ID**: `vdrmota/contact-info-scraper`
4. This actor is **pay-per-use** (no monthly rental)

**What it does**: Extracts emails, phone numbers, and social media profiles from product websites.

### Step 3: Get Your Apify API Token

1. In Apify, click your profile icon → **Settings**
2. Navigate to **Integrations**
3. Find **API Tokens** section
4. Click **"Create new token"**
5. Name it: `n8n-product-hunt-scraper`
6. Copy the token (starts with `apify_api_...`)
7. **Store this securely** - you'll add it to the workflow configuration

### Step 4: Configure Apify API Token in n8n Workflow

**Important**: After importing the workflow to your n8n instance, you need to configure the Apify API token as a Bearer token in the HTTP Request nodes.

1. In the workflow, locate all **HTTP Request** nodes that connect to Apify:
   - `Create External Webhook2`
   - `Run Actor "Daily Top Products"2`
   - `Run Actor "Weekly Top Products"2`
   - `Fetch Dataset Items2`

2. For each HTTP Request node:
   - Double-click to open the node
   - Scroll to **Authentication** section
   - Select **"Predefined Credential Type"** → **"Header Auth"**
   - Click **"+ Add Credential"** for Header Auth
   - Configure the credential:
     - **Name**: `Apify Bearer Token`
     - **Header Name**: `Authorization`
     - **Header Value**: `Bearer YOUR_APIFY_API_TOKEN` (replace with your actual token)
   - Click **"Save"**

3. Select this credential in all Apify HTTP Request nodes

**Why Bearer Token?**: This method provides better security and follows Apify's recommended authentication approach for API calls.

---

## Google API Setup

**Good news!** The workflow automatically creates a Google Sheet for you and stores its ID in the Data Table. You don't need to manually create a spreadsheet.

### Step 1: Enable Required Google APIs

Before configuring OAuth credentials in n8n, you **must enable** the following Google APIs in your Google Cloud Console:

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project or select an existing one
3. Navigate to **"APIs & Services"** → **"Library"**
4. Enable the following APIs:
   - **Google Sheets API** - For creating and updating spreadsheets
   - **Google Drive API** - For file access and permissions
   - **Gmail API** - For sending outreach emails

5. For each API:
   - Search for the API name (e.g., "Google Sheets API")
   - Click on it
   - Click **"Enable"**
   - Wait for activation (usually instant)

**Important**: If these APIs are not enabled, the OAuth flow will fail or you'll get permission errors when the workflow runs.

### Step 2: Configure Google Sheets Authentication in n8n

#### OAuth2 (Recommended)

1. In n8n, go to **Credentials**
2. Click **"+ Add Credential"**
3. Search for **"Google Sheets"** and select **"Google Sheets OAuth2 API"**
4. Click **"Connect my account"**
5. Follow the Google OAuth flow
6. **Grant all requested permissions** for Sheets and Drive access
7. Name the credential: `Google Sheets`
8. Click **"Save"**

**Note**: When the workflow runs, it will automatically create a Google Sheet and store the ID in the Data Table. No manual sheet creation needed!

---

## Configure the Build Config Node

The **Build Config2** node is the central configuration hub.

1. Double-click the **"Build Config2"** node
2. Locate the JavaScript code
3. Update the following values:

```javascript
let config = {
  // ... existing config ...

  dataTableId: "YOUR_DATA_TABLE_ID",  // Replace with your Data Table ID (e.g., "3mOJkgv9zEcXiZP3")

  // ... more config ...

  externalScraper: {
    baseUrl: "https://api.apify.com/v2",
    payload: {
      webhook: {
        requestUrl: "https://YOUR_N8N_DOMAIN/webhook/product-hunt-scraper-workflow",
        // ↑ Replace with your n8n webhook URL (note the updated endpoint name)
      }
    }
  },

  // Note: apiBaseUrl has been removed in the latest version

  email: {
    fromEmail: "your-email@yourdomain.com",  // Your Gmail address for sending
    visitEmail: "https://yourwebsite.com",
    privacyPolicyUrl: "https://yourwebsite.com/privacy",
    termsOfServiceUrl: "https://yourwebsite.com/terms",
    ourWebsite: "Your Company Name"
  },

  day: {
    filter: {
      topNProducts: 50  // Daily scrape: 50 products (3 is just for testing)
    }
  },

  week: {
    filter: {
      topNProducts: 100  // Weekly scrape: 100 products
    }
  }
};
```

**Important Fields to Update:**

- **`dataTableId`**: Your Data Table ID from earlier (the workflow uses a new data table structure)
- **`requestUrl`**: Your n8n webhook URL - note the endpoint is `/product-hunt-scraper-workflow`
- **`fromEmail`**: Your Gmail address for sending (must match your Gmail API credentials)
- **`day.filter.topNProducts`**: Daily scraping set to **50 products** (the default of 3 is just for testing to avoid costs)
- **`week.filter.topNProducts`**: Weekly scraping set to **100 products**
- **Email branding fields**: Update URLs and company name
- **Note**: The `apiBaseUrl` field has been removed in the latest version

### Step 4: Find Your Webhook URL

1. In the workflow, locate the **"Receive Apify Webhook2"** node
2. Double-click to open it
3. The webhook path is: `/product-hunt-scraper-workflow` (updated from previous versions)
4. Your full webhook URL format:
   - **n8n Cloud**: `https://your-instance.app.n8n.cloud/webhook/product-hunt-scraper-workflow`
   - **Self-hosted**: `https://your-domain.com/webhook/product-hunt-scraper-workflow`

5. **Activate the workflow** (toggle in top-right corner) to enable the webhook

### Step 5: Update Webhook URL in Config

1. Copy your full webhook URL
2. Return to **"Build Config2"** node
3. Update the `requestUrl` field with your webhook URL
4. Click **"Save"**

### Step 6: Configure Credentials Throughout Workflow

The workflow has multiple nodes requiring credentials:

1. **Apify HTTP Request nodes** (4 nodes) - use Bearer token authentication as configured earlier:
   - `Create External Webhook2`
   - `Run Actor "Daily Top Products"2`
   - `Run Actor "Weekly Top Products"2`
   - `Fetch Dataset Items2`

   Ensure each has the Header Auth credential with `Authorization: Bearer YOUR_APIFY_TOKEN`

2. **Google Sheets nodes** (2 nodes):
   - `Import To Google Sheets2`
   - (Any other Sheets node)

   For each: Open node → Select your **Google Sheets OAuth2** credential

3. **Gmail node**:
   - `Send a message` (previously named "Send email1")
   - Configure Gmail OAuth2 credentials (already set up in Google API Setup section)

---

## Configure Gmail Node for Outreach

**The workflow uses Gmail API by default** for better deliverability, tracking, and reliability compared to SMTP.

**Note**: You must have already enabled Gmail API in the [Google API Setup](#google-api-setup) section above.

### Step 1: Configure Gmail Credentials in n8n

1. In n8n, go to **Credentials**
2. Click **"+ Add Credential"**
3. Search for **"Gmail OAuth2 API"**
4. Enter your OAuth2 Client ID and Secret
5. Click **"Connect my account"**
6. Complete the OAuth flow
7. Name it: `Gmail`

### Step 3: Configure the Gmail Node

The workflow already includes a Gmail node named **"Send a message"**:

1. Open the **"Send a message"** node in the workflow
2. Select your **Gmail OAuth2** credential
3. The email template is pre-configured with:
   - Dynamic recipient (primary email from scraped data)
   - CC addresses (secondary emails)
   - Professional HTML template
   - Personalized product information

4. Update the email template if desired to match your brand

![Email Template Example](https://articles.emp0.com/wp-content/uploads/2025/12/product-hunt-scraper-email-blast.png)

**What the Email Template Includes:**
- Personalized greeting with product name
- Reference to their Product Hunt launch
- Your value proposition
- Clear call-to-action
- Professional HTML formatting
- Unsubscribe link (required for compliance)
- Your contact information and branding

**Gmail Sending Limits:**
- **500 emails/day** for standard Gmail accounts
- **2,000 emails/day** for Google Workspace accounts
- Better deliverability than SMTP
- Built-in tracking and engagement metrics
- **Free** - no additional costs

### Step 4: Test Email Sending

1. Ensure your Gmail credential is properly authenticated
2. Click **"Test step"** on the **"Send a message"** node
3. Check your sent folder in Gmail
4. Verify the email formatting and personalization

---

## Testing Your Workflow

### Step 1: Manual Test Run

1. Ensure the workflow is **activated** (toggle on)
2. Click the **"Schedule Trigger1"** node
3. Click **"Execute Node"** to trigger manually
4. Watch the workflow execution
5. Check for errors in each node

### Step 2: Verify Data Flow

1. **Check Apify**:
   - Go to [console.apify.com/actors/runs](https://console.apify.com/actors/runs)
   - Verify actors are running
   - Wait for completion (~5-10 minutes)
![Apify Runs](https://articles.emp0.com/wp-content/uploads/2025/12/product-hunt-scraper-apify-runs.png)

2. **Check Webhook**:
   - In n8n, view the **"Receive Apify Webhook2"** executions
   - Should trigger automatically when Apify runs complete

3. **Check Google Sheets**:
   - Open your spreadsheet
   - Verify rows are being populated
   - Check data formatting

4. **Check Emails** (if enabled):
   - Verify test emails were sent
   - Check spam folder
   - Review email content

### Step 3: Monitor First Daily Run

The workflow runs automatically at **9:00 AM** (configurable).

1. Wait for the scheduled time or adjust the schedule:
   - Open **"Schedule Trigger1"** node
   - Change `triggerAtHour: 9` to your preferred time

2. Monitor the execution:
   - Check n8n **"Executions"** tab
   - Look for successful completion
   - Review any errors

---

## Cost Breakdown

![Apify Cost Breakdown](https://articles.emp0.com/wp-content/uploads/2025/12/product-hunt-scraper-apify-cost.png)

### Monthly Cost Estimate

Based on daily usage (30 days/month, 50 products/day):

| Service | Cost | Notes |
|---------|------|-------|
| **Apify - Actor Rental** | $15.00 | Product Hunt Scraper actor |
| **Apify - Compute Units** | $25-30 | Daily (50 products) + weekly runs (~75-90 compute units) |
| **Apify - Contact Scraper** | $25-30 | ~1,700 page scrapes/month (50 products/day × 30 days + weekly) |
| **n8n Cloud** | $20.00 | Or $0 if self-hosted on [Railway](https://railway.com/deploy/Hx5aTY?referralCode=jay) or [Hostinger](https://www.hostinger.com/vps/n8n-hosting?REFERRALCODE=jayemp0) |
| **Gmail API** | $0.00 | Free (500 emails/day for personal, 2,000/day for Workspace) |
| **TOTAL** | **$65-95/month** | **~$0.04-0.06 per lead** (1,500+ leads/month) |

**Cost Breakdown Details:**
- **Daily scraping**: 50 products × 30 days = 1,500 products/month
- **Weekly scraping**: 100 products × 4 weeks = 400 products/month
- **Total leads**: ~1,900 products/month
- **Cost per lead**: $65-95 ÷ 1,900 = **$0.03-0.05 per lead**

**Sign up for Apify**: Use our [affiliate link](https://www.apify.com/?fpr=99h7ds) or referral code **99h7ds** to support this project.

### Cost Optimization Tips

1. **Self-host n8n**: Save $20/month vs n8n Cloud → [Deploy on Railway](https://railway.com/deploy/Hx5aTY?referralCode=jay) or [Hostinger VPS](https://www.hostinger.com/vps/n8n-hosting?REFERRALCODE=jayemp0)
2. **Reduce scraping frequency**: Run 3x/week instead of daily → save ~50% Apify costs
3. **Lower "topNProducts"**: Scrape top 25 instead of 50 → save ~40% scraping and contact enrichment costs
4. **Skip weekly runs**: Only do daily scraping → save ~20% Apify costs
5. **Test with small batches first**: Start with 3 products to test → increase to 50 after verification
6. **Use Gmail API**: Free for up to 500 emails/day (personal) or 2,000/day (Workspace)

### Scaling Costs

- **Double the products** (100/day): Add ~$40-50/month (total: $105-145/month)
- **Triple the products** (150/day): Add ~$80-100/month (total: $145-195/month)
- **Add email verification** (ZeroBounce): +$16/1,000 verifications
- **Add more data points**: +$10-20/month Apify compute

---

## Optional Enhancements

### 1. Add Email Verification with ZeroBounce

Verify email addresses before sending to improve deliverability.

**Setup:**

1. Sign up at [zerobounce.net](https://www.zerobounce.net)
2. Get API key
3. Add ZeroBounce node after contact scraping:
   - Add **HTTP Request** node
   - Endpoint: `https://api.zerobounce.net/v2/validate`
   - Method: `GET`
   - Query params: `?email={{ $json.primary_email }}&api_key=YOUR_API_KEY`
4. Filter out invalid emails with **IF** node

**Cost**: $16/1,000 verifications (first 100 free)

### 2. Filter by Product Category

Only scrape specific categories (e.g., AI, Developer Tools):

1. Open **"Build Config2"** node
2. Add to the daily payload:

```javascript
day: {
  "filter": {
    "topNProducts": 50,
    "categories": ["AI", "Developer Tools", "Productivity"]  // Add this
  }
}
```

3. Available categories:
   - AI, Developer Tools, Productivity, SaaS, Marketing
   - Design Tools, Analytics, Mobile Apps, E-commerce
   - Social Media, Crypto, Health & Fitness, Education

### 3. Add Slack/Discord Notifications

Get notified when new leads are found:

**Discord:**

1. Add **Discord** node after Google Sheets import
2. Create Discord webhook in your server
3. Send message with lead count:

```javascript
{
  "content": "🎯 Found {{ $('Import To Google Sheets2').itemMatching(0).json.rowsAdded }} new leads today!"
}
```

**Slack:**

1. Add **Slack** node
2. Connect your Slack workspace
3. Post to channel with lead summary

### 4. CRM Integration

Push leads directly to your CRM:

- **HubSpot**: Use HubSpot node to create contacts
- **Salesforce**: Use Salesforce node to create leads
- **Pipedrive**: Use Pipedrive node to create deals
- **Airtable**: Use Airtable node as a visual CRM

### 5. Adjust Scraping Schedule

Change when the workflow runs:

1. Open **"Schedule Trigger1"** node
2. Modify schedule:

```javascript
// Current: Daily at 9 AM
{
  "triggerAtHour": 9
}

// Multiple times per day:
{
  "interval": [
    {"triggerAtHour": 9},
    {"triggerAtHour": 18}
  ]
}

// Specific days only:
{
  "rule": {
    "interval": [{"triggerAtHour": 9}],
    "daysOfWeek": [1, 3, 5]  // Monday, Wednesday, Friday
  }
}
```

---

## Troubleshooting

### Issue: Webhook Not Triggering

**Symptoms**: Apify runs complete, but n8n workflow doesn't continue

**Solutions:**

1. **Check webhook URL**:
   - Verify the URL in **Build Config2** matches your actual webhook
   - Must be publicly accessible (not localhost)
   - Must use HTTPS

2. **Activate the workflow**:
   - Toggle must be ON for webhook to listen

3. **Check Apify webhook registration**:
   - Go to [console.apify.com/actors/integrations](https://console.apify.com/actors/integrations)
   - Verify webhook is listed
   - Check recent deliveries for errors

4. **Test webhook manually**:
   - Copy webhook URL
   - Use Postman or curl to send test payload:

   ```bash
   curl -X POST https://your-n8n.com/webhook/apify-product-hunt \
     -H "Content-Type: application/json" \
     -d '{"eventType":"ACTOR.RUN.SUCCEEDED","resource":{"id":"test"}}'
   ```

5. **Check n8n logs**:
   - Self-hosted: `docker logs n8n`
   - Cloud: Contact n8n support

### Issue: Google Sheets Not Updating

**Symptoms**: Workflow completes but no data in spreadsheet

**Solutions:**

1. **Check Google Sheets credential**:
   - Re-authenticate OAuth2 connection
   - Verify permission scopes include "write"

2. **Verify spreadsheet ID**:
   - Check Data Table has correct `SHEET` value
   - Ensure spreadsheet exists and is accessible

3. **Check column mapping**:
   - Open **"Import To Google Sheets2"** node
   - Verify column names match exactly
   - Check for typos

4. **Test with manual data**:
   - Add **Set** node before Sheets node
   - Hardcode test data
   - Verify Sheets node can write

### Issue: Apify Actors Failing

**Symptoms**: Actors timeout or return errors

**Solutions:**

1. **Check Apify credits**:
   - Visit [console.apify.com/billing](https://console.apify.com/billing/)
   - Ensure you have sufficient credits

2. **Review actor runs**:
   - Go to [console.apify.com/actors/runs](https://console.apify.com/actors/runs)
   - Click failed run
   - Check logs for specific errors

3. **Reduce scraping load**:
   - Lower `topNProducts` from 50 to 20
   - Reduce `maxDepth` in contact scraper
   - Lower `maxRequests` values

4. **Check actor configuration**:
   - Verify payload in **Build Config2**
   - Ensure all required fields are present
   - Test actors manually in Apify console

### Issue: Emails Not Sending

**Symptoms**: Gmail node fails or emails don't arrive

**Solutions:**

1. **Check Gmail OAuth2 credentials**:
   - Re-authenticate if expired
   - Verify Gmail API is enabled in Google Cloud Console
   - Ensure proper scopes are granted (gmail.send)

2. **Check Gmail sending limits**:
   - Standard Gmail: 500 emails/day maximum
   - Google Workspace: 2,000 emails/day maximum
   - Reduce batch size and increase delay if hitting limits

3. **Check spam filters**:
   - Emails may be in recipient's spam folder
   - Warm up your account gradually (start with 10-20 emails/day)
   - Use professional email templates
   - Include unsubscribe link

4. **Check email content**:
   - Avoid spam trigger words ("free", "guarantee", etc.)
   - Personalize each message
   - Include proper sender information

5. **Monitor Gmail account health**:
   - Check for security alerts in Gmail
   - Review sent folder for bounces
   - Track delivery rates

### Issue: Duplicate Leads

**Symptoms**: Same products appear multiple times in spreadsheet

**Solutions:**

1. **Check deduplication node**:
   - Verify **"Deduplicate2"** node is active
   - Review the unique key logic

2. **Add Google Sheets duplicate check**:
   - Before import, query existing rows
   - Filter out products already in sheet

3. **Clear test data**:
   - Delete test runs from spreadsheet
   - Reset Data Table `RUNS` value

### Issue: High Costs

**Symptoms**: Apify bill higher than expected

**Solutions:**

1. **Review Apify usage**:
   - Check [billing page](https://console.apify.com/billing)
   - Identify which actors are expensive

2. **Optimize scraping**:
   - Reduce `topNProducts`
   - Lower `maxRequestsPerStartUrl` in contact scraper
   - Disable `scrapeSocialMediaProfiles` for unused platforms

3. **Adjust schedule**:
   - Run less frequently (3x/week vs daily)
   - Skip weekly scrape if not needed

4. **Monitor compute units**:
   - Check actor run duration
   - Optimize input parameters
   - Use cheaper actor alternatives

### Issue: Contact Scraper Returns Empty Data

**Symptoms**: No emails or social links found

**Solutions:**

1. **Check website structure**:
   - Some sites don't have public contact info
   - Verify manually that data exists

2. **Increase scraping depth**:
   - Raise `maxDepth` from 2 to 3
   - Increase `maxRequestsPerStartUrl`

3. **Enable browser mode**:
   - Ensure `useBrowser: true` is set
   - Helps with JavaScript-heavy sites

4. **Check domain validity**:
   - Some products don't have functional websites yet
   - Skip products with invalid URLs

---

## Important Warnings

### Email Sending Limits & Account Bans

**Critical**: Sending too many emails can result in account suspension, even with Gmail API.

![Email Account Banned Warning](https://articles.emp0.com/wp-content/uploads/2025/12/product-hunt-scraper-email-banned.png)

**WARNING**: The image above shows what happens when you send emails too aggressively without proper precautions. Your account can be suspended or permanently banned.

#### Gmail API Limits
- **500 emails/day** (personal Gmail accounts)
- **2,000 emails/day** (Google Workspace accounts)
- **Sending to spam traps** → account flagged or banned
- **High bounce rates** (>5%) → deliverability issues
- **Spam complaints** → account suspension or restrictions

**Why This Happens:**
- Sending too many emails too quickly (cold start without warm-up)
- High bounce rates from invalid email addresses (no email verification)
- Spam complaints from recipients
- Sending to spam traps or honeypots
- Poor email content triggering spam filters

**Best Practices:**
- **Start with 10-20 emails/day** for the first week (critical warm-up period)
- **Gradually increase** to 100-200/day for personal Gmail, 500-1000/day for Workspace
- **Use email verification** (ZeroBounce) to reduce bounces and protect sender reputation
- **Include clear unsubscribe link** in every email
- **Monitor metrics daily**: bounce rates, spam complaints, delivery rates
- **Personalize every email** - avoid generic templates
- **Space out sending** throughout the day using batch settings
- **Never send to purchased lists** - only to publicly available Product Hunt contacts

#### General Email Safety
- **Never send unsolicited bulk email** without consent
- **Always include an unsubscribe mechanism**
- **Follow CAN-SPAM Act** (US) and GDPR (EU) regulations
- **Monitor your sender reputation** (use tools like mail-tester.com)
- **Warm up new domains slowly** (increase volume gradually over 2-4 weeks)

### Ethical Scraping Guidelines

- **Respect robots.txt** files
- **Don't overwhelm servers** with rapid requests
- **Product Hunt Terms of Service**: Review before commercial use
- **Personal data handling**: Follow GDPR and local privacy laws
- **Use data responsibly**: Don't resell or misuse contact information

### API Rate Limits

- **Apify**: Generally no hard limits, but costs scale with usage
- **Google Sheets**: 500 requests/100 seconds/user
- **Gmail API**: 1 billion quota units/day (sending = 100 units/email)

### Data Accuracy Disclaimer

- Contact info may be **outdated or incorrect**
- Not all products have **public contact details**
- Email accuracy typically **70-85%** (use verification)
- **Always verify critical information** before important outreach

---

## Next Steps

Congratulations! Your Product Hunt Scraper is now fully configured.

### Recommended Actions:

1. **Monitor first week closely**:
   - Check daily for errors
   - Verify data quality
   - Adjust scraping parameters as needed

2. **Set up monitoring**:
   - Configure n8n error notifications
   - Enable Apify email alerts for failed runs
   - Track email deliverability metrics

3. **Optimize based on results**:
   - Refine product categories
   - Adjust scraping volume
   - Improve email templates

4. **Scale gradually**:
   - Start with conservative settings
   - Increase volume as you verify quality
   - Add optional enhancements one at a time

5. **Join the community**:
   - [Join our Skool community](https://www.skool.com/aia-ai-automation-2762) to share your results
   - Learn from other users
   - Get help and contribute improvements

---

## Support Resources

- **n8n Documentation**: [docs.n8n.io](https://docs.n8n.io)
- **Apify Documentation**: [docs.apify.com](https://docs.apify.com)
- **n8n Community Forum**: [community.n8n.io](https://community.n8n.io)
- **emp0 Community**: [Skool Community](https://www.skool.com/aia-ai-automation-2762)
- **Purchase Workflow**: [Gumroad](https://0emp0.gumroad.com/l/product-hunt-lead-generator)
- **emp0 Support**: [emp0.com](https://emp0.com)
- **discord Support**: [@jym.god](https://discord.gg/RqMKqA3jMS)


---

## Changelog

**Version 1.0** (December 2024)
- Initial release
- Daily and weekly scraping modes
- Google Sheets integration
- Automated email outreach
- Contact enrichment with social profiles

---

**Need help?** Join our [Skool community](https://www.skool.com/aia-ai-automation-2762) for live support and connect with other workflow users.

Happy lead generation! 🚀
