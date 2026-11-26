---
name: playful-transfer-button
description: 文件传送按钮（Playful file transfer button with SVG perimeter glow & CRC32 JSON parcel）。原生文件输入与内凹胶囊真实读取 49,152 字节块（ready→reading→complete），支持中途取消、失败重试与 Base64+CRC32 包裹下载，适用于本地文件上传与离线打包交互。
---

# 文件传送按钮

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

深灰阳极氧化精密硬件面板搭载可按压倒角机械主按钮、分段发光 LED 进度条与实时 CRC32 校验回执区，原生文件输入驱动 ready→reading→complete，可取消或失败后重试。沿归一化 SVG 圆角路径绘制短亮边，完成闭合为勾。进度必须取实际 49152 字节读取块，不许定时器虚增；此块大小可被三整除，使 Base64 可拼接。导出 JSON 保存名称、MIME、完整 Base64 和 CRC32，后者只是校验和不是签名。取消令牌在每个 await 后检查，空文件 CRC32 为 00000000，预先限制体积并回收 Blob URL。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`（1–2000 字符）、`--subtitle`（1–2000 字符）、`--accent`（3/6 位 HEX）、`--max-mb`（整数 1–64，单文件体积上限）、`--lang`（`en` / `zh`）、`-p` / `--params`（JSON 配置文件）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #a6b7ff;
  --bg: #14161b;
  --chassis: #1b1e25;
  --bay: #111318;
  --ink: #f3f5fa;
  --muted: #a2aab9;
  color-scheme: dark;
}
body {
  font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", "PingFang SC", "Segoe UI", sans-serif;
}
```

```html
<main class="skill-demo">
  <h1>Make a little delivery.</h1>
  <p>Turn any small file into a portable parcel. Everything happens on your device.</p>
  <button id="transfer" type="button">↗ Wrap this file</button>
  <button id="choose" type="button">Choose a file</button>
  <button id="cancel" type="button">Cancel</button>
  <input id="file" aria-label="file" type="file">
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Perimeter light and truthful progress

Styled as a precision-machined Teenage Engineering / Dieter Rams anodized dark hardware studio (`--bg: #14161b`, chassis `#1b1e25`, recessed instrument bays `#111318`, corner fasteners, calibrated millimeter index scale, and glowing LED status indicators). The main actuator `#transfer` is a sculpted beveled mechanical push-button with physical press travel; a normalized rounded SVG path (`pathLength="100"`) carries a short traveling `--accent` LED highlight while work is active and closes its perimeter with a checkmark on completion. Reduced-motion preferences stop the traveling highlight without changing processing.

The right OLED telemetry bay pairs a recessed segmented luminous LED progress meter (`role="progressbar"`) with tabular byte readouts, tactile hardware keycaps, and a live CRC32 checksum receipt bay. The state model is `ready → reading → complete`, with cancellation and read errors returning to a repeatable state. Never set progress from a timer: each percentage reflects a completed 49,152-byte chunk (divisible by three, so concatenated Base64 chunks decode accurately). CRC32 is a file-integrity checksum, not a cryptographic signature.

Cancellation invalidates an operation token checked after every awaited read and animation frame so a cancelled run cannot replace a newer result. Object URLs are revoked on reset. Empty files complete with CRC32 `00000000`. Cap file size (`max_mb`, 1–64) before reading to bound memory use. No upload endpoint is involved.

Keep the native file picker, focus outlines, byte progress bar, and status region when embedding. In a sandboxed iframe, downloads require `allow-downloads`.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`（可加 `--lang zh` 或 `-p params.json`）。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
