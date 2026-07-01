# AI Style Library Guide

Last updated: 2026.

---

## Purpose

Build reusable, subject-independent style reference assets (hairstyles, and potentially clothing, accessories, etc.) on faceless mannequins. These assets plug into the person generation workflow ([PERSON-GENERATION.md](PERSON-GENERATION.md)) as clean style references with zero face contamination.

---

## Why a Style Library

When generating images of a person with a specific style (hairstyle, outfit, etc.), using photos of real people as style references causes **face contamination** -- the model blends the reference person's face with the subject. The style library solves this by providing style references on featureless mannequins. The model has no face to contaminate from.

**Benefits:**
- Zero face contamination from style references
- Consistent construction across subjects (same mannequin, same angles)
- Reusable across any number of subjects
- Neutral color (brown) -- subject's color applied via texture swatch
- Multi-angle coverage enables angle-matched style refs

---

## Building a Style Entry

### Step 1: Create the Anchor View

Generate ONE three-quarter-right view first. This becomes the canonical reference for all other angles.

**Two approaches depending on source material:**

**From reference photos (real person wearing the style):**
- Send 1-3 reference photos + detailed text description
- Ask for a featureless white mannequin with this hairstyle
- The model extracts the style from the photos and applies it to the mannequin

**From text description only (common styles):**
- Describe the style construction in detail
- No reference photos needed for well-known styles (ponytails, braids, etc.)
- More creative control but less precision for unusual styles

**Prompt template:**
```
[Optional: IMAGES 1-N show this hairstyle on real people. Study the structure.]

Generate a hairstyle reference on a featureless white mannequin
(NO FACE -- smooth blank white surface). Plain white mannequin body
in a plain white fitted top. Neutral grey studio background, even
lighting. THREE-QUARTER RIGHT angle. Hair color: neutral medium brown.
NO EARRINGS. NO JEWELRY. NO ACCESSORIES of any kind. Bare ears.
Hairstyle catalog image. Sharp detail. Photorealistic hair on mannequin.

HAIRSTYLE: [Detailed construction description]

Show from head to at least hip level.
```

**No accessories on mannequins.** Always include "NO EARRINGS. NO JEWELRY. NO ACCESSORIES." Reference photos of subjects wearing earrings can cause accessory bleed onto the featureless mannequin.

**Settings:** 1:1 square, 2K, temperature 0.2, Flash model is fine.

**Verify the anchor before proceeding.** The construction must be correct -- every other angle will reference this image.

### Step 2: Generate Remaining Angles

Feed the anchor image as IMAGE 1 and ask the model to rotate the view:

```
IMAGE 1 is a hairstyle on a white mannequin from a three-quarter angle.
This is the CANONICAL reference.

Show this SAME mannequin with this SAME hairstyle from a different angle:
[Front view / Side profile / Back view]

Match the hairstyle from IMAGE 1 exactly. Same white featureless mannequin,
same grey background. Photorealistic.
```

**Standard angles (4 per style):**
1. Three-quarter right (the anchor)
2. Front
3. Side right
4. Back

**Key insight:** "Rotate this same mannequin" produces much more consistent results than re-describing the hairstyle construction for each angle. The anchor image is the single source of truth.

### Step 3: Fix Inconsistencies

If an angle doesn't match the anchor's construction:
- Re-run with the anchor + the full-body front view as two references
- Add original real-person reference photos as supplementary context
- For back views of complex styles, use 9:16 framing to show full length
- Don't try to describe the construction again in text -- let the visual anchor do the work

---

## Construction Descriptions

For complex or unusual styles, the construction description is critical. Structure it as:

1. **Origin** -- where the style starts (crown, nape, temples, etc.)
2. **Gathering** -- how the hair is collected (ponytail, bun, sections, etc.)
3. **Structure** -- what happens after gathering (braiding, weaving, flowing, etc.)
4. **Terminus** -- where it ends and how (tapers, pools on floor, etc.)
5. **Defining feature** -- the one thing that makes this style recognizable

**For simple styles** (ponytails, straight hair, basic braids), a brief description works. The model knows these from training data.

**For complex styles** (woven lattice, intricate updos, unusual braiding patterns), number the construction steps so the model builds it logically. Include what it should NOT do ("no loose hanging braids that aren't connected").

---

## Controlling Hair Length

Length is the hardest hairstyle attribute to control, and the intuitive fix doesn't work:

- **A length chart or reference image does NOT control output length.** Feeding a "how long is the hair" diagram gets ignored (tested across multiple Gemini and OpenAI image models).
- **Forceful, explicit anatomical endpoint phrasing in the prompt is what works.** Name where the hair ends on the body: "ends at the collarbone," "reaches mid-back," "stops at the waist -- do NOT stop at the shoulder." The more specific the landmark, the better.
- **The longest styles need two extra levers:** language that the hair pools or coils at the ends, and camera framing pulled back far enough to show the full length. Without the pulled-back framing the model crops before the hair ends.

---

## Using the Library

When generating an image of a person with a library hairstyle:

1. Select the angle-matched mannequin view (e.g., front mannequin for front-facing output)
2. Place it LAST in the image order (after identity refs and texture swatches)
3. Label it: "IMAGE X is a HAIRSTYLE reference (mannequin) -- IGNORE the face, use ONLY for hairstyle structure"
4. Include the subject's hair texture swatch for color
5. Add "make the hair organic and natural -- real texture, slight wisps, natural shine. Not stiff or mannequin-like"

The mannequin gives the model the structure; the texture swatch gives it the color; the organic instruction gives it life.

---

## Pose Variation

When generating a showcase or batch, vary the poses to make each image distinct:

**Standing poses:** power pose (hands on hips), casual (weight on one hip), editorial (chin up, strong expression), relaxed (hands clasped in front)

**Action poses:** walking mid-stride, mid-turn with dress flowing, brushing hair behind ear, adjusting jewelry

**Seated/leaning:** sitting cross-legged, leaning against wall, chin on hand

**Profile/angled:** looking over shoulder, three-quarter with eye contact, full profile

**Match the pose to the hairstyle's best viewing angle.** A front-facing pose hides a ponytail, braid, or back-of-head detail. For styles where the structure is behind or to the side, pose the subject turned away or in three-quarter back view with the head turned toward camera. The hairstyle should be the visual focus, not hidden behind the subject.

Match headshot angles to the pose -- use profile headshots for profile poses, front headshots for front-facing poses.

---

## Batch Generation

A typical showcase batch: 10 styles x 1 subject = 10 images.

**Per image, 5 inputs:**
1. Angle-matched headshot
2. Second headshot (different angle for triangulation)
3. Body reference
4. Hair texture swatch
5. Mannequin style reference (LAST)

At ~$0.02/image on Flash, a 10-image showcase costs ~$0.20 and takes ~5 minutes.

**Multi-subject efficiency:** Once the library exists, onboarding a new subject takes ~15 minutes (compress refs, generate 16 identity assets, run showcase). The library doesn't need to be regenerated.

---

## Current Library

62 styles (most in 4 angles), plus a 169-image Hair Looks library of real-world styled looks organized by cut, length, color, and texture. See [library/INDEX.md](library/INDEX.md) for the styles catalog and [library/looks/INDEX.md](library/looks/INDEX.md) for looks. Browse the [library/](library/) folder for all images.

| Category | Count |
|----------|-------|
| Braids | 6 |
| Editorial | 7 |
| Flowing | 18 |
| Ponytails & Half-Up | 8 |
| Short | 13 |
| Ultra-Long | 1 |
| Updos | 9 |

---

## Expanding the Library

**Adding new styles:**
1. Find or describe the style
2. Generate the three-quarter anchor
3. Verify construction
4. Generate front, side, back from the anchor
5. Test on one subject to confirm it works in practice

**Beyond hairstyles -- clothing profiles (tested, works well):**

The mannequin approach extends to clothing. Use a bald, featureless white mannequin (NO face, NO hair) wearing the target outfit. Send 2-3 real photos of the subject wearing the outfit as references for color/fit accuracy.

```
Generate this OUTFIT on a featureless white mannequin (NO FACE -- smooth blank white surface, NO hair).
Neutral grey studio background, even lighting. THREE-QUARTER RIGHT angle.
NO HAIR on the mannequin. Bald smooth white head. NO face features. NO earrings, NO jewelry.

OUTFIT: [Detailed garment description -- fabric, color, fit, construction details]

Show full outfit from shoulders to at least mid-thigh. Clothing catalog image. Sharp detail. Photorealistic.
```

Generate anchor (3/4 right) + front view minimum. Useful for signature outfits that appear across multiple generations (a professional's go-to blazer, a performer's stage outfit, etc.).

**Other extensions (untested):**
- Jewelry/accessory references (on a bust)
- Makeup looks (on a face mannequin with features but generic)

The principle is the same: isolate the style attribute on a neutral form so it can be applied to any subject without contamination.

---

## Cost Reference

| Task | Model | Cost |
|------|-------|------|
| Single anchor generation | Flash | ~$0.02 |
| Full style entry (4 angles) | Flash | ~$0.08 |
| Full library (62 styles x 4 angles) | Flash | ~$5.00 |
| 10-image showcase per subject | Flash | ~$0.20 |
