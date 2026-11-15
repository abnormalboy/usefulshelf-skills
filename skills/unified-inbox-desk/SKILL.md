---
name: unified-inbox-desk
description: 轻巧收件工作台。默认以暖白窄卡、右侧深色问题气泡、回答和五通道比例条为主构图，收件箱是可切换的第二视图。 Use for this interaction and adapt it to the existing framework.
---

# 轻巧收件工作台

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

默认以暖白窄卡、右侧深色问题气泡、回答和五通道比例条为主构图，收件箱是可切换的第二视图。7/30 天使用明确虚构的独立数据集，未解决=收到−解决，比例和一位小数来自同一数组；点击图例单独查看通道并恢复焦点。收件箱消息以稳定 ID 管理 pin/archive/edit/saved，存草稿只更新本页快照，撤销归档恢复最近项并清筛选。绝不把虚构分析说成真实 AI、真实账号或已发送消息。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`, `--subtitle`, `--accent`, `--bg`。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #557966;
  --bg: #f3f0e9;
  --ink: #2d382f;
}
body {
  font: 14px system-ui;
}
```

```html
<main class="skill-demo">
  <h1>把一天，理清楚。</h1>
  <p>问一个小问题，让每段对话都有清晰的去处。</p>
  <button id="analysis-view" type="button">洞察</button>
  <button id="desk-view" type="button">工作台</button>
  <select id="question-select" aria-label="消息从哪里来？ 哪些已被处理？ 哪里需要关注？"></select>
  <select id="period" aria-label="最近 7 天 最近 30 天"></select>
  <input id="search" aria-label="搜索消息" type="search">
  <button id="pin" type="button">pin</button>
  <button id="archive" type="button">归档这条 ↗</button>
  <textarea id="draft" aria-label="draft"></textarea>
  <button id="restore" type="button">恢复已存草稿</button>
  <button id="save" type="button">保存草稿</button>
  <button id="undo" type="button">撤销归档</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Visual and analytical model

The default view is a narrow centered warm-white card over four soft color fields, with a large serif heading. Its inset application contains a right-aligned dark question bubble, a short answer and five channel rows below a proportional bar. Keep this object and hierarchy primary; the inbox desk is a separate optional view.

Questions select received, resolved or outstanding counts. The 7-day and 30-day periods use separate fixed fictional datasets. Outstanding counts are received minus resolved, and both bar widths and one-decimal percentages derive from the same selected array. Selecting a legend row isolates its visual share and reports its actual count; select it again to return to all channels. Each row rebuild restores focus to that same row. Do not imply that these sample reports analyze the five independent demonstration messages or contact a remote model. All computation is local.

The view switch keeps question, period, selected channel and inbox edits in memory. Both views share exactly one live status region. There is no automatic motion; reduced-motion disables bar transitions. On narrow screens controls wrap and the card remains within the viewport.

#### Inbox desk model

#### State model
Messages use stable IDs and hold channel, pin, archive, edited draft and saved draft state. Filters and search derive the visible list without changing source order. A pinned sort is stable, and selecting a new filter chooses the first matching message when the previous selection is outside the filter. Empty lists hide the reader rather than showing an unrelated conversation. Channel bars are computed from actual unarchived sample messages.

#### Draft and archive semantics
Typing updates only that message's in-page edit buffer. Save copies the edit into its saved snapshot. Restore saved draft deliberately replaces subsequent unsaved edits. No messages are sent and no account or backend is connected. All state is lost on reload; do not imply persistent storage. Archive pushes an ID to an undo stack; Undo restores the most recent item and resets filters/search so it is visible again.

#### Adaptation
Replace the `messages` data with your own sample content. Render message text with textContent; do not interpolate user replies as markup. Inputs have descriptive labels in both languages. The list becomes a bounded scroll region on mobile and the reader follows below. The interface has no automatic movement.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 templates/index.template.html，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
