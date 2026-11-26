---
name: perspective-card-carousel
description: 透视弧形卡片轮播（3D perspective curved card carousel & study modal）。七张卡片处于隔离 perspective 场景，支持拖拽、键盘左右切换、圆点指示器与详情弹窗，适用于画廊、精选案例展示与深色编辑式作品集。
---

# 透视弧形卡片轮播

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

七张卡片处于隔离 perspective 场景，原生按钮包住 SVG 海报和不透明深色标题条。相对偏移 d 乘响应式角间距得到 a，x=sin(a)R，z=(cos(a)-1)R，rotateY 朝向弧心；远处降低亮度并隐藏 inert。选择以 ease-out 走最短环绕距离，打断旧 RAF。水平拖动保留捕获，取消回到稳定选择；自动播放用单个可重启 timeout，焦点/dialog/隐藏/减少动态效果时停播。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--brand`（1–180 字符）、`--title`（1–180 字符）、`--subtitle`（1–180 字符）、`--accent`（3/6 位 HEX）、`--speed`（0–2 浮点数）、`--lang`（`en` / `zh`）、`-p` / `--params`（JSON 配置文件）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #d6ef82;
  --ink: #f5f2ea;
  --muted: #a1a89d;
  --bg: #101311;
  --surface: #181c19;
  --line: rgba(255, 255, 255, 0.1);
  --cw: 256px;
  --ch: 350px;
}
body {
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Segoe UI", "Helvetica Neue", Arial, sans-serif;
}
```

```html
<main class="skill-demo">
  <h1>A collection of perspectives.</h1>
  <p>Independent studies in form, color and movement.</p>
  <button id="prev" type="button">Previous study</button>
  <button id="motion" type="button">Pause</button>
  <button id="next" type="button">Next study</button>
  <button id="close" type="button">Close study</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.


### Perspective composition

Seven collector-edition cards are drawn in an isolated 3D perspective gallery stage (`#101311`). Each card has a native button wrapper, a multi-layered art-directed inline SVG poster with diagonal glass sheen and specular edge rim lighting, and a frosted dark glass caption strip for guaranteed legibility. The center card is largest and illuminated by an overhead gallery spotlight and a dynamic color-matched ambient floor glow (`--stage-glow`); adjacent cards turn smoothly into the arc and recede. Tactile hardware pill controls and an expanding pill-dot indicator sit cleanly below the stage.

#### Geometry

For relative offset d, the card angle is d multiplied by a responsive gap. Its x position is sin(angle) times the radius; z is (cos(angle) - 1) times the radius. Rotation around Y faces the card into the arc. Visibility and brightness decrease away from the center. Hidden cards are inert and removed from the accessibility tree; the active card is keyboard focusable. Slide selectors remain available for every item.

#### Lifecycle

Selection interrupts the previous requestAnimationFrame transition. Position interpolates to the shortest wrapped destination with an ease-out curve, then normalizes to an integer. Pointer capture keeps horizontal dragging consistent outside the stage, and cancellation restores a stable selected state. Autoplay uses a single timeout restarted after each transition, rather than a competing interval.

Reduced motion changes are observed dynamically. Hidden tabs stop autoplay and finish the current selection. Keyboard focus and open dialogs prevent unattended movement. Keep this state centralized when adding controls; multiple independent timers can produce duplicate steps.

#### Adapting content

Project names, categories, descriptions, and original SVG art live in the same script. Replace those together. Captions are plain text constants controlled by the template author; do not interpolate untrusted network content as HTML. The Python generator only accepts escaped presentation text, validated color, and bounded speed. No network services or external media are part of the package.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`（可加 `--lang zh` 或 `-p params.json`）。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
