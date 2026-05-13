# Text Removal Prompts

## Default Prompt

Use in a clean context where the only visible image is the target slide:

```text
重绘这张图片，去掉所有文字信息，我要重新编辑文字。其他内容一点儿都不要改变。
```

For English-language operation, use the equivalent:

```text
Redraw this image and remove all text so I can edit the text again. Do not change anything else.
```

## Slightly Stricter Prompt

Use when the default prompt leaves residual text or changes non-text elements:

```text
只处理这张图片。重绘这张图片，去掉所有文字信息，我要重新编辑文字。其他内容一点儿都不要改变。
保留所有图标、表格线、色块、插图、背景、阴影和布局位置，不要新增或删除非文字元素。
```

For English-language operation, use the equivalent:

```text
Only process this image. Redraw it and remove all text so I can edit the text again. Do not change anything else.
Keep all icons, table lines, color blocks, illustrations, background details, shadows, and layout positions. Do not add or remove any non-text element.
```

## Failure Signals

Reject the generated textless background when:

- a new icon, card, or panel appears
- an existing icon, table, or illustration disappears
- table row/column geometry drifts enough that text overlay would not align
- readable text remains
- the slide is restyled instead of cleaned

If failures repeat, isolate harder: pass only the single target image to a fresh context/subagent.
