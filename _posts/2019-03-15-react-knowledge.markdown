---
layout:     post
title:      "React知识点整理"
subtitle:   " \"React知识点（更新中）\""
date:       2019-03-15 22:00:00
author:     "iwalking11"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - react
---


### 1.React组件间通讯（数据传递方式）
1. 父传子（props）
2. 子传父（回调函数-props,）
3. 兄弟组件（状态提升）
4. 嵌套层级深组件（context，provider，consumer）
5. 观察者模式（引入eventProxy模块）
6. redux（引入react-redux）
7. 父组件中对不确定的子组件进行props传递（组合模式传值）
```
// 使用 React.Children.map,React.cloneElement 
// React.Children.map，可以遍历 children 中所有的元素，因为 children 可能是一个数组嘛。
// Children.map() 类似于 Array.map() 方法。但请务必使用Children.map()，因为 children.props 具有不透明的数据结构，使得 Array.map() 方法不适合此用例。
const Parent = React.createClass({
  doSomething: function(value) {
    console.log('doSomething called by child with value:', value);
  },
  render: function() {
    const childrenWithProps = React.Children.map(this.props.children,
      (child) => React.cloneElement(child, { doSomething: this.doSomething }));
    return <div>{childrenWithProps}</div>
  }
});
```

### 2.React 中 keys 的作用是什么？

Keys 是 React 用于追踪哪些列表中元素被修改、被添加或者被移除的辅助标识。

```
// js
render () {
  return (
    <ul>
      {this.state.todoItems.map(({item, key}) => {
        return <li key={key}>{item}</li>
      })}
    </ul>
  )
}
```

在开发过程中，我们需要保证某个元素的 key 在其同级元素中具有唯一性。在 React Diff 算法中 React 会借助元素的 Key 值来判断该元素是新近创建的还是被移动而来的元素，从而减少不必要的元素重渲染。此外，React 还需要借助 Key 值来判断元素与本地状态的关联关系，因此我们绝不可忽视转换函数中 Key 的重要性。

### 3.调用 setState 之后发生了什么？

在代码中调用 setState 函数之后，React 会将传入的参数对象与组件当前的状态合并，然后触发所谓的调和过程（Reconciliation）。经过调和过程，React 会以相对高效的方式根据新的状态构建 React 元素树并且着手重新渲染整个 UI 界面。在 React 得到元素树之后，React 会自动计算出新的树与老树的节点差异，然后根据差异对界面进行最小化重渲染。在差异计算算法中，React 能够相对精确地知道哪些位置发生了改变以及应该如何改变，这就保证了按需更新，而不是全部重新渲染。

### 4.react 生命周期函数

React16.3之前：
- 初始化阶段：
  - getDefaultProps:获取实例的默认属性（Constructor）
  - getInitialState:获取每个实例的初始化状态（class形式不存在）
  - componentWillMount：组件即将被装载、渲染到页面上
  - render:组件在这里生成虚拟的 DOM 节点
  - componentDidMount:组件真正在被装载之后
- 运行中状态：setState() 或 forceUpdate() 方法更新状
  - componentWillReceiveProps:组件将要接收到属性的时候调用
  - shouldComponentUpdate:组件接受到新属性或者新状态的时候（可以返回 false，接收数据后不更新，阻止 render 调用，后面的函数不会被继续执行了）
  - componentWillUpdate:组件即将更新不能修改属性和状态
  - render:组件重新描绘
  - componentDidUpdate:组件已经更新
- 销毁阶段：
  - componentWillUnmount:组件即将销毁
![image](https://github.com/semlinker/reactjs-interview-questions/raw/master/images/phases16.3.jpg)

React16.3之后：
- 初始化阶段：
  - Constructor
  - getDerivedStateFromProps
  - render
  - componentDitMount
- 运行中状态：
  - getDerivedStateFromProps
  - shouldComponetUpdate
  - Pre-commit 在组件实际将更改应用于 DOM 之前，有一个时刻允许 React 通过getSnapshotBeforeUpdate()捕获一些 DOM 信息（例如滚动位置）。
  - render
  - componentDidUpdate
- 销毁阶段：
  - componentWillUnMount
![image](https://github.com/semlinker/reactjs-interview-questions/raw/master/images/phases.png)


### 5.shouldComponentUpdate 是做什么的，（react 性能优化是哪个周期函数？）

shouldComponentUpdate 这个方法用来判断是否需要调用 render 方法重新描绘 dom。因为 dom 的描绘非常消耗性能，如果我们能在 shouldComponentUpdate 方法中能够写出更优化的 dom diff 算法，可以极大的提高性能。

参考[react 性能优化-sf](https//segmentfault.com/a/1190000006254212)

### 6.为什么虚拟 dom 会提高性能?(必考)

虚拟 dom 相当于在 js 和真实 dom 中间加了一个缓存，利用 dom diff 算法避免了没有必要的 dom 操作，从而提高性能。

用 JavaScript 对象结构表示 DOM 树的结构；然后用这个树构建一个真正的 DOM 树，插到文档当中当状态变更的时候，重新构造一棵新的对象树。然后用新的树和旧的树进行比较，记录两棵树差异把 2 所记录的差异应用到步骤 1 所构建的真正的 DOM 树上，视图就更新了。

参考 [如何理解虚拟 DOM?-zhihu](https://www.zhihu.com/question/29504639?sort=created)

### 7.react diff 原理（常考，大厂必考）

- React 通过制定大胆的 diff 策略，将 O(n3) 复杂度的问题转换成 O(n) 复杂度的问题；  
统Diff算法需要找到两个树的最小更新方式，所以需要[两两]对比每个叶子节点是否相同，对比就需要O(n^2)次了，再加上更新（移动、创建、删除）时需要遍历一次，所以是O(n^3)。

- React 通过分层求异的策略，对 tree diff 进行算法优化；

- React 通过相同类生成相似树形结构，不同类生成不同树形结构的策略，对 component diff 进行算法优化；
 
- React 通过设置唯一 key的策略，对 element diff 进行算法优化；

- 合并操作，调用 component 的 setState 方法的时候, React 将其标记为 dirty.到每一个事件循环结束, React 检查所有标记 dirty 的 component 重新绘制.

- 选择性子树渲染。开发人员可以重写 shouldComponentUpdate 提高 diff 的性能。

- 建议，在开发组件时，保持稳定的 DOM 结构会有助于性能的提升；
 
- 建议，在开发过程中，尽量减少类似将最后一个节点移动到列表首部的操作，当节点数量过大或更新操作过于频繁时，在一定程度上会影响 React 的渲染性能。



参考：[React 的 diff 算法](https//segmentfault.com/a/1190000000606216)

### 8.React 中 refs 的作用是什么？

Refs 是 React 提供给我们的安全访问 DOM 元素或者某个组件实例的句柄。我们可以为元素添加 ref 属性然后在回调函数中接受该元素在 DOM 树中的句柄，该值会作为回调函数的第一个参数返回：

何时会使用？
1. 管理焦点，文本选择，或媒体播放。
1. 触发命令动画。
1. 与第三方 DOM 库集成。

几种使用方式：
1. 字符串 String Refs（hoc不能传递，React v16 版本中被移除）
2. 回调函数
3. React.createRef() (React16.3之后)
4. React.forwardRef() (专门处理hoc中的refs， React16.4之后)

```
// js
class CustomForm extends Component {
  handleSubmit = () => {
    console.log("Input Value: ", this.input.value)
  }
  render () {
    return (
      <form onSubmit={this.handleSubmit}>
        <input
          type='text'
          ref={(input) => this.input = input} />
        <button type='submit'>Submit</button>
      </form>
    )
  }
}
```

上述代码中的 input 域包含了一个 ref 属性，该属性声明的回调函数会接收 input 对应的 DOM 元素，我们将其绑定到 this 指针以便在其他的类函数中使用。另外值得一提的是，refs 并不是类组件的专属，函数式组件同样能够利用闭包暂存其值：

```
// js
function CustomForm ({handleSubmit}) {
  let inputElement
  return (
    <form onSubmit={() => handleSubmit(inputElement.value)}>
      <input
        type='text'
        ref={(input) => inputElement = input} />
      <button type='submit'>Submit</button>
    </form>
  )
}
```


### 9.如果你创建了类似于下面的 Twitter 元素，那么它相关的类定义是啥样子的？

```
// js
<Twitter username='tylermcginnis33'>
  {(user) => user === null
    ? <Loading />
    : <Badge info={user} />}
</Twitter>
```

```
// js
import React, { Component, PropTypes } from 'react'
import fetchUser from 'twitter'
// fetchUser take in a username returns a promise
// which will resolve with that username's data.
class Twitter extends Component {
  // finish this
}
```

如果你还不熟悉回调渲染模式（Render Callback Pattern），这个代码可能看起来有点怪。这种模式中，组件会接收某个函数作为其子组件，然后在渲染函数中以 props.children 进行调用：

```
// js
import React, { Component, PropTypes } from 'react'
import fetchUser from 'twitter'
class Twitter extends Component {
  state = {
    user: null,
  }
  static propTypes = {
    username: PropTypes.string.isRequired,
  }
  componentDidMount () {
    fetchUser(this.props.username)
      .then((user) => this.setState({user}))
  }
  render () {
    return this.props.children(this.state.user)
  }
}
```

这种模式的优势在于将父组件与子组件解耦和，父组件可以直接访问子组件的内部状态而不需要再通过 Props 传递，这样父组件能够更为方便地控制子组件展示的 UI 界面。譬如产品经理让我们将原本展示的 Badge 替换为 Profile，我们可以轻易地修改下回调函数即可：

```
// js
<Twitter username='tylermcginnis33'>
  {(user) => user === null
    ? <Loading />
    : <Profile info={user} />}
</Twitter>
```

### 10.展示组件(Presentational component)和容器组件(Container component)之间有何不同

- 展示组件关心组件看起来是什么。展示专门通过 props 接受数据和回调，并且几乎不会有自身的状态，但当展示组件拥有自身的状态时，通常也只关心 UI 状态而不是数据的状态。
- 容器组件则更关心组件是如何运作的。容器组件会为展示组件或者其它容器组件提供数据和行为(behavior)，它们会调用 Flux actions，并将其作为回调提供给展示组件。容器组件经常是有状态的，因为它们是(其它组件的)数据源。

### 11.类组件(Class component)和函数式组件(Functional component)之间有何不同

- 类组件不仅允许你使用更多额外的功能，如组件自身的状态和生命周期钩子，也能使组件直接访问 store 并维持状态
- 当组件仅是接收 props，并将组件自身渲染到页面时，该组件就是一个 '无状态组件(stateless component)'，可以使用一个纯函数来创建这样的组件。这种组件也被称为哑组件(dumb components)或展示组件

### 12.(组件的)状态(state)和属性(props)之间有何不同

- State 是一种数据结构，用于组件挂载时所需数据的默认值。State 可能会随着时间的推移而发生突变，但多数时候是作为用户事件行为的结果。
- Props(properties 的简写)则是组件的配置。props 由父组件传递给子组件，并且就子组件而言，props 是不可变的(immutable)。组件不能改变自身的 props，但是可以把其子组件的 props 放在一起(统一管理)。Props 也不仅仅是数据--回调函数也可以通过 props 传递。

### 13.何为受控组件(controlled component)

在 HTML 中，类似 `<input>`, `<textarea>` 和 `<select>` 这样的表单元素会维护自身的状态，并基于用户的输入来更新。当用户提交表单时，前面提到的元素的值将随表单一起被发送。但在 React 中会有些不同，包含表单元素的组件将会在 state 中追踪输入的值，并且每次调用回调函数时，如 onChange 会更新 state，重新渲染组件。一个输入表单元素，它的值通过 React 的这种方式来控制，这样的元素就被称为"受控元素"。

### 14.何为高阶组件(higher order component)

高阶组件是一个以组件为参数并返回一个新组件的函数。HOC 运行你重用代码、逻辑和引导抽象。最常见的可能是 Redux 的 connect 函数。除了简单分享工具库和简单的组合，HOC 最好的方式是共享 React 组件之间的行为。如果你发现你在不同的地方写了大量代码来做同一件事时，就应该考虑将代码重构为可重用的 HOC。

### 15.为什么建议传递给 setState 的参数是一个 callback 而不是一个对象

因为 this.props 和 this.state 的更新可能是异步的，不能依赖它们的值去计算下一个 state。

### 16.除了在构造函数中绑定 this，还有其它方式吗

你可以使用属性初始值设定项(property initializers)来正确绑定回调，create-react-app 也是默认支持的。在回调中你可以使用箭头函数，但问题是每次组件渲染时都会创建一个新的回调。

### 17.(在构造函数中)调用 super(props) 的目的是什么

在 super() 被调用之前，子类是不能使用 this 的，在 ES2015 中，子类必须在 constructor 中调用 super()。传递 props 给 super() 的原因则是便于(在子类中)能在 constructor 访问 this.props。

### 18.应该在 React 组件的何处发起 Ajax 请求

在 React 组件中，应该在 componentDidMount 中发起网络请求。这个方法会在组件第一次“挂载”(被添加到 DOM)时执行，在组件的生命周期中仅会执行一次。更重要的是，你不能保证在组件挂载之前 Ajax 请求已经完成，如果是这样，也就意味着你将尝试在一个未挂载的组件上调用 setState，这将不起作用。在 componentDidMount 中发起网络请求将保证这有一个组件可以更新了。

### 19.描述事件在 React 中的处理方式。

为了解决跨浏览器兼容性问题，您的 React 中的事件处理程序将传递 合成事件SyntheticEvent 的实例，它是 React 的浏览器本机事件的跨浏览器包装器。

这些 SyntheticEvent 与您习惯的原生事件具有相同的接口，除了它们在所有浏览器中都兼容。有趣的是，React 实际上并没有将事件附加到子节点本身。React 将使用单个事件监听器监听顶层的所有事件。这对于性能是有好处的，这也意味着在更新 DOM 时，React 不需要担心跟踪事件监听器。  
使用event.preventDefault()阻止时间默认冒泡，stopPropagation()无效

### 20.createElement 和 cloneElement 有什么区别？

React.createElement():JSX 语法就是用 React.createElement()来构建 React 元素的。它接受三个参数，第一个参数可以是一个标签名。如 div、span，或者 React 组件。第二个参数为传入的属性。第三个以及之后的参数，皆作为组件的子组件。

```
// js
React.createElement(
    type,
    [props],
    [...children]
)
```

React.cloneElement()与 React.createElement()相似，不同的是它传入的第一个参数是一个 React 元素，而不是标签名或组件。新添加的属性会并入原有的属性，传入到返回的新元素中，而就的子元素奖杯替换。

```
// js
React.cloneElement(
  element,
  [props],
  [...children]
)
```

### 21.React 中有三种构建组件的方式

React.createClass()、ES6 class 和无状态函数。

### 22.react 组件的划分业务组件技术组件？

- 根据组件的职责通常把组件分为 UI 组件和容器组件。
- UI 组件负责 UI 的呈现，容器组件负责管理数据和逻辑。
- 两者通过 React-Redux 提供 connect 方法联系起来。

### 23.简述 flux 思想

Flux 的最大特点，就是数据的"单向流动"。

1. 用户访问 View
2. View 发出用户的 Action
3. Dispatcher 收到 Action，要求 Store 进行相应的更新
4. Store 更新后，发出一个"change"事件
5. View 收到"change"事件后，更新页面

### 24.React 项目用过什么脚手架（本题是开放性题目）

creat-react-app Yeoman 等

### 25.了解 redux 么，说一下 redux 把

- redux 是一个应用数据流框架，主要是解决了组件间状态共享的问题，原理是集中式管理，主要有三个核心方法，action，store，reducer，工作流程是 view 调用 store 的 dispatch 接收 action 传入 store，reducer 进行 state 操作，view 通过 store 提供的 getState 获取最新的数据，flux 也是用来进行数据操作的，有四个组成部分 action，dispatch，view，store，工作流程是 view 发出一个 action，派发器接收 action，让 store 进行数据更新，更新完成以后 store 发出 change，view 接受 change 更新视图。Redux 和 Flux 很像。主要区别在于 Flux 有多个可以改变应用状态的 store，在 Flux 中 dispatcher 被用来传递数据到注册的回调事件，但是在 redux 中只能定义一个可更新状态的 store，redux 把 store 和 Dispatcher 合并,结构更加简单清晰
- 新增 state,对状态的管理更加明确，通过 redux，流程更加规范了，减少手动编码量，提高了编码效率，同时缺点时当数据更新时有时候组件不需要，但是也要重新绘制，有些影响效率。一般情况下，我们在构建多交互，多数据流的复杂项目应用时才会使用它们

### 26.redux 有什么缺点

- 一个组件所需要的数据，必须由父组件传过来，而不能像 flux 中直接从 store 取。
- 当一个组件相关数据更新时，即使父组件不需要用到这个组件，父组件还是会重新 render，可能会有效率影响，或者需要写复杂的 shouldComponentUpdate 进行判断。

### 27.React Fiber架构
Fiber 是 React v16 中新的 reconciliation 引擎，或核心算法的重新实现。React Fiber 的目标是提高对动画，布局，手势，暂停，中止或者重用任务的能力及为不同类型的更新分配优先级，及新的并发原语等领域的适用性

React16 以前，对virtural dom的更新和渲染是同步的。就是当一次更新或者一次加载开始以后，diff virtual dom并且渲染的过程是一口气完成的。如果组件层级比较深，相应的堆栈也会很深，长时间占用浏览器主线程，一些类似用户输入、鼠标滚动等操作得不到响应  

到了 v16.0.0 的时候，React 有一个重大改变——核心代码被重写，引入了叫 Fiber 这个全新的架构。这个架构使得 React 用异步渲染成为可能

React16 用了分片的方式解决上面的问题。 就是把一个任务分成很多小片，当分配给这个小片的时间用尽的时候，就检查任务列表中有没有新的、优先级更高的任务，有就做这个新任务，没有就继续做原来的任务。这种方式被叫做异步渲染(Async Rendering)

生命周期函数也被分为2个阶段了：


```
// 第1阶段 render/reconciliation
componentWillMount
componentWillReceiveProps
shouldComponentUpdate
componentWillUpdate

// 第2阶段 commit
componentDidMount
componentDidUpdate
componentWillUnmount
```


componentWillMount componentWillReceiveProps componentWillUpdate 几个生命周期方法不再安全，由于任务执行过程可以被打断，这几个生命周期可能会执行多次，如果它们包含副作用（比如AJax），会有意想不到的bug。React团队提供了替换的生命周期方法。建议如果使用以上方法，尽量用纯函数，避免以后采坑。shouldComponentUpdate 也可能被多次调用，只是它只返回true或者false，没有副作用，可以暂时忽略。

### 28.函数化的 Hooks

Hooks 的目的，简而言之就是让开发者不需要再用 class 来实现组件。

```
const {value, setValue} = useState();
```

```
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
```
第一，react首次渲染和之后的每次渲染都会调用一遍传给useEffect的函数。而之前我们要用两个声明周期函数来分别表示首次渲染（componentDidMount），和之后的更新导致的重新渲染（componentDidUpdate）。

第二，useEffect中定义的副作用函数的执行不会阻碍浏览器更新视图，也就是说这些函数是异步执行的，而之前的componentDidMount或componentDidUpdate中的代码则是同步执行的。这种安排对大多数副作用说都是合理的，但有的情况除外，比如我们有时候需要先根据DOM计算出某个元素的尺寸再重新渲染，这时候我们希望这次重新渲染是同步发生的，也就是说它会在浏览器真的去绘制这个页面前发生。

怎么跳过一些不必要的计算呢？我们只需要给useEffect传第二个参数即可。用第二个参数来告诉react只有当这个参数的值发生改变时，才执行我们传的副作用函数（第一个参数）。

其他hooks

```
useContext
useReducer
useCallback
useMemo
useRef
useImperativeMethods
useMutationEffect
useLayoutEffect
```
### 29.setState()方法可以只修改一个对象的部分属性吗？
要注意不要返回原对象，应该返回一个新对象

- redux中
```
 dispatch({
   type: 'paramSetting/updateAction',
   payload: {
     param: {
       ...param,
       isLoading: true
     }
   },
})
```

- 普通react对象中

```
// 1.用一个对象调用 setState() 来与状态合并：使用 Object.assign() 创建对象的副本：

const user = Object.assign({}, this.state.user, { age: 42 })
this.setState({ user })


// 2.使用扩展运算符：

const user = { ...this.state.user, age: 42 }
this.setState({ user })

// 3.使用一个函数调用 setState()：

this.setState(prevState => ({
  user: {
    ...prevState.user,
    age: 42
  }
}))
```

### 30.stateless component vs PureComponent(React.memo())
sc 和 PureComponent 各有优缺点。

- sc 的缺点：
当 props 更新或父组件重新渲染就会 re-render

- PureComponent 的缺点
如果 shouldComponentUpdate 为 true，那么相当于多做了两次检查（SCU 一次，re-render 时 diff 一次）

> shallowCompare 比 diffing 算法需要耗费更多的时间

所以一个组件如果经常变更的话，那么 PureComponent 多带来的两次检查会让他通常更慢。

使用这个经验法则：pure component适用于复杂的表单和表格，但它们通常会减慢简单元素（按钮、图标）的效率。

一般来说，sc 适用于小的组件（就索性让它做 diff，diff 的开销小，反正不变就不会改变真实 DOM）。PureComponent 适用于稍大一点的组件，diff 的代价是随着组件的增大而提升，shallowCompare 与组件无关的复杂度无关而是与 props 的数量挂钩，不同的组件和 props 的要根据实际情况来判断。

其他性能优化注意点：

```
// bad 字面量数组和new Array()效果是一样的，始终生成新的实例 [] !== []
<RadioGroup options={this.props.options || []} />

// good
const DEFAULT_OPTIONS = []
<RadioGroup options={this.props.options || DEFAULT_OPTIONS} />
```


```
// bad 每次绑定生成新的函数
<Button onClick={this.update.bind(this)} />

// bad 每次也会生成新的函数
<Button
  onClick={() => {
    console.log("Click");
  }}
/>

//good 
const onClick = () => {
  console.log("Click");
}
<Button
  onClick={this.onClick}
/>
```

