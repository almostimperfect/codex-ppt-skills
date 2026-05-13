# Codex PPT Skills

版本：v1.1

本仓库包含两个可复用的 Codex skills，用于制作和转换图片版 PowerPoint。

## Skills

### `image-based-ppt-generator`

创建、修改、重新生成或打包图片版 PPTX。每一页幻灯片都是一张完整的整页图片。

适用场景：

- 用户明确要求图片版 PPT 或 PPTX
- 用户要求用图片生成模型制作整套幻灯片
- 用户要求 full-slide-image PowerPoint
- 用户要求对图片版 PPT 的部分页面重新生成

核心行为：

- 将内容规划和图片生成分离
- 在最终图片生成前，始终使用干净的 prompt-writing 子代理
- 将生成的整页幻灯片图片打包成 PPTX
- 支持指定页面重新生成和版本化输出
- 内置图片模型生成幻灯片的 prompt 规则和 QA 检查清单

### `image-ppt-to-editable`

将纯图片或图片版 PPT 转换成半可编辑 PPTX。

转换后的 PPT 结构是：

- 底层：去除文字后的幻灯片背景图片
- 上层：根据原图重建的可编辑 PowerPoint 文本框

这个 skill 适合在图片生成的 PPT 需要继续修改文字时使用，同时尽量保留原有视觉风格。

## 安装

使用 skills CLI 安装：

```bash
npx skills add almostimperfect/codex-ppt-skills -a codex -g
```

安装前查看可用 skills：

```bash
npx skills add almostimperfect/codex-ppt-skills --list
```

只安装指定 skills：

```bash
npx skills add almostimperfect/codex-ppt-skills \
  --skill image-based-ppt-generator \
  --skill image-ppt-to-editable \
  -a codex -g
```

然后重启或刷新 Codex，让新的 skills 被发现。

## 仓库结构

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

## 依赖

内置脚本使用 Python 和 `python-pptx`。

如果环境中还没有安装依赖，可以执行：

```bash
python3 -m pip install python-pptx
```

图片生成、图片编辑、幻灯片渲染和视觉 QA 取决于你的 Codex 环境中可用的工具。

## 开发说明

- 保持 skill 指令通用，适合共享。
- 不要把本地路径、未公开名称、敏感数据、凭据或生成内容提交到 skill 文件中。
- 可复用的实现代码放在 `scripts/`。
- 详细 prompt 和 QA 规则放在 `references/`。

## 更新记录

### v1.1

- 将发布用 skills 从 `.codex/skills/` 移动到 `.agents/skills/`，便于 skills CLI 发现，也便于本地项目级测试。
- 将安装说明更新为 `npx skills add`。

### v1.0

- 添加 `image-based-ppt-generator`。
- 添加 `image-ppt-to-editable`。
- 添加本地和生成产物的忽略规则。
- 添加 MIT 协议。

## 许可证

MIT。见 [LICENSE](LICENSE)。
