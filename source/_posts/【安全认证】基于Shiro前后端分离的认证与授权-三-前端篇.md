---
title: 【安全认证】基于Shiro前后端分离的认证与授权(三.前端篇)
date: 2020-02-07 02:10:46
tags: [后端开发, 安全认证]
categories: 安全认证
---


前两篇我们整合了`SpringBoot+Shiro+JWT+Redis`实现了登录认证，接口权限控制，接下来将要实现前端Vue的动态路由控制。

### 1. 前端权限控制思路（Vue）
前端的权限控制，不同的权限对应着不同的路由，同时菜单也需根据不同的权限，异步生成。
先回顾下整体流程：<!-- more -->
![](http://cdn.chaooo.top/Java/auth-global.png)

- 登录: 提交账号和密码到服务端签发`token`，拿到`token`之后存入浏览器，再携带`token`(一般放在请求头中)再去获取用户的详细信息(包括用户权限等信息)。
- 权限验证：通过用户权限信息 构建 对应权限的路由，通过`router.addRoutes`动态挂载这些路由。

接下来将基于Vue开源后台模板[vue-admin-template](https://github.com/PanJiaChen/vue-admin-template)来演示具体流程，这里只演示重要代码，完整项目移步文章末尾获取源码。


### 2. 登录
1. 先准备基础的静态路由(`src/router/index.js`)

``` javaScript
export const constantRoutes = [
  // 登陆页面
  {
    path: '/login',
    component: () => import('@/views/login/index'),
    hidden: true
  },
  // 首页
  {
    path: '/',
    component: Layout,
    redirect: '/dashboard',
    children: [{
      path: 'dashboard',
      name: 'Dashboard',
      component: () => import('@/views/dashboard/index'),
      meta: { title: '首页', icon: 'dashboard' }
    }]
  },
  {
    path: '/404',
    component: () => import('@/views/404'),
    hidden: true
  }
]
```

2. 登录页面(src/views/login/index.vue)`click`事件触发登录操作

``` javaScript
this.$store.dispatch('user/login', this.loginForm).then(() => {
  this.$router.push({ path: '/' }); //登录成功之后重定向到首页
}).catch(err => {
  this.$message.error(err); //登录失败提示错误
});
```

3. 登录逻辑(src/store/modules/user.js)`action`:

``` javaScript
const actions = {
  // user login
  login({ commit }, userInfo) {
    const { account, password } = userInfo
    return new Promise((resolve, reject) => {
      // 这里的login调用api接口请求数据
      login({ account: account.trim(), password: password }).then(response => {
        const { data } = response
        commit('SET_TOKEN', data)
        setToken(data)
        resolve()
      }).catch(error => {
        reject(error)
      })
    })
  },
  // get user info
  getInfo({ commit, state }) {
    return new Promise((resolve, reject) => {
      // 这里的getInfo调用api接口请求数据
      getInfo(state.token).then(response => {
        const { data } = response
        if (!data) {
          reject('Verification failed, please Login again.')
        }
        const { nickname, avatar, roles, permissions } = data
        // 全局储存用户信息
        commit('SET_NAME', nickname)
        commit('SET_AVATAR', avatar)
        // 角色信息
        commit('SET_ROLES', roles)
        // 指令权限信息
        commit('SET_PERMISSIONS', permissions)
        resolve(data)
      }).catch(error => {
        reject(error)
      })
    })
  },
}
```

4. `/login`与`/getInfo`接口与返回的数据

``` javaScript
POST (localhost:8282/login)
请求参数: {"username":"admin","password":"123456"}
响应: {
    "code":0,
    "msg":"登录成功",
    "data":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJhZG1pbiIsInVpZCI6MSwiZXhwIjoxNTgwOTk4MTIzfQ.6jgqt_opjnosASlJ2oSIYZn1Sb2BQO-eUo_6OVTHv50"
}
// ------------getInfo--------------
GET (localhost:8282/user/info)
Headers: {"X-Token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJhZG1pbiIsInVpZCI6MSwiZXhwIjoxNTgwOTk4MTIzfQ.6jgqt_opjnosASlJ2oSIYZn1Sb2BQO-eUo_6OVTHv50"}
响应: {
    "code": 0,
    "msg": "获取成功",
    "data": {
        "account": "admin",
        "nickname": "超级管理员",
        "roles": ["admin"],
        "permissions": ["user:list","user:add","user:delete","user:update"]
    }
}
```

5. 获取用户信息(src/permission.js)
    + 用户登录成功之后，我们会在全局钩子`router.beforeEach`中拦截路由，判断是否已获得`token`，在获得`token`之后我们就要去获取用户的基本信息 并且根据用户角色动态挂载路由。

``` javaScript
const whiteList = ['/login'] // 白名单
router.beforeEach(async(to, from, next) => {
  // 判断是否已获得token
  const hasToken = getToken()
  if (hasToken) {
    if (to.path === '/login') {
      next({ path: '/' })
    } else {
      const hasRole = store.getters.role
      if (hasRole) {
        next()
      } else {
        try {
          // 获取用户角色 ['admin'] 或,['developer','editor']
          const { roles } = await store.dispatch('user/getInfo')
          // 动态根据 角色 算出其对应有权限的路由
          const accessRoutes = await store.dispatch('permission/generateRoutes', roles)
          // 动态挂载路由
          router.addRoutes(accessRoutes)
          // addRouter是让挂载的路由生效，但是挂载后'router.options.routes'并未刷新(应该是个bug)
          // 所以还需要手动将路由加入'router.options.routes'
          router.options.routes = constantRoutes.concat(accessRoutes)
          next()
        } catch (error) {
          await store.dispatch('user/resetToken')
          Message.error(error || 'Has Error')
          next(`/login?redirect=${to.path}`)
        }
      }
    }
  } else {
    /* has no token*/
    if (whiteList.indexOf(to.path) !== -1) {
      next()
    } else {
      next(`/login?redirect=${to.path}`)
    }
  }
})
```


### 3. 动态挂载路由
主要思路，前端会有一份包含所有路由的路由表。创建`Vue`实例时会先挂载登录等公共路由；当用户登录之后，通过`getInfo(token)`获取用户的角色(`roles`)，动态根据用户的`roles`算出其对应有权限的路由，再通过`router.addRoutes`动态挂载路由；使用`vuex`管理路由表，根据`vuex`中可访问的路由渲染菜单。但这些控制都只是页面级的，后端接口也需要做权限验证。
1. 改造一下路由表，添加异步路由列表，将角色添加到元数据`meta`中：

``` javaScript
// 动态路由
export const asyncRoutes = [
  {
    path: '/user',
    component: Layout,
    redirect: '/user/list',
    name: 'User',
    meta: { title: '用户管理', icon: 'example' },
    children: [
      {
        path: 'list',
        name: 'UserList',
        component: () => import('@/views/user/list'),
        meta: { title: '用户列表', icon: 'nested' }
      },
      {
        path: 'edit',
        name: 'UserEdit',
        component: () => import('@/views/user/form'),
        meta: { title: '添加用户', icon: 'form' }
      }
    ]
  },
  {
    path: '/admin',
    component: Layout,
    children: [
      {
        path: 'index',
        name: 'Form1',
        component: () => import('@/views/test/index'),
        meta: { title: '管理员角色测试', icon: 'form', roles: ['admin'] }
      }
    ]
  },
  {
    path: '/editor',
    component: Layout,
    children: [
      {
        path: 'index',
        name: 'Form2',
        component: () => import('@/views/test/index'),
        meta: { title: '编辑角色测试', icon: 'form', roles: ['editor'] }
      }
    ]
  },
  {
    path: '/form',
    component: Layout,
    children: [
      {
        path: 'index',
        name: 'Form3',
        component: () => import('@/views/test/index'),
        meta: { title: '用户角色测试', icon: 'form', roles: ['user'] }
      }
    ]
  },
  {
    path: '/nested',
    component: Layout,
    redirect: '/nested/menu3',
    name: 'Nested',
    meta: { title: '子菜单权限测试', icon: 'form' },
    children: [
      {
        path: 'menu1',
        component: () => import('@/views/test/index'),
        name: 'Menu1',
        meta: { title: '管理员可见', roles: ['admin'] }
      },
      {
        path: 'menu2',
        component: () => import('@/views/test/index'),
        name: 'Menu1',
        meta: { title: '编辑者可见', roles: ['editor'] }
      },
      {
        path: 'menu3',
        component: () => import('@/views/test/index'),
        name: 'Menu1',
        meta: { title: '普通用户可见', roles: ['user'] }
      }
    ]
  },
```


2. 根据前面获取用户信息的代码可发现，通过`store.dispatch('permission/generateRoutes',roles)`来获得有权限的路由，新建`src/store/modules/permission.js`如下：

``` javaScript
import { asyncRoutes, constantRoutes } from '@/router'
/** 判断用户是否拥有此路由的权限 */
function hasPermission(roles, route) {
  if (route.meta && route.meta.roles) {
    return roles.some(role => route.meta.roles.includes(role))
  } else {
    return true
  }
}
/** 递归组装路由表，返回符合用户角色权限的路由列表 */
export function filterAsyncRoutes(routes, roles) {
  const res = []
  routes.forEach(route => {
    const tmp = { ...route }
    if (hasPermission(roles, tmp)) {
      if (tmp.children) {
        // 递归调用
        tmp.children = filterAsyncRoutes(tmp.children, roles)
      }
      res.push(tmp)
    }
  })
  return res
}
const state = {
  routes: [], // 所有路由,包括静态路由和动态路由
  addRoutes: [] // 动态路由
}
const mutations = {
  SET_ROUTES: (state, routes) => {
    state.addRoutes = routes
    // 合并路由
    state.routes = constantRoutes.concat(routes)
  }
}
const actions = {
  // 生成动态路由
  generateRoutes({ commit }, roles) {
    return new Promise(resolve => {
      let accessedRoutes
      if (roles.includes('admin')) {
        // '超级管理员'拥有所有的路由，这样判断节省加载时间
        accessedRoutes = asyncRoutes || []
      } else {
        // 筛选出该角色有权限的路由
        accessedRoutes = filterAsyncRoutes(asyncRoutes, roles)
      }
      commit('SET_ROUTES', accessedRoutes)
      resolve(accessedRoutes)
    })
  }
}
export default {
  namespaced: true,
  state,
  mutations,
  actions
}
```


### 4. axios拦截器
通过`request`拦截器在每个请求头里面塞入`token`，好让后端对请求进行权限验证；代码位置:`src/utils/request.js`。

``` javaScript
import axios from 'axios'
import store from '@/store'
import { getToken } from '@/utils/auth'
// create an axios instance
const service = axios.create({
  baseURL: process.env.VUE_APP_BASE_API, // url = base url + request url
  withCredentials: false, // send cookies when cross-domain requests
  timeout: 5000 // request timeout
})
service.interceptors.request.use(
  config => {
    if (store.getters.token) {
      // 登陆后将token放入headers['X-Token']中
      config.headers['X-Token'] = getToken()
    }
    return config
  },
  error => {
    return Promise.reject(error)
  }
)
export default service
```

### 5. 指令权限
可以使用全局权限判断函数，实现按钮级别的权限判断。

``` html
<template>
  <el-button v-if="checkPermission('user:add')">添加</el-button>
  <el-button v-if="checkPermission('user:delete')">删除</el-button>
  <el-button v-if="checkPermission('user:update')">修改</el-button>
  <el-button v-if="checkPermission('user:list')">查看</el-button>
</template>
<script>
import checkPermission from '@/utils/permission' // 权限判断函数
export default{
   methods: {
    checkPermission(value) {
      return checkPermission(value)
    }
   }
}
</script>
```

`src/utils/permission.js`:
``` javaScript
import store from '@/store'
export default function checkPermission(value) {
  const permissions = store.getters.permissions
  return permissions.indexOf(value) > -1
}
```


### 6. 效果演示：
![](http://cdn.chaooo.top/Java/auth-admin.jpg)


> 注：搭建到这里的代码在`github`源码`tag`的`V3.0`中。
> 源码地址: [https://github.com/chaooo/springboot-vue-shiro.git](https://github.com/chaooo/springboot-vue-shiro.git)
