---
name: reference-pscp-recursive-destdir
description: "gcloud compute scp on Windows uses pscp.exe which (unlike openssh) does NOT create the remote destination dir during recursive copies; pre-create the dir, then scp contents."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 5367d3b5-2222-403f-93b3-db52f823f3be
---

# `gcloud compute scp --recurse` on Windows needs the dest dir to exist

`gcloud compute scp` on Windows invokes `pscp.exe` from the PuTTY toolchain.
PuTTY's recursive scp differs from openssh `scp -r` in one critical way:

| Behavior | openssh `scp -r` | pscp `-r` |
|---|---|---|
| `scp -r local_dir remote:/tmp/new_dir` (new_dir absent) | creates `/tmp/new_dir`, copies contents in | fails: `unable to open /tmp/new_dir: failure` for every file |

Symptom: the error repeats for **each** file you tried to copy, because
pscp is opening the destination directory once per file and failing.

**Fix pattern:**

```powershell
$remoteStaging = "/tmp/rstudies_push_$([Guid]::NewGuid().ToString('N').Substring(0,8))"

# 1. Pre-create the remote dir
gcloud compute ssh "$VMUser@$VMInstance" --zone=$Zone --project=$Project `
    --command="mkdir -p ${remoteStaging}"

# 2. scp the staging dir CONTENTS (multiple sources) into the existing dir
$sources = Get-ChildItem -Path $staging -Force | ForEach-Object { $_.FullName -replace '\\','/' }
$scpArgs = $sources + @("${VMUser}@${VMInstance}:${remoteStaging}/")
gcloud compute scp --recurse $scpArgs --zone=$Zone --project=$Project
```

**Why this works:** pscp is happy to recurse into an existing target dir when
the destination exists, and accepts multiple sources in one call (one ssh
handshake, not N).

**How to apply:** Any PowerShell helper that wraps `gcloud compute scp` for
a directory-to-directory push. The `push-db-to-vm.ps1` pattern (single file →
single file) doesn't have this problem; it only bites recursive copies.

Caught 2026-05-26 writing `push-rstudies-to-vm.ps1`.
