# 实现与排查

## 核心做法

实际渲染层是带圆角的通知卡片。Moect 把两种主题的颜色和阴影放在 CSS 变量里，给同一个卡面混合一层有透明度的主题底色，再用 `backdrop-filter` 模糊卡面背后的页面内容。滤镜采样来自背后的像素，卡片自己的文字和操作按钮按正常清晰度绘制。

下面是精简后的等效片段。完整细节以归档中的 [toast.css](../assets/moect-source/toast.css) 为准。

```css
.toast-glass {
  --toast-glass-density: 12%;
  --toast-glass-blur: 5px;
  background-color: var(--card);
}

.toast-glass-dark {
  --toast-glass-density: 20%;
}

@supports (backdrop-filter: blur(1px)) {
  .toast-glass {
    background-color: color-mix(in srgb, var(--card) var(--toast-glass-density), transparent);
    backdrop-filter: blur(var(--toast-glass-blur)) saturate(120%);
  }
}

@media (prefers-reduced-transparency: reduce), (forced-colors: active) {
  .toast-glass {
    background-color: var(--card);
    backdrop-filter: none;
  }
}
```

完整配方用主题色和边框色混合边缘，高光仅由 `inset 0 1px` 内阴影产生。轻微的近处阴影和较宽的外阴影给卡片与页面之间留出视觉距离。Moect 使用 20 px 圆角、1 px 实线边框和 16 px 内边距，这三项由组件样式提供。自行接入普通 HTML 时，用 `border-width` 和 `border-style` 设定边框，避免内联 `border` 简写覆盖 CSS 中的边框色。

## 接入目标主题

直接把 `--card` 接到容器底色、`--primary` 接到主题强调色、`--border` 接到边框色。示例是 Moect 默认紫色主题的基础值。

```css
:root {
  --card: #fffbff;
  --primary: #7056c4;
  --border: #e6deef;
  --card-foreground: #1d1b20;
  --muted-foreground: #625b71;
}
.dark {
  --card: #1e1a25;
  --primary: #cdbbff;
  --border: #3b3247;
  --card-foreground: #e9e0ef;
  --muted-foreground: #b7adc5;
}
```

浅色卡片使用 `toast-glass`，深色卡片同时使用 `toast-glass` 和 `toast-glass-dark`。Moect 的 React 入口以 `<Toaster position="bottom-right" theme="macos"/>` 挂载通知区，主题切换时由组件读取根节点的 `dark` 类。只移植 CSS 时，目标框架可以直接管理两种类名。完整 React 组件另需 React、Framer Motion、Lucide 和 Tailwind 工具类，文件按相邻路径导入 `./toast.css`。

## 透明度看起来没生效时

1. 先确认应用实际加载的是新编译的 CSS。检查卡片的 `backgroundColor`、`backgroundImage`、`backdropFilter`，同时读取系统降低透明度和强制配色状态。回退状态使用实色底。
2. 查看中央还有没有渐变、第二个底色、发光伪元素或其他通知。多个半透明层一起绘制会抬高综合不透明度，背景可能接近实色。需要定位哪一层时，逐层临时关掉它，在同一位置比较。
3. 检查卡片和背景是否在同一渲染场景，确认中间祖先的透明度、滤镜、遮罩等属性是否影响背景的采样边界。单独的系统窗口一般采不到另一个窗口中的像素，系统窗体磨砂要使用对应平台的合成机制。
4. 用真实页面上的浅色文字、深色内容和明显色块一起验收。均匀底色天然缺少可见轮廓，强对比色条只适合作为有无滤镜的辅助测试。Moect 曾用 24 px 强模糊和多层填色，最后因背景层次太弱调整到 5 px 和单层薄底。叠放要不要保留属于产品选择。

可在开发者工具对选中的卡片运行下面这段诊断。

```js
const card = document.querySelector('.toast-glass');
const style = getComputedStyle(card);
console.table({
  background: style.backgroundColor,
  overlay: style.backgroundImage,
  backdrop: style.backdropFilter,
  density: style.getPropertyValue('--toast-glass-density').trim(),
  reducedTransparency: matchMedia('(prefers-reduced-transparency: reduce)').matches,
  forcedColors: matchMedia('(forced-colors: active)').matches,
});
```

Moect 用 Edge 和原生 WebView2 比较过亮暗主题与实际背景像素，并在浏览器模拟两种无障碍回退。若移植到别的渲染器，重新验证其 `backdrop-filter` 和 `color-mix` 支持，以及普通状态下卡面与前景文字的可读性。
