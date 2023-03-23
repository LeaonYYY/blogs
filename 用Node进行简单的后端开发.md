# 用Node进行简单的后端开发



**对于大部分前端开发者来说,从零开始去学一门后端开发语言是比较难受的.你可能会吐槽:我做前端做好好的,干嘛闲着没事去搞后端的东西??? 当然人各有志,目标是全栈开发的小伙伴还是大有人在的.我是因为毕设(要求前后端都要有)才被逼无奈去写后端的.开始的时候有学一点点java后端开发,当时看完JavaSE之后,接着看jdbc就想睡觉了.因此我也对毕设的后端开发一直抱着比较抵触的心理.但在之后学了一点node的皮毛后真真切切感受到了用node搭一个简易的后端服务器对一个前端小白来说是多么方便.**

**本文需要一丢丢的node基础,如果不熟悉node语法的小伙伴可以先去搜一下node基础看一下,没什么难度的.**

**闲话少说,我们直接进入正题**

## 配置node环境

https://nodejs.org/en/download

大家可以到上面的页面中根据自己的情况下载并安装node

下载完成后在命令行输入

```shell
node -v
```

能够查看到当前的node版本就说明已经下载成功了.

### nvm

现在更推荐使用nvm进行node版本管理,关于nvm的下载和环境配置在网上有很多教程,我就不过多赘述了.

#### 用法

```shell
nvm ls // 列出当前可用的node版本
nvm install [version] // 下载指定版本的node 例如: nvm install 16
nvm use [version] // 使用指定版本的node 例如 nvm use 16
```

==tips==:如果你的电脑是Mac并搭载M1芯片,用nvm下载node的时候有个坑,因为芯片架构不同,nvm下载路径为*:<u>https://[nodejs](https://so.csdn.net/so/search?q=nodejs&spm=1001.2101.3001.7020).org/dist/v12.22.12/node-v12.22.12-darwin-arm64.tar.xz</u>*,但是在压缩包官网中并没有此类的压缩包.所以我们在执行install命令前需要先执行M1兼容命令

```shell
arch -x86_64 zsh
```

## Express

express的作用其实跟node内置http模块的作用类似,它的作用就是用来构建web服务器的.实际上express就是基于node的http模块进一步封装出来的

### 基本使用

安装express: npm install npm

```javascript
const express = require('express');
// 创建express服务器
var app = express();


// 启动服务器
app.listen(3000, function () {
  console.log('服务器已启动在端口: 3000')
})
```

#### POST请求

post请求的参数从 ==req.body==中拿到

```js
const url = '#'
//req就是request, res就是response
app.post(url, (req, res) => {
  res.send(
    //...
		// 你可以发送任何你想要发送的内容
  )
})
```

#### GET请求

get请求的参数从==req.params==中拿到

```js
const url = '#'
app.get(url, (req, res) => {
  res.send(
    //...
		// 你可以发送任何你想要发送的内容
  )
})
```

req和res的api大家可以[express官网](https://www.expressjs.com.cn/)中查到

### 模块化路由

在实际开发中,我们总不能把所有接口都写在一个文件里吧,所以express支持将路由模块化

我们来举个例子假设我们在app.js中进行服务器搭建

之后我们再创建一个user.js, 在user.js中写上

```js
// user.js
const router = require('express').Router() // 创建路由

router.get('/', (req, res) => {
  res.send('hahahaha')
})

module.exports = router
```

然后我们在app.js挂载路由

```js
const userRouter = require('./user.js') // 路径大家自己按自己的情况选择
const express = require('express')

const app = express()

app.use(userRouter)

app.listen(3000, function () {
  console.log('服务器已启动在端口: 3000')
})
```

这样我们就完成了简单的express模块化路由

## 连接数据库

我使用的数据库是mysql,所以这里只介绍在node项目中连接mysql.

### 下载依赖模块

```shell
npm install mysql
```

### 连接

```js
const mysql = require('mysql')

const connection = mysql.createConnection({
	host: 'localhost', // IP地址
  user: 'root', // 数据库用户
  password: '123', // 用户密码
  database: 'test' // 使用的数据库
})
// 由于上面提到了路由模块化,为了方便各个路由的使用 我们把这个连接也可以写成一个模块
module.exports = connection
```

### 基本使用

假设我们这时候已经在同目录下写了上面的连接模块, 文件名为==dblook.js==

```js
const db = require('./dblook')

// 其中我们可以用?代替变量, 之后在执行的时候传入位置明确的变量数组
const sql = `
	select * from students
	where id = ?;
`
// 第一个参数为sql语句,第二个参数就是sql语句中所需的变量,根据?出现的顺序放进指定的变量,第三个函数就是执行完成后触发的callback
// 回调函数第一个参数为错误信息, 无错的时候为null; 第二个参数是sql执行的结果
db.query(sql, [1], (error, result) => {
  // ···
})
```

## 身份验证:JWT

前端在登陆的时候,往往会拿到一个token,然后在之后的请求中带上这个token,这个token是后端来判断请求者身份的令牌.

在express中我们可以用jsonwebtoken外接模块来轻松形成并验证token

### 下载依赖模块

```shell
npm install jsonwebtoken
```

### 基本使用

```js
const jwt = require('jsonwebtoken');

// 用于加密的密钥
const SECRET = 'niganma'

// 创建token
// payload是我们附加进token的信息,例如用户信息
module.exports.publish = (payload = {}, maxAge = 60 * 60 * 24) => {
  return jwt.sign(payload, SECRET, {
    expiresIn: maxAge
  })
}

// 验证token
module.exports.verify = (req) => {
  let token = req.headers.authorization;
  if (!token) {
    return null;
  }
  //格式：authorization:bearer token
  token = token.split(" ");
  token = token.length === 1 ? token[0] : token[1];
  try {
    let result = jwt.verify(token, SECRET);
    console.log("----->" + result);
    return result;
  } catch (err) {
    return null;
  }
}

```

