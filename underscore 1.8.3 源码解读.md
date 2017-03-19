#underscore 1.8.3 源码解读

##一些基本设置 

underscore的源码总体结构是 `function(){}.call(this)`

```javascript
var root = this;
```

在浏览器环境下，this就是`window`，如果是在node环境下，this就是`exports`

###`_`构造函数

```javascript
var _ = function(obj){
	if(obj instanceof _) return obj;
	if(! (this instanceof _)) return new _(obj);
	this._wrapped = obj;
}
```

上述代码中我们看到了 `new` ，这说明 _ 这个函数其实是一个构造函数。代码写到这里，如果我们为这个构造函数传入一个对象 `[1, 2, 3]` 会得到一个 _ 的实例对象：`{_wrapped: [1, 2, 3]}`，该实例的 constructor 就是 _，而原型就是Object。如果我们对 `_(_([1, 2, 3]))`求值，得到的还是 `{_wrapped: [1, 2, 3]}`，因为已经是 _ 的实例，就直接把这个对象返回了。

###optimizeCb

```javascript
var optimizeCb = function(func, context, argCount){
	if(context === (void 0)) return func;
	switch( (argCount == null) ? 3 : argCount ){
		case 1: return function(value){};
		case 2: return function(value, other){};
		case 3: return function(value, index, collection){};
		case 4: return function(accumulator, value, index, collection){}
	};
	return function(){ return func.apply(context, arguments);}
}
```

可以看到optimizeCb是对函数的一个统一管理。该函数接受的第一个参数就是个函数，最终返回的也都是函数。

首先判断了context这个参数有没有，如果没有的话，就直接把传入的函数原样返回了。此处判断 context 有没有，并没有使用 `undefined`，而是用了 `void 0`。这是因为 `undefined`有被改写的风险。如果在chrome的console下面，直接改写 `undefined = 10`，然后求undefined的值，返回的仍然是 `undefined` 这是因为undefined在全局下已经是一个只读的属性了。但是在局部作用域中，仍然可以被改写。例如：

```javascript
var test = function(){
	var undefined = 'abc';  //注意，声明undefined为局部变量
	console.log(undefined);
}
test(); //'abc'
```

void实际上是一个运算符，什么东西经过它运算之后，都会返回undefined。一般`void 0`或者`void(0)`是比较常见的写法。

在`optimizeCb`中，如果传入了context，并且argCount没有传入或者传入的argCount是1、2、3或4，返回的都是`func.call(context, ...)`。当argCount大于4个时，才返回`func.apply(context, arguments)`。网上有解释说因为call的效率要比apply高很多。参考：https://segmentfault.com/q/1010000007894513

###cb

```javascript
var cb = function(value, context, argCount){
	if(value == null) return _.identity;
	if(_.isFunction(value)) return optimizeCb(value, context, argCount);
	if(_.isObject(value)) return _.matcher(value);
	return _.property(value);
}
```

###_.identity

`_.identity`这个函数的作用就是将传入的值原样返回。
```
var stooge = {name: 'moe'};
stooge === _.identity(stooge); //true
```

###_.matcher

`_.matcher(attrs)`这个函数的作用就是返回一个predicate function，这个predicate function接收object作为参数之后，返回值就是是否这个object包含attrs中的所有键值对属性。
```javascript
var func = _.matcher({name: "Amanda", age: 17});
func({name: "Amanda", age: 18}); //false
func({name: "Amanda", age: 17, sex: "female"}); //true
```

###_.property

`_.property(key)`这个函数的作用就是返回一个函数，这个函数接收一个object作为参数之后，返回这个object中key属性的值。
```javascript
var stooge = {name: 'moe'};
_.property('name')(stooge); //'moe'
```

###_.iteratee

```javascript
_.iteratee = function(value, context){ return cb(value, context, Infinity); }
```

_.iteratee的用法跟作用其实跟内部的cb差不多。分为下面四种情况：

- 没有传入任何参数 `_.iteratee()`，那么返回的就是 _.identity()函数
- 传入的参数是个函数 `_.iteratee(f)`，那么返回的就是一个optimized callback，其实就是`optimizeCb(f)`
- 传入的是对象`_.iteratee(obj)`，那么返回的就是`_.matcher(obj)`，也就是一个predicate函数，这个函数会判断所传入的参数是否包含obj所有的key/value键值对
	- 传入的是其他值，如`_.iteratee(prop)`，返回的是`_.property(prop)`函数，这个函数会返回传入参数的属性为prop的值。

###createAssigner

```javascript
var createAssigner = function(keysFunc, undefinedOnly){
	return function(obj){
		var length = arguments.length;
		if(length < 2 || obj == null) return obj;
		for(var index = 1; index < length; index++) {
			var source = arguments[index],
				keys = keysFunc(source),
				l = keys.length;
			for (var i = 0; i < l; i++){
				var key = keys[i];
				if(!undefinedOnly || obj[key] === void 0) obj[key] = source[key];
			}
		}
		return obj;
	};
};
```

createAssigner是一个创建赋值函数的函数，它的返回值是一个function。而且这是一个经典闭包，undefinedOnly这个参数是在返回函数之外的，但是在返回函数内会用到。
createAssigner中的keysFunc其实传入的是`_.keys`或`_.allkeys`。分别来看一下这两个函数：

###_.keys

```javascript
_.keys = function(obj){
	if(!_.isObject(obj)) return []; //如果传入的不是对象，那么返回空函数
	if(nativeKeys) return nativeKeys(obj); //nativeKeys = Object.keys 是原生的方法
	var keys = [];
	for(var key in obj) if(_.has(obj, key)) keys.push(key); //_.has 其实就是hasOwnProperty.call(obj, key)
	if(hasEnumBug) collectNonEnumProps(obj, keys);
	return keys;
}
```

###_.allKeys

```javascript
_.allKeys = function(obj){
	if(!_.isObject(obj)) return [];
	var keys = [];
	for(var key in obj) keys.push(key);
	if(hasEnumBug) collectNonEnumProps(obj, keys);
	return keys;
}
```

可见，`_.keys`和`_.allKeys`两者定义非常接近，只不过`_.keys`返回的是obj属于自己的键名，而`_.allKeys`则将不属于自己的键名也一起返回。

`for in`循环会输入自身以及原型链上可枚举的属性。使用`hasOwnProperty`过滤一下，就能把原型链上的属性过滤掉。

`Object.keys`方法就相当于for in循环加上hasOwnProperty过滤，输出的是自身上可枚举的属性。

hasEnumBug和collectNonEnumProps是怎么回事？

```javascript
var hasEnumBug = !{toString: null}.propertyIsEnumerable('toString');
var nonEnumerableProps = ['valueOf', 'isPrototypeOf', 'toString', 'propertyIsEnumerable', 'hasOwnProperty', 'toLocalString'];

function collectNonEnumProps(obj, keys){
	var nonEnumIdx = nonEnumerableProps.length;
	var constructor = obj.constructor;
	var proto = (_.isFunction(constructor) && constructor.prototype) || ObjProto; //这一句用来找obj的原型，其中ObjProto = Object.prototype

	var prop = 'constructor';
	if(_.has(obj, prop) && !_.contains(keys, prop)) keys.push(prop);

	while(nonEnumIdx--){
		prop = nonEnumerableProps[nonEnumIdx];
		if(prop in obj && obj[prop] !== proto[prop] && !_.contains(keys, prop)){
			keys.push(prop);
		}
	}
}
```

据underscore自己的解释说（我未亲证），在IE<9中，对象`{toString: null}`的toString属性不会被for in循环到，用此刻判断IE的版本是否小于9。

回到createAssigner函数，这个函数所返回的函数，如果参数少于两个，或者传入的是null，那么直接传进去什么就返回什么（因为没有什么好拷贝的）。

如果参数个数大于一个，那么这说明第一个是被添加属性的对象，而后面的不论多少参数都是要被拷贝的对象模子。

如果拷贝期间，原本obj上已有的属性值要被覆盖，那么undefinedOnly就可以不传参，或者传入false。如果原本obj上已有的属性，后面模子上也有的，原obj上保持不变，那么undefinedOnly就传入true。

用到createAssigner的有下面三个方法：

###_.extend, _.extendOwn, _.defaults

```javascript
_.extend = createAssigner(_.allKeys); //不管三七二十一，后面的obj上的属性（包括自己的和原型链上的）都拷贝过来
_.extendOwn = _.assign = createAssigner(_.keys); //只拷贝后面obj上属于自己的属性，原型链上的不拷贝
_.defaults = createAssigner(_.allKeys, true); //拷贝后面obj上的所有属性（包括原型链上的），但是如果遇到了重复的属性，则以原来的为准，此处undefinedOnly这里就传入了true
```

可以看出，underscore中的`_.extend`，`_.extendOwn`，`_.defaults`实现的都是浅拷贝。

###baseCreate

```javascript
var baseCreate = function(prototype){
	if(!_.isObject(prototype)) return {};
	if(nativeCreate) return nativeCreate(prototype); //nativeCreate = Object.create
	Ctor.prototype = prototype;
	var result = new Ctor;
	Ctor.prototype = null;
	return result;
}
```

baseCreate函数就是生成一个以参数为原型的对象。

```javascript
var isArrayLike = function(collection){
	var length = getLength(collection);
	return typeof length == 'number' && length >= 0 && length <= MAX_ARRAY_INDEX;
}
```

在underscore中，只有一个对象具备length属性，并且length属性的值是数字，并且是在最大范围允许的情况下，就被认为这个对象是arraylike的。

##用于集合的一些方法

###_.each(list, iteratee, [context])

对集合中的每一项进行iteratee的操作

```javascript
_.each = _.forEach = function(obj, iteratee, context){
	iteratee = optimizeCb(iteratee, context);
	var i, length;
	if(isArrayLike(obj)){
		for(i = 0, length = obj.length; i < length; i++){
			iteratee(obj[i], i, obj);
		}
	} else {
		var keys = _.keys(obj);
		for(i = 0, length = keys.length; i < length; i++){
			iteratee(obj[key[i]], keys[i], obj);
		}
	}
	return obj;
};
```

###_.map(list, iteratee, [context])

对list中的每一项进行iteratee的操作，将新生成的数组返回

```javascript
_.map = _.collect = function(obj, iteratee, context){
	iteratee = cb(iteratee, context);
	var keys = !isArrayLike(obj) && _.keys(obj),
		length = (keys || obj).length,
		results = Array(length);
	for(var index = 0; index < length; index++){
		var currentKey = keys ? keys[index] : index;
		results[index] = iteratee(obj[currentKey], currentKey, obj);
	}
	return results;
}
```

从源码可以看出，_.map返回的始终是数组。哪怕传入的对象，返回的也是数组。在这里，iteratee过的不是optimizeCb而是cb，所以iteratee这里不一定是function，也可以传object或字符串。

```javascript
_.map([1, 2, 3], function(i){ return i * i; }) //[1, 4, 9]
_.map([1, 2, 3], [2]) //[false, false, false]
//第二个参数不是函数，而是对象的时候，那么就是判断每一项是否包含与这个对象同样的键值对；下面这个例子中，第二个参数是[2]，可以把它视为这样一个对象 {'0': '2'}，那么其中可以过关的只有第二项[2, 3, 1]，其中包含'0'/2这样的键值对
_.map([[1, 2, 3], [2, 3, 1], [3, 1, 2]], [2]) //[false, true, false]
//第二个参数如果是数字或字符串，就是返回每一项以这个数字/字符串为key的值
_.map([1, 2, 3], 1) //[undefined, undefined, undefined]
_.map([[1, 2, 3], [2, 3, 1], [3, 1, 2]], 1) //[2, 3, 1]
_.map({'a': 1, 'b': 2, 'c': 3}, function(i){ return i * i; }) //[1, 4, 9]
_.map({ first: {'a': 1, 'b': 2}, second: {'b': 2, 'c': 3}, third: {'a': 1, 'c': 3}}, {'b': 2}) //[true, true, false]
_.map({ first: {'a': 1, 'b': 2}, second: {'b': 2, 'c': 3}, third: {'a': 1, 'c': 3}}, 'b') //[2, 2, undefined]
```

###_.reduce(list, iteratee, [memo], [context]), _.reduceRight(list, iteratee, [memo], [context])

将list中的每一项通过iteratee的方法叠加起来，如果传入memo的话，那么memo是默认的初始值，如果没有传入memo，那么在_.reduce中是第一项、_.reduceRight中是最后一项作为初始值

```javascript
function createReduce(dir){
	//以下定义了迭代器，memo就是一开始的默认开始积累的值
	function iterator(obj, iteratee, memo, keys, index, length){
		for(; index >= 0 && index < length; index += dir){
			var currentKey = keys ? keys[index] : index;
			//下面就是将进行过一次操作之后的值作为新的memo值，然后后面持续这样积累
			memo = iteratee[memo, obj[currentKey], currentKey, obj];
		}
		return memo;
	}
	return function(obj, iteratee, memo, context){
		iteratee = optimizeCb(iteratee, context, 4);
		var keys = !isArrayLike(obj) && _.keys(obj);
			length = (keys || obj).length,
			index = dir > 0 ? 0 : length - 1;
		//如果arguments不足三个，就说明没有传入memo，那么reduce就将obj的第1项视为memo；而reduceRight就将obj的最后一项视为memo
		if(arguments.length < 3){
			memo = obj[keys ? keys[index] : index];
			index += dir;
		}
		return iterator(obj, iteratee, memo, keys, index, length);
	}
}

_.reduce = _.foldl = _.inject = createReduce(1);
_.reduceRight = _.foldr + createReduce(-1);
```

```javascript
_.reduce([1, 2, 3], function(a, b){ return a + b; }, 0); //6
_.reduce([1, 2, 3], function(a, b){ return a + b; }, 10); //16
_.reduce([1, 2, 3], function(a, b){ return '' + a + b; }); //'321'
```

###_.find(list, predicate, [context])

//返回list中第一个能通过predicate的项

```javascript
_.find = _.detect = function(obj, predicate, context){
	var key;
	if(isArrayLike(obj)){
		key = _.findIndex(obj, predicate, context);
	} else {
		key = _.findKey(obj, predicate, context);
	}
	if(key !== void 0 && key !== -1) return obj[key];
}
```

可以看到这个方法其实是分别调用了`_.findIndex`和`_.findKey`两个方法。这两个方法到后面看到对于数组和对象的操作时，再分别说。


###_.filter(list, predicate, [context])

将list中能够通过predicate的所有项组成一个数组返回

```javascript
_.filter = _.select = function(obj, predicate, context){
	var results = [];
	predicate = cb(predicate, context);
	_.each(obj, function(value, index, list){
		if(predicate(value, index, list)) results.push(value);
	});
	return results;
}
```

注意，这里在源码中用的是cb，这说明predicate这里除了函数以外，也可以传入对象、数字或字符串。如果传入的是对象，那么其实就是看各项是否包含该传入对象的键值对；如果传入的是数字或字符串，那么就是看各项中以这个数字/字符串为key的值是否为true

```javascript
_.filter([[1, 2, 3], [2, 3, 1], [3, 1, 2]], [2]) //会返回[[2, 3, 1]]
_.filter([[1, 2, 3], [2, 3, 1], [3, 1, 2]], 2) //会返回整个[[1, 2, 3], [2, 3, 1], [3, 1, 2]]，因为各项的“key”为2的值分别是3, 1, 2，都为true，所以都被返回来了。
_.filter([[1, 2, 0], [2, 3, 0], [3, 1, 2]], 2) //这时候返回的就是[[3, 1, 2]]
```

###_.reject(list, predicate, [context])

与_.filter方法相反，返回的数组是由那些通不过predicate的项组成的

```javascript
_.reject = function(obj, predicate, context){
	return _.filter(obj, _.negate(cb(predicate)), context);
}
```

该方法中引用了_.negate方法。这个方法是返回一个新的函数，这个函数与原来的predicate求得的值正好相反。

```javascript
_.negate = function(predicate){
	return function(){
		return !predicate.apply(this, arguments);
	}
}
```

###_.every(list, [predicate], context)

如果list中的每一项都通过了predicate，那么就返回true，否则返回false

```javascript
_.every = _.all = function(obj, predicate, context){
	predicate = cb(predicate, context);
	var keys = !isArrayLike(obj) && _.keys(obj),
		length = (keys || obj).length;
	for(var index = 0; index < length; index++){
		var currentKey = keys ? obj[index] : index;
		if(!predicate(obj[currentKey], currentKey, obj)) return false;
	}
	return true;
}
```

注意，在这个方法中predicate也可以不传的，不传的话，就是看list中的每一项本身是否为真。predicate除了是function以外，也可以传对象，那么就是看各项是否都包含该对象的键值对；也可以传数值或字符串，那就是看各项中以该数值/字符串为key的值是否都为真。

###_.some(list, [predicate], [context])

list中，只有有一项可以通过predicate，那么就返回真，否则就返回假。

```javascript
_.some = _.any = function(obj, predicate, context){
	predicate = cb(predicate, context);
	var keys = !isArrayLike(obj) && _.keys(obj),
		length = (keys || obj).length;
	for(var index = 0; index < length; index++){
		var currentKey = keys ? keys[index] : index;
		if(predicate(obj[currentKey], currentKey, obj)) return true;
	}
	return false;
}
```

写法与_.every方法极为类似，不再赘述。

###_.contains(list, value, [fromIndex])

如果value存在于list中，那么返回true。可以传入fromIndex来确定从哪里开始检索list。

```javascript
_.contains = _.includes = _.include = function(obj, item, fromIndex, guard){
	if(!isArrayLike(obj)) obj = _.values(obj); //_.values函数返回传入对象的值所组成的数组
	if(typeof fromIndex != 'number' || guard) fromIndex = 0;
	return _.indexOf(obj, item, fromIndex) >= 0;
}
```

**☆ 这里的guard是干什么用的？**

###_.invoke(list, methodName, *arguments)

在list的每一项上调用名为methodName的函数，如果有额外的参数，可以在后面传进去，在调用methodName的函数的时候，会转传进去。

```javascript
_.invoke = function(obj, method){
	var args = slice.call(arguments, 2);
	var isFunc = _.isFunction(method);
	return _.map(obj, function(value)){
		var func = isFunc ? method : value[method];
		return func == null ? func : func.apply(value, args);
	}
}
```

可以看到，传入的method既可以是个函数，也可以是个字符串。举个例子：

```javascript
_.invoke([[3, 1, 2], [7, 8, 5, 9]], 'sort'); //[[1, 2, 3], [5, 7, 8, 9]]
```

在上面这个例子中，传入的`'sort'`是字符串，走到源码当中，通过`var func = isFunc ? method : value[method]`来判断。因为method不是函数，所以func最终取了value[method]的值，对应到上面的例子来说就是`[3, 1, 2]['sort']`和`[7, 5, 8, 9]['sort']`。这两个返回的都是数组原型链上的原生的sort方法，即`[].sort`这个函数。

上面那个例子其实相当于：

```javascript
_.invoke([[3, 1, 2], [7, 8, 5, 9]], [].sort);
```

从源码中看到，_.invoke就是调用的_.map，但是与_.map不同的是，_.invoke还可以为iteratee这个函数传入额外的参数。例如：

```javascript
_.invoke([[3, 1, 2], [7, 8, 5, 9]], 'sort', function(a, b){ return b - a; }) //[[3, 2, 1], [9, 8, 7, 5]]
_.invoke([[3, 1, 2], [5, 7, 8, 2]], 'concat', 'a', 'b', 'c') //[[3, 1, 2, 'a', 'b', 'c'], [5, 7, 8, 2, 'a', 'b', 'c']]
_.invoke([[3, 1, 2], [5, 7, 8, 2]], 'push', 'a', 'b', 'c') //[6, 7] 返回[6, 7]是因为push方法返回的值是新数组的长度
```

###_.pluck(list, propertyName)

将list中，属性名为propertyName的值都抽出来组成一个数组返回。

```javascript
_.pluck = function(obj, key){
	return _.map(obj, _.property(key));
}
```

用法如下：

```javascript
var stooges = [{name: 'moe', age: 40}, {name: 'larry', age: 50}, {name: 'curly', age: 60}];
_.pluck(stooges, 'name');
=> ["moe", "larry", "curly"]
```

###_.where(list, properties)

在list中看每一项，如果这个项包含properties中所有的键值对的话，那么就把这个项过滤出来放到一个数组里，最后返回的数组里包含所有符合这种要求的项

例如：

```javascript
var listOfPlays = [
	{title: "Cats", author: "Webber", year: "I do not know"},
	{title: "Cymbeline", author: "Shakespeare", year: 1611},
	{title: "another play", author: "whoever", year: "1933"},
    {title: "The Tempest", author: "Shakespeare", year: 1611}
];
_.where(listOfPlays, {author: "Shakespeare", year: 1611});
/*返回
[{title: "Cymbeline", author: "Shakespeare", year: 1611},
 {title: "The Tempest", author: "Shakespeare", year: 1611}]
*/
```

源码：

```javascript
_.where = function(obj, attrs){
	return _.filter(obj, _.matcher(attrs));
}
```

###_.findWhere(list, properties)

在list里面找，返回第一个匹配properties中所有键值对的值。

在源码中，调用的还是`_.find`方法：

```javascript
_.findWhere = function(obj, attrs){
	return _.find(obj, _.mathcer(attrs));
}
```

###_.max(list, [iteratee], [context])

返回list中最大的值，如果提供了iteratee的话，那么iteratee作用在list的每一项上锁返回的值作为比较大小的标准。如果list为空的话，返回`-Infinity`。非数值类型的list会被忽略。

用法：

```javascript
var stooges = [{name: 'moe', age: 40}, {name: 'larry', age: 50}, {name: 'curly', age: 60}];
_.max(stooges, function(stooge){ return stooge.age; });
//返回 {name: 'curly', age: 60}
```

源码：

```javascript
_.max = function(obj, iteratee, context){
	var result = -Infinity, lastComputed = -Infinity,
		value, computed;
	if(iteratee == null && obj != null){
		obj = isArrayLike(obj) ? obj : _.values(obj);
		for(var i = 0, length = obj.length; i < length; i++){
			value = obj[i];
			if(value > result){
				result = value;
			}
		}
	} else {
		iteratee = cb(iteratee, context);
		_.each(obj, function(value, index, list){
			computed = iteratee(value, index, list);
			if(computed > lastComputed || computed === -Infinity && result === -Infinity){
				result = value;
				lastComputed = computed;
			}
		});
	}
	return result;
}
```

关于三目运算，两个三目运算并在一起了，怎么个看法：

```javascript
1 || 2 && 0; //1 相当于 1 || (2 && 0)
0 || 1 && 2; //2 相当于 0 || (1 && 2)
false || 0 && 1; //0 相当于 false || (0 && 1)
```

###_.min(list, [iteratee], [context])

用法和源码的写法与`_.max`方法正好相反。

```javascript
_.min = function(obj, iteratee, context){
	var result = Infinity, lastComputed = Infinity, value, computed;
	if(iteratee == null && obj != null){
		obj = isArrayLike(obj) ? obj : _.values(obj);
		for(var i = 0, length = obj.length; i < length; i++){
			value = obj[i];
			if(value < result){
				result = value;
			}
		}
	} else {
		iteratee = cb(iteratee, context);
		_.each(obj, function(value, index, list){
			computed = iteratee(value, index, list);
			if(computed < lastComputed || computed === Infinity && result === Infinity){
				result = value;
				lastComputed = computed;
			}
		})
	}
	return result;
}
```

###_.shuffle(list)

“shuffle”在英文中就是洗牌的意思，也就是说这个方法就是要把list中各项的顺序打乱

```javascript
_.shuffle([1, 2, 3, 4, 5, 6]); //[4, 1, 6, 3, 5, 2]
```

源码：

```javascript
_.shuffle = function(obj){
	var set = isArrayLike(obj) ? obj : _.values(obj);
	var length = set.length;
	var shuffled = Array(length);
	for(var index = 0, rand; index < length; index++){
		rand = _.random(0, index);
		if(rand !== index) shuffled[index] = shuffled[rand];
		shuffled[rand] = set[index];
	}
	return shuffled;
}
```

要理解上面的算法，我们可以把每次的相关数值都打印出来：

```javascript
_.shuffle([1, 2, 3]);
/* 第一次循环
index: 0	random: 0
shuffled[index]: 1	shuffled[rand]: 1	set[index]: 1
shuffled: 1,,
*/
/* 第二次循环
index: 1	random: 1
shuffled[index]: 2	shuffled[rand]: 2	set[index]: 2
shuffled: 1, 2,
*/
/* 第三次循环
index: 2	random: 0
shuffled[index]: 1	shuffled[rand]: 3	set[index]: 3
shuffled: 3, 2, 1
*/
//最终返回[3, 2, 1]
```

可见在第三次循环中，index与rand的数值不相同，于是shuffled[2]的位置就被赋值为shuffled[0]，也就是数组的最后一个数值1；接着，原来shuffled[0]就被赋值为最初的set[2]的值，即3。

###_.sample(list, [n])

返回n个随机的list中的项。如果n没有传，就默认返回随机的其中一项。

```javascript
_.sample = function(obj, n, guard){
	if(n === null || guard){
		if(!isArrayLike(obj)) obj = _.values(obj);
		return obj[_.random(obj.length - 1)];
	}
	return _.shuffle(obj).slice(0, Math.max(0, n));
}
```

此处的guard是为了_.sample也可以跟_.map连用。比如：

```javascript
_.map([[1, 2, 3], [4, 5, 6], [7, 8, 9]], _.sample); //[2, 6, 8]
//这个时候，传入_.sample的参数分别是每一项，每一项的index，还有这个对象。这时候这个n !== null，于是在进行if判断的时候走了guard这边，此时的guard就是[[1, 2, 3], [4, 5, 6], [7, 8, 9]]，if判断为真，于是正确返回了我们想要的结果。
```

如果源码中没有guard的话，`_.map([[1, 2, 3], [4, 5, 6], [7, 8, 9]], _.sample); `返回的就是`[[], [6], [7, 9]]`这样的结果，因为此处传入`_.sample`的index被误认为要返回的值的数量，从而得到这种有点匪夷所思的结果。

###_.sortBy(list, iteratee, [context])

返回list的一个拷贝，其中各项是按照iteratee运行在各项上得到的结果的升序排列的。iteratee也可以是属性名。

```javascript
_.sortBy([1, 2, 3, 4, 5, 6], function(num){ return Math.sin(num); });
//[5, 4, 6, 3, 1, 2]

var stooges = [{name: 'moe', age: 40}, {name: 'larry', age: 50}, {name: 'curly', age: 60}];
_.sortBy(stooges, 'name');
//[{name: 'curly', age: 60}, {name: 'larry', age: 50}, {name: 'moe', age: 40}];
```

源码：

```javascript
_.sortBy = function(obj, iteratee, context){
	iteratee = cb(iteratee, context);
	return _.pluck(_.map(obj, function(value, index, list){
		return {
			value: value,
			index: index,
			criteria: iteratee(value, index, list)
		}
	}).sort(function(left, right){
		var a = left.criteria;
		var b = right.criteria;
		if(a !== b){
			if(a > b || a === void 0) return 1;
			if(a < b || b === void 0) return -1;
		}
		return left.index - right.index;
	}), 'value')
}
```

###_.groupBy(list, iteratee, [context])

将list分成几组，以各项运行iteratee之后的结果作为分组依据。如果iteratee是个字符串，而不是函数的话，那么就以各项以这个iteratee为属性的值为依据进行分组。

```javascript
_.groupBy([1.3, 2.1, 2.4], function(num){ return Math.floor(num); });
// {1: [1.3] 2: [2.1, 2.4]}

_.groupBy(['one', 'two', 'three'], 'length');
//{3: ["one", "two"], 5: ["three"]}
```

源码：

```javascript
var group = function(behavior){
	return function(obj, iteratee, context){
		var result = {};
		iteratee = cb(iteratee, context);
		_.each(obj, function(value, index){
			var key = iteratee(value, index, obj);
			behavior(result, value, key);
		})
		return result;
	}
}

_.groupBy = group(function(result, value, key){
	if(_.has(result, key)) result[key].push(value); else result[key] = [value];
})
```

###_.indexBy(list, iteratee, [context])

给定一个list，还有一个iteratee，这个iteratee运行在list的每一项上时，返回一个key或属性名，然后返回一个对象，对象中每一项都是以这个key为键名的。它跟groupBy很像，但是当你知道返回的key都是唯一的时候，就可以用它。

```javascript
var stooges = [{name: 'moe', age: 40}, {name: 'larry', age: 50}, {name: 'curly', age: 60}];
_.indexBy(stooges, 'age');
/*
{
  "40": {name: 'moe', age: 40},
  "50": {name: 'larry', age: 50},
  "60": {name: 'curly', age: 60}
}
*/
```

源码：

```javascript
_.indexBy = group(function(result, value, key){
	result[key] = value;
});
```

###_.countBy(list, iteratee, [context])

跟groupBy有点类似，但是返回的是每个组里面的元素的数量

```javascript
_.countBy([1, 2, 3, 4, 5], function(num){
	return num % 2 == 0 ? 'even': 'odd';
});
//{odd: 3, even: 2}
```

源码：

```javascript
_.countBy = group(function(result, value, key){
	if(_.has(result, key)) result[key]++; else result[key] = 1;
})
```

###_.toArray(list)

根据list创建一个真正的数组

```javascript
_.toArray = function(obj){
	if(!obj) return [];
	if(_.isArray(obj)) return slice.call(obj);
	if(isArrayLike(obj)) return _.map(obj, _.identity);
	return _.values(obj);
}
```

###_.size(list)

返回list中有多少个值

```javascript
_.size({one: 1, two: 2, three: 3}); //3
```

源码：

```javascript
_.size = function(obj){
	if(obj == null) return 0;
	return isArrayLike(obj) ? obj.length : _.keys(obj).length;
}
```

###_.partition(array, predicate)

将一个数组分成两个数组，其中一个是通过了predicate的元素组成的，另外一个是未通过predicate的元素组成的

```javascript
_.partition([0, 1, 2, 3, 4, 5], isOdd);
//[[1, 3, 5], [0, 2, 4]]
```

源码：

```javascript
_.partition = function(obj, predicate, context){
	predicate = cb(predicate, context);
	var pass = [], fail = [];
	_.each(obj, function(value, key, obj){
		(predicate(value, key, obj) ? pass : fail).push(value);
	});
	return [pass, fail];
}
```

##用于数组的一些方法

###_.first(array, [n])

返回数组中前n个元素，没有n就默认返回1个

```javascript
_.first = _.head = _.take = function(array, n, guard){
	if(array == null) return void 0;
	if(n == null || guard) return array[0];
	return _.initial(array, array.length - n);
}
```

此处guard的作用在前面`_.sample`中讲过了，作用是一样的。

源码中调用了`_.initial`方法。

###_.initial(array, [n])

返回的数组中将最后n的元素去掉了，如果n没有传入的话，那么就是把最后一个去掉。

```javascript
_.initial = function(array, n, guard){
	return slice.call(array, 0, Math.max(0, array.length - (n == null || guard ? 1 : n)));
}
```

###_.last(array, [n])

返回数组的最后第n个元素，n没有的话，默认返回最后一个

```javascript
_.last = function(array, n, guard){
	if(array == null) return void 0;
	if(n == null || guard) return array[array.length - 1];
	return _.rest(array, Math.max(0, array.length - n)); 
}
```

源码中调用了`_.rest`方法。

###_.rest(array, [index])

返回从index元素开始一直到最后的所有元素，如果没有传入index，那么就是从第2个（index为1）开始返回后面所有的元素

```javascript
_.rest = _.tail = _.drop = function(array, n, guard){
	return slice.call(array, n == null || guard ? 1 : n);
}
```

###_.compact(array)

返回array的一个副本，并且将其中所有的falsy value都去掉。在JS中，`false`、`null`、`0`、`""`、`undefined`和`NaN`都是falsy value。

```javascript
_.compact = function(array){
	return _.filter(array, _.identity);
}
```

##_.flatten(array, [shallow])

将嵌套的数组弄“平”。如果传入的shallow为true，那么，数组就是会被弄“平”一层。

```javascript
_.flatten([1, [2], [3, [[4]]]]); //[1, 2, 3, 4]
_.flatten([1, [2], [3, [[4]]]], true); //[1, 2, 3, [[4]]]
```

源码：

```javascript
var flatten = function(input, shallow, strict, startIndex){
	var output = [], idx = 0;
	for(var i = startIndex || 0, length = getLength(input); i < length; i++){
		var value = input[i];
		if(isArrayLike(value) && (_.isArray(value || _.isArguments(value)))){
			if(!shallow) value = flatten(value, shallow, strict);
			var j = 0, len = value.length;
			output.length += len;
			while(j < len){
				output[idx++] = value[j++];
			}
		} else if(!strict){
			output[idx++] = value;
		}
	}
	return output;
}

_.flatten = function(array, shallow){
	return flatten(array, shallow, false);
}
```

可以看到，内部的`flatten`方法用到了递归。将数组中的每一个value都flatten，当value不再是数组或类数组后，就会走到`else if(!strict)`这个分支中来，将一个个值添加到最终的结果数组中。如果没有那个`else if(!strict)`这个分支的话，当深度flatten的时候，最早最下面一层，必然会返回非数组的value，如果没有这个分支，这些非数组的value就加不到结果数组中。也就是说，如果没有这个分支，那么当`shallow`为false的时候，就一定会返回空数组。

###_.without(array, *values)

返回这个数组的拷贝，并且将里面为*values的值都去掉

```javascript
_.without([1, 2, 1, 0, 3, 1, 4], 0, 1); //[2, 3, 4]
```

源码：

```javascript
_.without = function(array){
	return _.difference(array, slice.call(arguments, 1));
}
```

源码中调用了`_.difference`方法。

###_.difference(array, *others)

与without类似，但是返回的是数组中不再*others数组中的值所构成的数组

```javascript
_.difference([1, 2, 3, 4, 5], [5, 2, 10]); //[1, 3, 4]
```

源码：

```javascript
_.difference = function(array){
	var rest = flatten(arguments, true, true, 1); //flatten(input, shallow, strict, 1) 这一步中，shallow设置为true，即将arguments从第2个参数开始（因为startIndex为1），展开一层。又因为strict设置为true，那么如果参数中有非arraylike的对象的话，是不会出现在rest里面的。
	//例如：_.difference([1, 2, 3, 4, 5], [3, 5], [1, 3], 10) 它的在运行到这一步的时候，rest其实是[3, 5, 1, 3]，其中10就没有加进来。因为strict设置为true，10在flatten的两个分支里，哪个都走不通，所以没有添加进来。
	return _.filter(array, function(value){
		return !_.contains(rest, value);
	})
}
```















