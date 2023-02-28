# 学习React(2) 添加交互性

## 添加处理事件

添加处理事件是最常见的添加交互性的方式,React中能够允许我们将事件处理程序加入到JSX中.事件是我们自己定义的,通过点击、悬停、聚焦等动作来触发.

添加处理事件的第一步是定义事件,我们可以在组件中定义一个函数来作为一个事件.并将它添加到适当的JSX中

```jsx
function App () {
  function handleClick () {
    console.log('haha')
  }
  return (
  	<button onClick={handleClick}>click</button>
  )
}
```

当我们添加的事件需要参数时,为了避免渲染的时候因为写了括号而导致的直接调用,我们可以在外层写上一个箭头函数,在这个箭头函数里进行我们想要的事件调用.

```jsx
function App () {
  function handleClick (name) {
		console.log(name)
  }
  return (
  	<button onClick={() => {handleClick('Lihua')}}>click</button>
  )
}
```

### 事件传播

事件处理程序可能捕获组件可能拥有子事件,当我们触发一个事件时,它会从事件触发的地方开始向上冒泡.

```jsx
export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('You clicked on the toolbar!');
    }}>
      <button onClick={() => alert('Playing!')}>
        Play Movie
      </button>
      <button onClick={() => alert('Uploading!')}>
        Upload Image
      </button>
    </div>
  );
}
```

当我们点击按钮时,会先执行button的点击事件,然后再执行div的执行事件.如果我们想让它停止传播,需要调用==e.stopPropagation==.

```jsx
export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('You clicked on the toolbar!');
    }}>
      <button onClick={(e) => {
          e.stopPropagation()
          alert('Playing!')
        }}>
        Play Movie
      </button>
      <button onClick={() => alert('Uploading!')}>
        Upload Image
      </button>
    </div>
  );
}
```

### 防止默认行为

一些浏览器事件具有自己默认行为,最常见的就是form标签的提交事件,当我们按下提交按钮时,触发form提交事件,然后将重新加载整个页面.

```jsx
export default function App () {
  return (
  	<form onSubmit={() => alert('ok')}>
      // ...
    </form>
  )
}
```

我们可以通过调用==e .preventDefault==来阻止默认行为的产生

```jsx
export default function App () {
  return (
  	<form onSubmit={(e) => {
        e.preventDefault()
      	alert('ok')
      }}>
      // ...
    </form>
  )
}
```

## state:组件的内存

### 猜想

我们通常需要组件能够记住一些状态和数据,以便于我们进行组件交互,最一般的想法是添加一个变量:

```jsx
export default function App () {
  let state = 0;
  return (
    <>
    	<div>state: {state}</div>
    	<button onClick={() => state++}>click</button>
    </>
  )
}
```

我们在运行时会发现,当按钮点击时,数据并没有发生变化.其实导致这种结果主要是两点原因:

* 本地变量在第二次渲染时,还是会把state初始化为0,并不会记住之前作出的任何更改
* 对本地变量的更改不会触发重新渲染

### 添加状态变量

由上点得知,如果想要获得我们想要的结果,需要有一个东西能够记住状态的修改,并且在作出修改时需要触发组件的重新渲染.

我们可以通过==useState==来添加状态变量:

```jsx
import { useState } from 'react'
export default function App () {
  const [count, setCount] = useState(0)
  return (
    <>
    	<div>state: {count}</div>
    	<button onClick={() => setCount(count + 1)}>click</button>
    </>
  )
}
```

其中count是状态变量,setCount是一个setter函数.

### state的特性

我们可以在组件中定义任意个state,那么React怎么知道要返回哪个状态?其实在内部，React为每个组件保存一个状态对数组。它还维护当前对索引，该索引在渲染前设置为`0`。每次您调用`useState`时，React都会为您提供下一个状态对并增加索引.

state是独立且私有的,如果渲染同一个组件多次,每个副本的state都是完全隔离的.

### state其实是一种快照

state可能看起来像是一个普通的js变量,但如果你重新设置它,产生的结果不是修改这个变量,而是会触发重新渲染.实际上state更像是一个快照,我们改变state时只有在下次组件重新渲染才能拿到一张新的快照,在本次渲染中==state是不会发生变化的==

举一个简单的例子

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

试一下点击按钮后会发生什么,结果可能跟你想的大相径庭: 1! 而不是3.

这就是上面所说的:当改变state的时候,只有下次渲染时state才会发生改变.可以详细的说明一下调用onClick发生的事情:

1.setNumber(number + 1),此时number为0

2.setNumber(number + 1),此时number还是为0

3.setNumber(number + 1),此时number仍然为0

你也可以将本次渲染时的state看作一张快照,即在本次渲染时number始终是为0的.

对于下次渲染, 就是进行

```jsx
<button onClick={() => {
  setNumber(1 + 1);
  setNumber(1 + 1);
  setNumber(1 + 1);
}}>+3</button>
```

在上面的基础上,再加入一个计时器呢?

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setTimeout(() => {
          alert(number);
        }, 3000);
      }}>+5</button>
    </>
  )
}
```

我们可以看到,alert中的内容还是为0.

本次click事件等同于

```jsx
setNumber(0 + 5);
setTimeout(() => {
  alert(0);
}, 3000);
```

**状态变量的值在渲染中永远不会改变，**即使其事件处理程序的代码是异步的.

### 将一系列的状态更新排入队列

**设置状态变量将本次设置的内容排入队列等待下次渲染**,但有时我们希望在下次渲染之前就能拿到最新的内容进行操作.比如上面对number的三次加一操作,我们希望能更新计数器三次

这时候我们可以使用更新器函数, 类似 n => n + 1. 当我们将它传给状态设置器时,React队列会在所有其它代码运行后处理这个函数,在下一次的渲染中,React会穿过队列,并拿到最终的更新状态

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```

这时候我们点击按钮后可以看到结果变成了3

以下是React在执行事件处理程序时如何通过这些代码行工作：

1. `setNumber(n => n + 1)`: `n => n + 1`是一个函数。React将其添加到队列中。
2. `setNumber(n => n + 1)`: `n => n + 1`是一个函数。React将其添加到队列中。
3. `setNumber(n => n + 1)`: `n => n + 1`是一个函数。React将其添加到队列中。

下一次渲染时,就能穿过队列拿到最新的状态: 3.

如果代码改成这样呢

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
        setNumber(42);
      }}>Increase the number</button>
    </>
  )
}
```

这时候结果就变成了42.

以下是React在执行此事件处理程序时如何通过这些代码行工作：

1. `setNumber(number + 5)`：`number`是`0`，所以`setNumber(0 + 5)`React在其队列中添加了*“替换为`5`*”。
2. `setNumber(n => n + 1)`: `n => n + 1`是一个更新函数。React将*该函数*添加到其队列中。
3. `setNumber(42)`：React在其队列中添加了*“替换为`42`*”.
4. 

## React渲染

React渲染主要分为三个步骤:

* 触发渲染

  触发渲染主要有两个原因:

  * 组件初始渲染

    ```jsx
    import App from './App.jsx'
    import { createRoot } from 'react-dom/client';
    
    const root = createRoot(document.getElementById('root'))
    root.render(<App />);
    ```

    

  * 组件状态更新导致的渲染

* 渲染组件

  在初始渲染时,React为给要渲染的组件内的标签创建DOM节点.如果是重新渲染,React会计算与上次渲染对比下产生的变化,在提交阶段前,它还不会对这些变化信息做任何事

* 提交DOM

  在初始渲染时,React会使用appendChild()创建新的DOM节点,对于重新渲染,React将会使用代价最小的必要操作,让DOM匹配最新的渲染输出

