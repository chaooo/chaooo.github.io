---
title: 「Spring Security」前后端分离权限控制-指令级权限
date: 2021-12-30 12:28:00
tags: [后端开发, SpringSecurity, 安全认证, Permissions]
categories: 安全认证
---


实现按钮级别的权限控制，基于上一篇[Spring Secuirty（六）前后端分离菜单权限控制-前端动态路由](https://my.oschina.net/chaoo/blog/5380267)的扩展。
前端部分还是基于[vue-element-admin](https://panjiachen.github.io/vue-element-admin-site/zh/)模板来演示。

这里实现按钮级别的权限判断的逻辑：每个按钮对应一个`权限标识`，后台根据用户角色计算出当前用户可访问的`权限标识`列表，前端登录后得到`权限标识`列表存入全局，通过单个按钮的`权限标识`去匹配列表里的。来实现按钮级别的权限判断。<!-- more -->

### 1. 数据库添加权限表
``` sql
-- 系统权限表
 DROP TABLE IF EXISTS `system_permission`;
 CREATE TABLE `system_permission` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `menu_id` BIGINT NOT NULL COMMENT '菜单ID',
  `name` VARCHAR(100) DEFAULT NULL COMMENT '权限标识',
  `title` VARCHAR(100) NOT NULL COMMENT '权限名称',
  PRIMARY KEY (`id`)
 ) ENGINE=INNODB DEFAULT CHARSET=UTF8MB4 COMMENT='系统权限表';

 -- 权限&角色 关联表
DROP TABLE IF EXISTS `system_role_permission`;
CREATE TABLE `system_role_permission` (
  `id` BIGINT NOT NULL AUTO_INCREMENT,
  `role_id` BIGINT DEFAULT NULL COMMENT '角色ID',
  `permission_id` BIGINT DEFAULT NULL COMMENT '权限ID',
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=UTF8MB4 COMMENT='系统角色权限关联表';
```


### 2. 前端代码改造
登录后通过接口直接返回用户可访问指令级权限列表：
![](up-584b8affd5e003dbb50e53f100779ae49c9.webp)

[vue-element-admin](https://panjiachen.github.io/vue-element-admin-site/zh/)模板已经封装了一个通过角色来判断的指令权限：[v-permission](https://github.com/PanJiaChen/vue-element-admin/tree/master/src/directive/permission)。

这里需要修改其逻辑：
#### 2.1 全局`state`中添加`permissions`列表

``` javascript
// ---------------- state 中添加 permissions
/**
 * src\store\modules\user.js
 */ 
// ... other code
const state = {
    token: getToken(),
    username: '',
    roles: [],
    menus: [],
    permissions: [],
    // ... other code
}
const mutations = {
    SET_TOKEN: (state, token) => {
    state.token = token
    },
    SET_PERMISSIONS: (state, permissions) => {
      state.permissions = permissions
    },
    // ... other code
}
const actions = {
  // user login
  login({ commit }, userInfo) {
    // ... other code
  },
  // get user info
  getInfo({ commit, state }) {
    return new Promise((resolve, reject) => {
      getInfo()
        .then(response => {
          const { data } = response
          if (!data) {
            reject('Verification failed, please Login again.')
          }
          const { roles, menus, permissions, id, username, avatar, roleDesc, fullName, phone } = data
          // roles must be a non-empty array
          if (!roles || roles.length <= 0) {
            reject('getInfo: roles must be a non-null array!')
          }
          commit('SET_ROLES', roles)
          commit('SET_MENUS', menus)
          commit('SET_PERMISSIONS', permissions)
          commit('SET_USERID', id)
          commit('SET_USERNAME', username)
          commit('SET_AVATAR', avatar)
          commit('SET_ROLEDESC', roleDesc)
          commit('SET_FULLNAME', fullName)
          commit('SET_PHONE', phone)
          resolve(data)
        })
        .catch(error => {
          reject(error)
        })
    })
  },

  // user logout
  logout({ commit, state, dispatch }) {
    // ... other code
  },
  // ... other code
}
export default {
  namespaced: true,
  state,
  mutations,
  actions
}

// ---------------- getters 中添加 permissions
/**
 * src\store\getters.js
 */
const getters = {
  // ... other code
  permissions: state => state.user.permissions,
  // ... other code
}
export default getters
```

#### 2.2 重写全局`v-permission`指令逻辑

``` javascript
/**
 * src\directive\permission\permission.js
 */
import store from '@/store'
function checkPermission(el, binding) {
  const { value } = binding
  if (value && value.length > 0) {
    const permissions = store.getters && store.getters.permissions
    const permissionButton = value

    const hasPermission = permissions.length > 0 && permissions.some(permission => {
      return permission === permissionButton
    })
    if (!hasPermission) {
      el.parentNode && el.parentNode.removeChild(el)
    }
  } else {
    console.error(`need permissions! Like v-permission="sys:menu:edit"`)
    return false
  }
}
export default {
  inserted(el, binding) {
    checkPermission(el, binding)
  },
  update(el, binding) {
    checkPermission(el, binding)
  }
}
```

#### 2.3 重新`checkPermission`权限判断函数逻辑
``` javascript
/**
 * src\utils\permission.js
 */
import store from '@/store'
/**
 * @param {String} value
 * @returns {Boolean}
 * @example see @/views/permission/directive.vue
 */
export default function checkPermission(value) {
  if (value && value.length > 0) {
    const permissions = store.getters && store.getters.permissions
    const permissionButton = value

    const hasPermission = permissions.length > 0 && permissions.some(permission => {
      return permission === permissionButton
    })
    return hasPermission
  } else {
    console.error(`need permissions! Like v-permission="sys:menu:edit"`)
    return false
  }
}
```


### 3. 使用指令级权限
使用`v-permission`指令
``` html
<template>
  <!-- menu delete permission can see this -->
  <el-tag v-permission="'sys:menu:del'">delete</el-tag>
  <!-- menu edit permission can see this -->
  <el-tag v-permission="'sys:menu:edit'">edit</el-tag>
</template>

<script>
import permission from '@/directive/permission/index.js'
export default{
  directives: { permission }
}
</script>
```

使用`checkPermission`函数
``` html
<template>
  <!-- menu delete permission can see this -->
  <el-tag v-if="checkPermission('sys:menu:del')">delete</el-tag>
  <!-- menu edit permission can see this -->
  <el-tag v-if="checkPermission('sys:menu:edit')">edit</el-tag>
</template>

<script>
import checkPermission from '@/utils/permission'
export default{
   methods: {
    checkPermission
   }
}
</script>
```


### 4. 添加按钮权限，与菜单/按钮对角色授权演示
菜单中添加按钮的权限演示
![](up-3304cdd327e2f4f01862531203ae17bfa62.webp)

菜单/按钮对角色授权演示
![](up-dde01457a309a26fc04f512830e18e71485.webp)

完整代码请查看源码：
> 源码地址：[https://github.com/chaooo/spring-security-jwt.git](https://github.com/chaooo/spring-security-jwt.git),
> 这里我将本文的前后端分离后台菜单权限控制放在github源码tag的V5.0中，防止后续修改后代码对不上。

### 附：当前实例完整SQL
``` sql
-- 系统用户表
DROP TABLE IF EXISTS `sys_user`;
CREATE TABLE `sys_user` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `username` varchar(255) NOT NULL COMMENT '用户名',
  `password` varchar(255) NOT NULL COMMENT '密码',
  `avatar` varchar(255) DEFAULT 'https://wpimg.wallstcn.com/f778738c-e4f8-4870-b634-56703b4acafe.gif' COMMENT '头像',
  `fullName` varchar(100) NOT NULL COMMENT '姓名',
  `phone` varchar(20) NOT NULL COMMENT '电话',
  `login_flag` char(1) NOT NULL DEFAULT '0' COMMENT '是否阻止登录：0否，其他是',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `del_flag` char(1) NOT NULL DEFAULT '0' COMMENT '删除标记：0未删，其他删除',
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='系统用户表';

-- 系统角色表
DROP TABLE IF EXISTS `sys_role`;
CREATE TABLE `sys_role` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '角色ID',
  `role_name` varchar(50) NOT NULL COMMENT '角色名称',
  `role_desc` varchar(255) DEFAULT NULL COMMENT '描述',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `del_flag` char(1) NOT NULL DEFAULT '0' COMMENT '删除标记：0未删，其他删除',
  PRIMARY KEY (`id`),
  UNIQUE KEY `role_name` (`role_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='系统角色表';

-- 系统菜单表
DROP TABLE IF EXISTS `sys_menu`;
CREATE TABLE `sys_menu` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '菜单ID',
  `title` varchar(100) NOT NULL COMMENT '菜单名称',
  `name` varchar(100) NOT NULL COMMENT '路由名称(前端匹配路由用)',
  `icon` varchar(50) DEFAULT NULL COMMENT '图标',
  `parent_id` bigint NOT NULL DEFAULT '0' COMMENT '父级菜单Id',
  `hidden` char(1) NOT NULL DEFAULT '0' COMMENT '隐藏状态：0显示，1隐藏',
  `status` char(1) NOT NULL DEFAULT '0' COMMENT '状态：0启用，1停用',
  `sort` int NOT NULL DEFAULT '0' COMMENT '排序',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `del_flag` char(1) NOT NULL DEFAULT '0' COMMENT '删除标记：0未删，其他删除',
  PRIMARY KEY (`id`),
  UNIQUE KEY `name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='系统菜单表';

-- 系统权限表
DROP TABLE IF EXISTS `sys_permission`;
CREATE TABLE `sys_permission` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `menu_id` bigint NOT NULL COMMENT '菜单ID',
  `name` varchar(100) DEFAULT NULL COMMENT '权限标识',
  `title` varchar(100) NOT NULL COMMENT '权限名称',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='系统权限表';

 -- 权限&角色 关联表
DROP TABLE IF EXISTS `sys_role_permission`;
CREATE TABLE `sys_role_permission` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `role_id` bigint DEFAULT NULL COMMENT '角色ID',
  `permission_id` bigint DEFAULT NULL COMMENT '权限ID',
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=UTF8MB4 COMMENT='系统角色权限关联表';

-- 用户&角色 关联表
DROP TABLE IF EXISTS `sys_role_user`;
CREATE TABLE `sys_role_user` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `role_id` bigint DEFAULT NULL COMMENT '角色ID',
  `user_id` bigint DEFAULT NULL COMMENT '用户ID',
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=UTF8MB4 COMMENT='系统用户角色关联表';

-- 菜单&角色 关联表
DROP TABLE IF EXISTS `sys_role_menu`;
CREATE TABLE `sys_role_menu` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `role_id` bigint DEFAULT NULL COMMENT '角色ID',
  `menu_id` bigint DEFAULT NULL COMMENT '菜单ID',
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=UTF8MB4 COMMENT='系统角色菜单关联表';

-- 初始数据
INSERT  INTO `sys_user`(`id`,`username`,`password`,`create_time`) VALUES (1,'sysadmin','$2a$10$Ml/uEJ5BnUSdKspYM4vkneUHM0piXyAVU0aueWya/v7FBauz6XWE6',NOW());
INSERT  INTO `sys_role`(`id`,`role_name`,`role_desc`,`create_time`) VALUES (1,'admin','管理员',NOW());
INSERT INTO `sys_menu` (`id`, `title`, `name`, `icon`, `parent_id`, `hidden`, `create_time`)
VALUES  ('1','系统设置','SysSetting','el-icon-s-tools','0','0',NOW()),
        ('2','菜单管理','SysMenus','el-icon-menu','1','0',NOW()),
        ('3','角色管理','SysRoles','peoples','1','0',NOW()),
        ('4','用户管理','SysUsers','user','1','0',NOW()),
        ('5','系统图标','SysIcons','el-icon-picture','1','0',NOW()),
        ('6','菜单列表','SysMenuList','','2','1',NOW()),
        ('7','菜单编辑','SysMenuEdit','','2','1',NOW()),
        ('8','角色列表','SysRoleList','','3','1',NOW()),
        ('9','角色编辑','SysRoleEdit','','3','1',NOW()),
        ('10','用户列表','SysUserList','','4','1',NOW()),
        ('11','用户编辑','SysUserEdit','','4','1',NOW()),
        ('12','其他菜单','Other','bug',0,'0',NOW());
INSERT  INTO `sys_permission`(`id`,`menu_id`,`name`,`title`) VALUES (1,6,'sys:menu:edit','编辑'),(2,6,'sys:menu:del','删除'),(3,8,'sys:role:edit','编辑'),(4,8,'sys:role:del','删除'),(5,10,'sys:user:edit','编辑'),(6,10,'sys:user:del','删除');
INSERT INTO `sys_role_user`(`user_id`, `role_id`) VALUES (1, 1);
INSERT INTO `sys_role_menu`(`role_id`, `menu_id`) VALUES (1, 1),(1, 2),(1, 3),(1, 4),(1, 5),(1, 6),(1, 7),(1, 8),(1, 9),(1, 10),(1, 11),(1, 12);
INSERT  INTO `sys_role_permission`(`id`,`role_id`,`permission_id`) VALUES (1,1,1),(2,1,2),(3,1,3),(4,1,4),(5,1,5),(6,1,6);
```
