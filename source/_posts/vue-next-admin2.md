---
title: vue-next-admin管理系统添加页面
date: 2023/6/12 21:03
tags: vue-next-admin
categories: vue-next-admin
top_img: ./img/41.jpg
cover: ./img/28.jpg
---

# vue-next-admin添加一个页面



## 目标就是在系统设置这个下面添加一个区块链管理的页面

![](./img/79.png)



### 1.首先在vue-next-admin的src/views/system目录下新增一个文件夹fabric，包含两个文件——dialog.vue    index.vue



### 2.修改src/router目录下的route.ts文件新增如下

```typescript
{
						path: '/system/fabric',
						name: 'systemFabric',
						component: () => import('/@/views/system/fabric/index.vue'),
						meta: {
							title: 'message.router.systemFabric',
							isLink: '',
							isHide: false,
							isKeepAlive: true,
							isAffix: false,
							isIframe: false,
							roles: ['admin'],
							icon: 'iconfont icon-icon-',
						},
					},
```



### 3.修改src/i18n/lang目录下的zh-cn.ts文件新增一行systemFabric: '区块链管理',

```typescript
// 定义内容
export default {
	router: {
		home: '首页',
		system: '系统设置',
		systemMenu: '菜单管理',
		systemRole: '角色管理',
		systemUser: '用户管理',
		systemFabric: '区块链管理',
    }
}
```



### 4.新添加的src/views/system/fabric/dialog.vue 

这个文件是用来配置弹出对话（dialog），这里暂时先不用管



### 5.最后就是新添加的src/views/system/fabric/index.vue ,也是最重要的

##### 上面的图可以看到区块链管理页面最主要的就是一个查询框+一个表格，这两个都是在index.vue中的，这里直接提供代码

```vue
<template>
	<div class="system-user-container layout-padding">
		<el-card shadow="hover" class="layout-padding-auto">
			<div class="system-user-search mb15">
				<el-input size="default"  placeholder="请输入TID" v-model="tid" style="max-width: 180px"> </el-input>
				<el-button size="default" type="primary" class="ml10"  @click="queryWithTID">
					<el-icon>
						<ele-Search />
					</el-icon>
					查询
				</el-button>
				<el-button size="default" type="success" class="ml10" @click="onOpenAddUser('add')">
					<el-icon>
						<ele-FolderAdd />
					</el-icon>
					查询全部
				</el-button>
			</div>
			<el-table :data="state.tableData.data" v-loading="state.tableData.loading" style="width: 100%">
				<el-table-column prop="id" label="酒号" show-overflow-tooltip></el-table-column>
				<el-table-column prop="name" label="酒名" show-overflow-tooltip></el-table-column>
				<el-table-column prop="value" label="价格" show-overflow-tooltip></el-table-column>
				<el-table-column prop="owner" label="拥有者" show-overflow-tooltip></el-table-column>
				<el-table-column prop="birth" label="生产年份" show-overflow-tooltip></el-table-column>
				<el-table-column prop="capacity" label="容量" show-overflow-tooltip></el-table-column>
				<el-table-column prop="describe" label="用户描述" show-overflow-tooltip></el-table-column>
				<el-table-column label="操作" width="100">
					<template #default="scope">
						<el-button :disabled="scope.row.userName === 'admin'" size="small" text type="primary" @click="onOpenEditUser('edit', scope.row)"
							>修改</el-button
						>
						<el-button :disabled="scope.row.userName === 'admin'" size="small" text type="primary" @click="onRowDel(scope.row)">删除</el-button>
					</template>
				</el-table-column>
			</el-table>
			<el-pagination
				@size-change="onHandleSizeChange"
				@current-change="onHandleCurrentChange"
				class="mt15"
				:pager-count="5"
				:page-sizes="[10, 20, 30]"
				v-model:current-page="state.tableData.param.pageNum"
				background
				v-model:page-size="state.tableData.param.pageSize"
				layout="total, sizes, prev, pager, next, jumper"
				:total="state.tableData.total"
			>
			</el-pagination>
		</el-card>
		<FabricDialog ref="fabricDialogRef" @refresh="getTableData()" />
	</div>
</template>
```



### 这样就创建好了fabric页面，可以看到查询框绑定了queryWithTID方法

```vue
<el-button size="default" type="primary" class="ml10"  @click="queryWithTID">
```

### 而且，我们还希望这个fabric页面初次加载的时候就显示所有链上的酒，下一篇会介绍这两个方法的实现
