---
name: subscription-video-sources
description: "MarketLife and BigPictureTrading (BPT) member videos — login-gated, Vimeo/YouTube embeds, how to get IDs and captions"
metadata:
  node_type: memory
  type: reference
  originSessionId: 114ccae7-272d-41a4-b8e4-e2de182f39a3
  modified: 2026-10-26T14:54:45.841Z
---

- MarketLife (marketlifetrading.com/videos/show?id=N) and BPT (secure.bigpicturetrading.com/.../members_home/archives.php) are paid, login-gated. User is signed in only in their own Edge/Chrome; the built-in browser pane is not signed in (redirects to sign_in). Never enter credentials — ask the user to sign in in the pane or use Claude in Chrome.
- BPT archive (checked 2026-10-26): each trading day has two Vimeo videos, "Macro Outlook" and "Where Is The Trade?"; Fridays add a MacroVoices YouTube embed. Vimeo embeds carry no `?h=` hash, so they are likely domain-restricted (play only embedded on BPT).
- Workflow used: a DevTools console script on the archive page collects iframe/data-vimeo/link URLs and groups IDs by date heading + section title (console.table + copy()). Captions: switch the console context to the player.vimeo.com frame and read `window.playerConfig.request.text_tracks` (fetch the VTT). If no tracks exist, use local speech-to-text (e.g. Whisper).
- Deliver summaries (macro view, levels, setups) — not verbatim transcripts; content is copyrighted.
