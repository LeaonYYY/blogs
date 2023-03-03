# 将Effect和事件处理程序分开

2023-3-3

我们在编写组件的时候总是按惯性思维来使用useEffect这个hook,并在其中混入了过多的本该是由事件处理程序进行的逻辑.这可能导致组件进行了很多不必要的重新渲染,更严重的会导致组件会出现意想不到的bug,而且这个bug是很难去发现并修复的.

## 如何在事件处理程序和Effect之间进行选择

首先我们得先清楚这二者是因为什么才被触发运行的:

当用户进行特定的交互,例如点击了某个按钮,这时候就应该由事件处理程序来处理这个特定的交互;而Effect只有当组件需要同步的时候,这个Effect才会被运行.看起来就像是事件处理程序是需要手动触发的,而Effect是被自动触发的.

一个更重要的选择依据是逻辑是否应该是响应式的.事件处理程序内部的逻辑不是响应式的,除非用户再次执行相同的交互,否则它不会被再次运行.而Effect内部的逻辑是响应式的,如果它所依赖的某个或者某些反应值(props、state或者在组件中定义的变量)发生改变时,那么Effect就会销毁上一步Effect逻辑的内容(执行useEffect回调中返回的销毁函数),接着就会重新进行use Effect中的回调.

## 从Effect中提取出属于事件处理程序的逻辑

例如,当用户连接到聊天时，您想要显示通知。您从道具中读取当前主题（深色或浅色），以便以正确的颜色显示通知

```js
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId, theme]); 
  // ...
```

这样写是不会报错的,但是在用户体验上是非常糟糕的.因为当theme改变的时候,这个聊天室还是会重新连接,而用户并不希望改变一下主题自己就会掉线.

### ⚠️⚠️⚠️ 实验性API: useEffectEvent

这时候我们就可以用一个名为useEffectEvent的特殊钩子来提取出非响应式的逻辑

因为这个钩子还在建设中,我们需要这样写来使用它:

```js
import { experimental_useEffectEvent as useEffectEvent } from 'react';
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); 
  // ...
```

这个钩子传入的回调函数可以拿到最新的==响应值==
