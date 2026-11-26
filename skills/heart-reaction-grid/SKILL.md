---
name: heart-reaction-grid
description: 点赞反馈动效（深色卡片点赞网格与独立粒子试播台）。四张场景卡片维护独立布尔状态与筛选视图，心形按压/填色/粒子动效严格隔离于真实赞数。Trigger when building interactive like/heart reaction buttons, burst particle feedback, filterable reaction cards, or accessible aria-pressed toggle grids.
---

# 点赞反馈动效

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

四张曜石黑玻璃场景卡片各保存一个布尔点赞值，单项计数只能是 0 或 1，总数取和。筛选只改变可见性，不修改点赞。按钮文案、aria-pressed、发光心形、径向扩散环与 10 枚多色火花粒子同时更新。独立演示心形控制台与真实四个布尔值隔离，一个可取消定时器和 runId 管理按压/填色粒子/完成三个阶段；重置、隐藏页和减少动态效果都终止旧回调。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`-p/--params`（JSON 配置）、`--lang`（`en` 或 `zh`）、`--title`（1–100 字符）、`--subtitle`（1–300 字符）、`--label`（顶栏品牌标签，1–24 字符）、`--accent`（3/6 位 HEX）、`--stagger`（试播阶段延迟毫秒，整数 `150–2000`，默认 `420`）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #ff527b;
  --bg: #0d0f14;
  --surface: #141821;
  --surface-elev: #1a1f2b;
  --ink: #f4f6fa;
  --muted: #9fa9ba;
  --line: rgba(255, 255, 255, 0.09);
}
body {
  font-family: -apple-system, BlinkMacSystemFont, "SF Pro Text", "PingFang SC", "Segoe UI", sans-serif;
}
```

```html
<main class="skill-demo">
  <h1>点一下，让反馈更明确。</h1>
  <p>点击卡片下方的按钮点赞，再点一次取消。每次操作都会更新当前卡片和总计；独立试播只展示动效。</p>
  <button id="preview" type="button">试播一次</button>
  <button id="stop" type="button">停止</button>
  <button id="reset" type="button">重置点赞</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Like-feedback component model

Four explicit contexts illustrate tutorial content, portfolio work, product updates, and community discussion. The UI cards have no fabricated histories, timestamps, or user activity. Each includes a bespoke SVG scene glyph, an identifiable like button, and its local count.

#### Visual craft and surfaces

Built as a high-craft obsidian creator studio (`--bg: #0d0f14`, elevated dark glass cards `#141821` → `#1a1f2b`, subtle architectural dot matrix, razor-thin specular top rims) so the luminous ruby-coral `--accent` (`#ff527b`) heart reactions carry clear focal energy. Liked cards illuminate with a subtle accent perimeter tint while keeping >= 4.5:1 AA contrast across every label, counter, and filter control.

#### User state

Four boolean likes are the source of truth. Each card count is one when selected and zero otherwise; the total is their sum. Filters only change visibility. A toggle updates pressed state, the action label, card count, total, reset availability, and status text.

Reset clears all booleans and restores the All filter. If a toggle hides the focused card, move focus to the next visible card or active filter. Filled hearts, glowing radial ring + 10-particle burst, button text, counts, and pressed state together express selection; color is not the only cue.

#### Separate demonstration state

The preview heart sits inside an elevated obsidian telemetry console with no connection to the likes array. One cancellable timer advances its press, fill/particle, and completion phases. A run identifier invalidates stale callbacks. Stop, page visibility, and motion-preference changes cancel pending work. There is no initial autoplay and no repeating loop.

The historical stagger argument controls delay between preview phases. It does not change user cards. Reduced motion removes scale, ring, and particle animation while keeping all semantic actions usable.

#### Language and boundaries

A Chinese and English dictionary supplies visible text and accessible labels. Runtime language follows `?lang=zh` or `?lang=en`. Explicit text overrides are never translated or inserted as HTML. Query parameters cannot inject arbitrary content.

This demo is offline and ephemeral. Replace local state with application-owned state only during integration. Real backend use needs its own requests, error handling, optimistic-update rollback, and persistence; this template does not implement or claim them.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
