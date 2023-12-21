---
title: vue-next-admin管理系统访问api
date: 2023/6/12 21:17
tags: vue-next-admin
categories: vue-next-admin
top_img: ./img/41.jpg
cover: ./img/80.png
---

# vue-next-admin管理系统访问api



### 一，页面加载时查询所有访问后台api查询所有链上的酒，并输出在表格里

![](./img/80.png)

```typescript
// 页面加载时
onMounted(() => {
	getTableData();
});
```



```typescript
// 初始化表格数据
const getTableData = async () => {
	try {
    const url = 'http://xxx.xx.xxx.xx:8080/fabric/gateway';
    const method = 'POST';
    const headers = {
      'Content-Type': 'application/json',
      'accept': 'application/json',
      'connection': 'Keep-Alive',
      'source': 'local'
    };
    const requestBody = {
      cmd: {
        channel: 'winechannel',
        chaincode: 'mycontract',
        action: 'all'
      }
    };

    const response = await axios.post(url, requestBody, {
      headers: headers
    });

	const data = [];
	for (let i = 0; i < response.data.length; i++) {
		const responseData = response.data[i]; // 获取数组中的第一个对象
	const ID = responseData.ID; // 获取ID属性的值
	const Name = responseData.Name; // 获取Name属性的值
	const Value = responseData.Value; // 获取Value属性的值
	const Owner = responseData.Owner; // 获取Owner属性的值
	const Birth = responseData.Birth; // 获取Birth属性的值
	const Capacity = responseData.Capacity; // 获取Capacity属性的值
	state.tableData.loading = true;
		data.push({
			id:  ID,
			name: Name,
			value: Value,
			owner: Owner,
			birth: Birth,
			capacity: Capacity,
		});
	}
	state.tableData.data = data;
	state.tableData.total = state.tableData.data.length;
	setTimeout(() => {
		state.tableData.loading = false;
	}, 500);
	} catch (error) {
    console.error('Error:', error);
  }
};
```



### 二，queryWithTID方法

![](./img/85.png)

```typescript
const queryWithTID =async () =>{
	try {
	  
    const url = 'http://xxx.xx.xx.xx:8080/fabric/gateway';
    const method = 'POST';
    const headers = {
      'Content-Type': 'application/json',
      'accept': 'application/json',
      'connection': 'Keep-Alive',
      'source': 'local'
    };
	const para = tid.value;
	console.log(para);
    const requestBody = {
      cmd: {
        channel: 'winechannel',
        chaincode: 'mycontract',
        action: 'query',
		args:[para]
      }
    };

    const response = await axios.post(url, requestBody, {
      headers: headers
    });
    console.log('Response:', response.data);
	
	var responseString = 'call fail...';
    responseString = JSON.stringify(response.data);

	// const result = []
	const data = [];

	const responseData = response.data; // 获取数组中的第一个对象
	const ID = responseData.ID; // 获取ID属性的值
	const Name = responseData.Name; // 获取Name属性的值
	const Value = responseData.Value; // 获取Value属性的值
	const Owner = responseData.Owner; // 获取Owner属性的值
	const Birth = responseData.Birth; // 获取Birth属性的值
	const Capacity = responseData.Capacity; // 获取Capacity属性的值
	state.tableData.loading = true;
		data.push({
			id:  ID,
			name: Name,
			value: Value,
			owner: Owner,
			birth: Birth,
			capacity: Capacity,
		});
	
	state.tableData.data = data;
	state.tableData.total = state.tableData.data.length;
	setTimeout(() => {
		state.tableData.loading = false;
	}, 500);
	} catch (error) {
    console.error('Error:', error);
  }
}
```

### 
