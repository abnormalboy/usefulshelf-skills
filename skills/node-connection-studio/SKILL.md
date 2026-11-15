---
name: node-connection-studio
description: 节点接线工作台。点阵画布包含暖冷胶囊节点，边保存稳定端点与路径，支持直连、分支合流、回环预设。 Use for this interaction and adapt it to the existing framework.
---

# 节点接线工作台

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

点阵画布包含暖冷胶囊节点，边保存稳定端点与路径，支持直连、分支合流、回环预设。移除连线与撤销更新同一图模型，鼠标与键盘均可拆线。回环仅表达路线，不执行循环计算。独立文字管道将纯文本做大写/反转/分词等确定变换，共用渲染与 SVG 导出状态，不把图连线宣称为实际远程工作流。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`, `--accent`, `--phrase`, `--curvature`。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #e57950;
  font-family: ui-sans-serif, system-ui, sans-serif;
  color: #ece7df;
  background: #111211;
  color-scheme: dark;
}
```

```html
<main class="skill-demo">
  <h1>给灵感接上线。</h1>
  <p>暖色节点发出信号，冷色节点接住想法。试试直连、分支合流与回环。</p>
  <button id="show-routes" type="button">接线画布</button>
  <button id="show-pipeline" type="button">文字管道</button>
  <button id="route-undo" type="button">撤销改动</button>
  <svg id="route-svg" aria-label="可删除连线"></svg>
  <button id="warm-tl" type="button">暖节点左输入</button>
  <button id="warm-tc" type="button">暖节点中输入</button>
  <button id="warm-tr" type="button">暖节点右输入</button>
  <button id="warm-bl" type="button">暖节点左输出</button>
  <button id="warm-bc" type="button">暖节点中输出</button>
  <button id="warm-br" type="button">暖节点右输出</button>
  <button id="cool-tl" type="button">冷节点左输入</button>
  <button id="cool-tc" type="button">冷节点中输入</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Two modes, two explicit state models

The initial route editor restores a dark dotted canvas, a warm upper capsule and a cool lower capsule. Three inputs and three outputs sit on each capsule perimeter. The warm stroke uses the generator accent; the destination remains cool blue. User-space SVG gradients keep even perfectly vertical wires visible. On narrow screens the preset sidebar moves above the canvas while the capsules retain their long, rounded proportions.

Presets produce one direct connection, three warm outputs converging into one cool input, or two forward connections plus a large left-side return loop. These are editable route objects, not hardcoded screenshots. Arbitrary cross-node output-to-input connections can be made by two clicks, keyboard activation or dragging. Multiple incoming wires are valid in this mode; duplicate pairs and same-node connections are rejected. Return loops are visual only and never execute a data cycle.

Each wire has a broad transparent hit area and a native delete button at its geometric midpoint. Hovering the wire or focusing its button reveals the red delete handle; clicking the line itself does not silently delete it. Always-visible route chips provide an equivalent touch/keyboard action. Deletion restores focus to Undo change. A bounded copied-state history supports presets, clear, individual deletion and manual connections. Switching modes preserves route state and the independent text pipeline.

Validate direct=1, merge=3 and loop=3 wires; clear=0 and undo restores the prior set. Verify multiple outputs really share an input, a cool output can return to a warm input, and removal via the line handle updates the route list. Make sure Escape cancels the temporary wire without deleting completed routes.

### A small, real dataflow

The dotted canvas contains three kinds of nodes: a text source, a transform and a poster output. A port click starts a connection; another compatible port click completes it. Pointer dragging provides the same operation. Keyboard users can activate the native port buttons, and Escape cancels a pending wire. Bezier endpoints come from the live port bounds, so a ResizeObserver redraws the routes when the layout changes. On narrow screens ports move to the top and bottom and the curve direction changes to vertical.

The permitted graph is acyclic: source → transform, transform → poster, or source → poster. Every input accepts one upstream connection. Replacing an incoming wire is a single undoable change. The poster displays nothing computed until a complete path reaches it. Direct wiring preserves the original text; transformed wiring applies the selected uppercase, Unicode-code-point reverse, echo or spacing operation. This is a text experiment, not a general workflow execution engine.

Keep edge state separate from painted paths. Removal through the visible route chips gives keyboard users the same action as clicking a wire. A bounded undo stack stores copies of edge lists. Suppress redundant input clicks after pointer-up completion so the successful connection message is not overwritten. With reduced motion, the optional traveling-dash pulse disappears; the actual graph still works.

SVG export uses the actual current output and XML-escapes its text. Long text wraps using the actual 34 px sans-serif text measurement into 490 px lines (with a conservative character-width fallback when canvas measurement is unavailable) and increases the exported page height rather than truncating content. The saved file contains only vector shapes and text. Export requires allow-downloads in a sandboxed iframe. Validate both full and bypass routes, changing operations after connecting, removal, undo, empty input, Unicode and XML special characters.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 templates/index.template.html，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
