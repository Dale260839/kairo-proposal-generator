# Kairo Proposal Generator

A single-page web app that generates, edits, exports, and emails construction
proposals. It's **frontend-only** — one self-contained `index.html`
(HTML + CSS + JS) that loads its libraries from CDNs. Host it anywhere static,
or embed it in GoHighLevel.

Branded for **Alliance For Contractors / BuildSuite** (deep navy `#0A1F44` +
gold `#E0A82E`).

---

## What it does

A three-state flow, one screen visible at a time:

1. **Form** — collect job details (trade, client, location, scope, budget, notes).
2. **Generating** — loading spinner while Kairo writes the proposal.
3. **Result** — an editable markdown source on the left and a **live preview** on
   the right, plus export, copy, **send to email**, regenerate, and "new
   proposal" actions.

The proposal comes back as markdown. You can edit the raw markdown and the
rendered preview updates as you type. Export and email use your **current
(edited)** content.

---

## 1. Configure the endpoint

Open `index.html` and find the **CONFIG** block near the top of the `<script>`
section:

```js
const API_URL = "https://kairo-output-production.up.railway.app/generate";
```

That's the only value to set. Leave it as-is unless your backend URL changes.

> **No API key in the browser.** This app does **not** ship a secret. The
> backend authorizes requests by checking the **Origin** of this page (plus
> rate-limiting), so there's nothing in the page to leak. To lock things down,
> serve the page from an allow-listed origin (e.g. your GoHighLevel
> members/portal area) and configure the backend's allowed origins accordingly.

The **Send to Email** endpoint is derived automatically from `API_URL` by
swapping the trailing `/generate` for `/send-email` — so both live on the same
backend and you only configure one URL.

### API contract (for reference)

**Generate** — `POST {API_URL}` (i.e. `.../generate`)

```
Content-Type: application/json
```

Body (exact, case-sensitive keys):

```json
{
  "trade": "",
  "scope": "",
  "clientName": "",
  "location": "",
  "budget": "",
  "notes": ""
}
```

Response:

```json
{ "proposal": "...markdown...", "sources_used": 12 }
```

`sources_used` is shown on the result screen as the "knowledge chunks used" count.

Form-field mapping: *Scope of Work → `scope`*, *Client Name → `clientName`*.

**Send email** — `POST {API base}/send-email`

```
Content-Type: application/json
```

Body:

```json
{
  "clientName": "",
  "clientEmail": "client@example.com",
  "subject": "",
  "coverNoteHtml": "<div>...</div>",
  "pdfBase64": "<base64 PDF, no data: prefix>",
  "pdfFilename": "client-proposal.pdf"
}
```

The frontend renders the preview to a PDF (via html2pdf.js), base64-encodes it,
and posts it with the recipient, subject, and cover-note HTML. The backend
handles the GHL contact upsert, PDF upload, and email send. A non-2xx response
(or a JSON `{ "error": "..." }`) is surfaced to the user as an error toast.

---

## 2. Host it

It's a single static file — no build step, no server.

### Netlify (drag & drop)
1. Go to <https://app.netlify.com/drop>.
2. Drag the folder (or just `index.html`) onto the page.
3. You get a live URL instantly. (Put it behind your login if you want access
   controlled.)

### Netlify / Vercel (Git)
1. Push this repo to GitHub.
2. In Netlify or Vercel, "Import" the repo. No build command needed; the
   publish/output directory is the repo root.
3. Deploy.

### Any static host / your own server
Upload `index.html` to S3 + CloudFront, GitHub Pages, Cloudflare Pages, nginx,
Apache — anything that can serve a static file. No special config required.

> Whatever you choose, the host's origin must be **allow-listed by the backend**,
> otherwise `/generate` and `/send-email` will reject the requests.

### GoHighLevel embed
Because everything is in one file, you can embed it inside a GHL page:

- **Custom Code / HTML element:** paste the entire contents of `index.html` into
  a GHL "Custom Code" or "Code Element" block (you can drop the outer
  `<!DOCTYPE html>` / `<html>` / `<head>` / `<body>` wrappers if GHL complains —
  keep the `<style>`, the markup inside `<body>`, the CDN `<script>` tags, and
  the app `<script>`), **or**
- **iframe:** host `index.html` somewhere static (above) and embed it with
  `<iframe src="https://your-host/index.html" style="width:100%;height:1200px;border:0;"></iframe>`.

---

## Result-screen actions

On the result screen, the markdown editor and live preview sit side by side. The
toolbar below them offers:

### Export

Pick a format and click **Download**. Exports use the current (possibly edited)
markdown:

- **PDF (.pdf)** — the rendered preview (real headings, bold, lists, tables)
  exported via html2pdf.js, styled with the brand palette.
- **Markdown (.md)** — the raw markdown.
- **Plain text (.txt)** — markdown stripped of `#`, `**`, list markers, etc. for
  clean text.

Filenames are `Proposal - {Client Name}.{ext}`.

### Send to Email

Click **✉️ Send to Email** to open a modal that's pre-filled from the job
details:

- **Client email** (required, validated).
- **Subject** — defaults to e.g. *"{Trade} Proposal for {Client}"*, editable.
- **Cover note** — a friendly default greeting, editable; shown in the email body.

On send, the app renders the current preview to a PDF (capped at 5 MB),
base64-encodes it, and posts it to the `/send-email` endpoint along with the
recipient, subject, and cover-note HTML. Success and failures are reported via
toast. The modal closes on **Escape**, on a backdrop click, or via **Cancel** /
the **×** button.

### Other actions

- **Copy** — current content to the clipboard.
- **Regenerate** — re-calls the API with the same inputs (asks for confirmation
  since it discards edits).
- **New proposal** — back to the form.

---

## Libraries (all client-side, loaded from CDN)

- [marked.js](https://github.com/markedjs/marked) `12.0.2` — markdown → HTML
- [DOMPurify](https://github.com/cure53/DOMPurify) `3.1.6` — sanitizes rendered
  HTML before preview/PDF/email
- [html2pdf.js](https://github.com/eKoopmans/html2pdf.js) `0.10.2` — HTML → PDF
  (bundles jsPDF + html2canvas)

An internet connection is required so the browser can load these CDN scripts.

---

## Files

- `index.html` — the entire app.
- `README.md` — this file.
