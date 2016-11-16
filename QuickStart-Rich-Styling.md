---
id: quickstart-rich-styling
title: Rich Styling
layout: docs
category: Quick Start
next: advanced-topics-entities
permalink: docs/quickstart-rich-styling.html
---

现在，我们已经确定了顶层的 API 的基础，接下来就可以考虑如何将基础的富文本样式添加到编辑器中了。可以在[这里](https://github.com/facebook/draft-js/tree/master/examples/rich)查看我们已经完成的例子。

## EditorState: 你的命令

前一篇文章介绍了 `EditorState` 对象是编辑器状态的完整快照，通过 `Editor` 核心代码实现了 `onChange` 事件回调的属性。因为顶层的 React 组件负责维护整体状态，所以可以修改 `EditorState` 符合期望值。

比如，对于内联的和块样式行为，可以使用 [`RichUtils`](/draft-js/docs/api-reference-rich-utils.html) 模块。该模块提供了大量的有用的函数帮助你更新状态信息。

类似的，[Modifier](/draft-js/docs/api-reference-modifier.html) 模块也提供了大量的通用操作帮助你应用编辑，包括对文本和样式的修改。这个模块是一套更简单、更细化，返回更新后 `EditorState` 对象的编辑函数。对于这个例子，我们仍旧使用 `RichUtils` 演示如何在顶层组件中应用基础富文本样式。

## RichUtils 和按键命令

`RichUtils` 有关于可用的核心按键命令如何应用在 Web 编辑器上的信息，比如 Cmd+B (bold)、Cmd+I (italic) 等。

我们可以通过 props 的 `handleKeyCommand` 事件来观察和处理按键命令，将这些应用到 `RichUtils` 以应用或删除目标样式。

```js
import {Editor, EditorState, RichUtils} from 'draft-js';

class MyEditor extends React.Component {
  constructor(props) {
    super(props);
    this.state = {editorState: EditorState.createEmpty()};
    this.onChange = (editorState) => this.setState({editorState});
    this.handleKeyCommand = this.handleKeyCommand.bind(this);
  }
  handleKeyCommand(command) {
    const newState = RichUtils.handleKeyCommand(this.state.editorState, command);
    if (newState) {
      this.onChange(newState);
      return 'handled';
    }
    return 'not-handled';
  }
  render() {
    return (
      <Editor
        editorState={this.state.editorState}
        handleKeyCommand={this.handleKeyCommand}
        onChange={this.onChange}
      />
    );
  }
}
```

> handleKeyCommand
>
> 提供给 `handleKeyCommand` 的 `command` 参数是一个字符串，也就是将要执行的命令的名称，这是从 DOM 的按键事件映射的。可以参考[高级 - 按键绑定](/draft-js/docs/advanced-topics-key-bindings.html)更多信息。

## UI 中的样式控制

在 React 组件中，你可以添加按钮或者其他控件以允许用户在编辑器中修改样式。在上面的例子中，我们使用预定义的按键命令来达成这个目标。但是，我们可以添加更加复杂的 UI 样式来丰富这些特征。

这里有一个相当简单的例子，用一个【粗体】按钮来切换加粗状态。

```js
class MyEditor extends React.Component {
  // …

  _onBoldClick() {
    this.onChange(RichUtils.toggleInlineStyle(this.state.editorState, 'BOLD'));
  }

  render() {
    return (
      <div>
        <button onClick={this._onBoldClick.bind(this)}>Bold</button>
        <Editor
          editorState={this.state.editorState}
          handleKeyCommand={this.handleKeyCommand}
          onChange={this.onChange}
        />
      </div>
    );
  }
}
```
