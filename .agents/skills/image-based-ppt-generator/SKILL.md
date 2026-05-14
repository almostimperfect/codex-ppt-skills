---
name: image-based-ppt-generator
description: 创建、修改、重新生成或打包图片版 PowerPoint/PPTX，每一页幻灯片都是一张完整的整页图片。适用于用户明确要求“图片版 PPT”“图片生成 PPT”“每页一张图的 PPT”“full-slide-image PowerPoint”“把这些图片打包成 PPTX”“重新生成某几页图片幻灯片”等场景。不要用于普通可编辑 PPT 制作；如果用户要求“可编辑版本”或“文字可编辑”，应使用 image-ppt-to-editable。
---

# Image-Based PPT Generator

Version: v1.3

## 用途

使用这个 skill 创建或修改图片版 PPTX：最终每一页都是一张整页 raster image。这个流程优先保证视觉完整度、页面设计一致性，以及可靠的整页图片打包。

## 核心规则

- 内容规划和图片生成必须分离。
- 把图像模型视为最终幻灯片渲染器。标准路径是直接生成完整幻灯片图片，然后检查、重新生成或局部编辑有问题的图片。
- 不要用程序化绘图替代图像生成。Python、PIL、SVG、HTML/CSS 截图、canvas、matplotlib 或其他绘图库可以用于内容提取、contact sheet、预览、QA 或 PPTX 打包，但不能作为最终页面的主要渲染方式，除非用户明确要求程序化渲染，或任务已经切换到非图片生成 PPT。
- 中文密集文本、数字、表格和标签意味着要写更严格的 prompt、使用更大字号、降低布局密度、仔细检查输出并重新生成有问题页面；这些问题本身不是绕过 `image_gen` 的理由。
- 用户意图和用户提供的材料是内容、风格、范围和输出格式的权威来源。
- 最终图片生成前必须使用干净的 prompt-writing subagent。这样可以避免旧草稿、已拒绝文案和无关上下文污染幻灯片图片。
- 使用 `spawn_agent` 且 `fork_context=false` 来写 prompt。只传入已批准或当前有效的任务 brief、提取内容、风格要求、页数和逐页约束。
- subagent 只负责返回干净的逐页图片 prompt。不要让它生成图片、编辑文件、打包 PPTX 或检查生成结果。
- 保留用户材料中的事实内容。不要编造名称、数字、日期、产品声明、表格行或状态信息。
- 创建 versioned outputs。不要覆盖用户提供的文件。
- 内置脚本只是辅助工具和参考实现，不是强制运行时依赖。如果脚本在用户环境中无法运行，继续使用可用的等价方法，但必须保持同样的输出契约。

## 工作流程

1. **确认任务契约**
   - 确认 deck 类型、受众、页数或页码范围、视觉方向、输出格式和版本命名。
   - 确认内容严格度：逐字保留、忠实总结、自由创作、指定页面替换，或纯视觉重设计。
   - 对事实性强、高风险、文本密集、表格密集或依赖源材料的 deck，在最终图片生成前要求内容确认。
   - 只有当用户明确要求直接生成，或 deck 是探索性/创意型时，才跳过内容确认。

2. **准备 prompt 输入**
   - 如果有输入文件，只提取 slide planning 所需材料。
   - 如果任务是概念型，写一个紧凑 brief，包含受众、叙事结构、页数和风格方向。
   - 将 brief、提取笔记和 prompts 保存在带版本号的工作目录中，便于追踪。

3. **确认或推导风格契约**
   - 优先使用用户提供的风格、参考图片/参考 deck、品牌规范或之前已接受的输出。
   - 如果没有指定风格，根据受众、主题、信息密度和正式程度提出 1-3 个合适方向，并使用用户批准的选项。
   - 风格契约要通用且贴合当前任务。不要强加默认行业、组织或报告风格。

4. **在干净 subagent 中起草 prompts**
   - 请求 prompt 前阅读 `references/prompt-rules.md`。
   - 要求 subagent 返回：
     - `Slide NN`
     - `title`
     - `slide_text`
     - `visual_direction`
     - `prompt`
   - 图片生成前审查 prompts。移除陈旧上下文、过程说明、隐藏推理、无关约束和未经支持的产品 UI 声明。

5. **生成整页幻灯片图片**
   - 每页使用一次内置 `image_gen` 工具。
   - 每张图都要是完整 16:9 最终幻灯片图片，并包含所有可见文字。
   - 不要用 PIL、matplotlib、SVG、HTML/CSS、canvas 或类似确定性渲染器绘制最终幻灯片上的文字、卡片、表格、图表或布局。这些工具只允许用于辅助产物，例如源材料提取、contact sheet、QA 标注或打包 helper。
   - 如果精确文字保真很重要，降低视觉密度、放大文字、在允许时拆成更多页，或 QA 后重新生成/编辑图片。不要静默降级为程序化渲染。
   - 将 `$CODEX_HOME/generated_images` 中最新生成的图片复制到项目 slides 目录。保留原始生成文件。
   - 使用版本化目录，例如 `image-based-ppt-v1/slides/slide-01.png` 和 `image-based-ppt-v1/prompts/slide-01.txt`。

6. **打包 PPTX**
   - 如果 Python 和所需包已可用，优先使用 `scripts/images_to_pptx.py`。
   - 如果 helper 无法运行，把它作为所需行为的参考，继续使用当前环境中可用的等价打包方法，例如直接生成 Open XML PPTX zip、Node.js PPTX 库、本地办公软件或其他可靠 PPTX writer。
   - 不要因为辅助脚本缺少依赖就停止。只有在环境适合且获得必要批准后，才安装缺失包。
   - 如果是在已有 PPTX 上迭代，匹配源 deck 尺寸。否则默认 16:9，`20 x 11.25` inches。
   - 保存带版本号的 PPTX 文件名。

7. **交付前 QA**
   - 阅读 `references/qa-checklist.md`。
   - 有渲染路径时，把 PPTX 渲染成 PNG 预览；否则检查生成图片的 contact sheet。
   - 验证页数、页序、没有空白页、没有黑边或裁切，并且每页只有一张整页图片。
   - 对文本密集页检查错字、缺失标签、编造数值和不可读表格单元格。发现缺陷时重新生成或定向编辑对应页面。
   - 对指定页面更新，先把已接受且不变的 slide images 复制到新版本目录，只重新生成请求页面，再重新打包。

## 迭代模式

- **指定页面修复**：只重新生成受影响的 slide image，把已接受且未变化的图片复制到新版本目录，然后打包新的 PPTX。
- **删除内容**：加入直接负约束，例如 `Do not include <term> anywhere.` 不要只依赖从允许文本中省略。
- **风格刷新**：复用冻结的内容 spec，用修改后的风格契约重新生成所有幻灯片图片。
- **内容修正**：根据修正后的精确文本重新生成受影响页面。不要用原生 PPT 文本框补丁，除非用户切换到 hybrid editable workflow。
- **文字保真失败**：修改 prompt 并重新生成，或对受影响整页图片做 image editing。优先使用更大文字、更少表格行、更清晰表格层级和明确 exact-text blocks。除非用户批准，不要切换到程序化最终渲染。

## 非目标

- 这不是 PIL/matplotlib/HTML-to-image deck generator。程序化渲染会产生另一类输出，不应伪装成图片生成 PPT workflow。
- 如果因为用户要求而使用了非图像生成路径，要明确披露该 workflow。不要声称 prompts 只是用于追踪，而实际最终幻灯片图片是代码画出来的。
- 这不是文字可编辑 PPT 转换流程。如果用户要求“可编辑版本”或“文字可编辑”，应改用 `image-ppt-to-editable`。

## 资源

- `references/prompt-rules.md`：prompt 写作规则、模板和安全约束。
- `references/qa-checklist.md`：交付前检查清单。
- `scripts/images_to_pptx.py`：把整页图片目录打包成 PPTX 的辅助脚本和参考实现。

## 最终回复

返回：

- 最终 PPTX 绝对路径
- 预览图或 contact sheet 绝对路径
- 生成或修改过的页面清单
- 已执行的验证
- 剩余风险，尤其是图片模型在密集文字或表格上的文字保真风险，以及哪些页面已重新生成或仍需用户检查

## 版本记录

- v1.3: 将 skill frontmatter 和正文改为中文主导，并明确“可编辑版本/文字可编辑”应路由到 `image-ppt-to-editable`。
- v1.2: 明确内置脚本只是辅助工具，不是硬依赖；Agent 应根据用户环境中的可用工具继续完成打包，只在合适时安装依赖。
- v1.1: 明确 `image_gen` 是最终幻灯片图片的主渲染器；除非用户明确切换 workflow，程序化绘图只能用于辅助 QA、预览、提取和打包。
