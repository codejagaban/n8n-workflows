# Automatic Image Tagging & Organization (n8n + Uploadcare)

Automatically label every newly uploaded image with the Uploadcare Object Recognition add-on (AWS Rekognition), keep only the confident tags, and push them to your database or CMS for search and organization.

## What it does

1. Receives an Uploadcare `file.uploaded` webhook and extracts the file `uuid`.
2. Starts the Object Recognition add-on with `POST /addons/aws_rekognition_detect_labels/execute/` (`{ "target": "<UUID>" }`).
3. Waits a few seconds, then polls the add-on status endpoint with the returned `request_id`.
4. When the status is `done`, fetches the file info with `GET /files/<UUID>/` and reads `appdata.aws_rekognition_detect_labels.data.labels[]`.
5. A Code node filters labels by confidence (default `>= 80`) and keeps just the `Name` values as tags.
6. A Set node builds a clean record (`uuid`, `cdnUrl`, `tags`), and a final HTTP Request writes the tags to your DB/CMS.

## Prerequisites

- An Uploadcare account with a **public** and **secret** API key.
- The **Object Recognition** add-on enabled on your Uploadcare project (Dashboard → your project).
- A running n8n instance (cloud or self-hosted) reachable by Uploadcare for the webhook callback.
- A destination for the tags (a database, CMS, or any HTTP endpoint). The bundled "Store Tags" node points at a placeholder URL.

## Import & setup

1. In n8n, choose **Import from File** and select `workflow.json`.
2. Replace every `YOUR_PUBLIC_KEY` and `YOUR_SECRET_KEY` placeholder in the three Uploadcare HTTP Request nodes (Execute Rekognition, Check Add-on Status, Get File Info). The `Authorization` header uses `Uploadcare.Simple <PUBLIC_KEY>:<SECRET_KEY>` — this is server-side only, so keep it out of any client.
3. Open the **Webhook (file.uploaded)** node, copy its Production URL, and register it in the Uploadcare Dashboard → API → Webhooks for the `file.uploaded` event.
4. Edit the **Store Tags (DB/CMS)** node and point the URL (and headers/body) at your real database, CMS, or API.
5. Optionally adjust `CONFIDENCE_THRESHOLD` in the **Extract Tags** Code node, and tune the **Wait** durations to match how quickly your project returns add-on results.
6. Activate the workflow.

## Nodes

- **Webhook (file.uploaded)** — entry point; receives the Uploadcare event payload.
- **Extract UUID** — Set node that pulls `body.data.uuid` into a clean `uuid` field.
- **Execute Rekognition** — HTTP POST that triggers the Object Recognition add-on for the file.
- **Wait for Add-on** — pauses before the first status check (add-ons run asynchronously).
- **Check Add-on Status** — HTTP GET against the execute status endpoint using `request_id`.
- **Add-on Done?** — IF node; output 0 (true) when `status == "done"`, output 1 (false) loops back.
- **Wait & Retry** — short pause, then re-checks status until the add-on finishes.
- **Get File Info** — HTTP GET `/files/<UUID>/` to read the `appdata` add-on results.
- **Extract Tags** — Code node that filters labels by confidence and returns tag names.
- **Build Tag Record** — Set node assembling `uuid`, `cdnUrl`, and `tags`.
- **Store Tags (DB/CMS)** — HTTP POST that writes the tag record to your destination.

## Related blog post

Full walkthrough: [Automatically tag and organize images in n8n with Uploadcare](https://uploadcare.com/blog/n8n-automatic-image-tagging-uploadcare/)
