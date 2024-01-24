---
title: 给可任意选择一级的el-cascader赋值时子节点radio不默认选中的解决方式
author: DuCheng
tags:
  - vue
  - element
categories:
  - 前端
date: 2023-02-06 09:40:00
---

问题呈现：
![问题呈现](http://img.dcwedu.top/i/2024/01/24/65b0ba94142c9.png)
解决后：
![解决后](http://img.dcwedu.top/i/2024/01/24/65b0ba94636aa.png)
代码：

```javascript
// 解决el-cascader子节点不默认选中问题
this.$nextTick(() => {
  // 获取所有选中的节点
  let checkedLeaves = document.getElementsByClassName('in-checked-path')
  // 获取子节点
  let checkedLeaf = checkedLeaves[checkedLeaves.length - 1]
  // console.log('leaf', checkedLeaf)
  // 获取子节点中的radio元素
  let radio = checkedLeaf.childNodes[0].childNodes[0]
  radio.classList.add('is-checked')
})
```

**注：将该代码放在 mounted 生命周期中，由于此时元素不一定渲染完全，若有接口请求等异步操作，需要用`async/await`关键字，以此确保能获取到元素。**
`.in-checked-path`在 elementui 中表示该节点路径被选中（即图中样式不同部分），需要给 radio 加上`.is-checked`使其变成被选中的节点
