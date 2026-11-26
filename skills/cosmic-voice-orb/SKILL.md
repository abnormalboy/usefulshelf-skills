---
name: cosmic-voice-orb
description: 宇宙声纹球（Canvas 2D 语音助手波形球）。以圆形裁切、径向底色、65 条程序丝带与星点呈现静候/聆听/表达三种 AI 语音状态。Trigger when building AI voice assistant orbs, synthetic audio visualizers, listening/speaking state spheres, or glowing dark-mode Canvas 2D hero elements.
---

# 宇宙声纹球

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

Canvas2D 用圆形裁切、径向底色、65 条程序丝带、星点和边缘高光构成玻璃球。idle/listen/speak 改变频率和振幅，能量滑杆改变包络。演示只使用合成波形，明确不采集麦克风；演示最多 12 秒、单次脉冲 1.8 秒，DPR 上限 2。隐藏页面或减少动态效果时停止循环并绘制静态状态。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`-p/--params`（JSON 配置）、`--lang`（`en` 或 `zh`）、`--title`（1–300 字符）、`--accent`（3/6 位 HEX）、`--intensity`（整数 `0–100`，默认 `55`）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #bda8ff;
  --bg: #07060d;
  --surface: rgba(18, 16, 32, 0.78);
  --ink: #f5f2ff;
  --muted: #b6aed0;
  font-family: "SF Pro Display", -apple-system, BlinkMacSystemFont, "PingFang SC", sans-serif;
  background: var(--bg);
  color: var(--ink);
  color-scheme: dark;
}
```

```html
<main class="skill-demo">
  <h1>A little universe.</h1>
  <p>一颗安静的小宇宙。换一种状态、挑一种颜色，再给它一点能量，看看它如何回应你。</p>
  <input id="energy" aria-label="energy" type="range">
  <button id="play" type="button">▷ 播放 12 秒演示</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.


### Interaction and rendering notes

The glass orb is a Canvas2D approximation built from volumetric ambient nebula bloom, clipped additive (`lighter`) procedural ribbons, concentric acoustic calibration rings, Fresnel chromatic rim light, glass specular highlights, and deterministic starlight motes. It is not a physical shader. Idle, listening, and speaking each change the ribbon frequency, amplitude, and reactive equalizer spectrum strip. Manual energy and the explicitly labeled synthetic waveform control the envelope; never imply live microphone activity. The demo runs for at most 12 seconds. A pointer or keyboard pulse runs for 1.8 seconds, then becomes static. Respect reduced motion by rendering only a static state and explaining why the demo does not move. Pause on hidden tabs. Preserve semantic mode buttons, luminous palette orb controls, slider labels, status feedback, and visible focus. The rendering resolution is capped at devicePixelRatio 2.

#### Visual tokens

- Cosmic backdrop: `#07060d` (`--bg`)
- Frosted deck surface: `rgba(18, 16, 32, 0.78)` (`--surface`)
- Primary text: `#f5f2ff` (`--ink`)
- Secondary text: `#b6aed0` (`--muted`)
- Default accent: `#bda8ff` (`--accent`)

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
