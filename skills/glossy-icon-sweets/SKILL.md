---
name: glossy-icon-sweets
description: 糖果光泽图标集（3D 质感矢量图标调色与 SVG 导出工作台）。六枚原创球形糖果 SVG 图标支持逐项调节色值、高光亮度与倾斜角度并导出独立 SVG。Trigger when building glossy 3D-style SVG icon customizers, candy badge generators, tactile icon editors, or vector asset downloaders.
---

# 糖果光泽图标集

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

六枚原创 SVG 图标由球体、符号、镜面椭圆和地面阴影组成。主图和缩略图为渐变/滤镜分配唯一 ID。每枚图标分别存储色值、0–100 光泽和 −20–20° 倾斜，切换恢复各自参数，重置只影响当前项。导出选中 SVG，保留显式 viewBox、尺寸与全部内联定义，无外部资产。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`-p/--params`（JSON 配置）、`--lang`（`en` 或 `zh`）、`--title`、`--subtitle`、`--accent`（3/6 位 HEX，默认 `#6f4b99`）、`--bg`（3/6 位 HEX）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #6f4b99;
  --bg: #f3f0e9;
  --ink: #261e33;
  --muted: #504561;
  --rule: rgba(54, 40, 74, 0.14);
}
body {
  font: 14px/1.55 -apple-system, BlinkMacSystemFont, "PingFang SC", system-ui, sans-serif;
}
```

```html
<main class="skill-demo">
  <h1>给界面，一点甜。</h1>
  <p>挑一个形状，调一束光，把喜欢的小图标带走。</p>
  <input id="color" aria-label="color" type="color">
  <input id="light" aria-label="light" type="range">
  <input id="tilt" aria-label="tilt" type="range">
  <button id="download" type="button">带走这枚 SVG ↓</button>
  <button id="reset" type="button">恢复这枚的原色</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.


### Component design notes

#### Jelly-glass optical rendering
The six original `200 × 216` SVG candy-glass icons sit on a pearl-silk studio canvas (`--bg: #f3f0e9`, `--ink: #261e33`, `--muted: #504561`, `--accent: #6f4b99`). Each icon layers a tinted caustic ground bounce shadow, soft contact shadow, colored translucent spherical body, upper-left volume highlight, bottom subsurface refraction glow, crisp gradient specular rim, dimensional multi-detail enamel/glass symbol, and curved Apple Aqua dome sheen controlled live by the gloss slider. Every thumbnail and the inspector preview use a unique gradient/filter prefix to prevent SVG ID collisions.

#### Frosted inspector & per-icon edits
The right-hand frosted glass studio inspector displays a spotlight pedestal preview alongside native color swatch, gloss (`0–100%`) and tilt (`-20°` to `20°`) controls with `tabular-nums` badges. Each icon stores its own color, gloss and tilt independently; selecting an icon restores its own values, and Reset affects only that icon.

#### Export and accessibility
Download generates a self-contained offline SVG of the selected icon preserving its edited color, gloss optics and tilt transform, then revokes the Blob URL. All buttons, labels and status messages satisfy WCAG AA contrast (`>= 4.5:1`), hover elevation is gated behind `@media (hover: hover)`, and reduced motion disables transitions.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
