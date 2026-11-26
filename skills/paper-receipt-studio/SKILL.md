---
name: paper-receipt-studio
description: 纸条收据工坊（Tactile paper receipt studio & scrunchable PNG exporter）。离屏 Canvas 绘制真实收据纹理，屏上用 12×18 三角网格与局部正弦折痕呈现揉纸与抚平质感，支持自定义店名/币种/印章/条目与高清 PNG 导出，适用于生活手账、消费小票与品牌纪念卡片。
---

# 纸条收据工坊

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

离屏 Canvas 绘制真实收据纹理，屏上用 12×18 三角网格和局部正弦折痕映射纸张，明确不是布料物理。金额用整数分累加，1–5 行、每行名称最多 32 字、价格 0–9999.99；无效编辑禁用导出并保留最近有效纹理。拖动使用 pointer capture，松开/取消后有界衰减，另有揉皱和抚平按钮。PNG 导出平整离屏纹理，不能导出难读的折叠帧。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`、`--subtitle`、`--accent`（3/6 位 HEX）、`--bg`（3/6 位 HEX）、`--store-name`（1–40 字符）、`--currency`（1–6 字符，如 `USD`/`CNY`/`EUR`）、`--stamp`（1–32 字符）、`--lang`（`en` / `zh`）、`-p` / `--params`（JSON 配置文件，支持 `storeName`、`currency`、`stamp` 及 `items` 数组：1–5 项 `{name, price}`）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #47765d;
  --bg: #f4f1e9;
  --ink: #303b32;
}
body {
  font: 14px system-ui, sans-serif;
}
```

```html
<main class="skill-demo">
  <h1>Small things, worth keeping.</h1>
  <p>Make a tiny receipt for a lovely day. Scrunch it, smooth it, keep it.</p>
  <button id="add" type="button">+ Add a little thing</button>
  <button id="download" type="button">Keep as PNG ↓</button>
  <button id="fold" type="button">Scrunch paper</button>
  <button id="smooth" type="button">Smooth it out</button>
  <canvas id="paper" aria-label="Editable paper receipt"></canvas>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Component design notes

#### Real paper and real output
An offscreen Canvas draws the current receipt. The visible Canvas maps that texture onto a 12 × 18 triangle grid with a local sinusoidal fold. This is an artistic mesh approximation, not a cloth simulation. Pointer capture keeps drags continuous; release or cancellation starts a bounded decay back to flat. Keyboard users have equivalent Scrunch and Smooth buttons. Reduced motion suppresses continuous drag warping and instantly smooths on release; the explicit Scrunch action remains a static state change.

#### Editable receipt
Keep 1–5 lines, 32 characters per name and prices from 0 to 9999.99. Parse monetary values as integer cents rather than floating-point addition. Invalid edits disable export and retain the last valid receipt; show the correction message. Native text inputs use decimal input mode without accepting scientific notation or negative prices. Never interpret user-entered names as HTML.

#### Export and accessibility
PNG export uses the offscreen flat paper texture, so the saved receipt remains readable even when the on-screen paper is folded. The download happens only on the explicit button click. A containing sandbox must allow downloads. Canvas has an updated text alternative listing the items and total; the same information is also available in the editable fields. The receipt is a decorative souvenir and does not claim a real financial transaction or official payment status.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`（可加 `--lang zh` 或 `-p params.json`）。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
