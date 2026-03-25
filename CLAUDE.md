# AI Studio

Multi-day structured activity series teaching Francisco (~10) and Sebastian (~8) to direct AI tools through creative projects. Built by Andres (dad). See `CLAUDEf.md` for full family context and educational philosophy.

## What This Is

Self-contained HTML activity guides — dark-themed, polished, interactive — that the boys open in a browser and work through. Each "day" teaches a new skill layer, building on the last. No backend, no dependencies — just single HTML files with embedded CSS/JS.

## The Activities

| File | Day | Title | Audience | Core Skill |
|------|-----|-------|----------|------------|
| `francisco-ai-studio (1).html` | 1 | AI Studio | Francisco | Directing AI: graphic novels, image prompting, video shot briefs, critique, mini-trailer |
| `sebastian-ai-studio-day2.html` | 2 | AI Studio Day 2 | Sebastian | Word power, style control, video reshoots, two-shot sequencing. Split into Session A + B with break |
| `ai-studio-day3-build-your-guide.html` | 3 | Build Your Own Guide | Both | Full-project direction: pick topic, direct Claude Code to build illustrated HTML guide, add personal twist |
| `francisco-ai-studio-day4.html` | 4 | Build Your Own Game | Francisco | Game design doc (See/Do/Win-Lose), first playable, art pass with Nano Banana, playtesting with fresh eyes |
| `ai-studio-day5-the-forge.html` | 5 | The Forge | Both | 3D modeling (Three.js), STL export, 3D printing. Code-to-physical-object pipeline |
| `index.html` | — | Hub | — | Index page linking to all 5 days |

## Design System

All activities share a consistent visual language:

- **Fonts:** Syne (headings), DM Mono (labels/code), DM Sans (body) — loaded from Google Fonts
- **Color palette:** Dark backgrounds (#0a-#0e range), orange/teal/purple accents, muted text
- **Components:** Mission cards (expandable accordion), prompt boxes (orange left border), "say boxes" (purple left border for Claude Code instructions), step lists (numbered circles), tip boxes, example panels (toggle-reveal), XP skill pips, progress bars
- **Patterns:** Each day has a recap of prior skills, a "rule" card (framing metaphor), missions with Mark Complete buttons, and a cumulative skills display at the bottom
- **State:** In-memory JS only (no localStorage). Progress resets on page refresh. That's intentional — they're activity guides, not apps.

## Tone and Content Rules

From `CLAUDEf.md` — these are non-negotiable:

- **Treat them as capable.** No condescension. No "wow isn't this FUN!" energy.
- **Formal intellectual tone.** Substance over flash. The content should feel like entering a tradition of serious thinking.
- **Real vocabulary.** Don't simplify terminology — explain it.
- **No gamification.** No badges, no "Great job!" popups. The work itself is the reward.
- **Discovery-based.** Show the "why" before naming the concept.

The skill pips and progress bars are the ONE concession to gamification — and they're understated (monospace labels, muted colors, no celebrations).

## Francisco vs Sebastian

- **Francisco's track:** More missions per day, longer sessions, more advanced concepts (shot briefs, full trailers, game design docs, playtesting theory). Treats him as a junior creative professional.
- **Sebastian's track:** Fewer missions, shorter sessions, split into Session A + B with explicit breaks. More scaffolding and simpler entry points, but still intellectually honest. Sebastian is gifted (99th percentile) — the simplification is about session length and task scope, not cognitive level.

## Tools the Kids Use

- **Claude Code** — primary AI tool, runs on their machines
- **Nano Banana Pro MCP** — image generation (Google Gemini), configured on both machines
- **Kling / Veo 3** — video generation (external, manual)
- **Claude Chat / ChatGPT** — for image generation and prompt help
- **Three.js 3D Viewer MCP** — for Day 5 3D modeling
- **Slicer software** — for Day 5 3D printing pipeline

## Activity Design Principles

These emerged from iterating on the activities — keep them in mind when building or revising:

- **Don't cut steps that are functionally necessary to the outcome.** The instinct to simplify is good, but some steps exist because the output is bad without them. Day 5's "Teach the Machine" research step is the key example: cutting it to reduce mission count caused Claude Code to produce unprintable models — non-watertight meshes with separate floating pieces, thin walls that snap, overhangs that turn into spaghetti. The research step is what teaches Claude Code the specific constraints (watertight geometry, 45° overhang rule, 2mm minimum wall thickness, flat base) before it builds. Without that front-loaded domain knowledge, the AI defaults to "looks cool on screen" geometry that fails in the real world. If a step exists because the AI needs domain knowledge to produce good output, it stays — even if it makes the activity longer.
- **Project suggestions must connect to the audience.** Generic "cool engineering demos" don't land. Every suggestion should pass the test: would THIS kid get fired up about it? War helmets, custom chess pieces, fortress towers > parametric vases, gear trains.
- **Simpler ≠ better when the domain has real constraints.** For purely creative activities (graphic novels, guides), fewer missions and lighter structure works. For activities with engineering constraints (3D printing, anything with physical output), the activity needs enough structure to prevent failure. A failed 3D print wastes real time and filament. A mediocre graphic novel is just a redo. Match the rigor to the stakes.
- **The "rule card" framing metaphor is optional.** Day 1 and Day 4 use it well (Director's Rule, Game Designer's Rule). Day 5 uses it well too (the 45° gravity rule). But don't force one where it doesn't help.

## How to Build New Activities

1. Single self-contained HTML file. All CSS inline in `<style>`, all JS inline in `<script>`. No external deps except Google Fonts.
2. Follow the existing component patterns — don't invent new UI primitives.
3. Start with a header (eyebrow label, big Syne title, subtitle), then a recap card. Add a "rule" card only if the framing metaphor genuinely helps.
4. Structure missions as numbered accordion cards. Each mission needs: tag, title, time estimate badge, body with explanation, a prompt/say box, steps list, tool chips, and a Mark Complete button.
5. Include "SEE AN EXAMPLE" toggle panels — kids need concrete examples, not just instructions. Show weak vs strong examples side by side where possible.
6. End with XP pips (today's skills + cumulative from all prior days).
7. Add the new day to `index.html`.
8. Keep the director metaphor: the kid is the director, AI is the crew. Every mission reinforces that the kid's job is to describe, critique, and iterate — not to do the mechanical work themselves.
