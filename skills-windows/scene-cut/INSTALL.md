# Install scene-cut skill on this Windows PC

Claude Code: follow these steps in order. Ask the user for anything missing. Do not skip steps.

## Step 1 — Copy the skill into place

Copy `SKILL.md` (sitting next to this INSTALL.md in the same folder) to:

```
%USERPROFILE%\.claude\skills\scene-cut\SKILL.md
```

Create the destination folder if it doesn't exist. Verify the file is there after copy.

## Step 2 — Install dependencies

Run these in PowerShell. If a tool is already installed, skip it (check with `Get-Command <tool>`):

```powershell
winget install --id Gyan.FFmpeg -e --accept-source-agreements --accept-package-agreements
winget install --id Python.Python.3.12 -e --accept-source-agreements --accept-package-agreements
```

`curl.exe` is built into Windows 10/11 — don't install anything for it. Just verify with `Get-Command curl.exe`.

After install, PATH may not refresh in the current shell. Open a NEW PowerShell window for verification in Step 4.

## Step 3 — Set the two API keys as User environment variables

Ask the user for these two values:

- `FAL_KEY` — fal.ai API key (looks like `<uuid>:<hex>`)
- `OPENAI_API_KEY` — OpenAI API key (starts with `sk-`)

Then set them persistently using `setx`:

```powershell
setx FAL_KEY "<the value the user gave>"
setx OPENAI_API_KEY "<the value the user gave>"
```

`setx` writes to the User environment scope and persists across reboots. It does NOT update the current shell — that's fine, Step 4 opens a new one.

## Step 4 — Verify in a fresh shell

Tell the user: "Close this PowerShell window and open a NEW one, then run these commands. Paste the output back."

```powershell
Test-Path "$env:USERPROFILE\.claude\skills\scene-cut\SKILL.md"
$env:FAL_KEY.Length
$env:OPENAI_API_KEY.Length
ffmpeg -version | Select-Object -First 1
python --version
curl.exe --version | Select-Object -First 1
```

Expected:
- `True`
- non-zero number (~69)
- non-zero number (~50+)
- a line starting with `ffmpeg version`
- `Python 3.x.x`
- a line starting with `curl `

If any of those fail, fix that specific thing before proceeding.

## Step 5 — Restart Claude Code

Tell the user to close Claude Code completely and reopen it so the new skill is picked up. Skills are loaded at startup.

## Done

The skill is now installed. The user can trigger it by asking Claude Code for a three-cut video scene.
