---
name: message-state-playground
description: 消息状态游乐场。消息以稳定 ID 管理 draft/sending/delivered/failed/offline；重试原消息不能创建重复项。 Use for this interaction and adapt it to the existing framework.
---

# 消息状态游乐场

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

消息以稳定 ID 管理 draft/sending/delivered/failed/offline；重试原消息不能创建重复项。正常、失败一次、离线三个场景改变真实状态迁移，延迟只用于可读反馈。附件是本地副本，卡片收藏与三音符合成音频只在页面内运作；音频需用户触发。手机预览与事件日志观察同一状态源，取消旧操作防止异步覆盖。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`, `--accent`, `--contact`, `--latency`。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #5b95ad;
  font-family: ui-sans-serif, system-ui, sans-serif;
  color: #283431;
  background: #eff0eb;
}
```

```html
<main class="skill-demo">
  <h1>每条消息，都有下一步。</h1>
  <p>写一句话，试一次失败，再重试。每一步都留在左侧的轨迹里；所有消息只存在于这个小实验。</p>
  <select id="transport" aria-label="晴空万里 · 正常送达 一个小意外 · 下一条失败一次 暂时离线 · 等信号回来"></select>
  <button id="receive" type="button">↙ 收到一句问候</button>
  <button id="card" type="button">▧ 放入风景卡片</button>
  <button id="voice" type="button">♫ 放入三音符便条</button>
  <button id="attach" type="button">⌁ 选择本地附件</button>
  <input id="file" aria-label="file" type="file">
  <button id="reset" type="button">重新开始 ↺</button>
  <button id="retry" type="button">重试这条消息 ↻</button>
  <textarea id="draft" aria-label="消息内容"></textarea>
  <button id="send" type="button">发送消息</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Inspectable message states

The interface pairs a live state strip and bounded event trace with a phone-shaped chat preview. State is explicit: idle, composing, sending, delivered or failed. Delivery is a local simulation with a configurable latency; there is no server, real recipient or account connection. Preserve that explanation in the UI when adapting this template.

One outgoing message can be pending at a time. Capture its content before transmission and preserve its identity on failure and retry. A fail-once scenario resets the next scenario to success, while offline remains offline until the user changes it. During sending, the draft and scenario cannot change; after failure the scenario can change but the original draft remains protected until retry or reset. A generation token invalidates stale completion callbacks after reset, and reset also clears the timer. Repeated retries must update the same bubble rather than duplicate it.

Rich examples have real local behavior: landscape cards can be saved and unsaved in the current session; the audio memo is an explicitly labeled, synthesized three-note PCM WAV with native playback controls; a selected local file becomes a downloadable object URL preserving its bytes. Files are never uploaded. A 10 MB cap bounds accidental attachment size. Reset revokes all attachment and audio URLs. User text and file names enter the DOM through textContent, not HTML markup.

The feed scrolls independently inside the phone. Status announcements and a visible uppercase state label make transitions inspectable without relying on animation. Reduced motion removes bubble arrival effects. Validate success, fail-once → retry without a duplicate, offline → retry while still offline, reset during sending, card toggling, audio decoding and local attachment byte equality.

#### Localized regions

Both major regions have localized accessible names: Message state controls and Local chat preview in English, and their Chinese counterparts in the default interface. These names identify the state controls separately from the simulated conversation. Language changes do not alter message identity, retry behavior or audio/attachment processing.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 templates/index.template.html，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
