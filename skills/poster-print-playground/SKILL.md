---
name: poster-print-playground
description: 四色海报印刷间。四个 400×520 原创 SVG 场景分别表现透视卡、数据节点、人像对话和产品爆炸层，不能只换颜色冒充不同构图。 Use for this interaction and adapt it to the existing framework.
---

# 四色海报印刷间

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

四个 400×520 原创 SVG 场景分别表现透视卡、数据节点、人像对话和产品爆炸层，不能只换颜色冒充不同构图。共用一份标题、三套油墨与两档字级；切换场景保持编辑值。用字素分割与 Canvas measureText 按实际 SVG 字体断行，优先词边界，再缩字适配文本框，完整保留中英混排与 emoji。主图、缩略图、导出共用构建器，渐变和滤镜 ID 按实例唯一。

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
  --accent: #d95133;
  --bg: #f3f0e9;
  --ink: #282d27;
}
body {
  font:
}
```

```html
<main class="skill-demo">
  <h1>把灵感，印成一张。</h1>
  <p>四种画面，三套油墨。改一行文字，印一份属于你的灵感。</p>
  <input id="headline" aria-label="海报文字" type="text">
  <button id="download" type="button">下载这张 SVG 海报 ↓</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Component design notes

#### Four visual systems
The workshop contains four original 400 × 520 SVG compositions: a perspective physical card with large typography, a dark network of data nodes, an illustrated person with a conversation prompt, and an exploded product package with a glass bottle. Keep these distinct materials, depth cues and text placements when adapting the themes. The person is a grayscale vector illustration, not a photograph.

The editor shares one headline, three ink palettes and two type scales across all four compositions. Selecting a composition preserves those choices. Each inline SVG uses height:auto; thumbnail dimensions must not inherit the main artwork height. Gradient and filter IDs are unique per composition and main/thumbnail instance.

#### Multilingual typography
The headline input has a maximum length of 48. Segment by Unicode grapheme where supported, measure candidate lines with Canvas measureText using the exact SVG font, prefer word boundaries where available, then reduce the font size until both measured width and line height fit the composition’s headline box. Never use the UI language to guess the text language or slice a fixed count per line. XML-escape every line. Keep every character, including mixed Chinese, Latin, punctuation and emoji; line wrapping can replace whitespace between words.

#### Download and accessibility
One SVG builder supplies the main artwork, thumbnails and exported file, so measured typography and artwork remain consistent. Build a Blob from the selected composition and trigger a download only on an explicit button click, then revoke its object URL. A sandboxed host must allow downloads. Palette one derives its primary ink from the generator accent; other inks are intentional alternatives. Native buttons retain keyboard focus and announce selected state; changes are confirmed in the live status region. The page is offline and respects reduced motion.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 templates/index.template.html，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
