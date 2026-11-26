---
name: invite-code-flow
description: 邀请码揭纸动效（可拖拽卷纸揭开与一键复制流程）。覆盖纸从右向左卷曲揭开固定底层的邀请码，内置无障碍 inert 控制、键盘还原与剪贴板回退。Trigger when building referral/invite code peel-to-reveal cards, tactile paper-fold reveal widgets, coupon scratch/peel flows, or clipboard copy components.
---

# 邀请码揭纸动效

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

邀请码固定在底层压凹信笺卡托中，带烫金纹章与香槟金属箔背面的覆盖纸从右边缘向左揭开，拖动与点击共享 revealed 状态。折痕、金属箔纸背和移动阴影传达翻纸质感，不能把整个邀请码随纸拖走。未揭开时复制按钮 hidden/inert；Escape 重新盖上并还焦。每次复制使用 requestId，重新覆盖使旧回调失效，超时则选中可见代码供手动复制；不模拟注册或验证。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`-p/--params`（JSON 配置）、`--lang`（`en` 或 `zh`）、`--title`（1–180 字符）、`--subtitle`（1–180 字符）、`--accent`（3/6 位 HEX，默认 `#2f7d4a`）、`--bg`（3/6 位 HEX）、`--code`（邀请码文本，3–32 位字母/数字/`_`/`-`，默认 `HELLO-GOOD`）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --bg: #f1f3ee;
  --accent: #2f7d4a;
  --ink: #1e2721;
  --muted: #4b584f;
  --foil-gold: #c7a96c;
  --ease: cubic-bezier(0.22, 0.9, 0.25, 1);
}
body {
  font-family: -apple-system, BlinkMacSystemFont, "SF Pro Text", "PingFang SC", "Segoe UI", sans-serif;
}
```

```html
<main class="skill-demo">
  <h1>Peel to Reveal an Invite Code</h1>
  <p>Click or drag the covering paper to the left to reveal the invitation code. Copy it, then cover it again.</p>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.


### Design and behavior

A luxury embossed stationery experience on warm archival cotton paper (`--bg: #f1f3ee` with fine fiber texture and sunlit gallery falloff). A sculpted stationery tray frames an engraved letterpress invitation ticket whose code stays stationary underneath. Clicking the embossed cover sheet or dragging it to the left peels it away from its right edge: a metallic champagne-foil curled underside (`#paperBack`), satin sheen gradient, blind-debossed double frame, foil monogram seal, crease highlight, and moving drop shadow convey real lifted paper. The underlying `.code-text` never shifts or translates.

Once revealed, the magnetic high-contrast `--accent` copy control reports success or selects the code with a manual-copy instruction. Covering the code again restores the embossed sheet and clears feedback. No verification dialog, simulated registration, or automatic invitation exists.

#### State and accessibility

The paper control exposes the reveal state through `aria-expanded`. Keep covered actions unavailable to keyboard and pointer input via `inert` and `aria-hidden`. Provide a button-based reveal and re-cover path so dragging is optional. Closing with Escape restores the cover and returns focus. Feedback is announced through a polite live region. Reduced motion removes animated transitions while preserving reveal, copy, and re-cover states; there is no automatic demo timer.

Dragging left advances the same reveal state as clicking. Keep the underlying code geometry stable throughout the gesture, and settle interrupted or completed gestures into a usable covered or revealed state. Re-covering restores both paper position and interaction state so the effect can be repeated.

Each copy attempt receives a sequence number. Re-covering or starting another attempt invalidates older completions. Successful copying leaves feedback visible until another attempt or reset, rather than clearing it before the user can read it.

#### Clipboard behavior

In a standalone window, the click handler first starts `navigator.clipboard.writeText(code)`. It also tries `document.execCommand('copy')` synchronously within that same click, before transient user activation is lost. Either a resolved Clipboard API operation or a true legacy return value confirms success. Failure or a two-second timeout selects the visible code and requests manual copying. If selection itself fails, the message tells the user to select or long-press the code.

An iframe prefers an optional parent bridge, because sandbox clipboard permission can differ from the host page. The component remains usable without a bridge: it selects the code for manual copying after a short timeout. It sends:

```js
{ type: 'usefulshelf:copy', requestId: 'unique-per-attempt', text: 'HELLO-GOOD' }
```

The host performs the copy and returns to the requesting iframe:

```js
{ type: 'usefulshelf:copy-result', requestId: 'same-request-id', ok: true }
```

The child accepts only messages from `window.parent` with the pending request ID and a boolean `ok`. A negative result or no response within two seconds selects the code and explains manual copying. It never reports success merely because a message was sent. Late or cancelled results are ignored.

#### Integration boundaries

Provide the user's actual authorized invitation code through the host application's own data flow. This generated demo embeds a public sample code and only displays or copies it. Validity, expiration, membership permissions, registration, and message delivery belong to the product's backend and surrounding application.

All assets are inline. Text is escaped, JSON protects script boundaries, CSS colors are hex-only, and unknown configuration keys are rejected. `?lang=zh` and `?lang=en` switch all built-in visible text and accessible labels. Explicitly provided title and subtitle text are preserved.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
