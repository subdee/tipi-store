# BentoPDF

A privacy-first PDF toolkit you host yourself. Over 100 tools for editing, converting and securing PDFs, all of them running as WebAssembly inside your browser — the container is a plain nginx serving static files, and the documents you open never travel to it.

## Core features

- **Organise** — merge, split, reorder, extract, delete, rotate and crop pages, N-up and booklet layouts, posterise, compare two PDFs side by side
- **Edit** — annotate, highlight, comment, add shapes and stamps, edit existing text using the document's own fonts, edit bookmarks and metadata
- **Forms** — create fillable forms with text fields, checkboxes, dropdowns and signature fields, or fill in existing ones (XFA included)
- **Convert to PDF** — images (JPG, PNG, WebP, HEIC, TIFF, SVG, PSD), Word, Excel, PowerPoint and OpenDocument files, Markdown, EPUB, MOBI, CBZ, emails
- **Convert from PDF** — images, SVG, plain text, JSON, CSV, Excel, CBZ, plus table extraction and OCR to make scans searchable
- **Secure** — encrypt and decrypt, set permissions, draw or upload a signature, add and validate X.509 digital signatures, redact, sanitise, strip metadata
- **Optimise** — compress, repair, linearise for web viewing, deskew scans, remove blank pages, standardise page sizes
- **Workflow builder** — chain the tools together into a reusable pipeline with a visual node editor

## Configuration notes

- **Nothing to configure** — the app has no accounts, no database and no server-side state. Install it and open it.
- **HTTPS matters** — Office file conversion uses LibreOffice compiled to WebAssembly, which needs `SharedArrayBuffer` and therefore a secure context. Reaching the app over `https://` (or `http://localhost`) works; reaching it over a plain `http://192.168.x.x` LAN address will disable those particular tools. Expose the app through Runtipi with a domain and certificate if you want them.
- **WASM modules from a CDN** — the heavier engines (PyMuPDF, Ghostscript, CoherentPDF) are fetched by your browser from jsDelivr the first time a tool that needs them is used. Everything else is served from the container. You can point them elsewhere per browser under **Advanced Settings** in the UI, or [prepare an air-gapped bundle](https://github.com/alam00000/bentopdf#air-gapped--offline-deployment) if the machines have no internet access.
- **Hiding tools** — to remove tools from the UI, mount a `config.json` containing `{"disabledTools": ["edit-pdf", "sign-pdf"]}` at `/usr/share/nginx/html/config.json` in the container. Tool IDs are the page URL without `.html`.
- **Self-hosted build** — this app uses the `bentopdf-simple` image, which is the full tool set without the marketing pages of the public site. It is not a cut-down version.

## Data layout

- No volumes. Nothing is persisted, so there is nothing to back up.

## Licensing

BentoPDF is dual-licensed: AGPL-3.0 for personal and open-source use, or a paid commercial licence for proprietary and public-facing deployments. Self-hosting it for yourself falls under the AGPL.

Source: <https://github.com/alam00000/bentopdf> · Docker image: `ghcr.io/alam00000/bentopdf-simple`
