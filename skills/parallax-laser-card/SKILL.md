---
name: parallax-laser-card
description: 创建可调纵深和轮廓光强度的视差镭射卡片，用于网页卡片、作品展示和交互效果。用户提及视差轮廓、镭射卡片、全息轮廓光或可调 3D 层叠效果时使用；保留项目现有框架和设计系统。
---

# 视差轮廓镭射卡片

实现一个真实可交互的镭射卡片：彩色轮廓层叠、指针驱动倾斜、独立控制纵深和光强。不要用效果截图替代交互。

## 参考实现

完整包中的 [assets/LaserCard.vue](assets/LaserCard.vue) 是与分享页面同步的 Vue 3 单文件组件，设计参数与配色规格另见 [references/design.md](references/design.md)。Vue 项目优先复制并按现有界面调整；其他框架迁移相同的 CSS 分层与生命周期逻辑，不为这个效果更换技术栈。

如果用户只提供了本 SKILL.md，按下面的实现约束构建，不依赖缺失附件。无需付费素材、WebGL 或动画库。

## 参数与外观

| 参数 | 范围 | 默认 | 行为 |
| --- | --- | --- | --- |
| depth | 0–100 px | 48 px | 轮廓在 Z 轴上的总分层深度 |
| glow | 0–100% | 70% | 彩色轮廓与辉光的强度，0 时完全隐藏轮廓光 |
| autoRotate | 布尔值 | false | 无需指针的缓慢视角演示，可随时暂停 |
| lang | `'zh' \| 'en'` | `'zh'` | 控制默认标题文案与无障碍描述语言 |
| brand | 字符串 | `'USEFUL SHELF / INTERACTION LAB'` | 顶部细字品牌标识，替换为项目自有名称 |
| titlePrefix | 字符串 | 按 `lang` 回退（`光，有了` / `Light, in`） | 卡片主标题第一行 |
| titleStrong | 字符串 | 按 `lang` 回退（`纵深。` / `depth.`） | 卡片主标题强调行 |
| caption | 字符串（支持 `\n` 换行） | `'PARALLAX\nCONTOUR STUDY'` | 底部说明文案 |

同时提供两条有可见标签、数值和键盘支持的 range 滑杆。预设建议：柔和 24/35、经典 48/70、强烈 85/100。重置恢复 48/70 并关闭自动演示。

卡片保持可读的深色底材，镭射颜色仅用于轮廓，页面外壳沿用现有主题。建议宽度 280–330 px、宽高比 0.79、圆角 22 px。替换成用户的文字，不将参考文案作为必需品牌。桌面并排放预览与参数，窄屏垂直排列。

## 分层与轮廓

- 静止的外部容器设置 `perspective: 1100px`，卡片及轮廓容器设置 `transform-style: preserve-3d`。
- 背板、表面反光、12 层轮廓、文字、外边缘各自独立。不要在保留 3D 的祖先上设置 `overflow: hidden`、整体 `opacity < 1` 或 `filter`，它们可能使层叠平面化；裁切或滤镜放在不承载子层的装饰节点上。
- 轮廓序号 i 从 0 到 11，内缩 `6*i px`（上下）和 `4*i px`（左右），圆角 `43% 57% 60% 40% / 44% 34% 66% 56%`。变换为 `translateZ(depth * i / 11) rotate(i * -7deg)`。depth 为零时所有轮廓处于同一平面。
- 轮廓伪元素用约 1.5 px padding 和 `conic-gradient`：薄荷青、冰蓝、淡紫、粉色、浅金、薄荷青。通过内容盒与边框盒的 mask 相减，仅保留彩色边缘，同时提供 WebKit mask-composite 兼容写法。
- 第二个伪元素加宽至约 4 px 并模糊 5 px 形成辉光，两层 opacity 都取 `glow / 100`。模糊只作用于辉光装饰，不作用于卡片本体。
- 文字位于 `translateZ(depth + 8px)`，独立于辉光，不能因为强光失去对比度。装饰层不拦截指针。

## 指针与动画

使用**静止外部容器**的矩形计算指针位置，避免读取倾斜卡片导致反馈抖动：

```text
x = clamp((clientX - rect.left) / rect.width * 2 - 1, -1, 1)
y = clamp((clientY - rect.top) / rect.height * 2 - 1, -1, 1)
rotateX = -y * 12 degrees
rotateY = x * 16 degrees
lightX = 50 + x * 35 percent
lightY = 50 + y * 35 percent
gradientAngle = x * 55 degrees
```

连续坐标不进入框架的渲染状态；存于局部变量或 ref，用 requestAnimationFrame 更新卡片的 CSS 自定义属性。每帧 `current += (target - current) * 0.12`，接近目标就停止请求帧。滑杆数值可以使用框架响应式状态。

自动演示用正弦与余弦产生有限幅度目标值；指针进入时让指针接管。pointerleave、pointerup、pointercancel 复位，关闭自动演示后平滑回正。触屏保留 `touch-action: pan-y`，不要阻止整页滚动。

使用 IntersectionObserver 与 visibilitychange 在组件不可见或页面隐藏时停止循环。组件销毁时取消 requestAnimationFrame、断开观察器并移除所有监听器。

尊重 `prefers-reduced-motion: reduce`，初次挂载与设置变化时均处理；禁用视差和自动旋转，保留静态轮廓、文字和光强调节。CSS 同时提供静态回退。

## 验收

1. 指针从左向右移动时，旋转方向和光源随之改变；离开卡片后回正。
2. 分别把纵深设为 0 和 100，在同一倾角下能看到层间距离变化。
3. 光强 0 隐藏所有彩色轮廓光，100 恢复清晰镭射，文字始终可读。
4. 滑杆可用方向键操作，预设与重置同步更新数值；自动演示可暂停。
5. 390 px 手机宽度不出现水平滚动，触摸页面仍能正常纵向滚动。
6. 减少动态效果、离屏和组件卸载时无持续无用动画；控制台无异常。

交付完整实现及可运行示例，说明已验证的交互。复用本效果不意味着允许发布网站或发送外部消息，遵循当前用户任务的授权范围。

## English standalone implementation summary

Implement directly in the existing Vue, React or HTML stack without missing attachments. Use a stationary 1100px perspective container; 12 preserved-3D contour layers interpolate Z from 0 to depth (default 48px, range 0–100), rotating each layer by -7 degrees. Keep text at depth+8px. Glow ranges 0–100%, default 70%. Pointer coordinates normalized to [-1,1] drive rotateX=-y*12, rotateY=x*16, lightX=50+x*35 and lightY=50+y*35. Smooth each frame with current+=(target-current)*0.12, stop when settled, hidden or unmounted. Reduced motion disables tilt and autoplay while preserving the static contours and controls. Keep keyboard sliders, visible labels/focus and a usable 390px layout; do not flatten preserve-3D ancestors with opacity, filters or overflow clipping.
