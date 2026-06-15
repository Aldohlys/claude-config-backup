---
name: feedback_shiny_fileinput_upload_complete
description: "Shiny fileInput's green \"Upload complete\" is a built-in label (browser->server transfer), not custom text"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a2854ddf-d4f4-4f36-b9b1-d5d843760743
---

The green **"Upload complete"** text on a Shiny `fileInput` progress bar is **hardcoded in the framework** — it reflects only the browser→server file transfer, not any app-side parsing. It is not a string set anywhere in the app, so it can't be reworded without fragile JS/CSS overrides.

**How to apply:** Don't hunt for "Upload complete" in the codebase — it isn't there. To confirm the *actual* processing, add your own `showNotification(...)` on the success path (RReporting `server.R` `observeEvent(input$file)` emits "Processed N trade record(s) from <file>").
