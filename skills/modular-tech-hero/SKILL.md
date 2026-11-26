---
name: modular-tech-hero
description: 模块化科技首屏（Modular tech hero & interactive invoice parser terminal）。近黑底、绿色强调色和细分割线组合两列首屏与可执行代码终端、连续三格指标和折叠文档检查器，适用于 SaaS / 开发者工具 / API 基础设施落地页与本地解析演示。
---

# 模块化科技首屏

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

近黑底、绿色强调色和细分割线组合两列首屏与代码终端，下方连续三格指标和折叠文档检查器。读取最多 4000 字发票文本，顺序执行分类、提取 vendor/currency/amount、验证；要求 INVOICE、非空 vendor、USD，以及不超过 999999999.99 的非负两位小数。每阶段 250ms 仅改善可读性，busy 防重复；指标取真实本地运行次数，Python 页签明确是未执行伪代码。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--brand`（1–24 字符）、`--title`（1–120 字符）、`--description`（1–300 字符）、`--accent`（3/6 位 HEX）、`--lang`（`en` / `zh`）、`-p` / `--params`（JSON 配置文件）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --bg: #0a0d0d;
  --surface: #111616;
  --surface-elevated: #151c1b;
  --line: #232c28;
  --line-strong: #313d37;
  --ink: #f2f5f0;
  --muted: #98a69a;
  --accent: #b6ec85;
}
body {
  font: 15px/1.58 -apple-system, BlinkMacSystemFont, "PingFang SC", "Segoe UI", "Helvetica Neue", Arial, sans-serif;
}
```

```html
<main class="skill-demo">
  <h1>Less plumbing. More possibility.</h1>
  <p>Turn complex documents into dependable workflows. One modular platform for everything you build next.</p>
  <button id="run" type="button">Run sample pipeline ↗</button>
  <button id="copy" type="button">Copy ↗</button>
  <textarea id="invoice-text" aria-label="Sample invoice text"></textarea>
  <button id="valid-sample" type="button">Valid sample</button>
  <button id="invalid-sample" type="button">Invalid sample</button>
  <button id="close-dialog" type="button">Done</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.


### System notes

Deep architectural carbon (`#0a0d0d`), layered graphite surfaces (`#111616` / `#151c1b`), subtle dot-grid backdrop, emerald ambient bloom behind the workbench, and crisp hairline borders (`#232c28`) produce a precision developer flagship interface. Use the phosphor-emerald accent (`#b6ec85`) for active execution pills, focus rings, and primary actions (paired with `#081009` ink for high contrast). Syntax highlighting renders through safe DOM text spans so `textContent` remains exact raw code. Keep body copy legible cool-sage (`#98a69a`) and headings crisp white; avoid using the adjustable accent as the only cue for state.

A two-column hero pairs a short headline with a functional code terminal. A continuous three-cell metrics strip creates structure before the capability section. The lower area pairs a semantic accordion with a ruled document inspector. At 800 px these become one column and the navigation becomes a disclosure menu.

The local pipeline reads editable invoice text, extracts labeled vendor/currency/amount fields, and validates those values. It accepts a required vendor, INVOICE heading, USD currency, and a correctly formatted non-negative amount with up to two decimal places, at most 999,999,999.99. Valid and invalid presets expose both result paths. The 250 ms stage delays only make progress legible; each stage executes its named operation. A busy flag prevents duplicate timers, stage rows communicate progress textually, a JSON result is displayed on completion, and run count increases. Samples never leave the page. The JavaScript tab displays the actual functions used by the page. The Python tab is explicitly labeled non-executed pseudocode and references no SDK. Language controls preserve run state. Copy uses the Clipboard API when available and offers a selectable text dialog when unavailable.

The document inspector reflects parsed input and validation errors after each run. It has classification, fields, and validation modes synchronized with disclosure buttons. A single open mode keeps the visual explanation tied to its text. Use `aria-expanded`, `aria-controls`, and hidden content rather than only opacity. Keep anchor targets below the sticky navigation. Reduced-motion removes pulsing and smooth scrolling; the functional sample sequence can still complete without motion.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`（可加 `--lang zh` 或 `-p params.json`）。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
