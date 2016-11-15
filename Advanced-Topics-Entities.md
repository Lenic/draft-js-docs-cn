---
id: advanced-topics-entities
title: Entities
layout: docs
category: Advanced Topics
next: advanced-topics-decorators
permalink: docs/advanced-topics-entities.html
---

这篇文章我们讨论 Entity 系统，这被用于描述带有元数据的文本段。Entity 可以使得工程师做出充满现代风格的编辑器。链接、提及和嵌入的内容都可以用 Entity 实现。

在 Draft.js 源码中，[链接编辑器](https://github.com/facebook/draft-js/tree/master/examples/link)和 [Entity 示例](https://github.com/facebook/draft-js/tree/master/examples/entity)提供了实时的代码示例，来阐明如何使用 Entity 以及他们的内置行为。

[Entity API 参考](/draft-js/docs/api-reference-entity.html) 提供了创建、检索和更新 Entity 对象静态方法的详细信息。

## 介绍

Entity 是 Draft.js 中表示一段文本元数据的对象。有三个属性：

- **type**: 用于指示 Entity 类型的字符串，比如：`'LINK'`、`'MENTION'`、`'PHOTO'`。
- **mutability**: 不同于 `immutable-js`，这个属性表示使用此 Entity 对象在用户编辑内容时的行为，可以在下面获得更详细的信息。
- **data**: 一个可选的扩展元数据对象（key/value）。比如说，`LINK` 类型的 Entity 对象可能包含一个包含了 `href` 键值对的 `data` 对象。

所有的 Entity 存储在 `Entity` 模块的单个对象中，并通过唯一标识符的方式被 `ContentState` 引用，React 组件通过 Decorate 使用这个对象。*我们正考虑接下来将其存储到 `EditorState` 或 `ContentState` 中。*

通过 [decorators](/draft-js/docs/advanced-topics-decorators.html) 或者
[自定义块组件](/draft-js/docs/advanced-topics-block-components.html)，你可以基于 Entity 元数据向编辑器中添加丰富的呈现效果。

## 创建或检索 Entity

Entity 实例应该使用 `Entity.create` 方法创建，接受上面介绍的三个属性作为参数。方法的返回值是一个字符串，也就是唯一标识符。

这个标识符就是当前 Entity 实例对象时的指代，可以加到编辑的文本内容中去。比如使用 `Modifier` 模块包含的 `applyEntity` 方法：

```js
const key = Entity.create('LINK', 'MUTABLE', {href: 'http://lenic.coding.me'});
const contentStateWithLink = Modifier.applyEntity(
  contentState,
  targetRange,
  key
);
```

对于一段给定的文本，使用目标的偏移量，可以通过`ContentBlock` 对象的 `getEntityAt()` 获取关联的 Entity 唯一标识符。

```js
const blockWithLinkAtBeginning = contentState.getBlockForKey('...');
const linkKey = blockWithLinkAtBeginning.getEntityAt(0);
const linkInstance = Entity.get(linkKey);
const {href} = linkInstance.getData();
```

## 可变性

Entity 有三个可变性的值，区别在于用户编辑时的不同表现方式。

注意 `DraftEntityInstance` 对象总是不可变的，并且这个属性仅仅指示编辑器对于这段文本的可修改性。*考虑将来对这个属性重命名，以避免可能的命名混乱。*

### Immutable -- 不可变的

在不删除 Entity 的情况下，此种类型的 Entity 不能更改。具有这种可变性的 Entity 实体实际上是原子的。

> 译者注：原子性说的就是不可改变。原子性的事物，只有两种操作：创建和删除。

例如，在 Facebook 输入框中，添加一个页面的提及（这里可以理解为微博的 # 话题功能），然后添加或移除一个字符。注意，在添加字符时，Entity（这个提及）被删除，更改成普通文本；在移除字符时，Entity 整体被移除。

这种可变性在文本必须绝对匹配元数据的情况下非常有用，因为其不可改变。

### Mutable -- 可变的

这种类型的文本可以随意更改。比如，超链接的文本通常是可以更改的，这是因为超链接的文本与其目标网站并没有耦合在一起。

### Segmented -- 特定的文本

Segmented 类型的文本和 Immutable 类型的文本表现大体相同，除了可以通过删除来定制之外。

例如，在 Facebook 的输入框中，添加一个朋友的提及（这里可以理解为微博的 @ 某个人），然后添加一个字符。请注意，Entity 将会从整个字符串中移除，因为你提到的朋友没有在文本中更改自己的名字。

接下来，尝试删除提及中的字符或单词。注意仅仅只是删除了你删除的部分字符。这样，我们可以允许短名称提及，

## 修改 Entity

因为 `DraftEntityInstance` 是不可变的，所以你不能直接更新对象的 `data` 属性。

相反，Entity 有两个实例方法用于更新操作：`mergeData` 和 `replaceData`。前者可以通过传入要合并的数据来更新 Entity 实例，后者可以使用新的对象完全替换原对象。

## 将 Entity 用于富文本

本节的下一篇文章将介绍 Decorator 的用法，一般被用来检索 Entity 渲染为 DOM 元素。

可以查看这个[例子](https://github.com/facebook/draft-js/tree/master/examples/link)来了解 Entity 和 Decorator 是如何工作的。
