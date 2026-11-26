---
name: node-connection-studio
description: 节点接线工作台（Node connection studio & interactive SVG bezier wire editor）。点阵画布包含暖冷胶囊节点与实时数据流管道，支持直连、分支合流、回环预设及 SVG 海报导出，适用于可视化工作流、节点编排与连线交互演示。
---

# 节点接线工作台

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

点阵画布包含暖冷胶囊节点，边保存稳定端点与路径，支持直连、分支合流、回环预设。移除连线与撤销更新同一图模型，鼠标与键盘均可拆线。回环仅表达路线，不执行循环计算。独立文字管道将纯文本做大写/反转/分词等确定变换，共用渲染与 SVG 导出状态，不把图连线宣称为实际远程工作流。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`、`--accent`（3/6 位 HEX）、`--phrase`（1–160 字符）、`--curvature`（整数 30–200）、`--lang`（`en` / `zh`）、`-p` / `--params`（JSON 配置文件）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --cyan: #4ab6ff;
  --obsidian: #090b10;
  --surface: #0f1219;
  --ink: #f1f4f9;
  --muted: #9aa6b8;
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", ui-sans-serif, system-ui, sans-serif;
  color: var(--ink);
  background: var(--obsidian);
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


### Two modes, one obsidian dark-glass node engine

Core tokens: `--accent: #e57950`, `--cyan: #4ab6ff`, `--obsidian: #090b10`, `--surface: #0f1219`, `--ink: #f1f4f9`, `--muted: #9aa6b8`. The initial routing editor presents a dual-scale architectural micro-grid canvas with sculpted smoked-glass warm source and cool destination capsules, top specular rim highlights, glowing emblem medallions, and machined hardware port sockets.

#### Glowing bezier energy cables & presets
Each route draws layered SVG paths inside `.route-group`: a soft wide luminous bloom (`.route-glow`), a crisp amber-copper to electric-cyan gradient core (`.route-line`), an animated traveling energy pulse (`.route-pulse`, disabled under `prefers-reduced-motion`), a wide transparent hit path, and a midpoint delete button (`.route-delete`). Presets configure `direct` (1), `merge` (3 converging to `cool-tc`), `loop` (2 forward + 1 left return loop `cool-bl` → `warm-tl`), and `empty` (0), all backed by bounded copy-on-write undo history and touch-friendly route chips. Return loops are visual only and never execute a cycle.

#### Unified text pipeline mode
Switching to `#pipeline` keeps route state intact and renders three matching dark-glass cards (`Text seed`, `Word workshop`, `Pocket poster`) with acyclic routing (`source → transform → poster` or direct `source → poster`), deterministic uppercase/reverse/echo/spaced transformations, live resize-observed SVG bezier wires, and self-contained dark-card SVG poster export with canvas-measured text wrapping and XML escaping.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`（可加 `--lang zh` 或 `-p params.json`）。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
