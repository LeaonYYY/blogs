#  React：Reducer和执行上下文的使用

## Reducer

我们在编写组件的时候，大多数情况都是直接在组件中定义的函数完成逻辑的编写。但是当这个组件功能不断增长，这些逻辑就会显得非常冗长，导致这个组件看起来不是特别易懂，这对组件的维护与修改是十分不方便的。

我们可以使用Reducer将这些逻辑从组件中剥离出去，从原始的在组件中进行逻辑使用变成了类似在组件中派发事件给Reducer进行逻辑操作。

举个例子：

```JSX
import {useState} from 'react';

export default function TaskApp() {
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) {
    setTasks([
      ...tasks,
      {
        id: nextId++,
        text: text,
        done: false,
      },
    ]);
  }

  function handleChangeTask(task) {
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) {
          return task;
        } else {
          return t;
        }
      })
    );
  }

  function handleDeleteTask(taskId) {
    setTasks(tasks.filter((t) => t.id !== taskId));
  }
  // ...
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Visit Kafka Museum', done: true},
  {id: 1, text: 'Watch a puppet show', done: false},
  {id: 2, text: 'Lennon Wall pic', done: false},
];

```

首先我们将原本的对state的设置操作变成用dispatch派发事件出去，就像这样

```JSX
function handleAddTask(text) {
  dispatch({
    type: 'added',
    id: nextId++,
    text: text,
  });
}
```

那么这个事件派发给谁呢？ 这时候我们的reducer处理方法就需要出场了

它的格式如下

```JSX
function youReducer (state, action) {
    // 返回新的state状态给react
}
```

这个函数第一个参数是当前state的状态，当第一次渲染的时候，它由useReducer进行给定，第二个参数action就是我们dispatch的内容。在函数的最后我们要返回一个新的state状态给react。

前面的准备工作做完，我们就可以开始在组件中使用reducer 了

==useReducer==传入第一个参数就是reducer的处理函数，第二个参数是我们要操作的状态的初始状态。解构出的第一个变量就是类似state的状态变量，第二个变量就是我们用来派发事件的方法。

```JSX
import {useReducer} from 'react';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }
  //...
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Visit Kafka Museum', done: true},
  {id: 1, text: 'Watch a puppet show', done: false},
  {id: 2, text: 'Lennon Wall pic', done: false},
];

```

## 执行上下文

我们在进行组件间信息通信时，大部分都会使用props进行传参。假设一下我们有个父组件，需要把一些数据传给它的子组件的子组件（甚至更深）。这时候一层一层地往下传参，就会显得十分繁琐。使用执行上下文就能解决这个问题，我们在一个组件提供一个执行上下文数据，在它之下的任何地方都可以取到它所提供的数据（有点类似Vue中的provide和inject）

首先我们需要创建一个执行上下文

LevelContext.js

```jsx
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

这时候我们有了一个初始值为1的上下文数据。我们在需要使用这个数据地方这样写

```JSX
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js

export default function App () {
    const level = useContext(LevelContext);
    // ...
}
```

我们在这个地方就拿到了之前创建的上下文数据，这个数据是可以改变的，我们可以这样写，这个value就是改变后的上下文数据

```JSX
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

在被Provider包裹下的子组件的任何地方都可以使用useContext拿到这个上下文数据。
