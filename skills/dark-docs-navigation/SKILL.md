---
name: dark-docs-navigation
description: 深色文档导航（开发文档侧栏树与顶栏标签壳层）。黑底高对比、2px 强调色标签线、可折叠虚线层级树与移动端抽屉开关，支持自定义 sections/pages JSON。Trigger when building developer documentation layouts, dark sidebar tree navigation, collapsible doc drawers, or API/SDK reference shells in HTML/Vue/React.
---

# 深色文档导航

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

深色背景、2px 强调色标签线、280px 侧栏和可收缩虚线树组成布局。一个 aria-current 对应当前文档；父项用 aria-expanded/aria-controls，折叠内容 hidden/inert。顶栏标签使用 roving tabindex 与左右/Home/End。窄屏导航与正文上下排列；真实路由由目标项目注入，不伪称示例查询已连接后台。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`-p/--params`（JSON 配置，额外支持自定义 `sections` 导航树数组、`pages` 页面标题/正文映射、`fontFamily` 及全部颜色 Token）、`--lang`（`en` 或 `zh`）、`--title`、`--brandName`（同时渲染于顶栏品牌位）、`--brandDocs`、`--accent`、`--bg`、`--surface`、`--activeId`、`--intro`、`--sidebarWidth`（`240px`–`400px`，必须包含 `px` 单位，如 `--sidebarWidth 300px`）、`--transitionMs`（整数 `0–2000`）。窄屏（`<= 720px`）自动将侧栏收纳至无障碍菜单按钮（`aria-expanded`/`aria-controls`）背后，保证文档标题与正文直接位于首屏。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --bg: #000000;
  --surface: #1a1a1a;
  --hover: #141414;
  --text: #f2f2f2;
  --muted: #9a9a9a;
  --section: #7a7a7a;
  --accent: #e23b3b;
  --border: #222222;
  --tree: #3a3a3a;
  --sidebar-w: 280px;
  --radius: 6px;
  --row-h: 36px;
  --font: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  --duration: 220ms;
  --trans: var(--duration) cubic-bezier(0.2, 0.8, 0.2, 1);
}
body {
  font-family: var(--font);
}
```

```html
<main class="skill-demo">
  <h1>What is Atlas?</h1>
  <p>Explore the platform, follow the quickstart, and learn the core concepts.</p>
  <button id="tab-documentation" type="button">Documentation</button>
  <button id="tab-query" type="button">Query Reference</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Design and interaction reference

The shell uses a black background, muted section labels, bright text, and a single accent color. Header tabs use a 2px accent underline; selected sidebar rows use a lighter surface. Generic inline line icons require no icon font.

#### Layout

The body is a flex column. The header wraps when needed; below it, a fixed-width sidebar sits beside a flexible article panel. At 720px and below, navigation and content stack vertically; the sidebar has a 58vh maximum height and scrolls independently. Keep `min-width: 0` on the article panel so long text can wrap.

| Token | Default | Purpose |
| --- | --- | --- |
| `--bg` | `#000000` | Page background |
| `--surface` | `#1a1a1a` | Selected row |
| `--hover` | `#141414` | Hover feedback |
| `--text` | `#f2f2f2` | Main text |
| `--muted` | `#9a9a9a` | Secondary text |
| `--section` | `#7a7a7a` | Group labels |
| `--accent` | `#e23b3b` | Tab underline and focus ring |
| `--border` | `#222222` | Dividers |
| `--tree` | `#3a3a3a` | Dashed hierarchy connectors |
| `--sidebar-w` | `280px` | Desktop navigation width |
| `--radius` | `6px` | Row corners |
| `--row-h` | `36px` | Top-level row height |
| `--duration` | `220ms` | Transition duration |

#### State

`aria-current="page"` identifies one selected row. Parent buttons use `aria-expanded` and `aria-controls`; opening removes `inert` from the associated submenu. Closing restores `inert` and hides the children. Dashed borders on pseudo-elements provide the vertical and horizontal tree lines.

Top tabs use roving tabindex, Left/Right and Home/End keys, `aria-selected`, and a labelled panel. Selecting a sidebar row returns the header to Documentation. Both expandable groups open automatically for an initially selected child.

The expand transition uses a fixed 420px maximum height. If adding enough children to exceed that height, update the limit or measure the content height. Reduced-motion preferences disable transitions. Avoid encoding easing functions in variables used by CSS duration arithmetic.

#### Integration boundaries

Replace sample article text before using this in a live product. For real page navigation, use appropriate links and route state. Do not claim that the sample tabs provide search or query execution. Preserve visible keyboard focus, prevent collapsed controls from receiving focus, and verify customized colors for readable contrast.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
