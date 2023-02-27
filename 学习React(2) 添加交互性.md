# 学习React(2) 添加交互性

2023-2-26

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

