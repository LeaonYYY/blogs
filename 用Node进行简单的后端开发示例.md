# 用Node进行简单的后端开发示例

2023-3-25

之前写了篇博客简单的介绍了如何使用Node进行简单的后端开发，建议先看完那一篇博客再来看这篇博客。今天就来总结一下目前我写的项目中的是如何进行路由coding的。

由于目前毕设进度还在基础功能实现，所以路由中大部分的功能都是增删改查相关的操作。虽然比较简单，但用于介绍如何进行简单有效的路由编写还是很有用的。

我就用账单功能模块的路由来简单介绍

## 引入需要用到的模块

```js
const router = require('express').Router() // 路由配置
const jwt = require('../utils/jwt') // 身份验证（token）
const db = require('../db/dblook') // 数据库
const errs = require('../utils/errorType') // 自己封装的错误信息
```

中间的两个文件在前面介绍node后端开发的时候已经展示了，这边就只展示这个错误信息模块

```JS
module.exports = {
  BAD_REQUEST: 'Bad Request',
  UNAUTHORIZED: 'Unauthorized',
  NOT_FOUND: 'Not Found'
}
```

其实这个模块就只是用于需要抛出错误的时候能够快速的提示错误类型，避免写错错误信息。

## 数据库操作

```JS
// 管理员获取所有的账单
function getAllBills () {
  const sql = `
  select * from bills;
  `
  return new Promise((resolve, reject) => {
    db.query(sql, (err, result) => {
      if (err) reject(err)
      resolve(result)
    })
  })
}
```

由于数据库操作是异步操作，我们可以先用Promise封装返回数据库操作，然后在实际调用的时候使用async/await语法来进行数据库同步化操作。通俗的意思就是当前端发起请求后，路由配置中阻塞getAllbills这个方法，当数据库完成query后才继续进行同步逻辑。

## 配置路由

```JS
// 路由配置
// 获取所有账单
router.get('/bill/list', async (req, res, next) => {
  try {
    // 身份验证
    const user_info = jwt.verify(req)
    if (!user_info || user_info.user_role === 0) throw Error(errs.UNAUTHORIZED)
    // 操作数据库
    const result = await getAllBills()
    res.send({
      success: true,
      data: {
        list: [...result]
      }
    })
  } catch (e) { next(e) }
})
```

路由的配置方法在前面介绍node后端开发的博客也有提到。

解读一下上面这个路由配置的逻辑：首先使用try/catch语句来进行错误处理，这个next方法是express提供的进入下一个中间件的方法。下文会讲到错误处理。进入逻辑后首先进行身份验证，然后进行操作数据库，如果数据库出错，我们在上面数据库操作的时候有进行reject操作，这边的try语块会跳出进入catch块。如果没有错误，我们可以将数据库操作的结果返回给前端用户。

## 错误处理

当catch到错误的时候，我们使用next方法使路由进入到下一个中间件中，这个中间件专门用来处理错误的发生。

```JS
// error handler
app.use(function(err, req, res, next) {
  console.log(err)
  switch (err.message) {
    case 'Bad Request':
      res.send(400, '请求参数错误')
      break
    case 'Unauthorized':
      res.send(401, '您没有请求权限')
      break
    case 'Not Found':
      res.send(404, '请求路径错误')
      break
    default:
      res.send(500, '服务器出错')
      break
  }
})
```

express中间件相当于一个过滤器，介于用户发起请求和获得返回之间，express框架允许我们使用use方法进行中间件的添加。这个中间件实际上也就是一个处理函数。

## 完整示例

```js
const router = require('express').Router()
const jwt = require('../utils/jwt')
const db = require('../db/dblook')
const errs = require('../utils/errorType')
//  bills表
//  id int primary key auto_increment,
// 	bill_type varchar(10) comment '账单类型',
// 	rate varchar(10) comment '费用',
// 	adminId int references user(id),
// 	userId int references user(id),
// 	payee varchar(20) comment '收款方',
// 	mention varchar(100) comment '收款说明',
// 	updatetime varchar(30),
// 	status int comment '0: 未支付, 1: 已完成'

// 管理员获取所有的账单
function getAllBills () {
  const sql = `
  select * from bills;
  `
  return new Promise((resolve, reject) => {
    db.query(sql, (err, result) => {
      if (err) reject(err)
      resolve(result)
    })
  })
}
// 获取指定收款方的账单
function getBillsByUserId (userId) {
  const sql = `
  select * from bills
  where userId = ?
  `
  return new Promise((resolve, reject) => {
    db.query(sql, [userId], (err, result) => {
      if (err) reject(err)
      resolve(result)
    })
  })
}
// 添加账单
function addBill (formdata) {
  const {
    bill_type,
    rate,
    adminId,
    userId,
    payee,
    mention,
    updatetime,
    status
  } = formdata
  const sql = `
  insert into bills (bill_type, rate, adminId, userId, payee, mention, updatetime, status)
  values (?, ?, ?, ?, ?, ?, ?, ?);
  `
  return new Promise((resolve, reject) => {
    db.query(sql, [bill_type, rate, adminId, userId, payee, mention, updatetime, status], (err) => {
      if (err) reject(err)
      resolve()
    })
  })
}
// 调整账单
function updateBill (formdata) {
  const {
    id,
    bill_type,
    rate,
    adminId,
    userId,
    payee,
    mention,
    updatetime,
    status
  } = formdata
  const sql = `
  update bills
  set bill_type = ?, rate = ?, adminId = ?, userId = ?, payee = ?, mention = ?, updatetime = ?, status = ?
  where id = ?;
  `
  return new Promise((resolve, reject) => {
    db.query(sql, [bill_type, rate, adminId, userId, payee, mention, updatetime, status, id], (err) => {
      if (err) reject(err)
      resolve()
    })
  })
}
// 删除账单
function deleteBill (id) {
  const sql = `
  delete from bills
  where id = ?;
  `
  return new Promise((resolve, reject) => {
    db.query(sql, [id], (err) => {
      if (err) reject(err)
      resolve()
    })
  })
}

// 路由配置
// 获取所有账单
router.get('/bill/list', async (req, res, next) => {
  try {
    const user_info = jwt.verify(req)
    if (!user_info || user_info.user_role === 0) throw Error(errs.UNAUTHORIZED)
    const result = await getAllBills()
    res.send({
      success: true,
      data: {
        list: [...result]
      }
    })
  } catch (e) { next(e) }
})
// 获取指定用户的账单
router.get('/bill/list-stu', async (req, res, next) => {
  try {
    const user_info = jwt.verify(req)
    if (!user_info) throw Error(errs.UNAUTHORIZED)
    const result = await getBillsByUserId(user_info.id)
    res.send({
      success: true,
      data: {
        list: [...result]
      }
    })
  } catch (e) {next(e)}
})
// 添加账单
router.post('/bill/add', async (req, res, next) => {
  try {
    const user_info = jwt.verify(req)
    if (!user_info) throw Error(errs.UNAUTHORIZED)
    await addBill({...req.body, adminId: user_info.id})
    res.send({
      success: true,
      msg: '添加成功',
      showType: 3,
      data: null
    })
  } catch (e) { next(e) }
})
// 调整账单
router.post('/bill/update', async (req, res, next) => {
  try {
    const user_info = jwt.verify(req)
    if (!user_info) throw Error(errs.UNAUTHORIZED)
    await updateBill({ ...req.body, adminId: user_info.id })
    res.send({
      success: true,
      msg: '修改成功',
      showType: 3,
      data: null
    })
  } catch (e) { next(e) }
})
// 删除账单
router.post('/bill/delete', async (req, res, next) => {
  try {
    const user_info = jwt.verify(req)
    if (!user_info || user_info.user_role === 0) throw Error(errs.UNAUTHORIZED)
    await deleteBill(req.body.id)
    res.send({
      success: true,
      msg: '删除成功',
      showType: 3,
      data: null
    })
  } catch (e) {next(e)}
})

module.exports = router;
```

