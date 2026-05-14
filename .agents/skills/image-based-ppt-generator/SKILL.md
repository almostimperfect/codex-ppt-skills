---
name: image-based-ppt-generator
description: Create, revise, regenerate, or package image-based PowerPoint decks where each slide is a full-slide image. Use only when the user explicitly asks for an image-based PPT, image PPTX, image-generated deck, full-slide-image PowerPoint, or selected-page regeneration in that format; do not trigger for ordinary editable PPT creation.
---

# Image-Based PPT Generator

Use this skill to create a PPTX where every slide is a full-page raster image. The final deck prioritizes visual polish, consistent page design, and reliable full-slide packaging.

Version: v1.2

## Core Rules

- Separate content planning from image generation.
- Treat the image model as the final slide renderer. The expected path is to generate complete slide images directly, then inspect and regenerate or edit images as needed.
- Do not replace image generation with programmatic slide drawing. Python, PIL, SVG, HTML/CSS screenshots, canvas, or plotting libraries may support extraction, contact sheets, previews, QA, or PPTX packaging, but must not be used as the primary method to render final slide pages unless the user explicitly asks for programmatic rendering or the task switches away from image-generated PPT.
- Dense Chinese text, numbers, tables, and labels are reasons to write stricter prompts, use larger typography, simplify layout density, inspect outputs carefully, and regenerate flawed pages. They are not by themselves a reason to bypass `image_gen`.
- Keep user intent and supplied materials authoritative for content, style, scope, and output format.
- Always use a clean prompt-writing subagent before final image generation. This prevents earlier drafts, rejected copy, and unrelated context from leaking into slide images.
- Use `spawn_agent` with `fork_context=false` for prompt writing. Pass only the approved or active task brief, extracted content, style requirements, slide count, and page-specific constraints.
- Limit the subagent to clean per-slide image prompts. Do not ask it to generate images, edit files, package PPTX files, or inspect generated outputs.
- Preserve factual content from supplied materials. Do not invent names, figures, dates, product claims, table rows, or status details.
- Create versioned outputs. Do not overwrite user-supplied files.
- Treat bundled scripts as convenience helpers and reference implementations, not mandatory runtime requirements. If a script cannot run in the user's environment, continue with an equivalent local method that preserves the same output contract.

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
   - Do not create final slide images by drawing text, cards, tables, charts, or layouts with PIL, matplotlib, SVG, HTML/CSS, canvas, or similar deterministic renderers. Those tools are allowed only for auxiliary artifacts such as source extraction, contact sheets, QA annotations, or packaging helpers.
   - If exact text fidelity is critical, reduce visual density, make the text larger, split content across more slides when allowed, or regenerate/edit the image after QA. Do not silently downgrade to programmatic rendering.
   - Copy the newest generated image from `$CODEX_HOME/generated_images` into the project slide directory. Keep original generated files in place.
   - Use versioned folders, for example `image-based-ppt-v1/slides/slide-01.png` and `image-based-ppt-v1/prompts/slide-01.txt`.

6. **Package the PPTX**
   - Prefer `scripts/images_to_pptx.py` when Python and its required packages are already available.
   - If the helper cannot run, read it as a reference for the required behavior and continue with an equivalent packaging method available in the environment, such as a direct Open XML PPTX zip, a Node.js PPTX library, a locally available office tool, or another reliable PPTX writer.
   - Do not stop only because a convenience script is missing a dependency. Install missing packages only when appropriate for the environment and after any required user approval.
   - Match a source deck's dimensions when iterating from an existing PPTX. Otherwise default to 16:9, `20 x 11.25` inches.
   - Save a versioned PPTX filename.

7. **QA before delivery**
   - Read `references/qa-checklist.md`.
   - Render the PPTX to PNG previews when a rendering path is available, or inspect a contact sheet of generated slide images.
   - Verify expected slide count, page order, no blank slides, no black bars or cropping, and one full-page image per slide.
   - Inspect text-heavy slides for wrong characters, missing labels, invented values, and unreadable table cells. Regenerate or targeted-edit affected slides when defects are found.
   - For selected-page updates, copy accepted unchanged slide images into a new version folder, regenerate only requested pages, then repackage.

## Iteration Patterns

- **Selected-page fix:** Regenerate only the affected slide image, copy accepted unchanged images into a new version folder, and package a new PPTX.
- **Remove content:** Add direct negative constraints such as `Do not include <term> anywhere.` Do not rely only on omission from allowed text.
- **Style refresh:** Reuse the frozen content spec and regenerate all slide images with the revised style contract.
- **Content correction:** Regenerate the affected slide from corrected exact text. Do not patch text with native PPT boxes unless the user switches to a hybrid editable workflow.
- **Text fidelity failure:** Revise the prompt and regenerate or use image editing on the affected full-slide image. Prefer larger text, fewer rows per slide, clearer table hierarchy, and explicit exact-text blocks. Do not switch to programmatic final rendering unless the user approves that change.

## Non-Goals

- This is not a PIL/matplotlib/HTML-to-image deck generator. Programmatic rendering produces a different class of output and should not be presented as an image-generated PPT workflow.
- Do not claim that prompts were kept only for traceability if final slide images were actually drawn by code. If a non-image-generation path is used because the user asked for it, disclose that workflow explicitly.

## Useful Resources

- `references/prompt-rules.md`: prompt-writing rules, templates, and safety constraints.
- `references/qa-checklist.md`: final checks before delivery.
- `scripts/images_to_pptx.py`: convenience helper and reference implementation for packaging a directory of full-slide images into a PPTX.

## Final Response

Return:

- final PPTX absolute path
- preview or contact sheet absolute path
- concise list of generated or changed pages
- verification performed
- residual risks, especially image-model text fidelity for dense text or tables, plus any pages that were regenerated or still need user review

## Revision Notes

- v1.2: Clarified that bundled scripts are convenience helpers, not hard requirements; agents should adapt packaging to the user's available environment and only install dependencies when appropriate.
- v1.1: Clarified that `image_gen` is the primary renderer for final slide images; programmatic drawing is limited to auxiliary QA, previews, extraction, and packaging unless the user explicitly switches workflows.
