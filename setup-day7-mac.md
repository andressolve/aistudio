# Day 7 Setup — Andres's Mac (Pre-Flight)

Run this checklist tonight, before tomorrow's homeschool. Goal: confirm Nano Banana 2 image generation works, the graphic-novel skill is callable, and at least one web video tool (Kling.ai primary, Veo 3 fallback) returns a usable lip-synced clip from a known-good prompt.

If any of these fails tonight, you find out tonight — not at 9 AM tomorrow.

---

## 1. Nano Banana 2 — image generation MCP

The aistudio repo's `.mcp.json` does not currently include the Gemini MCP. Add it. The same key already lives in `~/Documents/ads_manager/.mcp.json` if you want to copy from there.

**Add this block to `/Users/andresrodriguez/Documents/aistudio/.mcp.json` under `mcpServers`:**

```json
"gemini-nanobanana-mcp": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "gemini-nanobanana-mcp@latest"],
  "env": {
    "GEMINI_API_KEY": "<copy from ads_manager/.mcp.json>",
    "GEMINI_IMAGE_ENDPOINT": "https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-image-preview:generateContent"
  }
}
```

**Critical:** the endpoint URL must end in `gemini-3.1-flash-image-preview:generateContent`. If a previous setup pointed at `gemini-2.5-flash-image-preview`, replace it. The `3.1` model is the one with native multi-reference, better text rendering, and web search grounding.

After editing, restart Claude Code in the aistudio working directory so the MCP reloads.

### 1a. Smoke test the image MCP

In Claude Code (aistudio dir), paste:

> Generate one 4:3 landscape image with Nano Banana 2: "Reference sheet for Leonardo da Vinci, age 60, long silver beard, dark velvet cloak, painterly Renaissance graphic novel style. 3/4 portrait pose, neutral expression, plain warm-toned background. NOT a children's book, realistic adult proportions. No text, no labels." Save to /tmp/nb2-smoke-davinci.png. Then tell me which model name responded.

**Pass criteria:**
- File exists at `/tmp/nb2-smoke-davinci.png`.
- Image shows a recognizable Da Vinci-ish older man, painterly, no cartoon features.
- The model name reported back includes `gemini-3.1-flash-image-preview`.
- No "model not found" or "endpoint mismatch" errors.

**If it fails:**
- 404 on endpoint → URL still pointing at an older model. Edit `.mcp.json` again.
- 401/403 → API key missing or revoked. Pull a fresh one from Google AI Studio.
- MCP doesn't appear → restart Claude Code after editing `.mcp.json`.

### 1b. Multi-reference test (matters tomorrow)

Same session:

> Generate a second 2:3 vertical image: "Leonardo da Vinci in his Milan workshop, dissecting a cadaver by candlelight, 1490s. Painterly oil-on-panel texture, NOT a children's book." Use the previous image (/tmp/nb2-smoke-davinci.png) as a character reference. Save to /tmp/nb2-smoke-page.png.

**Pass criteria:** the man in the new image clearly resembles the man in the reference sheet — same face, same beard, same age. If he's 30 instead of 60, multi-reference is silently broken; debug before moving on.

---

## 2. Graphic-novel skill

The skill lives at `/Users/andresrodriguez/.claude/skills/graphic-novel/SKILL.md`. Confirm it loads:

> List your available skills. Confirm graphic-novel is present.

**Pass criteria:** Claude Code reports the skill exists and can summarize its trigger condition. If it can't, the skill file is missing or malformed — re-read this repo's setup or the skill file directly.

---

## 3. Kling.ai — primary video tool

Open https://kling.ai in your browser. Log in.

**Known-good test prompt** (paste into Kling.ai, no reference frame attached for this smoke test):

> Leonardo da Vinci, age 60, long silver beard, dark velvet cloak, in his Milan workshop on a winter afternoon, 1495. Soft northern light through tall leaded windows. Wooden anatomical sketches and dissection tools on the table behind him. Medium shot, slightly low angle. He looks up from an open notebook, sets down his pen, and speaks directly to camera: "Obstacles cannot crush me; every obstacle yields to stern resolve." Photorealistic, oil-painting cinematography, NOT a cartoon, NOT a children's book illustration. Native lip sync. 6 seconds.

**Pass criteria:**
- Generation completes in under 5 minutes.
- Mouth syncs to the spoken audio (not perfectly, but close).
- The figure looks like a 60-year-old painterly Da Vinci, not a cartoon.
- The quote is intelligible.

**Watch for:**
- Rate limits on the free tier. Confirm you're on a paid plan, or know your remaining daily credits before tomorrow afternoon. Both kids will run at least 1–2 generations each (3 if Mission 3's recut takes a second iteration), so budget for ~6 generations as a safe margin.
- Latin or archaic phrasing in the quote → robotic audio. The Da Vinci line above is fine.
- Lip-sync offset of more than ~0.5s → flag it; that's the most common Kling failure tomorrow.

---

## 4. Veo 3 — fallback

Open Veo 3 in your browser (Google's video studio).

**Known-good test prompt:**

> A photorealistic medium shot, 35mm, slightly low angle. Leonardo da Vinci, age 60, silver beard, dark velvet cloak, sits at a wooden table in his Milan workshop, winter 1495. Soft northern light from tall leaded windows. Anatomical sketches and dissection tools rest on the table behind him. He looks up from an open notebook, sets down his pen, turns to camera, and says: "Obstacles cannot crush me; every obstacle yields to stern resolve." Oil-painting cinematography. Lip sync to spoken audio. 6 seconds.

**Pass criteria:** generates a usable clip even if lip sync is weaker than Kling. This is the fallback only — confirm it works once, save the URL bookmarked, move on.

---

## 5. Quick green/red summary

When the four checks above pass, write down on paper or in a note:

```
Day 7 Mac preflight — DATE: 2026-04-28
[ ] 1a Nano Banana 2 single image — gemini-3.1-flash-image-preview
[ ] 1b Multi-reference page — character match
[ ] 2  graphic-novel skill loads
[ ] 3  Kling.ai web — known-good prompt → usable MP4
[ ] 4  Veo 3 web — known-good prompt → usable MP4 (fallback)
```

If all four pass, the Mac is ready. If any fail, **fix tonight, not tomorrow morning.**

---

## 6. Re-run note

If anything drifts overnight (rare but possible — npx cache eviction, MCP restart, browser logout), re-run the same prompts in this doc tomorrow morning before the kids wake up. They are the canonical known-goods for this day.
