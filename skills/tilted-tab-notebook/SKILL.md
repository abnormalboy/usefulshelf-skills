---
name: tilted-tab-notebook
description: 倾斜标签笔记本。四个原生标签使用 aria-selected/controls 与单一 roving tabindex。 Use for this interaction and adapt it to the existing framework.
---

# 倾斜标签笔记本

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

四个原生标签使用 aria-selected/controls 与单一 roving tabindex。hover/focus 只让相邻标签倾斜，内文字反倾斜，不改变选中页或内容；减少动态效果禁用倾斜。切页前保存当前字段，三页独立标题/笔记，计划页存稳定 ID 的文字与勾选值。提示动作追加而不覆盖笔记，计划提示先填输入框等用户确认。Markdown 导出包括全部页和完成标记，数据仅在内存，刷新清空。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`, `--accent`, `--notebook`, `--tilt`。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #dbbe84;
  --tilt: 7deg;
  --panel: #282a2e;
  font-family: ui-sans-serif, system-ui, sans-serif;
  color: #ece8de;
  background: #151618;
  color-scheme: dark;
}
```

```html
<main class="skill-demo">
  <h1>让小想法，探出一点头。</h1>
  <p>划过一个标签，邻居会轻轻让一让。每一页都能留下不同的想法，写完就把它们一起带走。</p>
  <button id="tab-0" type="button">◌ 随手记</button>
  <button id="tab-1" type="button">☑ 小计划</button>
  <button id="tab-2" type="button">✧ 灵感角</button>
  <button id="tab-3" type="button">♡ 小确幸</button>
  <input id="page-title" aria-label="页面标题" type="text">
  <textarea id="writing" aria-label="笔记正文"></textarea>
  <input id="task-text" aria-label="新计划" type="text">
  <button id="add-task" type="button">添加计划</button>
  <button id="prompt" type="button">给我一个开头 ↗</button>
  <button id="next" type="button">翻到下一页 →</button>
  <button id="export" type="button">导出整本 Markdown ↓</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Neighboring tabs and independent pages

The active tab shares the panel surface and has small concave lower corners. Hovering or focusing a tab raises it and gently skews its immediate neighbors away. Inner labels counter-skew to stay readable. This effect is visual only: it never changes selection or notebook content. Reduced motion removes both transitions and transforms.

Use a native button tablist with aria-selected, aria-controls and a single roving tabindex. Left and right arrows wrap through the four pages; Home and End jump to the first and last. Activation updates the panel's aria-labelledby. Keep page data separate from DOM controls and save the current fields before switching, including when the next-page shortcut is used.

Three pages hold independent editable titles and notes. The plans page stores objects with stable IDs, text and checked state. Adding, checking and removing updates the same model used for export. The starting-line action appends a prompt without replacing existing writing; on the plans page it places a suggestion into the new-plan input and waits for the user to add it. Note length and plan count are bounded to keep the interface usable.

All state is intentionally in memory. Explain that a refresh clears it and provide one Markdown download containing every page and checked task markers. Do not silently introduce browser storage: the page is intended to work inside opaque-origin sandboxed previews. The download uses a Blob URL, with allow-downloads needed in an iframe. Validate writing on two pages, switching back, keyboard tab movement, checkbox persistence, appending a prompt, and matching the exported text to the current notebook.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 templates/index.template.html，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
