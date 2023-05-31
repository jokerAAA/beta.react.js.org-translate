
memo 让组件的 props 保持不变时跳过重复渲染

```javascript
const MemoizedComponent = memo(SomeComponent, arePropsEqual);
```

## 参考

### memo(Component, arePropsEqual?)

将组件包装到 memo 会得到一个组件的缓存版本，只要它的 props 版本没变，即使它的父组件重新渲染了，它也不会重新渲染。但是 React 仍然可能会重新渲染它：缓存是一项性能优化的手段，而不是一个保证:

```javascript
import {memo} from 'react';

const SomeComponent = memo(function SomeComponent(props)) {

}
```

### 参数

- Component: 你希望缓存的组件，memo 并不会修改这个组件，而是返回一个新的缓存组件。接受任何有效的 React 组件，包括函数组件和 forwardRef 组件
- arePropsEaual: 可选参数，一个接受两个参数的函数 —— 组件之前的 props 和当前的 props。如果新旧 props 一致应该返回 true，及组件在接受新旧 props 时渲染的结果是一样的，否则它应该返回 false。通常来说，你不用手动指定这个函数，默认情况下 React 会通过 Object.is 来对比 props

### 返回值

memo 返回一个新的 React 组件，和你提供给 memo 的组件几乎一样，除了当其父组件重复渲染且其 props 变化时才触发它的重新渲染。

## 用途

### props 未变时跳过重复渲染

通常来说父组件的重新渲染导致子组件的重复渲染，而你可以通过 memo 创建一个父组件重新渲染时只要其 props 不变就不会重复渲染的子组件，这种组件通常被成为缓存组件

要缓存一个组件，将其包装到 memo 中，并用它的返回值替换你的原始组件

```javascript
const Greeting = memo(function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
});

export default Greeting;
```

一个 React 组件应该始终是一个纯函数，这意味着当其 props、state 和 context 没变时它必须返回相同的输出，通过使用 memo，你告诉 React 你的组件符合这个需求，所以只要它的 props 没有变，React 就不会重复渲染。即使使用了 memo，如果组件自身的 state 或其使用 context 发生了变化，他也会重新渲染

参考下面的例子，Greeting  组件在 name 变化时会重新渲染，但在 address 变化时不会:

```javascript
import { memo, useState } from 'react';

export default function MyApp() {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  return (
    <>
      <label>
        Name{': '}
        <input value={name} onChange={e => setName(e.target.value)} />
      </label>
      <label>
        Address{': '}
        <input value={address} onChange={e => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log("Greeting was rendered at", new Date().toLocaleTimeString());
  return <h3>Hello{name && ', '}{name}!</h3>;
});

```

> 注意
> 你应该使用 memo 作为一个性能优化方式，如果你的代码离开它无法正常工作了，那你应该找到隐藏其后的问题并修复它，然后再添加 memo 来提高性能。

### 深度阅读: 是否应该到处使用 memo

如果你的应用和这个站点一样，大多数交互都很粗糙（例如替换页面或整个部分），则通常不需要记忆。另一方面，如果你的应用更像是一个绘图编辑器，并且大多数交互都是细粒度的（比如移动形状），那么你可能会发现缓存是很有用的

只有当你的组件经常使用完全相同的 props 重新渲染并且其重新渲染逻辑代价高昂时，使用 memo 优化才有价值。如果你的组件重新渲染时没有明显的延迟，则不需要 memo。记住，如果传递给组件的 props 总是不同的，例如传递了一个对象或渲染期间定义的函数，那么 memo 完全没用。这就是为什么你需要将 useMemo、useCallback 和 memo 一起使用。

在其他情况下，将组件包装到 memo 中没有任何好处，当然这样做也没什么大不了的。所以一些团队不考虑特殊情况，尽可能的使用 memo。这种方式的缺点在于代码的可读性变差，另外，不是所有的缓存都是有效的：一个 “永远是新的” 的值就足以破坏整个组件的缓存

实践中你可以通过遵循一些原则来避免大量缓存:

1. 当一个组件在视觉上包含了其他的组件，让它接受 JSX 作为子组件。这种方式当组件更新其 state 时，React 知道它的子组件不需要重复渲染
2. 更推荐使用局部 state，非必要不要提升 state。举个例子，不要将表单、悬停状态这样的瞬时状态保存到树的顶部或全局 state 中
3. 保证渲染逻辑为纯函数
4. 避免非必要的更新 state 的 Effect
5. 移除 Effects 非必要的依赖项

TODO: TRANSLATE 下面的内容和 useMemo 很像