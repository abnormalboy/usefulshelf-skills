---
name: bite-size-button
description: 咬一口按钮（SVG mask 咬痕微交互）。用原生按钮承载六次点击，每次在 SVG mask 切掉一角并散落碎屑，支持口味切换与重新烘焙。Trigger when building playful CTA buttons, SVG mask bite animations, crumb micro-interactions, or gamified multi-click buttons in HTML/Vue/React.
---

# 咬一口按钮

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

用原生按钮承载六次点击；每次将一组交叠咬痕圆加入 SVG `#cookie-mask`，真实切掉饼干边缘并由外层 `feDropShadow` 沿缺口实时投射立体阴影。第六次完成一轮并切换为重新烘焙；换口味与重置清空咬痕、碎屑和进度，但保留完成轮数。碎屑与回弹动效在 820ms 后清理，减少动态效果时不生成碎屑。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`-p/--params`（JSON 配置）、`--lang`（`en` 或 `zh`，省略时默认英文并随 `?lang=zh` 切换）、`--title`、`--subtitle`、`--accent`（3/6 位 HEX）、`--bg`（3/6 位 HEX）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #ac492e;
  --bg: #f4f1e9;
  --icing: #f3ddb0;
  --icing-hi: #fff6df;
  --icing-edge: #d8b574;
  --speck: #6e4522;
  --crust: #c8894b;
  --crust-deep: #8c5323;
  --ink: #2c2118;
  --muted: #5a4b3d;
}
body {
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Segoe UI", sans-serif;
}
```

```html
<main class="skill-demo">
  <h1>A little bite of joy.</h1>
  <p>Choose a flavor. Take six little bites. Leave no crumbs behind.</p>
  <button id="bite" type="button">Take a bite</button>
  <button id="reset" type="button">Bake a fresh one ↻</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Component design notes

#### Visual direction and material craft
Designed as an artisanal Tokyo patisserie showcase on warm bakery linen (`--bg: #f4f1e9`, fine crosshatch weave, sunlit counter radial falloff). The cookie rests on a glazed stoneware porcelain plate with a double gilt-hairline rim and soft ambient foot shadow. The cookie itself combines a multi-stop toasted golden-brown sablé crust (`#dea15d` → `#9e5e29`), a glossy flavor ganache glaze (`--icing`, `--icing-hi`, `--icing-edge`), specular top icing reflection, piped dotted `--accent` frame, and flavor-matched vanilla bean / berry / matcha micro-speckles. An SVG `feDropShadow` wraps the masked cookie group so the real-time drop shadow hugs every freshly bitten scallop edge.

#### State and feedback
A round contains six deliberate clicks. Overlapping scalloped circles added to `#holes` inside `#cookie-mask` remove real areas from the cookie; the final click removes the remaining cookie, increments the round counter, reveals `One more?`, and changes `#bite` to a bake-again action. Explicit reset or flavor change clears bites, resets the mask and crumbs, and keeps the count of completed cookies.

#### Adapting the interaction
Change `positions` to alter bite locations; keep the six progress pips and six-bite logic aligned if changing the count. The SVG is decorative (`aria-hidden="true"`) while the outer native `#bite` button owns keyboard focus and dynamic accessible name (`Take a bite` → `Bake another cookie`). Flavor pills update `--icing`, `--icing-hi`, `--icing-edge`, and `--speck` together with `aria-pressed`.

#### Motion and offline behavior
Each bite triggers a brief spring nibble recoil and scatters irregular rotated crust + glaze crumbs that self-remove after 820 ms. Under `prefers-reduced-motion: reduce`, recoil and crumbs are suppressed while six-bite masking and state updates remain fully functional. Zero external fonts, images, or network requests are used.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
