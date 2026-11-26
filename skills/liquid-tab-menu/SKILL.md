---
name: liquid-tab-menu
description: 液态快捷菜单（SVG Gooey 滤镜粘连胶囊与日志弹窗）。三个 72px 装饰圆借高斯模糊与 alpha 阈值产生液态融合，清晰图标层与原生 dialog 保障键盘无障碍。Trigger when building gooey/liquid metaball tab menus, organic floating action hubs, SVG filter morphing navigation, or journal quick-add bars.
---

# 液态快捷菜单

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

三个 72px 装饰圆通过 Gaussian blur 加 alpha threshold 在分离时粘连；清晰图标和透明原生按钮放在独立层，命中位置随 --size/--spread 同步。收起时两侧 inert/hidden；方向键/Home/End 可展开并选中，Escape 关闭还焦。豆子与杯子打开原生 dialog，保存 60 字名称与 100 字备注到内存列表，刷新清空；不伪称云同步。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`-p/--params`（JSON 配置）、`--lang`（`en` 或 `zh`）、`--title`（1–180 字符）、`--subtitle`（1–180 字符）、`--accent`（3/6 位 HEX）、`--bg`（3/6 位 HEX）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --bg: #eee9e2;
  --accent: #642d2b;
  --spread: 89px;
  --size: 72px;
  --ease: cubic-bezier(0.25, 1.15, 0.4, 1);
}
body {
  font-family: Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
}
```

```html
<main class="skill-demo">
  <h1>Small rituals, worth keeping.</h1>
  <p>A quiet home for your favorite coffees. Open the liquid menu to add beans or remember a cup.</p>
  <input id="coffee" aria-label="coffee" type="text">
  <input id="notes" aria-label="notes" type="text">
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Design and behavior

A warm neutral canvas holds a burgundy coffee journal and a serif editorial title. Three 72 px circles separate horizontally from the center button. A Gaussian blur plus alpha threshold joins the decorative shapes as they travel, while transparent buttons on a separate layer keep icons and input sharp.

The side controls are inert and hidden while collapsed. Opening updates `aria-expanded` and makes them operable. Arrow keys, Home, and End open the menu if necessary and move among beans, hub, and cup. Escape closes the menu and focuses the hub. Hover and focus share attraction styling; a reduced-motion preference removes both transitions and the SVG filter.

#### Working demonstration

Beans and cup actions open a native dialog with a required coffee name and optional note. Names accept up to 60 characters and notes up to 100; whitespace-only names are rejected. Saved entries replace the empty state. Each entry can be removed, and Clear journal resets the demonstration. Status updates use a polite live region. Entries are rendered with DOM text APIs, never HTML interpolation.

The coffee journal intentionally stores data only in memory. There is no persistence, account, upload, or backend. Refreshing resets the journal. All graphics, fonts, styles, and scripts work without network requests.

#### Geometry and limits

Keep hit targets aligned with blob centers when changing `--spread` or `--size`. SVG filter behavior may differ slightly by browser; the functional controls work without the filter. The effect uses CSS interpolation and SVG compositing rather than a fluid simulator. Text and colors are configurable through a validating Python generator; unknown keys and unsafe CSS values are rejected.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
