---
name: tilted-tab-notebook
description: 倾斜标签笔记本（Tilted tab notebook & Markdown export workbench）。四个带物理倾角交互的原生标签使用 aria-selected/controls 与 roving tabindex，支持笔记书写、待办勾选与整本 Markdown 导出，适用于轻量笔记、任务工作台与多页标签切换。
---

# 倾斜标签笔记本

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

四个原生标签使用 aria-selected/controls 与单一 roving tabindex。hover/focus 只让相邻标签倾斜，内文字反倾斜，不改变选中页或内容；减少动态效果禁用倾斜。切页前保存当前字段，三页独立标题/笔记，计划页存稳定 ID 的文字与勾选值。提示动作追加而不覆盖笔记，计划提示先填输入框等用户确认。Markdown 导出包括全部页和完成标记，数据仅在内存，刷新清空。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`、`--accent`（3/6 位 HEX）、`--notebook`（1–160 字符）、`--tilt`（整数 0–15，标签倾角）、`--lang`（`en` / `zh`）、`-p` / `--params`（JSON 配置文件）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --panel: #212329;
  --desk: #111216;
  --ink: #efe9dc;
  --muted: #adb4c0;
  --aside-ink: #c8cdd6;
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", ui-sans-serif, system-ui, sans-serif;
  color: var(--ink);
  background: var(--desk);
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


### Luxury dark leather & warm brass index notebook

Core tokens: `--accent: #dbbe84`, `--tilt: 7deg`, `--desk: #111216`, `--panel: #212329`, `--ink: #efe9dc`, `--muted: #adb4c0`, `--aside-ink: #c8cdd6` (WCAG AA contrast `> 8.5:1` on `#212329`). The obsidian-espresso folio sits over subtle leather micro-grain texture with warm overhead lamp ambience.

#### Sculpted physical index tabs
Four native `[role="tab"]` buttons live inside a recessed binding rail with roving `tabindex`, `aria-selected`, and `aria-controls`. Each tab carries a delicate color-coded top edge strip (brass, emerald, periwinkle, rose), smooth concave lower corners when selected, and hover/focus elevation that skews immediate neighbors by `±--tilt` while counter-skewing inner labels upright. Mobile tabs maintain `>= 12px` legibility with zero horizontal overflow. Reduced motion disables all transitions and transforms.

#### Writing folio, bookplate & Markdown export
Three pages store independent editable titles and notes over warm brass-ruled writing lines (`line-height: 2`, aligned `2em` rule spacing); the Plans page manages stable-ID checkboxes with add/remove actions. The right margin houses a gold-foil embossed collector `EX LIBRIS` bookplate vignette, high-contrast aside guidance (`#page-aside`), and a `tabular-nums` total character counter. All state stays in memory and exports cleanly as one complete Markdown file with `- [x]` / `- [ ]` task markers via a revoked Blob URL.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`（可加 `--lang zh` 或 `-p params.json`）。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
