# Generator函数的语法

Generator函数是ES6提供的一种异步编程解决方案，我们可以把Generator函数看做成一个封装了很多状态的函数，执行Generator函数会返回一个遍历器对象，这个遍历器对象可以依次遍历Generator函数内部的每一个状态。

## 定义Generator函数

Generator在写法上还是类似于普通的函数只是有两个不同，一个是在function关键字和函数名之间有一个==*==，第二个是函数内部的yield，用来定义每一个状态。当Generator遍历器进行遍历时，这些用yield定义的状态值就会被读取

```js
function* hellofunction* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

let hw = helloWorldGenerator();
```

上面的例子中定义了一个名为helloWorldGenerator的遍历器函数，其中有两个yield表达式，所以这个函数一共有三个状态：hello,world,ending

## 遍历状态

我们可以调用调用这个遍历器对象的next方法使得指针移动到下一个状态。

```js
hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```

第一次调用，Generator 函数开始执行，直到遇到第一个`yield`表达式为止。`next`方法返回一个对象，它的`value`属性就是当前`yield`表达式的值`hello`，`done`属性的值`false`，表示遍历还没有结束。

第二次调用，Generator 函数从上次`yield`表达式停下的地方，一直执行到下一个`yield`表达式。`next`方法返回的对象的`value`属性就是当前`yield`表达式的值`world`，`done`属性的值`false`，表示遍历还没有结束。

第三次调用，Generator 函数从上次`yield`表达式停下的地方，一直执行到`return`语句（如果没有`return`语句，就执行到函数结束）。`next`方法返回的对象的`value`属性，就是紧跟在`return`语句后面的表达式的值（如果没有`return`语句，则`value`属性的值为`undefined`），`done`属性的值`true`，表示遍历已经结束。

第四次调用，此时 Generator 函数已经运行完毕，`next`方法返回对象的`value`属性为`undefined`，`done`属性为`true`。以后再调用`next`方法，返回的都是这个值。

总结一下，调用 Generator 函数，返回一个遍历器对象，代表 Generator 函数的内部指针。以后，每次调用遍历器对象的`next`方法，就会返回一个有着`value`和`done`两个属性的对象。`value`属性表示当前的内部状态的值，是`yield`表达式后面那个表达式的值；`done`属性是一个布尔值，表示是否遍历结束。

没有规定说星号要和function关键字还是函数名写在一起， 下面的四种写法都是可以的

```javascript
function * foo(x, y) { ··· }
function *foo(x, y) { ··· }
function* foo(x, y) { ··· }
function*foo(x, y) { ··· }
```