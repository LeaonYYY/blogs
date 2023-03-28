# 学习ES6语法之：Set

## 前述

Set是ES6提供的新的数据结构。这个数据结构类似数组，但相较于数组，它的特点是它的成员的值都是唯一的。

## 定义

我们用`Set`构造函数来生成这个新的数据结构

```js
const set = new Set();
```

这边需要和Symbol的定义区别一下，很多人会把这两个的定义方式记错。这边用new生成了一个实例，而Symbol却不用。因为Symbol是原始数据类型，我们用Symbol使用`typeof `得到的是symbol，对set使用得到的是object。这就说明set实际上也算是一种需要实例化获得的对象。

```js
typeof Symbol() // 'symbol'
typeof new Set(); // 'object'
```

Set结构的实例拥有两个属性：`Set.prototype.constructor`和`Set.prototype.size`构造函数实际上也就是`Set`函数，第二个属性是该Set实例拥有的成员数

Set实例的方法分为操作方法和遍历方法。操作方法用来进行数据操作，遍历方法可以遍历Set结构的成员

## 方法

### 数据操作

* `Set.prototype.add(value)`：向Set实例添加某个值，返回Set实例本身

  ```js
  const set = new Set();
  set.add(1).add(2).add(2);
  
  s.size // 2 因为2只会被添加一次， set的成员的值是唯一的
  ```

  

* `Set.prototype.delete(value)`:删除某个值，返回一个布尔值，表示删除是否成功。

* `Set.prototype.has(value)`:返回一个布尔值，表示该值是否为`Set`的成员。

* `Set.prototype.clear()`:清除所有成员，没有返回值。

### 遍历

* `Set.prototype.keys()`返回键名的遍历器

* `set.prototype.values()`:返回键值的遍历器
* `Set.prototype.entries()`：返回键值对的遍历器
* `Set.prototype.forEach()`：使用回调函数遍历每个成员

```js
const set = new Set(['aaa', 'bbb', 'ccc'])
for (let item of set.keys()) {
  console.log(item);
}
// aaa
// bbb
// ccc

for (let item of set.values()) {
  console.log(item);
}
// aaa
// bbb
// ccc

for (let item of set.entries()) {
  console.log(item);
}
// ["aaa", "aaa"]
// ["bbb", "bbb"]
// ["ccc", "ccc"]
```

 Set 结构没有键名，只有键值（或者可以看成键名和键值是同一个值）

其实Set结构的实例是默认可遍历的，不需要我们显式调用values方法

```js
Set.prototype[Symbol.iterator] === Set.prototype.values
// true
```

我们还可以用扩展运算符来遍历Set

```js
const set = new Set([1, 2, 3])

const arr = [...set]
```

 转成数组后也可以用数组的方法来遍历，并可以使用数组的方法进行操作，最后再使用`Set`构造函数转回Set实例

因此使用Set可以很容易实现并集、交集、差集等操作

```js
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// 并集
let union = new Set([...a, ...b]);
// Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...a].filter(x => b.has(x)));
// set {2, 3}

// （a 相对于 b 的）差集
let difference = new Set([...a].filter(x => !b.has(x)));
// Set {1}
```

## WeakSet

WeakSet跟Set类似，不同点在于它的成员必须是对象。并且它对这些对象的引用是属于弱引用，这些对象随时可能会被js回收机制回收掉。因此WeakSet只有`add`、`delete`、`has`三个方法，不能对其成员进行遍历。
