---
title: 学习日记css小锦囊-持续更新
date: 2018-04-01 19:45:09
tags: 
	- css3
categories: 
	- css
	- css3
	- 学习日记
---
#### css3 截取前几行内容并且垂直居中显示
	```css
	{
	position: absolute;
	left: 50%;
	top: 50%;
	transform: translate(-50%,-50%);
	width: 100%;
	overflow: hidden;
	text-overflow: ellipsis;
	display: -webkit-box;
	-webkit-line-clamp: 2;
	-webkit-box-orient: vertical;
	}

	/* vue打包会丢失box-orient属性，需要特殊处理 */
	/*! autoprefixer: off */
	-webkit-box-orient: vertical;
	/* autoprefixer: on */
	```



#### input placeholder修改颜色
	```css
	::-webkit-input-placeholder { /* WebKit browsers */ 
	color: #999; 
	} 
	:-moz-placeholder { /* Mozilla Firefox 4 to 18 */ 
	color: #999; 
	} 
	::-moz-placeholder { /* Mozilla Firefox 19+ */ 
	color: #999; 
	} 
	:-ms-input-placeholder { /* Internet Explorer 10+ */ 
	color: #999; 
	} 
	```
#### 手动写三角
	```css
	{
    width: 0px;
    height: 0;
    border: .11rem solid transparent;
    border-top: .15rem solid #8D3A3A;
    display: block;
	}
	```
