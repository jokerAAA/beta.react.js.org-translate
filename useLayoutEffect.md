
> 陷阱:
>
> useLayoutEffect 会影响性能，尽可能的使用 useEffect

useLayoutEffect 是 useEffect 的一个版本，它在浏览器重绘屏幕之前触发

tips: DOM 更新和绘制屏幕不是一回事，这里的顺序是 DOM 更新 -> 触发 useLayoutEffect(同步) -> 绘制屏幕 -> 触发 useEffect(异步)

```javascript
useLayoutEffect(setup, dependencies?)
```

## 参考

### useLayoutEffect(setup, dependencies?)

在浏览器重绘屏幕之前调用 useLayoutEffect 执行布局测量:

```javascript
import { useState, useRef, useLayoutEffect } from 'react';

function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientReact();
    setTooltipHeight(height);
  }, [])
}
```

### 参数

- setup: 类似 useEffect
- dependencies: 类似 useEffect

### 返回值

返回 undefined

### 注意事项

1. 是 Hook，调用遵循 Hook 调用规则
2. 严格模式下调用两次
3. 对象和函数依赖永远不同
4. 仅在客户端运行，ssr 不触发
5. useLayoutEffect 中的代码和其中的 state 更逻辑都会阻塞浏览器的重绘，这会让应用变慢，尽可能的使用 useEffect

## 用途

### 在浏览器重绘前测量布局

多数组件不需要知道它们在屏幕上的位置和大小来决定渲染什么，它们只返回 JSX。然后浏览器会计算出它们的布局并绘制屏幕

有时候这还不够，想象一下悬停时出现在某个元素旁边的 tooltip，它应该出现在元素上方，如果空间不够，就出现在元素下方。为了在正确的位置渲染 tooltip，你需要知道它的高度。

要实现这一点，你需要分两次渲染:

1. 在任何地方渲染 tooltip
2. 测量它的高度并决定将其放在哪里
3. 在正确的位置再次渲染 tooltip

这些流程都需要发生在浏览器绘制屏幕之前，你不想让用户看到 toolip 的移动，在浏览器重绘屏幕之前调用 useLayoutEffect 以执行布局的测量

```javascript
function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const {height} = ref.current.getBoundingClientRect();
    setTooltipHeight(height)
  }, [])
}
```

以下是其工作的步骤:

1. Tooltip 首次渲染时 tooltipHeight = 0，所以 tooltip 的位置是错的
2. React 把它放在 DOM 中，并运行 uselayoutEffect 中的代码
3. 在 useLayoutEffect 内测量 tooltip 内容的高度，并触发重新渲染
4. Tooltip 再次被渲染，并伴以正确的 tooltipheight，所以这次 tooltip 的位置是对的
5. React 在 DOM 中更新它，浏览器最终显示了 tooltip

TODO: CODE

注意到即使 Tooltip 组件必须分两次渲染（首先，将 tooltipHeight 初始化为 0，然后使用实际测量的高度），你只会看到最终的结果。这就是为什么在这个例子中你需要 useLayoutEffect 而不是 useEffect

> 注意
> 分两个阶段渲染并阻塞浏览器重绘会降低性能，尽量避免这么做


