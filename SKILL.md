---
name: windows-utf8-shell
description: Prevent Unicode, Chinese, Japanese, emoji, JSON, Markdown, and API payload corruption when Codex runs Windows PowerShell commands, inline scripts, Python snippets, model/API tests, or file reads/writes. Use whenever a task on Windows/PowerShell touches non-ASCII text, Chinese prompts, UTF-8 files, JSONL/YAML/Markdown reports, subprocess output, or previous output shows question marks or mojibake.
---

# Windows UTF-8 Shell

Keep Windows/PowerShell commands UTF-8 safe.

## Core Rules

1. Source the bundled prelude before commands that read, write, print, or send non-ASCII text:

```powershell
$codexHome = if ($env:CODEX_HOME) { $env:CODEX_HOME } else { Join-Path $env:USERPROFILE ".codex" }
. (Join-Path $codexHome "skills\windows-utf8-shell\scripts\utf8_prelude.ps1")
```

2. Do not pipe inline scripts containing Chinese, Japanese, emoji, prompts, JSON payloads, or Markdown through PowerShell here-strings, `echo`, or shell interpolation. Put that content in UTF-8 files first.

3. Prefer `apply_patch` for stable text files. Prefer Python file I/O for generated reports or exact UTF-8 writes:

```python
Path(path).read_text(encoding="utf-8")
Path(path).write_text(text, encoding="utf-8")
```

4. For PowerShell reads, use literal paths, raw text, and explicit UTF-8:

```powershell
Get-Content -LiteralPath $path -Raw -Encoding UTF8
```

5. Avoid PowerShell redirection and `Out-File` for non-ASCII reports. If PowerShell must write a file, use `-LiteralPath` and explicit encoding, then verify the output. Prefer Python when exact UTF-8 no BOM matters, because Windows PowerShell 5.1 and PowerShell 7 differ on `-Encoding utf8`.

6. Before Python or Node subprocesses, keep these environment settings active:

```powershell
$env:PYTHONIOENCODING='utf-8'
$env:PYTHONUTF8='1'
[Console]::InputEncoding = [System.Text.UTF8Encoding]::new($false)
[Console]::OutputEncoding = [System.Text.UTF8Encoding]::new($false)
$OutputEncoding = [Console]::OutputEncoding
```

For Python stdout/stderr, add this when output may contain Unicode:

```python
import sys
sys.stdout.reconfigure(encoding="utf-8")
sys.stderr.reconfigure(encoding="utf-8")
```

## API Payloads

Build JSON payloads in Python and encode bytes explicitly:

```python
body = json.dumps(payload, ensure_ascii=True).encode("utf-8")
headers = {"Content-Type": "application/json; charset=utf-8"}
```

Use `ensure_ascii=True` for unproven gateways. Use `ensure_ascii=False` only after that path has been proven safe.

## Validation And Recovery

After writing or reading important non-ASCII files, check the output:

```powershell
Get-Content -LiteralPath $path -Raw -Encoding UTF8 | Select-Object -First 1
rg -n "�|锟斤拷|鈥|Ã|Â|\?\?|<U\+FFFD>|mojibake" $path
```

If text becomes `??`, mojibake, empty model replies, or unreadable Chinese, suspect shell transport first. Move prompts and payloads into UTF-8 files, read them with Python, send JSON with `ensure_ascii=True`, and retry before blaming the model or API.
