---
name: scroll-hide-toolbar
description: 滚动隐藏悬浮导航。外层设备画布包含独立可滚动图标网格，短胶囊与右侧加号分开。 Use for this interaction and adapt it to the existing framework.
---

# 滚动隐藏悬浮导航

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

外层设备画布包含独立可滚动图标网格，短胶囊与右侧加号分开。被动 scroll 监听只排一帧 RAF，记录钳制后的滚动位置、方向与累计位移，反向先清累计，超过默认 8px 才隐藏或恢复；距顶部 24px 内总展开。收起宽度由 160px 到 56px，默认 320ms，头像仍可操作，其余 inert；键盘聚焦保持展开，方向键/Home/End 管理单一 tabstop。实际产品中应绑定真实滚动容器，并为底部内容留避让。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`, `--label`, `--accent`, `--theme`, `--hideThreshold`, `--navHideMs`, `--bg`, `--stage`, `--text`, `--muted`, `--iconRadius`。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --bg: #e8eaef;
  --stage: #ffffff;
  --text: #111111;
  --muted: #6b7280;
  --accent: #2f7dff;
  --glass: rgba(255,255,255,0.62);
  --glass-border: rgba(255,255,255,0.55);
  --shadow: 0 24px 60px rgba(15,23,42,0.18);
  --icon-radius: 28%;
  --nav-hide-ms: 320ms;
  --font: ui-sans-serif, system-ui, -apple-system, "Segoe UI", sans-serif;
  --phone-w: 390px;
  --phone-h: 844px;
}
```

```html
<main class="skill-demo">
  <h1>Scroll-aware toolbar</h1>
  <p>A floating glass toolbar that follows your scroll direction.</p>
  <button id="fab" type="button">Add a sample icon</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Scroll-aware toolbar design

#### Structure

A responsive device stage contains a separately scrollable two-column grid. A short pill with three view buttons sits on the left; a separate circular plus button sits on the right with a wide gap between them. On screens up to 480px wide, the stage fills the viewport using dynamic viewport height. Safe-area insets lift the controls above device edges.

The demo uses generic inline SVG icons with CSS backgrounds. It contains no external image or font assets. The view buttons update their own selection only; the plus button adds a sample tile and scrolls it into view.

#### Scroll algorithm

A passive scroll listener schedules one requestAnimationFrame callback. Each callback clamps the current scroll position to the container's valid range, updates the previous position, and accumulates travel in the current direction. A reversal resets that accumulation, preventing a previous direction's partial delta from triggering the opposite action.

After travel reaches `hideThreshold`, downward motion contracts the pill to the profile circle and fades the plus button. Upward motion restores both controls. Within 24px of the top they remain expanded. The listener never changes scroll position, so it does not create a feedback loop.

The profile circle remains interactive while the contracted pill's other buttons and the plus button become inert. Keyboard focus on those controls keeps the pill expanded. Left/Right and Home/End move among view buttons, using a single tab stop. The plus button scrolls to the added sample tile unless reduced motion is requested.

#### Appearance

- Light theme: pale gray page, white stage, translucent white glass.
- Dark theme: near-black page and stage, translucent dark glass.
- Explicit color parameters override the selected theme's defaults.
- The pill uses 22px backdrop blur, a highlighted border, an inset cyan glow, and a soft shadow. Without backdrop-filter support its translucent background remains usable.
- Contracting animates the pill from 160px to 56px over 320ms by default. The plus button fades and shifts down while the profile circle stays fixed.
- Active buttons use the accent color with a lightly tinted surface. Focus gets a visible accent outline.

#### Integration boundaries

For a full page, replace the demo scroller reference with the project's actual scrolling element and position the toolbar within the appropriate fixed or sticky context. Ensure there is sufficient bottom padding for content behind the toolbar. Keep navigation accessible even when scrolling is unavailable or reduced motion is enabled. Replace the example view selection with links or application state when actual navigation is needed.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 templates/index.template.html，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
