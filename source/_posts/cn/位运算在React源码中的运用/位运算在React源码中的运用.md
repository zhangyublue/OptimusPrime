---
title: 位运算在React源码中的使用
catalog: true
date: 2021-12-1 17:50
subtitle: 该文章基于react版本17.0.2
lang: cn
header-img: /img/header_img/lml_bg.jpg
tags:
- React
categories:
- React
---

# 位运算在react源码中的使用

> - 位运算直接对二进制位进行计算，位运算直接处理每一个比特位，是非常底层的运算。
>
>   优点是速度极快，缺点是不直观。
>
> - 位运算只对整数起作用，如果参与运算的数不是整数，会自动转为整数后在进行。

##  运算符

### 1. & 按位与

对于两个二进制操作数的每个bit进行计算，如果都为1，则结果为1，否则为0。

计算 `3 & 2`， 先将操作数转化为二进制32位 `Int32`

```javascript
// 3对应的 Int32
  0b000 0000 0000 0000 0000 0000 0000 0011 
// 2对应的 Int32
& 0b000 0000 0000 0000 0000 0000 0000 0010 
-------------------------------------------
  0b000 0000 0000 0000 0000 0000 0000 0010
```

所以 `3 & 2` 计算结果转化为浮点数为 2

### 2. | 按位或

对于两个二进制操作数的每个bit，如果都为0， 则结果为0，否则为1.

计算 10 |  3

```javascript
   0b000 0000 0000 0000 0000 0000 0000 1010 
&  0b000 0000 0000 0000 0000 0000 0000 0011
--------------------------------------------
   0b000 0000 0000 0000 0000 0000 0000 1011
```

计算结果转化为浮点数为11

### 3. ~ 安位非

对一个二进制操作数的每个bit，逐位进行取反操作（0，1互换）

对于 `~3`

```javascript
// 3对应的 Int32
~  0b000 0000 0000 0000 0000 0000 0000 0011 
--------------------------------------------
// 逐位取反
   0b111 1111 1111 1111 1111 1111 1111 1100
```

计算结果为 -4



## 在React中判断状态

以下为 react 源码中的一段，通过现在执行的上下文判断react是否在执行任务：

```javascript
if（
  (executionContext & LegacyUnbatchedContext) !== NoContext &&
  (executionContext & (RenderContext | CommitContext)) === NoContext
）{
	// do something
}
```



`excutionContext` 为当前上下文

`LegacyUnbatchedContext` , `RenderContext` , `CommitContext` 为执行阶段的各个上下文

`NoContext` 为没有上下文即 react 没有在执行任务



###  1. 进入某一段上下文时

如 `RenderContext`

```javascript
let excutionContext = 0;
const RenderContext = 0b0010000;
excutionContext |= RenderContext;

  0b0000000
| 0b0010000
------------
  0b0010000
```



### 2. 判断在某一上下文时

```javascript
let excutionContext = 0b0010000;
const RenderContext = 0b0010000;
(excutionContext & RenderContext) !== NoContext

  0b0010000
& 0b0010000
------------
  0b0010000
```



### 离开某个上下文后

```javascript
let excutionContext = 0b0010000;
const RenderContext = 0b0010000;

excutionContext &= ~RenderContext;

	0b1101111
& 0b0010000
------------
  0b0000000
```



## 在React中进行优先级计算

react 中定义了一系列的优先级， renact中用31位保存更新的优先级

处在月底 `bit` 位的更新优先级越高。

如下取了一部分的优先级定义：

```javascript
export const NoLanes: Lanes = /*                        */ 0b0000000000000000000000000000000;
export const NoLane: Lane = /*                          */ 0b0000000000000000000000000000000;

export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000001;
export const SyncBatchedLane: Lane = /*                 */ 0b0000000000000000000000000000010;

export const InputDiscreteHydrationLane: Lane = /*      */ 0b0000000000000000000000000000100;
const InputDiscreteLanes: Lanes = /*                    */ 0b0000000000000000000000000011000;

const InputContinuousHydrationLane: Lane = /*           */ 0b0000000000000000000000000100000;
const InputContinuousLanes: Lanes = /*                  */ 0b0000000000000000000000011000000;
```



经常需要在一系列的更新中找到优先级最高：

```javascript
let lanes = NoLanes;
lanes |= SyncBatchedLane;
lanes |= InputContinuousHydrationLane;

  0b0000000000000000000000000000000
| 0b0000000000000000000000000000010
| 0b0000000000000000000000000000100
-------------------------------------
  0b0000000000000000000000000000110

```



在复合优先级后如何取出这个lanes中最高的那个优先级呢？

react通过的是 `getHighestPriorityLane` 函数

```javascript
function getHighestPriorityLane(lanes) {
  return lanes & -lanes;
}
```

`-lanes` 可以看做两步操作：

1. ~ lanes
2. 加 1

```javascript
// 取反
~  0b0000000000000000000000000000110
--------------------------------------
   0b1111111111111111111111111111001
// 加 1
+  0b0000000000000000000000000000001
--------------------------------------
   0b1111111111111111111111111111010

   0b0000000000000000000000000000110
&  0b1111111111111111111111111111010
---------------------------------------
   0b0000000000000000000000000000010

```



所取到的就是`bit`  位最低的一位，也就是优先级最高的一个。