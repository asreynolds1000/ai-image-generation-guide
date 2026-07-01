# AI Image Generation Guide

Practical techniques for generating photorealistic AI images of specific real people while preserving their identity. Covers the full workflow from collecting references through multi-provider generation, plus a reusable style library system using featureless mannequins.

**This is a work in progress** -- techniques and provider recommendations evolve as models update. Last verified April 2026.

> **Ethics & Consent:** This guide is intended for use with subjects who have given explicit consent to AI image generation in their likeness. Do not use these techniques to generate images of people without their knowledge or permission.

## What's Here

- **[PERSON-GENERATION.md](PERSON-GENERATION.md)** -- The full workflow: collecting reference photos, generating AI headshots for identity anchoring, body profiles, texture swatches, and final image generation with identity preservation. Includes provider comparison (Gemini, OpenAI, Grok), prompt templates, and troubleshooting.

- **[STYLE-LIBRARY.md](STYLE-LIBRARY.md)** -- How to build reusable, subject-independent style references (hairstyles, clothing) on featureless mannequins. Eliminates face contamination from style references. Includes the mannequin generation technique, multi-angle consistency, and a catalog of 35 hairstyle entries.

## Key Techniques

- **Mannequin-based style references** solve face contamination -- the biggest problem in multi-reference generation. Generate styles on featureless white mannequins, then combine with identity refs at generation time.
- **AI headshots as identity refs** resist style contamination better than real photos. Generate controlled headshots (multiple angles, hair pulled back, neutral background) as your primary identity references.
- **Image ordering matters.** Identity refs go first, style refs go last. The model weights earlier images more heavily.
- **3 face refs max.** More causes face averaging, not better accuracy.
- **Two-pass generation** for dramatic transformations: get identity right in pass 1, apply the style change in pass 2.

## Provider Comparison (April 2026)

| Provider | Best For | Limitation |
|----------|----------|------------|
| Gemini 3.1 Flash Image | Default for most work. Strong identity, 1000 RPD. | Slightly less creative than Pro |
| Gemini 3 Pro Image | 4K output, editorial work | 250 RPD limit |
| OpenAI gpt-image-1.5 | 16 reference images, native transparency | Max 1024x1536, tends to airbrush |
| Grok Imagine | Single-photo edits, $0.02/image | Single image input only |

## Cost

A complete subject pipeline (60+ identity assets + 30-35 final images) runs about $5-7 total on Gemini Flash.

## License

CC-BY-4.0
