---
title: Tree & Object 相互转换
date: 2021-12-02 17:59:12
tags: 
  - 算法
categories: js基础
---

数组转化为树结构，这种需求确实在开发中也有遇到，尤其是一些树图一类的处理。于是思考并整理了几个相关的算法，包括常见的递归和效率高一点的循环。

``` js
const arr = [
  {id: 1, name: '部门1', pid: 0},
  {id: 2, name: '部门2', pid: 1},
  {id: 3, name: '部门3', pid: 1},
  {id: 4, name: '部门4', pid: 3},
  {id: 5, name: '部门5', pid: 4},
]

const obj = {
  id: 1,
  name: "部门1",
  pid: 0,
  children: [
    {
      id: 2,
      name: "部门2",
      pid: 1,
      children: []
    },
    {
      id: 3,
      name: "部门3",
      pid: 1,
      children: [
        {
          id: 4,
          name: "部门4",
          pid: 3,
          children: [
            {
              id: 5,
              name: "部门5",
              pid:4,
              children: []
            }
          ]
        }
      ]
    }
  ]
}
```

tips：以下算法均以上面的数据结构实现，主要体现思路，注意注释

## array to tree

### 递归

定义方法获取指定 `id` 的所有 `children`，每个 `children` 递归调用该方法获取 `children`.

``` js
/**
 * @param {arr: array 原数组数组, id: number 父节点id} 
 * @return {children: array 子数组}
 */
function getChildren(arr, id) {
  const res = [];
  for (let item of arr) {
    if (item.pid === id) { // 找到当前id的子元素
      // 插入子元素，每个子元素的children通过回调生成
      res.push({...item, children: getChildren(arr, item.id)});
    }
  }
  return res;
}

getChildren(arr, 0);
```

定义方法获取指定 `id` 的 `node`，`node.children` 的每个元素 调用该方法获取新的子元素

``` js
/**
 * @param {arr: array 原数组数组, id: number 当前元素id} 
 * @return {children: array 子数组}
 */
function getNode(arr, id){
  const node = arr.find(i => i.id === id); // 找到当前元素
  node.children = [];
  for(const item of arr) {
    if (item.pid === id) {
      // 给当前元素添加子节点，每个子节点递归调用
      node.children.push(getNode(arr, item.id));
    }
  }
  return node;
}

getChildren(arr, 1);
```

### 循环

利用 map 存储数组数据，并且每次循环处理对应的元素，利用指针指向内存空间的原理

``` js
/**
 * @param {arr: array, pid: number}
 * @return {obj: object}
 */
function fn(arr, pid) {
  const map = new Map(); // 生成map存储元素
  for(const item of arr) {
    if (!map.has(item.id)) { // 若map中没有当前元素，添加并初始化children
      map.set(item.id, {...item, children: []});
    }
    if (map.has(item.pid)){ // 查找父元素，存在则将该元素插入到children
      map.get(item.pid).children.push(map.get(item.id));
    } else { // 否则初始化父元素，并插入children
      map.set(item.pid, {children: [map.get(item.id)]});
    }
  }
  return map.get(pid);
}

fn(arr, 0);
```

## tree to array 【扁平化数据】

### 递归

``` js
/**
 * @param {obj: object, res: array}
 * @return {arr: array}
 */
function fn(obj, res = []) { // 默认初始结果数组为[]
  res.push(obj); // 当前元素入栈
  // 若元素包含children，则遍历children并递归调用使每一个子元素入栈
  if (obj.children && obj.children.length) {
    for(const item of obj.children) {
      fn(item, res);
    }
  }
  return res;
}

fn(arr);
```

``` js
/**
 * @param { arr: array } 接收子数组，调用时可以构造 `{children: [...]}` 开始调用
 * @return { arr: array }
 */
function fn(arr) {
  const res = [];
  res.push(...arr); // chilren插入结果数组
  for(const item of arr) { // 遍历子元素，若包含children则递归调用
    if (item.children && item.children.length) {
      res.push(...fn(item.children))
    }
  }
  return res;
}

fn([children: [obj]]);
```

### 循环

基本大部分递归都可以用栈的思想改为循环，将每次需要处理的元素压入栈中，处理后弹出，至栈为空

``` js
/**
 * @param {obj: object} 
 * @return {arr: array}
 */
function fn(obj) {
  const stack = [];         // 声明栈，用来存储待处理元素
  const res = [];           // 接收结果
  stack.push(obj);          // 将初始元素压入栈
  while(stack.length) {     // 栈不为空则循环执行
    const item = stack[0];  // 取出栈顶元素
    res.push(item);         // 元素本身压入结果数组
    stack.shift();          // 将当前元素弹出栈
    // 逻辑处理，如果当前元素包含子元素，则将子元素压入栈
    if (item.children && item.children.length) {
      stack.push(...item.children);
    }
  }
  return res;
}

fn(obj);
```
