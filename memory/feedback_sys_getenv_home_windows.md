---
name: Sys.getenv("HOME") on Windows resolves to USERPROFILE, not Documents
description: Windows R sets HOME to C:\Users\<user>, NOT C:\Users\<user>\Documents. Launchers building paths from HOME silently break. Use the cwd already set by the .bat instead.
type: feedback
originSessionId: d15d36be-5525-4cb1-836d-35a4a21221dd
---
On Windows R, `Sys.getenv("HOME")` returns `C:\Users\<user>` (USERPROFILE), not `C:\Users\<user>\Documents`. Old `RApplication` launcher scripts (`RJournal/RJournal.R`, `RReporting/launch.R`) used `paste0(Sys.getenv("HOME"), "/RApplication/<App>/app")`, which only worked when something else made HOME=Documents. After the apparent reset to USERPROFILE, both launchers crashed with `No Shiny application exists at the path "C:\Users\aldoh/RApplication/RReporting/app"`.

**Why:** R's HOME on Windows is set by R itself, not by the OS. Default is USERPROFILE. RStudio used to override it to Documents in older configurations, which is probably how the path "happened to work" historically.

**How to apply:**
- All `.bat` launchers in `scripts/` already `cd /d "<project_root>"` before invoking Rscript. So the launched R script just needs `runApp("app", ...)` (relative).
- Never write `paste0(Sys.getenv("HOME"), "/RApplication/...")` in any launcher. Either rely on the cwd, or use an explicit absolute path. `~` and `path.expand("~")` have the same problem on Windows.
- If you find another R script with this pattern, fix it the same way (RJournal, RReporting fixed 2026-05-05; ROrder/RPreTrade already use inline `runApp('.')` so they're fine).
- Symptom is fast: window opens, prints "Lancement <App>", then closes immediately. Add `pause` to the .bat to see the actual error.
