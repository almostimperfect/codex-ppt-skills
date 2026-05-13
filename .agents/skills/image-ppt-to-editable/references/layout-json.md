# Visual Text Layout JSON

Use this shape for per-slide layout records. Coordinates should use source-image pixels unless noted otherwise.

```json
{
  "slide": 1,
  "image_size": {"width": 1920, "height": 1080},
  "items": [
    {
      "id": "title_1",
      "text": "Quarterly Update",
      "bbox": [96, 84, 560, 72],
      "role": "title",
      "group": "header",
      "style": {
        "font_size": 36,
        "bold": true,
        "color": "#172f52",
        "align": "left"
      }
    }
  ]
}
```

Guidelines:

- Keep table cells as separate items when the user is likely to edit them.
- Keep metric number, metric label, and metric sublabel separate.
- Use stable group names such as `summary_table`, `detail_table`, `metric_cluster_1`, `bottom_cards`.
- Record uncertain items with `"needs_review": true`.
