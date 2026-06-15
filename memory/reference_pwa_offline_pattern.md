---
name: reference_pwa_offline_pattern
description: "Template for building an offline-capable HTML deliverable for iPhone — manifest.json + sw.js + register-from-load. Use `tools/bs_calculator/` as the reference implementation."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 3ed4255d-f6de-4459-bd06-43a46d51dc9a
---

When the user asks for a mobile HTML deliverable that "works offline once saved to Home Screen" on iPhone, the working pattern is:

**Three files, all served from the same origin:**

1. `index.html` — main app. In `<head>`, add `<link rel="manifest" href="manifest.json">`. At the end of the script block, register the SW:
   ```js
   if ('serviceWorker' in navigator) {
     window.addEventListener('load', function () {
       navigator.serviceWorker.register('sw.js').catch(function () {});
     });
   }
   ```

2. `manifest.json` — declares the page as an installable PWA so iOS treats Add-to-Home-Screen as a standalone app launch:
   ```json
   {
     "name": "App Name", "short_name": "App",
     "start_url": "./index.html", "scope": "./",
     "display": "standalone", "orientation": "portrait",
     "background_color": "#0b1220", "theme_color": "#0b1220"
   }
   ```

3. `sw.js` — cache-first service worker. Precaches index.html + manifest.json on install; intercepts every GET and serves from cache if present. After first visit, the page works offline forever — even if the hosting URL dies. See `tools/bs_calculator/sw.js` for the working ~25-line implementation (install + activate + fetch handlers, single cache key `bsm-v1`).

**Hosting constraint:** Service Workers require HTTPS (or `localhost`). Plain HTTP over LAN IP does NOT work — iOS Safari refuses to register the SW. Either:
- Host on Netlify Drop (drag the folder onto https://app.netlify.com/drop, free, no signup, gives instant HTTPS URL); or
- GitHub Pages (permanent, requires public repo on free tier); or
- Any HTTPS static host.

**iOS quirks to remember:**
- Don't test by emailing the HTML file — iOS Mail uses QuickLook (no JS event listeners). See [[feedback_ios_mail_quicklook_no_js]].
- iOS Safari HTTPS-upgrade can block plain `http://lan-ip:port` URLs; type the full URL with explicit `http://` + path (e.g. `/index.html`).
- "Add to Home Screen" alone does NOT cache offline without a Service Worker — it just bookmarks the URL.

**Cache-key discipline:** every time you ship new HTML/JS for an installed PWA, bump the `CACHE` constant in `sw.js` (e.g. `bsm-v1` → `bsm-v2` → `bsm-v3`). The `activate` handler diffs the cache keys and deletes the old one, then `install` precaches the new ASSETS on next online visit. Without the bump, the SW serves the stale cached HTML forever even though you redeployed. The 2026-05-19 BSM session went through three rev cycles (v1→v2→v3) and each rev needed a key bump to roll out.

**Reference implementation:** `tools/bs_calculator/` (TODO #66, shipped 2026-05-18, extended 2026-05-19 with Vol/Risks tabs + Position + Taylor-decomposed PnL; see [[project_bs_calculator_design]]). Single-file BSM calculator that became a PWA via the pattern above. Live-tested airplane-mode after install — works.
