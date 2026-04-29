# Day 7 Setup — Kids' PCs (Windows)

This is the morning preflight Andres runs at each kid's PC, **before** opening the Day 7 morning HTML. Goal: confirm in under 90 seconds that the kid's machine is ready, and that the kid will not hit an MCP failure mid-mission.

---

## 0. One-time setup (do this tonight, while sitting at each PC)

### 0a. Verify Nano Banana MCP config points at gemini-3.1-flash-image-preview

Open the Claude Code config or the per-project `.mcp.json` and confirm the Gemini Nano Banana entry has:

```
"GEMINI_IMAGE_ENDPOINT": "https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-image-preview:generateContent"
```

If it points at `gemini-2.5-flash-image-preview` or anything else, **edit the URL** and restart Claude Code. The 3.1 model is what tomorrow's activity assumes.

### 0b. Drop the graphic-novel skill onto the PC

The skill lives in this repo at `~/.claude/skills/graphic-novel/SKILL.md` (Mac path). Copy it onto each kid's PC at:

```
%USERPROFILE%\.claude\skills\graphic-novel\SKILL.md
```

PowerShell paste pattern (run on the PC):

```powershell
$skillDir = "$env:USERPROFILE\.claude\skills\graphic-novel"
New-Item -ItemType Directory -Path $skillDir -Force | Out-Null
# then paste the SKILL.md content into $skillDir\SKILL.md using your editor of choice
```

### 0c. Confirm `Documents\nano\` exists

```powershell
$nanoDir = "$env:USERPROFILE\Documents\nano"
New-Item -ItemType Directory -Path $nanoDir -Force | Out-Null
```

---

## 1. Morning-of paste — single Claude Code prompt

When the kid sits down tomorrow and opens Claude Code, **before** they open the Day 7 morning HTML, Andres pastes this one block. The output is a single line: GREEN or RED.

```
Run a Day 7 preflight and report a single one-line status.

Check 1 — Nano Banana 2 image MCP:
  Generate one tiny test image with whatever Nano Banana / Gemini image MCP is configured.
  Prompt: "A small painterly portrait of an elderly bearded man, NOT a children's book."
  Aspect ratio: 4:3.
  Save to: %USERPROFILE%\Documents\nano\preflight-test.png
  Then report which model name the MCP responded with. PASS only if the model name
  contains "gemini-3.1-flash-image-preview". If the MCP is missing, off, or pointing
  at an older endpoint, FAIL.

Check 2 — graphic-novel skill:
  List available skills. PASS if "graphic-novel" appears. FAIL if not.

Check 3 — nano directory writable:
  Confirm %USERPROFILE%\Documents\nano\ exists and is writable.
  PASS if Check 1's image actually wrote to disk there. FAIL otherwise.

Final output, EXACTLY one line, no extra prose:
  GREEN — all 3 checks passed. Ready for Day 7.
or
  RED — failures: [check N: <one-sentence reason>], [check M: <one-sentence reason>].
```

---

## 2. Reading the result

**GREEN.** Open the Day 7 morning HTML. Hand the kid the keyboard. Done.

**RED.** Andres comes over. The error line tells you exactly which check failed. Common fixes:

| Failure | Fix |
|---------|-----|
| Check 1 — wrong model name | Edit `.mcp.json` to point at `gemini-3.1-flash-image-preview:generateContent`. Restart Claude Code. Re-run preflight. |
| Check 1 — MCP not present | Restore `.mcp.json` and confirm `npx` runs without prompting. Re-run preflight. |
| Check 1 — 401/403 | API key missing or revoked. Pull a fresh key from Google AI Studio, paste into the env block, restart. |
| Check 2 — skill not found | Re-paste `SKILL.md` into `%USERPROFILE%\.claude\skills\graphic-novel\`. Restart Claude Code so it picks up the new skill. |
| Check 3 — directory missing | Run the PowerShell snippet from section 0c. |

After the fix, re-run the same preflight prompt. When it returns GREEN, hand the kid the keyboard.

---

## 3. Why this is one line

Kids don't read paragraphs while they're trying to start. The whole point of GREEN / RED is that the kid sees one line and either keeps going or yells for Andres. No interpretation required.

If you ever find yourself adding a fourth check, ask whether it really needs to be in the preflight — or whether it can live as a tip inside the morning HTML where the kid will encounter it in context.
