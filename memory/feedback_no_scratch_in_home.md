---
name: Don't write scratch scripts to C:\Users\aldoh\
description: One-shot scripts/SQL/logs Claude generates must land under RApplication/ or NewTrading/ from the start, never the user's Windows home directory.
type: feedback
originSessionId: fd5e246c-38e1-4547-a0ba-784fc9e1dbdf
---
When generating one-shot helpers (fix_*.sh, fix_*.sql, restore_*.R, ad-hoc deploy scripts, SQL migrations, transcripts), write them directly to the correct project subdir, not to `C:\Users\aldoh\` or `/tmp`.

**Why:** On 2026-05-07 we cleaned out 9 stray files that had accumulated in the home dir over months — VM scripts, renv deploy helpers, SQL ALTER, Tlogger logs, a 29MB stale DB push artifact, a podcast transcript. The user had to ask for a cleanup. Default-to-cwd is the root cause: when cwd happens to be `C:\Users\aldoh\` (e.g., from a fresh shell), Bash heredocs and `Rscript -e` files land there silently.

**How to apply:** Before writing any non-trivial helper, pick a destination from this map:
- Shell/R/PowerShell operational scripts → `RApplication/scripts/`
- SQL migrations / one-shot ALTERs → `RApplication/data/`
- Tlogger / R update logs → `RApplication/logs/` (gitignored, keep there)
- Podcast/video transcripts, trade analysis text → `NewTrading/Transcripts/` or appropriate `NewTrading/` subdir
- Build tarballs, package artifacts → `RApplication/` root (existing convention)

If unsure where it belongs, ask before writing — don't default to `$HOME`. Temp `/tmp/*.R` for Rscript multiline workarounds is fine (those are ephemeral); the rule applies to anything the user might want to find again.
