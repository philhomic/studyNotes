#jQuery逐行分析学习笔记

##jQuery整体架构

以jQuery v2.0.3为例


```js
(function(){

	(21, 94) 定义了一些变量和函数 jQuery = function(){};
	
	(96, 283) 给JQ对象添加一些方法和属性

	(285, 347) extend : JQ的继承方法

	(349, 817) jQuery.extend() : 扩展一些工具方法

	(877, 2856) Sizzle : 复杂选择器的实现

	(2880, 3042) Callbacks : 回调对象：对函数的统一管理

	(3043, 3183) Deferred : 延迟对象：对异步的统一管理

	(3184, 3295) support : 功能检测

	(3308, 3652) data() : 数据缓存

	(3653, 3797) queue() : 队列管理

	(3803, 4299) attr() prop() val() addClass等 : 对元素属性的操作

	(4300, 5128) on() trigger() : 事件操作的相关方法

	(5140, 6057) DOM操作 : 添加 删除 获取 包装 DOM筛选

	(6058, 6620) css() : 样式的操作

	(6621, 7854) 提交的数据和ajax() : ajax() load() getJson()

	(7855, 8584) animate() : 运动的方法
	
	(8585, 8792) offset() : 位置和尺寸的方法

	(8802, 8821) JQ支持模块化的模式 

	(8826) window.jQuery = window.$ = jQuery;

})()
```

##jQuery逐行解析

###(21, 94) 定义了一些变量和函数 jQuery = function(){};
	

```js
//匿名函数自执行传入window的好处

(function(window){
	//window
	//1. 找到window更快，不需要一层一层地找最外层的window
	//2. 有利于压缩代码，例如：(function(w){ 在这里用w即可 })(window)
})(window)
```

```js
//匿名函数为什么要传入undefined

function(window, undefined){
	//undefined在外面是window下的一个属性，如果不传入的话，有被修改的风险
	//在里面用undefined的时候，先找参数undefined，而不会找外面的
})(window)
```

```js
"use strict";

//js的严格模式，写代码要十分规范
```

```js
//对undefined的判断
window.a == undefined; //在IE9中 tyoeof xmlNode.method用这种方法检查不出来
typeof a == 'undefined'; //用这种方式检查比较保险
```

变量名防冲突

```js
var $ = 10;

//如果外面对$或jQuery进行过定义了，那么来到jQuery内部的时候走下面两句：
var _jQuery = window.jQuery,
    _$ = window.$, //此处就将在外部定义的$的值赋给了_$，以防止与内部的$冲突。
//如果在外部没有对$或jQuery进行过其他定义，那么这里的_jQuery和_$存的就是undefined
```

jQuery实现链式操作的原理

```js
jQuery = function(selector, context) {
	return new jQuery.fn.init(selector, context, rootjQuery);
}

//jQuery函数返回的是一个对象，jQuery.fn.init才是真正的构造函数

//...

jQuery.fn = jQuery.prototype = { //jQuery.fn就是jQuery的原型
	//...
}

/*
一般的面向对象的写法：
function Aaa(){}
Aaa.prototype.init = function(){};
Aaa.prototype.css = function(){};

var a1 = new Aaa();
a1.init();
a1.css();
*/

//jQuery的return new jQuery.fn.init()，直接在创建对象的时候，就将init函数执行掉了

//...
jQuery.fn.init.prototype = jQuery.fn; //jQuery.fn.init的原型就是jQuery的原型，所以可以用new
```

匹配数字的正则

```js
core_pnum = /[+-]?(?:\d*\.|)\d+(?:[eE][+-]?\d+|)/.source,

//包含正负号、小数点、科学计数法
```

匹配标签或id

```js
rquickExpr = /^(?:\s*(<[\w\W]+>)[^>]*|#([\w-]*))$/

//例如 <p>aaaa 或#div1
```

匹配成对标签

```js
rsingleTag = /^<(\w+)\s*\/?>(?:<\/\1>|)$/

//例如 <p></p> <div></div> 或 <img />
```

匹配厂商前缀：

以margin-left为例，webkit会转成webkitMarginLeft，但是对于IE来说却是例外，要写成 MsMarginLeft，注意这里的首字母M大写了。

###(96, 283) 给JQ对象添加一些方法和属性

该部分简化的代码：

```js
jQuery.fn = jQuery.prototype = { //添加实例属性和方法
	jquery: 版本
	constructor: 修正指向问题
	init(): 初始化和参数管理
	selector: 存储选择字符串
	length: this对象的长度
	toArray(): 转数组
	get(): 转原生集合
	pushStatck(): JQ对象的入栈
	each(): 遍历集合
	ready(): DOM加载的接口
	slice(): 集合的截取
	first(): 集合的第一项
	last(): 集合的最后一项
	eq(): 集合的指定项
	map(): 返回新集合
	end(): 返回集合前一个状态
	push(): (内部使用)
	sort(): (内部使用)
	splice(): (内部使用)
}
```

了解constructor

```js
function Aaa(){}
//当构造函数生成之后，js源码中会自动在其prototype下面生成一个constructor，指向这个构造函数
//相当于执行了一句 Aaa.prototype.constructor = Aaa;
var a1 = new Aaa();
alert(a1.constructor); //function Aaa(){...}

//在面向对象的写法当中，以下两种写法大有不同，第二种写法会一不小心就改掉了constructor属性

Aaa.prototype.name = 'hello'; //添加写法
Aaa.prototype.age = 30;

Aaa.prototype = { //覆盖写法 这个时候就需要constructor的修正
	//修正 constructor: Aaa,
	name: 'hello',
	age: 30
};
```

jQuery中的init需要处理的各种情况

- selector为字符串时
	- $(""), $(null), $(undefined), $(false)
	- $('#div1') $('.box') $('div') $('#div1 div.box')
	- $('\<li\>') $('\<li\>1\</li\>\<li\>2\</li\>') 创建标签的写法
- selector为元素的时候
	- $(this) $(document)
- selector为函数的时候
	- $(function(){})
- $([]) $({}) 

在jQuery源码中，是这样判断的：

```js
if (){
	//$('<li>') $('<li>1</li><li>2</li>')

	//match = [null, '<li>', null]; 或
	//match = [null, '<li>1</li><li>2</li>', null]
} else {
	//$('#div1') $('.box') $('div') $('#div1 div.box')
	//$('<li>hello') -> 例如 $('<li>hello').appendTo($('ul'));  这个时候只能添加li，但是hello却添加不了，所以它也相当于$('<li>')

	//当为$('#div1')和$('<li>hello')的时候，match通过rquickExpr.exec(selector)可以匹配成这样：
	//匹配id的时候，match = ['#div1', null, 'div1'];
	//匹配$('<li>hello')的时候，match = ['<li>hello', '<li>', null]

	//但是$('.box') $('div') $('#div1 div.box') 这些的时候，正则匹配不到，所以 match = null;

	//总结下来：
	/*
	match = null; //$('.box') $('div') $('#div1 div.box')
	match = ['#div1', null, 'div1']; //$('#div1')
	match = ['<li>hello', '<li>', null] //$('<li>hello') */
}

if() { //能进入的有： $('<li>') $('#div1')
	if (){ //能进入的是创建标签
		//$('<li>')
		//创建标签时候，可以写第二个参数的情况有：$('<li>', document) 这时候，这个document写不写，都是在当前document中创建li，但是写成 $('<li>', contentWindow.document) 就是有iframe的时候，在iframe的document上创建li
	} else { //else走的是选择id
		//$('#div1')
	}
}
```

类似数组的对象

```js
this = {
	0 : 'li',
	1 : 'li',
	2 : 'li',
	length : 3
}
```

对于这样一个对象，可以采用for循环。

```js
for (var i = 0; i < this.length; i++){
	this[i].style.background = 'red';
}
```

在jQuery中，确实就是如此，因此：

```js
$(function(){
	//$('li') -> this
	//因为this是存成上面的那样的形式的，所以
	//$('li')[1] 就可以找到里面具体存的一个li，而且这个li是原生的
	$('li')[1].style.background = 'red'; //就可以单独对一个li进行样式操作了
});
```

jQuery.parseHTML: 将字符串转为节点数组

```js
$(function(){
	var str = '<li>1</li><li>2</li><li>3</li><script>alert(4)<\/script>'; 
	//注意script标签这里的反斜杠要转义，面对让计算机误以为与页面前方出现的<script>标签成要一对
	var arr = jQuery.parseHTML(str, document, true);  //第二个参数是指定根节点；第三个参数是布尔值，就是用来看最后的那个script标签能不能添进去，第三个参数默认为false，也就是说不能添加script标签，但是如果传入的为true，就代表能够添加script标签
	//arr = ['li', 'li', 'li']
	$.each(arr, function(i){
		$('ul').append(arr[i]); //确实能添加到页面上
	})
})
```

jQuery.merge的用法

```js
//jQuery.merge对外使用的时候，作用是数组合并
var arr = ['a', 'b'];
var arr2 = ['c', 'd'];

$.merge(arr, arr2); //['a', 'b', 'c', 'd']

//对内使用，还可以进行json的合并，但是json必须是特殊形式
var arr = {
	0: 'a', 
	1: 'b',
	length: 2
}

var arr2 = ['c', 'd'];
$.merge(arr, arr2);  //合并完之后成为一个json了
/*
{
	0: 'a',
	1: 'b',
	2: 'c',
	3: 'd'
	length: 4
}
*/
```

针对创建标签并添加属性的形式：

```js
$('<li>', {title: 'hi', html: 'abcd', css: {background: 'red'}}).appendTo('ul');
```

要完成上面的功能，jq中是这样实现的：

```js
if(rsingleTag.test(match[1]) && jQuery.isPlainObject(context)){
	for(match in context){
		if(jQuery.isFunction(this[match])){
			this[match](context[match]);
			//如果jquery中有这个方法，比如 $(...).html()方法或者$(...).css()方法，那么就直接调用方法
		} else {
			this.arrt(match, context[match]);
			//如果jquery中没有对应方法，如此例中的title，那么就添加属性
		}
	}
}
```

$(#id)的处理

```js
//match = ['#div1', null, 'div1'] //$('#div1')
elem = document.getElementById(match[2]);
if(elem && elem.parentNode){
	this.length = 1;
	this[0] = elem;
}
this.context = document;
this.selector = selector;
return this;
```

jQuery.makeArray的用法

```js
//比如在页面上有三个div

//$.makeArray对外使用，是可以将类似数组的转为数组
$(function(){
	var aDiv = document.getElementByTagName('div');
	//aDiv.push() 报错
	$.makeArray(aDiv);
	//aDiv.push() 不再报错了

	//$.makeArray在内部使用，可以传入第二个参数，可以将转化出来的数组添加到后面第二个参数的对象上去
	console.log($.makeArray(aDiv, {length: 0}));
	//{0: div, 1: div, 2: div, length: 3}
})
```

toArray()方法：转数组

```js
//假设有三个div
$(function(){
	console.log($('div')); //对象
	console.log($('div').toArray()); //数组（原生元素组成的数组，后面要调用原生的方法）
})

//$(...).toArray()的实现方法
function(){
	return core_slice.call(this);
	//在jq源码中，core_slice = core_deletedIds.slice
	//而前面定义的core_deletedIds = [];
	//所以core_slice.call(this)就相当于[].slice.call(this); 典型地转换位数组的做法
}
```

get()方法：转原生集合

```js
//get方法的使用
/*
比如在DOM中：
<div>111111111</div>
<div>111111111</div>
<div>111111111</div>
*/
//$('div').get(0); //得到第一个原生div，后面用原生方法
$('div').get(0).innerHTML = '22222';

//$('div').get(); //get()方法不传参，就返回三个div的一个集合
for(var i = 0; i < $('div').get().length; i++){
	$('div').get(i).innerHTML = '33333';
}

//get方法的实现
function(num){
	return num == null?
		this.toArray() : //返回一个“干净”的数组
		(num < 0 ? this[this.length + num] : this[num]);
}
```

pushStack()方法：JQ对象的入栈（栈是先进后出）

```js
//pushStack()的使用
/* 页面中
<div>div</div>
<span>span</span>
*/

//$('div').pushStack($('span')).css('background', 'red'); //span背景变红
$('div').pushStack($('span')).css('background', 'red').css('background', 'green'); //span最终变成绿色了
$('div').pushStack($('span')).css('background', 'red').end().css('background', 'yellow'); //span为红，div为黄
//用end()可以追溯到stack的下一层

//pushStack()方法的实现
function(elems){
	var ret = jQuery.merge(this.constructor(), elems);
	//this.constructor()是一个空的jQuery对象
	ret.prevObject = this;
	ret.context = this.context;
	return ret;
}
```

end()方法

```js
function(){
	//这里的prevObject就是在pushStack方法里面添加的属性
	return this.prevObject || this.constructor(null);
}
```

slice()方法


```js
/* 假设DOM中有四个div */
$('div').slice(1, 3).css('background', 'red'); //第2、3个div背景变为红色

$('div').slice(1, 3).css('background', 'red').end().css('color', 'blue'); //第2、3个div背景变为红色，然后全部四个div的文字颜色变为蓝色

//slice()方法的实现
function(){
	return this.pushStack(core_slice.apply(this, arguments))
}
```

each()、ready()方法都调用了工具方法，等讲到工具方法的时候再说。

eq()方法：集合的指定项

```js
//eq的实现
function(i){
	var len = this.length;
	var j = +i + (i < 0 ? len : 0);
	return this.pushStack(j >= 0 && j < len ? [this[j]] : []);
}
```

###(285, 347) extend : JQ的继承方法

```js
//extend的使用方法
//$.extend()
//$.fn.extend()

//当只写一个对象自变量的时候，JQ中扩展插件的形式
$.extend({ //扩展工具方法
	aaa: function(){
		alert(1);
	},
	bbb: function(){
		alert(2);
	}
});
$.aaa(); //1
$.bbb(); //2

$.fn.extend({ //扩展实例方法
	aaa: function(){
		alert(3);
	},
	bbb: function(){
		alert(4);
	}	
});
$().aaa(); //3
$().bbb(); //4

//$.extend(); -> this -> $ -> this.aaa -> $.aaa()
//$.fn.extend(); -> this -> $.fn -> this.aaa -> $().aaa()

//当写多个对象自变量的时候，后面的对象都扩展到第一个对象上
var a = {};
$.extend(a, {name: 'hello'}, {age: 30});
console.log(a); //{name: 'hello', age: 30}

//还可以做深拷贝和浅拷贝，默认是浅拷贝
var a = {};
var b = {name: 'hello'};
$.extend(a, b);
a.name = 'hi';
alert(b.name); //'hello'

var a = {};
var b = {name: {age: 30}};
$.extend(a, b);
a.name.age = 20;
alert(b.name.age); //20 受到影响了，因为默认是浅拷贝

//使用深拷贝 $.extend(true, a, b) 这样就不受影响了
```

extend的实现

```js
//简化的总体结构

jQuery.extend = jQuery.fn.extend = function(){
	//定义一些变量

	if(){} 看是不是深拷贝情况
	if(){} 看参数是否正确
	if(){} 看是不是插件情况
	for(){ 可能有多个对象的情况
		if(){} 防止循环引用
		//$.extend(a, {name: a}); 这样出现了循环引用
		if(){} 深拷贝
		//需要判断 例如：
		//var a = { name : { job : 'it' } }
		//var b = { name : { age : 30 } }
		//$.extend(true, a, b);
		//a -> { name : { job: 'it', age: 30} }
		else if(){} 浅拷贝
	}
}
```

```js
//根据jQuery原理，简单实现了一个深拷贝的函数
function deepClone(target){
  var options, name, src, copy, clone;
  for(var i = 1; i < arguments.length; i++){
    options = arguments[i];

    for(name in options){
      src = target[name];
      copy = options[name];

      if(src && src instanceof Object){
        clone = src;
      } else if(copy instanceof Array){
        clone = [];
      } else if(copy instanceof Object){
        clone = {};
      }

      if(copy && copy instanceof Object){
        target[name] = deepClone(clone, copy)
      } else {
        target[name] = copy;
      }
    }
  }
  return target;
}
```

jQ中选择：拷贝继承
JS：类式继承 （new 构造函数） / 原型继承 {}

###(349, 817) jQuery.extend() : 扩展一些工具方法

工具方法既可以给JQ用也可以给原生来用。

```js
jQuery.extend({
	expando: 生成唯一JQ字符串（内部）
	noConflict(): 防止冲突
	isReady: DOM是否加载完（内部）
	readyWait: 等待多少文件的计数器（内部）
	holdReady(): 推迟DOM触发
	ready(): 准备DOM触发
	isFunction(): 是否为函数
	isArray(): 是否为数组
	isWindow(): 是否为window
	isNumeric(): 是否为数字
	type(): 判断数据类型
	isPlainObject(): 是否为对象自变量
	isEmptyObject(): 是否为空的对象
	error(): 抛出异常
	parseHTML(): 解析节点
	parseJSON(): 解析JSON
	parseXML(): 解析XML
	noop(): 空函数
	globalEval(): 全局解析JS
	camelCase(): 转驼峰
	nodeName(): 是否为指定节点名（内部）
	each(): 遍历集合
	trim(): 去前后空格
	makeArray(): 类数组转真数组
	inArray(): 数组版indexOf
	merge(): 合并数组
	grep(): 过滤新数组
	map(): 映射新数组
	guid: 唯一标识符（内部）
	proxy(): 改this指向
	access(): 多功能值操作（内部）
	now(): 当前时间
	swap(): CSS交换（内部）
});
jQuery.ready.promise = function(){}; 监测DOM的异步操作（内部）
function isArraylike(){} 类似数组的判断（内部）
```

noConflict 防止冲突


```js
//用法
var miaov = $.noConflict();

var $ = 123;

miaov(function(){
	alert($);
});

//实现
//在jq一开头中有定义变量：
var _jQuery = window.jQuery,
	_$ = window.$;

function(deep){ //这里的deep是决定是否放弃jQuery这个接口的
	if(window.$ === jQuery){
		window.$ = _$;
	}
	if(deep && window.jQuery === jQuery){
		window.jQuery = _jQuery;
	}
}
```

DOM加载相关

```js
$(function(){}) //页面中的DOM加载完，就走中间的函数
window.onload = function(){} //页面所有内容都加载完，就走后面的函数

//DOMContentLoaded事件，原生

//$(function(){})调用的接口是
if ( jQuery.isFunction( selector ) ) {
	return rootjQuery.ready( selector );
}
//rootjQuery.ready(selector); 就相当于 $(document).ready(function(){})
//所以 $(function(){})和$(document).ready(function(){})是一回事

//$().ready() -> 这是一种实例方法
//$.ready() -> 这是工具方法

//在$().ready() 中，调用的是
jQuery.ready.promise().done(fn); //最后是一个 .done() 就看出来，前面的jQuery.ready.promise()返回的应该是一个延迟对象。fn这个函数会被存起来，然后等到适当的时机再触发

//jQuery.ready.promise()是这样定义的：
function( obj ) {
	if ( !readyList ) {
		readyList = jQuery.Deferred(); //创建延迟对象
		if ( document.readyState === "complete" ) {
			setTimeout( jQuery.ready );
		} else {
			document.addEventListener( "DOMContentLoaded", completed, false );
			window.addEventListener( "load", completed, false );
		}
	}
	//不管走if还是else，最终走的都是jQuery.ready()这个工具方法
	return readyList.promise( obj );
};

//以上代码中的completed回调函数是这样定义的：
completed = function() {
	document.removeEventListener( "DOMContentLoaded", completed, false );
	window.removeEventListener( "load", completed, false );
	jQuery.ready();
};
```

```js
//现在来看$.ready()工具方法的定义
function( wait ) {
	if ( wait === true ? --jQuery.readyWait : jQuery.isReady ) {
		return;
	}

	jQuery.isReady = true;

	if ( wait !== true && --jQuery.readyWait > 0 ) {
		return;
	}

	readyList.resolveWith( document, [ jQuery ] ); //这一句就是在延迟对象中看是否已完成 resolveWith代表已完成，已经准备好了。当走到这一句的时候，后面就调用 jQuery.fn了

	if ( jQuery.fn.trigger ) {
		jQuery( document ).trigger("ready").off("ready");
	}
}
```

```js
//readyList.resolveWith(document, [jQuery]);
//看到这一句，可以来验证
$(function(arg){
	alert(arg); //这里的arg就是jQuery
	alert(this); //这里的this就是document
})
```

```js
// if ( jQuery.fn.trigger ) {
// 	jQuery( document ).trigger("ready").off("ready");
// }
//以上代码对应的是 $(function(){})和$(document).ready(function(){})的另一种写法
$(document).on('ready', function(){});
//以上就是利用jQuery的主动触发来触发的 jQuery.fn.trigger就是主动触发的方法
```

$.holdReady() 推迟DOM触发的用法

```js
$.holdReady(true); //把ready推迟了
//$.getScript()方法是异步的，所以不影响后续加载，可能会让ready限制性，所以要把ready推迟，才能保证执行顺序
$.getScript('a.js', function(){ //a.js的内容是 alert(1)
	$.holdReady(false); //释放推迟
})

$.holdReady(true);
$.getScript('b.js', function(){
	$.holdReady(false);
})

$.holdReady(true);
$.getScript('c.js', function(){
	$.holdReady(false);
})

//上面holdReady调用了三回，那么holdReady函数中的jQuery.readyWait++就加了三回，然后释放的时候，就在ready函数里面jQuery.readyWait再依次递减。当readyWait为0的时候，ready函数内部再继续向下走

$(function(){
	alert(2);
})
```

$.isArray判断是否为数组的实现

```js
//采用原生的Array.isArray即可判断
isArray: Array.isArray
```

$.isWindow判断是否为window对象

```js
function(obj){
	return obj != null && obj === obj.window;
}
```

$.isNumeric判断是否为数字

```js
function(obj){
	return !isNaN(parseFloat(obj)) && isFinite(obj);
}
```





