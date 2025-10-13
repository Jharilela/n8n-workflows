# 🐱 MemeCoin Art Generator - using Gemini Flash NanoBanana & upload to Twitter

Automatically generates **memecoin art** and posts it to **Twitter (X)** powered by **Google Gemini**, **NanoBanana image generation**, and **n8n automation**.

---

## 🧩 Overview

This workflow creates viral style memecoin images (like *Popcat*) and posts them directly to Twitter with a witty, Gen Z style tweet.  

It combines **text to image AI**, **scheduled triggers**, and **social publishing**, all in one seamless flow.

**Workflow flow:**
1. Define your memecoin mascot (name, description, and base image URL).  
2. Generate an AI image prompt and a meme tweet.  
3. Feed the base mascot image into **Gemini Image Generation API**.  
4. Render a futuristic memecoin artwork using **NanoBanana**.  
5. Upload the final image and tweet automatically to Twitter.

---

## 🧠 Workflow Diagram

![Workflow Diagram](https://articles.emp0.com/wp-content/uploads/2025/10/memecoin-workflow.png)

---

## ⚙️ Key Components

| Node | Function |
|------|-----------|
| **Schedule Trigger** | Runs automatically at chosen intervals to start meme generation. |
| **Define Memecoin** | Defines mascot name, description, and base image URL. |
| **AI Agent** | Generates tweet text and creative image prompt using Google Gemini. |
| **Google Gemini Chat Model** | Provides trending topic context and meme phrasing. |
| **Get Source Image** | Fetches the original mascot image (e.g., Popcat). |
| **Convert Source Image to Base64** | Prepares image for AI based remixing. |
| **Generate Image using NanoBanana** | Sends the prompt and base image to Gemini Image API for art generation. |
| **Convert Base64 to PNG** | Converts the AI output to an image file. |
| **Upload to Twitter** | Uploads generated image to Twitter via media upload API. |
| **Create Tweet** | Publishes the tweet with attached image. |

---

## 🪄 How It Works

1️⃣ **Schedule Trigger** - starts the automation (e.g., hourly or daily).  
2️⃣ **Define Memecoin** - stores your mascot metadata:
   ```yaml
   memecoin_name: popcat
   mascot_description: cat with open mouth
   mascot_image: https://i.pinimg.com/736x/9d/05/6b/9d056b5b97c0513a4fc9d9cd93304a05.jpg
   ```  
3️⃣ **AI Agent** - prompts Gemini to:
   - Write a short 100 character tweet in Gen Z slang.
   - Create an image generation prompt inspired by current meme trends.  
4️⃣ **NanoBanana API** - applies your base image + AI prompt to create art.  
5️⃣ **Upload & Tweet** - final image gets uploaded and posted automatically.

---

## 🧠 Example Output

**Base Source Image:**  
![Base Image](https://articles.emp0.com/wp-content/uploads/2025/10/popcat-original.jpg)

**Generated Image (AI remix):**  
![Generated Image](https://articles.emp0.com/wp-content/uploads/2025/10/popcat-edit.png)

**Published Tweet:**  
![Tweet Example](https://articles.emp0.com/wp-content/uploads/2025/10/popcat-edit-twitter-1.png)

Example tweet text:
> Popcat's about to go absolutely wild, gonna moon harder than my last test score! 🚀📈 We up! #Popcat #Memecoin

---

## 🧩 Setup Tutorial

### 1️⃣ Prerequisites

| Tool | Purpose |
|------|----------|
| **n8n (Cloud or Self hosted)** | Workflow automation platform |
| **Google Gemini API Key** | For generating tweet and image prompts |
| **Twitter (X) API OAuth1 + OAuth2** | For uploading and posting tweets |

---

### 2️⃣ Import the Workflow

1. Download `memecoin art generator.json`.  
2. In n8n, click **Import Workflow → From File**.  
3. Set up and connect credentials:
   - Google Gemini API  
   - Twitter OAuth  
4. (Optional) Adjust **Schedule Trigger** frequency to your desired posting interval.

---

### 3️⃣ Customize Your MemeCoin

In the **Define Memecoin** node, edit these fields to change your meme theme:
```javascript
memecoin_name: "doggo"
mascot_description: "shiba inu in astronaut suit"
mascot_image: "https://example.com/shiba.jpg"
```

That’s it - next cycle will generate your new meme and post it.

---

### 4️⃣ API Notes

- **Gemini Image Generation API Docs:**  
  [https://ai.google.dev/gemini-api/docs/image-generation#gemini-image-editing](https://ai.google.dev/gemini-api/docs/image-generation#gemini-image-editing)
- **API Key Portal:**  
  [https://aistudio.google.com/api-keys](https://aistudio.google.com/api-keys)

---

## 🌐 Links

- 💼 [Emp0 — AI Revenue Team](https://emp0.com)  
- 🧠 [Content Farming v3 Workflow](https://0emp0.gumroad.com/l/content-farming-v3)  
- 💬 [Discord Community](https://discord.gg/emp0)  
