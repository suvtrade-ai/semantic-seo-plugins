---
name: image-gen
description: AI image generation for local business websites via Gemini API (gemini-3.1-flash-image-preview, 2K resolution). Reads page content, maps sections, generates geotagged JPEG images with city-level GPS for local SEO, compresses to under 200KB. Use when user says generate images, add images, create hero image, images for my site, or /image-gen.
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
  Filename: bespoke-suit-dress-form-hero-[business-slug].jpg
  Alt: Bespoke suit on dress form at [Business Name], [City]

Section: Our Process
  Type: Hands / craft action
  Entity: tailor pinning suit jacket shoulder seam
  Filename: tailor-pinning-suit-shoulder-seam-[business-slug].jpg
  Alt: Expert tailor pinning bespoke suit shoulder at [Business Name]

Section: Fabric Selection
  Type: Flat lay
  Entity: premium wool suiting fabric swatches dark tones
  Filename: premium-wool-suiting-fabric-swatches-[business-slug].jpg
  Alt: Premium wool and linen fabric swatches for bespoke suits at [Business Name]
```

---

## Step 2 — Check existing images

Before generating, check the `images/` directory. If a filename already exists, do not regenerate it — reuse it only if the entity and context are genuinely the same section. Otherwise, create a differentiated filename.

```bash
ls images/*.jpg 2>/dev/null && echo "---existing images above---" || echo "No existing images"
```

If you find an image like `tailor-workshop-interior.jpg` already used on the homepage, do NOT use it again on the About page. Instead generate `tailor-workshop-workbench-closeup.jpg` or `tailor-atelier-interior-detail.jpg`.

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

## Step 4 — Generate images via Gemini API (Nano Banana 2)

Generate one image per section using the Gemini REST API directly. Images are sequential (Gemini doesn't support true parallel image batching), so fire them in a loop.

**Required:** `GEMINI_API_KEY` env var.  
**Model:** `gemini-3.1-flash-image-preview` (Nano Banana 2, up to 4K)  
**Resolution:** `2K` for hero/service, `1K` for process/blog inline  
**Format:** PNG (Gemini output) → JPEG (after geotag conversion)

```python
import os, base64, json, requests

GEMINI_KEY = os.environ.get("GEMINI_API_KEY")
GEMINI_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-image-preview:generateContent"

os.makedirs("images", exist_ok=True)

generated = {}

for section_name, prompt, base_filename in image_queue:
    # Hero images: 2K + 16:9 | Square/detail: 2K + 1:1 | Blog inline: 1K + 16:9
    if "hero" in base_filename or "exterior" in base_filename:
        img_size, aspect = "2K", "16:9"
    elif "flatlay" in base_filename or "fabric" in base_filename:
        img_size, aspect = "2K", "1:1"
    else:
        img_size, aspect = "2K", "4:3"

    payload = {
        "contents": [{"parts": [{"text": prompt}]}],
        "generationConfig": {
            "responseModalities": ["IMAGE"],
            "imageConfig": {
                "aspectRatio": aspect,
                "imageSize": img_size   # MUST be uppercase
            }
        }
    }

    resp = requests.post(
        f"{GEMINI_URL}?key={GEMINI_KEY}",
        json=payload,
        headers={"Content-Type": "application/json"},
        timeout=120
    )

    if resp.status_code != 200:
        print(f"{section_name}: Gemini error {resp.status_code} — {resp.text[:200]}")
        generated[section_name] = {"status": "failed", "filename": base_filename}
        continue

    data = resp.json()
    parts = data.get("candidates", [{}])[0].get("content", {}).get("parts", [])
    img_part = next((p for p in parts if "inlineData" in p), None)

    if not img_part:
        finish = data.get("candidates", [{}])[0].get("finishReason", "unknown")
        print(f"{section_name}: No image returned — finishReason={finish}")
        generated[section_name] = {"status": "blocked", "reason": finish, "filename": base_filename}
        continue

    # Save raw PNG from base64
    png_path = f"images/{base_filename}.png"
    with open(png_path, "wb") as f:
        f.write(base64.b64decode(img_part["inlineData"]["data"]))

    print(f"{section_name}: PNG saved → {png_path} ({os.path.getsize(png_path)//1024}KB)")
    generated[section_name] = {"status": "ok", "png": png_path, "filename": base_filename}
```

**If Gemini blocks a prompt** (finishReason=IMAGE_SAFETY): rephrase — replace any descriptive words about people, faces, or culturally sensitive elements. Try "a tailor's workshop interior" instead of "a Thai tailor at work".

---

## Step 5 — Geotag + Convert to JPEG + Compress

Every generated PNG goes through this pipeline:
1. **Convert PNG → JPEG** (Pillow, quality=92) — piexif only works on JPEG
2. **Embed GPS EXIF** (piexif) — city-level coordinates only, not the exact street address
3. **Compress** — target <200KB; re-compress at lower quality if over

**GPS rule:** use city-center coordinates, not the exact business address. This gives the local SEO signal without exposing precise location.

```python
from PIL import Image
import piexif, os

# === SET THESE FROM 01-foundation.md OR business intake ===
BUSINESS_LAT  = 13.7563   # Bangkok example — replace with actual city center lat
BUSINESS_LON  = 100.5018  # Bangkok example — replace with actual city center lon
# ==========================================================

def to_dms(value):
    """Convert decimal degrees to DMS rational tuple for piexif."""
    d = int(abs(value))
    m = int((abs(value) - d) * 60)
    s = round(((abs(value) - d) * 60 - m) * 60 * 10000)
    return ((d, 1), (m, 1), (s, 10000))

def geotag_jpeg(jpeg_path, lat, lon):
    exif_dict = {"0th": {}, "Exif": {}, "GPS": {}, "1st": {}}
    exif_dict["GPS"] = {
        piexif.GPSIFD.GPSVersionID:    (2, 0, 0, 0),
        piexif.GPSIFD.GPSLatitudeRef:  b"N" if lat >= 0 else b"S",
        piexif.GPSIFD.GPSLatitude:     to_dms(lat),
        piexif.GPSIFD.GPSLongitudeRef: b"E" if lon >= 0 else b"W",
        piexif.GPSIFD.GPSLongitude:    to_dms(lon),
    }
    exif_bytes = piexif.dump(exif_dict)
    piexif.insert(exif_bytes, jpeg_path)

os.makedirs("images", exist_ok=True)

for section_name, meta in generated.items():
    if meta["status"] != "ok":
        continue

    png_path  = meta["png"]
    # Filename uses .jpg convention: service-city-description.jpg
    jpg_name  = meta["filename"].replace(".jpg", "").replace(".png", "") + ".jpg"
    jpg_path  = f"images/{jpg_name}"

    # 1. Convert PNG → JPEG
    img = Image.open(png_path).convert("RGB")
    img.save(jpg_path, "JPEG", quality=92)
    os.remove(png_path)  # clean up PNG

    # 2. Geotag the JPEG with city-level GPS
    geotag_jpeg(jpg_path, BUSINESS_LAT, BUSINESS_LON)

    # 3. Check size — compress further if over 200KB
    size_kb = os.path.getsize(jpg_path) / 1024
    if size_kb > 200:
        img2 = Image.open(jpg_path).convert("RGB")
        img2.save(jpg_path, "JPEG", quality=78)
        geotag_jpeg(jpg_path, BUSINESS_LAT, BUSINESS_LON)  # re-embed after re-save
        size_kb = os.path.getsize(jpg_path) / 1024

    # 4. Update meta with final path
    meta["jpg"] = jpg_path
    print(f"{section_name}: {jpg_path} ({size_kb:.0f}KB) — GPS {BUSINESS_LAT},{BUSINESS_LON} embedded")

# Install dependencies if missing:
# pip install Pillow piexif --break-system-packages
```

**Filename convention:** `service-city-description.jpg`
Examples:
- `custom-tailor-bangkok-suit-fitting.jpg`
- `bespoke-suit-bangkok-hero.jpg`
- `tailor-bangkok-fabric-flatlay.jpg`

**Why JPEG not WebP:** piexif requires JPEG for GPS embedding. WebP does not support EXIF GPS. The local SEO signal from geotagged JPEG outweighs the slight compression advantage of WebP for this use case.

---

## Step 6 — Place images with mobile-optimised HTML

All images use `srcset` + `sizes` and responsive CSS. No fixed pixel heights on mobile.

### Hero image (eager load, full width)
```html
<div class="img-hero">
  <img
    src="images/{filename}.jpg"
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
    src="images/{filename}.jpg"
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
    src="images/{filename}.jpg"
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
"Premium Italian wool and linen fabric swatches for bespoke suits at The Outfit Custom Ta