# Web Graph Mapper (Single-File HTML)

A **single-file, mobile-friendly** webpage mapper that crawls a target URL, builds a **node graph** of internal navigation + external links, and renders it as a clean, nicely spaced “web” using Cytoscape with the **fCoSE** layout.

## What it does

- Takes a **Target URL**
- Fetches page content (via **Jina Reader**, your **proxy**, or direct fetch if CORS allows)
- Extracts links and crawls **internal pages** up to **Max Depth** and **Max Pages**
- Builds a graph:
  - **Page nodes** (internal URLs)
  - **External nodes** (outbound domains)
  - **Terminal nodes** (pages with no internal links, or max depth reached, or fetch failed)
- Renders a spaced-out graph with:
  - **fCoSE layout** (best spacing)
  - **COSE fallback**
  - Optional **concentric rings** based on depth
- Mobile-first UI, touch friendly, “Reflow” button, spacing slider

## File

- `web-graph-mapper.html` (single file)

## Quick start

1. Save the HTML file as:

   ```bash
   web-graph-mapper.html

2. Run a local static server (recommended so browsers behave consistently):

python -m http.server 8080


3. Open in your browser:

http://localhost:8080/web-graph-mapper.html


4. In the UI:

Set Target URL

Leave Mode = Jina Reader for easiest “no backend” usage

Click Map




Controls

Control	Meaning

Target URL	Starting point (root)
Fetch Mode	How the page HTML/text is fetched (CORS-safe options included)
Proxy Base	Base URL for proxy mode
Max Depth	Internal crawl depth limit
Max Pages	Total internal pages to fetch limit
Layout	fCoSE (best), COSE (fallback), Concentric (depth rings)
Spacing	Controls ideal edge length and separation (visual “web” spacing)
Map	Runs crawl + build graph
Reflow	Re-runs layout with current spacing/layout settings
Clear	Clears graph + log


Fetch modes (important)

Browsers block cross-domain HTML fetches without CORS headers. That is normal.

1) Jina Reader (recommended for no backend)

Proxy base default:

https://r.jina.ai/

Fetch format used:

https://r.jina.ai/https://target.site/path


Notes:

Jina often returns text/Markdown-like output. The app includes a fallback link extractor for that.

Some sites block third-party fetchers. If that happens, use your own proxy.


2) Your proxy: query param

When you have an endpoint like:

GET /fetch?url=https://target.site

Set:

Mode: proxy_param

Proxy Base example:


http://localhost:8080/fetch

The app will call:

http://localhost:8080/fetch?url=<encoded URL>

3) Your proxy: path append

When you have an endpoint like:

GET /fetch/<ENCODED_URL>

Set:

Mode: proxy_path

Proxy Base example:


http://localhost:8080/fetch/

The app will call:

http://localhost:8080/fetch/<encoded URL>

4) Direct

Only works if the target site sends permissive CORS headers. Most do not.

Optional local proxy (Node/Express)

If you want consistent HTML fetching and control, run a small proxy.

Minimal example (add allowlists and SSRF protection for any serious use):

// fetch-proxy.js
import express from "express";
import fetch from "node-fetch";

const app = express();

app.get("/fetch", async (req, res) => {
  const url = req.query.url;

  if (!url || !/^https?:\/\//i.test(url)) {
    return res.status(400).send("Invalid URL");
  }

  const r = await fetch(url, { redirect: "follow" });
  const text = await r.text();

  res.set("Access-Control-Allow-Origin", "*");
  res.set("Content-Type", "text/html; charset=utf-8");
  res.send(text);
});

app.listen(8080, () => console.log("Proxy: http://localhost:8080/fetch?url="));

Run:

node fetch-proxy.js

Then in the mapper UI:

Mode: proxy_param

Proxy Base:


http://localhost:8080/fetch

Security note

A proxy that fetches arbitrary URLs can be abused (SSRF). If you deploy this anywhere beyond local LAN:

Add a domain allowlist

Block private IP ranges

Add rate limiting

Add timeouts

Strip sensitive headers


Layout tips (make the “web” look good)

Start with:

Layout: fCoSE

Spacing: 140 to 180


If graph is dense:

Lower Max Pages

Increase Spacing

Use “Reflow” after changing Spacing


If you want depth rings:

Layout: Concentric



Limitations

This is a browser-based crawler. JS-rendered content (SPAs) may not expose links unless you use a renderer backend (Playwright).

Jina mode depends on a third-party fetcher and can be blocked by some sites.

No export yet (JSON/GraphML/SVG) unless you add it.


Roadmap (practical next upgrades)

Section nodes (header/nav/main/footer) and link attachment per section

Collapse/clustering (nav/footer/link clouds) for cleaner graphs

Export graph: JSON + GraphML + SVG/PNG snapshot

Screenshot tiles per page node

Playwright backend mode for JS-heavy pages
