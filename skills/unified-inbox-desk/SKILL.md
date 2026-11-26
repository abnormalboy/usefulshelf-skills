---
name: unified-inbox-desk
description: 轻巧收件工作台（Unified inbox desk & channel analytics card）。以暖白窄卡、右侧深色问题气泡、五通道比例条为主构图，并可切换至筛选/搜索/草稿/归档撤销收件箱视图，适用于客服分析看板、多渠道消息中心与 SaaS 工作台。
---

# 轻巧收件工作台

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

默认以暖白窄卡、右侧深色问题气泡、回答和五通道比例条为主构图，收件箱是可切换的第二视图。7/30 天使用明确虚构的独立数据集，未解决=收到−解决，比例和一位小数来自同一数组；点击图例单独查看通道并恢复焦点。收件箱消息以稳定 ID 管理 pin/archive/edit/saved，存草稿只更新本页快照，撤销归档恢复最近项并清筛选。绝不把虚构分析说成真实 AI、真实账号或已发送消息。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`、`--subtitle`、`--accent`（3/6 位 HEX）、`--bg`（3/6 位 HEX）、`--lang`（`en` / `zh`）、`-p` / `--params`（JSON 配置文件）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --ink: #1b2620;
  --muted: #425148;
}
body {
  font: 14px/1.5 -apple-system, BlinkMacSystemFont, "PingFang SC", system-ui, sans-serif;
}
```

```html
<main class="skill-demo">
  <h1>把一天，理清楚。</h1>
  <p>问一个小问题，让每段对话都有清晰的去处。</p>
  <button id="analysis-view" type="button">洞察</button>
  <button id="desk-view" type="button">工作台</button>
  <select id="question-select" aria-label="换个问题"></select>
  <select id="period" aria-label="时间"></select>
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


### Luminous frosted-glass productivity console & inbox desk

Core tokens: `--accent: #557966`, `--bg: #f3f0e9`, `--ink: #1b2620`, `--muted: #425148`. The default view centers a hyper-crisp translucent frosted-glass window (`backdrop-filter: blur(28px) saturate(148%)`, specular hairline white rim, concentric `28px`/`16px` radii) floating over a silky multi-stop pastel aurora mesh backdrop.

#### Channel insights analytical model
Inside the frosted application card, macOS-style jewel window controls frame a right-aligned dark pill question bubble, concise plain-language summary, a `12px` jewel-toned segmented channel bar (`#e8a528`, `#2aa374`, `#3892d4`, `#6b6fd6`, `#d65683`), interactive `tabular-nums` legend rows, and elevated pill `<select>` query controls for topic (`volume`, `resolved`, `attention`) and period (`week`, `month`). Outstanding counts are `volume - resolved`; clicking any channel isolates its share and restores keyboard focus to the same row. All datasets are explicitly fictional and computed locally.

#### Inbox desk view & draft/archive semantics
Switching to `#desk-view` expands the frosted glass container into a two-column split inbox preserving insight state in memory. Filters, search, pinning, local-only draft saving/restoring, and undoable archiving operate on stable message IDs with one shared live status region (`#status`). Empty filter results hide `#reader`. Zero external resources, WCAG AA `>= 4.5:1` contrast throughout, and clean mobile reflow down to `390px`.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`（可加 `--lang zh` 或 `-p params.json`）。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
