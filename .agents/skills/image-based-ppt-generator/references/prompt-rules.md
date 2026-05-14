# Prompt Rules

Version: v1.1

## Clean Prompt Subagent

Use this structure when asking a subagent for prompts:

```text
You only write clean image-generation prompts for an image-based PowerPoint deck. Do not generate images, edit files, package files, or inspect outputs.

Inputs:
<brief, extracted notes, input paths, or page-level requirements>

Task:
Create <N> slide prompts for a presentation deck.

Rules:
1. Follow the current task contract for content strictness, slide count, style, and page-specific changes.
2. Keep prompts clean: no process notes, no "the user said", no old-version references, no hidden reasoning.
3. Make each prompt self-contained so it can be sent directly to an image model.
4. Each prompt must start with:
   Create a 16:9 PowerPoint slide as a single polished final slide image, all text included, Chinese text crisp and readable.
5. Do not propose rendering the final slide with Python, PIL, SVG, HTML/CSS, canvas, matplotlib, or other programmatic drawing. The prompt is for direct image-model generation.
6. Output each slide as:
   Slide NN
   title:
   slide_text:
   visual_direction:
   prompt:
```

## Full-Slide Generation Template

Use one prompt per slide.

```text
Create a 16:9 PowerPoint slide as a single polished final slide image, all text included, Chinese text crisp and readable.
Generate the complete slide image directly with all text included.
Preserve the exact text, product names, numbers, dates, and table rows below.
Do not describe or imply a programmatic drawing workflow. This prompt will be sent directly to an image model.

Style contract:
<paste the user-approved style contract or reference-derived style. Do not invent one if confirmation is required.>

Slide exact content:
<paste exact slide content or approved slide brief>

Design emphasis:
<main message, metric, diagram, comparison, table, or visual focus>
```

## Style Contract

Use one of these sources, in priority order:

1. User-provided style instructions.
2. User-provided reference deck or image.
3. Prior accepted generated slides.
4. A task-specific style option approved by the user.

If the user has not supplied a style and confirmation is required, propose 1-3 options that differ meaningfully in density, formality, color, and visual energy.

## Real Product Safety

For real products, avoid invented UI:

```text
Do not show any product webpage, dashboard screenshot, browser window, real UI page, or fake product interface.
Use an abstract capability architecture diagram made of neutral modules, nodes, and connecting lines.
```

Only show a real product page when a verified screenshot, official product page asset, or user-approved mockup instruction is supplied.

## Dense Content

Prefer slide-friendly structures over long tables:

- three-card comparison
- layered architecture diagram
- capability map
- left-to-right flow
- two-column matrix
- decision framework

Use a dense table only when exact table preservation is required.

For dense table slides, add:

```text
The table must dominate the slide and remain readable.
Preserve every row and every column exactly.
Do not merge rows. Do not omit rows.
Do not invent extra labels, amounts, names, or statuses.
Use clear header bars and light row separation.
Use large enough typography for visual inspection. If the content is too dense for one readable slide, prioritize faithful readable layout over decorative density.
```

## Regeneration Fixes

When removing or replacing content on specific pages, regenerate only those pages and add direct negative constraints:

```text
Do not include <removed label> anywhere.
Do not include <forbidden visual element> anywhere.
```

Do not rely only on omission from the allowed text list; image models may infer familiar labels from surrounding context.

## Targeted Slide Edit

Use this pattern when editing a generated slide image:

```text
Edit the visible PowerPoint slide image.
Change only: <specific target>.
Preserve everything else exactly: all other text, numbers, tables, icons, colors, background, and layout.
Do not add new text. Do not move other elements.
Output a complete 16:9 slide image.
```

## Revision Notes

- v1.1: Added explicit direct image-generation constraints and blocked programmatic final rendering as a fallback for dense text.
