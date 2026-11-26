---
name: svg-highlight-nav
description: 手绘圈选导航（Hand-drawn SVG highlight ring tab navigation）。暖白编辑式排版中，每个原生导航标签内置三条手绘 SVG 椭圆路径并随悬停/焦点/键盘选择流畅描边，适用于工作室官网导航、编辑式标签页与创意作品集头部。
---

# 手绘圈选导航

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

暖白编辑式排版中，每个导航按钮拥有自己的三条 SVG 手绘路径，不依赖全局坐标。hover 只预览候选圈线，不更换选中的正文；click 更新标签、heading/body、计数、aria-selected 与 panel 关联。左右/Home/End 在 tablist 内移动并选择。新圈线开始先取消旧 Web Animations，减少动态效果直接显示最终圈；文本走安全数据，圈线 fixed vector 加 non-scaling stroke。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`（1–180 字符，左上角字标与网页标题）、`--subtitle`（1–180 字符，底部说明）、`--accent`（3/6 位 HEX，手绘圈选描边色）、`--bg`（3/6 位 HEX）、`--duration`（整数 0–2000 ms，描边动画时长）、`--lang`（`en` / `zh`）、`-p` / `--params`（JSON 配置文件，支持 `duration` 与 `items` 数组：2–8 项 `{label, heading, body}`）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #e4bb32;
  --bg: #f8f8f3;
  --ink: #222423;
}
body {
  font-family: Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
}
```

```html
<main class="page">
  <header class="topline">
    <div class="wordmark">Fieldwork®</div>
    <div class="availability">Available for good ideas</div>
  </header>
  <section class="content">
    <p class="eyebrow">A small gesture. A clear direction.</p>
    <div class="nav" role="tablist" aria-label="Explore sections">
      <button id="section-tab-0" type="button" role="tab" aria-controls="content-panel" aria-selected="true" class="marked" tabindex="0">
        <span>Home</span>
        <svg class="ring" viewBox="0 0 112 50" preserveAspectRatio="none" aria-hidden="true">
          <path d="M8 29C1 14 24 3 57 5S108 16 105 29 77 47 42 44 1 36 7 21C10 15 20 11 31 10" pathLength="1" />
        </svg>
      </button>
      <button id="section-tab-1" type="button" role="tab" aria-controls="content-panel" aria-selected="false" tabindex="-1">
        <span>About</span>
        <svg class="ring" viewBox="0 0 112 50" preserveAspectRatio="none" aria-hidden="true">
          <path d="M8 27C3 12 32 3 62 5S108 15 104 30 68 47 36 44 0 34 10 20C15 13 28 9 37 8" pathLength="1" />
        </svg>
      </button>
      <button id="section-tab-2" type="button" role="tab" aria-controls="content-panel" aria-selected="false" tabindex="-1">
        <span>Work</span>
        <svg class="ring" viewBox="0 0 112 50" preserveAspectRatio="none" aria-hidden="true">
          <path d="M7 30C0 16 26 5 57 4S108 14 105 29 75 47 40 44 3 37 6 24C9 16 24 10 36 9" pathLength="1" />
        </svg>
      </button>
    </div>
    <section class="panel" id="content-panel" role="tabpanel" aria-labelledby="section-tab-0" tabindex="0">
      <h1>Good things start with a little curiosity.</h1>
      <p>Thoughtful digital experiences, built around the small moments that make a lasting impression.</p>
    </section>
  </section>
  <footer class="bottom">
    <span>An expressive navigation study.</span>
    <span id="count">01 / 05</span>
  </footer>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Design and behavior

A warm off-white page, small studio wordmark, wide navigation, and large serif statement give the thin golden ring enough room. Three slightly different inline SVG paths produce a deliberate, drawn gesture. Each ring belongs to its button, so wrapping and resizing do not require global coordinate measurements.

#### Interaction contract

- Hover animates the ring around a candidate without replacing the selected content. Leaving returns to the focused or selected tab.
- Clicking selects a tab and updates its heading, paragraph, counter, `aria-selected`, and tab panel label.
- Arrow keys, Home, and End are handled inside the tab list only, moving focus and selection. One selected tab remains in the normal tab sequence.
- Pending Web Animations are cancelled when another highlight starts. This prevents delayed animation completion from restoring a stale selection.
- The reduced-motion preference is monitored live; the final ring remains visible without drawing motion.

#### Adaptation

`items` supplies content directly; each object must have only `label`, `heading`, and `body`. The generator validates lengths and count, rejects unknown configuration, escapes text, and protects inline JSON against closing-script injection. Colors are hex-only.

Ring paths are locally drawn vectors, rendered with a non-scaling stroke. The animation is a stroke reveal, not physical ink simulation. All assets are inline and the output works offline. The bundled content represents section tabs rather than multiple network-loaded pages.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`（可加 `--lang zh` 或 `-p params.json`）。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
