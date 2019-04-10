---
layout:     post
title:      "React组件设计模式"
subtitle:   " \"React组件设计实践\""
date:       2019-03-02 23:00:00
author:     "iwalking11"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
  - components
  - react
---


## 1. 现代 js 框架存在的根本原因
现在前端框架非常多了，如果让我们回答 “为什么要用前端框架” 这个问题，你觉得是下面这些原因吗？

- 组件化。
- 拥有强大的开源社区。
- 拥有大量第三方库解决大部分问题。
- 拥有大量现成的第三方组件。
- 拥有浏览器拓展/工具帮助快速 debug。
- 友好的支持单页应用。

不，这些都不是根本原因，最多算前端框架的营销手段。  
最根本原因是：
解决 UI 与状态同步的难题。

### 如何做到
有两种思路：

1. 组件级重渲染：比如 React，当状态改版后，映射出改变后的虚拟 DOM，最终改变当前组件映射的真实 DOM，这个过程被称为 reconciliation。
1. 监听修改：比如 Angluar 和 Vue.js，状态改变直接触发对应 DOM 节点中 value 值的变化。

这里稍微说明下，React 虽然是整体渲染，但在虚拟 DOM 作用下，效率不比 observable 低。observable 在值不能完整映射 UI 时，也需要做更大范围的 rerender。另外，Vue.js 与 Angluar 也早已采用了虚拟 DOM。


### 那 web components 呢？
问题就出现在 html、js、css 三者分离上。

html、css、js 各是一套独立的体系，但 js 又能同时控制 html 与 css，那为了解决同步问题，最好将控制权全部交给 js。

这样 web components 的问题也就好理解了，web components 解决的是 html 问题，注定与 js 无关。

html 官方规范估计很难出现现代框架的设计了，因为官方设计中前端三剑客是相互分离的方案，为了解决现阶段前端框架的问题，html 必须由 js 完全接管，这几乎就是 jsx，或者支持 template 语法的 html，可这与最初网页设计思路是违背的。

html 是独立的，甚至可以不依赖 js 运行，这天然导致了 UI 与状态同步这个难题。
1. 现代 js 框架主要在解决 UI 与状态同步的问题。
1. 仅使用原生 js 难以写出复杂、高效、又容易维护的 UI 代码。
1. Web components 没有解决这个主要问题。
1. 虽然使用虚拟 DOM 库很容易造一个解决问题的框架，但不建议你真的这么做！

## 2. React的基础原则
要真正理解 React，开发者必须要明白这几点：
1. React 界面完全由数据驱动；
1. React 中一切都是组件；
1. props 是 React 组件之间通讯的基本方式。

## 3. 组件实践（1）：如何定义清晰可维护的接口
原则：
1. 保持接口小，props 数量要少；使用 propTypes 来定义组件的 props
1. 根据数据边界来划分组件，充分利用组合（composition）；
1. 把 state 往上层组件提取，让下层组件只需要实现为纯函数。

## 4. 组件实践（2）：组件的内部实现
1. 尽量每个组件都有自己专属的源代码文件；（低耦合）
1. 用解构赋值（destructuring assignment）的方法获取参数 props 的每个属性值；
1. 利用属性初始化（property initializer）来定义 state 和成员函数。

## 5. 组件实践（3）：组件化样式
1. React 将内容、样式和动态功能聚集在一个模块中，是高聚合的表现；
1. React 原生 style 属性的用法；
1. 使用ES module导入 CSS 文件
1. 组件化样式 styled jsx 的用法。

## 6. 组件设计模式（1）：聪明组件和傻瓜组件
这么做的好处，是可以灵活地修改数据状态管理方式，比如，最初你可能用 Redux 来管理数据，然后你想要修改为用 Mobx，如果按照这种模式分割组件，那么，你需要改的只有聪明组件，傻瓜组件可以保持原状。
### 1. Component、PureComponent、Stateless Function区别
### 2. React.memo
虽然 PureComponent 可以提高组件渲染性能，但是它也不是没有代价的，它逼迫我们必须把组件实现为 class，不能用纯函数来实现组件。

如果你使用 React v16.6.0 之后的版本，可以使用一个新功能 React.memo 来完美实现 React 组件：


```
const Joke = React.memo(() => (
    <div>
        <img src={SmileFace} />
        {this.props.value || 'loading...' }
    </div>
));
```

React.memo 既利用了 shouldComponentUpdate，又不要求我们写一个 class，这也体现出 React 逐步向完全函数式编程前进。

## 7. 组件设计模式（2）：高阶组件（HOC） 替代mixin
### 1. 基本形式

```
onst withDoNothing = (Component) => {
  const NewComponent = (props) => {
    return <Component {...props} />;
  };
  return NewComponent;
};
```

1. 高阶组件不能去修改作为参数的组件，高阶组件必须是一个纯函数，不应该有任何副作用。
1. 高阶组件返回的结果必须是一个新的 React 组件，这个新的组件的 JSX 部分肯定会包含作为参数的组件。
1. 高阶组件一般需要把传给自己的 props 转手传递给作为参数的组件。

### 2. 用高阶组件抽取共同逻辑
### 3. 高阶组件的高级用法
可以传入多个 React 组件给高阶组件

```
const withLoginAndLogout = (ComponentForLogin, ComponentForLogout) => {
  const NewComponent = (props) => {
    if (getUserId()) {
      return <ComponentForLogin {...props} />;
    } else {
      return <ComponentForLogout{...props} />;
    }
  }
  return NewComponent;
};

const TopButtons = withLoginAndLogout(
  LogoutButton,
  LoginButton
);
```
### 4. 链式调用高阶组件
挨个使用高阶组件包装，
代码如下：

```
const X1 = withOne(X);
const X2 = withTwo(X1);
const X3 = withThree(X2);
const SuperX = X3; //最终的SuperX具备三个高阶组件的超能力
```
避免使用中间变量 X1 和 X2，直接连续调用高阶组件，如下

```
const SuperX = withThree(withTwo(withOne(X)));
```
对于 X 而言，它被高阶组件包装了，至于被一个高阶组件包装，还是被 N 个高阶组件包装，没有什么差别。而高阶组件本身就是一个纯函数，纯函数是可以组合使用的，所以，我们其实可以把多个高阶组件组合为一个高阶组件，然后用这一个高阶组件去包装X，代码如下：
```
const hoc = compose(withThree, withTwo, withOne);
const SuperX = hoc(X);
```
在上面代码中使用的 compose，是函数式编程中很基础的一种方法，作用就是把多个函数组合为一个函数，在很多开源的代码库中都可以看到，下面是一个参考实现：

```
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```
假如一个应用中多个组件都需要同样的多个高阶组件包装，那就可以用 compose 组合这些高阶组件为一个高阶组件，这样在使用多个高阶组件的地方实际上就只需要使用一个高阶组件了。

### 5. 不要滥用高阶组件
当 React 渲染出错的时候，靠组件的 displayName 静态属性来判断出错的组件类，而高阶组件总是创造一个新的 React 组件类，所以，每个高阶组件都需要处理一下 displayName。

```
const withExample = (Component) => {
  const NewComponent = (props) => {
    return <Component {...props} />;
  }
  
  NewComponent.displayName = `withExample(${Component.displayName || Component.name || 'Component'})`;
  
  return NewCompoennt;
};

```
使用高阶组件，一定要非常小心，要避免重复产生 React 组件

```
// bad
const Example = () => {
  const EnhancedFoo = withExample(Foo);
  return <EnhancedFoo />
}

//good
const EnhancedFoo = withExample(Foo);
const Example = () => {
  return <EnhancedFoo />
}
```
不要在render函数中使用高阶组件

```
render() {
  // 每一次render函数调用都会创建一个新的EnhancedComponent实例
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // 每一次都会使子对象树完全被卸载或移除
  return <EnhancedComponent />;
}
```
这个问题不仅仅关乎于性能，卸载组件会造成组件状态和其子元素全部丢失。

相反地，在组件定义外应用高阶组件，以便生成的组件只会被创建一次。然后，它的标识符在每次渲染中都是相同的。无论如何，这才是你想要的。

在一些极少的例子中你需要动态地引用高阶组件，你可以在组件的声明周期函数中使用或者在构造函数中使用。

### 6. Refs不会被传递
虽然高阶组件的惯例是将所有 属性(props) 传递给包裹的组件，但是对 refs 不起作用。 那是因为 ref 不是一个真正的属性(props) - 不像 key。React 对它进行了特殊处理。 如果你向一个由高阶组件创建的组件的元素添加ref应用，那么ref指向的是最外层容器组件实例的，而不是包裹组件。

解决这个问题的方法是使用 React.forwardRef API

## 8. 组件设计模式（3）：渲染属性 (Render Props)
### 概念
术语 “render prop” 是指一种技术，用于使用一个值为函数的 prop 在 React 组件之间的代码共享。

带有渲染属性(Render Props)的组件需要一个返回 React 元素并调用它的函数，而不是实现自己的渲染逻辑

```
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```

使用 render props 的库包括 React Router 和 Downshift。

###  依赖注入  

render props 其实就是 React 世界中的“依赖注入”（Dependency Injection)。

所谓依赖注入，指的是解决这样一个问题：逻辑 A 依赖于逻辑 B，如果让 A 直接依赖于 B，当然可行，但是 A 就没法做得通用了。依赖注入就是把 B 的逻辑以函数形式传递给 A，A 和 B 之间只需要对这个函数接口达成一致就行，如此一来，再来一个逻辑 C，也可以用一样的方法重用逻辑 A。

在上面的代码示例中，Login 和 Auth 组件就是上面所说的逻辑 A，而传递给组件的函数类型 props，就是逻辑 B 和 C。
###  render props 和高阶组件的比较
- render props 模式的应用，就是做一个 React组件
- 而高阶组件，虽然名为“组件”，其实只是一个产生 React 组件的函数  

render props 不像高阶组件有那么多毛病，如果说 render props 有什么缺点，那就是 render props 不能像高阶组件那样链式调用，当然，这并不是一个致命缺点。  
render props 相对于高阶组件还有一个显著优势，就是对于新增的 props 更加灵活。

所以，**当需要重用 React 组件的逻辑时，建议首先看这个功能是否可以抽象为一个简单的组件；如果行不通的话，考虑是否可以应用 render props 模式；再不行的话，才考虑应用高阶组件模式。**

这并不表示高阶组件无用武之地，在后续章节，我们会对 render props 和高阶组件分别讲解具体的实例。
### 注意
你可以在 react-motion API 里看到这一技术的使用。

由于这一技术有些不寻常，当你在设计一个类似的 API 时，你可能要直接地在你的 propTypes 里声明 children 应为一个函数类型。

```
Mouse.propTypes = {
  children: PropTypes.func.isRequired
};
```
### 小结
render props就是A组件把一个返回jsx元素的函数作为props传递给另外一个组件B，在B组件的render方法中又会调用该函数生成jsx元素，函数参数可以通过B组件动态设置，反过来传递给函数。  
这个jsx元素在A组件或者其他引用B组件的组件中又可以动态设置，以此达到共用B组件的逻辑。

## 9. 组件设计模式（4）：提供者模式
### React v16.3.0 之前的提供者模式
在 React v16.3.0 之前，要实现提供者，就要利用Context API实现一个 React 组件，不过这个组件要做两个特殊处理。

1. 需要实现 getChildContext 方法，用于返回“上下文”的数据；
1. 需要定义 childContextTypes 属性，声明“上下文”的结构。

### React v16.3.0 之后的提供者模式
到了 React v16.3.0 的时候，新的 Context API 出来了，这套 API 毫不掩饰自己就是“提供者模式”的实现，命名上就带 “Provider” 和 “Consumer”。

```
const ThemeContext = React.createContext();
const ThemeProvider = ThemeContext.Provider;
const ThemeConsumer = ThemeContext.Consumer;


 <ThemeProvider value={{mainColor: 'green', textColor: 'red'}} >
    <Page />
  </ThemeProvider>
  
  
  class Subject extends React.Component {
  render() {
    return (
      <ThemeConsumer>
        {
          (theme) => (
            <h1 style={{color: theme.mainColor}}>
              {this.props.children}
            </h1>
          )
        }
      </ThemeConsumer>
    );
  }
}
```

“提供者”，它可以提供一些信息，而且这些信息在它之下的所有组件，无论隔了多少层，都可以直接访问到，而不需要通过 props 层层传递。

**避免 props 逐级传递，即是提供者的用途。**

## 10. 组件设计模式（5）：组合组件
组合组件模式要解决的是这样一类问题：父组件想要传递一些信息给子组件，但是，如果用 props 传递又显得十分麻烦。

一看到这个问题描述，读者应该能立刻想到上一节我们介绍过的 Context API，利用 Context，可以让组件之间不用 props 来传递信息。

不过，使用 Context 也不是完美解法，上一节我们介绍过，使用 React 在 v16.3.0 之后提供的新的 Context API，需要让“提供者”和“消费者”共同依赖于一个 Context 对象，而且消费者也要使用 render props 模式。

如果不嫌麻烦，用 Context 来解决问题当然好，但是我们肯定会想有没有更简洁的方式。

### 问题描述
为了让问题更加具体，我们来解决一个实例。

很多界面都有 Tab 这样的元件，我们需要一个 Tabs 组件和 TabItem 组件，Tabs 是容器，TabItem 是一个一个单独的 Tab，因为一个时刻只有一个 TabItem 被选中，很自然希望被选中的 TabItem 样式会和其他 TabItem 不同。

```
<TabItem active={true} onClick={this.onClick}>One</TabItem>
<TabItem active={false} onClick={this.onClick}>Two</TabItem>
<TabItem active={false} onClick={this.onClick}>Three</TabItem> 
```
上面的 TabItem 组件接受 active 这个 props，如果 true 代表当前是选中状态，当然可以工作，但是，也存在大问题：

- 每次使用 TabItem 都要传递一堆 props，好麻烦；
- 每增加一个新的 TabItem，都要增加对应的 props，更麻烦；
- 如果要增加 TabItem，就要去修改 Tabs 的 JSX 代码，超麻烦。

我们不想要这么麻烦，理想情况下，我们希望可以随意增加减少 TabItem 实例，不用传递一堆 props，也不用去修改 Tabs 的代码，最好代码就这样：

    <Tabs>
      <TabItem>One</TabItem>
      <TabItem>Two</TabItem>
      <TabItem>Three</TabItem>
    </Tabs>
像上面这样，Tabs 和 TabItem 不通过表面的 props 传递也能心有灵犀，二者之间有某种神秘的“组合”，就是我们所说的“组合组件”。

### 实现方式

#### 1. 提供者(Provider)模式

##### 1. 创建一个Context

```
export const TabsContext = React.createContext();
```

##### 2. 创建Provider

```
class TabsProvider extends Component {
  state = {
      selected: this.props.selected
  }
  render() {
    return (
      <TabsContext.Provider 
        value={{
          selected: this.state.selected,
          handleClick: value => {this.setState({
            selected: value
        })}
      }}>
        {this.props.children}
      </TabsContext.Provider>
    )
  }
}
export default TabsProvider;
```

##### 3. 实现Tabs组件

```
class Tabs extends React.Component {

  // 将TabItem设置为Tabs的静态属性，
  // 使用时可以直接从Tabs中引用
  static TabItem = TabItem;
 
  render() {
    return (
      <div class={styles.container}>
        {children}
      </div>
    )
  }
}
export default Tabs;
```

##### 4. 实现TabItem组件
使用 Consumer 订阅状态更改,并且调用回调函数  
Consumer不能直接将TabItem包裹其中，需要使用render props
```
export const TabItem = itemValue => (
  <TabContext.Consumer>
    {value => {
       const {selected, handleClick} = value;
       // 通知Provider选中条目selected发生改变
       const onItemClick = ()=>{
           handleClick(itemValue);
       }
       return (
         <div 
           onClick = {onItemClick}
           class={selected === itemValue 
           ? styles.selected 
           : styles.unSelected}>
           {this.props.child}
         </div>
       )
    }}
  </TabContext.Consumer>
)
```

##### 5. 利用Provider包裹Tabs组件以及其子组件使用

```
class App extends React.Component {
  render() {
    return (
      <TabsProvider selected='a'>
        <Tabs>
          <TabItem value='a'>a</TabItem>
          <TabItem value='b'>b</TabItem>
        </Tabs>
      </TabsProvider>
    );
  }
}
```

#### 2. render props 模式
上面的方法中，我们在Context.consumer中就已经使用过render props模式了，能不能不用Context来实现呢
##### 1. Tabs组件

```
class Tabs extends React.component {
  state = {
    selected: this.props.value
  }
  const handleClick = value => {
    this.setState(selected: value);
  }
  render() {
    return (
      {this.props.children(this.state.selected, this.handleClick)}
    );
  }
}
export default Tabs;
```

##### 2. TabItem组件

```
export const TabItem = (value, selected, handleClick) => {
  const onItemClick = () => {
    handleClick(value);
  }
  return (
    <div 
      onClick = {onItemClick}
      class={selected === value 
      ? styles.selected 
      : styles.unSelected}>
      {this.props.child}
    </div>
  )
}
```

##### 3. 使用方式

```
<Tabs value='a'>
  {(selected, handleClick) => (
    <React.Fragment>
      <TabItem 
        value='a' selected={selected}, handleClick={handleClick}>
        a
      </TabItem>
      <TabItem 
        value='b' selected={selected}, handleClick={handleClick}>
        b
      </TabItem>
      <TabItem 
        value='c' selected={selected}, handleClick={handleClick}>
        c
      </TabItem>
    </React.Fragment>
  )}
    
    <Tabs.TabItem value='b'>b</Tabs.TabItem>
    <Tabs.TabItem value='c'>c</Tabs.TabItem>
</Tabs>
```


#### 3. 组合模式
##### 1. 先实现TabItem组件

```
export const TabItem = {
  value, active, handleClick, children
  } => {
  const onItemClick = ()=> {
    handleClick(value);
  }
  return (
    <div 
      onClick = {onItemClick}
      class = {active 
      ? styles.selected 
      : styles.unSelected}>
      {clidren}
    </div>
  )
}
```

##### 2.再实现Tabs组件

```
class Tabs extends React.component {

  // 同样将TabItem设置为Tabs的静态属性
  static TabItem = TabItem;
  
  state = {
      seleted : this.props.value
  }
  
  render() {
    const newChildren =  React.Children.map(this.porops.children, child => {
      if (child.type) {
        return React.cloneElement(child, {
          active: this.state.seleted === child.value,
          handleClick: value => {
            this.setState(selected: value);
          } 
        });
      } else {
        return child;
      }
    })
    return (
      <Fragment>
        {newChildren}
      </Fragment>
    )
  }
}
```
在 render 函数中，我们用了 React 中不常用的两个 API：
- React.Children.map
- React.cloneElement  
使用 React.Children.map，可以遍历 children 中所有的元素，因为 children 可能是一个数组嘛。Children.map() 类似于 Array.map() 方法。但请务必使用Children.map()，因为 children.props 具有不透明的数据结构，使得 Array.map() 方法不适合此用例。

使用 React.cloneElement 可以复制某个元素。这个函数第一个参数就是被复制的元素，第二个参数可以增加新产生元素的 props，我们就是利用这个机会，把 active 和 onClick 添加了进去。

##### 3. Tabs组件使用方式
最终的使用效果和我们最开始的需求一致

```
<Tabs value='a'>
  <Tabs.TabItem value='a'>a</Tabs.TabItem>
  <Tabs.TabItem value='b'>b</Tabs.TabItem>
  <Tabs.TabItem value='c'>c</Tabs.TabItem>
</Tabs>
```

#### 4. 3种模式比较
1. Provider模式  
我们使用新的 Context API来保存一个全局的value对象在Provider中
。  
任何组件，我们只要用Consumer包裹就可以通过render props的方式
获取到Provider中的value对象。  
可这种设计模式的问题在于它需要一些初始设置才能工作，虽然我们可以在应用程序中的任何地方使用此组件，但它仍然不是真正可重用的。我们仍然需要 Context 的引用才能使其工作。每个子组件都要通过Consumer去绑定，不太方便
2. render props模式  
父组件的props(props.children)变量为函数，函数返回值为我们的子组件，Tabs中的每个子组件都可以访问所有 props。 它本质上给了我们与 context API 相同的 props 曝露，我们不必手动将 props 传递给每个子项。 这种对组件设计的简单调整解决了我们上面Provider模式提到的问题。
3. 组合模式  
上面的代码可以看出来，对于组合组件这种实现方式，TabItem 非常简化；Tabs 稍微麻烦了一点，但是好处就是把复杂度都封装起来了，从使用者角度，连 props 都看不见。

所以，应用组合组件的往往是共享组件库，把一些常用的功能封装在组件里，让应用层直接用就行。在 antd 和 bootstrap 这样的共享库中，都使用了组合组件这种模式。

如果你的某两个组件并不需要重用，那么就要谨慎使用组合组件模式，毕竟这让代码复杂了一些。

而且组合模式不是很灵活，组件需要是父组件的直接子组件，否则 props 传递会中断。这时候只能使用上面的两种模式了



## 参考
- [React 官方文档](http://react.html.cn/docs/render-props.html)  
- [[译] 使用 Render props 吧！（比较Mixin、HOC、Render props）](https://juejin.im/post/5a3087746fb9a0450c4963a5)  
- [如何掌握高级React设计模式: 复合组件【译】](https://zhuanlan.zhihu.com/p/43039142)
- [如何掌握React设计模式: Context API【译】](https://zhuanlan.zhihu.com/p/43220287)
- [如何掌握高级react设计模式: Render Props](https://zhuanlan.zhihu.com/p/43457891)
- [揭密React setState](https://zhuanlan.zhihu.com/p/43522965)

