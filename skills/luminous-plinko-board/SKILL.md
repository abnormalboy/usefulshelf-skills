---
name: luminous-plinko-board
description: 发光弹珠实验台（Canvas 2D 120Hz 物理碰撞与光影落点分布实验）。固定 640×460 模拟空间实现半隐式积分、动态阴影、弹性调节、单秒步进与实时直方图统计。Trigger when building interactive Plinko/Galton board physics demos, luminous Canvas 2D collision experiments, probability histograms, or fixed-timestep simulations.
---

# 发光弹珠实验台

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

模拟坐标固定为 640×460，与显示尺寸解耦；使用 120Hz 有界累加器、先更新速度后更新位置的半隐式积分。圆形重叠沿法向分离并按恢复系数反射速度，侧墙反弹，微小横向扰动防止球停在钉顶。最多 24 球，不模拟球间碰撞；真实撞钉才计碰撞，穿过收集线才给一个落点柱加一。减少动态效果默认暂停，单秒步进计算 120 步后只绘一帧，绝不伪造落点。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`-p/--params`（JSON 配置）、`--lang`（`en` 或 `zh`）、`--title`（1–2000 字符）、`--subtitle`（1–2000 字符）、`--accent`（3/6 位 HEX）、`--gravity`（重力加速度，整数 `200–1400`，默认 `700`）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #b5d8ff;
  --bg: #070b14;
  --chassis: rgba(14, 22, 38, 0.88);
  --ink: #eef4fc;
  --muted: #9eb0cb;
  font-family: "SF Pro Display", Inter, -apple-system, BlinkMacSystemFont, "PingFang SC", system-ui, sans-serif;
  background: var(--bg);
  color: var(--ink);
  color-scheme: dark;
}
```

```html
<main class="skill-demo">
  <h1>Let curiosity fall.</h1>
  <p>One small nudge. A different journey. Explore light, chance and collisions.</p>
  <canvas id="board" aria-label="Pegboard simulation. Click to release a ball at that position, or press Space to release at the launch position."></canvas>
  <input id="launch" aria-label="launch" type="range">
  <input id="bounce" aria-label="bounce" type="range">
  <button id="drop" type="button">Release a ball ↓</button>
  <button id="burst" type="button">Release five</button>
  <button id="pause" type="button">Pause</button>
  <button id="step" type="button">Advance 1s</button>
  <button id="clear" type="button">Start over</button>
  <button id="path" type="button">Last path: on</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.


### Light, collisions and observable outcomes

Use a fixed 640 × 460 simulation space independent of displayed size inside a midnight optical glass chassis (`#070b14` backdrop, `#040812` board bay). A bounded 120 Hz accumulator separates physics from refresh rate. Semi-implicit integration adds gravity before position. Circle overlaps are resolved along the collision normal, then inward velocity is reflected using restitution. A small horizontal nudge prevents exact top-of-peg equilibria. Side walls reflect velocity. Ball-to-ball collisions are deliberately absent; this experiment compares independent journeys through a peg field.

Light is layered as finely etched calibration rails, radial additive color blooms, directional geometric peg shadows, machined optical glass pegs that emit expanding hit ripples on collision, tapered comet-tail ball streaks, and illuminated collector slots. It is a Canvas 2D approximation, not ray tracing. The canvas backing store is twice its coordinate size (`1280 × 920`). Keep active balls capped at 24 and trails bounded. Only real peg impacts increment collision counts, and only crossing the collection line increments one histogram bin. The last completed trajectory remains available as an inspectable dashed path. No score, prize or wagering exists.

Launch position and bounciness controls change actual simulation variables. Release, pause, single-second step, reset and path visibility all work repeatedly. Under reduced-motion preferences, begin paused with a static sample field; advancing one second computes 120 steps and renders once. Resume is an explicit user choice. Keep the DOM histogram and live status so the canvas is not the sole representation of results.

For validation, reset, release one ball and advance several seconds: exactly one bin should gain one landing. Repeat at another launch position; counts must accumulate and reset must zero all state. Check that keyboard activation on the canvas releases a ball and moving shadows remain visible while paused. Extreme restitution or gravity changes may alter time to landing; do not fabricate outcomes to satisfy a presentation timer.

#### Visual tokens

- Deep midnight navy backdrop: `#070b14` (`--bg`)
- Frosted instrument panel: `rgba(14, 22, 38, 0.88)` (`--chassis`)
- Primary text: `#eef4fc` (`--ink`)
- Secondary text: `#9eb0cb` (`--muted`)
- Default accent: `#b5d8ff` (`--accent`)

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
