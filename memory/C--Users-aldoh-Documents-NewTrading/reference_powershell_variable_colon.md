---
name: reference-powershell-variable-colon
description: "PowerShell parses `$Var:` as a scope-qualified variable reference; use `${Var}:` to disambiguate when a colon follows a variable in a string."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 5367d3b5-2222-403f-93b3-db52f823f3be
---

# PowerShell `$Var:` is a scope-qualified reference

When a variable is followed by `:` inside a double-quoted string (e.g.
`"$VMInstance:$Path"`), PowerShell's parser reads it as a **scope-qualified**
variable reference — `$scope:name`, like `$env:PATH` or `$script:Foo`. If the
text after the colon isn't a valid variable-name character sequence, it errors:

```
Variable reference is not valid. ':' was not followed by a valid variable name character.
Consider using ${} to delimit the name.
```

**Fix:** wrap the name in braces:

```powershell
# WRONG — scope-qualified parse, fails
"$VMInstance:$VMInstallPath"

# RIGHT — explicit name delimiter
"${VMInstance}:${VMInstallPath}"
```

**Why:** The known scopes (`global:`, `local:`, `script:`, `private:`, drive
prefixes like `env:` and `function:`) share grammar with arbitrary `$Foo:bar`,
so the parser commits before checking. Braces force the closure of the name
before the colon.

**How to apply:** Anywhere a PowerShell variable is interpolated immediately
before a `:` — scp targets (`user@host:/path`), URLs (`http://${host}:port`),
Windows drive prefixes constructed dynamically, time-of-day strings. Grep
authored scripts for `\$\w+:` before commit; it's almost always a bug.

Caught 2026-05-26 in `RApplication/scripts/push-analyze-to-vm.ps1` —
`"$VMInstance:$VMInstallPath"` failed at parse time before the script could
even run.
