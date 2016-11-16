---
id: advanced-topics-decorators
title: Decorators
layout: docs
category: Advanced Topics
next: advanced-topics-key-bindings
permalink: docs/advanced-topics-decorators.html
---

内联或块样式不是富文本的唯一选择，比如 Facebook 的评论输入提供了 Tag 和提及的背景高亮功能。

为了支持灵活的富文本样式，Draft.js 提供了 Decorator 系统。[Tweet](https://github.com/facebook/draft-js/tree/master/examples/tweet)提供了一个实时的例子。

## CompositeDecorator -- 组合装饰器

Decorator 这个概念的来源是扫描给定的 [ContentBlock](/draft-js/docs/api-reference-content-block.html) 内容，以匹配定义策略的文本范围，然后使用关联的 React 组件渲染。

可以使用 `CompositeDecorator` 定制你想要的 Decorator 行为。这个类也允许你提供多个 `DraftDecorator` 对象，这将逐个执行每个策略的匹配。

所有的 Decorator 都存储在 `EditorState` 中。在创建新的 `EditorState` 对象时，比如通过 `EditorState.createEmpty()` 方法创建时，可以额外添加 Decorator。

> Under the hood
>
> 编辑器中的文本修改时，新生成的 `EditorState` 对象使用这些 Decorator 扫描新的 `ContentState`，并且标识出作用的范围。此时形成一个完整的块、Decorator 和内联样式的树，作为接下来渲染输出的基础。
>
> 这样，就可以保证内容改变时，通过 Decorator 渲染的最终结果总是与 `EditorState` 是一致的。

```js
const compositeDecorator = new CompositeDecorator([
  {
    strategy: handleStrategy,
    component: HandleSpan,
  },
  {
    strategy: hashtagStrategy,
    component: HashtagSpan,
  },
]);
```

在给定的文本块中，按照定义的顺序从上到下执行。这个 `compositeDecorator` 会先匹配 @ 模式，再匹配 hashtag 模式。

```js
// Note: these aren't very good regexes, don't use them!
const HANDLE_REGEX = /\@[\w]+/g;
const HASHTAG_REGEX = /\#[\w\u0590-\u05ff]+/g;

function handleStrategy(contentBlock, callback) {
  findWithRegex(HANDLE_REGEX, contentBlock, callback);
}

function hashtagStrategy(contentBlock, callback) {
  findWithRegex(HASHTAG_REGEX, contentBlock, callback);
}

function findWithRegex(regex, contentBlock, callback) {
  const text = contentBlock.getText();
  let matchArr, start;
  while ((matchArr = regex.exec(text)) !== null) {
    start = matchArr.index;
    callback(start, start + matchArr[0].length);
  }
}
```

这些策略的函数，会在 `start` 和 `end` 限定的文本段内被调用。

## Decorator 组件

对于这些被修饰的文本范围，必须定义一个 React 组件渲染输出，这些往往是添加了 CSS 样式的简单的 `span` 元素。

在我们当前的例子中，上面这个 `CompositeDecorator` 对象将名称为 `HandleSpan` 和 `HashtagSpan` 的组件用于 Decorator。它们仅仅只是无状态的组件：

```js
const HandleSpan = (props) => {
  return <span {...props} style={styles.handle}>{props.children}</span>;
};

const HashtagSpan = (props) => {
  return <span {...props} style={styles.hashtag}>{props.children}</span>;
};
```

注意 `props.children` 被传递到渲染输出，这样做能保证输出的文本在 `span` 元素内呈现。

你可以在超链接中发现相同的用法，比如我们的[超链接示例](https://github.com/facebook/draft-js/tree/master/examples/link)。

### 超越 CompositeDecorator

提供给 `EditorState` 元素的 Decorator 对象，只需要匹配 [DraftDecoratorType](https://github.com/facebook/draft-js/blob/master/src/model/decorators/DraftDecoratorType.js) 的期望值。这意味着可以按照自己的想法创建 Decorator，只要与预期的类型相符 ———— 不用被 `CompositeDecorator` 束缚住。

## 设置新的 Decorator

此外，在正常的状态传播期间，可以通过创建不可变对象的方式，在 `EditorState` 上设置一个新的 Decorator 对象。

这意味着在应用程序的工作流程中，如果装饰器无效或者需要修改，就可以创建一个新的 Decorator 对象（使用 `null` 或者移除所有的 Decorator），并且使用 `EditorState.set()` 来启动新的 Decorator 设置。

例如，由于某种原因，我们需要禁用到 @ 句柄的 Decorator 处理，就可以使用如下代码：

```js
function turnOffHandleDecorations(editorState) {
  const onlyHashtags = new CompositeDecorator([{
    strategy: hashtagStrategy,
    component: HashtagSpan,
  }]);
  return EditorState.set(editorState, {decorator: onlyHashtags});
}
```

这个 `editorState` 的 `ContentState` 将会被重新计算，而 @ 句柄的 Decorator 处理，不会出现在下一个渲染过程中。

再次声明，由于跨越不可变对象的数据持久性，这保持记忆效率。

> Again, this remains memory-efficient due to data persistence across immutable objects.
