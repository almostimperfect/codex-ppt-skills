# Codex PPT Skills

Version: v1.0

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

Copy the skill folders into your Codex skills directory:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R .codex/skills/image-based-ppt-generator "${CODEX_HOME:-$HOME/.codex}/skills/"
cp -R .codex/skills/image-ppt-to-editable "${CODEX_HOME:-$HOME/.codex}/skills/"
```

Restart or refresh Codex so the new skills are discovered.

## Repository Layout

```text
.codex/skills/
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

The bundled scripts use Python and `python-pptx`.

Install the Python dependency if it is not already available:

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

### v1.0

- Added `image-based-ppt-generator`.
- Added `image-ppt-to-editable`.
- Added shared ignore rules for local and generated artifacts.
- Added MIT license.

## License

MIT. See [LICENSE](LICENSE).
