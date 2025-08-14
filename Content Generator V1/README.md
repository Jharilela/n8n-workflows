# Blog Generator V1 — Automated AI-Powered Blog Creation

![Workflow Screenshot](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v1-workflow.png)

## 📌 Overview
The **Blog Generator V1** is an automated **n8n workflow** that fetches trending topics, generates high-quality SEO-optimized blog posts, creates a featured image, and publishes them directly to your WordPress site — all without manual intervention.

This workflow is designed for:
- **Content marketers** who want to automate daily blog posting
- **SEO specialists** aiming for consistent content output
- **Businesses** that want to maintain a regular publishing schedule

---

## ⚙️ Features
- **Automated Topic Discovery** — Finds an interesting news topic each day using AI.
- **AI-Powered Content Generation** — Creates a 1,500-word SEO-optimized blog post in HTML format.
- **Featured Image Creation** — Uses AI to generate a high-quality blog cover image.
- **Direct WordPress Publishing** — Posts the content automatically with the image attached.
- **Scheduling** — Runs daily (or at your preferred interval) using n8n's schedule trigger.

---

## 🛠 How It Works
1. **Schedule Trigger**  
   Initiates the workflow at a specified time.
2. **AI Topic Finder**  
   Uses OpenAI to find a trending, simple blog topic.
3. **Content Creation**  
   Passes the topic to an AI Agent for a 1,500-word HTML-formatted blog post.
4. **WordPress Draft Creation**  
   Posts the generated content as a draft.
5. **Image Generation & Upload**  
   Creates a featured image based on the post title and uploads it to WordPress.
6. **Publishing**  
   Updates the draft to published status with the featured image set.

---

## 📸 Example Output
![Generated Blog Posts Screenshot](https://articles.emp0.com/wp-content/uploads/2025/08/content-generator-v1-blog.png)

---

## 🔧 Requirements
- **n8n** instance (self-hosted or cloud)
- **OpenAI API key**
- **WordPress API credentials**
- **Categories & tags** pre-created in WordPress

---

## 🚀 Setup Instructions
1. Import the `content generator v1.json` workflow into your n8n instance.
2. Add your **OpenAI API credentials**.
3. Add your **WordPress API credentials**.
4. Adjust the **Schedule Trigger** to your desired posting frequency.
5. Update **category** and **tag IDs** in the `Create a post` node.
6. Activate the workflow.

---

## 📌 Notes
- The blog length, style, and SEO optimizations can be customized in the AI Agent's system message.
- Image size is currently set to `1792x1024` — adjust in the `Generate an image` node if needed.
- Ensure your WordPress API user has permission to create and update posts.
