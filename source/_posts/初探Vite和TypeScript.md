---
title: 初探Vite和TypeScript
date: 2023/8/21
tags: 疑难杂症
categories: vue-next-admin
top_img: ./img/41.jpg
cover: ./img/41.png

---

## 初探Vite和TypeScript

TypeScript和Vite都与前端开发有关。TypeScript是一种编程语言，它是JavaScript的超集，添加了静态类型和其他功能。Vite是一个现代的前端构建工具和开发服务器，它专注于快速的开发启动和热模块替换。Vite通常与TypeScript一起使用，以提供更好的类型检查和开发体验。所以，你可以使用Vite来构建和开发TypeScript项目。



### 一，关于.vue文件的引入

比如在Trace.vue中引入 BlockInfo.vue

```vue
<script setup lang="ts">

const BlockInfo = defineAsyncComponent(()=>import('/@/views/visualizing/BlockInfo.vue'));
    
</script>
```



### 二，结合react定义对象

```react
const state = reactive<BlockState>({
	tableData: {
		data: [],
		total: 0,
		loading: false,
		param: {
			pageNum: 1,
			pageSize: 10,
		},
	},
});
```

这段代码使用了Vue 3的Composition API中的`reactive`函数来创建一个响应式对象`state`。让我逐步解释：

1. `const state = reactive<BlockState>({})`：这行代码定义了一个名为`state`的常量，并使用`reactive`函数来将一个空对象转化为响应式对象。`<BlockState>`是一个类型参数，它指定了对象的类型，可能是一个接口或类型别名。

2. 在这个响应式对象中，有一个属性`tableData`，它的值是一个包含以下属性的对象：

   - `data: []`：这是一个空数组，可能用于存储表格的数据。
   - `total: 0`：这是一个表示总数据量的数字，初始值为0。
   - `loading: false`：这是一个表示数据是否正在加载的布尔值，初始值为`false`。
   - `param`：这是一个对象，包含以下属性：
     - `pageNum: 1`：表示当前页数，初始值为1。
     - `pageSize: 10`：表示每页显示的数据条数，初始值为10。

通过将整个`tableData`对象嵌套在`state`对象中，可以确保`tableData`及其内部属性的任何更改都会触发响应式更新。这意味着，如果在代码的其他地方修改了`state.tableData.data`、`state.tableData.total`等属性，相关的界面元素将自动更新以反映这些更改，而无需手动操作DOM。

总之，这段代码利用Vue 3的`reactive`函数创建了一个包含响应式数据的状态对象，用于管理某个特定功能的数据和状态。

```typescript
declare interface BlockState {
	tableData: BlockTableType;
}

interface BlockTableType extends TableType {
	data: RowFabricType[];
}

// block
declare type RowBlockType<T = any> = {
	queryBlockByNumDrawer: boolean;
	channelBlockInfo: string;
	blockInfo: string;
	previousBlockInfo: string;
	previousBlockInfo2: string;
	open: boolean,
	blockInfoHash: string;
	option: {
	  data: T[];
	};
	traceId: string;
	retailerInfo: string;
	productData: string;
	activeName: string;
	cropsProcessDetailsArray: string;
	cropsDriverArray:string;
	machingInfo: string;
	operationArray:[];
	cropsDetails: string;
};
```



### 三，ref()的用法

```
const fabricDialogRef = ref();
```

这段代码使用了Vue 3的Composition API中的`ref`函数：

1. `const fabricDialogRef = ref();`：这行代码定义了一个名为`fabricDialogRef`的常量，并使用`ref`函数将其初始化为一个响应式引用。响应式引用是Vue 3中管理响应式数据的一种方式。

2. `fabricDialogRef`是一个可以存储任意数据的引用，它的值可以是基本类型、对象、数组等等。通过将数据包装在`ref`函数中，Vue 3会使这个数据变成响应式的，这意味着当数据发生改变时，相关的界面元素会自动更新以反映这些改变，无需手动操作DOM。

一般情况下，你会将需要响应式处理的数据（如状态、变量等）存储在`ref`中，然后在组件的模板或逻辑中使用它。在你的代码中，`fabricDialogRef`可以用来存储与某个对话框或组件相关的数据，以便在界面上动态展示或修改它。例如，你可以使用`fabricDialogRef.value`来访问存储在引用中的数据，并根据需要更新它。



#### 进行弹窗：

```typescript
// 打开新增用户弹窗
const onOpenAddUser = (type: string) => {
	fabricDialogRef.value.openDialog(type);
};
```

