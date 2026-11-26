---
name: tactile-scene-toggle
description: 昼夜场景拨片（Tactile day/night scene toggle switch & CSS illustration）。两个原生按钮共享滑块并联动纯 CSS 日夜小屋插画，支持键盘方向键切换与添云/许愿互动，适用于主题切换器、微交互展示与情绪卡片。
---

# 昼夜场景拨片

## 独立使用 / Standalone use

若仅提供本 SKILL.md，请按以下规范在目标项目（Vue / React / 原生 HTML）中直接实现，无需依赖外部生成器。沿用项目的技术栈、主题、路由与组件约定；先交付真实可操作的界面，不用截图或固定播放视频替代交互。

If only this SKILL.md is supplied, implement the specification directly in the target Vue, React or plain HTML project. The generator, template and design reference in the full ZIP are optional reference implementations, never prerequisites.

## 核心实现规范

两个原生按钮共享带 `--accent` 描边与高光斜面的立体滑块，左右/Home/End 同步选择与焦点；按钮宽度和默认 124px 滑块位移一致。拱形立体绘本微缩窗景以同一日夜状态联动金色时刻渐变天空与极光靛蓝夜空、层叠山丘、松柏、日月、袅袅炊烟与四格暖光窗灯；白天完全隐藏月牙遮罩。云最多加五朵，夜晚暂隐藏并保留，许愿增加本次访问的固定星点且封顶。流星 1.3 秒后清理；减少动态效果取消过渡与流星但仍添加静态星点。

## 参数与内容边界

- 将标题、副标题、强调色与场景数据作为可配置输入；未指定时采用下面的设计参考，而不强制迁移用户的技术栈。
- 完整包的可选 CLI 参数：`--title`、`--subtitle`、`--accent`（3/6 位 HEX，驱动滑块描边、活跃图标、标题圆点与操作按钮强调色）、`--bg`（3/6 位 HEX）、`--lang`（`en` / `zh`）、`-p` / `--params`（JSON 配置文件）。命令行只是便利入口；网页定制后的 HTML 可直接离线打开。
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
  --accent: #bd743e;
  --bg: #f4f1e9;
  --ink: #272e28;
  --muted: #4b544b;
}
body {
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Hiragino Sans GB", "Segoe UI", sans-serif;
}
```

```html
<main class="skill-demo">
  <h1>A world in a little switch.</h1>
  <p>Slide from golden afternoons to quiet evenings. There is a little magic in both.</p>
  <button id="day" type="button">☀ Daylight</button>
  <button id="night" type="button">☾ Moonlight</button>
  <button id="magic" type="button">Add a cloud +</button>
</main>
```

## Detailed implementation reference (English)

The following specification is included in full so this document works without accompanying files. Preserve its state machine, geometry, numeric bounds and integration boundaries when adapting it.

### Component design notes

#### Storybook diorama atmosphere
Framed in a beveled arched gallery shadow-box on warm matboard (`--bg: #f4f1e9`), the scene transitions smoothly between a multi-stop golden-hour sky (`#7aaec4` → `#f0d49c` → `#f6be82`) and a twilight indigo aurora sky (`#0c162b` → `#244064` with soft teal-indigo aurora veil). The architectural storybook cottage features layered plaster shading, a pitched slate-terracotta roof, brick chimney with gentle rising smoke wisps, an arched oak door with brass knob, and a 4-mullion paned window that radiates warm amber light across the night meadow.

#### Segmented control
Two native buttons share one sculptural elevated sliding thumb with a crisp 2 px `--accent` rim, subtle `--accent` surface tint, top specular bevel, and luminous `--accent` active icon. `aria-pressed` reflects the current option, while left/right and Home/End switch and focus the chosen side. Keep button widths (`120px`) and the thumb's `124px` offset aligned when resizing.

#### Scene behavior
The same `.night` state class crossfades the sky, aurora, hills, cypress trees, sun/crescent moon, chimney smoke, and glowing cottage window. The lunar cutout is completely hidden in daylight. The scene's accessible description follows the selected state. Up to five extra clouds are retained when switching into night and reappear in daylight. Each nighttime wish launches a 1.2 s shooting star and adds a stationary star (capped at 60).

#### Motion
Under `prefers-reduced-motion: reduce`, transitions, chimney smoke motion, and shooting-star animations are disabled while stationary wish stars and cloud additions remain fully functional. Zero external images, fonts, or network calls are required.

## 可选完整包工作流 / Optional package workflow

已获得完整 ZIP 时，可以参考 [references/design.md](references/design.md) 与 `templates/index.template.html`，并执行 `python3 scripts/generate.py -o index.html`（可加 `--lang zh` 或 `-p params.json`）。仅获得本文时请直接实现上述约束，不等待或虚构这些附件。复用本 Skill 不意味着获得部署网站、发送消息或操作他人数据的授权。
