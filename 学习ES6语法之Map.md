# 学习ES6语法之:Map

2023-3-31

## 前述

我们在开发中经常需要使用到hash结构的数据结构,大部分情况下我们都是使用对象(object)来满足我们的需求,但对象的键只能是字符串或者Symbol,实际上这个hash结构是`键-值`对,这个对它的使用带来了很大的限制.为此,ES6提供了Map数据结构,它也是属于hash结构的数据结构类型,但它的键不限于字符串,Map的结构更像是`值-值`对,相比对象来说更加完善了

## 定义

类似于`Set`,我们用`Map`构造函数来创建Map实例,

```js
let map = new Map()
```

构造函数可以接受一个子项为键值对数组的数组为Map初始值.

```js
const items = [
  ['name', '张三'],
  ['title', 'Author']
];

let map = new Map(items)

// 实际上等效于
const items = [
  ['name', '张三'],
  ['title', 'Author']
];

const map = new Map();

items.forEach(
  ([key, value]) => map.set(key, value)
);
```

不仅仅是数组，任何具有 Iterator 接口、且每个成员都是一个双元素的数组的数据结构都可以当作`Map`构造函数的参数

```js
const set = new Set([
  ['foo', 1],
  ['bar', 2]
]);
const m1 = new Map(set);
m1.get('foo') // 1

const m2 = new Map([['baz', 3]]);
const m3 = new Map(m2);
m3.get('baz') // 3
```

## 方法

###  操作

* `size`: Map结构的成员总数
* `set(key, value)`:`set`方法设置键名`key`对应的键值为`value`，然后返回整个 Map 结构。如果`key`已经有值，则键值会被更新，否则就新生成该键。
* `get(key)`:get`方法读取`key`对应的键值，如果找不到`key`，返回`undefined
* `has(key)`: `has`方法返回一个布尔值，表示某个键是否在当前 Map 对象之中。
* `delete(key)`: `delete`方法删除某个键，返回`true`。如果删除失败，返回`false`。
* `clear()`: 清除所有成员

### 遍历

- `Map.prototype.keys()`：返回键名的遍历器。

- `Map.prototype.values()`：返回键值的遍历器。

- `Map.prototype.entries()`：返回所有成员的遍历器。

- `Map.prototype.forEach()`：遍历 Map 的所有成员。

  ```js
  const map = new Map([
    ['F', 'no'],
    ['T',  'yes'],
  ]);
  
  for (let key of map.keys()) {
    console.log(key);
  }
  // "F"
  // "T"
  
  for (let value of map.values()) {
    console.log(value);
  }
  // "no"
  // "yes"
  
  for (let item of map.entries()) {
    console.log(item[0], item[1]);
  }
  // "F" "no"
  // "T" "yes"
  
  // 或者
  for (let [key, value] of map.entries()) {
    console.log(key, value);
  }
  // "F" "no"
  // "T" "yes"
  
  // 等同于使用map.entries() 
  // map[Symbol.iterator] === map.entries  // true
  for (let [key, value] of map) {
    console.log(key, value);
  }
  // "F" "no"
  // "T" "yes"
  ```

## WeakMap

 WeakMap与Map功能相似,不同的地方在于WeakMap的键只能是对象(null除外),且与WeakSet类似,WeakMap对这个对像的引用也是弱引用.

如果你要往对象上添加数据，又不想干扰垃圾回收机制，就可以使用 WeakMap。一个典型应用场景是，在网页的 DOM 元素上添加数据，就可以使用`WeakMap`结构。当该 DOM 元素被清除，其所对应的`WeakMap`记录就会自动被移除。

```js
const wm = new WeakMap();

const element = document.getElementById('example');

wm.set(element, 'some information');
wm.get(element) // "some information"
```

==WeakMap 弱引用的只是键名，而不是键值。键值依然是正常引用。==

WeakMap也没有遍历操作,同时没有size属性.



## WeakRef

[详见](https://es6.ruanyifeng.com/#docs/set-map#WeakRef)

WeakSet 和 WeakMap 是基于弱引用的数据结构，[ES2021](https://github.com/tc39/proposal-weakrefs) 更进一步，提供了 WeakRef 对象，用于直接创建对象的弱引用。

```js
let target = {};
let wr = new WeakRef(target);
```

WeakRef 实例对象有一个`deref()`方法，如果原始对象存在，该方法返回原始对象；如果原始对象已经被垃圾回收机制清除，该方法返回`undefined`。

```js
let target = {};
let wr = new WeakRef(target);

let obj = wr.deref();
if (obj) { // target 未被垃圾回收机制清除
  // ...
}
```

