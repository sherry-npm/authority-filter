用于管理端权限管理的工具包。

## 安装
```bash
npm install authority-filter
```
&nbsp;

## 功能简介
authority-filter 提供了三个维度的权限过滤方法。
分别为：getMenuList、getRouterList、getPageOperations
根据当前登录用户具有的权限，对后台管理系统分别进行：菜单过滤、路由过滤、页面操作权限过滤。
&nbsp;

## 快速上手
### 【step1】定义权限字典表
```javascript
// authDictionary.js
const dictionary = {
	// 101：角色id
	101: [
		{
			// path：当前角色，有权访问的路由
			path: '/demo1',
			/** operations：
			   * 当前角色，对当前页面具有的操作权限。
			   * 名称由开发者自定义。
			   * 不需要进行操作控制时，operations可不定义。
			   */
			operations: ['create', 'edit', 'delete']
		},
		{
			path: '/demo2',
			operations: ['create', 'edit']
		},
		{
			path: '/demo3',
			operations: ['create', 'edit', 'delete']
		},
		{
			path: '/demo3/edit',
			operations: ['create', 'edit']
		},
		{
			path: '/demo3/detail'
		}
	],
	102: [
		{
			path: '/demo1/edit',
			operations: ['edit', 'delete']
		},
		{
			path: '/demo2'
		}
	]
}

export default dictionary;

```
&nbsp;

### 【step2】定义菜单
```javascript
// allMenus.js
const menus = [
	{
		title: '菜单1',
		icon: 'clock',
		url: '/demo1'
	},
	{
		title: '菜单2',
		icon: 'gear',
		submenu: [
			{
				title: '菜单2.1',
				url: '/demo2'
			}
		]
	},
	{
		title: '菜单3',
		icon: 'clock',
		submenu: [
			{
				title: '菜单3.1',
				icon: 'clock',
				submenu: [
					{
						title: '菜单3.1.1',
						url: '/demo3'
					}
				]
			}
		]
	},
]

export default menus

```
####【说明】
对于菜单的定义，submenu、url两个字段，不可以使用自定义名称。
其他字段可自定义。
&nbsp;

### 【step3】定义路由
```javascript
// vue-router为例。allRouters.js
const pages_demo3_detail = () => import('pages/demo3/detail.vue')
const pages_demo3_edit = () => import('pages/demo3/edit.vue')
const pages_demo3_list = () => import('pages/demo3/list.vue')

const allRouters = [
	{
		path: '/demo3/detail',
		component: pages_demo3_detail
	},
	{
		path: '/demo3/edit',
		component: pages_demo3_edit
	},
	{
		path: '/demo3',
		component: pages_demo3_list
	}
]

export default allRouters

```
&nbsp;

### 【step4】根据用户权限，进行过滤
```javascript
// router.js
import $Auth from 'authority-filter'
import $authDic from '../model/authDictionary'
import $allMenus from '../model/allMenus'
import $allRouters from '../model/allRouters'

// 假定，从userInfo中拿到的roleId，值为101
const roleId = 101

// step1：【创建】传入权限字典 + 用户角色id
const auth = new $Auth($authDic, roleId)
// step2：全局存储 auth 对象。这里以vuex伪代码为例。（这步可选。我们建议这样做，如果需要进行页面操作权限的过滤，则必须这么做）
$store.commit('user/setAuth', auth)
// step3：【路由过滤】获取经过权限过滤后的路由
const routerList = auth.getRouterList([...$allRouters])
// step4：【菜单过滤】获取经过权限过滤后的菜单
const menuList = auth.getMenuList($allMenus)

```
&nbsp;

```javascript
<!-- list.vue（以vue项目为例。页面中使用）-->
<template lang="pug">
	.p-page
		.title 页面操作过滤demo
		.buttons
		 	// 'create'、'edit'、'delete'与权限字典表的operations定义字段，相对应
			el-button(v-if="operations.includes('create')") 创建
			el-button(v-if="operations.includes('edit')") 编辑
			el-button(v-if="operations.includes('delete')") 删除
</template>

<script>
import { mapState } from 'vuex'

export default {
	computed: {
		...mapState('user', [
			'auth'
		])
	},
	data() {
		return {
			operations: []
		}
	},
	mounted() {
		this.operations = this.auth.getPageOperations(this.$route.path)
	}
}
</script>

```
&nbsp;
