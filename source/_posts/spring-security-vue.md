---
title: 「Spring Security」前后端分离菜单权限控制-前端动态路由
date: 2021-12-27 17:35:00
tags: [后端开发, SpringSecurity, 安全认证, Vue]
categories: 安全认证
---

前端部分，这里基于[vue-element-admin](https://panjiachen.github.io/vue-element-admin-site/zh/)模板来演示，
[vue-element-admin](https://panjiachen.github.io/vue-element-admin-site/zh/)是一个后台前端解决方案，它基于[vue](https://github.com/vuejs/vue)和[element-ui](https://github.com/ElemeFE/element)实现。<!-- more -->

### 1. 安装 vue-element-admin 
``` bash
# 克隆项目
git clone https://github.com/PanJiaChen/vue-element-admin.git

# 进入项目目录
cd vue-element-admin

# 安装依赖， 建议不要用 cnpm 安装 会有各种诡异的bug 可以通过如下操作解决 npm 下载速度慢的问题
npm install --registry=https://registry.npm.taobao.org

# 本地开发 启动项目
npm run dev
```

### 2. 改造前端路由挂载方式
[vue-element-admin](https://panjiachen.github.io/vue-element-admin-site/zh/)中权限的实现方式是：通过获取当前用户的权限去比对路由表，生成当前用户具有的权限可访问的路由表，通过`router.addRoutes`动态挂载到`router`上。

这里改造得更灵活一点，后台根据用户计算出可访问得菜单列表，直接返回用户可访问得菜单列表，前端也需要保存一份全的路由表，用户登录后得到可访问菜单，匹配前端保存的路由表然后动态挂载。

用户登录成功之后，在全局钩子`router.beforeEach`中拦截路由，判断是否已获得`token`，在获得`token`之后我们就要去获取用户的基本信息及可访问菜单，然后动态挂载路由。
``` javascript
/**
 * src/permission.js
 */ 
// router.beforeEach
const hasRoles = store.getters.roles && store.getters.roles.length > 0
if (hasRoles) {
next()
} else {
  // get user info
  const { menus } = await store.dispatch('user/getInfo')
  // generate accessible routes map based on menus
  const accessRoutes = await store.dispatch('permission/generateRoutes', menus)
  // dynamically add accessible routes
  router.addRoutes(accessRoutes)
  // ... other code
}
```

#### 2.1 根据接口返回的菜单列表`menus`动态挂载路由
接口返回菜单数据：

![](up-44f247866636a16715b39a0ed8941673c89.webp)

动态挂载路由：
``` javascript
/**
 * src\store\modules\permission.js
 */
import { constantRoutes, asyncRoutes, afterRoutes } from '@/router'

/**
 * 返回当前路由名称对应的菜单
 * @param menus 菜单列表
 * @param name  路由名称
 */
function filterMeun(menus, name) {
  if (name) {
    for (let i = 0; i < menus.length; i++) {
      const menu = menus[i]
      if (name === menu.name) {
        return menu
      }
    }
  }
  return null
}

/**
 * 通过后台请求的菜单列表递归过滤路由表
 * @param routes asyncRoutes
 * @param menus  接口返回的菜单
 */
export function filterAsyncRoutes(routes, menus) {
  const res = []
  routes.forEach(route => {
    const tmp = { ...route }
    const meun = filterMeun(menus, tmp.name)
    if (meun != null && meun.title) {
      tmp.hidden = meun.hidden !== 0
      // 显示的菜单替换后台设置的标题
      if (!tmp.hidden) {
        tmp.meta.title = meun.title
        tmp.sort = meun.sort
      }
      if (meun.icon) {
        tmp.meta.icon = meun.icon
      }
      if (tmp.children) {
        tmp.children = filterAsyncRoutes(tmp.children, menus)
      }
      res.push(tmp)
    }
  })
  return res
}

/**
 * 对菜单进行排序
 */
function sortRouters(accessedRouters) {
  for (let i = 0; i < accessedRouters.length; i++) {
    const router = accessedRouters[i]
    if (router.children && router.children.length > 0) {
      router.children.sort(compare('sort'))
    }
  }
  accessedRouters.sort(compare('sort'))
}

/**
 * 升序比较函数
 */
function compare(p) {
  return (m, n) => {
    const a = m[p]
    const b = n[p]
    return a - b
  }
}

const state = {
  routes: [],
  addRoutes: []
}

const mutations = {
  SET_ROUTES: (state, routes) => {
    state.addRoutes = routes
    state.routes = constantRoutes.concat(routes)
  }
}

const actions = {
  generateRoutes({ commit }, menus) {
    return new Promise(resolve => {
      // 通过后台请求的菜单列表递归过滤路由表
      const roleAsyncRoutes = filterAsyncRoutes(asyncRoutes, menus)
      // 对可访问菜单进行排序
      sortRouters(roleAsyncRoutes)
      // 拼接尾部公共菜单
      const accessedRoutes = roleAsyncRoutes.concat(afterRoutes)
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

#### 2.2 前端保存的全路径路由表
``` javascript
/**
 * src\router\index.js
 */
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)
import Layout from '@/layout'

/**
 * 没有权限要求的基本路由
 */
export const constantRoutes = [
  {
    path: '/redirect',
    component: Layout,
    hidden: true,
    children: [
      {
        path: '/redirect/:path(.*)',
        component: () => import('@/views/redirect/index')
      }
    ]
  },
  {
    path: '/login',
    component: () => import('@/views/login/index'),
    hidden: true
  },
  {
    path: '/404',
    component: () => import('@/views/error-page/404'),
    hidden: true
  },
  {
    path: '/',
    component: Layout,
    redirect: '/dashboard',
    children: [
      {
        path: 'dashboard',
        component: () => import('@/views/dashboard/index'),
        name: 'Dashboard',
        meta: { title: '后台首页', icon: 'dashboard', affix: true }
      }
    ]
  },
  {
    path: '/profile',
    component: Layout,
    redirect: '/profile/index',
    hidden: true,
    children: [
      {
        path: 'index',
        component: () => import('@/views/profile/index'),
        name: 'Profile',
        meta: { title: '个人中心', icon: 'user', noCache: true }
      }
    ]
  }
]

/**
 * 动态加载的路由
 */
export const asyncRoutes = [
    {
      path: '/sys',
      component: Layout,
      redirect: '/sys/menus',
      alwaysShow: true, // will always show the root menu
      name: 'SysSetting', // name必须和后台配置一致，不然匹配不到
      meta: { title: '系统设置', icon: 'el-icon-s-tools' },
      children: [
        {
          path: 'menus',
          component: () => import('@/views/sys/menus/index'),
          redirect: '/sys/menus/list',
          name: 'SysMenus',
          meta: { title: '菜单管理', icon: 'el-icon-menu' },
          children: [
            {
              path: 'list',
              hidden: true,
              component: () => import('@/views/sys/menus/list.vue'),
              name: 'SysMenuList',
              meta: { title: '菜单列表' }
            },
            {
              path: 'edit',
              hidden: true,
              component: () => import('@/views/sys/menus/form.vue'),
              name: 'SysMenuEdit',
              meta: { title: '编辑菜单' }
            },
            {
              path: 'add',
              hidden: true,
              component: () => import('@/views/sys/menus/form.vue'),
              name: 'SysMenuEdit',
              meta: { title: '添加菜单' }
            }
          ]
        },
        {
          path: 'roles',
          component: () => import('@/views/sys/roles/index'),
          redirect: '/sys/roles/list',
          name: 'SysRoles',
          meta: { title: '角色管理', icon: 'lock' },
          children: [
            {
              path: 'list',
              hidden: true,
              component: () => import('@/views/sys/roles/list.vue'),
              name: 'SysRoleList',
              meta: { title: '角色列表' }
            },
            {
              path: 'edit',
              hidden: true,
              component: () => import('@/views/sys/roles/form.vue'),
              name: 'SysRoleEdit',
              meta: { title: '编辑角色' }
            },
            {
              path: 'add',
              hidden: true,
              component: () => import('@/views/sys/roles/form.vue'),
              name: 'SysRoleEdit',
              meta: { title: '添加角色' }
            }
          ]
        },
        {
          path: 'users',
          component: () => import('@/views/sys/users/index'),
          redirect: '/sys/users/list',
          name: 'SysUsers',
          meta: { title: '用户管理', icon: 'user' },
          children: [
            {
              path: 'list',
              hidden: true,
              component: () => import('@/views/sys/users/list.vue'),
              name: 'SysUserList',
              meta: { title: '用户列表' }
            },
            {
              path: 'add',
              hidden: true,
              component: () => import('@/views/sys/users/form.vue'),
              name: 'SysUserEdit',
              meta: { title: '添加用户' }
            },
            {
              path: 'edit',
              hidden: true,
              component: () => import('@/views/sys/users/form.vue'),
              name: 'SysUserEdit',
              meta: { title: '编辑用户' }
            }
          ]
        },
        {
          path: 'icons',
          component: () => import('@/views/sys/icons/index'),
          name: 'SysIcons',
          meta: { title: '系统图标', icon: 'el-icon-picture', noCache: true }
        }
      ]
    },
  /** when your routing map is too long, you can split it into small modules **/
  // componentsRouter,
  // chartsRouter,
  // nestedRouter,
  // tableRouter,
]

/**
 * 没有权限要求的底部基本路由
 */
export const afterRoutes = [
  {
    path: 'external-link',
    component: Layout,
    children: [
      {
        path: 'https://www.test.com/',
        meta: { title: '友情链接', icon: 'link' }
      }
    ]
  },
  // 404 page must be placed at the end !!!
  { path: '*', redirect: '/404', hidden: true }
]

const createRouter = () =>
  new Router({
    // mode: 'history', // require service support
    scrollBehavior: () => ({ y: 0 }),
    routes: constantRoutes
  })

const router = createRouter()

// Detail see: https://github.com/vuejs/vue-router/issues/1234#issuecomment-357941465
export function resetRouter() {
  const newRouter = createRouter()
  router.matcher = newRouter.matcher // reset router
}

export default router
```


> 源码地址：[https://github.com/chaooo/spring-security-jwt.git](https://github.com/chaooo/spring-security-jwt.git),
> 这里我将本文的前后端分离后台菜单权限控制放在github源码tag的V4.0中，防止后续修改后代码对不上。
