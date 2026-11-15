---
name: expanding-fab
description: 展开式快捷操作。54px 原生悬浮按钮展开三个带短暂错峰的操作，装饰背景模糊但文字与图标清晰。 Use for this interaction and adapt it to the existing framework.
---

# 展开式快捷操作

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

54px 原生悬浮按钮展开三个带短暂错峰的操作，装饰背景模糊但文字与图标清晰。打开后聚焦首项，方向键/Home/End 移动，Tab 在操作与触发器间循环；Escape、遮罩与再次点击关闭并还焦。收起项 inert。接收/发送/兑换只打开演示地址或金额表单，绝不执行转账；完成提示明确为本地预览。

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
  --accent: #202428;
  --bg: #e9ecee;
  --ink: #181a1b;
  --muted: #71787f;
  --ease: cubic-bezier(0.2, 0.85, 0.25, 1.15);
}
body {
  font-family:
}
```

```html
<main class="skill-demo">
  <h1>A little button. More possibilities.</h1>
  <p>Three quick actions, one quiet home. A wallet interaction with room to breathe.</p>
  <input id="amount" aria-label="amount" type="number">
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Design and behavior

A pale gray canvas holds a compact wallet and large introductory type. The black 54 px floating trigger anchors three colored actions. Action labels and icons animate upward with a short stagger; the background blurs while the menu is open. The trigger rotates into a close glyph without disappearing.

#### Interaction contract

- Clicking the trigger opens the menu and focuses its first action. Escape, the backdrop, or the trigger closes it and restores focus.
- Arrow keys cycle menu items; Home and End jump to the first and last. Tab loops through the three actions and trigger while expanded.
- Hidden actions use `inert`, so they cannot receive keyboard focus.
- Receive opens a sample address dialog with clipboard support and a visible fallback. Send and exchange open amount forms with native minimum, maximum, and step validation. Completion announces a preview; no transaction is sent.
- Native dialogs provide modal focus management and Escape support. Closing returns focus to the trigger.
- Reduced motion removes transitions. The layout stacks on small screens and keeps the wallet inside the viewport width.

#### Customization

Colors accept only hexadecimal CSS values. Text is escaped in HTML and serialized safely in inline JavaScript. Configuration rejects unknown keys, empty labels, and invalid counts.

The template is standalone HTML/CSS/JavaScript with system fonts and inline SVG. CSS easing approximates an elastic reveal; there is no physics engine. The sample wallet, address, balances, and exchange behavior are UI demonstrations. Clipboard availability depends on browser permissions and secure context; when unavailable the address stays visible in the notification.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 templates/index.template.html，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
