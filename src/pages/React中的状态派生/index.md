---
title: React中的状态派生
date: 2020-05-16
spoiler: 详解如何正确在React中进行状态派生
---

在详解如何在React中进行状态派生之前，建议大家先看一下官方的这篇博客：[You Probably Don't Need Derived State](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)

## 何为状态派生

状态派生主要发生在类组件中，因为组件状态的初始值来自props，当props更新时我们就需要正确处理props的变更，不然页面状态可能就不会发生变化或者产生一些我们意想不到的效果

下面我们就从这段代码开始，详解如何正确处理React中的状态派生：

**PS:** 这是一个刻意写的状态派生例子，WordsList组件其实根本不需要从props中进行state初始化

```jsx
import React, { Component, createRef, Component } from "react";

const inputEl = createRef();

export default class App extends Component {
  state = {
    wordsList: [],
  };

  handleWordAdd = () => {
    const value = inputEl.current.value;
    if (value) {
      this.state.wordsList.push(value);
      this.setState({
        wordsList: this.state.wordsList,
      });
    }
  };

  render() {
    const { wordsList } = this.state;
    return (
      <div>
        <h3>React App</h3>
        <input type="text" ref={inputEl} />
        <button onClick={this.handleWordAdd}>Add</button>
        <WordsList wordsList={wordsList} />
      </div>
    );
  }
}

class WordsList extends Component {
  state = {
    wordsList: this.props.wordsList || [],
  };

  render() {
    const { wordsList } = this.state;
    return (
      <>
        <h3>Words Count: {wordsList.length}</h3>
        <ul>
          {wordsList.map((word, index) => (
            <li key={index}>{word}</li>
          ))}
        </ul>
      </>
    );
  }
}
```

代码很简单，主要流程就是当我们点击add按钮，就会把input输入框中的内容添加到wordsList中，然后更新WordsList组件，并将wordsList数组中所有内容显示出来

代码运行之后，效果符合预期，但是其实是有几个问题的，我们可以把WordsList组件继承自PureComponent试一下：

```jsx
class WordsList extends PureComponent {
  state = {
    wordsList: this.props.wordsList || [],
  };

  render() {
    const { wordsList } = this.state;
    return (
      <>
        <h3>Words Count: {wordsList.length}</h3>
        <ul>
          {wordsList.map((word, index) => (
            <li key={index}>{word}</li>
          ))}
        </ul>
      </>
    );
  }
}
```

再次运行代码，发现没有任何效果。其实这里主要原因是PureComponent在props更新时做了一次浅比较来确认是否要重新render，我们可以看一下父组件是如何更新wordsList的：

```jsx
handleWordAdd = () => {
  const value = inputEl.current.value;
  if (value) {
    this.state.wordsList.push(value);
    this.setState({
      wordsList: this.state.wordsList,
    });
  }
}
```

这里用了push方法，然后把push之后的数组重新赋值给了state中的wordsList，这里就是问题所在，**因为wordsList是引用类型的值，它存储的只是数组的引用地址，在setState时其实并没有改变引用地址，所以当WordsList组件进行prpos的浅比较时发现props没有变化，所以没有进行render，页面也就没有任何效果**

那么如何改变引用地址，很简单，我们可以使用concat返回一个新数组或者使用扩展运算符，这里我们试一下扩展运算符：

```jsx
handleWordAdd = () => {
  const value = inputEl.current.value;
  if (value) {
    this.setState({
      wordsList: [...this.state.wordsList, value],
    });
  }
}
```

再次运行，发现还是没有任何效果，其实这里props已经发生变化了，只是由于组件的状态是派生自props，所以我们这里需要正确处理props的变更

## 正确处理状态派生

还是以上面的代码为例，由于props发生变化，我们需要正确处理props的变更，常见的做法是利用componentWillReceiveProps这个声明周期方法，这里我们只需要在props发生变化时重置一下state即可

### componentWillReceiveProps

```jsx
class WordsList extends PureComponent {
  state = {
    wordsList: this.props.wordsList || [],
  };

  componentWillReceiveProps(nextProps) {
    this.setState({
      wordsList: nextProps.wordsList,
    });
  }

  render() {
    const { wordsList } = this.state;
    return (
      <>
        <h3>Words Count: {wordsList.length}</h3>
        <ul>
          {wordsList.map((word, index) => (
            <li key={index}>{word}</li>
          ))}
        </ul>
      </>
    );
  }
}
```

再次运行，效果符合预期，但是这里存在一个bug，如果父组件重新render，即使props没有发生变化，state也会被重置，这里我们改下一下代码：

```jsx
import React, { Component, createRef, PureComponent } from "react";

const inputEl = createRef();

export default class App extends Component {
  state = {
    wordsList: [],
  };

  handleWordAdd = () => {
    const value = inputEl.current.value;
    if (value) {
      this.setState({
        wordsList: [...this.state.wordsList, value],
      });
    }
  };

  handleRender = () => {
    this.setState({});
  };

  render() {
    const { wordsList } = this.state;
    return (
      <div>
        <h3>React App</h3>
        <button onClick={this.handleRender}>render</button>
        <hr />
        <input type="text" ref={inputEl} />
        <button onClick={this.handleWordAdd}>Add</button>
        <WordsList wordsList={wordsList} />
      </div>
    );
  }
}

const wordsListInputEl = createRef();

class WordsList extends PureComponent {
  state = {
    wordsList: this.props.wordsList || [],
  };

  componentWillReceiveProps(nextProps) {
    this.setState({
      wordsList: nextProps.wordsList,
    });
  }

  handleAdd = () => {
    const value = wordsListInputEl.current.value;
    if (value) {
      this.setState({
        wordsList: [...this.state.wordsList, value],
      });
    }
  };

  render() {
    const { wordsList } = this.state;
    return (
      <>
        <h3>Words Count: {wordsList.length}</h3>
        <ul>
          {wordsList.map((word, index) => (
            <li key={index}>{word}</li>
          ))}
        </ul>
        <input type="text" ref={wordsListInputEl} />
        <button onClick={this.handleAdd}>add</button>
      </>
    );
  }
}
```

跟之前不同的是，我们在父组件加了一个render按钮，用来重新渲染父组件，同时在WordsList组件中加了一个add按钮，逻辑同父组件，当我们点击wordsList组件中的按钮添加一个word时，在点击父组件的render，会发现状态被重置了，这里其实不应该重置的，因为props中的wordsList其实并没有发生变化，这也是我们进行状态派生时经常忽略的一个点，就是我们要判断一下props是否真的发生了更改：

```jsx
componentWillReceiveProps(nextProps) {
  if (nextProps.wordsList !== this.props.wordsList) {
    this.setState({
      wordsList: nextProps.wordsList,
    });
  }
}
```

这样就一切正常了，但是componentWillReceiveProps这个方法已经被React标记为不安全的声明周期方法，我们可以使用静态方法getDerivedStateFromProps来替代

### getDerivedStateFromProps

此方法与componentWillReceiveProps有一个很重要的不同点是它在组件Mounting阶段时也会触发，而componentWillReceiveProps只会在组件Updating时触发

与componentWillReceiveProps处理逻辑类似，我们要判断一下props是否真的发生了更改，由于getDerivedStateFromProps比较特殊，我们不能获取到上一次的props，所以这里我们需要手动存储一下：

```jsx
  static getDerivedStateFromProps(props, state) {
    if (props.wordsList !== state.prePropsWordsList) {
      return {
        prePropsWordsList: props.wordsList,
        wordsList: props.wordsList,
      };
    } else {
      return null;
    }
  }
```

## 总结

至此，关于React的状态派生基本就讲完了，总的来说如果无法避免使用状态派生，我们就需要非常小心的使用componentWillReceiveProps或者getDerivedStateFromProps，由于前者在新版本中可能会被废弃，所以这里推荐后者
