---
name: image-gen
description: Context-aware AI image generation for local business websites using the Kie API (Nano Banana / Google Gemini Imagen). Reads page content and structure, identifies every section that needs an image, generates section-appropriate images, converts to WebP, and returns ready-to-place HTML with correct alt text. Works for any page type — service pages, blog articles, about sections, process sections, fabric/material sections, etc. Use when user says "generate images", "add images to this page", "create hero image", "images for my site", "/image-gen", or when weekly-pipeline or content-brief needs images after writing a page.
tools:
  - Read
  - Write
  - Bash
---

# Image Generation Skill — `/image-gen`

Context-aware AI image generation. Reads any page (HTML, markdown, or brief), maps every section that needs an image, generates the right image for each section, converts to WebP, and returns placement-ready HTML.

**This skill works for:**
- New pages written by `/content-brief` or `/weekly-pipeline`
- Existing HTML preview pages
- Any future page — service, blog, about, location, landing page
- Batch site-wide image generation at launch

---

## How context-aware image selection works

Before generating a single image, the skill reads the page and categorises every section:

| Section type | Image style | Example |
|---|---|---|
| Hero / page header | Wide cinematic, establishes the service/entity | Tailor workshop exterior, suit on dress form |
| About / story | Interior warmth, craftsmanship, human element | Workshop interior, tools, fabric detail |
| Service section | Editorial product/process, clean and focused | Close-up of specific garment type |
| Process / how-it-works | Hands at work, step in progress | Tailor measuring, pinning, cutting |
| Fabric / material | Flat lay or close-up texture, studio lighting | Fabric swatches, bolt of wool |
| Wedding / occasion | Lifestyle, aspirational, soft tones | Wedding suit on hanger, boutonniere |
| Location / find us | Exterior architecture, tropical setting | Boutique entrance, street view |
| Blog article hero | Editorial documentary, scene-setting | Context shot for the article topic |
| CTA / contact | Warm, inviting, action-oriented | Workshop door, welcoming space |
| Testimonials | Social proof context — optional | Garment detail, finished product |

---

## Step 1 — Read and map the page

Read the full page content. Extract every section with its heading and content summary.

For each section, determine:
- Does it need an image? (not all sections do — FAQ, pricing tables, CTA text usually don't)
- What type of image is appropriate?
- What is the primary entity of this section?

Output a silent image map (don't show unless asked):

```
Section: Hero
  → Image type: Wide cinematic hero
  → Entity: bespoke tailoring workshop Phuket
  → Filename: bespoke-tailoring-workshop-hero-phuket.webp

Section: About / Our Story
  → Image type: Workshop interior
  → Entity: tailor workshop interior Bangtao
  → Filename: tailor-workshop-interior-bangtao-phuket.webp

Section: Fabric Selection
  → Image type: Flat lay fabric swatches
  → Entity: premium suit fabrics wool linen
  → Filename: premium-suit-fabric-swatches-phuket.webp

Section: Process
  → Image type: Hands / process close-up
  → Entity: tailor measuring fitting suit
  → Filename: tailor-measuring-fitting-bespoke-suit.webp
```

**Sections that do NOT need images (skip):**
- Pure text FAQ sections
- Pricing tables
- Navigation, footers, breadcrumbs
- Testimonial/review quote blocks (text is the content)
- CTA buttons (unless it's a full CTA banner with background)

---

## Step 2 — Check existing images

Before generating, check `images/` directory for any existing WebP files that match a section's entity. If a suitable image already exists, reuse it rather than generating a duplicate.

```bash
ls images/*.webp 2>/dev/null || echo "No existing images"
```

---

## Step 3 — Build section-specific prompts

Use this formula for each image type:

### Hero images (wide, 1200×460–630px)
```
Wide editorial [photography style] of [primary entity + business type] in [location context].
[Lighting — warm golden light / soft window light / moody ambient].
[Setting detail — tropical workshop / dark boutique / lush greenery].
[Aesthetic — luxury fashion, cinematic, dark moody background with warm tones].
No people. No text. No watermarks.
```

### Service section images (square or 4:3, 800×600px)
```
Editorial [photography style] of [specific service entity — garment type, process step].
[Detail focus — close-up of fabric / garment / specific element].
[Lighting — warm natural / soft studio].
[Aesthetic — luxury editorial, clean background or shallow DOF].
No faces. No text.
```

### About / interior images (wide or 3:2, 1200×800px)
```
Interior editorial photography of a [business type] in [location].
[Warm / ambient / natural light]. [Setting details — tables, tools, fabrics, shelving].
[Mood — refined, crafted, welcoming, tropical luxury].
No people. No text.
```

### Process / hands images (square or 4:3)
```
Close-up editorial photography of [specific craft action — measuring, pinning, cutting, hand-stitching].
[Material detail — fabric type, tools visible].
[Lighting — warm window light, shallow DOF].
Luxury bespoke tailoring aesthetic. No faces. No text.
```

### Fabric / material images (wide flat lay)
```
Aerial editorial flat lay photography of [fabric types and colours].
[Surface — dark wooden table / marble / craft paper].
[Props — measuring tape, tailor's chalk, buttons].
Soft natural light. Top-down composition. No text.
```

### Blog article hero (wide editorial)
```
Editorial [documentary / lifestyle / product] photography of [article primary entity].
[Scene context matching article topic]. [Warm natural light].
Luxury [niche] aesthetic. No text. No faces unless specified.
```

### Location / exterior images
```
Elegant exterior photography of a [business type] boutique in a tropical setting.
[Time of day — warm evening / golden hour / soft morning light].
[Architectural details — signage, entrance, greenery].
No people. No text.
```

---

## Step 4 — Generate images (batch)

Fire ALL image jobs simultaneously, then poll. Never generate one at a time.

```python
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

print(f"Fired {len(task_ids)} image jobs. Waiting 30 seconds...")
```

---

## Step 5 — Poll and download

```python
import time, requests

time.sleep(30)

for section_name, meta in task_ids.items():
    status = mcp__kie-ai-mcp-server__get_task_status(
        api_key=KIE_API_KEY,
        task_id=meta["task_id"]
    )
    state = status["api_response"]["data"]["state"]
    
    if state == "success":
        result_json = json.loads(status["api_response"]["data"]["resultJson"])
        url = result_json["resultUrls"][0]
        
        # Download PNG
        subprocess.run(["curl", "-s", "-o", f"images/{meta['filename'].replace('.webp', '.png')}", url])
        
        # Convert to WebP
        subprocess.run(["convert", 
            f"images/{meta['filename'].replace('.webp', '.png')}",
            "-quality", "85",
            f"images/{meta['filename']}"])
        
        # Remove PNG
        os.remove(f"images/{meta['filename'].replace('.webp', '.png')}")
        
        print(f"{section_name}: saved as images/{meta['filename']}")
    else:
        print(f"{section_name}: still processing ({state}) — retry in 10s")
```

Poll every 10 seconds for any outstanding jobs until all are complete.

---

## Step 6 — Place images in sections

After all images are downloaded, generate placement HTML for each section.

### HTML placement rules by section type

**Hero image — full width, above fold, eager loading:**
```html
<div style="overflow:hidden;max-height:460px;border-bottom:1px solid var(--border);">
  <img src="images/{filename}.webp"
       alt="{entity-based alt text}"
       width="1200" height="460"
       style="width:100%;height:460px;object-fit:cover;object-position:center top;display:block;"
       loading="eager">
</div>
```

**Section image — within content section, lazy loading:**
```html
<figure style="margin:40px 0;border:1px solid var(--border);overflow:hidden;border-radius:4px;">
  <img src="images/{filename}.webp"
       alt="{entity-based alt text}"
       width="1200" height="600"
       style="width:100%;height:360px;object-fit:cover;display:block;"
       loading="lazy">
  <figcaption style="padding:12px 16px;font-size:0.78rem;color:var(--muted);background:var(--bg2);">
    {short descriptive caption}
  </figcaption>
</figure>
```

**Process / hands image — smaller, beside text or centred:**
```html
<figure style="margin:32px 0;overflow:hidden;border-radius:4px;">
  <img src="images/{filename}.webp"
       alt="{entity-based alt text}"
       width="800" height="500"
       style="width:100%;max-width:640px;height:320px;object-fit:cover;display:block;"
       loading="lazy">
</figure>
```

**About section image — full width with caption:**
```html
<figure style="margin:40px 0 0;border-top:1px solid var(--border);overflow:hidden;">
  <img src="images/{filename}.webp"
       alt="{entity-based alt text}"
       width="1200" height="500"
       style="width:100%;height:400px;object-fit:cover;object-position:center;display:block;"
       loading="lazy">
</figure>
```

**Blog article inline image — within article body:**
```html
<figure style="margin:40px 0;border:1px solid var(--border);overflow:hidden;border-radius:4px;">
  <img src="images/{filename}.webp"
       alt="{entity-based alt text}"
       width="760" height="420"
       style="width:100%;height:300px;object-fit:cover;display:block;"
       loading="lazy">
  <figcaption style="padding:10px 14px;font-size:0.76rem;color:var(--muted);background:var(--bg2);">
    {caption}
  </figcaption>
</figure>
```

---

## Step 7 — Alt text formula

Every image must have entity-based alt text:

```
{Primary entity} {action or context} at {business name}, {location detail}

Examples:
"Bespoke suit on dress form at The Outfit Custom Tailor workshop, Bangtao Beach Phuket"
"Expert tailor hands measuring suit jacket at The Outfit, Choeng Thale Phuket"
"Premium wool and linen fabric swatches for bespoke tailoring at The Outfit, Phuket"
"Tailoring workshop interior at The Outfit Custom Tailor, Bangtao Beach Phuket"
"Wedding suit on hanger at The Outfit bespoke tailor, Phuket Thailand"
```

Never use generic alt text like "image of suit" or "photo".

---

## Step 8 — Semantic filename convention

```
{entity}-{qualifier}-{location}-{context}.webp

Max 60 characters. Lowercase. Hyphens only. No underscores.

Examples:
bespoke-suit-dress-form-phuket-tailor.webp
tailor-workshop-interior-bangtao-phuket.webp
premium-suit-fabric-swatches-phuket.webp
tailor-measuring-fitting-bespoke-suit.webp
wedding-suit-hanger-tropical-boutique-phuket.webp
garment-alteration-hands-phuket-tailor.webp
tailoring-atelier-wide-shot-phuket.webp
```

---

## Integration with content-brief and weekly-pipeline

When `/content-brief` finishes a page brief, or `/weekly-pipeline` finishes an article:

1. Extract all section headings from the written content
2. Map each section to an image type (see table at top)
3. Ask: "Should I generate images for this page? I'll create [X] images — one for the hero, one for [section], one for [section]."
4. If yes, run Steps 1–7 for that page automatically
5. Return all images with placement HTML inserted into the correct positions in the content

**The image-gen skill is always offered after content is written — not just for the hero but for every section that would benefit from an image.**

---

## Output summary format

After all images are placed, report:

```
Images generated and placed for: [page name]

Hero:
  → images/bespoke-tailoring-workshop-hero-phuket.webp (87KB) — placed after <header>

About section:
  → images/tailor-workshop-interior-bangtao-phuket.webp (112KB) — placed in .about-section

Process section:
  → images/tailor-measuring-fitting-bespoke-suit.webp (64KB) — placed before process steps

Fabric section:
  → images/premium-suit-fabric-swatches-phuket.webp (143KB) — placed before fabric grid

Total: 4 images, 406KB combined
```

---

## WordPress integration (Novamira MCP)

After generating and converting, upload each WebP to the WordPress media library:

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

Then use the returned media URL when inserting into WordPress page content.
