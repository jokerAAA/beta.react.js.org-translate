
React 会改变你的对于设计和构建应用的看法。当你使用 React 构建用户界面时，你首先会将其分解为多个单元，我们成为组件；然后你为每个组件描述不同的视觉状态；最终你将组件组合到一起，让数据穿过它们。在本教程中，我们将引导你完成用 React 构建一个可搜索的表格的思考过程；

## 从视觉稿开始

假设你已经有了有个 JSON 的 API 接口和一个设计稿。  
JSON API 接口返回的数据如下:

```json
[
  { category: "Fruits", price: "$1", stocked: true, name: "Apple" },
  { category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit" },
  { category: "Fruits", price: "$2", stocked: false, name: "Passionfruit" },
  { category: "Vegetables", price: "$2", stocked: true, name: "Spinach" },
  { category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin" },
  { category: "Vegetables", price: "$1", stocked: true, name: "Peas" }
]
```

设计稿如下: [设计稿](https://react.dev/images/docs/s_thinking-in-react_ui.png)。  
在 React 中实现一个 UI需要遵循以下的五个步骤:

### 将 UI 分解为组件层次结构

在设计稿中圈出每一个组件和其子组件，并为其命名。如果你和设计师一起工作，他们可能在设计工作中已经完成了类似的工作。  
根据你的背景不同，你会用不同的方式将设计稿拆分为组件:  

- 代码: 和考虑是否创建函数或对象一样，遵循单一职责原则 —— 即一个组件始终应该做一件事，如果它的功能变复杂了，考虑拆分成为更小的组件；
- CSS: 考虑如何为其添加 CSS 选择器
- 设计: 考虑如何组织设计的层次

如果你的 JSON 的结构很棒，你会发现其实它就是你的 UI 的组件映射。这是因为 UI 和数据模型总是具有相同的信息结构 —— 即相同的形状。将你的 UI 拆分为组件，每个组件其实就是数据模型一部分的映射。  
在这个例子中有五个组件: [五个组件](https://react.dev/images/docs/s_thinking-in-react_ui_outline.png)

1. `FilterableProductTable`: 包含整个 app
2. `SearchBar`: 处理用户输入
3. `ProductTable`: 根据用户输入展示和过滤数据列表
4. `ProductCategoryRow`: 为每个分类展示一个标题
5. `ProductRow`: 为每个产品展示一行

如果你看了 `ProductTable`，你会发现