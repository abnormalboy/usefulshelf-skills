---
name: dot-spiral-studio
description: 点阵螺旋工坊（Canvas + SVG 生成式点阵艺术工坊）。固定点阵中心，以方螺旋/同心环/波浪/三形态循环驱动单元格大小与色带，支持矢量海报 SVG 导出。Trigger when building generative dot-matrix studios, algorithmic spiral/wave posters, Canvas-to-SVG art generators, or interactive math art widgets.
---

# 点阵螺旋工坊

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

固定网格中心，只更新圆角单元的大小与颜色。方形螺旋用角度加 Chebyshev 半径，圆环用 Chebyshev 等值环，波浪用交叉周期场；三种形态在相位三等分区间以 smoothstep 混合。Canvas 与 SVG 共用 cells() 几何，相位滑杆是唯一数据源。用户启动八秒单轮播放，只有勾选循环才持续；隐藏页、手动改相位、减少动态效果立即停帧。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`-p/--params`（JSON 配置）、`--lang`（`en` 或 `zh`）、`--title`（1–300 字符）、`--accent`（3/6 位 HEX）、`--grid`（整数 `17–41`，默认 `31`）、`--phase`（整数 `0–100`，默认 `44`）、`--mode`（`spiral` | `rings` | `wave` | `cycle`，默认 `spiral`）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #ff7948;
  font-family: Arial, "PingFang SC", sans-serif;
  color: #f5eee7;
  background: #11100f;
  color-scheme: dark;
}
```

```html
<main class="skill-demo">
  <h1>让秩序，转个弯。</h1>
  <p>拖动时间，让冷白点阵卷成螺旋。独立探索三种图案，或让它们连续变换，把红橙黄的轨迹留成矢量海报。</p>
  <input id="phase" aria-label="phase" type="range">
  <input id="grid" aria-label="grid" type="range">
  <input id="repeat" aria-label="repeat" type="checkbox">
  <button id="play" type="button">▷ 播放一圈</button>
  <button id="save" type="button">保存 SVG ↗</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Rendering and interaction

The matrix remains fixed in position: only each rounded cell's scale and color change. Three analytic fields use angle plus Chebyshev radius (square spiral), Chebyshev rings, and crossing waves. A saturated red–accent–yellow band follows the field's shoulder while the larger cells stay cool white. Preserve enough dark negative space to read the topology; adding a uniform glow to every cell obscures it. The same `cells()` geometry drives Canvas and SVG, so an exported frame matches the displayed frame. Export numeric geometry and validated colors only.

The range input is the source of manual phase changes. Canvas clicks advance the phase; left/right arrows provide one-percent steps. Playback is user initiated, runs one eight-second cycle, and stops by default. The explicit Repeat loop checkbox permits continued playback until paused; changing that preference stops existing playback. Three-form cycle blends spiral → rings → waves → spiral with smoothstep interpolation over phase thirds; wave fields are periodic so phase zero and one agree. Independent presets remain available. Changing phase stops playback. Hidden-page and reduced-motion changes cancel animation; reduced-motion playback takes a static step. Matrix count equals grid squared, and every control remains operable when no frames are running.

The canvas pixel density is capped at 2. At the maximum grid size it draws 1,681 simple rounded rectangles without per-cell blur. Narrow screens place the square art before the controls. Never shrink the art into a tiny preview just to keep controls on one row.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
