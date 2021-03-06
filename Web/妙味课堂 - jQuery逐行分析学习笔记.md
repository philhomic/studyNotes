#妙味课堂 - jQuery逐行分析学习笔记

##jQuery整体架构

以jQuery v2.0.3为例


```js
(function(){

	(21, 94) 定义了一些变量和函数 jQuery = function(){}; //√

	(96, 283) 给JQ对象添加一些方法和属性 //√

	(285, 347) extend : JQ的继承方法 //√

	(349, 817) jQuery.extend() : 扩展一些工具方法 //√

	(877, 2856) Sizzle : 复杂选择器的实现

	(2880, 3042) Callbacks : 回调对象：对函数的统一管理 //√

	(3043, 3183) Deferred : 延迟对象：对异步的统一管理 //√

	(3184, 3295) support : 功能检测 //√

	(3308, 3652) data() : 数据缓存 //√

	(3653, 3797) queue() : 队列管理 //√

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
jQuery.fn.init.prototype = jQuery.fn; //jQuery.fn.init的原型就是jQuery的原型，所以可以用new，而且jQuery原型上有的东西，new一个jQuery.fn.init()之后得到的对象上也都有
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

$.type的使用及实现

```js
var a = 'hello';
var b = [];
var c = {};
var d = null;
var e = new Date;
alert($.type(a)); //string
alert($.type(b)); //array typeof的话会返回object
alert($.type(c)); //object
alert($.type(d)); //null
alert($.type(e)); //date typeof的话会返回object
alert($.type(null)) //null
alert($.type(undefined)) // undefined
```

```js
//$.type的实现
function(obj){
	if(obj == null){
		return String(obj);
		//通过这个方法，将null和undefined转为字符串
		//null == null -> true
		//undefined == null -> true
	}
	return typeof obj === "object" || typeof obj === "function" ?
		class2type[core_toString.call(obj)] || "object" :
		typeof obj;
}

//其中 core_toString 存的是class2type.toString
//而class2type存的是{}，所以core_toString存的就是json的toString这个函数，用这个函数call上要判断的这个obj

{}.toString.call([]); //[object Array]
{}.toString.call([]) == '[object Array]'; //true
{}.toString.call(new Date); //[object Date]

//class2type的值在jq后面有定义，是这样定义的
jQuery.each("Boolean Number String Function Array Date RegExp Object Error".split(' '), function(){
	class2type["[object " + name + "]"] = name.toLowerCase();
})
```

isPlainObject() 方法：判断是否为对象自变量

```js
//对象自变量就是json或new object形式
var obj = {};
alert($.isPlainObject(obj)); //true
var obj1 = { name: 'hello' };
alert($.isPlainObject(obj1)); //true
var obj2 = new Object();
alert($.isPlainObject(obj2)); //true
$.isPlainObject([]); //false

//$.isPlainObject的实现
function(obj){
	if(jQuery.type(obj) !== "object" || obj.nodeType || jQuery.isWindow(obj)){
		return false;
	}

	try {
		if(obj.constructor && !core_hasOwn.call(obj.constructor.prototype, "isPrototypeof")){
			//core_hasOwn  = class2type.hasOwnProperty
			//其中 class2type被定义为一个json，因此，core_hasOwn就是json的hasOwnProperty属性
			//hasOwnProperty是判断对象下面的属性是不是它自己的
			//只有Object.prototype上有"isPrototypeof"方法，因此，如果一个obj的自己的property有isPrototypeof，那它肯定就是object，否则就不是
			return false;
		}
	} catch(e){
		return false;
	}
	return true;
}
```

isEmptyObject(): 是否为空的对象

```js
//用法
var obj = {name: 'hello'};
$.isEmptyObject(obj); //false;
var obj1 = {};
$.isEmptyObject(obj1); //true;
$.isEmptyObject([]); //true;
function Aaa(){}
var obj2 = Aaa();
$.isEmptyObject(obj2); //true

//实现
function(obj){
	var name;
	for(name in obj){ //如果不是自身之下的属性和方法，那就for in不到
		return false;
	}
	return true;
}
```

error(): 抛出异常

```js
//用法
$.error('这是错误');
//实现
function(msg){
	throw new Error(msg); //抛出自定义异常，开发者给自己留的便签
}
```

parseHTML()方法：解析节点，把字符串转为节点

```js
//用法
var str = '<li></li><li></li><script><\/script>';
$.parseHTML(str); //转成了两个li放在了数组里面
$.parseHTML(str, document, false); //script标签不会被转化，数组里之后li
$.parseHTML(str, document, true); //script标签页被转为节点，放在了数组里面

//parseHTML的实现
function(data, context, keepScripts){
	if(!data || typeof data != "string"){
		return null;
	}
	if(typeof context == "boolean"){
		keepScripts = context;
		context = false;
	}
	context = context || document;

	//rsingleTag = /^<(\w+)\s*\/?>(?:<\/\1>|)$/
	//这个正则匹配的是单标签形式：<p></p>, <div></div>或<img />这种形式
	var parsed = rsingleTag.exec(data),
	    scripts = !keepScripts && [];
	//单标签情况
	if(parsed) {
		return [ context.createElement(parsed[1]) ];
	}

	//多标签情况
	parsed = jQuery.buildFragment([data], context, scripts);
	if(scripts){
		jQuery(scripts).remove();
	}
	return jQuery.merge([], parsed.childNodes);
}
```

parseJSON() 解析JSON

```js
//用法
var str = '{"name": "hello"}';
$.parseJSON(str).name; //hello

//实现
JSON.parse //就是只能解析严格格式JSON的eval
//将JSON解析为字符串
JSON.stringify() //与JSON.parse功能正好相反
```

parseXML() 解析XML

```js
//用法请查看jQuery官网示例
//实现
function(data){
	var xml, tmp;
	if(!data || typeof data != "string"){
		return null;
	}

	try {
		tmp = new DOMParser(); //创建了一个解析XML的实例对象
		xml = tmp.parseFromString(data, "text/xml"); //ie9下，如果传入的data有问题，那么这句代码会报错；其他浏览器不会报错，但是会创建一个parsererror的标签
	} catch(e){
		xml = undefined;
	}

	if(!xml || xml.getElementByTagName("parsererror").length){
		jQuery.error("Invalid XML: " + data);
	}
	return xml;
}
```

$.noop 返回空函数

```js
//实现
function(){}

//用法
//当我们写插件或组件的时候，可能会有一些默认参数
function Aaa(){
	this.defaults = {
		show: $.noop
	}; //默认配置中并没有提供show的具体方法，这时候需要一个空函数，就可以直接使用$.noop，这样可以进行容错处理，不让这个地方不写的话，程序出错
}
Aaa.prototype.init = function(opt){
	$.extend(this.defaults, opt); //用配置参数覆盖默认参数
}
```

globalEval(): 全局解析JS

```js
//使用
function test(){
	//var newVar = true;

	jQuery.globalEval("var newVar = true;");
	//在这里将newVar弄成全局可访问的变量
}
test();
alert(newVar); //true

//实现
function(code){
	var script, indirect = eval;
	code = jQuery.trim(code);

	if(code){
		if(code.indexOf("use strict") === 1){
			script = document.createElement("script");
			script.text = code;
			document.head.appendChild(script).parentNode.removeChild(script);
		} else {
			indirect(code); //这里用了indirect，没有直接用eval，解释可以看下面的例子
		}
	}
}
```

eval与window.eval的差别

```js
function test(){
	eval('var a = 1');
	alert(a);
}
test();
//以上代码会弹出1
//这里是分割线
function test(){
	eval('var a = 1');
}
test();
alert(a);//全局下这个a找不到
//这里是分割线
function test(){
	window.eval('var a = 1');
}
test();
alert(a) //这时候就弹出了1
//这里是分割线
function test(){
	var val = eval;
	val('var a = 1');
}
test();
alert(a); //这时候1也能找到，这说明 var val = eval这种写法与写window.eval作用是一样的，但是直接写eval是不行的

//eval既是js中的一个关键字，也是window下的一个属性。如果直接使用eval，那么会作为关键字使用。关键字只会在局部范围内起作用。但是window.eval是window下的一个属性，可以全局找到，是可以全局解析到的。
```

camelCase() 转驼峰

```js
//把css中的样式转成js可以接受的形式
$.camelCase('margin-top'); //marginTop

//实现
function(string){
	return string.replace(rmsPrefix, "ms-").replace(rdashAlpha, fcamelCase);
	// -ms-transform -> msTransform //ie这里的前缀开头是要小写的
	// -webkit-transform -> WebkitTransform
	// -moz-transform -> MozTransform

	//rmsPrefix = /^-ms-/
	//rdashAlpha = /-([\da-z])/gi //匹配一个横杠加一个字母或数字
	//fcamelCase = function( all, letter ) { return letter.toUpperCase();
	}
}
```

$.nodename() 是否为指定节点名

```js
$(function(){
	$.nodename(document.documentElement, 'html'); //true
	$.nodename(document.body, 'html'); //false
	$.nodename(document.body, 'body'); //true
})

//实现
function(elem, name){
	return elem.nodeName && elem.nodeName.toLowerCase() === name.toLowerCase();
}
```

each()：遍历集合，针对数组、类数组和JSON

```js
//用法
//针对arr, arguments, childNodes, getElementByTagName
var arr = ['a', 'b', 'c', 'd'];
$.each(arr, function(i, value){
	console.log(i);
	console.log(value);
})

var json = { name: 'hello', age: 20 };
$.each(json, function(i, value){
	console.log(i);
	console.log(value);
	return false; //这样会跳出循环
})

//$.each方法的实现
function( obj, callback, args ) { //args是供内部使用的
	var value,
		i = 0,
		length = obj.length,
		isArray = isArraylike( obj ); //也适用于this -> {1: , 2: , 3: , ... , length: }

	if ( args ) { //如果有args，也就是说供内部使用的，走前面的if
		if ( isArray ) {
			for ( ; i < length; i++ ) {
				value = callback.apply( obj[ i ], args );

				if ( value === false ) { //这就是为什么在$.each的循环中写return false会跳出循环
					break;
				}
			}
		} else {
			for ( i in obj ) {
				value = callback.apply( obj[ i ], args );

				if ( value === false ) {
					break;
				}
			}
		}

	} else { //没有args就是供外部使用的，走else
		if ( isArray ) {
			for ( ; i < length; i++ ) {
				value = callback.call( obj[ i ], i, obj[ i ] );

				if ( value === false ) {
					break;
				}
			}
		} else {
			for ( i in obj ) {
				value = callback.call( obj[ i ], i, obj[ i ] );

				if ( value === false ) {
					break;
				}
			}
		}
	}

	return obj;
}
```

$.trim()方法：去前后空格

```js
//用法
var str = '   hello   ';
alert('(' + str + ')');
alert('(' + $.trim(str) + ')');

//实现
function(text){
	return text == null ? "" : core_trim.call(text);
	//core_trim在前面存为core_version.trim
	//core_version在前面存为 "2.0.3" 是一个字符串，所以就是 "".trim 也就是 String.prototype.trim
	//原生的用法 (' hello ').trim()
	// ''.trim.call('  hello  ')
}
```

$.makeArray(): 类数组转真数组，还可以将字符串和json转成数组

```js
//假设页面中有三个div
//用法
window.onload = function(){
	var aDiv = document.getElementsByTagName('div');
	$.makeArray(aDiv); //[div, div, div]
	var str = 'hello';
	$.makeArray(str); //["hello"]
	var num = 123;
	$.makeArray(num); //[123]

	//$.makeArray供内部使用还可以接收第二个参数，一个json，这个json中一定要有length
	$.makeArray(num, {length: 0});// {0: 123, length: 1}
}

//实现
function(arr, results){
	var ret = results || [];
	if(arr != null){
		if(isArraylike(Object(arr))){
			//如果arr传进来的是字符串，那么这里的if会为真，所以下面专门对字符串进行了判断
			jQuery.merge(ret, typeof arr === "string" ? [arr] : arr);
		} else {
			core_push.call(ret, arr);
			//core_push就是[].push，就是Array.prototype.push 即数组的push方法
		}
	}
	return ret;
}
```

$.inArray() 数组版indexOf

```js
//用法
var arr = ['a', 'b', 'c', 'd'];
$.inArray('b', arr); //1
$.inArray('w', arr); //-1

//实现
function(elem, arr, i){
	return arr == null ? -1 : core_indexOf.call(arr, elem, i);
	//core_indexOf就是[].indexOf就是Array.prototype.indexOf
}
```

$.merge() 合并数组，对内也可以转为特殊形式的json

```js
//实现
function(first, second){
	var l = second.length,
		i = first.length,
		j = 0;
	if(typeof l === "number"){
		//走if的 $.merge(['a', 'b'], ['c','d'])
		//$.merge({0: 'a', 1: 'b', length: 2}, {0: 'c', 1: 'd'})
		for(; j < l; j++){
			first[i++] = second[j];
		}
	} else {
		//走else的 $.merge(['a', 'b'], {0: 'c', 1: 'd'})
		while(second[j] != undefined){
			first[i++] = second[j++];
		}
	}
	first.length = i;
	return first;
}
```

$.grep(): 过滤数组

```js
//使用
var arr= [1, 2, 3, 4];
arr = $.grep(arr, function(n, i){
	return n > 2;
});
arr1 = $.grep(arr, function(n, i){
	return n > 2;
}, true); //这里$.grep接受了第三个参数，如果为true，那么过滤的就正好跟不加第三个参数过滤的正好相反
console.log(arr); //[3, 4]
console.log(arr1); //[1, 2]

//实现
function(elems, callback, inv){
	var retVal,
		ret = [],
		i = 0,
		length = elems.length;
		inv = !!inv;
	for(; i < length; i++){
		retVal = !!callback(elems[i], i);
		if(inv !== retVal) {
			ret.push(elems[i]);
		}
	}
	return ret;
}
```

$.map(): 映射新数组

```js
//使用
var arr = [1, 2, 3, 4];
arr = $.map(arr, function(n, i){
	return n + 1;
})
console.log(arr); //[2, 3, 4, 5]

arr1 = $.map(arr, function(n, i){
	return [n + 1];
})
arr1 = [2, 3, 4, 5]; //而不是想象中的[[2], [3], [4], [5]] 因为在源码中最后返回的时候，使用了 [].concat.apply()

//实现
function(elems, callback, arg){
	var value,
		i = 0,
		length = elems.length,
		isArray = isArraylike(elems),
		ret = [];
	if(isArray){
		for(; i < length; i++){
			value = callback(elems[i], i, arg);
			if(value != null){
				ret[ret.length] = value;
			}
		}
	} else {
		for(i in elems){
			value = callback(elems[i], i, arg);
			if(value != null){
				ret[ret.length] = value;
			}
		}
	}

	//将嵌套数组扁平化
	return core_concat.apply([], ret);
	//core_concat就是[].concat，即Array.prototype.concat
	//[].concat(1) -> [1]
	//[].concat([1]) -> [1]
}
```

guid的作用

```js
//<input type="button" value="点击">
//<input type="button" value="取消绑定">

$(function(){
	function show(){
		alert(this);
	}

	$('input:eq(0)').click(show); //show是一个事件函数，点击弹出第一个button
	$('input:eq(1)').click(function(){
		$('input:eq(0)').off();
	})
})
```

```js
$(function(){
	function show(){
		alert(this);
	}

	$('input:eq(0)').click($.proxy(show, window)); //通过$.proxy来改变show中this的指向
	//点击弹出的是window
	//当用$.proxy改变show中this指向的时候，show就不是事件函数, $.proxy(show, window)才是事件函数，这时候取消事件的时候，直接用下面的方式还可以取消绑定，这就是guid在起作用
	$('input:eq(1)').click(function(){
		$('input:eq(0)').off(); //这个时候，尽管show被嵌套了，但是还是能够被取消
	})
})
```

$.proxy(): 改变this指向

```js
function show(){
	alert(this);
}
show(); //弹出window
$.proxy(show, document); //这时候指向改了，但是没有执行
$.proxy(show, document)(); //弹出document

function show(n1, n2){
	alert(n1);
	alert(n2);
	alert(this);
}
$.proxy(show, document)(3, 4); //依次弹出3，4，document
//也可以像下面这样传参
$.proxy(show, document, 3, 4);
//还可以这样传参
$.proxy(show, document, 3)(4);

var obj = {
	show: function(){
		alert(this);
	}
};
$(document).click(obj.show); //这时候弹出document
$(document).click($.proxy(obj.show, obj)); //弹出obj
//简写写法
$(document).click($.proxy(obj, 'show')); //与上面的一句是一样的
```

```js
//$.proxy的实现
function(fn, context){
	var tmp, args, proxy;

	//支持简写方式 $(document).click($.proxy(obj, 'show'));
	if(typeof context === "string"){
		tmp = fn[context];
		context = fn;
		fn = tmp;
	}

	if(!jQuery.isFunction(fn)){
		return undefined;
	}

	args = core_slice.call(arguments, 2); //前两个参数，对应着fn和context不要
	proxy = function(){
		return fn.apply(context || this, args.concat(core_slice.call(arguments)));
	};

	proxy.guid = fn.guid = fn.guid || jQuery.guid++;

	return proxy;
}
```

☆ $.access(): 多功能值操作（供内部使用）

> 这里看得不是很明白


```js
//access的作用

//以下方法具有 设置set 和 获取get的功能，就是根据参数的不同自行判断
$().css();
$().attr();

//<div id="div1" style="width: 100px; height: 100px; background: red">aaa</div>
$(function(){
	console.log($('#div1').css('width')); //获取 100px
	$('#div1').css('background', 'yello'); //设置
	$('#div1').css({ background: 'green', width: '300px' }) //设置多个值
})
```

```js
//实现
function(elems, fn, key, value, chainable, emptyGet, raw){
	//chainable为true就代表要设置
	//chainable为false就代表要获取
	var i = 0,
		length = elems.length,
		bulk = key == null;
	if(jQuery.type(key) === "object"){
		//针对这种情况：
		//$('#div1').css({ background: 'green', width: '300px' })
		chainable = true;

		for(i in key){
			jQuery.access(elems, fn, i, key[i], true, emptyGet, raw);
		}
	} else if(value !== undefined){ //设置一组值的情况
		chainable = true;

		if(!jQuery.isFunction(value)){
			raw = true;
		}

		if(bulk){ //如果没有key值，那就不是设置什么东西，而是单纯的回调函数
			if(raw){ //如果value是字符串，不是函数
				fn.call(elems, value); //那么就把value传到回调函数fn里面就行了
				fn = null;
			} else { //如果value是函数
				bulk = fn;
				fn = fn(elem, key, value){
					return bulk.call(jQuery(elem), value);
				} //这时候fn并不执行，只是又套了一层，将值为函数的value传到fn里面，然后到下面再执行
			}
		}

		if(fn){ //
			for(; i < length; i++){
				fn(elems[i], key, raw ? value : value.call(elems[i], i, fn(elems[i], key)));
			}
		}
	}

	return chainable ?
			elems :
			bulk ?
				fn.call(elems) :
				length ? fn(elems[0], key) : emptyGet;
}
```

$.now()：获取当前时间

```js
alert($.now());
//(new Date()).getTime()

//实现
Date.now
```

$.swap(): CSS交换（供内部使用）

```js
$(function(){
	//在没有添加display: none之前，jq和原生方式都可以获取到样式
	alert($('#div1').width()); //100
	alert($('#div1').get(0).offsetWidth); //100

	//添加display: none之后
	//如果元素隐藏了，添加了样式 display: none
	//那么原生的方式就获取不到样式了
	alert($('#div1').get(0).offsetWidth); //0
	alert($('#div1').width()); //100 jq还是可以获取到隐藏元素的值

	//jq怎么能获取到隐藏元素的样式的呢？它是这样做的
	//将display设置为block，然后再添加两个样式：visibility: hidden; position: absolute; 这个时候跟display: none的效果是一样的，但是这时候就可以获得样式了

	//jq中先将老样式存起来 → 然后给元素加上新样式，在这种状态下去获取样式 → 再把正确的样式还原回来 这时候就利用了swap

})

//<div id="div1" style="width: 100px; height: 100px; background: red; display: none">aaa</div>
```

```js
//实现
//结合上面的例子来看
function(elem, options, callback, args){
	var ret, name,
		old = {}; //old就是存储老的样式数据的

	for(name in options){
		//先把老的存起来
		old[name] = elem.style[name];
		elem.style[name] = options[name];
	}

	//样式变了，就可以获取到值了
	ret = callback.apply(elem, args || []);

	//然后再把老的样式还原回来
	for(name in options){
		elem.style[name] = old[name];
	}

	return ret;
}
```

isArraylike: 判断是否为类数组

```js
//实现
function isArraylike(obj){
	var length = obj.length,
		type = jQuery.type(obj);
	if(jQuery.isWindow(obj)){
		return false;
	}
	if(obj.nodeType === 1 && length){
		return true;
	}
	return type === "array" || type !== "function" && (length === 0 || typeof length === "number" && length > 0 && (length - 1) in obj);
}
```

###(2880, 3042) Callbacks : 回调对象：对函数的统一管理

```js
//基本使用
function aaa(){
	alert(1);
}
function bbb(){
	alert(2);
}

function ccc(){
	alert(3);
}

var cb = $.Callback(); //创建回调对象
cb.add(aaa);
cb.add(bbb);
cb.fire(); //弹出1、2 //这很类似于JS中的绑定事件

document.addEventListener('click', function(){ alert(1); }, false);
document.addEventListener('click', function(){ alert(2); }, false);
document.addEventListener('click', function(){ alert(3); }, false);
//在document上一点击，弹出1/2/3，这就是绑定事件的特点
```

```js
function aaa(){
	alert(1);
}
(function(){
	function bbb(){
		alert(2);
	}
})();

aaa(); //1能弹出来
bbb(); //2不能弹出来
```

```js
//解决上面的问题
var cb = $.Callbacks();
function(aaa){ alert(1); }
cb.add(aaa);
(function(){
	function bbb(){
		alert(2);
	}
	cb.add(bbb);
})
cb.fire(); //弹出1/2
```

```js
jQuery.Callbacks = function(options){
	// options
	/*
	once -> 作用到 fire 上，只进行一次for循环
	memory -> 作用到 add 上，通过add调用 fire 去执行函数
	unique -> 作用到 add 上，如果传入unique，那么重复的就不让添
	stopOnFalse -> 作用到 fire 中的 for 循环，如果在函数中遇到return false，就立即跳出循环
	*/
}
//公用方法接口
/*
add -> push到数组list里面
remove -> 与add相对 对数组list进行指定位置的删除splice
has -> 判断有没有
empty -> 清空整个list数组
disable -> 全部禁止
disabled -> 判断现在是不是全部禁止了
lock -> 锁住
locked
fireWith
fire -> 调用firewith -> 调用私有的fire函数：for循环数组list
fired
*/
```

```js
//参数once
function aaa(){
	alert(1);
}
function bbb(){
	alert(2);
}

var cb = $.Callback("once"); //传入参数once，那么fire只能触发一次
cb.add(aaa, bbb); //这样写也可以
cb.fire();
cb.fire(); //第二次不再触发
```

```js
//参数memory
function aaa(){
	alert(1);
}
function bbb(){
	alert(2);
}

var cb = $.Callback("memory"); //传入参数memory，不管在fire前面add还是后面add的，都能触发
cb.add(aaa);
cb.fire(); //如果不加参数，2弹出出来；如果加上参数memory，这里的2就弹出来了
cb.add(bbb);
```

```js
//参数unique
function aaa(){
	alert(1);
}

var cb = $.Callback("unique"); //传入参数unique，去重
cb.add([aaa, bbb]); //这样写也可以
cb.fire(); //默认不传入unique，会弹出两次2；传入参数unique，只弹出一次1
```

```js
//参数stopOnFalse
function aaa(){
	alert(1);
	return false;
}
function bbb(){
	alert(2);
}

var cb = $.Callback("stopOnFalse"); //传入参数once，那么fire只能触发一次
cb.add(aaa);
cb.add(bbb);
cb.fire(); //默认情况下，遇到return的false，没影响；但是如果传入了stopOnFalse，那么就只弹1不弹2了
```

```js
//参数可以任意组合
var cb = $.Callbacks("once memory");
cb.add(aaa);
cb.fire();
cb.add(bbb);
cb.fire();
//1、2会各弹一次
```

```js
//fire可以接收参数
function aaa(n){
	alert('aaa' + n);
}
function bbb(n){
	alert('bbb' + n);
}
cb.add(aaa, bbb);
cb.fire('hello'); //分别弹出 aaaHello 和 bbbHello
```

```js
//针对非空白的正则
core_rnotwhite = /\S+/g
//jQuery中针对options的操作
var optionsCache = {}; //建立了一个options的缓存

function createOptions(options){
	var object = optionsCache[options] == {};
	jQuery.each(options.match(core_rnotwhite) || [], function(_, flag){ //_这里只是用来占位的，没有实际用处
		object[flag] = true;
	});
	return object;
}

//举例来讲，options如果是"once memory"，那么最终optionsCache就会变成下面这样
/*
optionsCache = {
	"once memory": {
		"once": true,
		"memory": true
	}
}
*/
```

```js
//回调函数中的一些特殊情况
//下例是针对Callback源码中的stack的作用
var bBtn = true;
function aaa(){
	alert(1);
	//cb.fire(); //这样会造成死循环；而且bbb永远也走不到

	if(bBtn){
		cb.fire(); //弹出1/2/1/2，这说明先走alert(1)，然后走了alert(2)，然后再回来走的aaa中的cb.fire()，再弹出1，然后再弹出2，由于bBtn这时候已经变成了false，所以aaa中的if不会再进去了，所以就不会再继续fire下去
		//所以我们发现，这里的cb.fire()实际是被放到了运行函数的队列当中
		bBtn = false;
	}
}

function bbb() { alert(2); }

var cb = $.Callbacks();
cb.add(aaa);
cb.add(bbb);

cb.fire();
```

```js
function aaa(){ alert(1); }
function bbb() { alert(2); }

var cb = $.Callback('once');
cb.add(aaa);
cb.fire(); //只会弹出1，然后下面即使加入bbb，也不会弹出2。因为当初传入了once
cb.add(bbb);
cb.fire(); //这个不会触发，因为前一次cb触发过之后，由于once就不再触发了
```

```js
function aaa(){ alert(1); }
function bbb() { alert(2); }

var cb = $.Callback('once memory');
cb.add(aaa);
cb.fire(); //弹出1/2
cb.add(bbb);
cb.fire(); //这里就相当于cb.fire([]) //看Callbacks源码，这时候的list数组被清空了
```

```js
//disable和lock的区别
//disable
function aaa(){ alert(1); }
function bbb(){ alert(2); }

var cb = $.Callbacks('memory');

cb.add(aaa);

cb.fire();

cb.disable(); //禁止所有操作，下面的都不起作用了

cb.add(bbb);
cb.fire()
//以上代码，只弹1，2不弹
```

```js
//disable和lock的区别
//lock
function aaa(){ alert(1); }
function bbb(){ alert(2); }

var cb = $.Callbacks('memory');

cb.add(aaa);

cb.fire();

cb.lock(); //锁住，只会把后面的fire锁住，其他操作不会禁止

cb.add(bbb); //因为有memory，所以2也弹出来了

cb.fire() //这个fire没执行
//以上代码弹出1/2
```

###(3043, 3183) Deferred : 延迟对象：对异步的统一管理

```js
/*
jQuery.extend({
	Deferred: function(){},
	when: function(){}
});
*/

//$.Deferred(); // 基于$.Callbacks开发的
//$.when();

var cb = $.Callbacks();

setTimeout(function(){
	alert(111);
	cb.fire();
}, 1000);
cb.add(function(){
	alert(222);
})
//程序先弹出111，然后弹出222

// ============================
var dfd = $.Deferred();
setTimeout(function(){
	alert(111);
	dfd.resolve();
}, 1000);

dfd.done(function(){
	alert(222);
})
//以上这一块代码也是先弹出111，然后弹出222

// ============================
var dfd = $.Deferred();
setTimeout(function(){
	alert(111);
	dfd.reject();
}, 1000);

dfd.fail(function(){
	alert(222);
})
//以上这一块代码也是先弹出111，然后弹出222

// ============================
var dfd = $.Deferred();
setTimeout(function(){
	alert(111);
	dfd.notify();
}, 1000);

dfd.progress(function(){
	alert(222);
})
//以上这一块代码也是先弹出111，然后弹出222
```

```js
$.ajax({
	url: 'xxx.php',
	success: function(){
		alert('成功');
	},
	error: function(){
		alert('失败');
	}
});

//有了延迟对象之后，可以写成下面这样
$.ajax('xxx.php')
	.done(function(){ alert('成功'); })
	.fail(function(){ alert('失败'); });
```

```js
//源码中延迟对象的实现分析

function(func){
	var tuples = [
		["resolve", "done", jQuery.Callbacks("once memory"), "resolved"],
		["reject", "fail", jQuery.Callbacks("once memory"), "rejected"],
		["notify", "progress", jQuery.Callbacks("memory")]
	],
	//以上这一组映射关系中：
	//数组中的第一项"resolve"、"reject"和"notify"调用的就是回调函数中的fire
	//数组中的第二项"done"、"fail"和"progress"对应的就是回调函数中的add
	//接下来我们看，这些与回调函数中的fire和add是怎么映射上的：

	//...中间的一些代码先不看

	jQuery.each(tuples, function(i, tuple){
		var list = tuple[2], //tuple[2]就是回调对象
			stateString = tuple[3];

		promise[tuple[1]] = list.add; //promise就是那个延迟对象，现在就是分别将回调对象的add方法付给了延迟对象的done、fail或progress属性
	})

	//...中间的一些代码先不看
	deferred[tuple[0]] = function(){
		deferred[tuple[0] + 'With'](this === deferred ? promise : this, arguments);
		return this;
	};
	deferred[tuple[0] + 'With'] = list.fireWith; //我们这里会看到，延迟对象的resolveWith、rejectWith和notifyWith调用的都是list.fireWith，也就是回调对象的fireWith方法（fireWith跟fire其实是一回事，只不过fireWith带传参）
}
```

```js
//为什么resolve和reject对应的回调函数中添加了once，但是notify却没有添加
var dfd = $.Deferred();
setInterval(function(){
	alert(111);
	dfd.resolve();
}, 1000);
dfd.done(function(){
	alert('成功');
})
//以上代码，弹出111，然后弹出“成功”，然后就一直弹111，“成功”不再弹出了。回调函数中的“once”起了作用

//==============================
var dfd = $.Deferred();
setInterval(function(){
	alert(111);
	dfd.reject();
}, 1000);
dfd.fail(function(){
	alert('成功');
})
//以上代码，弹出111，然后弹出“成功”，然后就一直弹111，“成功”不再弹出了。回调函数中的“once”起了作用

//==============================
var dfd = $.Deferred();
setInterval(function(){
	alert(111);
	dfd.notify();
}, 1000);
dfd.progress(function(){
	alert('进行中');
})
//以上代码，111和“进行中”会一直触发，因为回调函数中没有“once”
```

```js
//看看下面代码会发生什么情况
$(function(){
	var cb = $.Callbacks('memory');

	cb.add(function(){
		alert(1);
	});
	cb.fire();
	$('input').click(function(){
		cb.add(function(){
			alert(2);
		});
	});
});

//以上代码，在没有点击input的时候，先弹出1
//然后一点击input，就会弹出2，因为回调函数那里有memory参数。也就是说，有memory之后，一旦add了，那么就会立即触发
```

```js
$(function(){
	var dfd = $.Deferred();
	setTimeout(function(){
		alert(111);
		dfd.resolve();
	}, 1000);

	dfd.done(function(){
		alert('aaa');
	})

	$('input').click(function(){
		dfd.done(function(){
			alert('bbb');
		});
	});
})
//以上代码先弹出111，再弹出aaa，然后点击按钮之后，bbb立即弹出
```

```js
//继续看jQuery中的Deferred源码
//其中done、fail、progress放在了promise这个对象里面，对应的是回调的add
//resolve、reject和notify放在了deferred这个对象里面，对应的是回调的fire

//promise和deferred都是延迟对象，但是有区别

/* promise对象中的方法
state
alway
then
promise
pipe //promise.pipe = promise.then //说明promise下面的pipe和then的代码是同一套，但是写了两种，就是因为两者的功能不同
done
fail
progress
*/

deferred = {};//一开始deferred对象是空的，后续代码通过jQuery.each遍历，添加了一些方法

/*deferred对象中的方法
resolve
reject
notify
*/

promise.promise(deferred); //将deferred对象作为参数传到了promise对象的promise方法中

//接下来看promise下面的promise方法：
function(obj){
	return obj != null ? jQuery.extend(obj, promise) : promise;
}
//现在promise有参数为deferred，然后就走了jQuery.extend(deferred, promise)，也就是promise继承给deferred对象，通过这一句话，就把原本promise对象下的所有的方法都给了deferred，于是deferred就拥有了下面这么多方法：
/*
resolve
reject
notify
state
alway
then
promise
pipe
done
fail
progress
*/
//我们发现deferred对象比promise对象多出来三个方法 resove, reject和notify，这三个方法其实就是三个状态。
```

```js
function aaa(){
	var dfd = $.Deferred();
	setTimeout(function(){
		dfd.resolve();
	}, 1000);

	return dfd;
}

aaa().done(function(){
	alert('成功');
}).fail(function(){
	alert('失败');
})
//以上代码1秒钟之后弹成功

//=============================

function aaa(){
	var dfd = $.Deferred();
	setTimeout(function(){
		dfd.resolve();
	}, 1000);

	return dfd;
}

var newDfd = aaa();

newDfd.done(function(){
	alert('成功');
}).fail(function(){
	alert('失败');
})
newDfd.reject();
//这一下弹出来的结果就是“失败”，这是因为，还没等1秒之后，这个newDfd的状态就已经变成reject了，也就是，在1秒钟之前，延迟对象的状态就被改变了。失败一触发，就不会再走其他的了。这说明我们的状态是很容易被修改掉的

//=============================
//那么怎么样可以让延迟对象的状态不被修改掉

function aaa(){
	var dfd = $.Deferred();
	setTimeout(function(){
		dfd.resolve();
	}, 1000);

	return dfd.promise(); //使用这种方式，状态不会被改变
}

var newDfd = aaa();

newDfd.done(function(){
	alert('成功');
}).fail(function(){
	alert('失败');
})
newDfd.reject();

//这样写会弹出“成功”，而且还会报错，就是说你不能在修改延迟对象的状态了

//我们在前面的分析中看到，deferred对象下面有那么多方法，其中resolve、reject和notify都有，那么你调用reject方法，它肯定就会执行，这样状态就改变了。
//但是promise下面根本就没有表示状态的resolve、reject和notify方法，所以也就改变不了状态了。dfd.promise()不写参数，调用的promise方法后，返回的就是promise对象
```

```js
//延迟对象的状态
function aaa(){
	var dfd = $.Deferred();

	alert(dfd.state());

	setTimeout(function(){
		dfd.resolve();
		alert(dfd.state());
	}, 1000);
	return dfd.promise();
}

var newDfd = aaa();

newDfd.done(function(){
	alert('成功');
})
//以上代码弹出 pending，然后弹出“成功”，然后弹出“resolved”
```

```js
//继续看jQuery源码中，对状态state的处理
if(stateString){ //在tuples中，只有resolve和reject这两个数组最后有resolved和rejected这样的stateString，在nofity这里是没有的，所以，只有在状态完成、未完成的情况下会进到if里面
	list.add(function(){
		state = stateString;
	}, tuples[i^1][2].disable, tuples[2][2].lock);
	//在回调对象中一下子又添加了好几个函数，其中第一个function(){ state = stateString; } 这个没什么好说的；后面的两个又代表什么意思呢？
	//一旦触发了未完成，就不能再触发已完成，就是通过后面这两个函数控制的。
	// ^ 这个是个“位运算符” 0 ^ 1会返回1；1 ^ 0会返回0
	//所以，如果i为0，也就是对应着done的话，那么tuples[0^1][2]就是tuples[1][2]即fail的功能全部禁用掉（disable了）；同理，如果为当前状态为fail的话，那么done就会被全部disable掉，然后tuples[2][2].lock指的就是progress不能再被fire了，状态已经完成了。
}
```

```js
//always方法，就是不管成功还是失败都触发，所以是done和fail写在一起了
function(){
	deferred.done(arguments).fail(arguments);
	return this;
}

//==================
var dfd = $.Deferred();
setTimeout(function(){
	dfd.reject();
	//dfd.resolve();
}, 1000);
dfd.always(function(){
	alert('hello');
});
//以上代码，无论写的是dfd.reject()还是dfd.resolve()都会弹出hello
```

```js
//then方法
var dfd = $.Deferred();
setTimeout(function(){
	dfd.reject();
	//dfd.resolve();
}, 1000);
dfd.then(function(){
	alert(1); //对应成功的回调
}, function(){
	alert(2); //对应reject的回调
}, function(){ //对应progress的回调

}); //后面两个参数可省

//======================
//还记得回调函数里面的fire是可以传参吗？因为dfd是基于Callbacks写的，所以也可以传参

var dfd = $.Deferred();
setTimeout(function(){
	dfd.reject('hi'); //这里就是利用了fire能传参
}, 1000);
dfd.then(function(){
	alert(1);
}, function(){
	alert(arguments[0]);
});
//弹出hi
```

```js
var dfd = $.Deferred();
setTimeout(function(){
	dfd.resolve('hi');
}, 1000);
var newDfd = dfd.pipe(function(){
	return arguments[0] + '妙味'; //这里的arguments[0]其实就是'hi'
});
newDfd.done(function(){
	alert(arguments[0]); //弹出‘hi妙味’
})

//pipe接收参数也是三个，第一个是完成，第二个是未完成，第三个是进行时。不同的是pipe各个函数的返回值是会作为新的延迟对象的参数，然后这个新的延迟对象一旦done、fail的话，那么就会立即执行，因为对应的resolve和reject是在源码中执行的
//如果pipe中直接返回函数，那么就会走
/*
returned.promise()
	.done(newDefer.resolve)
	.fail(newDefer.reject)
	.progress(newDefer.notify)
*/
//如果pipe返回的是字符串，那么就会走
/*
newDefer(action + "With")(this === promise ? newDefer.promise() : this, fn ? [returned] : arguments);
*/
//action就是状态，所以如果pipe返回的是字符串，还是会执行状态，只不过有returned就走returned，没有returned就还是走arguments
```

$.when的使用与实现

```js
//$.when是延迟对象的辅助方法

/*
var dfd = $.Deferred();

dfd.done();

$.when().done();
$.when().fail();
$.when().then();
*/

//$.when()这个方法的返回值就是延迟对象，所以后面可以跟done, fail, then
//$.when的源码最后返回的就是deferred.promise()，就是返回了一个外部不能更改状态的延迟对象

//dfd.done()与$.when().done有什么差别呢？
//$.when()可以对多个延迟对象的成功失败进行整体操作

function aaa(){
	var dfd = $.Deferred();
	dfd.resolve();
	return dfd;
}
aaa().done(function(){
	alert('成功');
})
//以上写法只针对一个延迟对象

//=========================
function aaa(){
	var dfd = $.Deferred();
	dfd.resolve();
	return dfd;
}
function bbb(){
	var dfd = $.Deferred();
	dfd.resolve();
	return dfd;
}
//我的需求是等aaa和bbb的延迟对象都完成之后，再弹出“成功”
$.when(aaa(), bbb()).done(function(){
	alert('成功');
}); //这就代表要等aaa()和bbb()这两个延迟对象都完成之后，再弹出成功

//=========================
function aaa(){
	var dfd = $.Deferred();
	dfd.reject();
	return dfd;
}
function bbb(){
	var dfd = $.Deferred();
	//dfd.reject();
	return dfd;
}

$.when(aaa(), bbb()).fail(function(){
	alert('失败');
}); //只要aaa()或bbb()这两个延迟对象中有一个reject了，那么就会触发这里的fail中的函数
```

```js
//$.when的源码中有计数器remaining，记录有多少未完成的延迟对象
/*
$.when(aaa(), bbb(), ccc(), ddd()).done(function(){
	alert(1);
})
其中aaa(), bbb(), ccc(), ddd()都是$.when的参数，分别是arguments[0], arguments[1], arguments[2], arguments[3]。这些都是延迟对象，每当一个argument完成之后，就会触发自己的done，这时候计数器就会减减。当所有的arguments都done了之后，计数器就会为0。一旦计数器为0，$.when的源码中有一个$.Deferred，一旦计数器为0，我们就去触发这个延迟对象的resolve。然后return这个$.Deferred的延迟对象。所以$.when().done()这时候就会触发。
*/
```

```js
function aaa(){
	var dfd = $.Deferred();
	dfd.resolve();
	return dfd;
}
function bbb(){
	var dfd = $.Deferred();
	dfd.reject();
	return dfd;
}
$.when(aaa(), bbb()).done(function(){
	alert('成功');
}).fail(function(){
	alert('失败');
})
//上述代码弹出“失败”。

// $.when()中的参数必须是延迟对象

//===============================
function aaa(){
	var dfd = $.Deferred();
	dfd.resolve();
	return dfd;
}
function bbb(){
	var dfd = $.Deferred();
	dfd.reject();
	//return dfd;
	//这里的bbb并没有返回延迟对象
}
$.when(aaa(), bbb()).done(function(){
	alert('成功');
}).fail(function(){
	alert('失败');
})
//这时候代码会弹出“成功”，因为当dfd没有在bbb中return，那么$.when就会将bbb看成是一个普通函数了，这时候就会跳过它。因为跳过了bbb，所以计数器也不再记它了，而且aaa()走成功了，所以会弹出“成功”

//如果是下面这种情况
$.when(123, 123).done(function(){
	//arguments[0] => 123
	// $.when中的参数不是延迟对象的时候，只能作为传参的作用
	alert('成功');
}).fail(function(){
	alert('失败');
}) //这时候也会弹出“成功”，即使$.when()当中什么参数都没写，也还是会弹出“成功”
//如果$.when()中的参数不是延迟对象，就会被自动跳过
```

###(3184, 3295) support : 功能检测

```js
//jQuery.support是这样定义的
jQuery.support = (function(support){
	//...
	return support;
})({})
//匿名函数自执行，将空对象传入，然后return出来的也是个JSON
```

```js
$(function(){
	for(var attr in $.support){
		$('body').append('<div>' + attr + ': ' + $.support[attr] + '</div>')
	}
})
//以上代码输出出来大概是这样的形式：
/*
checkOn: true
optSelected: true
reliableMarginRight: true
boxSizingReliable: true
pixelPosition: true
noCloneCheck: true
optDisabled: true
radioValue: true
checkClone: true
focusinBubbles: false
clearCloneStyle: true
cors: true
ajax: true
boxSizing: true
*/
```

###(3308, 3652) data() : 数据缓存

```js
//data()的使用与attr()和prop()非常类似
/*
<div id="div1"></div>
*/
$(function(){
	// $('#div1').attr('name', 'hello');
	// alert($('#div1').attr('name'));
	document.getElementById('div1').setAttribute('name', 'hello');
	alert(document.getElementById('div1').getAttribute('name'));

	// $('#div1').prop('name', 'hello');
	// alert($('#div1').prop('name'));
	document.getElementById('div1')['name'] = 'hello';
	alert(document.getElementById('div1')['name']);

	//以上两种方式不适合挂载大量数据，使用data()方法可以解决这个问题
	$('#div1').data('name', 'hello');
	alert($('#div1').data('name'));
})
```

```js
//内存泄漏问题，JS自带垃圾回收机制

//JS有几种情况会引起内存泄漏
//DOM元素与对象之间互相引用，大部分浏览器就会出现内存泄漏

var oDiv = document.getElementById('div1');
var obj = {};

oDiv.name = obj;
obj.age = oDiv;
//以上情况会引起内存泄漏

$('#div1').attr('name', obj); //如果出现这种情况，万一后面obj又来引用了$('#div1')，那么就产生了内存泄漏，所以现在要解决这个问题，利用data()就没有问题

$('#div1').data('name', obj); //这时候不用担心内存泄漏问题，这就是data()方法与attr()和prop()的区别，它可以防止内存泄漏
```

```js
//data()防止内存泄漏的原理
//data()利用了中介，将DOM和对象间接联系到一起
//这个中介就是cache这个对象

$('#div1').data('name', obj);
$('body').data('age', obj);

//首先$('#div1').data('name', obj)
//第一步，找到div1的DOM节点，然后在上面生成一个自定义属性，这个自定义属性的值设为一个数字标识。例如：
//<div xxx="1"></div>
//<body xxx="2"></div>
//这时候cache可能变成下面这种形式
var cache = {
	1: {
		name: obj
	},
	2: {
		age: obj
	}
}
```

```js
//jQuery源码中data()方法的实现，简化版
jQuery.extend({
	acceptData
	hasData
	data
	removeData
	_data //前面带下划线，表示私有的。并不是对外的接口，是针对内部的
	_removeData //前面带下划线，表示私有的
});

jQuery.fn.extend({
	data
	removeData
})
```

```js
$(function(){
	//实例方法
	$('#div1').data('name', 'hello');
	$('#div1').removeData('name');
	alert($('#div1').data('name'));

	//工具方法
	$.data(document.body, 'age', 30);
	alert($.data(document.body, 'age')); //30
	alert($.hasData(document.body, 'age')); //true
	$.removeData(document.body, 'age');
	alert($.data(document.body, 'age'));
	alert($.hasData(document.body, 'age')); //false
})
```

```js
//先讲工具方法
//data_user = new Data(); //这是对外的一个数据缓存对象
//data_priv = new Data(); //这是一个针对内部使用的一个数据缓存对象

//acceptData, data, removeData, hasData这些方法最终调用的还是new Data()对象当中的方法，所以还是要重点了解这个Data对象

Data.prototype = {
	key //分配映射
	set //怎样往cache中设置
	get //怎样从cache中取值
	access //这是个set和get的集合方法，如果传三个参数就会调用set方法；如果传两个参数，就会调用get方法
	remove //怎样去除cache中的数据
	hasData //判断cache中是否有某某数据
	discard //删除cache下面的相应
}
```

```js
//JQ中的源码分析
function Data(){
	Object.defineProperty(this.cache = {}, 0, {
		get: function(){
			return {};
		}
	})
}

//========================
var obj = {name: 'hello'};
Object.freeze(obj); //obj只能获取，不能修改属性值
obj.name = 'hi';
alert(obj.name); //'hello'

//=======================
var obj = {name: 'hello'};
Object.defineProperty(obj, 0, {
	get: function(){
		return {};
	}
});
this.expando = jQuery.expando + Math.random();
//在这里为cache添加了一个属性0，该属性的值是一个空函数，而且是不能被修改的。
//这里的expando其实就是为这个对象所设置的自定义属性，就相当于<div id="div1" jQuery203089541586732714850.884093129098725="1"></div>


//defineProperty的三个参数：第一个是操作的对象，第二个参数0代表在obj下面添加了一个0这个属性，这个属性的值就是get方法所提供的
//通过上面这种方式是只能获取不能设置的，因为没有写set方法
alert(obj[0]);
obj[0] = 123;
alert(obj[0]); //并没有改为123

//什么时候可能会出现cache[0]的情况呢？在源码中
if(!Data.accepts(owner)){ return 0; }
//那么!Data.accepts(owner)是什么情况呢？
return owner.nodeType ? owner.nodeType === 1 || owner.nodeType === 9 : true;
//owner这里就是代表对应的对象，如果ower是node节点的话，如果nodeType为1代表它是个元素，nodeType为9代表它是Document。这些都是在cache中分配1/2/3/4的，除此以外，其他的nodeType是不能分配标示的，也就对应着0，返回空的JSON。如果owner不是node节点，就是普通对象的话，那么是都可以分配数字标识的。
```

```js
try {
	descriptor[this.expando] = {value: unlock};
	Object.defineProperties(owner, descriptor);
} catch(e){
	descriptor[this.expando] = unlock;
	jQuery.extend(owner, descriptor);
}

//之所以要先尝试一下Object.defineProperties方法，如果不行，在使用jQuery.extend的方法是因为使用jQuery.extend这种方法，有可能人们会人为的更改这一属性，但是如果使用Object.defineProperties的方法，这个属性就不能被更改了。
```

```js
//data()方法，设置是会一下子设置多个，但是获取，通常是获取第一个
$(function(){
	$('#div1').data('name', 'hello');
	$('#div1').data('age', '30');

	console.log($('#div1').data('name'));
	console.log($('#div1').data()); //返回{'name': 'hello', 'age': '30'}
})
```

```js
//<div id="div1" data-miaov-all="妙味">aaa</div>
//alert($('#div1').get(0).dataset.miaovAll); //"妙味"
$('#div1').data('name', 'hello');
$('#div1').data('age', '30');
console.log($('#div1').data()); //{name: "hello", age: "30", miaovAll: "妙味"}
//jQuery会将元素上的HTML5的dataset也视为该元素的数据缓存
```

###(3653, 3797) queue() : 队列管理

队列：先进先出

```js
//简化jQuery中queue的源码
jQuery.extend({
	queue
	dequeue
	_queueHooks
})

jQuery.fn.extend({
	queue
	dequeue
	delay
	clearQueue
	promise
})
```

```js
//queue和dequeue，类似于array的push和shift操作，只不过往队列中存的必须是函数
$(function(){
	function aaa(){
		alert(1);
	}
	function bbb(){
		alert(2);
	}

	$.queue(document, 'q1', aaa); //在document上建立了名为q1的队列，队列中存了一个aaa的函数

	$.queue(document 'q1', bbb); //队列中又存入了bbb函数

	//以上两句相当于$.queue(document, 'q1', [aaa, bbb]);

	console.log($.queue(document, 'q1')); //$.queue只写两个参数，就能返回这个队列 [aaa(), bbb()]

	$.dequeue(document, 'q1'); //弹出1 出队不仅是取出位于头部的函数，而且还会调用该函数
	$.dequeue(document, 'q1'); //弹出2
})
```

```js
//实例方法与工具方法是一回事
$(function(){
	function aaa(){
		alert(1);
	}
	function bbb(){
		alert(2);
	}

	$(document).queue('q1', aaa);
	$(document).queue('q1', bbb);

	console.log($(document.queue('q1'))); //[aaa(), bbb()]

	$(document).dequeue(); //弹1
	$(document).dequeue(); //弹2
})
```

```js
//<div id="div1"></div>
//#div {width: 100px; height: 100px; background: red; position: absolute;}
$(function(){
	$('#div1').click(function(){
		$(this).animate({width: 300}, 2000);
		$(this).animate({height: 300}, 2000);
		$(this.animate({left: 300}, 2000));
		//以上运动是依次完成的，而不是同时进行的，这就利用了队列管理
	})
})
```

queue比Deferred更强大。Deferred是针对一个异步的进行管理，queue是对多个异步进行管理。

```js
$(function(){
	$('#div1').click(function(){
		//$(this).animate({width: 300}, 2000).animate({left: 300}, 2000);

		//$(this).animate({width: 300}, 2000).queue('fx',function(){}).animate({left: 300}, 2000);
		//以上在宽度变为300之后，就不再继续执行了，因为在为fx队列添加了一个函数之后，没有出队操作，因此队列后面的函数都不会执行

		$(this).animate({width: 300}, 2000).queue('fx',function(){
			//fx是默认的队列名称
			$(this).dequeue();
		}).animate({left: 300}, 2000);
		//以上这样就可以变完width再改变left了

	})
//=========================
	$('#div1').click(function(){
		$(this).animate({width: 300}, 2000).queue(function(next){
			next(); //与 $(this).dequeue(); 是一回事
		}).animate({left: 300}, 2000);
	})
//==========================
	$('#div1').click(function(){
		$(this).animate({width: 300}, 2000).queue(function(next){
			$(this).css('height', 300);
			next()
		}).animate({left: 300}, 2000);
	})
	//以上代码，#div1先慢慢变成宽300，然后突然变为高300，然后慢慢移动为left 300.
//==========================
//以上效果，利用animate的回调也同样可以实现这个效果
	$('#div1').click(function(){
		$(this).animate({width: 300}, 2000, function(){
			$(this).css('height', 300);
		}).animate({left: 300}, 2000);
	})

//虽然回调也可以做，但是队列管理是更强大的，例如我们做一个更为复杂的
	$('#div1').click(function(){
		$(this).animage({width: 300}, 2000, function(){
			//$(this)css('height', 300);
			var This = this;
			var timer = setInterval(function(){
				This.style.height = This.offsetHeight + 1 + 'px';
				if(This.offsetHeight == 200){
					clearInterval(timer);
				}
			}, 30)
		}).animate({left: 300}, 2000);
	})
	//如果写成上面这个样子，就会发现，当宽度变化之后，高度和left是同时变化的，这是回调中开的是定时器，这个定时器不会影响后续操作，所以定时器运行的时候，后面的left也会操作，但是如果用入队和出队就不会出问题了
//==========================
	$('#div1').click(function(){
		$(this).animate({width: 300}, 2000).queue(function(next){
			var This = this;
			var timer = setInterval(function(){
				This.style.height = This.offsetHeight + 1 + 'px';
				if(This.offsetHeight == 200){
					next();
					clearInterval(timer);
				}
			}, 30);
		}).animate({left: 300}, 2000);
	});
	//这个时候，就是宽变完了变高，高变完了变left
})
```

```js
//队列的delay实例方法
$(function(){
	$('#div1').click(function(){
		$(this).animate({width: 300}, 2000).delay(2000).animate({left: 300}, 2000)
	})
	//delay能让队列延迟执行
})
```

```js
//队列的promise实例方法
$(function(){
	$('#div1').click(function(){
		$(this).animate({width: 300}, 2000).animate({left: 300}, 2000)
	})
	$(this).promise().done(function(){
		alert(123);
	}); //promise的作用就是当所有的运动结束之后，再来调用后面的方法
})
```

###(3803, 4299) attr() prop() val() addClass等:对元素属性的操作

```js
jQuery.fn.extend({
	attr
	removeAttr
	prop
	removeProp
	addClass
	removeClass
	toggleClass
	hasClass
	val
})

jQuery.extend({ //这些工具方法大多是内部使用的，外部使用很少
	valHooks
	attr
	removeAttr
	attrHooks
	propFix
	prop
	propHooks
})
```

```js
//attr, removeAttr, prop, removeProp的基本使用
//<div id="div1">aaaaaaaa</div>
$(function(){
	$('#div1').attr('title', 'hello'); //设置
	alert($('#div1').attr('id')); //获取
	$('#div1').prop('title', 'hello'); //设置
	alert($('#div1').prop('id')); //获取
	//attr和prop的差别：要了解原生js中的setAttribute()、.或[]两种设置属性的方法。

	/*var oDiv = document.getElementById('div1');
	oDiv.setAttribute('title', 'hello');
	oDiv.title = 'hello';
	oDiv['title'] = 'hello';*/

	//针对自定义属性的时候，两种设置会有差别。因为title是元素本身带有的属性，所以两种设置方法没有差别。

	$('#div1').attr('miaov', 'hello'); //属性加上去了
	$('#div1').prop('miaov', 'hello'); //自定义属性在标签上体现不出来

	$('#div1').attr('miaov'); //可以获取自定义属性
	$('#div1').prop('miaov'); //不一定能获取到自定义属性
})
```

```js
//<a id="a1" miaov="妙味" href="miaov.com">aaaaaa</a>
	$('#a1').attr('href'); //得到 miaov.com
	$('#a1').prop('href'); //得到 完整的包含本地的地址
```

```js
//removeAttr, removeProp
$('#div1').attr('miaov', 'hello');
$('#div1').removeAttr('miaov');
$('#div1').attr('miaov'); //为空，因为上面一句已经将miaov属性删掉了
```

```js
//实例方法中的attr其实调用的是工具方法中的attr
//实例方法中的removeAttr调用的是工具方法removeAttr
//实例方法中的prop调用的是工具方法的prop
```
