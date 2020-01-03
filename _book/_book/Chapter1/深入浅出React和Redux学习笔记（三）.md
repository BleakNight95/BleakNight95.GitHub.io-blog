# 深入浅出React和Redux学习笔记（三）

## 从Flux到Redux

Redux管理应用状态的框架：

1. 单向数据流的始祖Flux;
2. Flux理念的一个更强实现Redux;
3. 结合React和Redux;

## 1.Flux

Redux是Flux思想的另一种实现方式；

Flux(含Redux)贯彻的重要观点——单向数据流；

Flux推翻了MVC框架，用了新的思维来管理数据流转；

### 1.1MVC框架的缺点

MVC框架把应用分为三部分：

1. Model(模型)：负责管理数据，大部分业务逻辑在Model之中；
2. View(视图)：负责渲染用户界面，应避免在View中涉及业务逻辑；
3. Controller(控制器)：负责接收用户的输入，根据用户输入的Model部分逻辑，把产生的数据交给View部分，让View渲染必要的输出；

MVC框架的缺点：

对于非常巨大的代码库和庞大的组织，MVC框架很快变得很复杂。每新增一个功能时，对代码的修改很容易引入新的Bug,不同模块之间的依赖关系让系统变得“脆弱且不可预测”。

Flux的特点：

更严格的数据流控制；

Flux应用分为四个部分：

1. Dispatcher:处理动作分发，维持Store之间的依赖关系；

2. Store：负责存储数据和处理数据相关逻辑；

3. Action：负责驱动Dispatcher的JavaScrip对象；

4. View：视图部分，负责显示用户界面；

    ![img](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016011503.png) 

Flux和MVC结构上的区别：

1. Flux的Dispatcher相当于MVC的Controller；
2. Flux的Store相当于MVC的Model;
3. Flux的View相当于MVC的View;
4. Flux的Action可以理解为MVC框架的用户请求；

### 1.2Flux应用

一个EventEmitter实例对象支持的函数：

1. emit函数：可以广播一个特定事件，第一个参数是字符串类型的事件名称；
2. on函数:可以增加一个挂在此EventEmitter对象特定事件的处理函数，第一个参数是字符串类型的事件名称，第二个参数是处理函数；
3. removeListener函数：和on函数做的事情相反，删除一个挂在此EventEmitter对象特定事件的处理函数，和on函数一样的是，第一个参数是字符串类型的事件名称，第二个参数是处理函数。注意：如果要调用removeListener函数，就一定要保留对处理函数的引用；

如何使Dispatcher调用回调函数的顺序是我们预期的呢？

Dispatcher的waitFor函数。

Dispatcher的waitFor函数可以接受一个数组作为参数，数组中的每一个元素都是Dispatcherregister函数的返回结果，即dispatchToken。这个waitFor函数告诉Dispatcher，当前的处理必须暂停，直到dispatchToken代表的那些已注册回调函数执行结束才能继续。

存在于Flux框架中的React组件需要实现的功能：

1. 创建时读取Store的状态来初始化组件的状态；

2. 当Store上状态发生变化时，组件要立刻更新内部状态保存一致；

3. View如果要改变Store状态，必须而且只能派发Action;

   

### 1.3Flux的优势

Flux的架构下，应用的状态被放在Store中，React组件只扮演View的作用，被动根据Store的状态来渲染。

Flux的优势主要表现在“单向数据流”的管理方式。

在Flux的理念中，改变界面就必须改变Store的状态，改变Store的状态就必须派发一个对象。

MVC最大的缺点就是无法禁绝View和Model之间的直接对话。

在Flux的体系下，驱动界面的改变始于一个动作的派发，别无他法。

### 1.4Flux的不足

1. Store之间的依赖关系；
2. 难以进行服务端渲染；
3. Store混杂了逻辑和状态；

## 2.Redux

如果把Flux看做一个框架理念的话，Redux就是Flux的一种实现，除Redux之外，实现Flux的框架有Reflux、Fluxible等。

### 2.1Redux的基本原则

Redux的三个基本原则：

1. 唯一数据源（Single Source of Truth):应用的状态只存储在唯一的一个Store上；

2. 保持状态只读(State is only-read):不能直接去修改状态，要修改Store的状态，必须派发一个Action对象完成。

   驱动用户界面渲染，就要改变应用的状态，但不是去修改状态的值，而是创建一个新的状态对象给Redux，由Redux完成新的状态的组装。

3. 数据改变只能通过纯函数改变(Changes are made with pure functions);

   

### 2.2Redux实例

无

### 2.3容器组件和傻瓜组件

在Redux的框架下，一个React组件基本要完成两个功能：

1. 和Redux Store打交道，读取Store的状态，用于初始化组件状态，同时还要监听Store的状态改变；当Store状态发生改变时，需要更新组件状态，从而驱动组件重新渲染；当需要更新Store状态时，就要派发Action对象；
2. 根据当前props和state,渲染出用户界面；

让一个组件专注一件事，如果发现一个组件做的事情太多，就可以把这个组件拆分成多个组件，让每个组件依然只做一件事情。

容器组件(Container Component):承担第一个任务的组件，即负责和Redux  Store打交道的组件，处于外层；另一种说法叫做聪明组件(Smart Component)。容器组件做的事情涉及一些状态转换。

展示组件(Dumb Component):承担第二个任务的组件，即只专心负责渲染界面的组件，处于内层；另一种说法叫做傻瓜组件(Dumb Component)。傻瓜组件其实就是一个纯函数，根据props产生结果。

### 2.4组件Context

组件Context:就是所谓的“上下文环境”，让一个树状组件上的所有组件都能够访问的对象，为了完成这个任务，需要上级组件和下级组件配合。

首先，上级组件要宣称自己支持Context，并且提供一个函数来返回代表Context的对象。

其次，此上级组件之下所有的子孙组件，只要宣称自己需要这个Context，就可以通过this.context访问到这个共同的环境对象。

1. 上级组件提供Context；
2. 下级组件使用Context;

Redux对Store的封装好,没有提供直接修改状态的功能。虽然组件能够访问全局唯一的Store，但是不能直接Store中的状态，克服Store作为全局对象的缺点。

### 2.5React-Redux

改进React应用的两个方法：

1. 组件拆分为容器组件和傻瓜组件；
2. 提供组件直接访问的Context；

Redux两个最主要的功能：

1. connect:连接容器组件和傻瓜组件；
2. Provider:提供包含Store的Context;

**1.connect**

容器组件的工作：

1. 把Store的状态转化为内层傻瓜组件的prop；
2. 把傻瓜组件的用户动作转化为派发给Store的动作；

简言之，一是对内层傻瓜组件的输入，二是内层傻瓜组件的输出。

**2.Provider**

React-Redux要求Store所包含的三个函数的Object:

1. subscribe
2. dispatch
3. getState

拥有以上三个函数的对象，才能称之为Redux的Store。

React-Redux定义了Provider的componentWillReceiveProps函数，在React的生命周期中，componentWillReceiveProps函数每次渲染时都会被调用。

每个Redux应用只能有一个Redux Store ,在整个Redux的生命周期中都应该保持Store的唯一性。

