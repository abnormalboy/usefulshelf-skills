---
name: icon-sticker-workbench
description: 图标贴纸工作台（可折叠侧栏贴纸拼贴与 SVG 矢量导出画板）。支持多组矢量贴纸集合、指针拖动与键盘微调、改色/旋转/混排与干净 SVG 导出。Trigger when building collapsible sidebar sticker studios, SVG drag-and-drop canvas editors, vector collage builders, or creative workbench UIs.
---

# 图标贴纸工作台

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

侧栏从图标轨道展开为有标签的集合，选择集合只更换贴纸托盘，不删除画布现有对象。最多十二张贴纸各存形状、颜色、位置、角度；箭头键微移，Delete 删除，重绘后恢复选中对象焦点。拖动用 SVG 坐标，转换时计入 preserveAspectRatio 留白与缩放，使用 pointer capture。导出克隆 SVG，删除编辑框和编辑属性，保留文字呈现属性。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`-p/--params`（JSON 配置）、`--lang`（`en` 或 `zh`）、`--title`（1–2000 字符）、`--subtitle`（1–2000 字符）、`--accent`（3/6 位 HEX）、`--sticker-size`（JSON 键 `sticker_size`，整数 `48–140`，默认 `92`）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
- 文案用纯文本节点/框架插值，颜色只接受合法 HEX，数字按参考约束限制。内联 JSON 必须转义 `<`，不能把用户文字送进 innerHTML、eval、事件属性或选择器。
- 中英文文案使用明确字典，界面语言与用户输入内容分离；不得翻译或覆盖用户填写的名称、便笺或文件内容。
- 演示里的本地数据不代表实际账号、付款、远程 AI 或业务后台；要接入真实服务时遵守用户授权和服务端准入，不向访客公开凭据、运营额度或其他人的任务。

## 无障碍、响应式与生命周期

- 以原生 button、label、input、dialog 等语义元素承载操作，键盘路径与指针路径更新同一状态，保持可见焦点和必要的 aria-pressed/selected/expanded/current。
- 在 390px 及 320px 宽度验证控件可达、正文和主要操作不被裁切；布局优先 Grid 的 minmax(0,1fr)，媒体保持比例。
- 尊重 prefers-reduced-motion，静态状态仍可完成核心任务；页面隐藏或组件卸载时撤销 RAF、定时器、Observer 和监听器，释放 Blob URL。
- 用户请求导出时才生成文件；独立 HTML/SVG/PNG 必须包含所需本地样式和资源，不依赖 CDN、收费素材或缺失附件。

## 验收 / Acceptance

1. 按“核心实现规范”操作一次完整流程，检查状态、视觉反馈与实际输出一致，再重复操作和重置。
2. 只使用键盘完成选择、确认、关闭和恢复焦点；触屏纵向滚动不能被不相关的拖动拦截。
3. 切换语言和参数后，文本、可访问名称与导出反映当前设置，用户输入保持原样。
4. 断网、减少动态效果和窄屏均能使用；无后台循环、重复回调、失效下载或控制台异常。
5. 若需要外部业务集成，明确目前是本地演示的边界，再按项目接口接入；不假造成功状态。

## 实现骨架与默认 Token / Structure and default tokens

以下保留此效果的默认样式 Token 与主要可操作节点。将状态、几何和反馈规则接到这些节点；SVG / Canvas 的图形按核心规范绘制，控件名称可通过明确的中英字典切换。默认值可调整，不依赖外部字体或网络资源。

```css
:root {
  --accent: #f06c45;
  --ink: #1e2a24;
  --muted: #46554b;
  --rule: rgba(30, 42, 36, 0.14);
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", Inter, ui-sans-serif, system-ui, sans-serif;
  color: var(--ink);
  background: #eceae2;
}
```

```html
<main class="skill-demo">
  <h1>A tiny sticker studio.</h1>
  <p>Pick a collection. Make it yours. Take a little joy with you.</p>
  <button id="toggle" type="button">Collapse sidebar</button>
  <svg id="board" aria-label="Sticker composition. Select a sticker, then use arrow keys to move it."></svg>
  <input id="ink" aria-label="Sticker color" type="color">
  <button id="rotate" type="button">Rotate ↻</button>
  <button id="remove" type="button">Remove</button>
  <button id="shuffle" type="button">Mix it up</button>
  <button id="export" type="button">Export SVG ↗</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.


### Sidebar as a creative vinyl sticker studio

The collapsible sidebar (`224px` expanded / `68px` collapsed) transitions cleanly from an icon rail into labeled collections with elevated floating vector flyout sheets. Selecting a collection updates the active ink and tray stickers without clearing existing canvas stickers.

#### Die-cut vinyl stickers & cutting mat
Core tokens: `--accent: #f06c45`, `--ink: #1e2a24`, `--muted: #46554b`, desk `#eceae2`, cutting mat `#cad7c7` → `#bdccb9` with subtle metric grid rules framing the cream micro-dot sticker sheet (`#fffdf7`). Each of the nine illustrated stickers (`smile`, `flower`, `star`, `planet`, `cloud`, `bolt`, `leaf`, `heart`, `spark`) renders with a thick glossy white die-cut kiss-cut border, subtle hairline edge contour, crisp ink details, and a lifted `#vinyl-peel` drop shadow.

#### Manipulation & export
Up to twelve stickers each store shape, color, position and rotation. Quick vinyl color swatches supplement the native `#ink` color picker. Arrow keys nudge the focused sticker, Delete removes it, and focus is restored cleanly after re-rendering. Pointer dragging uses SVG coordinate math accounting for `preserveAspectRatio` letterboxing and pointer capture. Exporting clones `#board`, strips `.selection` and editor attributes (`tabindex`, `role`, `aria-*`, `data-id`), and produces a self-contained `1120 × 680` editable SVG.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
