---
id: getting-started
title: Overview
layout: docs
category: Quick Start
next: quickstart-api-basics
permalink: docs/overview.html
---

Draft.js 是一个提供了一个跨浏览器的抽象、使用 React 构建富文本编辑器的框架，由不可变模型驱动。可以轻松构建任何类型的富文本输入，无论你仅想支持一些内联样式，还是构建一个复杂的文本编辑器。2016 年 2 月，发布在 [React.js Conf](http://conf.reactjs.com/) 上。

<iframe width="650" height="365" src="https://www.youtube.com/embed/feUYwoLhE_4" frameborder="0" allowfullscreen></iframe>

### 安装

当前版本的 Draft.js 通过 NPM 发布，依赖于 React 和 React Dom 这两个库。

```sh
npm install --save draft-js react react-dom
```

### 用法

```js
import React from 'react';
import ReactDOM from 'react-dom';
import {Editor, EditorState} from 'draft-js';

class MyEditor extends React.Component {
  constructor(props) {
    super(props);
    this.state = {editorState: EditorState.createEmpty()};
    this.onChange = (editorState) => this.setState({editorState});
  }
  render() {
    const {editorState} = this.state;
    return <Editor editorState={editorState} onChange={this.onChange} />;
  }
}

ReactDOM.render(
  <MyEditor />,
  document.getElementById('container')
);
```

由于 Draft.js 支持 Unicode 编码格式，所以必须添加如下元数据到 HTML 的源文件 `head` 元素中：

```html
<meta charset="utf-8" />
```

接下来，让我们进入基础 API 的学习，理解利用 Draft.js 可以干什么。
