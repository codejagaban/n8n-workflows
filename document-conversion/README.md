# Document Conversion to PDF / Image Previews (n8n + Uploadcare)

Convert an uploaded document (for example DOCX to PDF, or a PDF first page to a PNG preview) using Uploadcare's asynchronous Document Conversion REST API, then deliver the converted file's CDN link by email.

## What it does

1. A **Webhook** receives a request with a document file in the binary field `file`.
2. **Upload Document** uploads the file to Uploadcare via the multipart Upload API and gets back a `<UUID>`.
3. **Start Conversion** calls `POST /convert/document/` with `paths: ["<UUID>/document/-/format/pdf/"]` and `store: "1"`.
4. **Extract Token** pulls the conversion `token` from the response.
5. **Wait for Conversion** pauses ~10 seconds so the cloud job can run.
6. **Check Status** calls `GET /convert/document/status/<token>/`.
7. **Conversion Finished?** (IF) routes on `status == "finished"`: true continues, false loops back through **Wait & Retry** to poll again.
8. **Build Output URL** constructs the CDN URL `https://2ta6v1zvst.ucarecd.net/<result UUID>/` for the converted file.
9. A **Gmail** node delivers the download link, then **Respond Success** returns JSON to the webhook caller.

## Prerequisites

- A free [Uploadcare account](https://uploadcare.com/) with your **Public Key** and **Secret Key** (Dashboard → API keys).
- A running n8n instance (Cloud or self-hosted).
- A Gmail account connected to n8n for the **Send Email** (Gmail) node (or swap it for Slack/Telegram).
- Note: the REST API Simple auth header (`Uploadcare.Simple PUBLIC:SECRET`) is server-side only — never expose your secret key in client code.

## Import & setup

1. In n8n, choose **Import from File** and select `workflow.json`.
2. Replace every `YOUR_PUBLIC_KEY` and `YOUR_SECRET_KEY` placeholder in **Upload Document**, **Start Conversion**, and **Check Status** with your real Uploadcare keys (or wire up an n8n Header Auth credential).
3. Open the **Webhook** node and copy its Production/Test URL. POST a request with the document in a binary field named `file`.
4. Configure the **Send Email** (Gmail) node with your Gmail account credential and a real recipient in the **To** field.
5. To produce a PNG preview of the first page instead of a PDF, change the path in **Start Conversion** to `"{{ $json.file }}/document/-/format/png/-/page/1/"`.
6. Activate the workflow.

## Nodes

- **Webhook** (`n8n-nodes-base.webhook`) — entry point; receives the document as binary field `file`.
- **Upload Document** (`n8n-nodes-base.httpRequest`) — multipart upload to `https://upload.uploadcare.com/base/`, returns `{ "file": "<UUID>" }`.
- **Start Conversion** (`n8n-nodes-base.httpRequest`) — `POST /convert/document/` with the conversion path and `store: "1"`.
- **Extract Token** (`n8n-nodes-base.set`) — reads `result[0].token` for status polling.
- **Wait for Conversion** (`n8n-nodes-base.wait`) — pauses before the first status check.
- **Check Status** (`n8n-nodes-base.httpRequest`) — `GET /convert/document/status/<token>/`.
- **Conversion Finished?** (`n8n-nodes-base.if`) — true (index 0) when `status == "finished"`; false (index 1) loops back to retry.
- **Wait & Retry** (`n8n-nodes-base.wait`) — short delay before re-polling on the false branch.
- **Build Output URL** (`n8n-nodes-base.set`) — builds `https://2ta6v1zvst.ucarecd.net/<result.uuid>/`.
- **Send Email** (`n8n-nodes-base.gmail`) — emails the converted-file link via Gmail.
- **Respond Success** (`n8n-nodes-base.respondToWebhook`) — returns JSON to the caller.

## Related blog post

Full walkthrough: [Automate document conversion to PDF and image previews in n8n with Uploadcare](https://uploadcare.com/blog/n8n-document-conversion-pdf-uploadcare/)
