# Windows skills for AI Studio

Windows-flavored ports of Mac skills. Sync these to each kid's PC.

## Install on a kid's PC

1. **Copy the skill folder** (e.g. `scene-cut/`) to:
   ```
   %USERPROFILE%\.claude\skills\scene-cut\
   ```
   So you end up with `%USERPROFILE%\.claude\skills\scene-cut\SKILL.md`.

2. **Install dependencies** (PowerShell, run once per machine):
   ```powershell
   winget install ffmpeg
   winget install Python.Python.3.12
   ```
   Then **close and reopen PowerShell** so PATH refreshes.

3. **Set API keys as User Environment Variables.**
   System Properties → Advanced → Environment Variables → New (under User variables):
   - `FAL_KEY` = `<your fal key>`
   - `OPENAI_API_KEY` = `<your openai key>`

   Close and reopen any open shells / Claude Code so the new env is picked up.

4. **Verify.** Open a fresh PowerShell:
   ```powershell
   $env:FAL_KEY.Length        # should print a non-zero number
   $env:OPENAI_API_KEY.Length # should print a non-zero number
   ffmpeg -version            # should print ffmpeg version
   python --version           # should print Python 3.x
   curl.exe --version         # should print curl version
   ```

5. **Restart Claude Code on the PC.** Skills are picked up at startup.

## Sync method

Easiest, low-tech: USB drive or OneDrive copy. The folder is small.

If you want the kids' PCs to track this repo, clone it on each PC, then symlink:
```powershell
New-Item -ItemType SymbolicLink `
  -Path "$HOME\.claude\skills\scene-cut" `
  -Target "<path-to-repo>\skills-windows\scene-cut"
```
(Symlinks require admin or Developer Mode enabled.)

## Skills available

| Skill | Description |
|---|---|
| `scene-cut/` | Day 9 — three-cut video scene via gpt-image-2 + Veo 3.1. |
