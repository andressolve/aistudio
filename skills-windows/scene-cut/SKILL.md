---
name: scene-cut
description: TRIGGER when the user wants to generate a short three-cut video scene (stitched into one MP4) of one character doing one thing in one environment. Drives gpt-image-2 (OpenAI) to build a character reference + environment reference, composes three storyboard frames anchored on those references, fans out fal.ai Veo 3.1 image-to-video calls in parallel with motion + ambient + optional dialogue, then ffmpeg-stitches into one scene. Designed for the Day 9 "The Scene" activity. DO NOT TRIGGER for single-shot talking-head quotes (use video-quote), text-to-video without start frames, or scenes longer than 3 cuts.
---

# scene-cut (Windows)

End-to-end pipeline for one short scene with three cuts. Windows port of the Mac skill — uses `curl.exe`, `python`, `start`, and Windows-style paths. Run from PowerShell (preferred) or `cmd.exe`.

1. The user names **one character**, **one environment**, **one action** the character does in that environment.
2. The skill writes three story beats from those inputs.
3. It generates a **character reference** and an **environment reference** with gpt-image-2.
4. It composes three **storyboard frames**, each anchored on both references so character and place stay locked across cuts.
5. It animates the three frames in parallel via Veo 3.1 image-to-video (with ambient sound and optional dialogue).
6. It stitches the three cuts into one MP4 (~24s) with ffmpeg.

## When to use

- The user wants a **three-cut narrative scene** of one character in one place doing one thing.
- Output is **one stitched MP4** (~24 seconds, three 8-second shots).
- The skill is permitted to add **ambient sound on every cut** and **dialogue on at most one cut** when the story warrants it.

Do NOT use for:
- Single talking-head quotes (`video-quote`).
- Two or more characters in dialogue.
- Text-to-video with no start frames.
- More than three cuts.

## Pre-flight (silent — fail loud, fix before talking to user)

Run from PowerShell:

```powershell
if (-not $env:FAL_KEY)        { throw "FAL_KEY not set. Set it in System Environment Variables." }
if (-not $env:OPENAI_API_KEY) { throw "OPENAI_API_KEY not set." }
Get-Command ffmpeg  -ErrorAction Stop | Out-Null
Get-Command curl.exe -ErrorAction Stop | Out-Null
Get-Command python  -ErrorAction Stop | Out-Null
```

Notes:
- **`curl.exe`** ships with Windows 10/11 — call it explicitly as `curl.exe` (PowerShell aliases `curl` to `Invoke-WebRequest`, which has different flags).
- **`python`** — if `python` isn't on PATH, try `py` (the Python launcher). Replace `python` with `py` throughout.
- **`ffmpeg`** — install via `winget install ffmpeg` or `choco install ffmpeg`. Reopen the shell so PATH refreshes.
- **Env vars** — set via System Properties → Environment Variables (User vars are fine). Reopen the shell after setting.

**Working directory.** Default: `$HOME\Documents\scenes\<slug>\`. Create if missing. Subfolders: `refs\`, `frames\`, `cuts\`.

```powershell
$slug = "<character-word>-<env-word>-<action-verb>"
$work = "$HOME\Documents\scenes\$slug"
New-Item -ItemType Directory -Force -Path "$work\refs","$work\frames","$work\cuts" | Out-Null
Set-Location $work
```

## Inputs to collect

Exactly three, in order:

1. **Character** — a noun phrase, 2–6 words.
2. **Environment** — a noun phrase, 2–6 words. Time of day / atmosphere if relevant.
3. **Action** — a verb phrase, present-progressive.

Slug: `<character-word>-<env-word>-<action-verb>` (e.g. `hobbit-woods-exploring`).

## Workflow

Eleven steps. Steps 3, 6, 9 fan out in parallel.

### Step 1 — Write the three beats

- **Beat 1 — Setup.** Character established in environment. One clear physical action.
- **Beat 2 — Turn.** The hinge. Notices, decides, discovers, commits.
- **Beat 3 — Payoff.** Consequence or reaction. New state.

Show the beats. User approves or names one to rewrite. After two rewrites of the same beat, suggest changing the input action — the story isn't there.

### Step 2 — Compose the reference prompts

**Character reference prompt:**

```
Full-body character reference sheet: <CHARACTER>. Neutral standing pose,
arms relaxed at sides, facing camera, slight 3/4 turn. Plain warm-grey
studio background, even soft frontal light. Cinematic photoreal, painterly
realism, NOT a cartoon. No props, no environment, no text.
```

**Environment reference prompt:**

```
Establishing landscape reference: <ENVIRONMENT>. Wide, atmospheric, no
human figures, no characters, no text. Strong sense of place: light source,
weather, foreground/midground/background depth. Cinematic photoreal,
painterly realism, NOT a cartoon.
```

Both call out **"cinematic photoreal, painterly realism, NOT a cartoon"** — that's the style lock that propagates downstream.

### Step 3 — Generate the two references in PARALLEL (gpt-image-2)

Use Start-Job to fire both at once. Each job runs a python script that builds JSON, hits the API, and decodes the base64 response to a PNG.

```powershell
$genScript = @'
import sys, os, json, base64, urllib.request

prompt, outpath = sys.argv[1], sys.argv[2]
body = json.dumps({
    "model": "gpt-image-2",
    "prompt": prompt,
    "n": 1,
    "size": "1536x1024",
    "quality": "high"
}).encode()
req = urllib.request.Request(
    "https://api.openai.com/v1/images/generations",
    data=body,
    headers={
        "Authorization": f"Bearer {os.environ['OPENAI_API_KEY']}",
        "Content-Type": "application/json",
    },
)
data = json.loads(urllib.request.urlopen(req, timeout=120).read())
b64 = data["data"][0]["b64_json"]
with open(outpath, "wb") as f:
    f.write(base64.b64decode(b64))
print(outpath, "OK")
'@
Set-Content -Path gen_image.py -Value $genScript -Encoding UTF8

$j1 = Start-Job { Set-Location $using:work; python gen_image.py $using:charPrompt "refs\character.png" }
$j2 = Start-Job { Set-Location $using:work; python gen_image.py $using:envPrompt  "refs\environment.png" }
Wait-Job $j1, $j2 | Out-Null
Receive-Job $j1, $j2
Remove-Job  $j1, $j2
```

**Fallback:** if `gpt-image-2` returns model-not-found, retry with `gpt-image-1`. Note the fallback to the user.

### Step 4 — Show the references to the user

**Open both files in the OS image viewer** so the user actually sees them — don't rely on the chat client's inline image rendering (it's inconsistent across Claude Code clients). On Windows:

```powershell
start refs\character.png
start refs\environment.png
```

`start` (alias for `Start-Process`) opens each file in its default app — Photos by default. Then ask the user to approve both, or name which one to regenerate. Iteration discipline:

- Regenerate only the one that's off.
- Keep the un-touched ref's prompt **verbatim** — it's the lock.
- If a face/proportion is wrong on the character ref, regenerate the character ref. Don't try to fix downstream.

Two regenerations max per reference. After that, propose a tighter character or environment phrase.

### Step 5 — Compose the three storyboard prompts

Each storyboard frame is composed with the character reference + environment reference passed in as input images. Per-beat prompt:

```
A single storyboard frame for cut <N>. <Beat description, present tense.>
The character on the left/right/center of frame. Wide cinematic
landscape composition, 16:9 framing, full body or medium shot per beat,
depth and ambient detail, no text in image.

Style: cinematic photoreal, painterly realism, NOT a cartoon. Match the
character's appearance from the character reference and the environment
from the environment reference.
```

Per beat:
- **Cut 1 (Setup):** wide shot, character small in landscape.
- **Cut 2 (Turn):** medium shot, focused on the hinge action.
- **Cut 3 (Payoff):** close or medium shot on reaction.

Vary distance and angle — that's what makes it feel cut, not a slideshow.

### Step 6 — Generate the three storyboard frames in PARALLEL (gpt-image-2 edits)

Use `curl.exe` for multipart upload. Fire three jobs in parallel:

```powershell
$editJob = {
    param($work, $beatPrompt, $outName)
    Set-Location $work
    & curl.exe -s "https://api.openai.com/v1/images/edits" `
        -H "Authorization: Bearer $env:OPENAI_API_KEY" `
        -F "model=gpt-image-2" `
        -F "image[]=@refs/character.png" `
        -F "image[]=@refs/environment.png" `
        -F "prompt=$beatPrompt" `
        -F "size=1536x1024" `
        -F "quality=high" `
        -F "n=1" `
        -o "frames\_$outName.json"
    python -c "import json,base64,sys; d=json.load(open('frames/_$outName.json')); open('frames/$outName','wb').write(base64.b64decode(d['data'][0]['b64_json']))"
}

$f1 = Start-Job $editJob -ArgumentList $work, $beat1Prompt, "frame_1.png"
$f2 = Start-Job $editJob -ArgumentList $work, $beat2Prompt, "frame_2.png"
$f3 = Start-Job $editJob -ArgumentList $work, $beat3Prompt, "frame_3.png"
Wait-Job $f1,$f2,$f3 | Out-Null
Receive-Job $f1,$f2,$f3
Remove-Job  $f1,$f2,$f3
```

**Fallback if the API rejects multi-image input:**
Pass only the character reference via `image[]=` and describe the environment richly in the prompt. Character lock matters more than environment lock.

### Step 7 — Show the three frames to the user

**Open all three files in the OS image viewer** in beat order:

```powershell
start frames\frame_1.png
start frames\frame_2.png
start frames\frame_3.png
```

Ask the user to approve all three or name which to regenerate. Same discipline as Step 4. After two regenerations of the same frame, propose rewriting the beat or the action.

### Step 8 — Compose the three Veo prompts

Each Veo prompt is 100–250 chars:

1. **Subject + position** — *"The hobbit in the foreground"*.
2. **One motion** — what changes between t=0 and t=8s. One verb chain that advances the beat.
3. **Ambient bed** — always. Match audio to visible motion.
4. **Dialogue or SFX** — optional, at most on **one** cut. ≤ 12 words. Format: `says quietly: "I shouldn't be here."`
5. **Style guard** — *"Cinematic, photoreal, subtle natural acting. NOT a cartoon."*

### Step 9 — Submit all three cuts to Veo 3.1 i2v in PARALLEL

For each frame, first upload to fal storage, then submit. Use a python helper to keep this short:

```python
# fal_submit.py
import os, sys, json, urllib.request

FAL_KEY = os.environ["FAL_KEY"]
H = {"Authorization": f"Key {FAL_KEY}", "Content-Type": "application/json"}

def initiate(file_name):
    body = json.dumps({"file_name": file_name, "content_type": "image/png"}).encode()
    req = urllib.request.Request("https://rest.alpha.fal.ai/storage/upload/initiate", data=body, headers=H)
    return json.loads(urllib.request.urlopen(req, timeout=30).read())

def put_bytes(upload_url, path):
    with open(path, "rb") as f:
        data = f.read()
    req = urllib.request.Request(upload_url, data=data, method="PUT", headers={"Content-Type": "image/png"})
    urllib.request.urlopen(req, timeout=120).read()

def submit(prompt, image_url):
    body = json.dumps({
        "prompt": prompt,
        "image_url": image_url,
        "duration": "8s",
        "resolution": "720p",
        "aspect_ratio": "16:9",
        "generate_audio": True,
    }).encode()
    req = urllib.request.Request("https://queue.fal.run/fal-ai/veo3.1/image-to-video", data=body, headers=H)
    return json.loads(urllib.request.urlopen(req, timeout=60).read())

if __name__ == "__main__":
    frame_path, prompt, out_json = sys.argv[1], sys.argv[2], sys.argv[3]
    info = initiate(os.path.basename(frame_path))
    put_bytes(info["upload_url"], frame_path)
    sub = submit(prompt, info["file_url"])
    with open(out_json, "w") as f:
        json.dump(sub, f)
    print(sub["request_id"])
```

Submit three in parallel:

```powershell
$v1 = Start-Job { Set-Location $using:work; python fal_submit.py "frames\frame_1.png" $using:veo1 "cuts\sub_1.json" }
$v2 = Start-Job { Set-Location $using:work; python fal_submit.py "frames\frame_2.png" $using:veo2 "cuts\sub_2.json" }
$v3 = Start-Job { Set-Location $using:work; python fal_submit.py "frames\frame_3.png" $using:veo3 "cuts\sub_3.json" }
Wait-Job $v1,$v2,$v3 | Out-Null
Receive-Job $v1,$v2,$v3
Remove-Job  $v1,$v2,$v3
```

Slug exactly: `fal-ai/veo3.1/image-to-video`. Wrong slug → fast COMPLETED + 404 on result fetch.

### Step 10 — Poll all three in parallel

Cadence: every 15s. Print only on status change. Expected wall time ~75–90s for the slowest cut. Surface any cut that exceeds 3 minutes.

```python
# fal_poll.py
import os, sys, json, time, urllib.request

FAL_KEY = os.environ["FAL_KEY"]
H = {"Authorization": f"Key {FAL_KEY}"}

sub = json.load(open(sys.argv[1]))
last_status = None
while True:
    req = urllib.request.Request(sub["status_url"], headers=H)
    s = json.loads(urllib.request.urlopen(req, timeout=30).read())
    if s.get("status") != last_status:
        print(sys.argv[1], s.get("status"))
        last_status = s.get("status")
    if s.get("status") == "COMPLETED":
        req = urllib.request.Request(sub["response_url"], headers=H)
        r = json.loads(urllib.request.urlopen(req, timeout=30).read())
        json.dump(r, open(sys.argv[2], "w"))
        break
    time.sleep(15)
```

### Step 11 — Download + stitch + save

Each cut's response: `{"video": {"url": "..."}}`. Download to `cuts\cut_1.mp4`, `cut_2.mp4`, `cut_3.mp4`:

```powershell
foreach ($n in 1..3) {
    $resp = Get-Content "cuts\resp_$n.json" | ConvertFrom-Json
    Invoke-WebRequest -Uri $resp.video.url -OutFile "cuts\cut_$n.mp4"
}
```

Build concat manifest and stitch with re-encode:

```powershell
@"
file 'cuts/cut_1.mp4'
file 'cuts/cut_2.mp4'
file 'cuts/cut_3.mp4'
"@ | Set-Content -Path concat.txt -Encoding ASCII

ffmpeg -y -f concat -safe 0 -i concat.txt `
    -c:v libx264 -preset medium -crf 20 `
    -c:a aac -b:a 192k `
    -pix_fmt yuv420p `
    scene.mp4
```

Final output: `$HOME\Documents\scenes\<slug>\scene.mp4`. Print the absolute path.

## Hard rules

- **One character. One environment. One action.** More than that → different skill.
- **Three cuts. 8s each.** Total ~24s.
- **Two references generated and approved before any storyboard work.**
- **All three storyboard frames composed with BOTH references attached.** Fallback: character ref only.
- **Vary shot distance across the three cuts.** Wide → medium → close.
- **Dialogue on at most one cut, ≤ 12 words.**
- **Ambient on every cut.**
- **Re-encode the stitch.** `-c copy` will bite you on Veo output variance.
- **Parallel fan-out** at Steps 3, 6, 9.
- **Always `start` images via the OS for user review** (Steps 4 and 7). Inline rendering in the chat client is inconsistent and Claude can't tell whether the image actually displayed. `start file.png` is deterministic.

## Common failure modes

| Symptom | Cause | Fix |
|---|---|---|
| `curl: The term 'curl' is not recognized` or wrong flags | PowerShell aliased `curl` to `Invoke-WebRequest` | Always invoke as `curl.exe`, never bare `curl` |
| `python` not found | Only `py` launcher installed | Replace `python` with `py` throughout |
| `ffmpeg` not on PATH after install | Shell opened before install | Close + reopen PowerShell |
| Env var not set | Set in current shell only, didn't survive restart | Set via System Properties → Environment Variables, then reopen shell |
| Character looks different between cuts | Storyboard frames not anchored on character ref | Re-run frames with original char ref attached |
| Environment shifts | Env ref not attached to edits | Pass env ref to all three edits |
| Scene feels like a slideshow | Three cuts at same camera distance | Re-do Step 8 with wide → medium → close |
| Multi-image edit endpoint rejects request | Model variant doesn't accept `image[]=` | Fall back to character-ref-only |
| Veo fast-COMPLETED with no result | Wrong slug | Verify exactly `fal-ai/veo3.1/image-to-video` |
| Scene MP4 won't play in Windows Media Player | Pixel format mismatch | Confirm `-pix_fmt yuv420p` |

## What this skill must NOT do

- Generate without character + environment references.
- Skip the multi-image edit step.
- Compose all three storyboard frames at the same shot distance.
- Include dialogue on more than one cut, or longer than 12 words.
- Stitch with `-c copy`. Always re-encode.
- Use bare `curl` (PowerShell alias confusion) — always `curl.exe`.
- Handle two characters. Different shape.
