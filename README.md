# Codex PPT Skills

版本：v1.5

English documentation: [README.en.md](README.en.md)

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

将纯图片或图片版 PPT 转换成文字可编辑的 PPTX。

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

内置脚本是方便使用的辅助工具和参考实现，不是强制运行时依赖。

当依赖已经可用时，Codex 可以直接使用这些脚本；如果脚本无法运行，Codex 也可以参考脚本逻辑，改用用户环境中可用的等价方法继续完成打包，例如 Node.js PPTX 库、直接生成 Open XML PPTX、可用的办公软件工具，或其他可靠的 PPTX 写入方式。

安装这些 skills 不应被理解为会自动安装 Python 包。如果某个辅助脚本需要 `python-pptx` 但环境中没有，安装包只是一个与环境相关的可选方案，并且可能需要用户批准。

如果当前环境适合使用内置 Python 辅助脚本，可以执行：

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

### v1.5

- 将 `image-ppt-to-editable` skill 指令改为中文主导，同时保留 `layout JSON`、`textless background`、`editable text boxes` 等关键技术词。
- 将 `image-based-ppt-generator` skill 指令改为中文主导，并明确“可编辑版本/文字可编辑”请求应路由到 `image-ppt-to-editable`。
- 将中文说明设为默认 README，并将英文说明移动到 `README.en.md`。

### v1.4

- 扩展 `image-ppt-to-editable` 的触发措辞，覆盖“可编辑版本”“文字可编辑版本”“editable version”“text-editable PPT”等请求。
- 将对外表述调整为“可编辑/文字可编辑”。

### v1.3

- 强化 `image-ppt-to-editable`：必须按视觉版式识别、逐页生成无文字底图、叠加可编辑文字层、渲染 QA 的流程执行。
- 明确 OCR 只能作为辅助检查，直接在原图上叠加可编辑文字属于失败转换。

### v1.2

- 明确内置脚本是辅助工具和参考实现，不是硬性运行时依赖。
- 明确 Codex 应根据本地可用工具继续完成任务，且不应假设安装 skill 会自动安装 Python 包。

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
