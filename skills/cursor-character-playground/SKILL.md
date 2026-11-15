---
name: cursor-character-playground
description: 指针进化游乐场。提供六种独立 SVG 指针形状，时间滑杆选择形状，尺寸和墨色改变同一 SVG 构建器。 Use for this interaction and adapt it to the existing framework.
---

# 指针进化游乐场

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

提供六种独立 SVG 指针形状，时间滑杆选择形状，尺寸和墨色改变同一 SVG 构建器。试验场含三个只能捕获一次的原生目标按钮，切换年代重置试验。指针位置按 viewBox 的真实热点对齐 clientX/clientY，不把尖端强制钳制在容器内部；键盘 Tab/Enter 完成同等试验。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`, `--accent`, `--era`, `--size`。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #454bc5;
  font-family: Arial, "PingFang SC", sans-serif;
  color: #282727;
  background: #faf9f6;
  color-scheme: light;
}
```

```html
<main class="skill-demo">
  <h1>小小指针，也有进化史。</h1>
  <p>从像素阶梯到柔和曲线，一支箭头，换了许多种性格。拖动时间轴，再亲自试用你喜欢的那一款。</p>
  <input id="era" aria-label="era" type="range">
  <input id="size" aria-label="size" type="range">
  <button id="try" type="button">试用这支指针 ↗</button>
  <button id="save" type="button">带走 SVG ↓</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Rendering and interaction

Six independently drawn cursor silhouettes form an illustrative shape timeline: line outline, pixel staircase, sharp fill, precise contour, soft corner, and expressive arrow. The years act as narrative markers, not claims that these drawings are exact historical system assets. Do not add product logos or system-specific icon files. Shape names are generic and switch with the language query.

The histogram is decorative; a native range input carries pointer, touch, and keyboard semantics. Its accessible value includes both the year and shape name. All SVG paths are fixed internal data, while size is bounded and ink is drawn from validated palette values. SVG export uses the same shape builder as the specimen and includes explicit dimensions.

Trial mode reveals three semantic target buttons. Each target can be captured once; changing eras cancels the trial, resets its count, and hides targets. Pointer motion moves the specimen only during a trial. Tab and Enter provide equivalent target capture, with focus moving to the next available target or replay control. Replaying clears all captured state. There are no animation loops; reduced motion disables size transitions. Preserve conventional system pointer behavior outside the trial arena.

During pointer trials, align each silhouette’s semantic tip to the event coordinates using its viewBox hotspot; only the noninteractive specimen is centered. Rounded tips use the topmost point of their quadratic contour. Do not inset or clamp the pointer position away from the actual event. Resizing must preserve this alignment. Mouse clicks keep the cursor at the click location; keyboard capture may move it to the focused target. Validate tip alignment across all six forms at minimum and maximum size, including near arena edges.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 templates/index.template.html，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
