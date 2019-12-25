# 深入浅出React和Redux学习笔记（一）

## React新的前端思维方式

React的介绍：

1. 如何初始化一个React项目；

2. 如何创建一个React组件；

3. React的工作方式；

   

## 1.初始化一个React项目

### 1.1 create-react-app工具

create-react-app是通过npm发布的安装包，在安装完成Node.js和npm后，在命令行工具中安装create-react-app:

```markdown
$ npm install --global create-react-app
```

初始化命名为first_react_app的第一个React项目：

```
$ create-react-app first_react_app
```

打开此项目，并运行此项目：

```
$ cd first_react_app
$ npm start
```

编译完成后，浏览器会自动打开一个网页，指向本机地址：http://localhost:3000/.

## 2.增加一个新的React组件

React的首要思想是通过组件(component)来开发应用。

组件(component):能够完成某个特点功能的独立的、可重用的代码。

基于组件的应用开发是广泛使用的软件开发模式，采用分而治之的方法。

### 2.1 JSX

JSX:JavaScript的语法扩展(eXtension),在javascript中可以编写HTML代码。

### 2.2 JSX是进步还是倒退

HTML代表内容，CSS代表样式，JavaScript代表交互行为，但这三种语言在三种不同的文件中，实际上把不同的文件分开管理了，并不是逻辑上的“分而治之”。

React的组件可以把JavaScript、HTML和CSS的功能放在一个文件中，实现了真正的组件封装。

## 3.分解React应用

无

## 4.React的工作方式

React的理念：

归结为一个公式， **UI=render(data)**

公式的含义：用户看的界面(UI),应该是一个函数(render)的执行结果，只接受数据(data)作为参数。这个函数是一个纯函数。

纯函数：没有任何副作用，输出完全依赖于输入的函数；两次函数如果输入相同，得到的结果也会完成相同；最终的用户的界面，在render函数确认的情况下完全取决于输入的数据。

React实践的是“响应式编程”(Reactive Programming),也是React为什么叫React的原因。

**Virtual DOM**

React应用通过重复渲染来实现用户交互。

React利用Virtual DOM,每次只渲染最少的DOM元素。

Web前端优化的一个原则：**尽量减少DOM操作**

## 5.React工作方式的优点

React利用函数式编程的思维来解决用户界面渲染的问题，最大的优势是开发者的开发效率会大大提升，开发出来的代码可维护性和可阅读性也大大增强。

React强制所有组件按照数据驱动渲染的模式来工作，无论应用的规模多大，都能让程序处于可控范围内。







