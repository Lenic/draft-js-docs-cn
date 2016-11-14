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

这个标识符就是使用这个 Entity 实例对象时的指代，加到编辑的文本内容中去。比如使用 `Modifier` 模块包含的 `applyEntity` 方法：

```js
const key = Entity.create('LINK', 'MUTABLE', {href: 'http://lenic.coding.me'});
const contentStateWithLink = Modifier.applyEntity(
  contentState,
  targetRange,
  key
);
```

For a given range of text, then, you can extract its associated entity key by using
the `getEntityAt()` method on a `ContentBlock` object, passing in the target
offset value.

```js
const blockWithLinkAtBeginning = contentState.getBlockForKey('...');
const linkKey = blockWithLinkAtBeginning.getEntityAt(0);
const linkInstance = Entity.get(linkKey);
const {href} = linkInstance.getData();
```

## "Mutability"

Entities may have one of three "mutability" values. The difference between them
is the way they behave when the user makes edits to them.

Note that `DraftEntityInstance` objects are always immutable Records, and this
property is meant only to indicate how the annotated text may be "mutated" within
the editor. _(Future changes may rename this property to ward off potential
confusion around naming.)_

### Immutable

This text cannot be altered without removing the entity annotation
from the text. Entities with this mutability type are effectively atomic.

For instance, in a Facebook input, add a mention for a Page (i.e. Barack Obama).
Then, either add a character within the mentioned text, or try to delete a character.
Note that when adding characters, the entity is removed, and when deleting character,
the entire entity is removed.

This mutability value is useful in cases where the text absolutely must match
its relevant metadata, and may not be altered.

### Mutable

This text may be altered freely. For instance, link text is
generally intended to be "mutable" since the href and linkified text are not
tightly coupled.

### Segmented

Entities that are "segmented" are tightly coupled to their text in much the
same way as "immutable" entities, but allow customization via deletion.

For instance, in a Facebook input, add a mention for a friend. Then, add a
character to the text. Note that the entity is removed from the entire string,
since your mentioned friend may not have their name altered in your text.

Next, try deleting a character or word within the mention. Note that only the
section of the mention that you have deleted is removed. In this way, we can
allow short names for mentions.

## Modifying Entities

Since `DraftEntityInstance` records are immutable, you may not update the `data`
property on an instance directly.

Instead, two `Entity` methods are available to modify entities: `mergeData` and
`replaceData`. The former allows updating data by passing in an object to merge,
while the latter completely swaps in the new data object.

## Using Entities for Rich Content

The next article in this section covers the usage of decorator objects, which
can be used to retrieve entities for rendering purposes.

The [link editor example](https://github.com/facebook/draft-js/tree/master/examples/link)
provides a working example of entity creation and decoration in use.
