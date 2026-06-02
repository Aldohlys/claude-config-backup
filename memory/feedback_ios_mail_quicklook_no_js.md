---
name: feedback-ios-mail-quicklook-no-js
description: "iOS Mail (and Files preview) renders HTML attachments via QuickLook, not Safari — JavaScript event listeners on document don't fire, localStorage may be sandboxed. Never test HTML apps that way."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3ed4255d-f6de-4459-bd06-43a46d51dc9a
---

When the user opens an `.html` attachment from iOS Mail (or taps-to-preview from the Files app), iOS uses **QuickLook**, a sandboxed HTML preview — not real Safari/WebKit. Symptoms observed on the BSM calculator (TODO #66, 2026-05-18):

- Page renders visually but tab buttons and segmented-control buttons don't respond to taps
- Same code works fine in desktop browsers and in real iOS Safari
- Switching from per-button `addEventListener` to a single delegated `document.addEventListener('click', ...)` did **not** fix it — the issue isn't binding, it's QuickLook restricting script execution

**Why:** Don't pre-validate HTML apps for iPhone use by emailing the file to oneself. The "test in Safari over HTTP" path is the only honest test.

**How to apply:**
- For mobile-targeted HTML deliverables, instruct the user to access via:
  - Local HTTP server on PC + iPhone Safari (`python -m http.server` on the PC, `http://<lan-ip>:<port>/file.html` on iPhone). On iOS 17+, type the URL with explicit `http://` and a path segment (e.g. `/index.html`) — bare-IP entries trigger HTTPS-upgrade and fail with "No secure connection".
  - Netlify Drop (drag-and-drop file, get instant HTTPS URL, ~24h lifetime, no signup)
- Once tested in real Safari, use Share → Add to Home Screen to install as offline PWA — at that point the file is cached and runs without network.
- Defensive coding still worth doing: wrap `localStorage` in try/catch so a sandboxed environment doesn't crash the script on init.
