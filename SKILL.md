---
name: toggle-auto-update
description: Toggle Claude Code auto-update on/off. Use when the user says 禁止自动更新, 允许自动更新, disable auto update, enable auto update, turn off auto update, turn on auto update, stop auto updating, start auto updating, or any variation of toggling automatic Claude Code updates.
---

# Toggle Auto-Update

Toggle Claude Code auto-update across all four disable layers. Always apply ALL layers -- never skip one.

## Step 0: Discover paths (do this first, every time)

Before applying any layer, dynamically discover the user's actual paths:

**settings.json path:**
```bash
# On Windows: %USERPROFILE%\.claude\settings.json
# On macOS/Linux: $HOME/.claude/settings.json
```
Use the user's home directory -- never hardcode a username.

**npm global claude-code directory:**
```bash
npm root -g
```
This returns something like `C:\Users\<username>\AppData\Roaming\npm\node_modules`. Append `\@anthropic-ai\claude-code` to get the target dir.

**Current username:**
```bash
# Windows: echo %USERNAME%
# macOS/Linux: whoami
```

**Memory file (optional -- only if the user already has one):**
Search for `feedback_no_auto_update.md` under the user's `.claude/projects/` directory. If found, keep it in sync. If not found, skip the memory update step and tell the user they can create one.

---

## Disable Auto-Update (禁止自动更新)

When the user asks to disable, apply all four layers in order:

### Layer 1a -- env var in settings.json
Add `"CLAUDE_CODE_DISABLE_AUTO_UPDATE": "1"` inside the `"env"` block of the user's settings.json.
Use Edit to add it -- place it after the last existing env entry. Preserve JSON validity (trailing comma on the new line, and ensure the line before it also has a comma).
If the settings.json does not have an `"env"` block yet, create one.

### Layer 1b -- remove SessionStart hook
In settings.json, inside `hooks.SessionStart[0].hooks`, find and remove the entry with `"command": "claude update"`. Delete the entire object including its trailing comma. Keep the remaining hooks array valid.
If no such hook exists, skip this layer (it's already disabled).

### Layer 2 -- filesystem ACL (Windows only)
Run (replace `<username>` and `<npm_root>` with values from Step 0):
```
icacls "<npm_root>\@anthropic-ai\claude-code" /deny <username>:W
```
Example: `icacls "C:\Users\zhangsan\AppData\Roaming\npm\node_modules\@anthropic-ai\claude-code" /deny zhangsan:W`

On macOS/Linux, skip this layer or use `chmod -R a-w <path>` instead.

### Layer 3 -- user env var
**Windows:**
```
setx CLAUDE_CODE_DISABLE_AUTO_UPDATE 1
```
**macOS/Linux:**
Add `export CLAUDE_CODE_DISABLE_AUTO_UPDATE=1` to the user's shell profile (`.bashrc`, `.zshrc`, etc.)

### Update memory file (if exists)
If a `feedback_no_auto_update.md` memory file was found in Step 0:
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
In settings.json, look for `hooks.SessionStart`. If it exists, inside `SessionStart[0].hooks`, add this entry:
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

If `hooks.SessionStart` does not exist, create the full structure:
```json
"hooks": {
  "SessionStart": [
    {
      "matcher": "",
      "hooks": [
        {
          "type": "command",
          "command": "claude update",
          "shell": "powershell",
          "timeout": 30,
          "async": true
        }
      ]
    }
  ]
}
```

### Layer 2 -- remove filesystem ACL (Windows only)
Run (replace `<username>` and `<npm_root>` with values from Step 0):
```
icacls "<npm_root>\@anthropic-ai\claude-code" /remove:d <username>
```

On macOS/Linux, reverse whatever was done in Layer 2 disable.

### Layer 3 -- clear user env var
**Windows:**
```powershell
[System.Environment]::SetEnvironmentVariable('CLAUDE_CODE_DISABLE_AUTO_UPDATE', '', 'User')
```
**macOS/Linux:**
Remove the `CLAUDE_CODE_DISABLE_AUTO_UPDATE` line from the shell profile.

### Update memory file (if exists)
If a `feedback_no_auto_update.md` memory file was found in Step 0:
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
| 2 | filesystem ACL | Deny write | No deny |
| 3 | OS env var | `1` | Empty |
| Mem | memory file state (if exists) | DISABLED | ALLOWED |

If any layer doesn't match the expected state, fix it before reporting completion.

---

## Platform notes

- **Windows**: All four layers apply. Use `icacls` for Layer 2, `setx` for Layer 3.
- **macOS/Linux**: Layers 1a and 1b apply the same way. Layer 2 uses `chmod -R a-w` / `chmod -R u+w`. Layer 3 uses shell profile exports. Always note which layers were applied and which were skipped.
