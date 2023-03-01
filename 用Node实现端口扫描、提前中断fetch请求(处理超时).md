# 用Node实现端口扫描、提前中断fetch请求(处理超时)

2023-3-1

给项目添加了一个小功能,大致记录一下.

功能需求是用node实现一个端口扫描器,找出当前主机被占用的所有端口,并且拿这些端口去请求一个特定的路径

## 端口扫描

大致的思路是逐个监听每个端口,获取端口的状态并记录下来.首先是先实现这个单端口监听检测功能,我们可以使用net包下的createServer功能进行监听,如果监听成功了,那么意味着这个端口没有被其他服务占用,如果监听失败,并且错误信息的code对应的是==EADDRINUSE==,那么这个端口就是被占用的,然后我们可以把它记录下来.

```typescript
/**
 * 检测端口是否被占用
 * @param port 端口号
 * @returns 端口是否被占用 true or false
 */
const isPortOccupied = (port: number) => {
  const net = require('net');
  let listener = net.createServer().listen(port);
  return new Promise((resolve, reject) => {
      // 如果监听成功，表示端口没有被其他服务占用，端口可用，取消监听，返回结果false
      listener.on("listening", () => {
        listener.close();
         resolve(false);
      })
      // 如果监听出错，并且错误原因是端口正在使用中, 返回结果true, 其它情况返回false
      listener.on("error", (err: any) => {
        listener.close();
        if(err.code === 'EADDRINUSE'){
            resolve(true);
        }else{
            resolve(false);
        }
      })
  })
}
```

最后加上循环和记录,就实现了本功能

## 提前中断fetch请求

从前面提到的功能拿到了被占用的端口数组,但在运行的时候程序却卡住了,经过调试才发现原来是在请求其中某个端口的时候请求严重超时,而fetch却无动于衷.于是我就手动地给这个请求加上了计时器

```typescript
const url = 'www.xxx.com'
// 计时器
let timeoutPromise = (timeout = 3000) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject('time is up');
    }, timeout);
  });
}
try {
  const res = await Promise.race([timeoutPromise(1000), fetch(url)]);
} catch (e) {}
```

这个计时器原理就是运用Promise的race方法,当fetch请求超过计时器的时间,Promise就会走计时器的回调从而跳出程序.

但这样写仍然是存在问题的,虽然我们的程序能够正确地运行,但实际上这个fetch请求还没有被关闭,仍然还在后台进行请求.如果这个请求一直被挂着不被目标处理,就会影响工程性能.

为了解决这个问题,我们可以使用AbortController来手动关闭这个fetch.当fetch超时的时候,我们在计时器的回调中用这个控制发出信号, 这个信号是提前绑在fetch上的,当fetch收到信号就能够被提前中断了.

```typescript
const url = 'www.xxx.com'

let controller = new AbortController();
let signal = controller.signal;

// 计时器
let timeoutPromise = (timeout = 3000) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject('time is up')
      controller.abort();
    }, timeout);
  });
}
try {
  const res = await Promise.race([timeoutPromise(1000), fetch(url, { signal: signal })]);
} catch (e) {}
```

