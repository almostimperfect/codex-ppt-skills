# Codex PPT Skills

Version: v1.3

中文说明: [README.zh-CN.md](README.zh-CN.md)

This repository contains reusable Codex skills for creating and converting image-based PowerPoint decks.

## Skills

### `image-based-ppt-generator`

Create, revise, regenerate, or package image-based PPTX decks where each slide is a full-slide image.

Use this skill when a user explicitly asks for:

- an image-based PPT or PPTX
- an image-generated slide deck
- a full-slide-image PowerPoint
- selected-page regeneration for an image-based deck

Key behavior:

- separates content planning from image generation
- always uses a clean prompt-writing subagent before final image generation
- packages generated slide images into a PPTX
- supports selected-page regeneration and versioned outputs
- includes prompt rules and QA checks for image-model slide generation

### `image-ppt-to-editable`

Convert picture-only or image-based PPT decks into semi-editable PPTX files.

The converted deck uses:

- a textless slide image as the background layer
- editable PowerPoint text boxes reconstructed over the background

This is useful when an image-generated deck needs follow-up text editing while preserving the original visual style.

## Installation

Install with the skills CLI:

```bash
npx skills add almostimperfect/codex-ppt-skills -a codex -g
```

List available skills before installing:

```bash
npx skills add almostimperfect/codex-ppt-skills --list
```

Install specific skills:

```bash
npx skills add almostimperfect/codex-ppt-skills \
  --skill image-based-ppt-generator \
  --skill image-ppt-to-editable \
  -a codex -g
```

Restart or refresh Codex so the new skills are discovered.

## Repository Layout

```text
.agents/skills/
  image-based-ppt-generator/
    SKILL.md
    agents/openai.yaml
    references/
    scripts/images_to_pptx.py
  image-ppt-to-editable/
    SKILL.md
    agents/openai.yaml
    references/
    scripts/package_editable_layers.py
```

## Requirements

The bundled scripts are convenience helpers and reference implementations, not mandatory runtime requirements.

Codex can use them when their dependencies are already available, or follow the same packaging logic with equivalent tools in the user's environment, such as a Node.js PPTX library, direct Open XML PPTX generation, an available office tool, or another reliable PPTX writer.

Codex should not assume that installing these skills automatically installs Python packages. If a helper needs `python-pptx` and it is missing, package installation is an environment-specific option and may require user approval.

For environments where using the bundled Python helper is appropriate:

```bash
python3 -m pip install python-pptx
```

Image generation, image editing, slide rendering, and visual QA depend on the tools available in your Codex environment.

## Development Notes

- Keep skill instructions generic and shareable.
- Do not add local paths, non-public names, sensitive data, credentials, or generated deck content to skill files.
- Keep reusable implementation code in `scripts/`.
- Keep detailed prompt and QA guidance in `references/`.

## Changelog

### v1.3

- Strengthened `image-ppt-to-editable` so the required flow is visual layout extraction, per-slide textless background generation, editable text overlay, and rendered QA.
- Clarified that OCR is only a secondary check and that overlaying editable text on the original image is a failed conversion.

### v1.2

- Clarified that bundled scripts are convenience helpers and reference implementations, not hard runtime requirements.
- Clarified that Codex should adapt to locally available tools and should not assume skill installation automatically installs Python packages.

### v1.1

- Moved published skills from `.codex/skills/` to `.agents/skills/` for skills CLI discovery and local project testing.
- Updated installation instructions to use `npx skills add`.

### v1.0

- Added `image-based-ppt-generator`.
- Added `image-ppt-to-editable`.
- Added shared ignore rules for local and generated artifacts.
- Added MIT license.

## License

MIT. See [LICENSE](LICENSE).
