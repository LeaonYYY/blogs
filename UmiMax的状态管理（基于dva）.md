# UmiMax的状态管理（基于dva）

普通的React组件通过在组件中维护state来进行状态管理，通过父组件传递props给子组件来实现数据传递。实际上页面的动态更改是通过state的变化来实现的。对于简单的组件来说，这些对于状态管理来说是绰绰有余的。但我们不妨设想一下，假如这个组件不断膨胀，组件中的状态也会越来越复杂，同时加上各式各样的数据传递，这就会导致数据发生紊乱，当我们改动其中某个数据的时候，很容易带来难以意料的结果。

## Umi内置Dva

为此，Umi内置了Dva提供了一套状态管理方案：

数据统一在 `src/models` 中的 model 管理，组件内尽可能的不去维护数据，而是通过 connect 去关联 model 中的数据。页面有操作的时候则触发一个 action 去请求后端接口以及修改 model 中的数据，将业务逻辑分离到一个环形的闭环中，使得数据在其中单向流动。

## 用法

### 配置dva

在配置文件中配置 ==dva：{}==打开Umi内置的dva组件

### 添加model

Umi会默认将src/models下的model定义自动挂载，你只需要在 model 文件夹中新建文件即可新增一个 model 用来管理组件状态。

model的定义分为如下几个部分

* **namespace**  model 的命名空间，唯一标识一个 model，如果与文件名相同可以省略不写

* **state**  model 中的数据

* **effects** 异步action 用来发送异步请求

  effects中要写Generator函数来实现异步

* **reducers**  同步action 用来发送同步请求（更改state）

model写法示例（来自官方文档）

```JS
import { queryUsers, queryUser } from '../../services/user';
 
export default {
  // namespace: 'user' //model的命名空间是全局的，需要保证这个是唯一的（默认为文件名）
  state: {
    user: {},
  },
 
  effects: {
    *queryUser({ payload }, { call, put }) {
      const { data } = yield call(queryUser, payload);
      yield put({ type: 'queryUserSuccess', payload: data });
    },
  },
 
  reducers: {
    queryUserSuccess(state, { payload }) {
      return {
        ...state,
        user: payload,
      };
    },
  },
 
  test(state) {
    console.log('test');
    return state;
  },
};
```

### connect

connect是用来将model和组件关联在一起，同时它会把相关的数据和dispatch添加到组件的props中。

例如

```jsx
import React, { Component } from 'react';
import { connect } from 'umi';
 
const mapModelToProps = allModels => {
  return {
    test: 'hello world',
    // props you want connect to Component
  };
};
 
@connect(mapModelToProps)
class UserInfo extends Component {
  render() {
    return <div>{this.props.test}</div>;
  }
}
 
export default UserInfo;
```

推荐通过注解的方式调用 connect，它等同于 `export default connect(mapModelToProps)(UserInfo);`

### dispatch

触发一个action到model中

例如

```JSX
render () {
  return <div onClick={() => {
   this.props.dispacth({
    type: 'modelnamespace/actionname',
    sometestdata: 'xxx',
    othertestata: {},
  }).then(() => {
    // it will return a promise
    // action success
  });
  }}>test</div>
}
```

