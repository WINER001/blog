---
title: mermaid_test
date: 2023/12/19 14:27
tags: 知识点
categories: 知识点
top_img: ./img/41.jpg
cover: ./img/31.jpg

---

# 一，流程图

### 	1.图形

```
graph TD
开始 --> 结束
```

graph声明了一个新图形和图形布局的方向
TB - 从上到下
BT - 从下到上
RL - 从右到左
LR - 从左到右
TD - 与TB相同

```markdown
{% mermaid %}
graph TD
开始 --> 结束
{% endmermaid %}
```

```
This is my website, click the button {% btn 'https://lightmeter30.github.io/',Butterfly %}
```

This is my website, click the button {% btn 'https://lightmeter30.github.io/',Butterfly %}

{% mermaid %}
pie
    title Key elements in Product X
    "Calcium" : 42.96
    "Potassium" : 50.05
    "Magnesium" : 10.01
    "Iron" :  5
{% endmermaid %}

### 2.节点和形状

(1)start				节点
(2)start1[文本节点]	带有文本的节点 
(3)start2(圆边)	具有圆边的节点
(4)start3((圆形))	圆形的节点   
(5)start4>不对称]  不对称节点
(6)start

