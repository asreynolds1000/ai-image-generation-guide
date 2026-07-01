# AI Person Image Generation Guide

Last updated: April 2026

> **Ethics & Consent:** This guide is intended for use with subjects who have given explicit consent to AI image generation in their likeness. Do not use these techniques to generate images of people without their knowledge or permission. Respect each provider's acceptable use policies.

**Note:** Ref count, prompt style, and Flash-vs-Pro findings based on single-subject testing (April 2026). Re-verify with new subjects.

---

## Purpose

Generate photorealistic AI images of a specific real person in new scenes, outfits, poses, or styles while preserving their identity. Covers the full workflow from collecting references through prompting, multi-era identity management, compositing into existing photos, and 4K upscaling.

---

## Provider Selection

| Provider | Strength | Limitation |
|----------|----------|------------|
| **Gemini 3.1 Flash Image** | Strong multi-constraint adherence, 1000 RPD, comparable quality to Pro for identity work | Slightly less creative latitude than Pro |
| **Gemini 3 Pro Image** | Best for 4K output, creative/editorial work | 250 RPD limit, no better than Flash for identity preservation |
| **OpenAI gpt-image-1.5** | 16 reference images, native transparency | Max 1024x1536, tends to smooth/airbrush faces, weaker identity preservation |
| **Grok Imagine** | Best for single-photo edits, $0.02/image, fast | Single image input only, no multi-ref |

**Default: Gemini 3.1 Flash Image.** Flash matches Pro on identity fidelity and has 4x the daily rate limit. Use Pro for 4K final renders and editorial work. Use Grok for single-photo targeted edits. Use OpenAI when you need transparency or text rendering.

---

## Step 1: Collect Reference Photos

Gather 8-15 real photos of the subject. Prioritize variety:

**Must-have angles:**
- Front-facing (clear, well-lit)
- Three-quarter view (both sides if possible)
- Profile/side view
- Full body (establishes build and proportions)

**Good to have:**
- Different lighting conditions
- Different expressions
- Different contexts (indoor/outdoor)
- Close-ups showing distinguishing features (freckles, dimples, scars, facial hair, etc.)

**Avoid:**
- Group photos where the subject's face is small
- Heavy filters or unusual lighting
- Photos where the subject is facing away from camera
- Low resolution or heavily compressed images

**Multi-era subjects:** If the subject's appearance has changed over time (weight, age, hair), collect separate photo sets per era. Each era needs its own identity assets (headshots, body profiles). Don't mix eras in a single ref set -- the model will average them.

**Compression for API:** Two tiers depending on use:
- **Identity refs** (face-critical): 1500px, quality 50. Face detail matters here.
  ```bash
  sips -Z 1500 -s format jpeg -s formatOptions 50 input.jpg --out output.jpg
  ```
- **Non-identity refs** (mannequins, scene refs, clothing): 1200px, quality 35.
  ```bash
  sips -Z 1200 -s format jpeg -s formatOptions 35 input.jpg --out output.jpg
  ```
For high-res originals (DSLR), crop to head-and-shoulders first (face should fill 50%+ of frame), then compress.

---

## Step 2: Generate AI Identity Headshots

Real photos have inconsistent lighting, backgrounds, and angles that confuse the model. Generate a clean, controlled set of AI headshots to use as identity references.

**Why this works:** The AI headshots are already in the model's "visual language" -- consistent lighting, neutral backgrounds, no distractions. They resist contamination from style references much better than real photos.

**Process:**
1. Select 3 of the best real photos -- all front-facing or near-front, face filling 50%+ of frame
2. Add feature closeups (nose, teeth/smile) as supporting refs -- 5 total images per call
3. Generate headshots at specific angles: front, 3/4 left, 3/4 right, profile left, profile right, slightly above
4. Hair pulled back tight (bun, ponytail, or slicked back) to expose the full face and hairline
5. Neutral grey studio background, black crew-neck top
6. Square 1:1 format, 2K resolution, low temperature (0.2)
7. Use angle-matched refs per shot (profile refs for profiles, front refs for front, etc.)

**3 identity refs, not 5-6.** More refs causes face averaging -- the model blends features across all inputs instead of anchoring to one face. Tested April 2026: 3 refs + 2 feature closeups consistently outperformed 5-6 identity refs.

**Let Gemini write the prompts.** Send the real photos to a Gemini text model (Flash) and ask it to describe the subject's specific face for use in generation prompts. The model identifies precise features (face shape, eye depth, lip shape, complexion) better than manual description. Fix any beautification in its output (it tends to describe teeth as "straight" and "white" even when they're not).

**Verify skin tone against real photos.** AI descriptions often default to "olive-toned" or "warm complexion" when the subject is actually fair/light with pink undertones. Get the skin tone description right -- an incorrect skin tone anchor is one of the biggest causes of "different person" output.

**Multi-era headshots:** If the subject's appearance has changed over time, generate separate headshot sets per era. Include era-specific descriptions of how the face differs.

**Verify before proceeding:** Review all headshots. They must be recognizable as the subject at that specific era. If they drift, regenerate with different real photo combinations. These headshots are the foundation -- everything downstream depends on them.

---

## Step 3: Generate Body Profiles (Optional)

When body shape accuracy matters in the output, generate controlled body silhouettes as reference images.

**Process:**
1. Send 4-6 full-body real photos to Gemini
2. Ask for body refs at 4 angles: front, side, three-quarter, back
3. Fitted neutral grey athletic wear (tank + leggings) that shows body shape
4. Neutral grey studio background, even lighting, barefoot
5. 9:16 format, 2K resolution

**Two versions per angle:**
- **Headless** (cropped at collarbone): Prevents face contamination when used as a body-only ref. Like a dressmaker's mannequin.
- **With face** (full head-to-toe): Preserves how the face relates to the body. Use era-matched face refs.

**Be specific about proportions.** Gemini tends to default toward average builds. Describe the subject's actual proportions explicitly rather than using vague terms like "fuller build."

**Every variant needs its own identity assets.** Skipping the identity pipeline (headshots + body profiles) for a variant and just extrapolating from a single image produces noticeably weaker results. The model needs dedicated multi-angle identity refs to maintain consistency across a batch.

---

## Step 4: Generate Texture Swatches (Optional)

When you need to preserve the subject's natural hair color, skin tone, or other texture attributes across different styles, generate pure texture reference images.

- Generate 2-3 versions matching the subject's natural variations (straight, wavy, curly, etc.)
- Frame should be 100% texture -- no face, no background, edge to edge
- Use 5+ real photos as color reference
- "Shot like a fabric swatch or material sample"

**Generate multiple color-condition swatches.** Most people's hair shows different tones in different lighting. Generate swatches for: base color (straight), wavy texture, curly texture, darkest shade (indoor/shadow), highlight tones (sunlit), and any warm/cool undertones. 6 swatches covers the full range and lets you color-match per scene lighting.

**When to use:** Include the appropriate swatch as a separate reference image when the style reference has a different color/texture than the subject's natural attributes. Label it explicitly: "IMAGE X is a color/texture swatch -- use this color, IGNORE the mannequin's color."

---

## Step 4b: Use the Style Library (Optional)

For hairstyles and other style attributes, use pre-built mannequin references from the Style Library instead of photos of real people. These eliminate face contamination entirely.

**See:** `STYLE-LIBRARY.md` in this repo for how to build and use the library.

**Quick usage:** Select the angle-matched mannequin view for the target pose, place it LAST in image order, include the subject's texture swatch for color, add "make it organic and natural, not stiff."

**Current library:** 75 styles (4 angles each), plus 169 styled looks. See `STYLE-LIBRARY.md` for the full catalog and how to build your own.

---

## Step 4c: Generate Subject-Specific Hairstyle Profiles

For the subject's 4-6 most common hairstyles, generate dedicated mannequin profiles in the subject's actual hair color, texture, and part side. These outperform generic library mannequins for everyday looks because the color, part, and construction are pre-baked.

**Process:**
1. Identify the subject's most common hairstyles from reference photos
2. Group reference photos by hairstyle -- include 2-3 real photos showing each style
3. Generate the three-quarter-right anchor using real photos as input (not text-only)
4. Verify the anchor -- check part side, volume, color, construction
5. Generate remaining angles (front, side, back) from the anchor + real photos

**Include real reference photos when generating mannequins.** Text-only descriptions default to center parts and generic construction. Sending 2-3 real photos of the subject wearing the style produces much more accurate mannequins.

**Verify hair part side.** Check the subject's natural part from professional (non-selfie) photos. Selfies mirror the image. Specify part side explicitly in every mannequin prompt using frame-relative language: "Part line on LEFT of the image, more volume to the RIGHT" -- not body-relative language like "part on her left" which is ambiguous in 3/4 views. If the model gets it wrong, use the correct mannequin from another style as a visual "part reference" (IMAGE 1, labeled "match the part side from this image").

**"Signature" hairstyle.** Beyond the standard styles, generate a dedicated mannequin for the subject's most common/default look. Emphasize natural volume and body -- mannequin hairstyles tend toward flat/sleek. Include real photos showing the subject's actual hair volume as references, and add aggressive volume language: "NATURAL VOLUME, BODY, MOVEMENT. NOT flat, NOT sleek. If flat, the image has FAILED."

**No accessories on mannequins.** Add "NO EARRINGS. NO JEWELRY. NO ACCESSORIES. Bare ears." to every mannequin prompt. Real photos of the subject wearing earrings can cause accessory bleed onto the featureless mannequin.

Reserve the generic library for experimental/fun styles outside the subject's usual rotation.

---

## Step 4d: Generate Expression Profiles

Generate 4-5 front-facing portraits of the subject showing distinct expressions. These serve as expression references during final generation -- select the one matching the desired mood.

**Group reference photos by expression.** For each expression, select 3-4 real photos where the subject shows that specific expression. This produces dramatically better results than text-only expression descriptions. The model needs to SEE the target expression, not just read about it.

**Standard expression set:**
1. Signature smile (their most common/natural expression)
2. Big open-mouth smile (joyful, energetic)
3. Warm closed-mouth smile (composed, professional)
4. Laughing (candid, eyes crinkled)
5. Composed/serious (minimal smile, direct eye contact)

**Hair down, not pulled back.** Expression profiles should show the subject's default hairstyle (hair down), not the pulled-back neutral headshots. This lets the expression profile double as a hair volume reference during generation.

**Flash is sufficient for expression profiles** (April 2026 testing, single subject). Flash's multi-constraint adherence matches Pro for face accuracy at this scale.

---

## Step 4e: Generate Clothing Profiles (Optional)

The mannequin approach extends to clothing. Use a bald, featureless white mannequin (NO face, NO hair) wearing the target outfit.

**Process:**
1. Send 2-3 real photos of the subject wearing the outfit
2. Generate anchor (3/4 right) + front view minimum
3. Bald smooth white head, no face features, no accessories
4. Focus prompt on garment details: fabric, color, fit, construction

**When to use:** When the subject has signature outfits that will appear in multiple generations (e.g., a professional's go-to blazer, a performer's stage outfit). Prevents having to describe the outfit in text every time and ensures consistency across images.

---

## Step 4f: Three-Quarter Angle Mirroring

If the model generates the same angle for both three-quarter-left and three-quarter-right despite explicit prompts (common when reference photos bias toward one angle):

1. Take the successful three-quarter-right output
2. Mirror/flip it horizontally (PIL `ImageOps.mirror()`)
3. Send the flipped image as a POSE REFERENCE (last position)
4. Send identity refs first (real photos + AI headshots)
5. Label: "IMAGE N is a POSE REFERENCE ONLY showing the desired angle. Use it ONLY for body orientation. The face MUST come from IMAGES 1-4."

This produces a true mirrored view while preserving identity from the real references.

---

## Step 5: Prompt Architecture

### Image Ordering (Critical)

**Identity references FIRST, style reference LAST.** Gemini weights earlier images more heavily. If a style ref with a prominent face is IMAGE 1, its facial features bleed into the output.

Typical ordering for a 5-image generation call:
- IMAGE 1: Real photo of the subject (angle-matched, face-dominant) -- strongest identity anchor
- IMAGE 2: AI headshot (pose-matched to target)
- IMAGE 3: Expression profile (expression-matched)
- IMAGE 4: Nose closeup
- IMAGE 5: Teeth closeup

For looks with hairstyle mannequins, drop teeth closeup and add mannequin as IMAGE 5.

**3 face identity refs maximum.** More causes face averaging -- the model blends features across all inputs instead of anchoring to one face (tested April 2026, single subject). Mix one real photo + one AI headshot + one expression profile for best results.

**Angle matching:** For profile outputs, include profile headshots. For front-facing, include front headshots. For back-turned-head poses, include profile + 3/4 views.

### Prompt Structure

**Refs carry identity, prompts carry scene.** Heavy identity blocks with negative constraints ("NOT narrow", "NOT angular") don't improve scores and can cause overcorrection. The reference images do the identity work. The prompt should describe what you want to see.

**Use a single flowing paragraph, not sectioned blocks:**

```
A [framing] portrait of [Name], [pose/angle]. [Expression]. 
[Eye color/detail]. [Hair description]. Wearing [outfit]. 
[Lip color/makeup if relevant]. Natural skin texture with visible 
[age-appropriate details], not overly smoothed or airbrushed. 
[Accessories]. [Background], [lighting]. [Aspect ratio].
```

**What to include in the prompt:**
- Framing + pose + angle
- Expression description
- Eye color and expression-specific detail
- Hair description (style, color, length, volume)
- Outfit (specific fabric, color, fit)
- One skin texture line ("natural skin texture, visible fine lines, not overly smoothed")
- Accessories and background
- Aspect ratio and resolution

**What NOT to include in the prompt:**
- Identity blocks listing facial features (the refs handle this)
- Negative constraints ("NOT narrow", "NOT angular") -- causes overcorrection
- "CRITICAL" or "NON-NEGOTIABLE" headers -- doesn't change model behavior
- Detailed nose/teeth descriptions -- the closeup refs are more effective

**Image labels still matter.** Label each ref image with its role:
- Real photos: "REAL PHOTO of [Name]. Her face MUST match this."
- AI headshots: "AI headshot of [Name]. Her face MUST match this."
- Feature closeups: "NOSE CLOSE-UP. Match exactly." / "TEETH CLOSE-UP. Match exactly."
- Hairstyle mannequins: "HAIRSTYLE mannequin. NO face. Use ONLY for hair structure."

**Per-image face-ignore labeling is critical.** Without explicit "IGNORE the face" on every non-identity reference, the model blends facial features from mannequins, expression refs, and clothing refs into the output.

### Ref Selection Per Generation (Critical)

**3 identity refs max + feature closeups.** More refs causes face averaging.

| Generation Type | Ref 1 | Ref 2 | Ref 3 | Ref 4 | Ref 5 |
|----------------|-------|-------|-------|-------|-------|
| **Headshots** | Real photo (angle-matched) | Real photo (face closeup) | Real photo (angle-matched) | Nose closeup | Teeth closeup |
| **Expressions** | Real photo (expression-matched) | Real photo (face closeup) | Real photo (expression-matched) | Nose closeup | Teeth closeup |
| **Looks (single-pass)** | Real photo (angle-matched) | AI headshot (pose-matched) | AI expression (expression-matched) | Nose closeup | Teeth closeup |
| **Looks (pass 2 hair swap)** | Pass 1 output (BASE) | Real photo | AI expression | Hairstyle mannequin | -- |
| **Showcase / batch** | AI headshot (pose-matched) | AI expression | Body profile | Hair texture swatch | Hairstyle mannequin |

**Showcase / batch generation requires all 5 refs.** Dropping the body profile produces wrong body proportions. Dropping the texture swatch produces wrong hair color (model copies the mannequin's brown instead of the subject's color). Both failures confirmed April 2026. For full-body showcase images, use 9:16 with explicit "head to feet, FULL BODY VISIBLE" language -- the model crops at the waist without it.

**Match refs to output angle and expression.** Profile refs for profile outputs, laughing refs for laughing expressions, etc. This is the single biggest factor in ref selection quality.

### Physical Realism

Always describe physics for long, flowing, or draped elements:
- "Hangs straight down obeying gravity, no curling or flipping at the ends"
- "One single continuous flow from [origin] to [destination], no breaks, no gaps, no disconnected sections"
- "Real physical weight pulling it toward the ground"
- "Same length on all sides -- symmetrical, no lopsided differences"

Without these, the model defaults to stylized rendering -- elements curling up, floating unnaturally, or appearing as disconnected decorative additions.

### When to Drop the Style Reference

For stubborn identity failures or dramatic body size changes, **describe the scene entirely in text** and skip the style reference image. This eliminates face contamination completely. Works best when the scene can be described clearly in words (specific pose, outfit, setting).

---

## Step 6: Two-Pass Workflow

The most reliable pattern for dramatic hairstyle changes and complex requirements. Essential when single-pass generation causes identity drift.

### When to Use Two-Pass

- Dramatic hairstyle changes (braids, curls, updos, bobs -- anything that significantly changes face framing)
- Any library hairstyle that drifted on first pass
- Complex transformations combining multiple style changes
- Identity fix-ups on existing generated images

### Standard Two-Pass: Identity First, Then Hair Swap

**Pass 1: Generate identity-correct base.** Use 5 refs total: 1 real photo (angle-matched) + 1 AI headshot (pose-matched) + 1 expression profile + nose closeup + teeth closeup. Subject wears their natural/signature hair + the target outfit. No hairstyle mannequin. All image slots dedicated to identity. This pass MUST produce a recognizable face.

**Pass 2: Swap the hairstyle.** Feed the pass 1 output + identity refs + hairstyle mannequin. Frame as a "single targeted edit" not a full regeneration.

### Pass 2 Image Ordering (Critical -- Base Image FIRST)

**Put the pass 1 output as IMAGE 1** (strongest anchor position). The model treats it as the base to edit rather than one reference among many. This was the single biggest breakthrough for identity preservation during hair swaps.

```
IMAGE 1: Pass 1 output -- "This IS [Name]. BASE IMAGE. ONLY change hair."
IMAGE 2: Real photo -- "Confirm same face. Preserve it."
IMAGE 3: Expression profile -- "Same face. Preserve this expression."
IMAGE 4 (LAST): Hairstyle mannequin -- "NO face. Hair structure only."
```

No feature closeups needed in pass 2 -- the face is already correct from pass 1.

### Pass 2 Prompt Framing

Frame as a targeted edit, not a recreation:
- "You are performing a SINGLE, TARGETED EDIT on IMAGE 1"
- "IMAGE 1 shows [Name] -- her face is CORRECT and must NOT change"
- "THE ONLY EDIT: Replace her current hair with [description]"
- "EVERYTHING ELSE stays IDENTICAL to IMAGE 1"
- "This is a hair swap, not a regeneration"
- Use lower temperature (0.2) to reduce drift

### Identity Fix on Existing Images

When a generated image has the right outfit/pose/hair but wrong face, use it as a scene reference:
- Send 5 refs (3 identity + 2 feature closeups) FIRST
- Send the existing image as SCENE REFERENCE
- "Recreate this EXACT image but replace the face with [Name]'s real face from IMAGES 1-5. The face in the scene reference is WRONG -- fix it."

### Expression Profiles as Triple-Duty Refs

Expression profiles count as identity + expression + hair volume reference in a single image slot. Use them as one of your 3 identity refs to free up slots. Label: "IMAGE 5: This is ALSO [Name] showing the target expression. Her face MUST match this."

### Keep Pass 1 Bases

Save pass 1 outputs -- they're usable as standalone natural-hair images and as bases for future hair swaps with different styles.

### Common Two-Pass Sequences
- Pass 1: identity + signature hair + outfit -> Pass 2: swap to library hairstyle
- Pass 1: identity + signature hair + outfit -> Pass 2: swap to dramatic updo/braids/curls
- Identity fix: existing image as scene ref -> regenerate with correct face
- Pass 1: identity + pose -> Pass 2: adjust framing/crop

### Expected Identity Scores by Hair Change (April 2026, single subject)

- Natural hair down: 8-9/10
- Pulled back (bun, ponytail): 7-8/10
- Moderate change (bob, braids): 7/10
- Dramatic change (pixie, slicked back): 6-7/10

More hair framing the face = better identity scores. The model uses hair as part of its identity anchor.

### What Doesn't Work
- Putting identity refs before the base image in pass 2 (base must be IMAGE 1)
- High temperature (0.3+) in pass 2 (use 0.2 for edits)
- Including too many non-identity refs in pass 2 (max 1 hairstyle mannequin)
- Extreme expressions (laughing with eyes closed) during dramatic hair changes -- use the signature smile for maximum identity preservation

---

## Step 7: Composites (Subject into Existing Photos)

Put the subject into an existing photograph, replacing the original model.

**Process:**
1. Select angle-matched headshots for the target pose
2. Include body profile if body accuracy matters
3. Include texture swatch if preserving natural coloring
4. Source photo goes LAST
5. Prompt: "Recreate IMAGE X exactly -- same pose, framing, outfit, setting, lighting -- but with [Name] as the subject."

**When subject proportions differ from the style reference:** Drop the source photo entirely. Describe the scene, pose, outfit, and setting in text only. The source photo creates compositing artifacts when proportions don't match.

**Multi-era composites:** Generate the same source photo 2-3 times using different era identity assets. Produces side-by-side comparison sets showing the same scene at different points in life.

---

## Step 8: 4K Upscaling

Upscale finished 2K images to 4K for final delivery.

**Process:**
1. Send the 2K image + one angle-matched headshot for face accuracy
2. Ask to "reproduce at maximum resolution"
3. Set `imageSize: "4k"` in generation config

**Per-image quality direction is critical.** One prompt doesn't fit all:

| Image Type | Quality Instruction |
|-----------|-------------------|
| Standard | "Sharp, crisp, vibrant natural colors. Ignore any softness or muted colors -- those are compression artifacts." |
| Sepia/vintage | "Preserve the sepia toning and vintage aesthetic. Increase detail within the vintage style." |
| Artistic depth of field | "Preserve background blur (intentional). Sharpen only the in-focus subject." |
| Dark/moody studio | "Preserve the dark atmosphere. Sharpen the subject against the dark background." |

**Always include:** "Do NOT change composition, pose, outfit, setting, or expression. Only improve resolution and detail."

---

## Troubleshooting

### AI beautification / idealization (nose, teeth, face shape, skin)
The model systematically narrows noses, straightens teeth, slims faces, and smooths skin. Gemini's own comparison of real vs AI confirmed these as the top differences. Countermeasures:
- **Feature close-up references are more effective than prompt language.** Crop tight close-ups of nose and teeth/smile. Send as dedicated refs. The model anchors to visual refs better than text descriptions of features.
- **One skin texture line in the prompt is sufficient.** "Natural skin texture with visible fine lines, not overly smoothed or airbrushed." Heavy anti-beautification blocks ("NO BEAUTIFICATION. NO SMOOTHING. NO IDEALIZATION.") don't score better than a single line. The refs do the heavy lifting.
- **Negative face constraints cause overcorrection.** "NOT narrow, NOT angular, NOT slim-jawed" produces a face that's too wide/square. Describe what the face IS (from Gemini text model analysis of real photos), not what it ISN'T.
- **Verify skin tone against real photos.** AI descriptions default to "olive-toned" or "warm complexion" even for fair/light-skinned subjects. An incorrect skin tone in the prompt is one of the biggest causes of "completely different person" output. Always verify the complexion description by looking at the actual photos.
- **Avoid trigger words.** Never use "beautiful," "flawless," "perfect," "stunning" anywhere in prompts.
- **Use identity refs at 1500px, quality 50.** Face detail at lower compression isn't enough for the model to anchor to nose shape and skin texture.
- **Use Gemini text model as QA.** Send a real photo + AI output to Gemini 2.5 Flash and ask for a scored comparison (1-10 per feature). Use this feedback to identify what drifted. Also use Gemini to write prompts -- it describes the subject's face more precisely than manual description.
- **3 refs, not 5-6.** More identity refs causes face averaging. The model blends features across all inputs instead of anchoring to one face. 3 carefully selected refs (angle-matched, face-dominant) outperform 5-6 varied refs.

### Face doesn't look like the subject
1. **Not enough identity refs.** Use 3-4 refs mixing real photos AND AI headshots. Real photos go FIRST. Two AI headshots alone drift ~20% of the time.
2. **Missing "IGNORE the face" labels.** Every non-identity reference (expression, clothing, hairstyle mannequin) needs explicit "IGNORE the face" in its label. Without this, the model blends features from all input images.
3. **Style ref face contamination.** Move style ref to LAST. If still failing, drop the style ref entirely and describe in text.
4. **Not enough angular coverage.** Generate more headshots at the specific angle needed.
5. **Elements obscuring face.** Add "face must be FULLY VISIBLE and not obscured."
6. **Wrong era refs.** Make sure headshots match the target era.

### Hair looks flat/sleek/lifeless
Mannequin hairstyle references are inherently flat. The model follows the mannequin too literally.
- Include a "volume reference" image -- the subject's signature expression profile (hair down with natural body) works well. Label: "HAIR VOLUME reference -- match this level of body and movement. IGNORE the face."
- Add explicit volume language in every prompt: "NATURAL VOLUME, BODY, MOVEMENT. NOT flat, NOT sleek, NOT plastered to the head."
- Use "If the hair looks flat or limp, the image has FAILED" as a hard constraint.
- For curly/wavy styles, include the matching texture swatch -- it shows the wave pattern the mannequin's smooth surface can't convey.

### Hair part on wrong side
- Verify part side from professional (non-selfie) photos -- selfies mirror.
- Use frame-relative language: "Part line on LEFT of the image" not "part on her left."
- If the model keeps getting it wrong, include a correct mannequin from another style as IMAGE 1 labeled "match the part side from this image."

### Body proportions wrong
- Include a body profile reference image
- Be specific about proportions rather than using vague descriptors
- Gemini tends to default toward average builds without explicit guidance
- **Iterate with targeted closeups.** First-attempt body profiles often underrepresent specific features. Send closeup real photos of the area that's wrong alongside the full-body refs on the next attempt.

### Mannequin creation from concept images (Grok defacing)
- When you have a good concept image (from Grok or other source) but need a mannequin version, ask Grok to "remove the face -- replace with smooth featureless blank surface, replace background with neutral grey." $0.02, ~5 seconds. Much faster than regenerating from scratch.
- Works well for body transformation profiles and fantasy hairstyles where the concept is right but you need the face removed for compositing.

### Element disconnected from body
- Add "one single continuous flow from [origin] to [destination], no breaks, no cuts, no disconnected sections"
- May need 2-3 attempts with increasingly explicit language about continuity

### Compositing/pasting artifacts
- If the subject's proportions differ significantly from a style reference, the model may paste the face onto the reference body. Drop the source photo and describe the scene in text instead.
- Use the no-style-ref approach when proportions don't match.

### Framing ignores full-body instruction
- Use 9:16 aspect ratio (gives vertical space)
- Make framing its own emphatic block with "FAILED" language
- Two-pass: generate normally, then reframe in a second call

### Content safety blocks
- Try `gemini-3.1-flash-image-preview` (looser threshold)
- Genericize the most revealing element
- **Content safety policies may apply differently to AI-generated vs. real photos.** Be aware that providers have different thresholds for different input types. Check each provider's acceptable use policy for your specific use case.

---

---

## Grok Batch Editing

For outfit changes, single-attribute edits, or style modifications on existing images, Grok Imagine is fast and cheap ($0.02/image).

**Batch pattern:** Upload image to R2/S3 for a public URL, send to Grok edit endpoint, download result, delete R2 file. This works reliably at scale (tested 21/21 in a batch).

```
for each image:
  1. sips compress -> upload to R2
  2. Grok POST /v1/images/edits with public URL
  3. Download result from imgen.x.ai URL (use User-Agent header)
  4. DELETE from R2
  5. sleep 1s
```

Grok handles AI-generated images well for outfit and style edits. See the xAI API documentation for details.

---

## Rate Limits and Batch Planning

**Gemini 3 Pro Image (Paid Tier 1):**
- 20 RPM (requests per minute) -- not usually the bottleneck
- **250 RPD (requests per day)** -- this is the hard ceiling
- 100K TPM (tokens per minute)

**Gemini 3.1 Flash Image (Paid Tier 1):**
- 100 RPM
- **1,000 RPD** -- 4x the daily headroom of Pro
- 200K TPM

**Planning large batches:**
- A full identity asset set (6 headshots + 6 textures + 8 body profiles + 5 expressions + 28 hairstyle mannequins + 8 clothing mannequins) = ~61 requests
- For batches over ~50 images: run sequentially with 2-3s delays, or split across Flash + Pro
- Running 3 parallel scripts causes 429 errors or silent stalls -- max 2 parallel (confirmed on both Pro and Flash, April 2026)

**Batch generation strategy (tested April 2026):**
- Generate in batches of 3-5 images, review between batches, fix failures before continuing
- Do NOT try to generate 25+ images in one unreviewed batch -- identity drift compounds and you waste API calls
- Natural-hair looks succeed on single pass (~90% hit rate with 5 refs: 3 identity + 2 feature closeups)
- Run Gemini 2.5 Flash QA spot-check after each batch (~$0.01-0.03 per check). Send real refs + generated image, score 1-10. For hairstyle-changed looks, tell QA to ignore hair. Flag below 7 for regeneration
- Dramatic hairstyle changes require two-pass (~80% hit rate with base-first ordering)
- Extreme expressions (laughing, eyes closed) during hairstyle changes are the hardest combo -- use signature smile instead
- Budget ~$0.04-0.06 per Pro image, ~$0.02 per Flash image. A full subject pipeline (61 assets + 35 final images) costs ~$5-6 total

**Pro vs Flash routing:**

| Task | Model | Why |
|------|-------|-----|
| Identity headshots | Flash | Comparable identity fidelity, 4x daily rate limit |
| Body profiles | Flash | Face is small at full-body scale |
| Expression profiles | Flash | Strong multi-constraint adherence |
| Final batch generation | Flash | Identity comparable, 1000 RPD headroom |
| Hairstyle mannequins | Flash | No face to preserve |
| Clothing mannequins | Flash | No face to preserve |
| Texture swatches | Flash | No face involved |
| 4K final renders | Pro | Pro needed for 4K output quality |
| Editorial/creative work | Pro | More creative latitude |

**Updated April 2026:** Flash 3.1 matches Pro on identity preservation in controlled testing (same scores on Gemini QA comparisons). Pro's advantage is 4K output and creative interpretation, not face fidelity. Default to Flash for everything except final 4K renders.

**Parallel execution:** 2 parallel scripts is the practical max. 3+ causes 429 errors and burns through RPD in minutes. For a 66-image batch: run 2 parallel + stagger the third, or run all sequential with 2-3s delays.

---

## Pose Variation

When generating a batch or showcase, vary the poses to make each image distinct rather than "standing facing camera" for every output.

**Standing:** power pose (hands on hips), casual (weight on one hip), editorial (chin up, strong expression), relaxed (hands clasped), angled body with face toward camera

**Action:** walking mid-stride, mid-turn with clothing flowing, brushing hair back, adjusting jewelry, reaching for something

**Seated/leaning:** sitting cross-legged, leaning against wall, chin on hand, perched on edge of chair

**Profile/angled:** looking over shoulder, three-quarter with eye contact, full profile

**Always angle-match headshots to the pose.** Use profile headshots for profile poses, front headshots for front-facing. This is the most common cause of identity drift in non-front-facing images.

---

## Onboarding a New Subject

Full identity asset pipeline (tested April 2026):

1. **Collect 15-25 reference photos** (~10 minutes). Include professional shots AND casual. Group by hairstyle and expression.
2. **Compress** identity refs to 1500px, quality 50 (~150-250KB). Non-identity refs to 1200px, quality 35 (~50-100KB). See Step 1.
3. **Generate identity headshots** on Flash -- 6 angles, hair pulled back (~5 minutes). Verify part side, verify likeness. Fix 3/4 angles with flipped-pose-reference technique if needed.
4. **Generate body profiles** on Flash -- 4 angles, faced + headless crops (~5 minutes)
5. **Generate texture swatches** on Flash -- 6 swatches covering color variations (~3 minutes)
6. **Generate expression profiles** on Flash -- 4-5 expressions, grouped by matching refs (~5 minutes)
7. **Generate subject-specific hairstyle mannequins** on Flash -- 4-7 styles from real refs, verify part side on each (~15 minutes)
8. **Generate clothing mannequins** on Flash -- 2-4 signature outfits (~5 minutes)
9. **Generate final images** on Flash in batches of 3-5, review + QA spot-check between batches:
   - Natural-hair looks: single pass, 5 refs (3 identity + 2 feature closeups), text for outfit (~90% hit rate)
   - Subject's own hairstyle variations: single pass with 1 mannequin ref (~80% hit rate)
   - Library hairstyle explorations: two-pass, base-first ordering (~80% hit rate)
   - Fix failures with identity-fix pass or full two-pass re-run

**Full pipeline: ~60 identity assets + 30-35 final images in ~2-3 hours, ~$5-6 total cost.**

**See also:** `STYLE-LIBRARY.md` in this repo for building and maintaining the reusable style library.

---

## Cost Reference

| Task | Model | Estimated Cost |
|------|-------|---------------|
| Single 2K image (5 refs) | Flash | ~$0.01-0.03 |
| Single 4K image (2 refs) | Pro | ~$0.05-0.10 |
| Two-pass image (2 API calls) | Pro | ~$0.08-0.12 |
| Full identity asset pipeline (~61 images) | Pro+Flash | ~$2.50-3.50 |
| Batch of 35 final images (mix of 1-pass and 2-pass) | Pro | ~$2.50-4.00 |
| Complete subject pipeline (assets + finals) | Pro+Flash | ~$5-7 |
| 4K upscale batch of 22 images | Pro | ~$1.50-2.50 |
| Veo morph video (Standard, ~8s) | Veo | ~$1.60 |
| Flash image (5 refs, 2K) | Flash | ~$0.01-0.03 |
| Grok single edit | Grok | $0.02 |
| Grok batch (21 edits) | Grok | ~$0.42 |
| Grok video (10s, 480p) | Grok | ~$0.25 |

---

## Quick-Start Checklist

**Identity Assets (once per subject per era):**
1. [ ] Collect 15-25 real photos (varied angles, expressions, hairstyles, full body)
2. [ ] Compress identity refs to 1500px/q50 (~150-250KB). Non-identity to 1200px/q35 (~50-100KB).
3. [ ] Verify hair part side from professional (non-selfie) photos
4. [ ] Generate 6 AI headshots on Flash (front, 3/4 L/R, profile L/R, above). Fix 3/4 angles if needed.
5. [ ] Verify headshots look like the subject
6. [ ] Generate body profiles on Flash (4 angles, headless + faced)
7. [ ] Generate 6 texture swatches on Flash (straight, wavy, curly + dark/highlight/auburn variations)
8. [ ] Generate 4-5 expression profiles on Flash (group refs by expression, hair down)
9. [ ] Generate 4-7 subject-specific hairstyle mannequins on Flash (from real refs, verify part side)
10. [ ] Generate 2-4 clothing mannequins on Flash (signature outfits)

**Generation (per batch of 3-5 images):**
11. [ ] Natural-hair looks: 5 refs (1 real photo + 1 AI headshot + 1 expression + nose closeup + teeth closeup), text for outfit
12. [ ] Hairstyle variations: 5 refs (3 identity + nose closeup + mannequin), text for outfit
13. [ ] Dramatic hairstyles: two-pass (identity-correct base first, then base-first hair swap at temp 0.2)
14. [ ] Review each batch. Fix identity drift with identity-fix pass or full two-pass.
15. [ ] Keep pass1 bases as standalone natural-hair images
16. [ ] 4K upscale finals with per-image quality direction
