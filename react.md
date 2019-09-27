# React面试复习

## 一、生命周期执行顺序详解

#### 1、组件生命周期的执行次数是什么样子的？？？

```
只执行一次： constructor、componentWillMount、componentDidMount

执行多次：render 、子组件的componentWillReceiveProps、componentWillUpdate、componentDidUpdate

有条件的执行：componentWillUnmount（页面离开，组件销毁时）

不执行的：根组件（ReactDOM.render在DOM上的组件）的componentWillReceiveProps（因为压根没有父组件给传递props）
```

#### 2、组件的生命周期执行顺序是什么样子的？？？

　　假设组件嵌套关系是 App里有parent组件，parent组件有child组件。
![img](https://images2018.cnblogs.com/blog/1414709/201808/1414709-20180830111120627-1774037102.png)

**如果不涉及到setState更新，第一次渲染的顺序如下：**
![img](https://images2018.cnblogs.com/blog/1414709/201808/1414709-20180830111229956-69121477.png)

```
App：   constructor --> componentWillMount -->  render --> 
parent: constructor --> componentWillMount -->  render --> 
child:    constructor --> componentWillMount -->  render  --> 
componentDidMount (child) -->  componentDidMount (parent) --> componentDidMount (App)
```

**这时候触发App的setState事件**

![img](https://images2018.cnblogs.com/blog/1414709/201808/1414709-20180830111331526-1907166573.png)

```repl
App：   componentWillUpdate --> render --> 
parent: componentWillReceiveProps --> componentWillUpdate --> render --> 
child:    componentWillReceiveProps --> componentWillUpdate --> render -->
componentDidUpdate (child) -->  componentDidUpdate (parent) --> componentDidUpdate (App)
```

**那如果是触发parent的setState呢？**

![img](https://images2018.cnblogs.com/blog/1414709/201808/1414709-20180830111503596-744353039.png)

```repl
parent： componentWillUpdate --> render --> 
child:     componentWillReceiveProps --> componentWillUpdate --> render --> 
componentDidUpdate (child) -->  componentDidUpdate (parent) 
```

**那如果是只是触发了child组件自身的setState呢？**

![img](https://images2018.cnblogs.com/blog/1414709/201808/1414709-20180830111543126-1591848462.png)

```repl
child： componentWillUpdate --> render -->  componentDidUpdate (child)
```

##### 结论

1. 如图：完成前的顺序是从根部到子部，完成时时从子部到根部。（类似于事件机制）
2. 每个组件的红线（包括初次和更新）生命周期时一股脑执行完毕以后再执行低一级别的红线生命周期。

![img](https://images2018.cnblogs.com/blog/1414709/201808/1414709-20180830111612245-37028468.png)

1. 第一级别的组件setState是不能触发其父组件的生命周期更新函数，只能触发更低一级别的生命周期更新函数。

总结起来就如下图：

![img](https://images2018.cnblogs.com/blog/1414709/201808/1414709-20180830111656526-534965409.png)

------

提问：
那么这里提一个问题，如果App里面有多个parent1 parent2 ...，parent里由多个child，那么生命周期执行顺序应该时什么样的？？？？

结论：
一套组件（父包括子，子包括孙）执行的时候一个整体，执行完毕在执行下一套，用到这里就是App里先执行parent1和parent1的子，子的子。。。，然后完毕再执行parent2这一套。

#### 3、什么时候该用componentWillReceiveProps？

是否每个子组件都需要componentWillReceiveProps生命周期函数来更新数据吗？ 你的原则是？？

```
A、开始前首先需要知道componentWillReceiveProps函数有一个参数nextProps，它是一个 { 对象 } ，从单词就可以看出它是update时候（也就是下一次）父组件传递过来的props。

B、还要知道 "第一条中" 所讲解的有些生命周期函数只执行一次，而有的执行多次，其中componentWillReceiveProps执行多次，而constructor等执行一次。

C、还需知道在子组件中每次传递过来的this.props对象其实和componentWillReceiveProps的nextProps是一样的，都是最新的。

D、由"第一条"得知： componentWillReceiveProps生命周期是在更新子组件最先执行的，优先于compoentWillUpdate，更优先于render。

E、render函数里不能使用setState()，否则会造成死循环。
```

那么知道了以上呢？

由C得知， this.props 和 componentWillReceiveProps的nextProps都是一样的，通过this.props就可以取到最新的值， 那么componentWillReceiveProps还有必要吗？

所以：大部分情况下 componentWillReceiveProps 生命周期函数是没用的，即可以略去不写，因为它确实没什么用。

**但是情况1：**

由D得知，componentWillReceiveProps是最先执行的，所以在其内可以setState(｛｝)，在接下来的render中能拿到最新的state后值，再加上B得知，

如果是下面这种情况： 在constructor函数中初始化了某个state，必须用 componentWillReceiveProps 来更新state，以便render中为新的state值。

![img](https://images2018.cnblogs.com/blog/1414709/201808/1414709-20180830112327385-408207776.png)
![img](https://images2018.cnblogs.com/blog/1414709/201808/1414709-20180830112334365-1343110317.png)

**情况2：**

　　如果父组件有一些请求，每次参数更新的时候才发请求，同时和子组件的关系比较密切，

可以将数据请求放在componentWillReceiveProps进行执行，需要传的参数则从(nextProps)中获取。

而不必将所有的请求都放在父组件中，于是该请求只会在该组件渲染时才会发出，从而减轻请求负担。

**情况3：**

　　watch监听props值变化，对子组件进行处理，比如：当传入的props.value发生变化，执行一些动作。

　　如果你接触过vue，会知道vue中有一个关于watch的选项，是根据setter获取新旧值，进行动作的执行

　　而react中最合适做watch的时机是在componentWillReceiveProps中

```
componentWillReceiveProps(nextProps) {
        // this.props中的值是旧值
        // nextProps中的值是新值
    const { value: oldValue } = this.props;
    const { value: newValue } = nextProps;
    if (newValue !== oldValue) {
            // TODO...
    }
}
```

结论：

1. 大部分情况下 componentWillReceiveProps 生命周期函数是没用的，即可以略去不写，
2. 但是在constructor函数中初始化了某个state，必须用 componentWillReceiveProps 来更新state，不可省去，否则render中的state将得不到更新。
   同时如果您想在子组件监听watch值变化做处理，也可以用到componentWillReceiveProps
3. 使用componentWillReceiveProps的时候，不要去向上分发，调用父组件的相关setState方法，否则会成为死循环。

附:React生命周期官方解析

```
componentWillMount 在渲染前调用,在客户端也在服务端。

componentDidMount : 在第一次渲染后调用，只在客户端。之后组件已经生成了对应的DOM结构，可以通过this.getDOMNode()来进行访问。 如果你想和其他JavaScript框架一起使用，可以在这个方法中调用setTimeout, setInterval或者发送AJAX请求等操作(防止异步操作阻塞UI)。

componentWillReceiveProps 在组件接收到一个新的 prop (更新后)时被调用。这个方法在初始化render时不会被调用。

shouldComponentUpdate 返回一个布尔值。在组件接收到新的props或者state时被调用。在初始化时或者使用forceUpdate时不被调用。 
可以在你确认不需要更新组件时使用。

componentWillUpdate在组件接收到新的props或者state但还没有render时被调用。在初始化时不会被调用。

componentDidUpdate 在组件完成更新后立即调用。在初始化时不会被调用。

componentWillUnmount在组件从 DOM 中移除的时候立刻被调用。
```

## 性能优化

## 一、 使用纯组件

如果 React 组件为相同的状态和 props 渲染相同的输出，则可以将其视为纯组件。

对于像 this 的类组件来说，React 提供了 PureComponent 基类。扩展 React.PureComponent 类的类组件被视为纯组件。

它与普通组件是一样的，只是 PureComponents 负责 shouldComponentUpdate——它对状态和 props 数据进行浅层比较（shallow comparison）。

如果先前的状态和 props 数据与下一个 props 或状态相同，则组件不会重新渲染。

 什么是浅层渲染？

在对比先前的 props 和状态与下一个 props 和状态时，浅层比较将检查它们的基元是否有相同的值（例如：1 等于 1 或真等于真），还会检查更复杂的 JavaScript 值（如对象和数组）之间的引用是否相同。

比较基元和对象引用的开销比更新组件视图要低。

因此，查找状态和 props 值的变化会比不必要的更新更快。

```jsx
import React from 'react';

export default class ApplicationComponent extends React.Component {
    constructor() {
        super();
        this.state = {
            name: "Mayank"
        }
    }
    updateState = () => {
        setInterval(() => {
            this.setState({
                name: "Mayank"
            })
        }, 1000)
    }
    componentDidMount() {
        this.updateState();
    }
    render() {
        console.log("Render Called Again")
        return (
            <div>
                <RegularChildComponent name={this.state.name} />
                <PureChildComponent name={this.state.name} />
            </div>
        )
    }
}
class RegularChildComponent extends React.Component {
    render() {
        console.log("Regular Component Rendered..");
        return <div>{this.props.name}</div>;
    }
}
class PureChildComponent extends React.PureComponent {
    // Pure Components are the components that do not re-render if the State data or props data is still the same
    render() {
        console.log("Pure Component Rendered..")
        return <div>{this.props.name}</div>;
    }
}
```

在上面的示例中，状态被传播到子组件 RegularChildComponent 和 PureChildComponent。PureChildComponent 是一个纯组件。

setstate 在一秒的间隔之后被调用，这将重新触发组件的视图渲染。由于初始 props 和新 props 的值相同，因此组件（PureChildComponent）不会被重新渲染。

状态的浅层比较表明 props 或状态的数据没有变化，因此不需要渲染组件，从而提升了性能。



## 二、使用 React.memo 进行组件记忆

React.memo 是一个高阶组件。

它很像 PureComponent，但 PureComponent 属于 Component 的类实现，而“memo”则用于创建函数组件。

这里与纯组件类似，如果输入 props 相同则跳过组件渲染，从而提升组件性能。

它会记忆上次某个输入 prop 的执行输出并提升应用性能。即使在这些组件中比较也是浅层的。

你还可以为这个组件传递自定义比较逻辑。

用户可以用自定义逻辑深度对比（deep comparison）对象。如果比较函数返回 false 则重新渲染组件，否则就不会重新渲染。

```jsx
function CustomisedComponen(props) {
    return (
        <div>
            <b>User name: {props.name}</b>
            <b>User age: {props.age}</b>
            <b>User designation: {props.designation}</b>
        </div>
    )
}

// The component below is the optimised version for the Default Componenent

// The Component will not re-render if same props value for "name" property

var memoComponent = React.memo(CustomisedComponent);
```

上面的组件将对前后两个 props 的值进行浅层比较。

如果我们将对象引用作为 props 传递给 memo 组件，则需要一些自定义登录以进行比较。在这种情况下，我们可以将比较函数作为第二个参数传递给 React.memo 函数。

假设 props 值（user）是一个对象引用，包含特定用户的 name、age 和 designation。

这种情况下需要进行深入比较。我们可以创建一个自定义函数，查找前后两个 props 值的 name、age 和 designation 的值，如果它们不相同则返回 false。

这样，即使我们将参考数据作为 memo 组件的输入，组件也不会重新渲染。

```jsx 
// The following function takes "user" Object as input parameter in props

function CustomisedComponen(props) {
    return (
        <div>
            <b>User name: {props.user.name}</b>
            <b>User age: {props.user.age}</b>
            <b>User designation: {props.user.designation}</b>
        </div>
    )
}

function userComparator(previosProps, nextProps) {
    if(previosProps.user.name == nextProps.user.name ||
       previosProps.user.age == nextProps.user.age ||
       previosProps.user.designation == nextProps.user.designation) {
        return false
    } else {
        return true;
    }
}

var memoComponent = React.memo(CustomisedComponent, userComparator);
```

上面的代码提供了用于比较的自定义逻辑