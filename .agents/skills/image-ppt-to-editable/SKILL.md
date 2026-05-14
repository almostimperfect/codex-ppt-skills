---
name: image-ppt-to-editable
description: 将图片版 PPT、纯图片 PPT、截图式幻灯片或图片生成的 PPT 转换成可编辑 PPTX，尤其适用于用户要求“可编辑版本”“文字可编辑”“图片版 PPT 转可编辑”“PPT 文字可编辑”“把 PPT 变成可改文字的版本”“editable version”“text-editable PPT”等场景。流程必须先逐页提取或渲染原图，用视觉理解识别文字和版式并输出 layout JSON，再为每页生成去除文字的 textless background，最后在清理后的底图上重建 editable text boxes。不要走 OCR-first，不要直接在原图上叠字。
---

# Image PPT To Editable

Version: v1.5

## 用途

使用这个 skill 将纯图片或图片版 PPT 转换成文字可编辑的 PPTX：

- 底层：去除文字后的整页图片底图，也就是 textless background
- 上层：根据原图重建的 PowerPoint 可编辑文本框，也就是 editable text boxes

这不是把图片 PPT 完全反编译成原生 PPT 形状的流程。目标是保留原始视觉风格，同时让文字可编辑；除非用户明确要求，不要尝试把每条表格线、图标或插图都重建成原生 PPT shape。

内置脚本只是辅助工具和参考实现，不是强制运行时依赖。如果脚本在用户环境中无法运行，继续使用可用的等价方法，但必须保持同样的输出契约。

## 不可变输出契约

最终输出必须是“文字可编辑”的重建版本，不是 OCR 识别文字后直接叠在原始幻灯片图片上。

每一页必须包含两层：

1. 一张整页底图，其中原始文字已经全部去除。
2. 叠加在清理后底图上的 PowerPoint 可编辑文本框。

不要把原始幻灯片图片作为最终背景再叠加文本框。这样会产生重复文字或残留文字，属于失败转换。

不要切换成 OCR-first 流程。OCR 只能作为辅助检查，用来发现视觉理解之后是否遗漏了小字或数字。真正的 source of truth 是 Agent 对原始页面的视觉理解，包括文本语义、分组、位置、字号、颜色、粗细和对齐。

如果当前环境没有任何可用的图像编辑或图像生成能力来创建 textless background，必须停止并报告 blocker。不要静默退化成“原图 + 可编辑文字层”的 PPTX。

## Hard Stop Checklist

创建任何可编辑 PPTX 之前，Agent 必须明确确认：

- 已识别源 deck 或逐页图片。
- 只打开了用户指定的源版本和必要工作文件；没有打开被用户禁止或无关的参考版本。
- 可执行逐页视觉理解 workflow。
- 有真实可用的图像编辑或图像生成路径来创建 textless backgrounds。
- OCR 不会作为 layout JSON 或 editable text boxes 的主要来源。
- 首个输出会是一个困难页 POC，除非用户明确要求直接批量处理。
- 最终 PPTX 打包路径会使用 textless backgrounds，而不是原始幻灯片图片。

如果任一项不满足，必须停止并报告 blocker。不要创建 fallback editable PPTX。

## Forbidden Fallbacks

- 不要创建“原始图片 + 可编辑文字叠加层”的 PPTX。
- 不要用 OCR 输出自动生成主要 layout JSON。
- 不要把本地 blur、mask、crop 或 inpaint 脚本当作 textless background 生成能力的替代品，除非每页经过视觉验证且没有任何可读文字残留。
- 不要在一个困难页 POC 通过视觉 QA 之前批量处理，除非用户明确要求直接批量处理。
- 不要交付背景中可见重复文字、ghost text 或可读文字残留的 PPTX。

## 核心原则

每页去文字底图都要在隔离上下文中单独生成。不要在包含多页幻灯片预览的长对话里让图像模型编辑某一页；长 multimodal context 容易让模型借用其他页面的版式，或者增删非文字元素。

优先使用这些隔离方式：

- 为每一页使用一个 subagent 或干净子上下文。
- 使用只包含目标页图片和短提示词的 clean work item。
- 如果没有 subagent，就在最小上下文里执行本地迭代，确保图像生成前唯一可见的图片就是目标页。

主 Agent 负责调度、验证和最终打包。隔离 worker 只负责一次生成一页 textless background。

## 工作流程

1. **输入整理**
   - 识别源文件是 PPTX 还是一组幻灯片图片。
   - 如果源文件是 PPTX，先把每页提取或渲染成整页 PNG。
   - 保持页面尺寸、宽高比和页序不变。

2. **视觉理解文字和版式**
   - 让 Agent 直接看原始目标页图片。
   - 从视觉理解中提取结构化 layout JSON，不要从 raw OCR 开始：
     - `text`
     - 近似 `bbox`
     - `role`：例如 `title`、`subtitle`、`metric`、`table_header`、`table_cell`、`card_title`、`card_body`、`icon_label`
     - `grouping`：例如表格、指标组、卡片组、流程步骤
     - `style hints`：字号、颜色、粗细、对齐
   - 只把文字作为可编辑内容记录下来；表格线、图标、插图、卡片、装饰形状都保留在图片底图里。
   - OCR 不是这个流程。只有在有帮助时，才用 OCR 辅助检查是否漏掉小字、数字或标签。

3. **生成 textless background**
   - 在干净的逐页上下文中运行图像生成或图像编辑。
   - 默认使用 `references/text-removal-prompts.md` 里的简短提示词。
   - 保留所有非文字视觉元素：图标、表格线、色块、插图、渐变、阴影、颜色和布局。
   - 移除所有可读文字，包括中文、英文、数字、百分比、标签、表格内容和小标题。
   - 输出并保留单独文件，例如 `textless/slide-01.png`；不要覆盖提取出来的原图。

4. **验证 textless background**
   - 视觉比较原图和去文字底图。
   - 只有满足这些条件才通过：
     - 没有可读文字残留
     - 没有新增非文字元素
     - 没有删除重要图标或插图
     - 表格和卡片结构足够接近，便于后续文字叠加
     - 页面尺寸和宽高比不变
   - 如果验证失败，用更严格的提示词在干净上下文中重试。不要盲目无限重试；持续失败时报告问题。

5. **重建可编辑文字层**
   - 把 textless background 作为整页底图放入 PPT。
   - 根据 layout JSON 添加 PowerPoint 文本框。
   - 打包前确认 background 路径指向清理后的 textless 图片，而不是原始提取图片。
   - 如果 Python 和 `python-pptx` 已可用，优先使用 `scripts/package_editable_layers.py`。
   - 如果辅助脚本无法运行，把它作为布局行为参考，继续使用当前环境中可用的等价方法，例如 Node.js PPTX 库、本地办公软件、直接生成 Open XML，或其他可靠 PPTX 写入方式。
   - 不要因为辅助脚本缺少依赖就停止。只有在环境适合且获得必要批准后，才安装缺失包。
   - 有源 PPTX 时沿用源页面尺寸；否则使用标准 16:9 宽屏。
   - 近似还原字号、颜色、粗细和对齐。
   - 需要中文或其他语言字体时，在 layout JSON 中设置合适字体；脚本默认字体只能作为 fallback。
   - 表格可以保留为图片底图；把可编辑的表格单元格文字叠加在上面。
   - 文本框保持简单、便于编辑。优先一行或一个单元格一个文本框，不要把整页内容塞进一个巨大的文本框。

6. **渲染和 QA**
   - 把重建后的 PPTX 渲染成 PNG 预览。
   - 检查：
     - 文字和背景是否错位
     - 是否有重复文字或残留 ghost text
     - 文本框是否裁切或重叠
     - 是否漏掉行、标签或指标
     - 是否误用了原始幻灯片图片作为背景
   - 尽可能与原图对比，并输出简短问题清单。
   - 如果能看到重复文字，视为转换失败。需要重新生成 textless background 或修正打包输入，不要把这种结果交付为可接受输出。

7. **交付**
   - 返回文字可编辑 PPTX 路径。
   - 返回预览图路径和每页已知问题。
   - 明确说明：文字可编辑；背景、图标、表格线和插图仍然是图片。

## 默认 POC 策略

默认情况下，批量转换前必须先测最难的一页，通常是密集表格页或指标页。只有用户明确要求直接批量处理时，才可以跳过 POC。

1. 先转换一页。
2. 渲染文字可编辑结果。
3. 让用户确认 textless background 和文字叠加质量是否可接受。
4. 样例通过后再批量处理剩余页面。

## 资源

- `references/text-removal-prompts.md`：逐页生成 textless background 的提示词。
- `references/layout-json.md`：视觉理解文字版式时推荐的 layout JSON 结构。
- `scripts/package_editable_layers.py`：把 textless backgrounds 和 layout JSON 打包成文字可编辑 PPTX 的辅助脚本和参考实现。

## 版本记录

- v1.5: 增加 Hard Stop Checklist 和 Forbidden Fallbacks，并将 POC 策略改为默认强制流程，防止 OCR-first、原图叠字和未验证批量处理。
- v1.4: 将 skill frontmatter 和正文改为中文主导，并保留关键英文技术词，提升中文“可编辑版本/文字可编辑”请求的触发和执行稳定性。
- v1.3: 扩展 “editable version”“text-editable” 及中文等价说法的触发措辞。
- v1.2: 明确文字可编辑重建契约：必须使用去文字底图，OCR 只能辅助检查，不能退化为在原图上叠加可编辑文字。
- v1.1: 明确内置脚本只是辅助工具，不是硬依赖；Agent 应根据本地可用工具继续完成打包，只在合适时安装依赖。
