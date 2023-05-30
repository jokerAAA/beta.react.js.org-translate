
useCallback 是一个 React Hook，允许你在重复渲染时缓存函数定义。

```javascript
const cachedFn = useCallback(fn, dependencies)
```

## 参考

### useCallback(fn, dependencies)

在组件的顶层调用 useCallback 可以在组件重新渲染时缓存一个函数定义:

```javascript
import { useCallback } from 'react';

export default function ProductPage({ productId, referrer, thtme}) {
  const handleSubmit = useCallback(orderDetails => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); 
}
```

### 参数

- fn: 你想缓存的函数值，它可以接受任意的参数并返回任意值，React 会在初次渲染时将其返回给你，在下一次渲染时，如果 dependencies 没有变 React 会再次给你同一个函数，否则它会给你在当前渲染中传递的函数，并将其存下来一边下次复用。React 不会调用这个函数，由你来决定什么时候调用以及是否调用
- dependencies： fn 内部代码中的响应式值的列表，响应式的值包括 props、state 及组架内部声明的变量和函数。如果你为 React 配置了 linter，它会检查是否正确的为配置响应式值添加了依赖项。依赖项数组的长度必须是已知的，并用行内的方式书写。React 会用 Object.is 来对比每一项值的变化

### 返回值

首次渲染时，useCallback 返回你传递的 fn 函数

在后续渲染期间，它将返回上次渲染中已存储的 fn 函数（如果依赖项没有变化），或者返回你在此渲染期间传递的 fn 函数

### 注意事项

- useCallback 是一个 Hook，所以你只能在组件顶层或自定义 Hook 内调用，并且不能在循环和条件分支中调用。如果有需要的话，提取一个新组建并将 state 移入其中
- 除非有特定原因，否则 React 不会丢弃缓存的函数。例如，在开发模式下，当你编辑组件的文件时 React 会丢弃缓存。在开发模式和生产模式下，React 如果你的组件在首次渲染时挂起，React 会丢弃缓存。将来，React 可能会添加更多利用丢弃缓存的特性 —— 比如，如果 React 未来添加对虚拟列表的内置支持，则丢弃滚出虚拟列表视口外的元素的缓存是有意义的。如果你依靠 useCallback 作为一个性能优化，这应该是符合预期的。否则，用 state 或 ref 可能更加合适

## 用处

### 跳过组件的重复渲染

当你优化渲染性能时，有时你可能需要缓存传递给子组件的函数，首先来看下如何实现这一点，然后看看它在哪些情况下有用

要在组件重复渲染时缓存一个函数，你可以将其定义在 useCallback 内:

```javascript
import { useCallback } from 'react';

function ProductPage({ produectId, referrer, theme}) {
  const handleSubmit = useCallback(() => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer])
}
```

你需要给 useCallback 传递两个东西:

1. 你想在重复渲染时缓存的函数定义
2. 一个依赖项数组，包含组件内部的且在函数内使用的每个值

在首次渲染时，你从 useCallback 中得到的返回函数就是你传递的那个

在以后的渲染，React 会对比 dependencies 列表，如果没有发生变化，useCallback 会返回之前的函数，否则 useCallback 会返回在这次渲染时传递的函数

换句话说，useCallback 在重复渲染时缓存了一个函数知道它的依赖项数组变了

让我们通过一个例子来看看这在什么时候会很有用

假设你从 ProductPage 中向 ShippingForm 组件传递了一个 handleSubmit 函数:

```javascript
function ProductPage({ productId, referrer, theme }) {
  // ...
  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
```

你注意到切换 theme 属性会让应用卡顿，但是如果你从 JSX 中删除 ShippingForm 后感觉快多了，这告诉值得尝试优化 ShippingForm 组件

默认情况下，当一个组件重复渲染时，React 会递归的渲染它的所有子组件，这就是为什么当 ProductPage 使用不同的 theme 重新渲染时，ShippingForm 也会重新渲染。这对于不需要太多计算来重新渲染的组件说很好，但是当你确认重复渲染很慢时，你可以告诉 ShippingForm 在它的 props 和上次渲染相同时通过将其包装到 memo 中来跳过重复渲染

```javascript
import {memo} from 'react';
const ShippingForm = memo(function ShippingForm({ onSubmit })) {

}
```

通过此更改，如果 ShippingForm 的所有 props 都和上次渲染时一样它会跳过渲染，这就是缓存函数变得重要的时候！假设你定义了没有使用 useCallback 的submit:

```javascript
function ProductPage({ productId, referrer, theme }) {
  // Every time the theme changes, this will be a different function...
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }
  
  return (
    <div className={theme}>
      {/* ... so ShippingForm's props will never be the same, and it will re-render every time */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

在 js 中，一个 `function() {}` 或 `() => {}` 总是创建一个不同的函数，类似于 `{}` 对象字面量总是创建一个新对象。通常来说这不是问题，但这意味着 ShippingForm 的 props 永远都不会一样，所以你的 memo 优化不会生效，这就是 useCallback 派上用场的地方:

```javascript
function ProductPage({ productId, referrer, theme }) {
  // Tell React to cache your function between re-renders...
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ...so as long as these dependencies don't change...

  return (
    <div className={theme}>
      {/* ...ShippingForm will receive the same props and can skip re-rendering */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

通过将 handleSubmit 包装在 useCallback 中，你可以确保他们在重复渲染时是相同的函数（直到依赖关系发生变化）。除非出于某些特定的原因，否则不必将函数包装在 useCallback 中。在这个例子中，原因是你将其传递给了一个用 memo 包装的组件，这种方式可以让它跳过重新渲染。还有一些其他可能的场景，在下面会一一介绍

> 注意
> 你应该仅将 useCallback 作为一个性能优化手段，如果你的代码少了它无法正常工作，那么应该首先找到问题并修复它，然后再添加 useCallback 

#### 深度阅读: useCallback 和 useMemo 的关系

你会经常看到 useMemo 和 useCallback 一起出现，当你尝试优化子组件时它们都很有用。它们让你记住（或者换句话说，缓存）你传递的东西：

```javascript
import { useMemo, useCallback } from 'react';

function ProductPage({ productId, referrer }) {
  const product = useData('/product/' + productId);

  const requirements = useMemo(() => { // Calls your function and caches its result
    return computeRequirements(product);
  }, [product]);

  const handleSubmit = useCallback((orderDetails) => { // Caches your function itself
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm requirements={requirements} onSubmit={handleSubmit} />
    </div>
  );
}
```

区别在于他们让你缓存的是什么:

- useMemo: 缓存函数调用的结果。在这个例子中，它缓存了 `computeRequirements(product)` 调用的结果，因此除非 `product` 变了否则它不会变。这允许你可以向下传递 `requirements` 对象而不会造成 ShippingForm 不必要的重复渲染。必要时，React 会再次调用你在渲染过程中传递的函数来计算结果。
- useCallback 缓存的是函数本身。和 useMemo 不同，它不会调用你提供的函数，而是缓存你的函数本身，所以除非 productId 或 referrer 变了，handleSubmit 本身是不会变的。这使你可以向下传递 handleSubmit 函数，而不会造成 ShippingForm 无意义的重复渲染。在用户提交表单以前，你的代码不会运行。

如果你已经熟悉 useMemo 了，你可能会发现将 useCallback 想为这样会很有帮助:

```javascript
function useCallback(fn, dependencies) {
  return useMemo(() => fn, dependencies);
}
```

在 useMemo 中越多关于两者的区别

#### 是否应该导出使用 useCallback

如果你的应用和这个站点一样，大多数交互都很粗糙（例如替换页面或整个部分），则通常不需要记忆。另一方面，如果你的应用更像是一个绘图编辑器，并且大多数交互都是细粒度的（比如移动形状），那么你可能会发现记忆是很有用的

使用 useCallback 缓存函数仅在少数情况下有价值:

- 你将其作为 prop 传递给了一个用 memo 包装的组件。你希望值没变是跳过重复渲染，记忆化允许你的组件在依赖项未变时可以跳过渲染
- 你传递的函数随后用作某些 Hook 的依赖项，比如，另一个包装在 useCallback 的函数依赖它，或者你在 useEffect 中依赖了这个函数

在其他情况下，将函数包装在 useCallback 中没有任何好处，当然这样做也没什么明显的坏处，所有有些团队渲染不考虑个别案例，并尽可能的缓存函数，缺点在于代码可读性变得更差了，此外，并不是所有的缓存都是高效的：一个 “永远是新的” 的值就足以破坏整个组件的记忆化了

注意，useCallback 不会阻止创建函数，你总是在创建一个函数，但 React 会忽略它并在没有任何改变的情况下返回一个缓存的函数

在实践中，你可以通过遵循一些原则来避免大量的缓存:

1. 当一个组件市局上包含了其他组件，让他接受 JSX 作为子组件，然后如果包装组更新了自己的 state，React 知道它的子组件不需要重新渲染
2. 尽量使用局部 state，非必要不提升 state。不要保存像表单这样的瞬时状态，无论一个东西在树的顶部还是全局状态库中。
3. 保持渲染逻辑是纯函数，如果组件的重新渲染会导致一个问题会产生一些明显的视觉问题，那就是组件的 bug！修复 bug 而不是添加缓存
4. 避免不必要的更新 state 的 Effect，React 应用的多数性能问题都源于 Effect 的更新链，其造成了组件的反复渲染
5. 尝试从 Effect 中删除不必要的依赖项，举个例子，将某些函数或对象移入 Effect 中或将其移出组件会比缓存结果要容易的多。

如果某些特定的交互仍然感觉滞后，请使用 React 开发工具的分析器检查哪些组件从记忆化中获益最多，并在需要的地方添加记忆。这些原则让你的组件更容易调试和理解，因此在任何情况下都最好遵循它们，从长远来看，我们正在研究自动缓存来一劳永逸的解决这个问题

### 在缓存回调中更新 state

有时候你需要基于之前的缓存回调中的 state 来更新 state.

这个 handleAddTodo 将 todos 指定为一个依赖项，因为要基于它来计算出下一个 todo：

```javascript
function TodoList() {
  const [todos, setTodos] = useState([]);
  const handleAddTodo = useCallback(text => {
    const newTodo = {
      id: nextId++,
      text
    }
    setTodos([...todos, newTodo])
  }, [todos])
}
```

你希望记忆函数的依赖项尽可能的少，当你读取某些 state 只是为了计算出下一个 state 时，你可以从依赖项中删除它并传递一个更新函数:

```javascript
function TodoList() {
  const [todos, setTodos] = useState([]);
  const handleAddTodo = useCallback((text) => {
    const newTodo = {
      id: nextId++,
      text
    }
    setTodos(todos => [...todos, newTodo])
  })
}
```

这里你将如何更新 state 的逻辑传递给了 React，而不是将 todos 设置为依赖项并在内部读取它，在 useStaet 中阅读更多关于更新函数的信息

### 防止 Effect 运行频率过快

有时候你可能希望在 Effect 中调用一个函数:

```javascript
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function createOptions() {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    // ...
```

这种方式导致了一个问题，Effect 内每个响应式的值都必须被声明为 Effect 的依赖。然而，如果你声明 createOptions 作为一个依赖项，它会导致你的 Effect 不断重连到聊天室

```javascript
  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // 🔴 Problem: This dependency changes on every render
  // ...
```

要解决这个问题，你可以用 useCallback 包装 Effect 中调用的函数

```javascript
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const createOptions = useCallback(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }, [roomId]); // ✅ Only changes when roomId changes

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // ✅ Only changes when createOptions changes
  // ...
```

这种方式保证了如果 roomId 在重复渲染时是相同的，则 createOptions 函数也是相同的。然而，从依赖项中移除函数是个更好的解决办法，将函数移动到 Effect 内部。

```javascript
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() { // ✅ No need for useCallback or function dependencies!
      return {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Only changes when roomId changes
  // ...
```

现在这段代码简单多了，并且不需要 useCallback 了。

### 优化自定义 Hook

如果你在写自定义 Hook，建议将它返回的任何函数包装到 useCallback 中

```javascript
function useRouter() {
  const { dispatch } = useContext(RouterStateContext);
  const navigate = useCallback(url => {
    dispatch({
      type: 'navigate',
      url
    }, [dispatch])
  })
  const goBack = useCallback(() => {
    dispatch({
      type: 'back'
    }, [dispatch])
  })
  return {
    navigate,
    goBack
  }
}
```

这保证了 Hook 的使用者可以根据需要来优化自己的代码

## 常见问题

### 每次组件渲染，useCallback 都返回一个不同的函数

确保你已将依赖数组指定为第二个参数

如果你忘记了依赖数组，useCallback 会在每次都返回一个新函数

```javascript
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }); // 🔴 Returns a new function every time: no dependency array
  // ...
```

下面是个正确的例子:

```javascript
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ✅ Does not return a new function unnecessarily
  // ...
```

如果这种改动没用，那么问题在于依赖项中的某一个在渲染时变了，你可以通过手动打印的方式来排查问题:

```javascript
 const handleSubmit = useCallback((orderDetails) => {
    // ..
  }, [productId, referrer]);

  console.log([productId, referrer]);
```
你可以右键点击控制台打印的变量并选择另存为一个全局变量，假设第一个被保存为 temp1，第二个被保存为 temp2，然后你可以用浏览器控制台来检查这些变量是否同一个:

```javascript
Object.is(temp1[0], temp2[0]); // Is the first dependency the same between the arrays?
Object.is(temp1[1], temp2[1]); // Is the second dependency the same between the arrays?
Object.is(temp1[2], temp2[2]); // ... and so on for every dependency ...
```

当你发现打破记忆化的依赖项后，你可以移除它或将暂存起来

### 循环中不能调用 useCallback

TODO: TRANSLATE