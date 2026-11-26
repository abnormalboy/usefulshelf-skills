---
name: word-dissolve-poster
description: 文字粒子海报（Kinetic glyph particle dissolve poster on HTML Canvas）。离屏 Canvas 按真实字形 alpha 采样粒子，支持中英文实时重排、侧风与重力散落/还原动画，适用于创意排版海报、品牌动效实验与交互字体展示。
---

# 文字粒子海报

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

离屏 Canvas 按字形 alpha 采样粒子，不用矩形粒子冒充文字。按实际可用宽高拟合中英混排，编辑海报最多 36 字、独立页面标题最多 300 字。横向风改变 x、二次重力改变 y，散落与聚拢共享粒子和当前进度，使打断后可反转。动画 1.5 秒结束即停；减少动态效果直接到可读/空白端点，Canvas Enter/Space 与按钮都能切换。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`（1–300 字符）、`--accent`（3/6 位 HEX）、`--density`（整数 3–8）、`--phrase`（1–36 字符，海报初始文字）、`--lang`（`en` / `zh`）、`-p` / `--params`（JSON 配置文件）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #e9ff68;
  font-family: Arial, "PingFang SC", sans-serif;
  color: #f3f3e9;
  background: #181a17;
  color-scheme: dark;
}
```

```html
<main class="skill-demo">
  <h1>Words have weight.</h1>
  <p>每个字都有自己的重量。写下几个词，看笔画散成颗粒、沿重力落下，再一点点回到原位。</p>
  <input id="words" aria-label="words" type="text">
  <button id="dissolve" type="button">让文字散落 ↓</button>
  <button id="gather" type="button">重新排版 ↑</button>
  <button id="remix" type="button">换句话 ↻</button>
  <input id="wind" aria-label="wind" type="range">
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Interaction and rendering notes

The editable poster is drawn into an offscreen canvas, then sampled from actual alpha-covered glyph pixels. Never replace this with rectangular bounding-box particles: the original shape must still read while it begins falling. Fit text to the available area and support both CJK and Latin text. Side wind affects horizontal displacement while quadratic gravity drives the vertical path. Scatter and gather share the same particles, so an interrupted transition can reverse from its current amount. The animation lasts 1.5 seconds and stops. Reduced motion commits a fully readable or empty endpoint. Keep editable copy as plain input/textContent, never HTML. Limit editable poster text to 36 characters while allowing the independent page heading to remain fully readable up to 300 characters. Canvas Enter/Space toggles scatter/gather; dedicated buttons make the action discoverable.

#### Visual tokens and layout

Pair a matte charcoal shell (`#181a17`) with a deep obsidian poster frame (`#0c0d0b`) and a single high-contrast accent (`#e9ff68` default) driving the top hairline rule, input ink, active button fill, and canvas glyph pixels. Maintain two balanced columns (`0.9fr / 1.1fr`) above 700px and stack into a single centered column on mobile with zero horizontal overflow. Keep the poster aspect ratio close to `0.85` and sample canvas dimensions at up to `2x` devicePixelRatio via ResizeObserver so strokes stay razor-sharp on retina screens.

#### State machine, accessibility, and lifecycle

Track explicit transition state (`set`, `falling`, `fallen`, `gathering`) with a single `requestAnimationFrame` handle cancelled on resample, language change, `visibilitychange`, or reduced-motion toggle. Announce state transitions in one polite live region (`#status`) and update the particle readout (`#particleCount`) after every build. Default poster text localizes to Chinese (`让声音被看见`) in Chinese mode unless the user supplies a custom `phrase` via CLI/params or edits `#words`.

#### Parameters and boundaries

Accept `--title` (1–300 chars), `--accent` (3- or 6-digit hex), `--density` (integer 3–8 controlling sampling stride), `--phrase` (1–36 chars), `--lang` (`en` or `zh`), and `-p`/`--params` JSON files. Escape all HTML and inline JSON (`<`, `>`, `&`), never pass user text into `innerHTML`, and keep rendering 100% offline with zero network calls.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`（可加 `--lang zh` 或 `-p params.json`）。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
