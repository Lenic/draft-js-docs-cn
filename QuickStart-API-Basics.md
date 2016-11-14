---
id: quickstart-api-basics
title: API Basics
layout: docs
category: Quick Start
next: quickstart-rich-styling
permalink: docs/quickstart-api-basics.html
---

这篇文档是 `Draft` API 的概览。一个有效的[工作示例](https://github.com/facebook/draft-js/tree/master/examples/plaintext)可以参考。

## 受控制的输入

`Editor` 这个 React 组件构建在受控的 ContentEditable 组件之上，并且提供了一个类似 React **受控输入**的顶层 API。

作为一个简单的回顾，受控输入包含两个关键部分：

1. 一个 *value* 表示输入的状态。
1. 通过 props 传递一个 *onChange* 回调函数接收输入的更新。

这种做法允许组件具有严格控制输入的状态，同时允许更新 DOM 元素提供用户已经写入的文本信息。

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

顶层组件可以通过 `value` 值控制 input 组件的输入。

## 受控的富文本输入

使用 React，在富文本的场景下，有两个明显的问题：

1. 一个简单字符串不足以表示复杂的富文本输入。
1. 包含 ContentEditable 属性的元素，不存在 `onChange` 事件。

因此，输入状态可被表示为一个不可变的 [EditorState](/draft-js/docs/api-reference-editor-state.html)  对象，并且提供一个 `onChange` 事件给顶层组件。

`EditorState` 对象是编辑器状态的完整快照，包含内容、光标，还有撤销重做的历史。所有的内容和选择变更都会创建新的 `EditorState` 对象。注意在通过不可变数据渲染期间会有所影响。

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

对于编辑器 DOM 的任何改变，`onChange` 事件都会被调用，附加包含这些更改的最新的 `EditorState` 对象。
