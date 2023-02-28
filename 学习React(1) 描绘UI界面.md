# 学习React(1): 描绘UI界面

## 构建简单的组件

组件是React的核心概念之一,我们平时看到的页面都是可以拆分成一个个简单的组件的.组件的独立性和可重用性大大提高了开发和维护的效率.

### 定义组件

React组件实际上就是一个JavaScript函数(函数式组件),并返回要渲染的内容

```jsx
export default function HelloWorld () {
  return (
    <div>'Hello world!'</div>
  )
}
```

这边使用到了es6的模块化管理将==HelloWorld==导出为一个组件.

tips:

* React的组件的名称开头需要大写!!,否则它不会起作用

* 我们在这个函数返回了一个标签,这个标签可以类似HTML那样编写,但它实际上还是属于JS,这种语法被称为JSX,它允许我们在js语句中嵌入标签.

* 返回的语句可以写在一行上

  ```jsx
  return <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  ```

  但是如果return和标签不在同一行上,必须要用小括号把标签包起来,如果没有括号,return后面的东西都将被忽略掉

* 组件可以嵌套另外一个组件,但是绝不能嵌套他们的定义

  ```jsx
  // 下面这种写法是不被允许的!
  export default function Father () {
    // ...
    function Son () {
      // ...
    }
    // ...
  }
  ```

## 导入和导出组件

在上面提到,组件是有可重用性的,我们可以将很多组件都写在同一个文件上,但是当嵌套的层数过多或者我们需要在其他地方重用这些组件时,将它们拆分成不同的文件是非常有意义的.这时候需要导入和导出组件.

导入和导出组件有两种方式:

* 默认形式
* 命名形式

默认导出使用==default==关键字,一个文件不能有多个默认导出,但它可以有任意数量的命名导出

| 句法 | 导出声明                            | 导入声明                              |
| ---- | ----------------------------------- | ------------------------------------- |
| 默认 | export default function Button() {} | import Button from './button.js';     |
| 命名 | export function Button() {}         | import { Button } from './button.js'; |

## 使用JSX编写标签

多年来，Web开发人员将内容保存在HTML中，在CSS中进行设计，在JavaScript中保存逻辑——通常保存在单独的文件中！内容在HTML中标记，而页面的逻辑在JavaScript中单独存在.JSX并不是将HTML原封不动地放到JS中,相比之下,JSX比HTML更为严格.

tips:

* 如果要从组件返回多个元素,需要用一个父标签包裹起来,如果不想添加额外的div,可以用<></> 或者 Fragment标签

  之所以需要有一个标签将它们包括起来,是因为在引擎下它会被转换为普通的JavaScript对象。如果不将两个对象包装成数组，就无法从函数返回它们。这解释了为什么您也不能在不将两个JSX标签包装到另一个标签或片段中的情况下返回它们。

* 所有都标签都需要闭合,像<img>这种自闭标签需要写成<img/>,像<li>haha 这种包装标签需要写成<li>haha</li>

* 属性需要用驼峰格式(camelCase)书写

## 通过大括号在JSX中编写JavaScript

在JSX中使用大括号,可以在大括号中编写js逻辑,我们只能在jsx中两个地方使用:

* 标签内容

  ```jsx
  function App () {
    const name = 'Lihua'
    return (
    	<h1>{name}</h1>
    )
  }
  ```

  

* 标签属性

  ```jsx
  function App () {
    const path = 'www.baidu.com'
    return (
    	<a href={path}></a>
    )
  }
  ```

## 传递props给组件

React组件之间使用props进行通信,每个父组件都能通过props将一些信息传递给子组件,这些信息可以是字符串、数值、对象、函数等.

父组件通过指定子组件的属性向子组件传递数据,这些属性都会整合成一个props对象作为子组件函数的第一个参数,下面的写法是通过解构语法来使用props

```jsx
function Son ({name, age}) {
  return (
    <>
  		<div>name is {name}</div>
    	<div>age is {age}</div>
    </>
  )
}
export default function Father() {
  const name = 'Lihua'
  const age  = 18
  return (
  	<Son name={name} age={age}></Son>
  )
}
```

有时候父组件并非直接传递数据给子组件,而是要传给子组件的子组件,这种情况有时候中间的组件会很复杂,我们可以使用扩展运算符来进行props的转发

```jsx
function Grandson ({name, age, height, weight}) {
  // ...
}
function Son (props) {
  return (
    <Grandson {...props}></Grandson>
  )
}
export default function Father() {
  const name = 'Lihua'
  const age  = 18
  return (
  	<Son name={name} age={age}></Son>
  )
}
```

随着时间的推移,组件可能会接收到不同的props.所以它不总是静止的.但实际上props是不可变的,当组件需要改变其props时,需要父组件重新传递一个新的props,然后旧的props会背JS引擎回收.

需要注意的是: 不要尝试改变props,当我们需要改变时,往往要使用state而非props.

## 条件渲染

在JSX中嵌入js逻辑来实现条件渲染,比较常见的是使用if/else语句、&&、和三目运算符? :,当我们不想返回任何东西时,可以返回一个null.

在使用&&运算符时,左边不能放数值0,如果你放了0,那么你的页面会渲染出0而不是什么都不做.一个比较常见的错误就是:

```jsx
function App ({count}) {
  return (
  	<div>
    	{count && <div>hello</div>}
    </div>
  )
}
```

当count为0时, 组件会渲染出 0

## 渲染列表

通常需要显示数据收集中的多个类似组件。您可以使用JS[数组](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array#)方法来操作数据数组。在此页面上，您将使用[`filter()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)和[`map()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/map)与React过滤数据数组并将其转换为组件数组。

假设我们的需求是渲染出

```jsx
function App () {
	return (
  	<ul>
    	<li>1</li>
      <li>2</li>
      <li>3</li>
      <li>4</li>
    </ul>
  )
}
```

我们可以写成这样:

```jsx
function App () {
  const arr = [1, 2, 3, 4]
  return (
  	<ul>
    	{
        arr.map(item => <li>{item}</li>)
      }
    </ul>
  )
}
```

这样写完后仍然还存在瑕疵,因为我们没有给每个子项设定唯一的key.虽然这样写也能渲染出来,但是对于组件性能来说是大打折扣的.key告诉React每个组件对应的数组项，以便稍后匹配它们。如果您的数组项可以移动（例如由于排序）、插入或被删除，这一点就变得很重要.可以做一个贴切的比喻,如果你的电脑桌面图标都长一个样,并且都是同样的名字,当它们移动或改变(增加或删除)时,这时候想要正确辨识它们就很困难了.

