# Day 6 Setup — Meshy + OpenSCAD Pipeline

Run this in a Claude Code session before the kids start Day 6.
Goal: verify that Meshy text-to-3D and OpenSCAD MCP are both working.

---

## 1. Verify Meshy MCP Server

The `.mcp.json` in this directory is already configured:

```json
{
  "mcpServers": {
    "meshy": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "meshy-mcp-server"],
      "env": {
        "MESHY_API_KEY": "msy_YSb7725pl5WhaRK1414bqhwABHrkOuhyC08s"
      }
    }
  }
}
```

**Test it:** Ask Claude Code to run one of the Meshy tools (e.g. list recent generations). If the MCP tools appear and respond, you're good.

If the MCP server doesn't connect, use the **curl fallback** below — it works the same way, just requires Claude Code to make HTTP calls instead of MCP calls.

### Curl Fallback (if MCP doesn't work)

Tell Claude Code:

> The Meshy API key is `msy_YSb7725pl5WhaRK1414bqhwABHrkOuhyC08s`. Use curl to call the Meshy API directly at `https://api.meshy.ai`. Auth header: `Authorization: Bearer <key>`.

Key endpoints:
- `POST /openapi/v2/text-to-3d` — create a generation task
- `GET /openapi/v2/text-to-3d/{task_id}` — poll task status
- `POST /openapi/v1/remesh` — remesh to STL with target polycount and size

The Day 6 activity say boxes work with either approach — they describe what to ask Claude Code to do, not which specific tool to call.

---

## 2. Verify OpenSCAD MCP

OpenSCAD MCP should already be configured on the kids' machines from Day 5. Quick check:

> Ask Claude Code: "Check if OpenSCAD is installed and working."

It should return version info. If not, install OpenSCAD from https://openscad.org/downloads.html.

---

## 3. Verify Node.js / npx

The Meshy MCP server runs via `npx`. Make sure Node.js is installed:

```bash
node --version   # should be 18+
npx --version
```

If missing, install from https://nodejs.org/.

---

## 4. Quick Smoke Test

Run this sequence to confirm the full pipeline works:

1. **Generate a test model:**
   > "Use Meshy to generate a 3D model of a small cube with rounded edges. Use preview mode."

2. **Wait for it** (~60 seconds). Claude Code should poll until complete.

3. **Remesh to STL:**
   > "Remesh the result to STL format, target 30000 polygons, 5 cm tall."

   Note: Meshy's `resize_height` parameter is in **meters** (so 5 cm = 0.05). Claude Code should handle this conversion, but if using curl directly, pass `"resize_height": 0.05`.

4. **Download the STL** to a known location.

5. **Verify the file:**
   > "Analyze this STL file — is it watertight? How many triangles?"

If all five steps work, Day 6 is ready to go.

---

## 5. Credit Check

Meshy-6 credit costs (current model):
- Preview: 20 credits (~60-70 sec)
- Refine (texture): 10 credits (~60 sec)
- Remesh: 5 credits

**Full pipeline per model: ~35 credits** (preview + refine + remesh).

Per kid budget:
- Mission 1 (generate + refine): 30 credits
- Mission 2 (reroll, optional): 30 credits
- Mission 3 (remesh to STL): 5 credits
- **Sebastian total: 35-65 credits**
- **Francisco total: 35-100 credits** (boss mission adds another full pipeline)

Check balance at https://app.meshy.ai or ask Claude Code to check via API. Make sure you have at least **200 credits** loaded before the session (covers both kids with margin for failed attempts).

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| MCP tools don't appear | Restart Claude Code in this directory. Check `.mcp.json` is in the working directory. |
| `npx` hangs or fails | Run `npx -y meshy-mcp-server` manually in terminal to see errors. May need `npm cache clean --force`. |
| API returns 401 | API key expired or wrong. Check https://app.meshy.ai/settings/api-keys. |
| STL download fails | Meshy returns a URL — use `curl -L -o file.stl <url>` to download. |
| OpenSCAD not found | Install from openscad.org. macOS: may need to run once manually to clear Gatekeeper. |
