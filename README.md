# Kairo Proposal Generator

A single-page web app that generates, edits, and exports construction proposals.
It's **frontend-only** — one self-contained `index.html` (HTML + CSS + JS) that
loads its libraries from CDNs. Host it anywhere static, or embed it in
GoHighLevel.

Branded for **Alliance For Contractors / BuildSuite** (deep navy `#0A1F44` +
gold `#E0A82E`).

---

## What it does

1. **Form** — collect job details (trade, client, location, scope, budget, notes).
2. **Generating** — loading spinner while Kairo writes the proposal.
3. **Result** — an editable markdown source on the left and a **live preview** on
   the right, plus export, copy, regenerate, and "new proposal" actions.

The proposal comes back as markdown. You can edit the raw markdown and the
rendered preview updates as you type. Export uses your **current (edited)**
content.

---

## 1. Add your API key

Open `index.html` and find the **CONFIG** block near the top of the `<script>`
section:

```js
const API_URL = "https://kairo-output-production.up.railway.app/generate";
const API_KEY = "PASTE_ROTATED_SECRET_HERE";  // sent as the x-api-key header
```

- Leave `API_URL` as-is unless your endpoint changes.
- Replace `PASTE_ROTATED_SECRET_HERE` with your rotated secret.

> ⚠️ **Security note:** The API key is shipped to the browser and is **visible**
> to anyone who opens the page or its dev tools. This is unavoidable for a
> client-side-only app. **Keep this page behind a login** (e.g. a GoHighLevel
> membership/portal area), and **rotate the key** if it ever leaks. Treat the
> key as a low-privilege, easily-rotated credential.

### API contract (for reference)

`POST {API_URL}` with headers:

```
Content-Type: application/json
x-api-key: <API_KEY>
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

Form-field mapping: *Scope of Work → `scope`*, *Client Name → `clientName`*.

---

## 2. Host it

It's a single static file — no build step, no server.

### Netlify (drag & drop)
1. Go to <https://app.netlify.com/drop>.
2. Drag the folder (or just `index.html`) onto the page.
3. You get a live URL instantly. (Add it to a password-protected site / put it
   behind your login.)

### Netlify / Vercel (Git)
1. Push this repo to GitHub.
2. In Netlify or Vercel, "Import" the repo. No build command needed; the
   publish/output directory is the repo root.
3. Deploy.

### Any static host / your own server
Upload `index.html` to S3 + CloudFront, GitHub Pages, Cloudflare Pages, nginx,
Apache — anything that can serve a static file. No special config required.

### GoHighLevel embed
Because everything is in one file, you can embed it inside a GHL page:

- **Custom Code / HTML element:** paste the entire contents of `index.html` into
  a GHL "Custom Code" or "Code Element" block (you can drop the outer
  `<!DOCTYPE html>` / `<html>` / `<head>` / `<body>` wrappers if GHL complains —
  keep the `<style>`, the markup inside `<body>`, the CDN `<script>` tags, and
  the app `<script>`), **or**
- **iframe:** host `index.html` somewhere static (above) and embed it with
  `<iframe src="https://your-host/index.html" style="width:100%;height:1200px;border:0;"></iframe>`.

Either way, **place the page inside a logged-in/members-only area** so the API
key isn't exposed publicly.

---

## Export formats

On the result screen, pick a format and click **Download**. Exports use the
current (possibly edited) markdown:

- **Markdown (.md)** — the raw markdown.
- **Plain text (.txt)** — markdown stripped of `#`, `**`, list markers, etc. for
  clean text.
- **PDF (.pdf)** — the rendered preview (real headings, bold, lists, tables)
  exported via html2pdf.js.

Filenames are `Proposal - {Client Name}.{ext}`.

Other actions: **Copy** (current content to clipboard), **Regenerate**
(re-calls the API with the same inputs — asks for confirmation since it discards
edits), **New proposal** (back to the form).

---

## Libraries (all client-side, loaded from CDN)

- [marked.js](https://github.com/markedjs/marked) — markdown → HTML
- [DOMPurify](https://github.com/cure53/DOMPurify) — sanitizes rendered HTML
  before preview/PDF
- [html2pdf.js](https://github.com/eKoopmans/html2pdf.js) — HTML → PDF (bundles
  jsPDF + html2canvas)

An internet connection is required so the browser can load these CDN scripts.

---

## Files

- `index.html` — the entire app.
- `README.md` — this file.
