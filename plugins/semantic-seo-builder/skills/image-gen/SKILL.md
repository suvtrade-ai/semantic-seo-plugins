---
name: image-gen
description: Context-aware AI image generation for local business websites using the Kie API (Nano Banana / Google Gemini Imagen). Reads page content and structure, identifies every section that needs an image, generates unique section-appropriate images, compresses WebP to under 100KB, and returns mobile-optimised placement HTML with correct alt text and semantic filenames. Works for any page type — service pages, blog articles, about sections, process sections, fabric/material sections, etc. Use when user says "generate images", "add images to this page", "create hero image", "images for my site", "/image-gen", or when weekly-pipeline or content-brief needs images after writing a page.
tools:
  - Read
  - Write
  - Bash
---

# Image Generation Skill — `/image-gen`

Context-aware AI image generation. Reads any page (HTML, markdown, or brief), maps every section that needs an image, generates a **unique and relevant** image for each section, compresses to under 100KB, and returns mobile-optimised placement HTML.

---

## Critical rules (read before generating anything)

**UNIQUE images per section.** Every section on every page gets a distinct image. Never reuse the same image across two sections or two pages. The image map (Step 1) tracks all generated filenames — if a filename already exists, invent a different angle, framing, or detail rather than reusing it.

**REALISTIC photography aesthetic.** Prompts must describe real-world photography, not AI illustration. Use terms like: "DSLR editorial photography", "35mm film grain", "shallow depth of field f/1.8", "natural window light", "warm golden hour". Never use: "digital art", "illustration", "render", "3D", "artistic", "painterly", "glowing", "fantasy".

**Confirm before generating.** When called after content is written, always ask: "Ready to generate [N] images for [page name]. Here's what I'll create: [list]. Shall I proceed?" Only start firing API jobs after confirmation.

**WebP compression target: under 100KB per image.** Use `-quality 72` for photos (not 85). After conversion check file size and re-compress at lower quality if over 100KB.

**Mobile-first HTML.** All image HTML uses `srcset`, `sizes`, and responsive CSS. No fixed pixel heights on mobile. See Step 6 for templates.

---

## Step 1 — Read and map the page

Read the full page. Extract every section heading and content summary.

For each section, determine:
- Does it need an image?
- What type of image (see table below)?
- What is the primary entity of this section?
- What filename will this image get?

Track all planned filenames in a set. If a filename was already used anywhere in this session, change the qualifier (e.g. `detail` → `closeup`, `wide` → `entrance`, `interior` → `workbench`).

| Section type | Image style | Do NOT use |
|---|---|---|
| Hero / page header | Wide cinematic, business exterior or hero garment on form | Generic studio backdrop |
| About / story | Workshop interior, tools, warm ambient light | Empty room, stock office |
| Service section | The specific garment type or process for THIS service | Generic suits from another service |
| Process / how-it-works | Hands performing the exact step described | Finished garment (save for service) |
| Fabric / material | Flat lay of the specific fabric category mentioned | Random colourful swatches |
| Wedding / occasion | Lifestyle — wedding suit on hanger, chapel corridor | Generic white dress |
| Location / find us | Exterior of a boutique or mall corridor, tropical setting | Indoor only |
| Blog article hero | Scene that directly represents the article topic | Unrelated luxury lifestyle |
| CTA / contact | Workshop entrance, inviting doorway | Abstract or graphic |

**Sections that NEVER need images (skip):**
- Pure text FAQ
- Pricing tables
- Navigation / footer / breadcrumbs
- Testimonial quote blocks (text is the content)
- Icon-only feature grids

Output the image map silently (show only on request):

```
Page: services/bespoke-suits.html

Section: Hero
  Type: Wide cinematic
  Entity: bespoke suit on dress form, luxury tailor workshop
  Filename: bespoke-suit-dress-form-hero-[business-slug].webp
  Alt: Bespoke suit on dress form at [Business Name], [City]

Section: Our Process
  Type: Hands / craft action
  Entity: tailor pinning suit jacket shoulder seam
  Filename: tailor-pinning-suit-shoulder-seam-[business-slug].webp
  Alt: Expert tailor pinning bespoke suit shoulder at [Business Name]

Section: Fabric Selection
  Type: Flat lay
  Entity: premium wool suiting fabric swatches dark tones
  Filename: premium-wool-suiting-fabric-swatches-[business-slug].webp
  Alt: Premium wool and linen fabric swatches for bespoke suits at [Business Name]
```

---

## Step 2 — Check existing images

Before generating, check the `images/` directory. If a filename already exists, do not regenerate it — reuse it only if the entity and context are genuinely the same section. Otherwise, create a differentiated filename.

```bash
ls images/*.webp 2>/dev/null && echo "---existing images above---" || echo "No existing images"
```

If you find an image like `tailor-workshop-interior.webp` already used on the homepage, do NOT use it again on the About page. Instead generate `tailor-workshop-workbench-closeup.webp` or `tailor-atelier-interior-detail.webp`.

---

## Step 3 — Build realistic photography prompts

### Rules for all prompts
- Lead with the photography format: "DSLR editorial photography", "35mm lifestyle photography", "macro close-up photography"
- Specify real lighting: "soft window light", "warm golden hour side lighting", "diffused overcast natural light", "warm interior tungsten light"
- Specify depth: "shallow depth of field f/1.8", "sharp across the frame f/8"
- End every prompt with: "No text. No watermarks. No people's faces. Photorealistic."
- Never use words: render, illustration, 3D, digital art, artistic, stylized, glowing, fantasy, dreamlike, vibrant

### Hero images (wide)
```
DSLR editorial photography of [specific entity — garment type on dress form / boutique exterior].
[Warm golden side lighting / soft window light]. [Setting detail — dark mahogany shelving /
tropical foliage visible through window]. [Mood — luxury, refined, quiet].
Shot on 35mm, shallow depth of field. No text. No watermarks. No faces. Photorealistic.
```

### Service section images
```
DSLR editorial close-up photography of [exact garment or service — not "a suit" but "a navy
herringbone two-piece suit with peak lapels"]. Soft studio window light. White or dark
neutral background. Shallow depth of field f/2.8. Luxury fashion editorial aesthetic.
No text. No watermarks. No faces. Photorealistic.
```

### About / interior images
```
DSLR interior photography of a [business type] workshop. [Warm tungsten / soft window light].
[Specific interior elements — tailor's cutting table, fabric bolts on shelves, dress forms,
hand tools arranged]. Quiet, refined, crafted atmosphere. 35mm, natural grain.
No people. No text. No watermarks. Photorealistic.
```

### Process / hands images
```
DSLR macro close-up photography of [specific craft action — "a tailor's hands pinning the
shoulder seam of a grey suit jacket" / "chalk-marking a suit lapel pattern on wool fabric"].
Warm window light from the left. Shallow depth of field f/1.8, hands in focus, background blur.
Luxury bespoke tailoring. No faces. No text. No watermarks. Photorealistic.
```

### Fabric / material flat lay
```
Top-down DSLR flat lay photography of [specific fabric types — "dark navy wool suiting fabric
and ivory linen swatches"]. Dark wooden surface. Props: tailor's measuring tape, chalk, a
few buttons. Soft diffused daylight. Sharp across the frame f/8.
No text. No watermarks. No people. Photorealistic.
```

### Blog article hero
```
DSLR editorial photography of [article-specific scene — NOT generic luxury, describe what
the article is actually about]. [Warm / natural / soft light]. [Specific location context].
Documentary editorial style. 35mm grain. No text. No watermarks. No faces unless natural
to the scene. Photorealistic.
```

### Location / exterior
```
DSLR exterior photography of a luxury tailoring boutique entrance in a tropical setting.
[Time of day — warm golden hour / soft morning]. [Specific detail — glass door, signage,
tropical greenery framing the entrance]. Quiet, welcoming, upscale atmosphere.
No people. No text. No watermarks. Photorealistic.
```

---

## Step 4 — Generate images (batch — all at once)

Fire ALL image jobs simultaneously. Never generate one at a time.

```python
import json, subprocess, time, os

KIE_API_KEY = "d3cc526a8f8241e1b35f7226a37ef49b"

task_ids = {}

for section_name, prompt, filename in image_queue:
    result = mcp__kie-ai-mcp-server__generate_nano_banana(
        api_key=KIE_API_KEY,
        prompt=prompt
    )
    task_ids[section_name] = {
        "task_id": result["data"]["taskId"],
        "filename": filename
    }

print(f"Fired {len(task_ids)} jobs simultaneously. Waiting 35 seconds...")
time.sleep(35)
```

---

## Step 5 — Poll, download, and compress

Poll every 10 seconds until all jobs complete.

```python
os.makedirs("images", exist_ok=True)

for section_name, meta in task_ids.items():
    for attempt in range(6):
        status = mcp__kie-ai-mcp-server__get_task_status(
            api_key=KIE_API_KEY,
            task_id=meta["task_id"]
        )
        state = status["api_response"]["data"]["state"]

        if state == "success":
            result_json = json.loads(status["api_response"]["data"]["resultJson"])
            url = result_json["resultUrls"][0]

            png_path = f"images/{meta['filename'].replace('.webp', '.png')}"
            webp_path = f"images/{meta['filename']}"

            # Download
            subprocess.run(["curl", "-s", "-o", png_path, url], check=True)

            # Convert to WebP at quality 72 (targets ~60-100KB for photos)
            subprocess.run([
                "convert", png_path,
                "-quality", "72",
                "-define", "webp:method=6",
                webp_path
            ], check=True)

            # Check file size — if over 100KB, re-compress lower
            size_kb = os.path.getsize(webp_path) / 1024
            if size_kb > 100:
                subprocess.run([
                    "convert", png_path,
                    "-quality", "60",
                    "-define", "webp:method=6",
                    webp_path
                ], check=True)
                size_kb = os.path.getsize(webp_path) / 1024

            # Clean up PNG
            os.remove(png_path)

            print(f"{section_name}: {webp_path} ({size_kb:.0f}KB)")
            break

        elif state in ("failed", "error"):
            print(f"{section_name}: FAILED — skipping")
            break
        else:
            print(f"{section_name}: still processing ({state})...")
            time.sleep(10)
```

---

## Step 6 — Place images with mobile-optimised HTML

All images use `srcset` + `sizes` and responsive CSS. No fixed pixel heights on mobile.

### Hero image (eager load, full width)
```html
<div class="img-hero">
  <img
    src="images/{filename}.webp"
    alt="{entity-based alt text}"
    width="1200" height="500"
    loading="eager"
    decoding="async">
</div>
```
```css
.img-hero {
  width: 100%;
  overflow: hidden;
}
.img-hero img {
  width: 100%;
  height: clamp(220px, 40vw, 500px);
  object-fit: cover;
  object-position: center top;
  display: block;
}
```

### Section image (lazy load, with caption)
```html
<figure class="img-section">
  <img
    src="images/{filename}.webp"
    alt="{entity-based alt text}"
    width="1200" height="600"
    loading="lazy"
    decoding="async">
  <figcaption>{short descriptive caption}</figcaption>
</figure>
```
```css
.img-section {
  margin: 32px 0;
  border: 1px solid var(--border);
  border-radius: 4px;
  overflow: hidden;
}
.img-section img {
  width: 100%;
  height: clamp(180px, 35vw, 420px);
  object-fit: cover;
  display: block;
}
.img-section figcaption {
  padding: 10px 14px;
  font-size: 0.78rem;
  color: var(--muted);
  background: var(--bg2);
}
```

### Process / hands image (compact, beside text on desktop)
```html
<figure class="img-process">
  <img
    src="images/{filename}.webp"
    alt="{entity-based alt text}"
    width="800" height="500"
    loading="lazy"
    decoding="async">
</figure>
```
```css
.img-process {
  margin: 24px 0;
  border-radius: 4px;
  overflow: hidden;
}
.img-process img {
  width: 100%;
  max-width: 640px;
  height: clamp(160px, 30vw, 360px);
  object-fit: cover;
  display: block;
}
```

### Mobile-first global rule (include in every page's `<style>`)
```css
img {
  max-width: 100%;
  height: auto;
}
@media (max-width: 640px) {
  .img-hero img   { height: clamp(180px, 56vw, 280px); }
  .img-section img { height: clamp(160px, 50vw, 240px); }
  .img-process img { height: clamp(140px, 44vw, 200px); }
}
```

---

## Step 7 — Alt text formula

Every image must have entity-specific alt text. Never generic.

```
Format: {Primary entity + detail} at {Business Name}, {Location}

Good:
"Bespoke navy suit on tailor's dress form at The Outfit Custom Tailor, Platinum Fashion Mall Bangkok"
"Expert tailor pinning shoulder seam of grey suit jacket at The Outfit, Bangkok"
"Premium Italian wool and linen fabric swatches for bespoke suits at The Outfit Custom Tailor"
"Tailor's cutting table and dress forms inside The Outfit workshop, Bangkok Thailand"
"Flat lay of navy herringbone suiting wool on dark wood surface, The Outfit Bangkok"

Bad (never use):
"image of suit"
"photo of tailor"
"fabric swatches"
"hero image"
```

---

## Step 8 — Semantic filename convention

```
{entity}-{specific-qualifier}-{context}.webp

Rules:
- Max 60 characters
- Lowercase, hyphens only, no underscores
- Must be unique across the entire site — no two pages share a filename
- Include a business slug or page slug as the last segment when needed for uniqueness

Examples:
bespoke-suit-dress-form-hero-theoutfit.webp      ← hero, this business
tailor-pinning-shoulder-seam-bespoke.webp         ← process, specific action
navy-herringbone-wool-fabric-flatlay.webp         ← fabric, specific type
tailor-workshop-cutting-table-interior.webp       ← interior, specific element
wedding-suit-hanger-tropical-corridor.webp        ← occasion, specific scene
bespoke-trousers-break-detail-closeup.webp        ← service, specific garment detail
boutique-entrance-golden-hour-bangkok.webp        ← exterior, specific time
blog-suit-care-folding-travel-method.webp         ← blog, topic-specific
```

If a filename would duplicate one already in `images/`, append `-[page-slug]` or change the qualifier.

---

## Confirm before generating (mandatory step)

When called after content is written, always show this confirmation before firing any API calls:

```
Ready to generate [N] images for: [page name]

  1. Hero — bespoke-suit-dress-form-hero-theoutfit.webp
     "DSLR editorial of bespoke suit on dress form, warm golden light..."

  2. Process section — tailor-pinning-shoulder-seam-bespoke.webp
     "DSLR macro of tailor's hands pinning shoulder seam..."

  3. Fabric section — navy-herringbone-wool-fabric-flatlay.webp
     "Top-down flat lay of navy herringbone wool, dark wood surface..."

Estimated time: ~35 seconds. Target size: under 100KB each.

Generate? (yes / skip / adjust)
```

Only proceed after confirmation.

---

## Output summary

After all images are placed, report:

```
Images generated for: [page name]

Hero:
  bespoke-suit-dress-form-hero-theoutfit.webp — 68KB — placed after <header>

Process section:
  tailor-pinning-shoulder-seam-bespoke.webp — 54KB — placed before process steps

Fabric section:
  navy-herringbone-wool-fabric-flatlay.webp — 81KB — placed before fabric grid

Total: 3 images, 203KB combined
All images: mobile-optimised with clamp() heights, lazy loading, entity alt text
```

---

## Integration with content-brief and weekly-pipeline

When `/content-brief` finishes a page or `/weekly-pipeline` finishes an article:

1. Extract all section headings from the written content
2. Map each section to an image type (see table in Step 1)
3. Show the confirmation list (see "Confirm before generating" above)
4. After approval, batch-fire all jobs (Step 4)
5. Download, compress to <100KB (Step 5)
6. Insert mobile-optimised HTML at the correct position in the page (Step 6)
7. Report the summary

**Always offer image generation after content is written — not just a hero, but every content section that would benefit from a visual.**

---

## WordPress upload (Novamira MCP)

After generating and compressing:

```
mcp__novamira__mcp-adapter-execute-ability(
    ability="upload_media",
    parameters={
        "file_path": "images/{filename}.webp",
        "alt_text": "{entity-based alt text}",
        "title": "{descriptive title}"
    }
)
```

Use the returned media URL when inserting into WordPress page content.
