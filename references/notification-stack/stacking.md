# 附赠配方　自动堆叠与顶层弱模糊

这份配方用于 Moect 右下角通知。玻璃的基础效果可以单独用，只有要复现多条通知的重叠与展开时才应用这一页。完整代码见 [toast.tsx](../../assets/moect-source/toast.tsx) 和 [toast.css](../../assets/moect-source/toast.css)。

## 顶层条件与最终参数

| 状态 | 底色透明度 | 不透明比例 | 背景模糊 |
| --- | ---: | ---: | ---: |
| 单条、堆叠后面的条目、悬停展开后的条目 | 浅色 88%／深色 80% | 浅色 12%／深色 20% | 5 px |
| 多条收起叠放时的最前一条 | 25% | 75% | 3 px |

只有队列至少有两条、当前条目是第零位、并且鼠标未悬停在通知区时，才给这一条附上 `data-stack-front`。CSS 把顶层规则放在深色覆盖之后，浅深色都使用顶层配方。这里的 25% 是底色透明度，文字不会跟着变透明。

```tsx
stackFront={index === 0 && items.length > 1 && !isHovered}
data-stack-front={stackFront ? "" : undefined}
```

```css
.toast-glass[data-stack-front] {
  --toast-glass-density: 75%;
  --toast-glass-blur: 3px;
}
```

## 层次与交互

- 新通知排到数组最前，同 ID 再加入时去重，队列最多保留 10 条。新建通知使用递增的 `zIndex`，保证最新的卡片绘制在前面。
- 收起时后层卡片仍在 DOM 中。以最新的卡片为第 0 层，每往后一层偏移 12 px，缩放减少 0.04，整张卡片的不透明度减少 0.15，下限为 0。右下角向上错位，因此能看到有可见透明度的背后卡片轮廓与层间模糊。不要整体隐藏后层卡片或将它们移出 DOM。
- 悬停时利用 `ResizeObserver` 测得的实际高度，按前面卡片的累积高度计算展开偏移，间隔 12 px。展开时卡片缩放和整卡不透明度都恢复为 1，最前一条的专用效果随 `data-stack-front` 一同解除。离开后延迟 150 ms 收起。
- 默认使用 Framer Motion 弹簧，刚度 350、阻尼 35、质量 1。关闭卡片后由队列重新选出最前一条。默认通知会自动消失，悬停、加载中和无限时长通知暂停自动移除。

验收顺序可以从单条开始，连续触发两条再观察错位和前层样式，悬停查看展开，离开查看复位，最后关闭一条确认剩下的单条恢复 5 px 模糊。比较 CSS 底色透明度时要等进入动画稳定，后层的整卡动画透明度属于另一个参数。

归档中的 CSS 和 TSX 是 Moect 1.3.2 工作区的最终源码副本，保留了组件原有的作者标记。Moect 源码适用归档中的 [GPL-3.0](../../assets/moect-source/LICENSE)。通知组件上游为 [shuakami/toast](https://github.com/shuakami/toast)，Moect 的相关前端改造沿用其项目中的 [QQ Chat Exporter](https://github.com/shuakami/qq-chat-exporter) 来源归属。
