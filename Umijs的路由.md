# Umijs的路由

2023-3-6

## 路由配置

为了便于SPA开发，Umi将在配置文件中通过routes配置来实现路由配置。格式为路由信息的数组

```typescript
// .umirc.ts
export default {
    routes: [
        { path: '/', component: 'index' },
        { path: '/user', component: 'user' }
    ]
}
```

Umi默认按页拆包，这样能够加快页面的加载速度，但是这个加载过程是异步的，所以一般我们需要写loading.tsx来提高用户体验。

在这个配置对象中常用的有五个属性

* path 路由路径 

  ```
  支持的形式如下：
  /groups
  /groups/admin
  /users/:id
  /users/:id/messages
  /files/*
  /files/:id/*
  目前不支持的形式如下：
  /users/:id?
  /tweets/:id(\d+)
  /files/*/cat.jpg
  /files-*
  ```

* component 组件路径

  如果是相对路径，会从src/pages开始寻找。如果是src目录下的文件，还是推荐使用==@==

  例如： @/layouts/basic 就是 src/layouts/basic

* routes 子路由

* redirect 重定向

* wrappers 路由组件的包装组件，可以给路由组件添加进更多的功能：例如权限校验

    ```ts
    export default {
      routes: [
        { path: '/user', component: 'user',
          wrappers: [
            '@/wrappers/auth',
          ],
        },
        { path: '/login', component: 'login' },
      ]
    }
    ```

  ```tsx
  // src/wrappers/auth
  import { Navigate, Outlet } from 'umi'
   
  export default (props) => {
    const { isLogin } = useAuth();
    if (isLogin) {
      return <Outlet />;
    } else{
      return <Navigate to="/login" />;
    }
  }
  ```

  ## 约定式路由

  约定式路由也叫文件路由，就是不需要手写配置，文件系统即路由，通过目录和文件及其命名分析出路由配置。

  **如果没有routes配置，Umi会进入约定式路由模式**，之后分析==src/pages==目录进行路由配置

  例如

  ```bash
  .
    └── pages
      ├── index.tsx
      └── users.tsx
  ```

  会得到以下路由配置

  ```javascript
  [
    { path: '/', component: '@/pages/index' },
    { path: '/users', component: '@/pages/users' },
  ]
  ```

  ### 动态路由

  带有==$==前缀的目录或文件为动态路由。如果==$==后面不指定参数名，则代表==*==通配。

  例如

  ```bash
  + pages/
    + foo/
      - $slug.tsx
    + $bar/
      - $.tsx
    - index.tsx
  ```

  ```javascript
  [
    { path: '/', component: '@/pages/index.tsx' },
    { path: '/foo/:slug', component: '@/pages/foo/$slug.tsx' },
    { path: '/:bar/*', component: '@/pages/$bar/$.tsx' },
  ];
  ```

  

