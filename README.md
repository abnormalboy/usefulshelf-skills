# UsefulShelf Agent Skills

Public Agent Skills distribution repository for [UsefulShelf](https://usefulshelf.com) (33 deterministic local/browser tool skills + stateless MCP gateway).

## Quick Install

```bash
npx skills add abnormalboy/usefulshelf-skills
```

## Stateless MCP Gateway

- Endpoint: `https://usefulshelf.com/api/mcp`
- Manifest: `https://usefulshelf.com/api/skills/manifest`
- Directory: `https://usefulshelf.com/skills`

## Exported Skills (33)

| Skill | Description | Web App |
| --- | --- | --- |
| [`bite-size-button`](./skills/bite-size-button/SKILL.md) | 咬一口按钮。用原生按钮承载六次点击；每次将固定圆形加入 SVG mask，真实切掉胶囊的一角。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/bite-size-button) |
| [`cartridge-memory-deck`](./skills/cartridge-memory-deck/SKILL.md) | 卡带记忆时间轴。四张卡带使用原生横向滚动与 scroll-snap；选中后更新唯一故事区域。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/cartridge-memory-deck) |
| [`cosmic-voice-orb`](./skills/cosmic-voice-orb/SKILL.md) | 宇宙声纹球。Canvas2D 用圆形裁切、径向底色、65 条程序丝带、星点和边缘高光构成玻璃球。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/cosmic-voice-orb) |
| [`cursor-character-playground`](./skills/cursor-character-playground/SKILL.md) | 指针进化游乐场。提供六种独立 SVG 指针形状，时间滑杆选择形状，尺寸和墨色改变同一 SVG 构建器。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/cursor-character-playground) |
| [`dark-docs-navigation`](./skills/dark-docs-navigation/SKILL.md) | 深色文档导航。深色背景、2px 强调色标签线、280px 侧栏和可收缩虚线树组成布局。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/dark-docs-navigation) |
| [`dot-spiral-studio`](./skills/dot-spiral-studio/SKILL.md) | 点阵螺旋工坊。固定网格中心，只更新圆角单元的大小与颜色。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/dot-spiral-studio) |
| [`expanding-fab`](./skills/expanding-fab/SKILL.md) | 展开式快捷操作。54px 原生悬浮按钮展开三个带短暂错峰的操作，装饰背景模糊但文字与图标清晰。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/expanding-fab) |
| [`glossy-icon-sweets`](./skills/glossy-icon-sweets/SKILL.md) | 糖果光泽图标集。六枚原创 SVG 图标由球体、符号、镜面椭圆和地面阴影组成。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/glossy-icon-sweets) |
| [`heart-reaction-grid`](./skills/heart-reaction-grid/SKILL.md) | 点赞反馈动效。四张场景卡片各保存一个布尔点赞值，单项计数只能是 0 或 1，总数取和。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/heart-reaction-grid) |
| [`ice-fracture-lab`](./skills/ice-fracture-lab/SKILL.md) | 冰面裂纹实验室。八边形晶体切成 24 个不重叠三角形；破碎量驱动径向位移、旋转和完整度，复原必须回到同一顶点。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/ice-fracture-lab) |
| [`icon-sticker-workbench`](./skills/icon-sticker-workbench/SKILL.md) | 图标贴纸工作台。侧栏从图标轨道展开为有标签的集合，选择集合只更换贴纸托盘，不删除画布现有对象。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/icon-sticker-workbench) |
| [`invite-code-flow`](./skills/invite-code-flow/SKILL.md) | 邀请码揭纸动效。邀请码固定在底层，覆盖纸从右边缘向左揭开，拖动与点击共享 revealed 状态。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/invite-code-flow) |
| [`kinetic-orb-gallery`](./skills/kinetic-orb-gallery/SKILL.md) | 动态圆球画廊。Canvas2D 在椭圆轨道上呈现八种程序几何球体，伸展滑杆改变轨道半径。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/kinetic-orb-gallery) |
| [`liquid-tab-menu`](./skills/liquid-tab-menu/SKILL.md) | 液态快捷菜单。三个 72px 装饰圆通过 Gaussian blur 加 alpha threshold 在分离时粘连；清晰图标和透明原生按钮放在独立层，命中位置随 --size/--spread 同步。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/liquid-tab-menu) |
| [`luminous-plinko-board`](./skills/luminous-plinko-board/SKILL.md) | 发光弹珠实验台。模拟坐标固定为 640×460，与显示尺寸解耦；使用 120Hz 有界累加器、先更新速度后更新位置的半隐式积分。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/luminous-plinko-board) |
| [`message-state-playground`](./skills/message-state-playground/SKILL.md) | 消息状态游乐场。消息以稳定 ID 管理 draft/sending/delivered/failed/offline；重试原消息不能创建重复项。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/message-state-playground) |
| [`modular-tech-hero`](./skills/modular-tech-hero/SKILL.md) | 模块化科技首屏。近黑底、绿色强调色和细分割线组合两列首屏与代码终端，下方连续三格指标和折叠文档检查器。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/modular-tech-hero) |
| [`node-connection-studio`](./skills/node-connection-studio/SKILL.md) | 节点接线工作台。点阵画布包含暖冷胶囊节点，边保存稳定端点与路径，支持直连、分支合流、回环预设。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/node-connection-studio) |
| [`orbit-agency-hero`](./skills/orbit-agency-hero/SKILL.md) | 轨道作品集首屏。暖纸色首屏以大字为中心、原创 SVG 项目围绕旋转。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/orbit-agency-hero) |
| [`paper-receipt-studio`](./skills/paper-receipt-studio/SKILL.md) | 纸条收据工坊。离屏 Canvas 绘制真实收据纹理，屏上用 12×18 三角网格和局部正弦折痕映射纸张，明确不是布料物理。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/paper-receipt-studio) |
| [`parallax-laser-card`](./skills/parallax-laser-card/SKILL.md) | 创建可调纵深和轮廓光强度的视差镭射卡片，用于网页卡片、作品展示和交互效果。用户提及视差轮廓、镭射卡片、全息轮廓光或可调 3D 层叠效果时使用；保留项目现有框架和设计系统。 | [Web Tool](https://usefulshelf.com/tools/parallax-laser-card) |
| [`perspective-card-carousel`](./skills/perspective-card-carousel/SKILL.md) | 透视弧形卡片轮播。七张卡片处于隔离 perspective 场景，原生按钮包住 SVG 海报和不透明深色标题条。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/perspective-card-carousel) |
| [`playful-transfer-button`](./skills/playful-transfer-button/SKILL.md) | 文件传送按钮。原生文件输入和内凹胶囊描述 ready→reading→complete，可取消或失败后重试。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/playful-transfer-button) |
| [`poster-print-playground`](./skills/poster-print-playground/SKILL.md) | 四色海报印刷间。四个 400×520 原创 SVG 场景分别表现透视卡、数据节点、人像对话和产品爆炸层，不能只换颜色冒充不同构图。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/poster-print-playground) |
| [`prism-refraction-lab`](./skills/prism-refraction-lab/SKILL.md) | 棱镜折光实验室。整幅场景用 SVG 表现：白光进入三角左面、穿过内部、从右面分成七条连续柔混的光楔；单色选择使用同一几何的一束，不另造方向。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/prism-refraction-lab) |
| [`scroll-hide-toolbar`](./skills/scroll-hide-toolbar/SKILL.md) | 滚动隐藏悬浮导航。外层设备画布包含独立可滚动图标网格，短胶囊与右侧加号分开。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/scroll-hide-toolbar) |
| [`story-portfolio-canvas`](./skills/story-portfolio-canvas/SKILL.md) | 故事作品集画布。六幅原创灰阶 SVG 共享筛选后的项目 ID，Scatter/Grid/Orbit 只改变同一组原生按钮布局。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/story-portfolio-canvas) |
| [`svg-highlight-nav`](./skills/svg-highlight-nav/SKILL.md) | 手绘圈选导航。暖白编辑式排版中，每个导航按钮拥有自己的三条 SVG 手绘路径，不依赖全局坐标。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/svg-highlight-nav) |
| [`tactile-scene-toggle`](./skills/tactile-scene-toggle/SKILL.md) | 昼夜场景拨片。两个原生按钮共享白色滑块，左右/Home/End 同步选择与焦点；按钮宽度和默认 124px 滑块位移一致。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/tactile-scene-toggle) |
| [`tilted-tab-notebook`](./skills/tilted-tab-notebook/SKILL.md) | 倾斜标签笔记本。四个原生标签使用 aria-selected/controls 与单一 roving tabindex。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/tilted-tab-notebook) |
| [`unified-inbox-desk`](./skills/unified-inbox-desk/SKILL.md) | 轻巧收件工作台。默认以暖白窄卡、右侧深色问题气泡、回答和五通道比例条为主构图，收件箱是可切换的第二视图。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/unified-inbox-desk) |
| [`usefulshelf-tools`](./skills/usefulshelf-tools/SKILL.md) | Stateless deterministic developer utilities from UsefulShelf (usefulshelf.com) via MCP Streamable HTTP (/api/mcp) or JSON-RPC CLI calls — JSON validate/format/repair, 5-field Cron schedule explainer, JWT claim & expiry inspector, UTM campaign URL builder, and MD5/SHA/HMAC digest calculator. | [Web Tool](https://usefulshelf.com/tools/usefulshelf-tools) |
| [`word-dissolve-poster`](./skills/word-dissolve-poster/SKILL.md) | 文字粒子海报。离屏 Canvas 按字形 alpha 采样粒子，不用矩形粒子冒充文字。 Use for this interaction and adapt it to the existing framework. | [Web Tool](https://usefulshelf.com/tools/word-dissolve-poster) |
