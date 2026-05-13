# QA Checklist

Before final response, verify:

- Output PPTX exists and has a versioned filename.
- Slide count matches the expected count.
- Deck dimensions match the source deck when a source deck exists.
- Each slide has exactly one full-slide image unless the user asked for a different structure.
- Rendered PNG previews or a contact sheet exists for visual inspection.
- No black bars, blank pages, wrong slide order, accidental cropping, or distorted images.
- The latest user-approved source or brief was used.
- User-deleted content was not restored.
- Forbidden terms, forbidden visual elements, and process wording are absent where visually inspectable.
- Known image-model text risks are disclosed when slides contain dense text, small labels, or tables.

Recommended final response fields:

- output path
- preview or contact sheet path
- generated or changed pages
- render or validation method
- inspection status
- unresolved issues or residual risks
