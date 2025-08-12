MCP Tool — Replicate (Flux) Image Generator → WordPress/Twitter

Generates images via **Replicate** Flux models and uploads to WordPress (and optionally Twitter/X). Built to act as an **MCP** module that other agents/workflows call for on-demand image creation.

- Models configured in this workflow:\
  `black-forest-labs/flux-schnell`, `black-forest-labs/flux-dev`, `black-forest-labs/flux-1.1-pro`
- Switch rationale: **lower cost** 💰, **broader model choice** 🎯, **full control of parameters** ⚙️
- Leonardo API credits cannot be used in the **web UI** 🙅‍♂️; separate spend for API vs UI

Links:

- 📜 Prior Leonardo-based workflow: [https://n8n.io/workflows/6363-generate-and-upload-images-with-leonardo-ai-wordpress-and-twitter/](https://n8n.io/workflows/6363-generate-and-upload-images-with-leonardo-ai-wordpress-and-twitter/)
- 📰 Blog automation consuming these images: [https://n8n.io/workflows/6734-ai-blog-automation-publish-hourly-seo-articles-to-wordpress-and-twitter-v3/](https://n8n.io/workflows/6734-ai-blog-automation-publish-hourly-seo-articles-to-wordpress-and-twitter-v3/)

---

## 📥 Inputs

| Field  | Type   | Description                       |
| ------ | ------ | --------------------------------- |
| prompt | string | Text description for the image    |
| slug   | string | Filename slug for WP media        |
| model  | string | One of the configured Flux models |

Example:

```json
{
  "prompt":"Joker watching a Batman movie on his laptop",
  "slug":"joker-watching-batman",
  "model":"black-forest-labs/flux-dev"
}
```

## 📤 Output

```json
{
  "public_image_url": "https://your-wp.com/wp-content/uploads/2025/08/img-joker-watching-batman.webp",
  "wordpress": {...},
  "twitter": {...}
}
```

---

## 🔄 Flow

1. Trigger with `prompt`, `slug`, `model`
2. Build model payload (quality/steps/ratio/output format)
3. Call Replicate: `POST /v1/models/{model}/predictions` (Prefer: wait)
4. Download the generated image URL
5. Upload to WordPress (returns public URL)
6. Optional: upload to Twitter/X
7. Return URL + metadata

---

## 🤖 MCP Use at Scale (emp0.com)

Operational pattern: I currently use this setup for my blog where i generate **~300 posts/month**, each with **4 images** (banner + 2 to 3 inline images) → **~1,000 images/month** produced by this MCP.

💡 **Hybrid Cost-Optimized Setup:**

- **High-priority images** (banners, main visuals): Generated using **Flux Dev** on Leonardo for slightly better prompt adherence.
- **Low-priority images** (inline blog visuals): Generated using **Flux Schnell** on Replicate for maximum cost efficiency.

---

## 💰 Pricing Comparison (per image)


Leonardo per-image cost uses API Basic math: **$9 / 3,500 credits = $0.0025714 per credit**.

- **Flux Schnell (Leonardo)** = 7 credits
- **Flux Dev (Leonardo)** = 7 credits
- **Flux 1.1 Pro equivalent in Leonardo** = **Leonardo Phoenix** based on my experience  = 10 credits

| Flux Model               | Replicate                 | Leonardo API*                  |
| ------------------------ | ------------------------- | ------------------------------- |
| `flux-schnell`           | **$0.0030** (=$3/1,000) | **$0.0180** (7 × $0.0025714)  |
| `flux-dev`               | **$0.0250**              | **$0.0180** (7 × $0.0025714)  |
| `flux-1.1-pro` / Phoenix | **$0.0400**              | **$0.0257** (10 × $0.0025714) |

**Replicate pricing:** [https://replicate.com/pricing](https://replicate.com/pricing)\
**Leonardo pricing:** [https://leonardo.ai/pricing/](https://leonardo.ai/pricing/)\
**Leonardo API usage:** [https://docs.leonardo.ai/docs/commonly-used-api-values](https://docs.leonardo.ai/docs/commonly-used-api-values)

---

## 📊 Monthly Cost Example (1,000 images/month)

Mix: **300 ×`flux-dev` on Leonardo**, 
**700 ×`flux-schnell` on Replicate**.

| Platform/Model           | Images | Price per Image | Total      |
| ------------------------ | ------ | --------------- | ---------- |
| Leonardo `flux-dev`      | 300    | $0.0180        | **$5.40** |
| Replicate `flux-schnell` | 700    | $0.0030        | **$2.10** |
| **Total Monthly Spend**  | 1000   | —               | **$7.50** |

💵 **If using Leonardo for both:**

- 300 × $0.0180 = $5.40
- 700 × $0.0180 = $12.60
- **Total = $18.00**

**Savings:** $10.50/month (**≈58% lower**) with the hybrid setup.

---

## 📌 Notes

- More Replicate models can be added in `Code1` node.
- Parameters tuned for aspect ratio, inference steps, quality, guidance.
- Leonardo credit model is API-only; credits are not spendable in Leonardo's web UI.
