---
name: image-based-ppt-generator
description: Create, revise, regenerate, or package image-based PowerPoint decks where each slide is a full-slide image. Use only when the user explicitly asks for an image-based PPT, image PPTX, image-generated deck, full-slide-image PowerPoint, or selected-page regeneration in that format; do not trigger for ordinary editable PPT creation.
---

# Image-Based PPT Generator

Use this skill to create a PPTX where every slide is a full-page raster image. The final deck prioritizes visual polish, consistent page design, and reliable full-slide packaging.

## Core Rules

- Separate content planning from image generation.
- Keep user intent and supplied materials authoritative for content, style, scope, and output format.
- Always use a clean prompt-writing subagent before final image generation. This prevents earlier drafts, rejected copy, and unrelated context from leaking into slide images.
- Use `spawn_agent` with `fork_context=false` for prompt writing. Pass only the approved or active task brief, extracted content, style requirements, slide count, and page-specific constraints.
- Limit the subagent to clean per-slide image prompts. Do not ask it to generate images, edit files, package PPTX files, or inspect generated outputs.
- Preserve factual content from supplied materials. Do not invent names, figures, dates, product claims, table rows, or status details.
- Create versioned outputs. Do not overwrite user-supplied files.

## Workflow

1. **Confirm the task contract**
   - Identify deck type, audience, slide count or page range, visual direction, output format, and version naming.
   - Identify content strictness: exact wording, faithful summarization, free creative copy, selected-page replacement, or visual-only redesign.
   - Require content confirmation before final image generation for factual, high-stakes, dense, table-heavy, or source-material-driven decks.
   - Skip content confirmation only when the user explicitly asks for direct generation or the deck is exploratory/creative.

2. **Prepare prompt inputs**
   - If input files exist, extract only the material needed for slide planning.
   - If the task is conceptual, write a compact brief with audience, story arc, slide count, and style direction.
   - Keep briefs, extracted notes, and prompts in a versioned workspace folder for traceability.

3. **Confirm or derive the style contract**
   - Use a user-provided style, reference image/deck, brand guidance, or prior accepted output when available.
   - If style is unspecified, propose 1-3 suitable style directions based on audience, subject, density, and formality, then use the approved option.
   - Keep the style contract generic and task-specific. Do not impose a default industry, organization, or report style.

4. **Draft prompts in a clean subagent**
   - Read `references/prompt-rules.md` before asking for prompts.
   - Ask the subagent to return:
     - `Slide NN`
     - `title`
     - `slide_text`
     - `visual_direction`
     - `prompt`
   - Review prompts before image generation. Remove stale context, process notes, hidden reasoning, unrelated constraints, and unsupported product UI claims.

5. **Generate full-slide images**
   - Use the built-in `image_gen` tool once per slide.
   - Generate each slide as a complete 16:9 final slide image with all visible text included.
   - Copy the newest generated image from `$CODEX_HOME/generated_images` into the project slide directory. Keep original generated files in place.
   - Use versioned folders, for example `image-based-ppt-v1/slides/slide-01.png` and `image-based-ppt-v1/prompts/slide-01.txt`.

6. **Package the PPTX**
   - Use `scripts/images_to_pptx.py` to place one image per slide.
   - Match a source deck's dimensions when iterating from an existing PPTX. Otherwise default to 16:9, `20 x 11.25` inches.
   - Save a versioned PPTX filename.

7. **QA before delivery**
   - Read `references/qa-checklist.md`.
   - Render the PPTX to PNG previews when a rendering path is available, or inspect a contact sheet of generated slide images.
   - Verify expected slide count, page order, no blank slides, no black bars or cropping, and one full-page image per slide.
   - For selected-page updates, copy accepted unchanged slide images into a new version folder, regenerate only requested pages, then repackage.

## Iteration Patterns

- **Selected-page fix:** Regenerate only the affected slide image, copy accepted unchanged images into a new version folder, and package a new PPTX.
- **Remove content:** Add direct negative constraints such as `Do not include <term> anywhere.` Do not rely only on omission from allowed text.
- **Style refresh:** Reuse the frozen content spec and regenerate all slide images with the revised style contract.
- **Content correction:** Regenerate the affected slide from corrected exact text. Do not patch text with native PPT boxes unless the user switches to a hybrid editable workflow.

## Useful Resources

- `references/prompt-rules.md`: prompt-writing rules, templates, and safety constraints.
- `references/qa-checklist.md`: final checks before delivery.
- `scripts/images_to_pptx.py`: package a directory of full-slide images into a PPTX.

## Final Response

Return:

- final PPTX absolute path
- preview or contact sheet absolute path
- concise list of generated or changed pages
- verification performed
- residual risks, especially image-model text fidelity for dense text or tables
