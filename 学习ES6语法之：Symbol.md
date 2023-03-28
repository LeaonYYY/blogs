# 学习ES6语法之：Symbol

2023-3-27

## 前述

Symbol是ES6引入的一种新的原始数据类型，它的作用是用来代表一个独一无二的值。举个例子，我们使用一个具有很多属性的对象，当我们想要给这个对象添加新的属性的时候，如果直接添加的话，就有覆盖原有属性的可能。其实symbol的引入的主要原因就是为了提供一个能够给出独一无二属性名的功能。另外Symbol还有一个功能就是定义匿名属性，因为使用Symbol作为属性的时候，这个属性不会被`for in `和`for of` 遍历到。

## 定义

定义symbol我们需要通过`Symbol()`函数进行生成

```JS
let a = Symbol();
console.log(typeof a); // "symbol"
```

注意，`Symbol()`函数前不能使用`new`命令，否则会报错。这是因为生成的 Symbol 是一个原始类型的值，不是对象，所以不能使用`new`命令来调用。另外，由于 Symbol 值不是对象，所以也不能添加属性。基本上，它是一种类似于字符串的数据类型

我们在生成symbol的时候可以在`Symbol`函数中传入一个字符串用来给这个symbol进行描述，作用就是给写代码和看代码的人更容易区分不同的symbol。这个字符串被存储在`Symbol.prototype.description`中

```js
let a = Symbol('aaa');
let b = Symbol('bbb');

a.description // 'aaa'

a.toString(); // "Symbol(aaa)"
b.toString(); // "Symbol(bbb)"
```

前面又说到，symbol的作用是用来代表一个独一无二的值，这个字符串描述只是一个描述，当两个Symbol传入一模一样的描述的情况下，这两个Symbol也不会相等的

```js
let a = Symbol('aaa');
let b = Symbol('bbb');

a === b // false
```

还有一个需要注意的是，Symbol不能与其他类型的变量进行计算，但Symbol可以==显式==地转为字符串,当Symbol被转为布尔值时， 其值为true

```js
let a = Symbol('aaa');
'1' + a // TypeError: can't convert symbol to string
`${a}` // TypeError: can't convert symbol to string
```

## 基础用法

### 作为对象的属性名

由于每个Symbol值都是不等的，只要Symbol值作为对象的属性名，那么就能保证不会出现同名的属性，这对于一个对象由多个模块构成的情况非常有用，能防止某一个键被不小心改写或覆盖。

````js
let mySymbol = Symbol();

// 第一种写法
let a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
let a = {
  [mySymbol]: 'Hello!'
};

// 第三种写法
let a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello world!' });

// 以上写法都得到同样结果
a[mySymbol] // "Hello!"
````

==Symbol 值作为对象属性名时，不能用点运算符==,因为点运算符后面总是跟着字符串。

## 方法

### Object.getOwnPropertySymbols()

前面说到symbol在对象中作为属性名的时候不会被遍历到，相当于一个匿名属性，但不是私有属性，因为可以由`Object.getOwnPropertySymbols`方法来拿到这个对象的所有属性名

```js
const obj = {};
let a = Symbol('a');
let b = Symbol('b');

obj[a] = 'Hello';
obj[b] = 'World';

const objectSymbols = Object.getOwnPropertySymbols(obj);

objectSymbols
// [Symbol(a), Symbol(b)]
```



### Symbol.for()，Symbol.keyFor()

有时，我们希望重新使用同一个 Symbol 值，`Symbol.for()`方法可以做到这一点。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建一个以该字符串为名称的 Symbol 值，并将其注册到全局

```js
let s1 = Symbol.for('foo');
let s2 = Symbol.for('foo');

s1 === s2 // true
```

而`Symbol.keyFor()`方法返回一个==已登记==的 Symbol 类型值的`key`。这边登记的意思是用`Symbol.for()`进行Symbol注册

```js
let s1 = Symbol.for("foo"); // 有注册
Symbol.keyFor(s1) // "foo"

let s2 = Symbol("foo"); // 没注册
Symbol.keyFor(s2) // undefined 
```

