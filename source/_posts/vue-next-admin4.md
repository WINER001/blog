---
title: vue-next-admin中实现一个有趣的部门组件
date: 2023/6/16 14:25
tags: vue-next-admin
categories: vue-next-admin
top_img: ./img/41.jpg
cover: ./img/86.png
---



# vue-next-admin中实现一个有趣的部门组件

![](./img/86.png)

### 1.先把表格中关于部门的这个组件的代码拿出来

```typescript
<el-col :xs="24" :sm="12" :md="12" :lg="12" :xl="12" class="mb20">
	<el-form-item label="部门">
		<el-cascader
			:options="state.deptData"
			:props="{ checkStrictly: true, value: 'deptName', label: 'deptName' }"
			placeholder="请选择部门"
			clearable
			class="w100"
			v-model="state.ruleForm.department"
		>
			<template #default="{ node, data }">
				<span>{{ data.deptName }}</span>
				<span v-if="!node.isLeaf"> ({{ data.children.length }}) </span>
			</template>
		</el-cascader>
	</el-form-item>
</el-col>
```

#### 可以看出el-cascader关联的是v-model="state.ruleForm.department"

#### 把state.ruleForm.department的代码也列出来

```typescript
const state = reactive({
	ruleForm: {
		userName: '', // 账户名称
		userNickname: '', // 用户昵称
		roleSign: '', // 关联角色
		department: [] as string[], // 部门
		phone: '', // 手机号
		email: '', // 邮箱
		sex: '', // 性别
		password: '', // 账户密码
		overdueTime: '', // 账户过期
		status: true, // 用户状态
		describe: '', // 用户描述
	},
	deptData: [] as DeptTreeType[], // 部门数据
	dialog: {
		isShowDialog: false,
		type: '',
		title: '',
		submitTxt: '',
	},
});
```



### 2.逐句分析

#### 这段代码是使用Vue.js框架的模板语法，用于生成一个表单项：

```
1. `<el-col :xs="24" :sm="12" :md="12" :lg="12" :xl="12" class="mb20">`

   这是一个`<el-col>`组件，它定义了一个栅格布局的列。通过`:xs="24" :sm="12" :md="12" :lg="12" :xl="12"`属性，指定了在不同屏幕大小下的列宽。`class="mb20"`为该列添加了一个名为"mb20"的CSS类，用于添加下边距。

2.  `<el-form-item label="部门">`

   这是一个`<el-form-item>`组件，用于包装表单项。`label="部门"`为该表单项添加了一个名为"部门"的标签。

3. `<el-cascader>`

   这是一个级联选择器组件，用于选择部门。在接下来的行中，我们将为该组件设置一些属性。

   

   :options="state.deptData"

   该行指定了一个名为"options"的动态属性，它的值来自Vue实例中的`state.deptData`。这个属性包含了级联选择器中显示的部门选项数据。

   

   :props="{ checkStrictly: true, value: 'deptName', label: 'deptName' }"

   这是`<el-cascader>`组件的另一个动态属性，用于配置组件的属性。`:props`指定了一个对象，其中包含了`checkStrictly`、`value`和`label`属性。`checkStrictly`属性被设置为`true`，`value`和`label`属性都被设置为`deptName`。这些属性配置了级联选择器中每个部门选项的值和显示文本。

    placeholder="请选择部门"

   这是`<el-cascader>`组件的静态属性，用于设置在选择器未选择任何部门时显示的占位文本。

    clearable

   这是`<el-cascader>`组件的静态属性，表示是否显示清空按钮，允许用户清除选择的部门。

    class="w100"

   这是为`<el-cascader>`组件添加了一个名为"w100"的CSS类，用于设置其宽度为100%。

    v-model="state.ruleForm.department"

   这是Vue的双向绑定语法，将`state.ruleForm.department`与`<el-cascader>`组件的值进行绑定。当用户选择一个部门时，`state.ruleForm.department`将被更新。

   

4.  `<template #default="{ node, data }">`

   这是一个Vue模板中的`<template>`元素，它定义了一个名为"default"的插槽。插槽的作用是自定义级联选择器中每个部门选项的显示内容和样式。

    `<span>{{ data.deptName }}</span>`

这是插槽中的一段代码，用于显示部门的名称。`data.deptName`是每个部门选项的名称。

    `<span v-if="!node.isLeaf"> ({{ data.children.length }}) </span>`

这也是插槽中的代码，用于在部门名称后显示该部门是否有子部门。`v-if="!node.isLeaf"`表示只有当当前部门不是叶子节点时才会显示这段代码。`data.children.length`表示该部门的子部门数量。
```

