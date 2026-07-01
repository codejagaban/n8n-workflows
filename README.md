# N8n + Uploadcare workflow examples

Ready-to-import [n8n](https://n8n.io) workflows that pair with [Uploadcare](https://uploadcare.com/). Each folder holds a `workflow.json` (import it into n8n) and a `README.md` with setup steps.

## Workflows

- [document-conversion](document-conversion/) — convert an uploaded document to PDF or image previews with Uploadcare's Document Conversion API, then email the CDN link.
- [image-tagging](image-tagging/) — auto-tag every uploaded image with the Uploadcare Object Recognition add-on and push confident labels to your DB/CMS.
