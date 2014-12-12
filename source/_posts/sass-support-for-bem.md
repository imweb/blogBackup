---
layout: post
title: SASS 3.4对BEM的支持初探
date: 2014-9-15 23:23
comments: true
author: Lqlong
tags: 
    - css 
    - 工具
---

#SASS 3.4对BEM的支持初探

##准备
下面是一个简单的html代码片段，包含一个模块以及它的扩展：
```html
<div class="blockA">
	<h3 class="blockA__elementA blockA__elementA--black">title1</h3>
	<p class="blockA__elementB">description1</p>
</div>

<div class="blockA blockA--modifierA">
	<h3 class="blockA__elementA">title2</h3>
	<p class="blockA__elementB">description2</p>
</div>
```
利用BEM模式编写出来的样式应该是这样的：
```css
.blockA { background-color: #fff; }
    .blockA__elementA { color: red; }
    .blockA__elementA--black { color: black; }
.blockA--modifierA { background-color: #333; }
    .blockA--modifierA .blockA__elementA { color: blue; }
```
上面涵盖了BEM所有的样式类型。
编写这样的样式有好些不爽的点：
- blockA写了多少次？
- elementA也写得不少
- 修饰element，为什么还要多写一次block？
- 样式类名太长了有木有？

有人说，这样的样式代码看起来很整齐，结构分明，但如果你把上面的缩进去掉，样式再复杂再多一点看看？如果你还能忍受，那现在我说，有其他更加简单，整洁并且熟悉的写法呢，你还能hold地住吗？

想知道的话就继续往下，一步一步地挖掘如何利用我们熟悉的css编写工具sass的高级特性来方便地编写BEM模式代码。

##神奇的&
要消灭重复，简单呀，我们可以这样：
```sass
.blockA {
	background-color: #fff;
	&__elementA {
		color: red;
		&--black {
			color: black;
		}
	}
	&--modifierA {
		background-color: #333;
		.blockA {
			&__elementA {
				color: blue;
			}
		}
	}
}
```
嗯，还是这种方式我们熟悉一点吧？不过，还是有两点不爽：
1. 代码结构性较差，难以阅读。
> 不会？block下面是element，然后还有modifier，挺清晰的嘛~
废话，白纸黑字都写着还能不清晰？你试试把上面的blockA，elementA，modifierA换成其他名字看看？
2. .blockA--modifierA .block__elementA的实现方式非常怪异。

下面我们来解决这两个不爽。
##利用mixins
我们可以利用mixins特性来解决第一个问题，如下：
```sass
$elementSeparator: "__";
$modifierSeparator: "--";

@mixin block($block) {
	.#{$block} { @content; }
}

@mixin element($element) {
    &#{$elementSeparator+$element} { @content; }
}

@mixin modifier($modifier) {
    &#{$modifierSeparator+$modifier} { @content; }
}

@include block(blockA) {
	background-color: #fff;
    @include element(elementA){
		color: red;
		@include modifier(black) {
			color: black;
		}
    }
	@include modifier(modifierA) {
		background-color: blue;
	}
}
```
这下结构就很清晰了，不管给block，element和modifier起怎样奇葩的名字，我都知道哪些是block，哪些是element，哪些是modifier了。

另外，利用文本编辑器的缩进功能，代码的结构会更加清晰：

![代码结构](/img/blog/sass-support-for-bem.png)


把叶子层级的@include缩进就能很清晰地了解html结构了，上面的blockA模块包含3个元素，有2个扩展，其中一个元素也被扩展了。

##问题与目标
上一小节解决了第一个不爽，现在来解决第二个不爽。实际上，上一小节的代码没有实现
.blockA--modifierA .block__elementA的样式。

如果只利用那些mixins的话，要实现该样式只能这样：
```sass
@include block(blockA) {
	@include modifier(modifierA) {
		background-color: blue;
		@include block(blockA) {
			@include element(elementA) {
				color: blue;
			}
		}
	}
}
```
上面的sass最终编译出来的结果是正确的，但是太难读了。如果是像下面这样的话就好多了：
```sass
@include block(blockA) {
	@include modifier(modifierA) {
		background-color: blue;
		@include element(elementA) {
			color: blue;
		}
	}
}
```
这段代码就好读多了：blockA有一个扩展modifierA，这个扩展修饰了elementA。
这个就是我们的目标，也是解决第二个不爽的关键，sass 3.4的新特性帮助我们解决这个问题。

##还是神奇的&
很明显，解决问题的关键是扩展element的mixin，当父类包含modifier时，就要改成子选择器。要找到父类，那就自然地想到父选择器&，但是在3.4版本之前，sass中的&只能用于选择器字符串的拼接上。到了3.4版本，sass增强了对&选择器的解释，&也可以作为一个变量来处理了。

要取到父类名，可以利用下面这个函数：
```sass
@function selectorToString($selector) {
	$selector: inspect($selector);
	$selector: str-slice($selector, 2, -2);
	@return $selector;
}
```
取得父类名之后，就能判断父类是否包含modifier了：
```sass
@function containsModifier($selector) {
	$selector: selectorToString($selector);
	@if str-index($selector, $modifierSeparator) {
		@return true;
	} @else {
		@return false;
	}
}
```
我们还需要一个获取block名的函数：
```sass
@function getBlock($selector) {
	$selector: selectorToString($selector);
	$modifierStart: str-index($selector, $modifierSeparator) - 1;
	@return str-slice($selector, 0, $modifierStart);
}
```
有了这些功能之后，我们就可以扩展element的mixin了：
```sass
@mixin e($element) {
	$selector: &;   //3.4以下版本这里会报错
	@if containsModifier($selector) {
		$block: getBlock($selector);
		@at-root {
			#{$selector} {
				#{$block+$elementSeparator+$element} {
					@content;
				}
			}
		}
	} @else {
		&#{$elementSeparator+$element} { @content; }
	}
}
```
> @at-root是3.3的特性，可以让样式脱离父类。

##最终代码
```css
@charset "utf-8";

$elementSeparator: "__";
$modifierSeparator: "--";

@function selectorToString($selector) {
	$selector: inspect($selector);
	$selector: str-slice($selector, 2, -2);
	@return $selector;
}

@function containsModifier($selector) {
	$selector: selectorToString($selector);
	@if str-index($selector, $modifierSeparator) {
		@return true;
	} @else {
		@return false;
	}
}

@function getBlock($selector) {
	$selector: selectorToString($selector);
	$modifierStart: str-index($selector, $modifierSeparator) - 1;
	@return str-slice($selector, 0, $modifierStart);
}

@mixin b($block) {
	.#{$block} { @content; }
}

@mixin e($element) {
	$selector: &;
	@if containsModifier($selector) {
		$block: getBlock($selector);
		@at-root {
			#{$selector} {
				#{$block+$elementSeparator+$element} {
					@content;
				}
			}
		}
	} @else {
		&#{$elementSeparator+$element} { @content; }
	}
}

@mixin m($modifier) {
    &#{$modifierSeparator+$modifier} { @content; }
}

//test code
@include b(blockA) {
	background-color: #fff;
    @include e(elementA){
		color: red;
		@include m(black) {
			color: black;
		}
    }
	@include m(modifierA) {
		background-color: #333;
		@include e(elementA) {
			color: blue;
		}
	}
}

//output css
.blockA { background-color: #fff; }
.blockA__elementA { color: red; }
.blockA__elementA--black { color: black; }
.blockA--modifierA { background-color: #333; }
.blockA--modifierA .blockA__elementA { color: blue; }
```
