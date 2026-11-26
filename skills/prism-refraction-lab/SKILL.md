---
name: prism-refraction-lab
description: 棱镜折光实验室（Interactive SVG prism light refraction & spectrum export lab）。整幅场景以 SVG 呈现白光穿过三角棱镜折射出七色光谱，支持入射角/色散滑块、单色切换、光脉冲与 SVG 导出，适用于光学交互演示、创意科技着陆页与矢量视觉实验。
---

# 棱镜折光实验室

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

整幅场景用 SVG 表现：白光进入三角左面、穿过内部、从右面分成七条连续柔混的光楔；单色选择使用同一几何的一束，不另造方向。入射角和色散幅度真实改变端点，按边界交点截光而不改方向。1.5 秒脉冲沿真实折线路径匀速运行，重复/改参/隐藏/减少动态效果取消旧脉冲。导出显式尺寸、深色背景、混色滤镜并移除瞬态脉冲；不是精密光学求解器。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`（1–300 字符）、`--accent`（3/6 位 HEX）、`--spread`（整数 12–78，色散角度）、`--angle`（整数 -25–25，入射角）、`--lang`（`en` / `zh`）、`-p` / `--params`（JSON 配置文件）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #b9a5ff;
  --bg: #07070c;
  --deck: rgba(16, 15, 26, 0.86);
  --ink: #f3f1fa;
  --muted: #aca4c2;
  font-family: "SF Pro Display", -apple-system, BlinkMacSystemFont, "PingFang SC", sans-serif;
  color: var(--ink);
  background: var(--bg);
  color-scheme: dark;
}
```

```html
<main class="skill-demo">
  <h1>把一束光，拆成彩虹。</h1>
  <p>转动入射光，展开一片颜色。调节色散，切换单色，再将你的光谱带走。</p>
  <svg id="optics" aria-label="白光经过三角棱镜，分成七条彩色光束"></svg>
  <input id="angle" aria-label="angle" type="range">
  <input id="spread" aria-label="spread" type="range">
  <button id="pulse" type="button">释放一束光 ↗</button>
  <button id="reset" type="button">回到起点 ↺</button>
  <button id="save" type="button">保存 SVG ↓</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.


### Rendering and interaction

This is a geometric visual study rather than a precision optics solver. Inside a dark optical-room bench (`#07070c`) with etched grid lines and goniometer arcs, a multi-pass incident laser beam enters the jewel-beveled triangular crystal prism on its left face, forms a luminous internal caustic path, and splits on the right face into seven contiguous warm-to-cool spectral wedges (`#ff4d5a` through `#c76bff`). Monochrome modes select one ray from that same geometry. Changing incoming direction and spectral spread changes ray endpoints; each endpoint intersects the visible drawing boundary without changing direction.

SVG contains the entire scene: emitter housing, paths, crystal bevel gradients, vertex glints, and softly blurred spectral dispersion. The exported clone receives explicit dimensions (`1800 × 920`) and a dark `#07070c` background and excludes the transient pulse. All interpolated path values are numeric, and palette values are fixed or validated; user copy never enters path markup.

A 1.5-second pulse traverses the actual piecewise center path at constant path speed. Starting another pulse cancels the previous one. Parameter changes, hidden pages, and reduced motion cancel the pulse. The static scene stays fully visible. Keep numeric readouts, source pressed states, and the SVG's accessible description synchronized. The controls use ordinary labeled range inputs and buttons, so the entire experience works without dragging.

Use contiguous, softly blended spectral wedges for the main rainbow, with crisp center filament rays for legibility. Keep adjacent color boundaries touching as dispersion changes. Include the blend filter in exported SVGs so their material matches the preview.

#### Visual tokens

- Dark optical bench backdrop: `#07070c` (`--bg`)
- Optical lab deck: `rgba(16, 15, 26, 0.86)` (`--deck`)
- Primary text: `#f3f1fa` (`--ink`)
- Secondary text: `#aca4c2` (`--muted`)
- Default accent: `#b9a5ff` (`--accent`)

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`（可加 `--lang zh` 或 `-p params.json`）。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
