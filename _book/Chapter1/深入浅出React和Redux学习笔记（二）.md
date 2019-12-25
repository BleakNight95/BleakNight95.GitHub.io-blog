# 深入浅出React和Redux学习笔记（二）

## 设计高质量的React组件

构建高质量的React组件的原则和方法：

1. 划分组件边界的原则；

2. React组件的数据种类；

3. React组件的生命周期；

   

## 1.易于维护的组件的设计要素

**分而治之：**每个小的组件只关注实现单个功能，但这些功能组合起来，也能满足复杂的实现需求。

拆分组件最关键的就是确定组件的边界，每个组件都应该独立的存在。

组件的划分要满足高内聚(High Cohesion)和低耦合(Low Coupling)的原则。

**高内聚：**把逻辑紧密相关的内容放在一个组件。

**低耦合：**不同组件之间的依赖关系要尽量弱化，每个组件要尽量独立。

## 2.React组件的数据

React组件的数据分为：prop和state。

prop:组件的对外接口；

state:组建的内部状态；

对外使用prop，对内使用state。

### 2.1React的prop

prop(property):从外部传递给组件的数据，一个React组件通过自己能够接受的prop就定义了自己对外的公共接口。

1. 给prop赋值

2. 读取prop值

3. propTypes检查

   组件规范的内容：

   1. 组件支持哪些prop
   2. prop的格式

### 2.2React的state

1. 初始化state

   ```
   this.state = {
   
   }
   ```

2. 读取和更新state

   ```
   this.setState({
   
   })
   ```

   

### 2.3 prop和state的对比

prop和state的区别：

1. prop用于定义外部接口，state用于定义内部状态；

2. prop的赋值在外部世界使用组件时，state的赋值在组件内部；

3. 组件不应该改变prop的值，而state存在的目的就是让组件来改变；

   

## 3.组件的生命周期

React组件的生命周期：

1. 装载过程(Mount):组件第一次在DOM树中渲染的过程；
2. 更新过程(Update):组件被重新渲染的过程；
3. 卸载过程(Unmount):组件从DOM中删除的过程；

三个不同的过程，React库会依次调用组建的成员函数，这些函数称为生命周期函数。

定制一个组件，实际上就是制定这些生命周期函数。

### 3.1装载过程

装载过程：

1. constructor

2. getInitalState

3. getDefaultProps

4. componentWillMount

5. render

6. componentDidMount

   

**1.constructor**

ES6中每个类的构造函数，要创造一个组件类的实例，就会调用对于的构造函数。

并不是每个组件都需要定义自己的构造函数，无状态的React组件往往就不需要定义构造函数。

React组件需要构造函数的目的：

1. 初始化state；

2. 绑定成员函数的this环境；

   

**2.getInitalState和getDefaultProps**

getInitalState函数的返回值会用来初始化组件的this.state，此方法只有用React.createClass方法创造的组件类才会发生作用。

getDefaultProps函数的返回值可以作为props的初始值，此方法只有用React.createClass方法创造的组件类才会发生作用。

**3.render**

render函数是React组件中最重要的函数。

render函数不做实际的渲染动作，只返回一个JSX描述的结构，最终由React来操作渲染过程。

注意：render函数是一个纯函数，根据this.state和this.props来决定返回的结果，不产生任何副作用。

**4.componentWillMount和componentDidMount**

在装载过程中，componentWillMount会在render函数调用之前被调用，componentDidMount会在render函数调用之后被调用，相当于render函数的前哨和后卫。

componentWillMount的存在主要目的是为了和componentDidMount对称。

注意：render函数被调用完后，componentDidMount函数并不会立即被调用，componentDidMount被调用时，render返回的东西已经发生了渲染，组件已经被"转载"在DOM树上。

render函数本省并不往DOM树上渲染和装载内容，只返回一个JSX表示的对象，由React库根据对象决定如何渲染。

componentWillMount和componentDidMount的区别：

componentWillMount可以在服务端和浏览器端被调用；

componentDidMount只能在浏览器端被调用，在服务端使用React时不会被调用；

React和其他UI库如何配合使用？

利用componentDidMount函数，componentDidMount函数在执行时，React对应的组件的DOM已经存在了，所有的事件处理函数都已经设置好了，这是可以调用其他UI库的代码了。

### 3.2 更新过程

更新过程：

1. componentWillReceiveProps
2. shouldComponentUpdate
3. componentWillUpdate
4. render
5. componentDidUpdate

但是，并不是所有的更新过程都会执行全部的函数。

**1.componentWillReceiveProps(nextProps)**

父组件的render函数的render函数被调用时，在render函数里面被渲染的子组件就会经历更新过程，不管父组件传给子组件的props有没有改变，都会触发子组件的componentWillReceiveProps函数。

注意：this.setSate方法触发的更新过程不会调用componentWillReceiveProps函数。

**2.shouldComponentUpdate(nextProps，nextState)**

shouldComponentUpdate是除render函数在React生命周期函数最重要的一个函数。

render函数重要，是因为render函数决定了改渲染什么；

shouldComponentUpdate函数重要，是因为它决定了一个组件什么时候不需要渲染；

render和shouldComponentUpdate函数是React生命周期函数中唯二两个要求有返回结果的函数。render函数返回的结果用于构造DOM对象，shouldComponentUpdate函数的函数是个布尔值，告诉React库这个组件这次更新过程是否需要继续。

在更新过程中，React库先调用shouldComponentUpdate函数，此函数返回true，就继续更新过程接着调用render函数；此函数返回false,就停止此次的更新过程，也不会调用render函数。

**3.componentWillUpdate和componentDidUpdate**

componentWillMount和componentDidMount**，componentWillUpdate和componentDidUpdate，这两对函数把render函数加载中间。

和装载过程不同是，componentDidUpdate函数在更新过程，无论发生在服务端和浏览器端都会被调用。

React组件在更新时，原有的内容被绘制，需要在componentDidUpdate函数调用其他UI库的代码。

### 3.3 卸载过程

卸载过程：

1. componentWillUnmount

**1.componentWillUnmount**

componentWillUnmount函数适合做一些清理性的工作。

和装载过程和更新过程不同的是，此函数没有与之匹配配的Did函数，卸载完了，没有“卸载完在座的事情”。

componentWillUnmount的工作和componentDidMount有关，在componentDidMount中创造了一些非React的方法创造DOM元素，如果放任不管的话，可能造成内穿泄露的危险，需要在componentWillUnmount函数中把这些DOM元素清理掉。

## 4.组件向外传递数据

组件向外传递数据：

利用prop,组件的prop可以是任何JavaScript对象，在JavaScript中，函数是一等公民，函数本身可以看做一种对象，既可以像其他对象一样作为prop的值，从父组件传递到子组件，又可以被子组件作为函数调用。

## 5.React组件的state和prop的局限

React组件的state和prop的局限：

1. 数据发生了重复；
2. 多级组件结构，顶层组件传递数据底层组件，用prop的方式，只能通过多级组件多次中转。

