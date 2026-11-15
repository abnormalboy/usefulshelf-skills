---
name: story-portfolio-canvas
description: 故事作品集画布。六幅原创灰阶 SVG 共享筛选后的项目 ID，Scatter/Grid/Orbit 只改变同一组原生按钮布局。 Use for this interaction and adapt it to the existing framework.
---

# 故事作品集画布

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

六幅原创灰阶 SVG 共享筛选后的项目 ID，Scatter/Grid/Orbit 只改变同一组原生按钮布局。Scatter 用显式 X/Y/Z 与轻旋转、Orbit 用 X/Z 投影和角度层级，Grid 提供常规对照；无持续自动播放。筛除项从键盘流和渲染同时隐藏。点击项目打开原生 dialog，聚焦关闭，Escape 还焦；换布局不打开故事。移动端散落有专用坐标，网格两列，SVG 高度自适应。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`, `--subtitle`, `--accent`, `--bg`。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #cf6948;
  --bg: #171917;
  --ink: #eeece6;
}
body {
  font: 13px system-ui;
}
```

```html
<main class="skill-demo">
  <h1>每个作品，都有故事。</h1>
  <p>翻阅一间独立工作室的创作笔记，发现每个小作品背后的想法。</p>
  <button id="previous" type="button">上一个作品</button>
  <button id="next" type="button">下一个作品</button>
  <button id="close" type="button">关闭故事</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Component design notes

#### Image field
A quiet studio header frames six original grayscale SVG studies. The portrait artworks use gradients, cast shadows, room light and vignette to create depth. They are illustrations, not photographs. Keep the heading small relative to the gallery and preserve each project’s visual identity. SVG gradient and filter IDs differ between gallery and detail instances.

#### Three layouts and filters
Filters determine visible project IDs. Scatter, Grid and Orbit position the same native buttons. Scatter uses explicit X/Y/Z positions, overlapping cards and restrained rotations. Pointer movement subtly adjusts the field perspective only in Scatter and when reduced motion is off. Orbit projects positions into X/Z space with stronger side turns and angle-based stacking; Previous/Next changes the centered project, and keyboard focus brings that project forward. There is no continuous autoplay. Grid offers a conventional accessible comparison layout. Keep filtered projects hidden from both rendering and keyboard flow.

#### Dialog and responsive behavior
Clicking a project opens a native modal dialog named by its heading. Focus goes to Close; Escape and explicit Close return focus to the opener. Changing layouts does not open a story. The live status region announces filter, layout and orbit changes.

On mobile, dedicated scatter positions and smaller portrait cards preserve a layered field; Grid uses two minmax(0,1fr) columns. Orbit crops its decorative periphery within its own stage. Every inline SVG uses height:auto. Reduced motion removes transition interpolation and pointer parallax while retaining direct state changes. The generator background controls the page surface, and accent controls small studio details. All assets are inline and operate without network access.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 templates/index.template.html，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
