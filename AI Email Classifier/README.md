# AI Email Classifier 📬  
Automate Email Classification, Prioritization, and Spam Detection Across Multiple Accounts  

![ai-email-classifier-workflow](https://articles.emp0.com/wp-content/uploads/2025/07/workflow-ai-email-classifier.png)  

This workflow can also be found on the [main n8n.io website](https://n8n.io/workflows/5789-multi-account-email-classifier-with-ai-gmail-discord-and-google-sheets/) 

Created by: Jayant Kumar ([@jharilela](https://github.com/Jharilela))  
🛠 Powered by: Gmail, Google Sheets, OpenAI, Discord, and n8n  

---

## Sample Discord labelling as Spam

![ai-email-classifier-spam](https://articles.emp0.com/wp-content/uploads/2025/07/ai-email-classifier-spam.png)  

## Sample Discord labelling as Legit

![ai-email-classifier-legit](https://articles.emp0.com/wp-content/uploads/2025/07/ai-email-classifier-legit.png)  

---

## Why I Built This

Focus is Expensive. Managing multiple email inboxes every day—personal, business, partnerships, invoices. Logging into each, skimming through noise, flagging important stuff, and deleting spam started eating up hours of my week. I needed a system that helped me **focus** only on what matters without building an entire helpdesk dashboard.  

I already live in Discord. It made sense to push my emails there—but in a fun, digestible, and actionable way. I built **AI Email Classifier 📬** to summarize emails, detect spam, assign priority, and make everything skimmable with pictures and links.  

And it works across **multiple Gmail accounts**.

---

## Key Features

- ✅ Works with **multiple Gmail inboxes**  
- 🧠 Uses AI to **classify spam vs legit**  
- 🎯 Assigns **priority levels**: High / Medium / Low  
- 🗂 Appends everything to a **central Google Sheet**  
- 📸 Sends **visual summaries to Discord** (with image + action links)  
- 🛠 Powered by open-source: [n8n_discord_trigger_bot](https://github.com/Jharilela/n8n_discord_trigger_bot)

---

## How It Works

Here’s the high-level flow:

1. New Email in any inbox triggers the worfklow to start
2. The AI Agent reads the raw content, subject, sender, Gmail labels.
3. It calls a **Google Sheet** that acts as our feedback memory:
   - Emails and domains manually marked as **spam** or **legit**.
4. AI classifies the incoming email using logic:
   - **Spam** if sender or domain is blacklisted, or content matches patterns like:  
     _"promotions, phishing, ads, mass emails, cold offers"_  
   - **Priority** is assigned by:
     - **High**: deadlines, legal, payments, clients, CEO emails  
     - **Medium**: team updates, meetings, project notifications  
     - **Low**: newsletters, FYIs, casual threads  
6. It produces a compact JSON output with:
   - Sender, recipient, subject, summary, priority, priority color, image URL, action URL  
7. The message is formatted visually and posted back to Discord as an embed with:
   - Summary text  
   - Actionable links  
   - Priority color code  
   - Thumbnail (if any)  

---

## Google Sheet Training Table  

The system uses this sheet as live memory to label spam and legit senders:  

```
╔════════════════════╦══════════════╦═════════════════╦══════════════╦════════════════╗
║       Email        ║    Domain    ║  Classification ║  Labelled By ║  Labelled Date ║
╠════════════════════╬══════════════╬═════════════════╬══════════════╬════════════════╣
║ offers@badsite.com ║  badsite.com ║  Spam           ║  Jayant      ║  08/07/2025    ║
║ ceo@trusted.com    ║  trusted.com ║  Legit          ║  Jayant      ║  08/07/2025    ║
╚════════════════════╩══════════════╩═════════════════╩══════════════╩════════════════╝

```


This allows **manual control** to teach the AI which senders to trust or ignore. Every time I see something marked wrong, I just reply in Discord with `"spam"` or `"legit"` on that message thread. That triggers an update to the Sheet via AI parsing and n8n.

---

## Why Manual Input Still Matters

AI isn’t perfect.  
Some spam emails are cleverly disguised. And some senders are contextually important only to *you*.

That’s why I kept a simple feedback loop:
- You tell the bot `"spam"` or `"legit"` on any Discord email message. Or anything along that line
- The AI agent detects the intent and updates the Sheet.
- The AI improves its judgment next time as it now **remembers** your preference

---

## Why Discord?

Because Slack charges per seat and email feels lonely.  
I run most of my operations inside Discord community chats, client rooms, bot alerts.  
Instead of making a full email UI, I turned each email into a **Discord card with a thumbnail, summary, and quick actions**.  
It’s fun. It’s visual. It doesn’t feel like work.  
Email becomes more like a game feed.

---

## Tech Stack

- Gmail → Discord via Gmail trigger node
- Discord → n8n Webhook via [`n8n_discord_trigger_bot`](https://github.com/Jharilela/n8n_discord_trigger_bot)  
- OpenAI GPT-4o (classification + summarization)  
- Google Sheets (feedback memory)  
- Discord Node (embed output with JSON + images)  

---

## Try It Yourself

Clone the workflow JSON, set up your Gmail integrations, and install the [n8n Discord Trigger Bot](https://github.com/Jharilela/n8n_discord_trigger_bot).  
I made this workaround because i couldnt find a discord trigger on n8n. 

Now I just scroll my Discord DMs and know what to reply to, and ignore everything else.Dont let Email spam your brain. Let your AI do the thinking.

---

## 📎 Repo & Credits

- Discord bot trigger: [n8n_discord_trigger_bot](https://github.com/Jharilela/n8n_discord_trigger_bot)
- Creator: [Jay (Emp₀)](https://twitter.com/jharilela)
- Automation tool: [n8n](https://n8n.partnerlinks.io/emp0)
