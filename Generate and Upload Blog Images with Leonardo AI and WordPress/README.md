# Overview

![Generate and Upload Blog Images with Leonardo AI and WordPress](https://articles.emp0.com/wp-content/uploads/2025/07/generate-and-upload-blog-images-with-leonardo-ai-and-wordpress.png)

This workflow generates high-quality AI images from text prompts using **Leonardo AI**, then automatically uploads the result to your WordPress media library and returns the final image URL.

It functions as a **Modular Content Production (MCP) tool** - ideal for AI agents or workflows that need to dynamically generate and store visual assets on-demand.

This workflow can also be found on the [main n8n.io website](https://n8n.io/workflows/6363-generate-and-upload-blog-images-with-leonardo-ai-and-wordpress/) 

---

## ⚙️ Features

- 🧠 **AI-Powered Generation**  
  Uses Leonardo AI to create 1472x832px images from any text prompt, with enhanced contrast and style UUID preset.

- ☁️ **WordPress Media Upload**  
  Uploads the image as an attachment to your connected WordPress site via REST API.

- 🔗 **Returns Final URL**  
  Outputs the publicly accessible image URL for immediate use in websites, blogs, or social media posts.

- 🔁 **Workflow-Callable (MCP Compatible)**  
  Can be executed standalone or triggered by another workflow. Acts as an image-generation microservice for larger automation pipelines.

---

## 🧠 Use Cases

### For AI Agents (MCP)
- Plug this into multi-agent systems as the "image generation module"
- Generate blog thumbnails, product mockups, or illustrations
- Return a clean `image_url` for content embedding or post-publishing

### For Marketers / Bloggers
- Automate visual content creation for articles
- Scale image generation for SEO blogs or landing pages

### For Developers / Creators
- Integrate with other n8n workflows
- Pass prompt and slug as inputs from any external trigger (e.g., webhook, Discord, Airtable, etc.)

---

## 📥 Inputs

| Field  | Type   | Description                            |
|--------|--------|----------------------------------------|
| prompt | string | Text prompt for image generation       |
| slug   | string | Filename identifier (e.g. `hero-image`) |

Example:
```json
{
  "prompt": "A futuristic city skyline at night",
  "slug": "futuristic-city"
}
```
## 📤 Output
```json
{
  "image_url": "https://yourwordpresssite.com/wp-content/uploads/2025/07/img-futuristic-city.jpg"
}
```
## 🔄 Workflow Summary
1. Receive Prompt & Slug
Via manual trigger or parent workflow execution
2. Generate Image
POST to Leonardo AI's API with the prompt and config
3. Wait & Poll
Delays 1 minute, then fetches final image metadata
4. Download Image
GET request to retrieve generated image
5. Upload to WordPress
Uses WordPress REST API with proper headers
6. Return Result
Outputs a clean image_url JSON object

## 🔐 Requirements
- Leonardo AI account and API Key
- WordPress site with API credentials (media write permission)
- n8n instance (self-hosted or cloud)
- This credential setup:
	- `httpHeaderAuth` for Leonardo headers
	- `httpBearerAuth` for Leonardo bearer token
	- `wordpressApi` for upload

## 🧩 Node Stack
- `Execute Workflow Trigger` / `Manual Trigger`
- `Code` (Input Parser)
- `HTTP Request` → Leonardo image generation
- `Wait` → 1 min delay
- `HTTP Request` → Poll generation result
- `HTTP Request` → Download image
- `HTTP Request` → Upload to WordPress
- `Code` → Return final image URL

## 🖼 Example Prompt
```json
{
  "prompt": "Batman typing on a laptop",
  "slug": "batman-typing-on-a-laptop"
}
```
Will return:
```bash
https://articles.emp0.com/wp-content/uploads/2025/07/img-batman-typing-on-a-laptop.jpg
```
## 🧠 Integrate with AI Agents
This workflow is MCP-compliant—plug it into:
- Research-to-post pipelines
- Blog generators
- Carousel builders
- Email visual asset creators

Trigger it from any parent AI agent that needs to generate an image based on a given idea, post, or instruction.