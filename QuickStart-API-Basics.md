---
id: quickstart-api-basics
title: API Basics
layout: docs
category: Quick Start
next: quickstart-rich-styling
permalink: docs/quickstart-api-basics.html
---

这篇文档是 `Draft` API 的概览。一个有效的[工作示例](https://github.com/facebook/draft-js/tree/master/examples/plaintext)可以参考。

## 受控的输入

`Editor` 这个 React 组件构建在受控的、可编辑内容的组件之上，并且提供了一个类似 React **受控输入**的顶层 API。

作为一个简单的回顾，受控输入包含两个关键部分：

1. 一个表示输入状态的 *value*。
1. 一个接受输入更新的回调 *onChange* 函数。

这种做法使得组件具有严格控制输入的状态，也可以在更新 DOM 前可以获得用于已经写入的文本信息。

```js
class MyInput extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};
    this.onChange = (evt) => this.setState({value: evt.target.value});
  }
  render() {
    return <input value={this.state.value} onChange={this.onChange} />;
  }
}
```

顶层组件可以通过 `value` 属性值控制 input 组件的输入。

## 控制富文本

在富文本的场景下，使用 React 有两个明显的问题：

1. 一个简单字符串不足以表示复杂的富文本状态。
1. 包含 ContentEditable 属性的元素，不存在 `onChange` 事件。

因此，状态可由单一的一个不可变的 [EditorState](/draft-js/docs/api-reference-editor-state.html) 对象表示，并且通过 `Editor` 核心代码实现一个 `onChange` 事件给顶层组件。这个 `EditorState` 对象是编辑器状态的完整快照，包含内容、光标，还有撤销重做的历史。任何内容和选择变更都会创建新的 `EditorState` 对象。注意使用不可变类型对象在数据持久化期间仍然是高效的。

```js
import {Editor, EditorState} from 'draft-js';

class MyEditor extends React.Component {
  constructor(props) {
    super(props);
    this.state = {editorState: EditorState.createEmpty()};
    this.onChange = (editorState) => this.setState({editorState});
  }
  render() {
    return <Editor editorState={this.state.editorState} onChange={this.onChange} />;
  }
}
```

编辑器中任何内容和选择的变更，`onChange` 事件都会被触发，参数就是包含这些更改的最新的 `EditorState` 对象。
