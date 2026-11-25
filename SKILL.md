---
name: frosted-glass-ui
description: 在 Web 或 Chromium 桌面界面实现、复现或排查半透明磨砂玻璃。适用于悬浮提示、浮层和卡片，包含 Moect 的实测参数及可选的通知堆叠方案。
---

# 半透明磨砂玻璃

从 [Moect 的实际 CSS](assets/moect-source/toast.css) 取样。把半透明底色和 `backdrop-filter` 放在同一个圆角表面上。文字、图标和按钮照常绘制。用边框、顶部内阴影和外阴影表现玻璃轮廓。

| Moect 状态 | 底色透明度 | 高斯模糊 |
| --- | ---: | ---: |
| 浅色普通卡片 | 88% | 5 px |
| 深色普通卡片 | 80% | 5 px |

底色由 `color-mix(in srgb, var(--card) 密度, transparent)` 生成。密度就是不透明度，底色透明度等于 100% 减密度。普通卡片的 `saturate` 为 120%。表内参数是经用户确认的 Moect 配方，移植时可以按目标背景和任务要求调整。

1. 找到实际放在页面内容上的卡片层，确认它后面有同一渲染场景里的内容可采样。
2. 将 CSS 的 `--card`、`--primary`、`--border` 接入目标主题。为组件保留实色底、`@supports` 检测和系统无障碍回退。
3. 用真实的列表、文字和色块验证。看轮廓能否透出、细节是否被柔化，再核对前景文字和操作按钮。桌面项目优先用实际内嵌渲染器验证。

[接入方式、排查顺序和验证代码](references/glass-implementation.md)涵盖主题变量、等效 CSS 和曾经遇到的遮盖问题。

需要复现自动堆叠时，阅读单独附赠的[通知堆叠配方](references/notification-stack/stacking.md)。[完整 React 通知源码](assets/moect-source/toast.tsx)与 CSS 一同归档，包含局部样式、动画、计时和展开逻辑。
