## 模式

React 的启动模式

- **legacy 模式：** `ReactDOM.render(<App />, rootNode)`。这是当前 React app 使用的方式。当前没有计划删除本模式，但是这个模式可能不支持这些新功能。
- **blocking 模式：** `ReactDOM.createBlockingRoot(rootNode).render(<App />)`。目前正在实验中。作为迁移到 concurrent 模式的第一个步骤。
- **concurrent 模式：** `ReactDOM.createRoot(rootNode).render(<App />)`。目前在实验中，未来稳定之后，打算作为 React 的默认开发模式。这个模式开启了*所有的*新功能。

[react官方解释](https://zh-hans.reactjs.org/docs/concurrent-mode-adoption.html#migration-step-blocking-mode)



1、setState只在合成事件和钩子函数中是“异步”的，在原生事件和 `setTimeout`、`Promise.resolve().then` 中都是同步的。

- a. [react合成事件](./合成事件)，如onClick等
- b. 钩子函数包括componentDidMount、useEffect等等

2、“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形式了所谓的“异步”。

3、批量更新优化也是建立在“异步”（合成事件、钩子函数）之上的，在原生事件和`setTimeout`、`Promise.resolve().then` 中不会批量更新，在“异步”中如果对同一个值进行多次修改，批量更新策略会对其进行覆盖，取最后一次的执行,类似于Object.assin的机制，如果是同时修改多个不同的变量的值，比如改变了a的值又改变了b的值，在更新时会对其进行合并批量更新，结果只会产生一次render。

4、在合成事件和钩子函数中的`setTimeout`、`Promise.resolve().then`，会在“异步”更新state执行并且渲染完组件之后



legacy 模式，在事件函数里，setState更新是批量的

```js
constructor() {
    super();
    this.state = {
      user: 1
    }
  }

  handleClick(){
    this.setState({
      user: this.state.user + 1
    })
    console.log(this.state.user, '1')
    this.setState({
      user: this.state.user + 1
    })
    console.log(this.state.user, '2')
    setTimeout(() => {
      console.log(this.state.user, '3')
      this.setState({
        user: this.state.user + 1
      })
      console.log(this.state.user, '4')
      this.setState({
        user: this.state.user + 1
      })
      console.log(this.state.user, '5')
    })
  }

// 打印出来的结果
1 '1'
1 '2'
2 '3'
3 '4'
4 '5'

```





