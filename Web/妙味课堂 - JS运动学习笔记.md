# 妙味课堂 - JavaScript中的运动

##运动的基础原理

- 运动基础
    - 让div运动起来
    - 速度——物体运动的快慢
- 匀速运动
    - 速度不变
- 运动框架
    - 在开始运动时，关闭已有定时器
    - 检测停止条件和执行运动对立（if / else）

- 运动及应用
- 运动框架实例
    - 例子1： “分享到”侧边栏
    - 例子2：淡入淡出的图片
    - 支持不同属性，能让某个值运动起来
        - 用currentStyle代替offset
- 多个物体同时运动
    - 例子：多个div，鼠标移入变宽
        - 单定时器，存在问题
        - 每个div一个定时器
- 多个值同时运动
    - for in的应用
- 运动回调 - 链式运动

- 摩擦运动
    - 逐渐变慢，最后停止
- 缓冲运动
    - 与摩擦力的区别：可以精确地停到指定目标点
    - 距离越远速度越大
        - 速度由距离决定
        - 速度 = (目标值 - 当前值) / 缩放系数
        - Bug：速度取整
        - 值取整

在js中，如何让一个页面元素动起来:

``` html
<input type="button" value="动起来" id="btn" />
<div id="div1"></div>
```

``` css
#div1 { width: 100px; height: 100px; background: red; position: absolute; left: 0px; top: 30px; }
```

``` js
var oBtn = document.getElementById('btn');
var oDiv = document.getElementById('div1');
var iTimer = null;

//点击按钮，让div1横向向右移动
//定时器
oBtn.onclick = function(){
    clearInterval(iTimer);
    iTimer = setInterval(function(){
        //oDiv.style.left = oDiv.offsetLeft + 10 + 'px';
        if(oDiv.offsetLeft == 500){
            clearInterval(iTimer);
        } else {
            oDiv.style.left = oDiv.offsetLeft + 10 + 'px';
        }
    }, 30)
}
```

1. 清除定时器：保证运动过程中只有一个定时器在执行
2. 开启定时器
3. 开始运动（同时在运动中加入判断，以便在需要的时候或者是在满足某个要求的时候停止运动）

**小数的计算精度问题**

``` js
//alert(0.1 + 0.2); // 0.3 -> 0.3000000004
//alert(0.2 + 0.7); //0.9 -> 0.8999999999

//近似值：可能比正确要小，也可能要比正确的大
//近似值：进行“四舍五入”（不是真正的四舍五入）以后可以得到正确值
```

###简单运动的函数封装

``` js
function css(obj, attr){
    if(obj.currentStyle){
        return obj.currentStyle[attr];
    } else {
        return getComputedStyle(obj, false)[attr];
    }
}

function startMove(obj, json, iSpeed, fn){
    clearInterval(obj.iTimer);
    var iCur = 0;
    
    obj.iTimer = setInterval(function(){
    
        var iBtn = true;

        for(var attr in json){
            
            var iTarget = json[attr];
            
            if(attr == 'opacity'){
                iCur = Math.round(css(obj, 'opacity') * 100);
            } else {
                iCur = parseInt(css(obj, attr));
            }
            
            if(iCur != iTarget){
                iBtn = false;
                if(attr == 'opacity'){
                    obj.style.opacity = (iCur + iSpeed) / 100;
                    obj.style.filter = 'alpha(opacity=' + (iCur + iSpeed) + ')';
                } else {
                    obj.style[attr] = iCur + iSpeed + 'px';
                }
            }
        }

        if(iBtn){
            clearInterval(obj.iTimer);
            fn && fn.call(obj);
        }
    
    }, 30);
}
```

###摩擦运动

摩擦，减速：在运动过程中，速度越来越慢

让iSpeed在定时器里面递减即可。iSpeed -= 2; 或 iSpeed /= 2; 或 iSpeed *= 0.9; 但是这种方法不太好控制目标点。

###缓冲运动

越接近目标点，速度越小。速度和距离成正比。

设置 iSpeed = ( 500 - oDiv.offsetLeft) * 0.2; iSpeed = iSpeed > 0 ? Math.ceil(iSpeed) : Math.floor(iSpeed);

**CSS解析和js解析：**

CSS解析是认小数点之后的值的，但是offsetLeft等等这些经过js运算过后的没有单位的值是不认小数点之后的值的（会将有小数点的值进行四舍五入运算）

###运动框架加入缓冲模式

``` js
function css(obj, attr){
    if(obj.currentStyle){
        return obj.currentStyle[attr];
    } else {
        return getComputedStyle(obj, false)[attr];
    }
}

function startMove(obj, json, fn){
    clearInterval(obj.iTimer);
    var iCur = 0;
    var iSpeed = 0; //速度初始化
    
    obj.iTimer = setInterval(function(){
    
        var iBtn = true;

        for(var attr in json){
            
            var iTarget = json[attr];
            
            if(attr == 'opacity'){
                iCur = Math.round(css(obj, 'opacity') * 100);
            } else {
                iCur = parseInt(css(obj, attr));
            }
            
            iSpeed = (iTarget - iCur) / 8;
            iSpeed = iSpeed > 0 ? Math.ceil(iSpeed) : Math.floor(iSpeed);
            
            if(iCur != iTarget){
                iBtn = false;
                if(attr == 'opacity'){
                    obj.style.opacity = (iCur + iSpeed) / 100;
                    obj.style.filter = 'alpha(opacity=' + (iCur + iSpeed) + ')';
                } else {
                    obj.style[attr] = iCur + iSpeed + 'px';
                }
            }
        }

        if(iBtn){
            clearInterval(obj.iTimer);
            fn && fn.call(obj);
        }
    
    }, 30);
}
```

##运动框架的应用

- 运动框架应用
    - 例子：多图片展开、收缩
        - 布局转换
    - 例子：运动的留言本
        - 链式运动
    - 幻灯片
- 返回顶部
    - 注意滚动条拖拽时，清除定时器

###多图展开收缩实例
        
``` html
<ul id="ul1">
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
</ul>
```

``` css
body { margin: 0; }
li { width: 100px; height: 100px; background: red; float: left; list-style: none; margin: 10px 0 0 10px; }
#ul1 { margin: 0; padding: 0; width: 330px; margin: 100px auto 0; position: relative; }
```

``` js
//要先引入startMove函数
/*
元素居中放大：除了要改变元素的宽高以外，还要改变元素定位（left, top）
如果图片放大一倍，那么位移放大的宽高的一半。
元素必须是定位的。
*/

window.onload = function(){

	var oUl = document.getElementById('ul1');
	var aLi = oUl.getElementsByTagName('li');
	var arr = [];
	var zIndex = 1;

	for(var i=0; i<aLi.length; i++){
		arr.push({left: aLi[i].offsetLeft, top: aLi[i].offsetTop});		
	}

	for(var i=0; i<aLi.length; i++){

		aLi[i].index = i;

		//在转化布局的时候，相对用户眼睛的位置保持不变。利用offsetLeft/offsetTop
		//在用js去设置css样式的时候注意：在同一个代码块当中，有些css样式的设置的权限要高于其他的样式。因为在一个代码块中，position = 'absolute'先被解析了，而offsetLeft和offsetTop要经过运算后才解析，所以先定位成了absolute，然后再计算offsetLeft和offsetTop就出现了问题。因此要把offsetLeft和offsetTop的设置放在单独的代码块中先行解析。

		/*
		aLi[i].style.left = aLi[i].offsetLeft + 'px';
		aLi[i].style.top = aLi[i].offsetTop + 'px';
		*/
		aLi[i].style.left = arr[i].left + 'px';
		aLi[i].style.top = arr[i].top + 'px';
		aLi[i].style.position = 'absolute';
		aLi[i].style.margin = '0px';

		aLi[i].onmouseover = function(){

			this.style.backgroundColor = 'green';
			this.style.zIndex = zIndex++;

			startMove(this, {
				width: 200,
				height: 200,
				left: arr[this.index].left - 50,
				top: arr[this.index].top - 50
			});
		}
		aLi[i].onmouseout = function(){
			startMove(this, {
				width: 100,
				height: 100,
				left: arr[this.index].left,
				top: arr[this.index].top
			});
		}

	}

}
```

###带运动效果的留言本

``` html
<textarea id="content" rows="10" cols="50"></textarea>
<input type="button" value="留言" id="btn">
<ul id="ul1"></ul>
```

``` css
#ul1 { margin: 0; position: aboslute; right: 10px; top: 8px; width: 700px; height: 500px; border: 1px solid #000; padding: 10px; overflow: auto; }
li { line-height: 28px; border-bottom: 1px dotted #000; list-style: none; word-break: break-all; overflow: hidden; }
```

``` js
//先引入startMove函数
window.onload = function(){
    
    var oContent = document.getElementById('content');
    var oBtn = document.getElementById('btn');
    var oUl = document.getElementById('ul1');
    
    oBtn.onclick = function(){
    
    	var oLi = document.createElement('li');
    	oLi.innerHTML = oContent.value;
    
    	if(oUl.children[0]){
    		oUl.insertBefore(oLi, oUl.children[0])
    	} else {
    		oUl.appendChild(oLi);
    	}
    
    	var iHeight = parseInt(css(oLi, 'height'));
    	oLi.style.height = '0px'; //初始化样式，然后再变
    	oLi.style.opacity = '0';
    	oLi.style.filter = 'alpha(opacity=0)';
    	startMove(oLi, {
    		height: iHeight,
    		opacity: 100
    	});
    }
}
```

###淘宝幻灯片

``` html
<div id="div1">
	<ul id="ul1">
		<li><img src="1.png"></li>
		<li><img src="2.jpg"></li>
		<li><img src="3.jpg"></li>
		<li><img src="4.jpg"></li>
		<li><img src="5.jpg"></li>
		<li><img src="6.jpg"></li>
	</ul>
	<p>
		<span></span>
		<span></span>
		<span></span>
		<span></span>
		<span></span>
		<span></span>
	</p>
</div>
```

``` css
#div1 { width: 520px; height: 280px; border: 1px solid #000; margin: 100px auto 0; position: relative; overflow: hidden; }
#ul1 { position: absolute; left: 0; top: 0; margin: 0; padding: 0; }
li { list-style: none; float: left; }
img { display: block; }
#div1 p { text-align: center; position: absolute; width: 100%; bottom: 10px; }
#div1 p span { padding: 2px 9px; background: #ccc; border-radius: 50%; margin-left: 5px; cursor: pointer; }
#div1 p span.current { background: #f90; }
```

``` js
window.onload = function(){
	var oDiv = document.getElementById('div1');
	var oUl = document.getElementById('ul1');
	var aLi = document.getElementsByTagName('li');
	var aSpan = oDiv.getElementsByTagName('span');
	var iLen = aLi.length;
	var iWidth = aLi[0].offsetWidth;

	oUl.style.width = iLen * iWidth + 'px';

	for(var i=0; i<iLen; i++){
		aSpan[i].index = i;
		aSpan[i].onclick = function(){

			for(var i=0; i<iLen; i++){
				aSpan[i].className = '';
			}
			this.className = 'current';
			startMove(oUl, {
				left: -this.index * iWidth;
			})
		}
	}

}
```

###带运动的返回顶部

``` html
<body style="height: 2000px">
	<div id="div1"></div>
</body>
```

``` css
#div1 { width: 100px; height: 100px; background: red; position: absolute; right: 0; top: 0; }
```

``` js
window.onload = function(){
	
	var oDiv = document.getElementById('div1');
	var iTimer = null;
	var b = 0;

	setTop();

	window.onscroll = function(){

		//console.log('2');
		if(b != 1){
			//如果b == 1，那么当前的scroll事件被定时器所触发；不过b != 1，那么就是非定时器的其他任何一个操作触发了该事件
			clearInterval(iTimer);
		}
		b = 2;

		setTop();
	}

	oDiv.onclick = function(){
		clearInterval(iTimer);
		var iCur = iSpeed = 0;
		iTimer = setInterval(function(){
			iCur = document.documentElement.scrollTop || document.body.scrollTop;
			iSpeed = Math.floor((0 - iCur) / 8);
			if(iCur == 0){
				clearInterval(iTimer);
			} else {
				document.documentElement.scrollTop = document.body.scrollTop = iCur + iSpeed;
			}
			//console.log('1');
			b = 1;	
		}, 30)				
	}

	function setTop(){
		var scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
		oDiv.style.top = scrollTop + document.documentElement.clientHeight - oDiv.offsetHeight + 'px';

	}
}
```

##图片预加载

- 图片预加载
    - 不直接修改img元素的src，加载完成后，再显示
    - 用到的事件
        - onload：加载完成后显示图片
        - onerror：加载失败时，进行其他处理（跳过、显示信息等）
    - 预判加载——自动加载下一张图片
    - 延迟加载——加载可视区图片，其他图片等进入可视区再加载

我们经常会用下载软件下载电视剧，一个电视剧可以有n集。

1. 先把所有的集数全部下载完成，然后一个个开开心心地看。你真的开心吗？
2. 我们先下一集，然后按完，看完以后再去下下一集，然后再看。
3. 我们先下载第一集，下载完成以后，在看第一集的时候去下载后面的内容。这样，在看前面的内容的时候，把后面的下完，节约了很多时间。

图片预加载就是采用上述第3种方式：在页面刚打开的时候，去加载第一张图片。然后页面加载完成以后，在用户看的时间内，去加载后面的内容。那么我们必须有个工具（迅雷）-> Image对象。

###图片预加载原理

``` html
<img id="img1" src='' />
```

``` js
window.onload = function(){

	var oImage = new Image();
	var oImg = document.getElementById('img1');

	/*
	属性：
		src：当我们给Image对象的src属性赋值一个url的时候，这个Image对象就会去加载url资源。加载完成以后的资源被保存到了浏览器的缓存文件夹里面。下次如果我们要去调用这个url地址的时候，直接是从缓存文件夹读取到的。所以速度很快。
	事件：
		onload：当资源加载完成的时候触发
		onerror：当资源加载失败的时候触发
	*/
	oImage.src = '1.png';
	oImage.onload = function(){
		alert('加载完成');

		document.onclick = function(){
			oImg.src = '1.png';
		}
	}
	oImage.onerror = function(){
		alert('加载出错');
	}

}
```

###图片预加载的应用实例

``` html
<img src="1.jpg" id="img1" style="width: 300px;" />
```

``` js
window.onload = function(){

	var oImg = document.getElementById('img1');
	var oImage = new Image();
	var arr = [
		'2.jpg',
		'3.jpg',
		'4.jpg',
		'5.jpg',
		'6.jpg',
		'7.jpg'
	];
	var iCur = 0;
	var i = 0;

	xunlei();

	oImg.onclick = function(){
		i++;
		if(i < arr.length){
			oImg.src = arr[i];
		}
	}

	function xunlei(){
		oImage.src = arr[iCur];
		oImage.onload = function(){
			iCur++;
			if(iCur < arr.length){
				xunlei();
			}
		}
	}

}
```

###图片的按需加载

``` html
<ul id="ul1">
    <li><img _src="1.jpg" src="white.jpg"></li>
    <li><img _src="2.jpg" src="white.jpg"></li>
    <li><img _src="3.jpg" src="white.jpg"></li>
    <li><img _src="4.jpg" src="white.jpg"></li>
    <li><img _src="5.jpg" src="white.jpg"></li>
    <li><img _src="6.jpg" src="white.jpg"></li>
    <li><img _src="7.jpg" src="white.jpg"></li>
    <li><img _src="1.jpg" src="white.jpg"></li>
    <li><img _src="2.jpg" src="white.jpg"></li>
    <li><img _src="3.jpg" src="white.jpg"></li>
    <li><img _src="4.jpg" src="white.jpg"></li>
    <li><img _src="5.jpg" src="white.jpg"></li>
    <li><img _src="6.jpg" src="white.jpg"></li>
    <li><img _src="7.jpg" src="white.jpg"></li>
    <li><img _src="1.jpg" src="white.jpg"></li>
    <li><img _src="2.jpg" src="white.jpg"></li>
    <li><img _src="3.jpg" src="white.jpg"></li>
    <li><img _src="4.jpg" src="white.jpg"></li>
    <li><img _src="5.jpg" src="white.jpg"></li>
    <li><img _src="6.jpg" src="white.jpg"></li>
    <li><img _src="7.jpg" src="white.jpg"></li>
    <li><img _src="1.jpg" src="white.jpg"></li>
    <li><img _src="2.jpg" src="white.jpg"></li>
    <li><img _src="3.jpg" src="white.jpg"></li>
    <li><img _src="4.jpg" src="white.jpg"></li>
    <li><img _src="5.jpg" src="white.jpg"></li>
    <li><img _src="6.jpg" src="white.jpg"></li>
    <li><img _src="7.jpg" src="white.jpg"></li>
    <li><img _src="1.jpg" src="white.jpg"></li>
    <li><img _src="2.jpg" src="white.jpg"></li>
    <li><img _src="3.jpg" src="white.jpg"></li>
    <li><img _src="4.jpg" src="white.jpg"></li>
    <li><img _src="5.jpg" src="white.jpg"></li>
    <li><img _src="6.jpg" src="white.jpg"></li>
    <li><img _src="7.jpg" src="white.jpg"></li>
    <li><img _src="1.jpg" src="white.jpg"></li>
    <li><img _src="2.jpg" src="white.jpg"></li>
    <li><img _src="3.jpg" src="white.jpg"></li>
    <li><img _src="4.jpg" src="white.jpg"></li>
    <li><img _src="5.jpg" src="white.jpg"></li>
    <li><img _src="6.jpg" src="white.jpg"></li>
    <li><img _src="7.jpg" src="white.jpg"></li>
    <li><img _src="1.jpg" src="white.jpg"></li>
    <li><img _src="2.jpg" src="white.jpg"></li>
    <li><img _src="3.jpg" src="white.jpg"></li>
    <li><img _src="4.jpg" src="white.jpg"></li>
    <li><img _src="5.jpg" src="white.jpg"></li>
    <li><img _src="6.jpg" src="white.jpg"></li>
    <li><img _src="7.jpg" src="white.jpg"></li>
</ul>
```

``` css
#ul1 { margin: 100px auto 0; padding: 0; }
li { float: left; margin: 0 0 10px 10px; list-style: none; border: 1px solid black;}
img { width: 300px; height: 200px; display: block; }
```

``` js
window.onload = function(){
	var oUl = document.getElementById('ul1');
	var aImg = oUl.getElementsByTagName('img');

	showImg();
	window.onscroll = showImage;

	function showImg(){

		var scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
		for(var i=0; i<aImg.length; i++){
			if(!aImg[i].isLoad && getTop(aImg[i]) < scrollTop + document.documentElement.clientHeight ){
				aImg[i].src = aImg[i].getAttribute('_src');
				aImg[i].isLoad = true;
			}
		}
	}

	function getTop(obj){
		var iTop = 0;
		while(obj){
			iTop += obj.offsetTop;
			obj = obj.offsetParent;
		}
		return iTop;
	}
}
```

##弹性运动

- 加减速运动
    - 速度不断增加或减小
    - 速度减小到负值，会向反方向运动

###加速运动

``` html
<input type="button" value="开始运动" id="input1">
<div id="div1"></div>
```

``` css
#div1 { width: 100px; height: 100px; background: red; position: absolute; left: 0; }
```

``` js
window.onload = function(){
	var oInput = document.getElementById('input1');
	var oDiv = document.getElementById('div1');
	var timer = null;
	var iSpeed = 0;

	oInput.onclick = function(){
		startMove();
	}

	function startMove(){

		clearInterval(timer);
		timer = setInterval(function(){

			iSpeed += 3;
			oDiv.style.left = oDiv.offsetLeft + iSpeed + 'px';

		}, 30)
	}
}
```

###减速运动

``` js
window.onload = function(){
	var oInput = document.getElementById('input1');
	var oDiv = document.getElementById('div1');
	var timer = null;
	var iSpeed = 80;

	oInput.onclick = function(){

		startMove();

	}

	function startMove(){

		clearInterval(timer);
		timer = setInterval(function(){

			iSpeed -= 3;
			oDiv.style.left = oDiv.offsetLeft + iSpeed + 'px';

		}, 30)

	}
}
```

- 弹性运动
    - 在目标点左边，加速；在目标点右边，减速
    - 根据距离，计算加速度

###弹性运动

``` html
<input type="button" value="开始运动" id="input1">
<div id="div1"></div>
<div id="bg"></div>
```

``` css
#div1 { width: 100px; height: 100px; background: red; position: absolute; left: 0; }
#bg { width: 1px; height: 500px; background: black; position: absolute; left: 500px; top: 0; }
```

``` js
window.onload = function(){
	var oInput = document.getElementById('input1');
	var oDiv = document.getElementById('div1');
	var timer = null;
	var iSpeed = 0;

	oInput.onclick = function(){

		startMove();

	}

	function startMove(){

		clearInterval(timer);
		timer = setInterval(function(){

			if(oDiv.offsetLeft < 500){
				iSpeed += 5;
			} else {
				iSpeed -= 5;
			}
			oDiv.style.left = oDiv.offsetLeft + iSpeed + 'px';

		}, 30)

	}
}
```

###弹性运动带加速度

``` js
window.onload = function(){
	var oInput = document.getElementById('input1');
	var oDiv = document.getElementById('div1');
	var timer = null;
	var iSpeed = 0;

	oInput.onclick = function(){

		startMove();

	}

	function startMove(){

		clearInterval(timer);
		timer = setInterval(function(){

			// if(oDiv.offsetLeft < 500){
			// 	iSpeed += (500 - oDiv.offsetLeft)/50;
			// } else {
			// 	iSpeed -= (oDiv.offsetLeft - 500)/50;
			// }

			iSpeed += (500 - oDiv.offsetLeft)/50;
			oDiv.style.left = oDiv.offsetLeft + iSpeed + 'px';
		}, 30)

	}
}
```

- 带摩擦力的弹性运动
    - 弹性运动 + 摩擦力

摩擦力：F = fM （f是摩擦系数、M是质量）

###弹性运动带摩擦

``` js
window.onload = function(){
	var oInput = document.getElementById('input1');
	var oDiv = document.getElementById('div1');
	var timer = null;
	var iSpeed = 0;

	oInput.onclick = function(){

		startMove();

	}

	function startMove(){

		clearInterval(timer);
		timer = setInterval(function(){

			// if(oDiv.offsetLeft < 500){
			// 	iSpeed += (500 - oDiv.offsetLeft)/50;
			// } else {
			// 	iSpeed -= (oDiv.offsetLeft - 500)/50;
			// }

			iSpeed += (500 - oDiv.offsetLeft)/50;
			iSpeed *= 0.95;

			if(Math.abs(iSpeed) <= 1 && Math.abs(500 - oDiv.offsetLeft) <=1 ){
				clearInterval(timer);
				oDiv.style.left = '500px';
				iSpeed = 0;
			} else {
				oDiv.style.left = oDiv.offsetLeft + iSpeed + 'px';
			}

				

		}, 30)

	}
}
```

###弹性运动与缓冲运动的区别

**弹性运动：**

- 速度 += (目标点 - 当前值) / 系数; //6, 7, 8
- 速度 *= 摩擦系数; //0.7, 0.75

**缓冲运动：**

- var 速度 = (目标点 - 当前值) / 系数;
- 速度取整

###弹性菜单实例

``` html
<ul id="ul1">
    <li id="mark"></li>
    <li class="box">首页</li>
    <li class="box">论坛</li>
    <li class="box">视频</li>
    <li class="box">留言</li>
</ul>
```

``` css
* { margin: 0; padding: 0; }
#ul1 { width: 428px; height: 30px; margin: 20px auto; position: relative; }
#ul1 li { width: 100px; height: 30px; background: red; border: 1px #000 solid; margin-right: 5px; float: left; list-style: none; line-height: 30px; text-align: center; }
#ul1 #mark { position: absolute; left: 0; top: 0; background: blue; height: 10px; }
```

``` js
window.onload = function(){

	var oMark = document.getElementById('mark');
	var aBox = document.getElementsByClassName('box');
	var timer = null
	var iSpeed = 0;

	for(var i=0; i<aBox.length; i++){
		aBox[i].onmouseover = function(){
			startMove(this.offsetLeft);
		}
		aBox[i].onmouseout = function(){
			startMove(0);
		}

		function startMove(iTarget){
			clearInterval(timer);
			timer = setInterval(function(){
				iSpeed += (iTarget - oMark.offsetLeft) / 6;
				iSpeed *= 0.75;
				if(Math.abs(iSpeed) <= 1 && Math.abs(iTarget - oMark.offsetLeft) <= 1){
					clearInterval(timer);
					oMark.style.left = iTarget + 'px';
					iSpeed = 0;
				} else {
					oMark.style.left = oMark.offsetLeft + iSpeed + 'px';
				}
			}, 30)
		}
	}

}
```

###弹性菜单优化

**滚动歌词效果**

``` html
<div id="div1"><span>阿里的房间啊高领导看见噶的离开房间爱多了几分</span></div>
<div id="div2"><span>阿里的房间啊高领导看见噶的离开房间爱多了几分</span></div>
```

``` css
* { margin: 0; padding: 0; }
#div1, #div2 { position: absolute; left: 0; top: 0; }
#div2 { color: red; width: 15px; height: 16px; overflow: hidden; }
#div2 span { position: absolute; left: 0; top: 0; width: 2000px; }
```

``` js
window.onload = function(){

	var oDiv2 = document.getElementById('div2');
	var oSpan2 = oDiv2.getElementsByTagName('span')[0];

	setInterval(function(){
		oDiv2.style.left = oDiv2.offsetLeft + 1 + 'px';
		oSpan2.style.left = -oDiv2.offsetLeft + 'px';
	}, 30)

}
```

**弹性菜单实例优化**

``` html
<ul id="ul1">
	<li id="mark">
		<ul>
			<li class="box">首页</li>
			<li class="box">论坛</li>
			<li class="box">视频</li>
			<li class="box">留言</li>		
		</ul>
	</li>
	<li class="box">首页</li>
	<li class="box">论坛</li>
	<li class="box">视频</li>
	<li class="box">留言</li>
</ul>
```

``` css
* { margin: 0; padding: 0; }
#ul1 { width: 428px; height: 30px; margin: 20px auto; position: relative; }
#ul1 li { width: 100px; height: 30px; background: red; border: 1px #000 solid; margin-right: 5px; float: left; list-style: none; line-height: 30px; text-align: center; }
#ul1 #mark { position: absolute; left: 0; top: 0; overflow: hidden; background: blue; }
#ul1 #mark ul { width: 428px; position: absolute; left: -1px; top: -1px; color: #fff; }
#mark ul li { background: blue; }
```

``` js
window.onload = function(){

	var oMark = document.getElementById('mark');
	var aBox = document.getElementsByClassName('box');
	var childUl = oMark.getElementsByTagName('ul')[0];
	var timer = null;
	var timer2 = null;
	var iSpeed = 0;

	for(var i=0; i<aBox.length; i++){
		aBox[i].onmouseover = function(){
			clearTimeout(timer2);
			startMove(this.offsetLeft);
		}
		aBox[i].onmouseout = function(){
			timer2 = setTimeout(function(){
				startMove(0);	
			}, 100);
			
		}
	}

	oMark.onmouseover = function(){
		clearTimeout(timer2);
	}
	oMark.onmouseout = function(){
		timer2 = setTimeout(function(){
			startMove(0);	
		}, 100);
	}

	function startMove(iTarget){
		clearInterval(timer);
		timer = setInterval(function(){
			iSpeed += (iTarget - oMark.offsetLeft) / 6;
			iSpeed *= 0.75;
			if(Math.abs(iSpeed) <= 1 && Math.abs(iTarget - oMark.offsetLeft) <= 1){
				clearInterval(timer);
				oMark.style.left = iTarget + 'px';
				childUl.style.left = -iTarget + 'px';
				iSpeed = 0;
			} else {
				oMark.style.left = oMark.offsetLeft + iSpeed + 'px';
				childUl.style.left = -oMark.offsetLeft + 'px';
			}
		}, 30)
	}
	
}
```

**弹性过界：**

IE老版本下，宽高不能为负数。

``` html
<div id="div1"></div>
```

``` css
#div1 { width: 100px; height: 30px; background: red; }
```

``` js
window.onload = function(){

	var oDiv = document.getElementById('div1');
	var timer = null;
	var iSpeed = 0;

	oDiv.onmouseover = function(){
		startMove(300);
	}
	oDiv.onmouseout = function(){
		startMove(30);
	}

	function startMove(iTarget){

		clearInterval(timer);
		timer = setInterval(function(){
			iSpeed += (iTarget - oDiv.offsetHeight) / 6;
			iSpeed *= 0.75;
			
			if(Math.abs(iSpeed) <= 1 && Math.abs(iTarget - oDiv.offsetHeight) <= 1){
				clearInterval(timer);
				iSpeed = 0;
				oDiv.style.height = iTarget + 'px';
			} else {
				var H = oDiv.offsetHeight + iSpeed; 
				if(H < 0) {
					H = 0;
				} //解决IE下的弹性过界的问题
				oDiv.style.height = H + 'px';
			}
		}, 30)

	}

}
```

###弹性运动框架

``` js 
function startMove(obj,json,fn){
        clearInterval(obj.timer);
        
        var iSpeed = {};
        for(var attr in json){
                iSpeed[attr] = 0;
        }
        
        obj.timer = setInterval(function(){
                
                var bBtn = true;
                
                for(var attr in json){
                        
                        var iCur = 0;
                        if(attr == 'opacity'){
                                iCur = Math.round(getStyle(obj,attr)*100);
                        }
                        else{
                                iCur = parseInt(getStyle(obj,attr));
                        }
                        
                        iSpeed[attr] += (json[attr] - iCur)/6;
                        iSpeed[attr] *= 0.75;
                        
                        if( Math.abs(iSpeed[attr])>1 || Math.abs(json[attr] - iCur)>1 ){
                                
                                bBtn = false;
                        }
                        
                        var value = iCur + iSpeed[attr];
                        
                        if(value < 0 && (attr == 'width'||attr == 'height')){
                                value = 0;
                        }
                        
                        if(attr == 'opacity'){
                                obj.style.filter = 'alpha(opacity='+ value +')';
                                obj.style.opacity = value/100; 
                        }
                        else{
                                obj.style[attr] = value + 'px';
                        }
                        
                        
                }
                
                if(bBtn){
                        clearInterval(obj.timer);
                        for(var attr in json){
                                iSpeed[attr] = 0;
                                if(attr == 'opacity'){
                                        obj.style.filter = 'alpha(opacity='+ json[attr] +')';
                                        obj.style.opacity = json[attr]/100; 
                                }
                                else{
                                        obj.style[attr] = json[attr] + 'px';
                                }
                        }
                        
                        if(fn){
                                fn.call(obj);
                        }
                        
                }
                
        },30);
}
        
        
function getStyle(obj,attr){
        if(obj.currentStyle){
                return obj.currentStyle[attr];
        }
        else{
                return getComputedStyle(obj,false)[attr];
        }
}

```

##碰撞运动

- 碰撞运动
    - 撞到目标点，速度反转（首先找到碰撞的临界点，再确定运动的方向，然后去改对应的速度，即速度取反）
    - 无重力的漂浮div
        - 速度反转
        - 滚动条闪烁的问题
            - 过界后直接拉回来
- 加入重力
    - 反转速度的同时，减小速度
    - 纵向碰撞，横向速度也减小
    - 横向速度小数问题（负数）

###无重力匀速碰撞运动
    
``` html
<div id="div1"></div>
```

``` css
#div1 { width: 100px; height: 100px; background: red; position: absolute; }
```

``` js
window.onload = function(){

	var oDiv = document.getElementById('div1');
	var iSpeedX = 10;
	var iSpeedY = 10;

	startMove();
	function startMove(){
		setInterval(function(){
			var L = oDiv.offsetLeft + iSpeedX;
			var T = oDiv.offsetTop + iSpeedY;

			if(T > document.documentElement.clientHeight - oDiv.offsetHeight){
				T = document.documentElement.clientHeight - oDiv.offsetHeight;
				iSpeedY *= -1;
			} else if(T < 0){
				T = 0;
				iSpeedY *= -1;
			}

			if(L > document.documentElement.clientWidth - oDiv.offsetWidth){
				L = document.documentElement.clientWidth - oDiv.offsetWidth;
				iSpeedX *= -1;
			} else if(L < 0){
				L = 0;
				iSpeedX *= -1;
			}
			oDiv.style.left = L + 'px';
			oDiv.style.top = T + 'px';
		}, 30);
	}

}
```

###自由落体加碰撞反弹

``` html
<input type="button" value="开始运动" id="input1">
<div id="div1"></div>
```

``` css
#div1 { width: 100px; height: 100px; background: red; position: absolute; }
```

``` js
window.onload = function(){
	var oInput = document.getElementById('input1');
	var oDiv = document.getElementById('div1');
	var timer = null;
	var iSpeed = 0;

	oInput.onclick = function(){
		startMove();
	}

	function startMove(){
		clearInterval(timer);
		timer = setInterval(function(){
			iSpeed += 3;
			var T = oDiv.offsetTop + iSpeed;
			if(T > document.documentElement.clientHeight - oDiv.offsetHeight){
				T = document.documentElement.clientHeight - oDiv.offsetHeight;
				iSpeed *= -1;
				iSpeed *= 0.75;
			}
			oDiv.style.top = T  + 'px';
		}, 30);
	}
}
```

###抛物线落体

``` html
<input type="button" value="开始运动" id="input1">
<div id="div1"></div>
```

``` css
#div1 { width: 100px; height: 100px; background: red; position: absolute; top: 500px; }
```

``` js
window.onload = function(){
	var oInput = document.getElementById('input1');
	var oDiv = document.getElementById('div1');
	var timer = null;
	var iSpeed = -40;
	var iSpeedX = 10;

	oInput.onclick = function(){
		startMove();
	}

	function startMove(){
		clearInterval(timer);
		timer = setInterval(function(){
			iSpeed += 3;
			var T = oDiv.offsetTop + iSpeed;
			if(T > document.documentElement.clientHeight - oDiv.offsetHeight){
				T = document.documentElement.clientHeight - oDiv.offsetHeight;
				iSpeed *= -1;
				iSpeed *= 0.75;
				iSpeedX *= 0.75;
			}
			oDiv.style.top = T  + 'px';
			oDiv.style.left = oDiv.offsetLeft + iSpeedX + 'px';
		}, 30);
	}
}
```

###iphone拖拽效果

``` html
<div id="iphone" >
	<div id="wrap">
		<ul id="ul1">
			<li style="background:url(images/pic1.png);" title="妙味课堂"></li>
			<li style="background:url(images/pic2.png);" title="妙味课堂"></li>
			<li style="background:url(images/pic3.png);" title="妙味课堂"></li>
			<li style="background:url(images/pic4.png);" title="妙味课堂"></li>
		</ul>
	</div>
</div>
```

``` js
window.onload = function(){
	var oUl = document.getElementById('ul1');
	var aLi = oUl.getElementsByTagName('li');
	
	var disX = 0;
	var downX = 0;
	var iNow = 0;
	var timer = null;
	var iSpeed = 0;
	
	oUl.onmousedown = function(ev){
		var ev = ev || window.event;
		disX = ev.clientX - oUl.offsetLeft;
		downX = ev.clientX;
		
		clearInterval(timer);
		
		document.onmousemove = function(ev){
			var ev = ev || window.event;
			oUl.style.left = ev.clientX - disX + 'px';
		};
		document.onmouseup = function(ev){
			document.onmousemove = null;
			document.onmouseup = null;
			var ev = ev || window.event;
			
			if( ev.clientX < downX ){
				//alert('←');
				if( iNow != aLi.length-1 ){
					iNow++;
				}
				
				startMove( - iNow * aLi[0].offsetWidth  );
			}
			else{
				//alert('→');
				
				if( iNow != 0 ){
					iNow--;
				}
				
				startMove( - iNow * aLi[0].offsetWidth  );
				
			}
			
		};
		return false;
	};
	
	function startMove(iTarget){
		clearInterval(timer);
		timer = setInterval(function(){
			
			iSpeed += (iTarget - oUl.offsetLeft)/6;
			iSpeed *= 0.75;
			
			if( Math.abs(iSpeed)<=1 && Math.abs(iTarget - oUl.offsetLeft)<=1 ){
				clearInterval(timer);
				iSpeed = 0;
				oUl.style.left = iTarget + 'px';
			}
			else{
				oUl.style.left = oUl.offsetLeft + iSpeed + 'px';
			}
			
		},30);
	}
	
};

```

###碰撞弹窗，模仿官网公告菜单实例

``` html
<div id="div1"></div>
```

``` css
#div1{ width:100px; height:100px; background:red; position:absolute;}
```

``` js
window.onload = function(){
	var oDiv = document.getElementById('div1');
	
	var disX = 0;
	var disY = 0;
	
	var prevX = 0;
	var prevY = 0;
	var iSpeedX = 0;
	var iSpeedY = 0;
	
	var timer = null;
	
	oDiv.onmousedown = function(ev){
		var ev = ev || window.event;
		disX = ev.clientX - oDiv.offsetLeft;
		disY = ev.clientY - oDiv.offsetTop;
		
		prevX = ev.clientX;
		prevY = ev.clientY;
		
		document.onmousemove = function(ev){
			var ev = ev || window.event;
			oDiv.style.left = ev.clientX - disX + 'px';
			oDiv.style.top = ev.clientY - disY + 'px';
			
			iSpeedX = ev.clientX - prevX;
			iSpeedY = ev.clientY - prevY;
			
			prevX = ev.clientX;
			prevY = ev.clientY;
			
		};
		document.onmouseup = function(){
			document.onmousemove = null;
			document.onmouseup = null;
			
			startMove();
			
		};
		return false;
	};
	
	function startMove(){
		clearInterval(timer);
		timer = setInterval(function(){
			
			iSpeedY += 3;
			
			var L = oDiv.offsetLeft + iSpeedX;
			var T = oDiv.offsetTop + iSpeedY;
			
			if(T>document.documentElement.clientHeight - oDiv.offsetHeight){
				T = document.documentElement.clientHeight - oDiv.offsetHeight;
				iSpeedY *= -1;
				iSpeedY *= 0.75;
				iSpeedX *= 0.75;
			}
			else if(T<0){
				T = 0;
				iSpeedY *= -1;
				iSpeedY *= 0.75;
			}
			
			if(L>document.documentElement.clientWidth - oDiv.offsetWidth){
				L = document.documentElement.clientWidth - oDiv.offsetWidth;
				iSpeedX *= -1;
				iSpeedX *= 0.75;
			}
			else if(L<0){
				L = 0;
				iSpeedX *= -1;
				iSpeedX *= 0.75;
			}
			
			oDiv.style.left = L + 'px';
			oDiv.style.top = T + 'px';
			
		},30);
	}
	
};
```

##运动框架（时间版）

### JQ的animate

- 一个典型的时间版运动框架
- 与经典的startMove的区别
    - 以时间为单位，而不是以速度为单位
    - 例子：从中间放大的图片

###Tween介绍

- 一个来自flash的运动算法
- JQ中也在使用tween算法
- Tween公式
    - t: current time (当前时间)
    - b: beginning value (初始值)
    - c: change in value (变化量)
    - d: duration (持续时间)
    - return (目标点)

###用原生JS写时间版运动框架

- 新运动框架
    - 如何获取当前时间
        - (new Date()).getTime()
    - 老版本中的BUG，可以在新版本中修复
        - 定时器缓慢的问题（切换页面时）
        - 例子：循环轮播图

**原生JS写运动框架**


``` js
function startMove(obj, json, time, fx, fn){
	//time: 运动事件 fx: 运动形式

	var iCur = {}; //初始值
	var startTime = now();

	for(var attr in json){
		iCur[attr] = 0; 

		if(attr == 'opacity'){
			iCur[attr] = Math.round(getStyle(obj, attr)*100);
		} else {
			iCur[attr] = parseInt(getStyle(obj, attr));
		}
	}

	clearInterval(obj.timer)
	obj.timer = setInterval(function(){
		var changeTime = now();

		var t = time - Math.max(0, startTime - changeTime + time) //范围：0到time

		for(var attr in json){
			var value = Tween[fx](t, iCur[attr], json[attr] - iCur[attr], time);

			if(attr == 'opacity'){
				obj.style.oapcity = value / 100;
				obj.style.filter = 'alpha(opacity=' + value + ')';
			} else {
				obj.style[attr] = value + 'px';
			}
		}

		if(t == time){
			clearInterval(obj.timer);
			if(fn){
				fn.call(obj);
			}
		}

	}, 13)

	function getStyle(obj, attr){
		if(obj.currentStyle){
			return obj.currentStyle[attr];
		} else {
			return getComputedStyle(obj, false)[attr];
		}
	}

	function now(){
		return (new Date()).getTime();
	}

}

var Tween = {
	//t: 当前时间 b: 初始值 c: 变化量 d: 持续时间
	//return: 返回的是运动到的目标点
	linear: function (t, b, c, d){  //匀速
		return c*t/d + b;
	},
	easeIn: function(t, b, c, d){  //加速曲线
		return c*(t/=d)*t + b;
	},
	easeOut: function(t, b, c, d){  //减速曲线
		return -c *(t/=d)*(t-2) + b;
	},
	easeBoth: function(t, b, c, d){  //加速减速曲线
		if ((t/=d/2) < 1) {
			return c/2*t*t + b;
		}
		return -c/2 * ((--t)*(t-2) - 1) + b;
	},
	easeInStrong: function(t, b, c, d){  //加加速曲线
		return c*(t/=d)*t*t*t + b;
	},
	easeOutStrong: function(t, b, c, d){  //减减速曲线
		return -c * ((t=t/d-1)*t*t*t - 1) + b;
	},
	easeBothStrong: function(t, b, c, d){  //加加速减减速曲线
		if ((t/=d/2) < 1) {
			return c/2*t*t*t*t + b;
		}
		return -c/2 * ((t-=2)*t*t*t - 2) + b;
	},
	elasticIn: function(t, b, c, d, a, p){  //正弦衰减曲线（弹动渐入）
		if (t === 0) { 
			return b; 
		}
		if ( (t /= d) == 1 ) {
			return b+c; 
		}
		if (!p) {
			p=d*0.3; 
		}
		if (!a || a < Math.abs(c)) {
			a = c; 
			var s = p/4;
		} else {
			var s = p/(2*Math.PI) * Math.asin (c/a);
		}
		return -(a*Math.pow(2,10*(t-=1)) * Math.sin( (t*d-s)*(2*Math.PI)/p )) + b;
	},
	elasticOut: function(t, b, c, d, a, p){    //正弦增强曲线（弹动渐出）
		if (t === 0) {
			return b;
		}
		if ( (t /= d) == 1 ) {
			return b+c;
		}
		if (!p) {
			p=d*0.3;
		}
		if (!a || a < Math.abs(c)) {
			a = c;
			var s = p / 4;
		} else {
			var s = p/(2*Math.PI) * Math.asin (c/a);
		}
		return a*Math.pow(2,-10*t) * Math.sin( (t*d-s)*(2*Math.PI)/p ) + c + b;
	},    
	elasticBoth: function(t, b, c, d, a, p){
		if (t === 0) {
			return b;
		}
		if ( (t /= d/2) == 2 ) {
			return b+c;
		}
		if (!p) {
			p = d*(0.3*1.5);
		}
		if ( !a || a < Math.abs(c) ) {
			a = c; 
			var s = p/4;
		}
		else {
			var s = p/(2*Math.PI) * Math.asin (c/a);
		}
		if (t < 1) {
			return - 0.5*(a*Math.pow(2,10*(t-=1)) * 
					Math.sin( (t*d-s)*(2*Math.PI)/p )) + b;
		}
		return a*Math.pow(2,-10*(t-=1)) * 
				Math.sin( (t*d-s)*(2*Math.PI)/p )*0.5 + c + b;
	},
	backIn: function(t, b, c, d, s){     //回退加速（回退渐入）
		if (typeof s == 'undefined') {
		   s = 1.70158;
		}
		return c*(t/=d)*t*((s+1)*t - s) + b;
	},
	backOut: function(t, b, c, d, s){
		if (typeof s == 'undefined') {
			s = 3.70158;  //回缩的距离
		}
		return c*((t=t/d-1)*t*((s+1)*t + s) + 1) + b;
	}, 
	backBoth: function(t, b, c, d, s){
		if (typeof s == 'undefined') {
			s = 1.70158; 
		}
		if ((t /= d/2 ) < 1) {
			return c/2*(t*t*(((s*=(1.525))+1)*t - s)) + b;
		}
		return c/2*((t-=2)*t*(((s*=(1.525))+1)*t + s) + 2) + b;
	},
	bounceIn: function(t, b, c, d){    //弹球减振（弹球渐出）
		return c - Tween['bounceOut'](d-t, 0, c, d) + b;
	},       
	bounceOut: function(t, b, c, d){
		if ((t/=d) < (1/2.75)) {
			return c*(7.5625*t*t) + b;
		} else if (t < (2/2.75)) {
			return c*(7.5625*(t-=(1.5/2.75))*t + 0.75) + b;
		} else if (t < (2.5/2.75)) {
			return c*(7.5625*(t-=(2.25/2.75))*t + 0.9375) + b;
		}
		return c*(7.5625*(t-=(2.625/2.75))*t + 0.984375) + b;
	},      
	bounceBoth: function(t, b, c, d){
		if (t < d/2) {
			return Tween['bounceIn'](t*2, 0, c, d) * 0.5 + b;
		}
		return Tween['bounceOut'](t*2-d, 0, c, d) * 0.5 + c*0.5 + b;
	}
}
```


**解决定时器缓慢的问题的方法**


``` js
window.onfocus = function(){ //当页面切换回来之后再把定时器打开
    console.log(1);
    timer = setInterval(toRun, 2000);
}
window.onblur = function(){ //当切换页面的时候把定时器关上
    console.log(2);
    clearInterval(timer);
}

//timer = setInterval(toRun, 2000);
```


###扩展JQ的运动形式

- 默认两种：swing、linear
- 可以通过extend进行扩展


``` js
$.extend(jQuery.easing, {

    //以下为Tween中的公式 为了匹配jQuery，添加了一个不用的参数x
    easeIn: function(x, t, b, c, d){  //加速曲线
		return c*(t/=d)*t + b;
	},
	easeOut: function(x, t, b, c, d){  //减速曲线
		return -c *(t/=d)*(t-2) + b;
	},
	easeBoth: function(x, t, b, c, d){  //加速减速曲线
		if ((t/=d/2) < 1) {
			return c/2*t*t + b;
		}
		return -c/2 * ((--t)*(t-2) - 1) + b;
	},
	easeInStrong: function(x, t, b, c, d){  //加加速曲线
		return c*(t/=d)*t*t*t + b;
	},
	easeOutStrong: function(x, t, b, c, d){  //减减速曲线
		return -c * ((t=t/d-1)*t*t*t - 1) + b;
	},
	easeBothStrong: function(x, t, b, c, d){  //加加速减减速曲线
		if ((t/=d/2) < 1) {
			return c/2*t*t*t*t + b;
		}
		return -c/2 * ((t-=2)*t*t*t - 2) + b;
	},
	elasticIn: function(x, t, b, c, d, a, p){  //正弦衰减曲线（弹动渐入） //a和p与运动幅度有关，可以不写，因为有默认值
		if (t === 0) { 
			return b; 
		}
		if ( (t /= d) == 1 ) {
			return b+c; 
		}
		if (!p) {
			p=d*0.3; 
		}
		if (!a || a < Math.abs(c)) {
			a = c; 
			var s = p/4;
		} else {
			var s = p/(2*Math.PI) * Math.asin (c/a);
		}
		return -(a*Math.pow(2,10*(t-=1)) * Math.sin( (t*d-s)*(2*Math.PI)/p )) + b;
	},
	elasticOut: function(x, t, b, c, d, a, p){    //正弦增强曲线（弹动渐出）
		if (t === 0) {
			return b;
		}
		if ( (t /= d) == 1 ) {
			return b+c;
		}
		if (!p) {
			p=d*0.3;
		}
		if (!a || a < Math.abs(c)) {
			a = c;
			var s = p / 4;
		} else {
			var s = p/(2*Math.PI) * Math.asin (c/a);
		}
		return a*Math.pow(2,-10*t) * Math.sin( (t*d-s)*(2*Math.PI)/p ) + c + b;
	},    
	elasticBoth: function(x, t, b, c, d, a, p){
		if (t === 0) {
			return b;
		}
		if ( (t /= d/2) == 2 ) {
			return b+c;
		}
		if (!p) {
			p = d*(0.3*1.5);
		}
		if ( !a || a < Math.abs(c) ) {
			a = c; 
			var s = p/4;
		}
		else {
			var s = p/(2*Math.PI) * Math.asin (c/a);
		}
		if (t < 1) {
			return - 0.5*(a*Math.pow(2,10*(t-=1)) * 
					Math.sin( (t*d-s)*(2*Math.PI)/p )) + b;
		}
		return a*Math.pow(2,-10*(t-=1)) * 
				Math.sin( (t*d-s)*(2*Math.PI)/p )*0.5 + c + b;
	},
	backIn: function(x, t, b, c, d, s){     //回退加速（回退渐入）
		if (typeof s == 'undefined') {
		   s = 1.70158;
		}
		return c*(t/=d)*t*((s+1)*t - s) + b;
	},
	backOut: function(x, t, b, c, d, s){
		if (typeof s == 'undefined') {
			s = 3.70158;  //回缩的距离
		}
		return c*((t=t/d-1)*t*((s+1)*t + s) + 1) + b;
	}, 
	backBoth: function(x, t, b, c, d, s){
		if (typeof s == 'undefined') {
			s = 1.70158; 
		}
		if ((t /= d/2 ) < 1) {
			return c/2*(t*t*(((s*=(1.525))+1)*t - s)) + b;
		}
		return c/2*((t-=2)*t*(((s*=(1.525))+1)*t + s) + 2) + b;
	},
	bounceIn: function(x, t, b, c, d){    //弹球减振（弹球渐出）
		return c - this['bounceOut'](x, d-t, 0, c, d) + b;
	},       
	bounceOut: function(x, t, b, c, d){
		if ((t/=d) < (1/2.75)) {
			return c*(7.5625*t*t) + b;
		} else if (t < (2/2.75)) {
			return c*(7.5625*(t-=(1.5/2.75))*t + 0.75) + b;
		} else if (t < (2.5/2.75)) {
			return c*(7.5625*(t-=(2.25/2.75))*t + 0.9375) + b;
		}
		return c*(7.5625*(t-=(2.625/2.75))*t + 0.984375) + b;
	},      
	bounceBoth: function(x, t, b, c, d){
		if (t < d/2) {
			return this['bounceIn'](x, t*2, 0, c, d) * 0.5 + b;
		}
		return this['bounceOut'](x, t*2-d, 0, c, d) * 0.5 + c*0.5 + b;
	}
});
```










