---
title: 学习日记es6-polyfill
date: 2018-02-16 01:35:36
tags: 
	- es6
	- javascript
categories: 
	- es6
	- 学习日记
---
### 浮点数误差
对于javascript的数字来说，“机器精度”的值通常为2^-52
ES6开始，该值定义在Number.EPSILON中，我们可以拿来直接用，也可以为ES6之前的版本写polyfill
```
if(  !Number.EPSILON  ){
Number.EPSILON  =  Math.pow(2,-52);
}
```
可以用Number.EPSILON来比较两个数字是否相等（在制定误差范围内）
```
function numbersCloseEnoughToEqual(  n1,  n2 ){
return Math.abs(  n1  - n2 )  <  Number.EPSILON ;
}
```