---
name: reference_python_conda_launch
description: Claude Code launched from plain cmd (not conda shell); activate conda before running Python
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7314cd86-c7d4-45e3-9431-a260c000ac0f
---

Claude Code is now launched from a plain MS-DOS / cmd prompt, NOT from a base-conda PowerShell prompt as before. So conda is not pre-activated in the shell environment.

To run Python in the Bash tool, activate conda first:

```
source /c/Users/aldoh/miniconda3/etc/profile.d/conda.sh && conda activate && python ...
```

Conda Python lives at `/c/Users/aldoh/miniconda3/python` (Python 3.13.5) and can also be called by full path directly. Avoid bare `python3` — it resolves to the Windows Store shim (`WindowsApps/python3`) and fails (exit 49).
