---
name: toggle-auto-update
description: Toggle Claude Code auto-update on/off. Use when the user says 禁止自动更新, 允许自动更新, disable auto update, enable auto update, turn off auto update, turn on auto update, stop auto updating, start auto updating, or any variation of toggling automatic Claude Code updates.
---

# Toggle Auto-Update

Toggle Claude Code auto-update across all four disable layers. Always apply ALL layers -- never skip one.

## Paths

| File | Path |
|------|------|
| settings.json | `C:\Users\lch\.claude\settings.json` |
| memory file | `C:\Users\lch\.claude\projects\C--Users-lch\memory\feedback_no_auto_update.md` |
| npm package dir | `C:\Users\lch\AppData\Roaming\npm\node_modules\@anthropic-ai\claude-code` |

## Disable Auto-Update (禁止自动更新)

When the user asks to disable, apply all four layers in order:

### Layer 1a -- env var in settings.json
Add `"CLAUDE_CODE_DISABLE_AUTO_UPDATE": "1"` inside the `"env"` block of settings.json.
Use Edit to add it -- place it after the last existing env entry. Preserve JSON validity (trailing comma on the new line, and ensure the line before it also has a comma).

### Layer 1b -- remove SessionStart hook
In settings.json, inside `hooks.SessionStart[0].hooks`, find and remove the entry with `"command": "claude update"`. Delete the entire object including its trailing comma. Keep the remaining hooks array valid.

### Layer 2 -- filesystem ACL
Run:
```
icacls "C:\Users\lch\AppData\Roaming\npm\node_modules\@anthropic-ai\claude-code" /deny lch:W
```

### Layer 3 -- Windows user env var
Run:
```
setx CLAUDE_CODE_DISABLE_AUTO_UPDATE 1
```

### Update memory file
Edit `C:\Users\lch\.claude\projects\C--Users-lch\memory\feedback_no_auto_update.md`:
- Change `**Current state: AUTO-UPDATE ALLOWED**` to `**Current state: AUTO-UPDATE DISABLED**`
- Update the `last toggled` date to today's date (format: YYYY-MM-DD)

### Verify
Check all four layers and report a summary table showing each layer's status.

---

## Enable Auto-Update (允许自动更新)

When the user asks to enable, reverse all four layers in order:

### Layer 1a -- remove env var from settings.json
Remove the line `"CLAUDE_CODE_DISABLE_AUTO_UPDATE": "1"` from the `"env"` block in settings.json. Clean up the trailing comma so JSON stays valid.

### Layer 1b -- add back SessionStart hook
In settings.json, inside `hooks.SessionStart[0].hooks`, add this entry after the existing entries (usually after the clawd-hook entry):
```json
{
  "type": "command",
  "command": "claude update",
  "shell": "powershell",
  "timeout": 30,
  "async": true
}
```
Make sure to add a comma after the preceding hook entry so the JSON array stays valid.

### Layer 2 -- remove filesystem ACL
Run:
```
icacls "C:\Users\lch\AppData\Roaming\npm\node_modules\@anthropic-ai\claude-code" /remove:d lch
```

### Layer 3 -- clear Windows user env var
Run in PowerShell:
```powershell
[System.Environment]::SetEnvironmentVariable('CLAUDE_CODE_DISABLE_AUTO_UPDATE', '', 'User')
```

### Update memory file
Edit `C:\Users\lch\.claude\projects\C--Users-lch\memory\feedback_no_auto_update.md`:
- Change `**Current state: AUTO-UPDATE DISABLED**` to `**Current state: AUTO-UPDATE ALLOWED**`
- Update the `last toggled` date to today's date (format: YYYY-MM-DD)

### Verify
Check all four layers and report a summary table showing each layer's status.

---

## Verification after every toggle

Always run these checks and report a table:

| Layer | Check | Expected (Disabled) | Expected (Enabled) |
|-------|-------|---------------------|--------------------|
| 1a | env var in settings.json | Present | Absent |
| 1b | SessionStart hook | Absent | Present |
| 2 | icacls deny | Present | Absent |
| 3 | Windows env var | `1` | Empty |
| Mem | memory file state | DISABLED | ALLOWED |

If any layer doesn't match the expected state, fix it before reporting completion.
