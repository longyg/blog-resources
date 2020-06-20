---
title: 理解各种CSS单位（px, em, rem, vw, vh, vmin, vmax）
cover: false
top: false
date: 2019-01-01 18:09:52
group: frontend
permalink: learn-css-unit
categories: 前端
tags:
- CSS
- CSS单位
keywords:
- 理解各种CSS单位
summary: 本文介绍了CSS里的各种单位的区别
---


### px

`px`是`绝对单位`，页面按照精确的`像素`展示。`1px`表示`1像素`。

### em

`em`是`相对单位`，相对于`父节点`或者`自身`的`字体大小`。比如父节点定义了字体大小为`font-size:10px`，那么子节点如果定义`1em`就等于`10px`，`1.2em`等于`12px`。 

如果所有父节点都没有定义字体大小，会相对于`根节点`计算相对大小。 

因此使用`em`单位可以很方便调整页面的显示。我们只需要修改父节点或者根节点的字体大小，就可以控制页面的显示效果。 

注意，`em`不仅仅可以作为字体大小单位，也可以其他任何属性的单位，例如`width`, `height`, `border`等等。 

示例：

```html
<html style="font-size:14px">
	<head>
		<title>Test HTML</title>
	</head>
	<body>
		<div style="width: 10em; height: 10em; border:0.2em solid gray;">
			<div style="font-size: 12px">
				<span style="font-size: 1.3em">This is the test text</span>
			</div>
		</div>
	</body>
</html>
```

第一个`div`的`宽度`，`高度`以及`边框大小`会相对于根节点`<html>`的`字体大小`，即`14px`。而第二个`div`下的`span`的字体大小`1.3em`会相对于第二个`div`的字体大小，也就是`12px`。

### rem

`rem`可以理解为`特殊的em`，因为它始终相对于`根节点`的字体大小。 

示例：

```html
<html style="font-size:14px">
	<head>
		<title>Test HTML</title>
	</head>
	<body>
		<div style="width: 10rem; height: 10rem; border:0.2rem solid gray;">
			<div style="font-size: 12px">
				<span style="font-size: 1.3rem">This is the test text</span>
			</div>
		</div>
	</body>
</html>
```

同样，第一个`div`的宽度，高度以及边框大小会相对于根节点`<html>`的字体大小，即`14px`。不同的是`span`的字体大小`1.3rem`同样会相对于根节点字体大小，即使父节点定义了不同的字体大小，也不会对其起任何作用。

### vw，vh，vmin，vmax

- `vw`：viewpoint width，`视窗宽度`，`1vw`等于视窗宽度的`1%`，`100vw`等于视窗宽度。 
- `vh`：viewpoint height，`视窗高度`，`1vh`等于视窗高度的`1%`，`100vh`等于视窗高度。 
- `vmin`：`vw`和`vh`中较小的那个。 
- `vmax`：`vw`和`vh`中较大的那个。 

这个单位可以使你的应用容易的适配各种大小的浏览器窗口。因为它会随着浏览器窗口大小而动态变化。 

示例：

```html
<html style="font-size:14px;">
	<head>
		<title>Test HTML</title>
	</head>
	<body style="margin:0px;padding:0px">
		<div style="width: calc(100vm - 2px); height:calc(100vh - 2px);border:1px solid gray">
			<div style="font-size: 12px">
				<span style="font-size: 1.3rem">This is the test text</span>
			</div>
		</div>
	</body>
</html>
```

对于第一个`div`，我们想让它始终占满整个窗口，就可以用`vm`和`vh`单位。 

因为`border`在宽度和高度上各占了`2px`，因此我们在`div`的`width`和`height`属性上各减去`2px`从而使`div`刚好占满浏览器窗口。 

你可以试试调整浏览器窗口的大小，或者缩放浏览器，这个`div`始终会跟随浏览器窗口而变化，不管怎么调整，都不会因为`div`边界超出浏览器窗口而出现滚动条。