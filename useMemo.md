
useMemo 是一个 React Hook，允许你在重复渲染时缓存一个计算结果

```javascript
const cachedValue = useMemo(calculateValue, dependencies)
```

## 参考

### useMemo(calculateValue, dependencies)

在组件顶层调用 useMemo 可以在重复渲染中缓存一个计算结果:

```javascript
import { useMemo } from 'react';

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  // ...
}
```

### 参数

- calculateValue: 你想缓存的计算值的函数，它应该是个纯函数，不接受参数，并返回任意值。React 会在初次渲染时调用函数，在下次渲染时，如果依赖项数组没有变，React 会再次返回相同的值；否则，React 会调用 calculateValue，返回其结果，并将其在下次渲染中复用
- dependencies: calculateValue 内代码中引用的所有的响应式值的列表，响应式值包括 props、state 和 组件内部定义的变量及函数，如果你为 React 配置了 linter，它会检查依赖项数组中是否包含了所有必须的响应式值，依赖项数组长度必须是固定的，且格式为行内数组，React 会用 Object.is 在重复渲染时比较值是否发生了变化

### 返回值

初次渲染时，useMemo 返回 calculateValue 的调用结果。

在以后的渲染中，要么返回上次渲染的结果，要么返回新的调用结果

### 注意事项

- 是一个 Hook，只能在组件顶层调用
- 严格模式下，React 会调用两次来帮助你发现问题
- 除非必要，React 不会丢弃 useMemo 的缓存值

## 用途

### 跳过昂贵的重复计算

要在重复渲染时缓存一个计算结果，在组件顶层将其用 useMemo 包装:

```javascript
import { useMemo } from 'react';

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

需要为 useMemo 传递两个参数:

1. 计算函数: 没有参数，返回你想计算的结果
2. 依赖数组: 在计算函数内使用的响应式值

TODO: TRANSLATE 略，一下部分和 useCallback 大同小异，几乎没有区别
