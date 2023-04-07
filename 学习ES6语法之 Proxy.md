# 学习ES6语法之: Proxy

2023-4-7

Proxy主要用于修改对象的一些默认操作,比如获取数据、修改数据之类的.可以理解成它给目标对象和外界之间加了一个拦截器,外界想要对这个对象做出一些操作的时候,就必须通过这层拦截.需要注意的是,Proxy的作用是生成一个目标对象代理,而不是直接改变目标对象的性质,外界需要对生成的代理进行操作拦截器才会生效.如果外界直接操作目标对象,拦截器就不会起作用.

## 基础使用

ES6提供了Proxy构造函数来生成proxy对象,这个构造函数的第一个参数是要代理的目标对象,第二个参数是拦截器的配置,当这个配置中没有写任何拦截,外界操作这个代理对象和操作原目标对象是一样的.

```js
const target = {};// 目标对象
const handler = {}; // 拦截的配置
const proxy = new Proxy(target, handler)
```

举一个简单的例子,我们给handler配置一个拦截get的方法,无论外界访问代理对象的任何属性,都返回一样的结果

```js
const target = {};
const handler = {
  /**
  * @param target 目标对象
  * @param propKey 属性名
  **/
  get(target, propKey) {
    return 'hello'
  }
};
const proxy = new Proxy(target, handler);

proxy.a // 'hello'
proxy.b // 'hello'
proxy.c // 'hello'
```

生成的proxy对象还可以作为其它对象的原型对象

```js
const son = Object.create(proxy)
son.a // hello
```

## 拦截器操作配置

拦截器中的方法名是给定的,下面列出Proxy支持的所有拦截操作

对这些操作的详细解释,详见 [Proxy实例的方法](https://es6.ruanyifeng.com/#docs/proxy#Proxy-%E5%AE%9E%E4%BE%8B%E7%9A%84%E6%96%B9%E6%B3%95)

- **get(target, propKey, receiver)**：拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`。
- **set(target, propKey, value, receiver)**：拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值。
- **has(target, propKey)**：拦截`propKey in proxy`的操作，返回一个布尔值。
- **deleteProperty(target, propKey)**：拦截`delete proxy[propKey]`的操作，返回一个布尔值。
- **ownKeys(target)**：拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`、`for...in`循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而`Object.keys()`的返回结果仅包括目标对象自身的可遍历属性。
- **getOwnPropertyDescriptor(target, propKey)**：拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。
- **defineProperty(target, propKey, propDesc)**：拦截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。
- **preventExtensions(target)**：拦截`Object.preventExtensions(proxy)`，返回一个布尔值。
- **getPrototypeOf(target)**：拦截`Object.getPrototypeOf(proxy)`，返回一个对象。
- **isExtensible(target)**：拦截`Object.isExtensible(proxy)`，返回一个布尔值。
- **setPrototypeOf(target, proto)**：拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
- **apply(target, object, args)**：拦截 Proxy 实例作为函数调用的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。
- **construct(target, args)**：拦截 Proxy 实例作为构造函数调用的操作，比如`new proxy(...args)`。

