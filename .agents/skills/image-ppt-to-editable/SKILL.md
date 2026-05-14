---
name: image-ppt-to-editable
description: Convert image-based PowerPoint decks into semi-editable PPTX files by isolating each slide image, using AI visual understanding to recover text and layout, generating a textless version of each slide in a clean context, then rebuilding editable text boxes over the cleaned background. Use when the user wants a picture-only PPT, generated slide deck, or screenshot-based presentation converted into a layered PPT where text can be revised.
---

# Image PPT To Editable

Version: v1.2

## Purpose

Use this skill to convert a picture-only deck into a semi-editable deck:

- bottom layer: textless slide image background
- top layer: editable PowerPoint text boxes reconstructed from the original image

This is not a full vector reconstruction workflow. Preserve image style and make text editable; do not attempt to recreate every table line, icon, or illustration as native PPT shapes unless the user explicitly asks.

Bundled scripts are convenience helpers and reference implementations, not mandatory runtime requirements. If a script cannot run in the user's environment, continue with an equivalent local method that preserves the same output contract.

## Non-Negotiable Output Contract

The output must be a semi-editable reconstruction, not OCR text placed over the original slide image.

Required layers for every converted slide:

1. A full-slide background image where all original text has been removed.
2. Editable PowerPoint text boxes placed above that cleaned background.

Do not use the original slide image as the final background after extracting text. That creates duplicate/ghost text and fails the purpose of this skill.

Do not switch to an OCR-first workflow. OCR may be used only as a secondary cross-check for missed text after Agent visual understanding has produced the layout. The source of truth is the Agent's visual interpretation of the original slide, including semantic role, grouping, position, and style.

If no image editing/generation path is available to create a textless background, stop and report the blocker. Do not silently degrade to a PPTX with editable text over the original image.

## Core Principle

Do the text-removal image edit in an isolated context per slide. Do not ask the image model to edit a target slide from a long thread containing many related slide previews. Long multimodal context can cause the model to borrow layout from other images and add or remove non-text elements.

Preferred isolation methods:

- Use a subagent or clean child context for exactly one slide image.
- Use a clean work item containing only the target slide image and the short text-removal prompt.
- If subagents are unavailable, start a minimal local iteration where the target image is the only visible image immediately before image generation.

The main agent owns orchestration, validation, and final packaging. Isolated workers only produce one textless background image at a time.

## Workflow

1. **Intake**
   - Identify the source PPTX or slide images.
   - If the source is PPTX, extract or render each slide to a full-page PNG.
   - Keep page size and slide order.

2. **Visual Text Understanding**
   - Use Agent visual understanding on the original target slide image.
   - Extract a structured text layout from visual inspection, not raw OCR:
     - text
     - approximate bounding box
     - role: title, subtitle, metric, table_header, table_cell, card_title, card_body, icon_label
     - grouping: table, metric cluster, card group, flow step
     - style hints: size, color, weight, alignment
   - Capture text as editable content only; table lines, icons, illustrations, cards, and decorative shapes remain part of the background image.
   - Traditional OCR is not the workflow. Use it only, if useful, to check for missed small text or numeric labels after the visual layout is drafted.

3. **Generate Textless Background**
   - Run image generation/editing in a clean per-slide context.
   - Use the concise prompt in `references/text-removal-prompts.md` as the default.
   - Preserve all non-text visual elements: icons, table lines, cards, illustrations, gradients, shadows, colors, and layout.
   - Remove all readable text in any language, including digits, percentages, labels, table contents, and small captions.
   - Produce and keep a separate file such as `textless/slide-01.png`; never overwrite the extracted original image.

4. **Validate Textless Background**
   - Compare original and textless images visually.
   - Pass only if:
     - no readable text remains
     - no new non-text elements were added
     - no important icons or illustrations were removed
     - table and card structure stayed close enough for text overlay
     - slide size/aspect ratio is unchanged
   - If validation fails, retry the textless background in a clean context with a slightly stricter prompt. Do not keep retrying blindly; report persistent failures.

5. **Rebuild Editable Text Layer**
   - Place the textless background as a full-slide image.
   - Add PowerPoint text boxes from the visual text layout.
   - Before packaging, confirm that the background image path points to the cleaned/textless image, not the original extracted slide image.
   - Prefer `scripts/package_editable_layers.py` when Python and `python-pptx` are already available.
   - If the helper cannot run, read it as a reference for the expected layout behavior and continue with an equivalent method available in the environment, such as a Node.js PPTX library, a locally available office tool, direct Open XML generation, or another reliable PPTX writer.
   - Do not stop only because a convenience script is missing a dependency. Install missing packages only when appropriate for the environment and after any required user approval.
   - Use the source deck size when available; otherwise use a standard 16:9 widescreen slide.
   - Approximate font size, color, boldness, and alignment.
   - Set language-appropriate fonts in the layout JSON when needed; rely on script defaults only as fallbacks.
   - Tables can remain image backgrounds; put editable table cell text on top.
   - Keep text boxes simple and easy to edit. Prefer one text box per meaningful line/cell over one giant text box.

6. **Render And QA**
   - Render the rebuilt PPTX to PNG previews.
   - Inspect for:
     - text/background misalignment
     - duplicate text or residual ghost text
     - clipped or overlapping text boxes
     - missing rows, labels, or metrics
     - accidental use of the original slide image as the background
   - When possible, compare against the original image and produce a short issue list.
   - If duplicate text is visible, treat it as a failed conversion and regenerate the textless background or fix the packaging input. Do not deliver it as acceptable output.

7. **Deliver**
   - Return the semi-editable PPTX path.
   - Return preview paths and any per-slide known issues.
   - State clearly that the output is semi-editable: text is editable, but backgrounds, icons, tables, and illustrations remain images.

## Recommended POC Strategy

Before batch conversion, test the hardest slide first: usually a dense table or metric page.

1. Convert one slide.
2. Render the semi-editable result.
3. Ask the user whether the textless background and overlay quality are acceptable.
4. Batch the remaining slides only after the sample passes.

## Resources

- `references/text-removal-prompts.md`: prompts for per-slide textless background generation.
- `references/layout-json.md`: suggested structure for visual text layout extraction.
- `scripts/package_editable_layers.py`: convenience helper and reference implementation for packaging textless backgrounds plus layout JSON into a semi-editable PPTX.

## Revision Notes

- v1.2: Made the semi-editable reconstruction contract explicit: cleaned textless background is required, OCR is only a secondary check, and agents must not fall back to overlaying editable text on the original image.
- v1.1: Clarified that bundled scripts are convenience helpers, not hard requirements; agents should adapt packaging to available local tools and only install dependencies when appropriate.
