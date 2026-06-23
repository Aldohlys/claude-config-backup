---
name: reference_schtasks_xml_registration
description: Registering a Windows scheduled task from an XML file — Git Bash flag mangling + UTF-16 requirement
metadata: 
  node_type: memory
  type: reference
  originSessionId: beb29f4f-db89-499e-ac90-41457a034cf6
---

Registering a Windows scheduled task via `schtasks /create /xml <file>` has two traps that each cost a failed attempt:

**1. Git Bash mangles the slash-flags.** Running `schtasks /create /tn ... /xml ... /f` through the **Bash tool** turns `/create` into a path (`C:/Program Files/Git/create`). `cmd //c schtasks /create ...` only protects the `//c`; `/create` is still mangled. **Fix: use the PowerShell tool**, where `schtasks /create /tn "\Folder\Name" /xml $path /f` runs verbatim. (Or `Register-ScheduledTask -Xml ... -TaskPath "\Folder\"` — but see #2.)

**2. The XML must be UTF-16.** `schtasks /xml` (and `Register-ScheduledTask -Xml`) require the task XML in **UTF-16**. The Write tool emits **UTF-8**, so registration fails with `(1,40)::ERREUR : impossible de changer d'encodage` (column 40 = the `encoding=` attribute). Existing working task XMLs (e.g. `scripts/RefreshInterestRates.xml`) are UTF-16 with `encoding="UTF-16"`. Fix after writing with the Write tool:
```powershell
$p = "...\Task.xml"
$c = (Get-Content -Raw $p) -replace 'encoding="UTF-8"', 'encoding="UTF-16"'
[System.IO.File]::WriteAllText($p, $c, [System.Text.Encoding]::Unicode)   # UTF-16 LE + BOM
schtasks /create /tn "RApplication\TaskName" /xml $p /f
```
The committed XML then stays UTF-16 (consistent with the other task files in `scripts/`).

**Settings worth setting** (see [[gcloud-vm]]-adjacent automation notes): `<StartWhenAvailable>true` runs ASAP if the PC was off at the trigger time; `<RestartOnFailure><Interval>PT30M</Interval><Count>12</Count></RestartOnFailure>` retries through the day when the action exits non-zero (e.g. a TWS-gated script that exits 1 while TWS is down) — pair it with a `.bat` action that does `exit /b %ERRORLEVEL%` so the failure actually propagates.

Reference implementation: `scripts/CollectOptionSurface.xml` + `scripts/collect_option_surface.bat` (TODO #50 daily IV-surface collection, registered 2026-06-10).
