# underscore 1.8.3 源码解读

<!-- MarkdownTOC -->

- 一些基本设置
	- `_`构造函数
	- optimizeCb
	- cb
	- _.identity
	- _.matcher
	- _.property
	- _.iteratee
	- createAssigner
	- _.keys
	- _.allKeys
	- _.extend, _.extendOwn, _.defaults
	- baseCreate
- 用于集合的一些方法
	- _.each\(list, iteratee, \[context\]\)
	- _.map\(list, iteratee, \[context\]\)
	- _.reduce\(list, iteratee, \[memo\], \[context\]\), _.reduceRight\(list, iteratee, \[memo\], \[context\]\)
	- _.find\(list, predicate, \[context\]\)
	- _.filter\(list, predicate, \[context\]\)
	- _.reject\(list, predicate, \[context\]\)
	- _.every\(list, \[predicate\], context\)
	- _.some\(list, \[predicate\], \[context\]\)
	- _.contains\(list, value, \[fromIndex\]\)
	- _.invoke\(list, methodName, *arguments\)
	- _.pluck\(list, propertyName\)
	- _.where\(list, properties\)
	- _.findWhere\(list, properties\)
	- _.max\(list, \[iteratee\], \[context\]\)
	- _.min\(list, \[iteratee\], \[context\]\)
	- _.shuffle\(list\)
	- _.sample\(list, \[n\]\)
	- _.sortBy\(list, iteratee, \[context\]\)
	- _.groupBy\(list, iteratee, \[context\]\)
	- _.indexBy\(list, iteratee, \[context\]\)
	- _.countBy\(list, iteratee, \[context\]\)
	- _.toArray\(list\)
	- _.size\(list\)
	- _.partition\(array, predicate\)
- 用于数组的一些方法
	- _.first\(array, \[n\]\)
	- _.initial\(array, \[n\]\)
	- _.last\(array, \[n\]\)
	- _.rest\(array, \[index\]\)
	- _.compact\(array\)
- _.flatten\(array, \[shallow\]\)
	- _.without\(array, *values\)
	- _.difference\(array, *others\)
	- _.uniq\(array, \[isSorted\], \[iteratee\]\)
	- _.union\(*arrays\)
	- _.intersection = function\(*array\)
	- _.zip\(*arrays\)
	- _.unzip\(array\)
	- _.object\(list, \[value\]\)
	- _.findIndex\(array, predicate, \[context\]\)
	- _.findLastIndex\(array, predicate, \[context\]\)
	- _.sortedIndex\(list, value, \[iteratee\], \[context\]\)
	- _.indexOf\(array, value, \[isSorted\]\)
	- _.lastIndexOf
	- _.range\(\[start\], stop, \[step\]\)
- 用于函数的一些方法
	- _.bind\(function, obj, *argument\)
	- _.bindAll\(obj, *methodNames\)
	- _.partial\(function, *arguments\)
	- _.memoize\(function, \[hashFunction\]\)
	- _.delay\(function, wait, *arguments\)
	- _.defer\(function, *arguments\)
	- _.throttle\(function, wait, \[options\]\)
	- _.debounce\(function, wait, \[immediate\]\)
	- _.wrap\(function wrapper\)
	- _.negate\(predicate\)
	- _.compose\(*functions\)
	- _.after\(count, function\)
	- _.before\(count, function\)
	- _.once\(function\)
- 用于对象的一些方法
	- _.values\(object\)
	- _.mapObject\(object, iteratee, \[context\]\)
	- _.pairs\(object\)
	- _.invert\(object\)
	- _.functions\(object\)
	- _.findKey\(object, predicate, \[context\]\)
	- _.pick\(object, *keys\)
	- _.omit\(object, *keys\)
	- _.defaults\(object, *defaults\)
	- _.create\(prototype, props\)
	- _.clone\(object\)
	- _.tap\(object, interceptor\)
	- _.isMatch\(object, properties\)
	- _.isEqual\(object, other\)
	- _.isEmpty\(object\)
	- _.isElement\(object\)
	- _.isArray\(object\)
	- _.isObject\(object\)
	- _.isArguments, _.isFunction, _.isString, _.isNumber, _.isDate, _.isRegExp, _.isError
	- _.isFinite\(object\)
	- _.isNaN\(object\)
	- _.isBoolean\(object\)
	- _.isNull\(object\)
	- _.isUndefined\(object\)
	- _.has\(object, key\)
- 其他一些实用方法
	- _.noConflict\(\)
	- _.identity\(value\)
	- _.constant\(value\)
	- _.noop\(\)
	- _.property\(key\)
	- _.propertyOf\(object\)
	- _.matcher\(attrs\)
	- _.times\(n, iteratee, \[context\]\)
	- _.random\(min, max\)
	- _.now\(\)README.md
	- _.escape\(string\), _.unescape\(string\)
	- _.result\(object, property, \[defaultValue\]\)
	- _.uniqueId\(\[prefix\]\)
	- _.template\(templateString, \[settings\]\)
	- _.chain\(obj\), _.chain\(obj\).value\(\)
- 有关OOP的一些代码
	- _.mixin\(object\)

<!-- /MarkdownTOC -->



## 一些基本设置 

underscore的源码总体结构是 `function(){}.call(this)`

```javascript
var root = this;
```

在浏览器环境下，this就是`window`，如果是在node环境下，this就是`exports`

### `_`构造函数

```javascript
var _ = function(obj){
	if(obj instanceof _) return obj;
	if(! (this instanceof _)) return new _(obj);
	this._wrapped = obj;
}
```

上述代码中我们看到了 `new` ，这说明 _ 这个函数其实是一个构造函数。代码写到这里，如果我们为这个构造函数传入一个对象 `[1, 2, 3]` 会得到一个 _ 的实例对象：`{_wrapped: [1, 2, 3]}`，该实例的 constructor 就是 _，而原型就是Object。如果我们对 `_(_([1, 2, 3]))`求值，得到的还是 `{_wrapped: [1, 2, 3]}`，因为已经是 _ 的实例，就直接把这个对象返回了。

导出underscore对象

```javascript
if(typeof exports != 'undefined'){
	if(typeof module !== 'undefined' && module.exports){
		exports = module.exports = _;
	}
	exports._ = _;
} else {
	root._ = _; //如果是浏览器的话，那么root就是window对象，这时候就将_对象挂载在window._属性下。
}
```

### optimizeCb

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

### cb

```javascript
var cb = function(value, context, argCount){
	if(value == null) return _.identity;
	if(_.isFunction(value)) return optimizeCb(value, context, argCount);
	if(_.isObject(value)) return _.matcher(value);
	return _.property(value);
}
```

### _.identity

`_.identity`这个函数的作用就是将传入的值原样返回。
```
var stooge = {name: 'moe'};
stooge === _.identity(stooge); //true
```

### _.matcher

`_.matcher(attrs)`这个函数的作用就是返回一个predicate function，这个predicate function接收object作为参数之后，返回值就是是否这个object包含attrs中的所有键值对属性。
```javascript
var func = _.matcher({name: "Amanda", age: 17});
func({name: "Amanda", age: 18}); //false
func({name: "Amanda", age: 17, sex: "female"}); //true
```

### _.property

`_.property(key)`这个函数的作用就是返回一个函数，这个函数接收一个object作为参数之后，返回这个object中key属性的值。
```javascript
var stooge = {name: 'moe'};
_.property('name')(stooge); //'moe'
```

### _.iteratee

```javascript
_.iteratee = function(value, context){ return cb(value, context, Infinity); }
```

_.iteratee的用法跟作用其实跟内部的cb差不多。分为下面四种情况：

- 没有传入任何参数 `_.iteratee()`，那么返回的就是 _.identity()函数
- 传入的参数是个函数 `_.iteratee(f)`，那么返回的就是一个optimized callback，其实就是`optimizeCb(f)`
- 传入的是对象`_.iteratee(obj)`，那么返回的就是`_.matcher(obj)`，也就是一个predicate函数，这个函数会判断所传入的参数是否包含obj所有的key/value键值对
	- 传入的是其他值，如`_.iteratee(prop)`，返回的是`_.property(prop)`函数，这个函数会返回传入参数的属性为prop的值。

### createAssigner

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

### _.keys

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

### _.allKeys

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

### _.extend, _.extendOwn, _.defaults

```javascript
_.extend = createAssigner(_.allKeys); //不管三七二十一，后面的obj上的属性（包括自己的和原型链上的）都拷贝过来
_.extendOwn = _.assign = createAssigner(_.keys); //只拷贝后面obj上属于自己的属性，原型链上的不拷贝
_.defaults = createAssigner(_.allKeys, true); //拷贝后面obj上的所有属性（包括原型链上的），但是如果遇到了重复的属性，则以原来的为准，此处undefinedOnly这里就传入了true
```

可以看出，underscore中的`_.extend`，`_.extendOwn`，`_.defaults`实现的都是浅拷贝。

### baseCreate

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

原理与简化版的`Object.create`的polyfill一样：

```javascript
//简化版Object.create
Object.create = Object.create || function(o){
    fundtion F(){}; //一个空函数作为构造器
    F.prototype = o; //将该构造器的原型设为传入的原型o
    return new F();
}
```

```javascript
var isArrayLike = function(collection){
	var length = getLength(collection);
	return typeof length == 'number' && length >= 0 && length <= MAX_ARRAY_INDEX;
}
```

在underscore中，只有一个对象具备length属性，并且length属性的值是数字，并且是在最大范围允许的情况下，就被认为这个对象是arraylike的。

## 用于集合的一些方法

### _.each(list, iteratee, [context])

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

### _.map(list, iteratee, [context])

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

### _.reduce(list, iteratee, [memo], [context]), _.reduceRight(list, iteratee, [memo], [context])

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

### _.find(list, predicate, [context])

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


### _.filter(list, predicate, [context])

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

### _.reject(list, predicate, [context])

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

### _.every(list, [predicate], context)

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

### _.some(list, [predicate], [context])

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

### _.contains(list, value, [fromIndex])

如果value存在于list中，那么返回true。可以传入fromIndex来确定从哪里开始检索list。

```javascript
_.contains = _.includes = _.include = function(obj, item, fromIndex, guard){
	if(!isArrayLike(obj)) obj = _.values(obj); //_.values函数返回传入对象的值所组成的数组
	if(typeof fromIndex != 'number' || guard) fromIndex = 0;
	return _.indexOf(obj, item, fromIndex) >= 0;
}
```

其他方法中的guard，通常是为了让方法能在`_.map`方法中使用而设计的。但是此处及时没有这个guard，也不妨碍`_.contains`在`_.map`中的使用。但是guard的作用正如其英文的意义所言，就是一道防线。例如：`_.contains`在作为参数传入到其他方法中时，万一正好在"fromIndex"的这个位置，传入的并不是我们真正想要的值，这时候由于guard的存在，依然能够正确地将fromIndex设置为默认值0，避免发生错误。

### _.invoke(list, methodName, *arguments)

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

### _.pluck(list, propertyName)

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

### _.where(list, properties)

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

### _.findWhere(list, properties)

在list里面找，返回第一个匹配properties中所有键值对的值。

在源码中，调用的还是`_.find`方法：

```javascript
_.findWhere = function(obj, attrs){
	return _.find(obj, _.mathcer(attrs));
}
```

### _.max(list, [iteratee], [context])

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

### _.min(list, [iteratee], [context])

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

### _.shuffle(list)

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

### _.sample(list, [n])

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

### _.sortBy(list, iteratee, [context])

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

### _.groupBy(list, iteratee, [context])

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

### _.indexBy(list, iteratee, [context])

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

### _.countBy(list, iteratee, [context])

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

### _.toArray(list)

根据list创建一个真正的数组

```javascript
_.toArray = function(obj){
	if(!obj) return [];
	if(_.isArray(obj)) return slice.call(obj);
	if(isArrayLike(obj)) return _.map(obj, _.identity);
	return _.values(obj);
}
```

### _.size(list)

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

### _.partition(array, predicate)

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

## 用于数组的一些方法

### _.first(array, [n])

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

### _.initial(array, [n])

返回的数组中将最后n的元素去掉了，如果n没有传入的话，那么就是把最后一个去掉。

```javascript
_.initial = function(array, n, guard){
	return slice.call(array, 0, Math.max(0, array.length - (n == null || guard ? 1 : n)));
}
```

### _.last(array, [n])

返回数组的最后第n个元素，n没有的话，默认返回最后一个

```javascript
_.last = function(array, n, guard){
	if(array == null) return void 0;
	if(n == null || guard) return array[array.length - 1];
	return _.rest(array, Math.max(0, array.length - n)); 
}
```

源码中调用了`_.rest`方法。

### _.rest(array, [index])

返回从index元素开始一直到最后的所有元素，如果没有传入index，那么就是从第2个（index为1）开始返回后面所有的元素

```javascript
_.rest = _.tail = _.drop = function(array, n, guard){
	return slice.call(array, n == null || guard ? 1 : n);
}
```

### _.compact(array)

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

### _.without(array, *values)

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

### _.difference(array, *others)

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

### _.uniq(array, [isSorted], [iteratee])

该方法会产生一个没有重复项的数组。如果事先知道传入的第一个参数数组是已经排序的，那么可以在isSorted的参数位置传入`true`，这样会使用一种更为快速的算法。如果是否是唯一的标准是iteratee运行在数组之中每一项的结果决定的，那么就传入iteratee参数。

```javascript
_.uniq([1, 2, 1, 4, 1, 3]); //[1, 2, 4, 3]
_.uniq([1, 2, 3, 4, 5], false, function(n){ return n % 2; }); //[1, 2]
```

源码：

```javascript
_.uniq = _.unique = function(array, isSorted, iteratee, context){
    if(!_.isBoolean(isSorted)){
        context = iteratee;
        iteratee = isSorted;
        isSorted = false;
    }
    if(iteratee != null) iteratee = cb(iteratee, context);
    var result = [];
    var seen = [];
    for(var i = 0, length = getLength(array); i < length; i++){
        var value = array[i],
            computed = iteratee ? iteratee(value, i, array) : value;
        if(isSorted){
            if(!i || seen !== computed) result.push(value);
            seen = computed;
        } else if(iteratee){
            if(!_.contains(seen, computed)){
                seen.push(computed);
                result.push(value);
            }
        } else if(!_.contains(result, value)){
            result.push(value);
        }
    }
    return result;
}
```

看到上面的写法，感觉有一点问题。例如：

```javascript
_.uniq([1, 2, 3, 4, 5], true, function(n){return n%2; });
//[1, 2, 3, 4, 5]
_.uniq([1, 2, 3, 4, 5], false, function(n){return n%2; });
//[1, 2]
```

可见，以上情况中，当isSorted传入false使，返回的是我们想要的结果，但是传入true是，是错误的结果。这是因为，在源码中，当1经过iteratee之后，得到的数值为1，那么这时候seen就保存为0了，然后将1加入到result中去了。接下来，2经过iteratee的0。这个computed的0不等于seen的1，于是2也被加入到result中去了，同时seen又被存成了0。接下来到3，经过iteratee，得到的computed又变成了1，与seen的0不相等，于是3又被存到result中去了。以此类推。

所以isSorted这个参数一定要慎用。而且这里究竟是看谁是sorted的？如果是看传入的第一个参数数组是否是sorted，这个经上例来看并不靠谱。这个isSorted当iteratee存在的时候，应该是看iteratee在每一项上走一遍，得到的结果所产生的的数组是否是sorted才对。

### _.union(*arrays)

传入多个数组，然后返回一个数组，这个数组里面包含所有数组里面的元素，而且是去重了的。顺序就是按照原本在各数组中出现的顺序。

```javascript

_.union([1, 2, 3], [101, 2, 1, 10], [2, 1]); //[1, 2, 3, 101, 10]
```

源码：

```javascript

_.union = function(){
    return _.uniq(flatten(arguments, true ,true));
}
```

### _.intersection = function(*array)

传入多个数组，返回一个数组，其中的元素在那多个数组中都出现过。

```javascript
_.intersection([1, 2, 3], [101, 2, 1, 10], [2, 1]); //[1, 2]
```

源码：

```javascript
_.intersection = function(array){
    var result = [];
    var argsLength = arguments.length;
    for(var i = 0, length = getLength(array); i < length; i++){
        for item = array[i];
        if(_.contains(result, item)) continue;
        for(var j = 1; j < argsLeng; j++){
            if(!_.contains(arguments[j], item)) break;
        }
        if(j === argsLength) result.push(item);
    }
    return result;
}
```

### _.zip(*arrays)

该方法将数组中每一项的值，按照对应位置拼在一起。如果你要操作的是一个嵌套的矩阵的话，那么这个方法可以用于转置矩阵。

```javascript
_.zip(['moe', 'larry', 'curly'], [30, 40, 50], [true, false, false]);
//[["moe", 30, true], ["larry", 40, false], ["curly", 50, false]]
```

源码：

```javascript
_.zip = function(){
    return _.unzip(arguments);
}
```

可见：`_.zip`源码中引用了`_.unzip`

### _.unzip(array)

与`_.zip`的用法正好相反。为`_.unzip`传入一个数组的数组，返回的是一系列新数组。第一个数组中包含的是原来所有数组中的第一个元素；第二个数组包含原来所有数组中的第二个元素；以此类推。

```javascript
_.unzip([["moe", 30, true], ["larry", 40, false], ["curly", 50, false]]);
//[['moe', 'larry', 'curly'], [30, 40, 50], [true, false, false]]
```

源码：

```javascript
_.unzip = function(array){
    var length = array && _.max(array, getLength).length || 0;
    var result = Array(length);

    for(var index = 0; index < length; index++){
        result[index] = _.pluck(array, index);
    }
    return result;
}
```

### _.object(list, [value])

将数组转为对象。如果有重复的键名存在的话，取最后出现的键值。

```javascript
_.object(['moe', 'larry', 'curly'], [30, 40, 50]); // {moe: 30, larry: 40, curly: 50}
_.object([['moe', 30], ['larry', 40], ['curly', 50]]); // {moe: 30, larry: 40, curly: 50}
```

```javascript
_.object = function(list, values){
    var result = {};
    for(var i = 0, length = getLength(list); i < length; i++){
        if(values){
            result[list[i]] = values[i];
        } else {
            result[list[i][0]] = list[i][1];
        }
    }
    return result;
}
```

### _.findIndex(array, predicate, [context])

从左向右，找到第一个通过predicate的元素的index

```javascript
_.findIndex([4, 6, 8, 12], isPrime); //-1
_.findIndex([4, 6, 7, 12], isPrime); //2
```

```javascript
function createPredicateIndex(dir){ //dir代表方向，1为从左向右，-1为从右向左
    return function(array, predicate, context){
        predicate = cb(predicate, context);
        var length = getLength(array);
        var index = dir > 0 ? 0 : length - 1;
        for(; index >= 0 && index < length; index += dir){
            if(predicate(array[index], index, array)) return index;
        }
        return -1;
    }
}

_.findIndex = createPredicateIndexFinder(-1);
```

### _.findLastIndex(array, predicate, [context])

从右向左，找到第一个通过predicate的元素的index

```javascript
_.findLastIndex = createPredicateIndexFiner(-1);
```

### _.sortedIndex(list, value, [iteratee], [context])

利用binary search来确定，这个value应该查到list的什么位置，并保持list的排序状态。如果iteratee提供了的话，那么排序依据就是list中各元素和value经过iteratee运行后的值。iteratee也可以是字符串类型，这时候它代表的就是list各项以及value的一个属性名。

```javascript
_.sortedIndex([10, 20, 30, 40, 50], 35); //3

var stooges = [{name: 'moe', age: 40}, {name 'curly', age: 60}];
_.sortedIndex(stooges, {name: 'larry', age: 50}, 'age'); //1
```

```javascript
_.sortedIndex = function(array, obj, iteratee, context){
    iteratee = cb(iteratee, context, 1);
    var value = iteratee(obj);
    var low = 0, high = getLength(array);
    while(low < high){
        var mid = Math.floor((low + high)/2);
        if(iteratee(array[mid]) < value) low = mid + 1; else high = mid;
    }
    return low;
}
```

从源码中我们看出，这个sortedIndex默认排序顺序就是升序的。因此会出现下面的结果：

```javascript
_.sortedIndex([5, 4, 3, 2, 1], 2.5); // 0 不是我们想要的结果
_.sortedIndex([1, 2, 3, 4, 5], 2.5); // 2
```

### _.indexOf(array, value, [isSorted])

返回value在array中的index，如果没有找到就返回-1。如果这个数组很大，而且你知道它是排序好了的，那么就可以在isSorted这里传入true，这样会使用binary search方法。参数第三个位置可以传入一个数字，代表从这个数字之后开始寻找匹配value的index

```javascript
_.indexOf([1, 2, 3], 2); // 1
```

```javascript
function createIndexFiner(dir, predicateFind, sortedIndex){
    return function(array, item, idx){
        var i = 0, length = getLength(array);
        if(typeof idx == 'number'){
            if(dir > 0){
                i = idx > 0 ? idx : Math.max(idx + length, i);
            } else {
                length = idx >= 0 ? Math.min(idx + 1, length) : idx + length + 1
            }
        } else if(sortedIndex && idx && length){
            idx = sortedIndex(array, item);
            return array[idx] === item ? idx : -1
        }
        if(item !== item){
            idx = predicateFind(slice.call(array, i, length), _.isNaN);
            return idx >= 0 ? idx + i : -1;
        }
        for(idx = dir > 0 ? i : length - 1; idx >= 0 && idx < length; idx += dir){
            if(array[idx] === item) return idx;
        }
        return -1
    }
}

_.indexOf = createIndexFinder(1, _.findIndex, _.sortedIndex);
```

### _.lastIndexOf

```javascript
createIndexFiner(-1, _.findLastIndex);
```

### _.range([start], stop, [step])

start如果没有传入的话，默认为0；step默认为1。该方法会返回一个由整数组成的数组，从start（含）开始，到stop（不含）结束。整数间距为step。注意，如果stop比start还要小的话，那么就会返回空数组。因此，如果你想得到一个负数组成的数组，请将step设为负值。

```javascript
_.range(10); //[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
_.range(1, 11); //[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
_.range(0, 30, 5); //[0, 5, 10, 15, 20, 25]
_.range(0, -10, -1); //[0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
_.range(0); //[]
```

```javascript
_.range = function(start, stop, step){
    if(stop == null){
        stop = start || 0;
        start = 0;
    }
    step = step || 1;
    var length = Math.max(Math.ceil((stop - start) / step), 0);
    var range = Array(length);

    for(var idx = 0; idx < length; idx++, start += step){
        range[idx] = start;
    }
    return range;
}
```

## 用于函数的一些方法

### _.bind(function, obj, *argument)

```javascript
var func = function(greeting){
	return greeting + ': ' + this.name
}
func = _.bind(func, {name: 'moe'}, 'hi');
func(); //'hi: moe'
```

```javascript
//确定一个函数究竟是作为构造函数来运行，还是作为普通函数来运行
var executeBound = function(sourceFunc, boundFunc, context, callingContext, args){

	if(!(callingContext instanceof boundFunc)){
		return sourceFunc.apply(context, args);
	}
	var self = baseCreate(sourceFunc.prototype);
	var result = sourceFunc.apply(self, args);
	if(_.isObject(result)) return result;
	return self;
}

_.bind = function(func, context){
	if(nativeBind && func.bind === nativeBind){
		return nativeBind.apply(func, slice.call(arguments, 1));
	}

	if(!_.isFunction(func)){
		throw new TypeError('Bind must be called on a function');
	}

	var args = slice.call(arguments, 2);
	var bound = function(){
		return executeBound(func, bound, context, this, args.concat(slice.call(arguments)));
	}
	return bound;
}
```

我们先来看一个简化版的bind的实现，参考《Javascript设计模式与开发实践》一书P32：

```javascript
Function.prototype.bind = Function.prototype.bind || function(){
    var self = this, //保存原函数
        context = [].shift.call(arguments), //需要绑定的this上下文
        args = [].shift.call(arguments); //剩余的参数转成数组
    return function(){ //返回一个新的函数
        return self.apply(context, [].concat.call(args, [].slice.call(arguments))); 
        //执行新的函数的时候，会把之前传入的context当作新函数体内的this
        //并且组合两次分别传入的参数，作为新函数的参数
    }
}
```

以上代码是不是看上去跟_.bind的代码很类似，_.bind的代码中也进行了`var args = slice.call(arguments, 2)`，并且后面也出现了`args.concat(slice.call(arguments))`。两者这一部分的原理是一样的。

举个例子：

```javascript
//举个例子：
var sourceFunc = function(age, sex){
	console.log('name in context: ' + this.name);
	console.log('argument: ' + age);
	console.log('argument: ' + sex);
}
var boundFunc = function(sex){
    return sourceFunc.apply({name: 'moe'}, [12, sex]);
}
boundFunc('male');
//name in context: moe
//argument: 12
//argument: male

/*
上面例子中的boundFunc的callingContext是全局变量，所以callingFunc不是boundFunc的instance，因此走的是if判断里面的return sourceFunc.apply(context, args);
在_.bind的源码中，调用executeBound的时候，最后一个参数是args.concat(slice.call(arguments))，其实就是将所有的参数拼在一起，对应到上面的例子，就是[12, sex]这里，对参数进行了拼接。再说得更细致一些，走到_.bind源码汇总的var args = slice.call(arguments, 2); 这一句的时候，这个arguments是_.bind的arguments，也就是：sourceFunc, {name: 'moe'}, 12。然后将前两个减掉，这时候的args = 12。接下来走到return executeBound(func, bound, context, this, args.concat(slice.call(arguments)))这一句。这里的arguments与上面的arguments不一样。这里的arguments是bound/boundFunc所接收的参数，即'male'。所以，在executeBound里面，用了concat拼接起来了。
当boundFunc是作为普通函数运行，而不是构造函数的时候，执行的其实就是上面这样一个过程
*/
```

当boundFunc是作为构造函数运行时，就不会走if(!(callingContext instanceof boundFunc))里面，走的是下面的这一部分代码：

```javascript
var self = baseCreate(sourceFunc.prototype);
var result = sourceFunc.apply(self, args);
if(_.isObject(result)) return result;
return self;
```

当boundFunc作为构造函数的时候，是要返回一个对象的。这时候仅仅`return sourceFunc.apply(context, args);`是不够的。还要判断sourceFunc.apply之后会返回什么。

构造函数，如果这个函数本身没有返回值，或者返回值为null或非对象的话，返回的就是实例。如果这个函数本身也返回了一个对象，那么就返回该对象。所以，我们在executeBound的源码中看到了这样的判断：

```javascript
if(_.isObject(result)) return result; //如果原本返回的就是对象，那么就返回该对象
return self; //否则就返回实例。self是sourceFunc的实例，继承了它的原型链
```

我们注意到，当boundFunc是`new boundFunc()`这样用的时候，它的`this`就是这个构造函数的实例。因此，这个时候`_.bind`传入的`context`已经没有用处了，传入的只有后面的参数。在`executeBound`源码汇中的if之外，就没有再用到context这个参数。

```javascript
var sourceFunc = function(age, sex){
    console.log('name in context: ' + this.name);
    console.log('argument: ' + age);
    console.log('argument: ' + sex);
}
var boundFunc = sourceFunc.bind({name: 'moe'}, 12);
var result = new boundFunc('male');
//name in context: undefined
//argument: 12
//argument: male
result;
//sourceFunc {} 这个是sourceFunc的实例，因为sourceFunc本身没有返回对象，所以返回了实例
```

```javascript
var sourceFunc = function(age, sex){
    console.log('name in context: ' + this.name);
    console.log('argument: ' + age);
    console.log('argument: ' + sex);
    return {name: this.name, age: age, sex: sex};
}
var boundFunc = sourceFunc.bind({name: 'peter'}, 12);
var result = new boundFunc('male');
//name in context: undefined
//argument: 12
//argument: male
result;
Object {name: undefined, age: 12, sex: "male"} //返回的是对象，并非实例，参数传了进去，但是传进去的context {name: 'peter'}并没有榜上，因为返回的result中的name是undefined
```

以上是通过原生的bind进行的绑定，可以看到，_.bind的polyfill的效果确实与其一致。

借此复习一下`new Constructor`究竟经历了怎样一个过程：

[这篇文章](http://www.cnblogs.com/zichi/p/4392944.html)是很有参考价值的一篇文章.此处参考《学习JavaScript设计模式与开发实践》一书P19-20上对于`new`的模拟。

```javascript
//定义了一个objectFactory的函数，来模拟new
var objectFactory = function(){
	var obj = new Object(), //从Object.prototype上克隆一个空的对象
		Constructor = [].shift.call(arguments); //取得外部传入的构造器，此例是下面的Person
		obj.__proto__ = Constructor.prototype; //指向正确的原型
		var ret = Constructor.apply(obj, arguments); //借用外部传入的构造器给obj设置属性

		return typeof ret ==='object' ? ret : obj; //确保构造器总是会返回一个对象
}

function Person(name){
	this.name = name;
}
Person.prototype.getName = function(){
	return this.name;
}

var a = objectFactory(Person, 'sven');
console.log(a.name); //输入：sven
console.log(a.getName()); //输出：sven
console.log(Object.getPrototypeof(a) === Person.prototype); //输出：true

/*
可见：
var a = objectFactory(A, 'sven');
var a = new A('sven');
产生了同样的效果。
*/
```

比对上述对于`new`的模拟和executeBound的源码，就看得更加清楚了。

### _.bindAll(obj, *methodNames)

该方法会将object下面名为methodNames的一些方法，其context绑在object上。十分适合用来绑定event handler。

```javascript
var buttonView = {
    label: 'underscore',
    onClick = function(){ alert('clicked: ' + this.label); },
    onHover = function(){ console.log('hovering: ' + this.label); }
};
_.bindAll(buttonView, 'onClick', 'onHover');
//当button被点击时，this.label会获得正确的值
//jQuery('#underscore_button').on('click', buttonView.onClick)
```

```javascript
_.bindAll = function(obj){
    var i, length = arguments.length, key;
    if(length <= 1) throw new Error('bindAll must be passed function names');
    for(i = 1; i < length; i++){
        key = arguments[i];
        obj[key] = _.bind(obj[key], obj);
    }
    return obj;
}
```

### _.partial(function, *arguments)

返回的也是函数，与_.bind非常接近，只不过不改变this值。你可以传递`_`作为参数，这代表这个参数不预先填充，在调用的时候再提供参数。

```javascript
var subtract = function(a, b){ return b - a };
sub5 = _.partial(subtract, 5);
sub5(20); //15

//使用 _ 占位符
subFrom20 = _.partial(subtract, _, 20);
subFrom20(5); //15
```

```javascript
_.partial = function(func){
    var boundArgs = slice.call(arguments, 1); //boundArgs是传入到_.partial中需要提前绑定的参数，其中真正的参数和占位符_都在里面
    var bound = function(){
        var position = 0, length = boundArgs.length;
        var args = Array(length);
        for(var i = 0; i < length; i++){
            args[i] == boundArgs[i] === _ ? arguments[position++] : boundArgs[i]; //将参数一个个查看过来，如果是占位符，就用新传入的参数代替；如果不是占位符，就保持原来的值不变
        }
        while(position < arguments.length) args.push(arguments[position++]); //position < arguments.length 说明还有新传入的参数，所以需要继续往里面添加
        return executeBound(func, bound, this, this, args);
    };
    return bound;
}
```

### _.memoize(function, [hashFunction])

该方法可以缓存计算的结果，对于加速耗时较长的计算很有帮助。如果传递了hashFunction之后，这个hasFunction会被用于计算存储结果的hash key。hash Function默认使用function的第一个参数作为key。被memoize的值的缓存可以通过返回的函数的cache属性得到。

```javascript
var fibonacci = _.memoize(function(n){
    return n < 2 ? n : fibonacci(n-1) + fibonacci(n - 2);
})
```

```javascript
_.memoize = function(func, hasher){
    var memoize = function(key){
        var cache = memoize.cache;
        var address = '' + (hasher ? hasher.apply(this, arguments) : key);
        if(!_.has(cache, address)) cache[address] = func.apply(this, arguments);
        return cache[address];
    };
    memoize.cache = {};
    return memoize;
}
```

```javascript
var add = function(a, b){ return a + b; };
add = _.memoize(add); 
add(1, 3); //4
add(1, 5); //4 因为是以第1个参数作为key的，所以如果之前key为1的值存为了4，后面add(1, 5)就不会再进行运算，而是直接会将add.cache中值为1的值4取出来。
add.cache; //{1: 4}
```

```javascript
//要解决上面代码块中出现的问题，就需要用到hashFunction，简单解决一下如下：
var add = function(a, b){ return a + b; };
add = _.memoize(add, function(){ return Math.random(); }); 
add(1, 3); //4
add(1, 5); //6
add.cache; //{0.1903425562450185: 4, 0.07820913718093436: 6}
```

_.memoize的作用，说白了就是，传进去一个函数，返回来一个函数，返回来的这个函数就是memoized函数。这个memoized函数再进行运算的时候，在运算之前会先到它自己的cache里面查，这个运算以前做过吧，做过cache里面就会有值，就把它取出来直接用就好了，如果cache里没有，那么就还是乖乖运算好了。这一方法用于计算阶乘等递归的函数，效果应该挺显著的。

### _.delay(function, wait, *arguments)

类似setTimeout，等待wait毫秒后调用function。如果传递可选的参数arguments，当函数function执行时，arguments会作为参数传入。

```javascript
var log = _.bind(console.log, console);
_.delay(log, 1000, 'logged later');
//'logged later' //1秒钟后显示
```

```javascript
_.delay = function(func, wait){
    var args = slice.call(arguments, 2);
    return setTimeout(function(){
        return func.apply(null, args);
    }, wait);
}
```

### _.defer(function, *arguments)

延迟调用function直到当前调用栈清空为止，类似使用延时为0的setTimeout方法。对于执行开销大的计算和无阻塞UI线程的HTML渲染时候非常有用。如果传递arguments参数，当函数function执行时，arguments会作为参数传入。

至于为什么会用到`setTimeout(f, 0)`，阮一峰的博客写得很好了，直接参考即可。[这一篇](http://javascript.ruanyifeng.com/advanced/timer.html)还有[这一篇](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)

源码：

```javascript
_.defer = _.partial(_.delay, _, 1); //相当于设置setTimeout(f, 1);
```

从上面代码可以看出：`_.defer(f)`相当于执行了`_.delay(f, 1)`，就相当于执行了`setTimeout(function(){ return f.apply(null); }, 1)`(没考虑传递更多参数的情况)。

```javascript
var f1 = function(){ console.log('f1'); };
var f2 = function(){ alert('f2'); };

f2();
f1();
//alert是阻塞式的，用户不点击确定，程序是不会继续向下执行的。所以执行上面两行代码，必须先点击弹出的f2的确认，然后在控制台才会打印出'f1'来。
```

```javascript
同样是上面的f1和f2
_.defer(f2);
f1();
//这个时候，f1会先打印出来，然后f2的弹窗才会出现要求确认。这说明f1作为现有任务立即执行了，而f2添加到下一轮的“任务队列”中，要到下一次Event Loop才执行。
```

### _.throttle(function, wait, [options])

创建并返回被传入的function的节流版本。当重复调用的时候，至少每个wait毫秒调用一次该函数。对于控制一些触发频率比较高的事件很有帮助，例如鼠标移动、mousemove事件、DOM元素动态定位、window对象的resize和scroll事件。

参考[这篇文章](http://www.css88.com/archives/4648)，使用setTimeout和clearTimeout可以实现简单的throttle来优化window resize或scroll事件：

```javascript
var resizeTimer = null;
$(window).on('resize', function(){
	if(resizeTimer){
		clearTimeout(resizeTimer);
	}
	resizeTimer = setTimeout(function(){
		console.log("window resize");
	}, 400);
})
//以上代码其实确保了resize上绑定的事件，至少要间隔400毫秒才会被执行一次。
```

节流原理，利用定时器。当触发一个事件时，先setTimeout让这个事件延迟执行。如果还没到时间又触发了事件，那么就将原来的定时器clear掉，再设置一个新的定时器。

又比如window scroll事件，使用underscore的_.throttle方法就是这样的：

```javascript
//先将document的高度设置高一些，这样可以进行滚动
var f = function(){ console.log('1') };
var v = _.throttle(function(){ console.log('2'); }, 5000);
window.addEventListener('scroll', f);
window.addEventListener('scroll', v);
//滚动页面，查看控制台打印数字的频率，其中'1'的打印是没有收到控制的，而'2'的打印是至少隔5秒才会出现一次
```

默认情况下，throttle会在你第一次调用这个function的时候，尽可能快地执行这个函数。而且，如果在wait期间内，你又再次调用了，不管再次调用几次，到这个wait时间区间过去之后，它又会再尽可能快地执行这个函数。`_.throttle`方法还接收options配置。如果想要禁用第一次首先执行的话，将option设置为`{leading: false}`；如果想禁用最后一次执行的话，传递`{trailing: false}`。

源码：

```javascript
_.throttle = function(func, wait, options){
	var context, args, result;
	var timeout = null;
	var previous = 0;
	if(!options) options = {};
	var later = function(){
		previous = options.leading === false ? 0 : _.now();
		timeout = null;
		//console.log('B');
		result = func.apply(context, args);
		if(!timeout) context = args = null;
	};
	return function(){
		var now = _.now();
		if(!previous && options.leading === false) previous = now;
		var remaining = wait - (now - previous);
		context = this;
		args = arguments;
		if(remaining <= 0 || remaining > wait){
			//由于options.leading === false的时候，previous = now，所以remaining = wait，走不到这个分支里面来，所以只要走到这里面来的，都是没有禁用leading的情况。
			//没有禁用leading的话，就是要尽快执行func，所以如果之前设置了延迟执行的话，都先曲线，然后直接尽快执行func。
			if(timeout){
				clearTimeout(timeout);
				timeout = null;
			}
			previous = now;
			console.log('A');
			result = func.apply(context, args);
			if(!timeout) context = args = null;
		} else if(!timeout && options.trailing != false){
			timeout = setTimeout(later, remaining);
		}
		return result;
	}
}
```

源码解析参考[这篇文章](https://github.com/hanzichi/underscore-analysis/issues/22)和[这篇文章](http://www.alloyteam.com/2012/11/javascript-throttle/)。两者这一部分的原理是一样的。

根据[这篇文章](https://github.com/hanzichi/underscore-analysis/blob/master/underscore-1.8.3.js/underscore-1.8.3-analysis.js)的注释，如果将上面源码中的`console.log`打开，会出现下面的情况：

```javascript
sample 1: _.throttle(function(){}, 1000)
print: A, B, B, B ...

step1: 当第一次触发，没有禁用leading和trailing的时候，previous为0 -> remaining<0 -> previous设置时间戳，（打印A）运行func
step2: 当再次触发，时间还没到的时候，走else if分支，设置定时器 
-> 定时器到时，previous设置时间戳，（打印B）执行func，将定时器清空
-> 定时器没到时，说明remaining也没有<=0，所以第一个分支进不去，第二个分支中有!timeout的判断，所以也进不去，所以这个时候throttled函数什么也执行不了。
step3: 再次触发，由于previous不为0，所以remaining = wait - (now - previous)。如果remaining <= 0 这说明wait这段时间距离上次func的触发已经过去了，于是又会走remaining <= 0的分支执行；如果remaining在0和wait之间，这说明距离上次func触发还没到wait时间，这时候就会走else if分支，设置延迟执行。
依次循环

///////////////////////

sample 2: _.throttle(function(){}, 1000, {leading: false})
print: B, B, B, B ...

step1: 第一次触发，禁用leading的时候，previous被设置为了now，remaining = wait，于是进入不了第一个分支，今儿进入第二个分支，进行定时器设置，延时操作 -> 定时器到时，previous设置为0，（打印B）执行func，清空定时器
step2: 再次触发，
-> 时间还没到时，remaining的时间不符合第一个分支，进不去，同时由于还存在定时器，所以第二个分支也进不去，什么也干不了。
-> 时间到了的时候，由于previous设置为0并且禁用了leading，所以previous又被设置为now，导致remaining = wait，第一个分支进不去，进入第二个分支设置定时器进行延迟执行，因此一直是打印B的。
依次循环

///////////////////////////////

sample 3: _.throttle(function(){}, 1000, {trailing: false})
print: A, A, A, A ...

step1: 第一次触发，trailing被禁用的时候，previous为0，remaining<0，进入第一个分支，（打印A）func立即执行
step2: 再次触发：
-> 时间还没到的时候，remaining的条件不符合，进入不了第一个分支；由于trailling === false 不符合第二个分支的判断，所以第二个分支也进不去，什么也不能干
-> 时间到了的话，进入第一个分支，（打印A）func立即执行
```

在源码中同时看到设置定时器和设置时间戳的方式。一种方式是通过时间戳看是否执行回调。先记下上次执行的时间，然后当函数要再次执行的时候，看看上次的时间和当前时间是否间隔达到要求，达到了就执行，没达到就不执行。这个就是`if(remaining <= 0)`所判断的。另外一种方式是通过定时器。设置了定时器之后，不到点儿的话，如果已有定时器就不能再设定时器。到了点之后，把定时器的Timer清理掉。这就是`else if(!timeout)`所判断的，定时器到点执行的later函数中，包括将之前定时器的timer设置为null。

有了`if(remaining <= 0)`和`else if(!timeout)`两个判断，凡是在wait时间之内，提前要执行的throttled函数，其实是哪个分支也走不进去，什么也干不了。一旦时间到了，要么走第一个分支立即执行，要么定时器触发，执行func。

至于leading和trailing的禁用，这里也很巧妙。其中trailing禁用比较简单，所谓禁用掉trailing就是指延迟要发生的那最后一次不让它执行，于是在源码中，就是直接最后这个延时器不让设置了。

leading禁用就更巧妙了，所谓leading禁用，就是马上要触发的这次不让它执行，这就说明，leading禁用的时候，就用定时器来设置func延迟执行。通过在leading禁用的时候，将previous设置为now，假装刚刚执行过，这次就不用执行了，使得remaining = wait 从而进入不了第一个分支，而只能走入第二个分支。但是这里有一个地方，设置previous = now的判断条件是 `leading === false && !previous`。关键就关键在这个 !previous 上，第一次执行throttled函数，previous为0，leading设置为禁用，当然可以走通这个判断，previous顺利设置为now。但是当定时器执行later函数的时候，为什么在later函数里，当`leading === false`的时候，previous又被设置为0呢？这是因为，如果不设置为0的话，下一次执行throttled函数的时候，`if(leading === false && !previous)`就走不通，于是previous就无法正常设置为now了，这样会导致直接进入第一个分支，立即执行func函数，而我们想要的是leading禁用，所以这样就出现问题了。

`leading === false`和`trailing === false`可以简单理解为“掐头”和“去尾”。

### _.debounce(function, wait, [immediate])

创建一个新的debounced的函数，使得function的执行要等到它最后一次触发之后wait毫秒之后。使用场景例如，要等到input输入告一段落之后再执行某个而行为。例如：渲染Markdown评论的预览、在窗口resize之后重新计算layout等等。

在wait毫秒过去之后，function执行时的arguments是按照最新的传入的参数来的。

将参数immediate设置为true，会在wait的开头而不是结尾触发function，这个时候wait这个参数会被忽略掉。使用场景例如，要防止误双击submit按钮，致使出发了两次提交。

```javascript
//用途举例
var lazyLayout = _.debounce(calculateLayout, 300);
$(window).resize(lazyLayout);
```

_.debounce和_.throttle的区别就是，_.debounce返回的函数，如果一直触发，那么就一直不执行，直到不再触发了，才会执行；_.throttle返回的函数，如果一直触发，那么就至少要间隔wait毫秒才会触发一次。

源码：

```javascript
_.debounce = function(func, wait, immediate){
    var timeout, args, context, timestamp, result;
    var later = function(){
        var last = _.now - timestamp;
        if(last < wait && last > 0){
            timeout = setTimeout(later, wait - last);
        } else {
            timeout = null;
            if(!immediate){
                result = func.apply(context, args);
                if(!timeout) context = args = null;
            }
        }
    };

    return function(){
        cnotext = this;
        args = arguments;
        timestamp = _.now();
        var callNow = immediate && !timeout;
        if(!timeout) timeout = setTimeout(later, wait);
        if(callNow){
            result = func.apply(context, args);
            context = args = null;
        }
        return result;
    }
}
```

当没有设置immediate的时候，debounced的函数执行，走了`if(!timeout) timeout = setTimeout(later, wait);`设置了定时器。如果还没到时间，debounced函数又要执行，这时候因为有`if(!timeout)`的限制，无法再设置定时器了。但是时间戳是更新了的（所以说，如果一直触发的话，时间戳会一直更新）。如果时间到了，那么later就开始执行了，首先将当前时间与上一次设置的时间戳进行比较。如果时间没到，就继续设置定时器，如果时间到了，就立即执行。注意later中进行了判断，如果immediate设置为true的话，是不会执行func的，因为immediate设置为true，是不应该通过定时器延迟执行的。

接下来看设置immediate为true的时候，debounced的函数第一次执行，设置了时间戳，callNow为真，设置了定时器。然后进入`if(callNow)`的分支，直接执行了func。再次触发debounced函数，如果wait时间还没过去，这时候是有定时器的，所以无法再次设置定时器，同时因为!timeout为false，所以callNow也为false，所以func也不会直接执行。如果再次触发debounced函数，wait时间已经过去了，之前设置的定时器会执行later，由于`if(!immediate)`的判断，func不会直接执行，但是定时器却清掉了。所以这个时候callNow的判断又为真了，又可以直接执行func了。因此，这样就实现了，设置immediate为true的话，在wait开始的头上直接执行func，但是wait期间，还是不会执行func。除非wait毫秒过去，才能再度触发func。

### _.wrap(function wrapper)

将第一个传入的function包在wrapper函数里面，使得function成为wrapper的第一个参数。这样允许你控制在function之前、之后做些什么事情，或者以什么条件来执行原始的function。

```javascript
//实现非常简单
_.wrap = function(func, wrapper){
	return _.partial(wrapper, func);
}
```

应用：

```javascript
var hello = function(name){ return "hello: " + name; };
hello = _.wrap(hello, function(func){
	return "before, " + func("moe") + ", after";
});
hello();
//'before, hello: moe, after'
```

### _.negate(predicate)

返回与predicate函数相反的新版本

```javascript
var isFalsy = _.negate(Boolean);
_.find([-2, -1, 0, 1, 2], isFalsy);
//0
```

### _.compose(*functions)

直白一些来说，compose的作用就是用f(), g(), h()这样几个函数，合成一个f(g(h()))出来.即前一个函数的参数是后一个函数的返回值。

```javascript
var greet = function(name){ return "hi: " + name; };
var exclaim = function(statement){return statement.toUpperCase() + "!"; };
var welcome = _.compose(greet, exclaim);
welcome('moe');
//'hi: MOE!'
```

```javascript
_.compose = function(){
	var args = arguments;
	var start = args.length - 1;
	return function(){
		var i = start;
		var result = arguments[start].apply(this, arguments);
		while(i--) result = args[i].call(this, result);
		return result;
	}
}
```

### _.after(count, function)

创建一个function的新版本，使得这个function只能在count次call之后才执行。用途是将异步的一些相应集合在一起，确保所有异步的操作都结束，再继续向后执行。

```javascript
var renderNotes = _.after(notes.length, render);
_.each(notes, function(note){
	note.asyncSave({success: renderNotes});
})
//在所有的notes都保存之后，render再执行。也就是说，renderNotes被触发了notes.length次之后，render才执行。
```

```javascript
_.after = function(times, func){
	return function(){
		if(--times < 1){
			return func.apply(this.arguments);
		}
	}
}
```

### _.before(count, function)

创建function的一个新版本，使它被触发不能超过count次。当达到count次后，最后一次function执行的结果会被memoized并返回。

```javascript
var monthlyMeeting = _.before(3, askForRaise);
monthlyMeeting();
monthlyMeeting();
monthlyMeeting();
//如果后续再触发的话，结果也不会变了，跟第二次的结果是一样的。_.before的第一个参数，是小于它，不包含等于它
```

```javascript
_.before = function(times, func){
	var memo;
	return function(){
		if(--times > 0){
			memo = func.applyIthis.arguments;
		}
		if(times <= 1) func = null;
		return memo;
	}
}
```

### _.once(function)

传入的函数只能被触发一次。再次触发没有效果。返回的值与首次执行返回的值相同。在初始化函数的时候很有用，这样就不需要再使用boolean flag然后进行检查是否可以再执行了。

```javascript
var initialize = _.once(createApplication);
initialize();
initialize();
//其实只初始化了一次，第二次是没有效果的
```

```javascript
_.once = _.partial(_.before, 2);
```

## 用于对象的一些方法

`_.keys`和`_.allKeys`前面都已经写过了。

### _.values(object)

返回对象所有own的属性的值

```javascript
_.values({one: 1, two: 2, three: 3});
//[1, 2, 3]
```

```javascript
_.values = function(obj){
	var keys = _.keys(obj);
	var length = keys.length;
	var values = Array(length);
	for(var i = 0; i < length; i++){
		values[i] = obj[keys[i]];
	}
	return values;
}
```

### _.mapObject(object, iteratee, [context])

类似于数组的map方法，只不过是针对对象的，将对象的值改变。

```javascript
_.mapObject({start: 5, end: 12}, function(val, key){
	return val + 5;
});
//{start: 10, end: 17}
```

```javascript
_.mapObject = function(obj, iteratee, context){
	iteratee = cb(iteratee, context);
	var keys = _.keys(obj),
		length = keys.length,
		results = {},
		currentKey;
	for(var index = 0; index < length; index++){
		currentKey = keys[index];
		results[currentKey] = iteratee(obj[currentKey], currentKey, obj);
	}
	return results;
}
```

### _.pairs(object)

将对象转为键值对的数组

```javascript
_.pairs({one: 1, two: 2, three: 3});
//[["one", 1], ["two", 2], ["three", 3]]
```

```javascript
_.pairs = function(obj){
	var keys = _.keys(obj);
	var length = keys.length;
	var pairs = Array(length);
	for(var i = 0; i < length; i++){
		pairs[i] = [keys[i], obj[keys[i]]];
	}
	return pairs;
}
```

### _.invert(object)

返回一个对象的拷贝，只不过原来的键变成了值，原来的值变成了键。为了要让这个函数奏效，是有要求的。那就是你所有的对象的值必须是唯一的，并且是可序列化的字符串。

```javascript
_.invert({Moe: "Moses", Larry: "Louis", Curly: "Jerome"});
//{Moses: "Moe", Louis: "Larry", Jerome: "Curly"};
```

```javascript
_.invert = function(obj){
	var result = {};
	var keys = _.keys(obj);
	for(var i = 0, length = keys.length; i < length; i++){
		result[obj[keys[i]]] = keys[ik];
	}
	return result;
}
```

### _.functions(object)

返回一个序列化的列表，将一个对象中所有方法的名字都返回回来——也就是说，返回对象中所有函数属性的名称。

```javascript
_.functions(_);
//["all", "any", "bind", "bindAll", "clone", "compact", "compose" ...
```

```javascript
_.functions = _.methods = function(obj){
	var names = [];
	for(var key in obj){
		if(_.isFunction(obj[key])) names.push(key);
	}
	return names.sort();
}
```

接下来的`_.extend`和`_.extendOwn`都已经在前面写过了，这里不重复了。

### _.findKey(object, predicate, [context])

会返回通过了predicate函数的属性的key，如果没有的话，返回`undefined`

```javascript
_.findKey = function(obj, predicate, context){
	predicate = cb(predicate, context);
	var keys = _.keys(obj), key;
	for(var i = 0; length = keys.length; i < length; i++){
		key = keys[i];
		if(predicate(obj[key], key, obj)) return key;
	}
}
```

### _.pick(object, *keys)

返回object的一个副本，但是是经过过滤的，留下的都是keys白名单中有的（或者是由有效的key组成的数组）。或者也可以提供predicate函数，过滤出哪些key是要被捡出来的。

```javascript
_.pick({name: 'moe', age: 50, userid: 'moe1'}, 'name', 'age');
//{name: 'moe', age: 50}
_.pick({name: 'moe', age: 50, userid: 'moe1'}, function(value, key, object) {
  return _.isNumber(value);
});
//{age: 50}
```

```javascript
_.pick = function(object, oiteratee, context){
	var result = {}, obj = object, iteratee, keys;
	if(obj == null) return result;
	if(_.isFunction(oiteratee)){
		keys = _.allKeys(obj);
		iteratee = optimizeCb(oiteratee, context);
	} else {
		keys = flatten(arguments, false, false, 1);
		//内部的flatten是这样用的：flatten = function(input, shallow, strict, startIndex)，具体解析在前面写过。
		iteratee = function(value, key, obj){ return key in obj; };
		obj = Object(obj);
	}
	for(var i = 0, length = keys.length; i < length; i++){
		var key = keys[i];
		var value = obj[key];
		if(iteratee(value, key, obj)) result[key] = value;
	}
	return result;
}
```

### _.omit(object, *keys)

返回object的一个copy，但是将列入黑名单的Keys（或者keys的数组）过滤掉。第二个参数还可以是一个predicate函数，用于判断哪些key要过滤掉。

```javascript
_.omit({name: 'moe', age: 50, userid: 'moe1'}, 'userid');
// {name: 'moe', age: 50}
_.omit({name: 'moe', age: 50, userid: 'moe1'}, function(value, key, object) {
  return _.isNumber(value);
});
// {name: 'moe', userid: 'moe1'}
```

```javascript
_.omit = function(obj, iteratee, context){
	if(_.isFunction(iteratee)){
		iteratee = _.negate(iteratee);
	} else {
		var keys = _.map(flatten(arguments, false, false, 1), String);
		iteratee = function(value, key){
			return !_.contains(keys, key);
		}
	}
	return _.pick(obj, iteratee, context);
}
```

### _.defaults(object, *defaults)

用defaults对象填充object中的undefined属性，并返回这个object。一旦这个属性被填充，再使用这个defaults方法将不会有任何效果。

```javascript
var iceCream = {flavor: "chocolate"};
_.defaults(iceCream, {flavor: "vanilla", sprinkles: "lots"});
// {flavor: "chocolate", sprinkles: "lots"}
```

```javascript
_.defaults = createAssigner(_.allkeys, true);
```

### _.create(prototype, props)

用给定的prototype创建一个新对象，可选择地将props作为它自己拥有的(own)属性。基本上就是跟Object.create一样，但是不用搞那些麻烦的属性描述符了。

```javascript
var moe = _.create(Stooge.prototype, {name: "Moe"});
```

```javascript
_.create = function(prototype, props){
	var result = baseCreate(prototype);
	if(props) _.extendOwn(result, props);
	return result;
}
```

### _.clone(object)

为提供的“纯”对象创建一个浅拷贝。任何嵌套的对象或数组都通过引用拷贝，不会复制。

纯对象就是直接通过声明变量创建的对象，而不是用new构造函数的形式创建的对象。

```javascript
_.clone = function(obj){
	if(!_.isObject(obj)) return obj;
	return _.isArray(obj) ? obj.slice() : _.extend({}, obj);
}
```

### _.tap(object, interceptor)

在object上触发interceptor函数，然后返回object。这个方法的主要目的就是“利用”chain方法，使得在链式调用中可以在这个链当中的结果上进行一些操作。

interceptor的中文意思是“拦截机、拦截器”

```javascript
_.chain([1, 2, 3, 200])
	.filter(function(num){ return num % 2 == 0; })
	.tap(alert)
	.map(function(num){ return num * num; })
	.value();
//弹出[2, 200]
//[4, 40000]
```

```javascript
_.tap = function(obj, interceptor){
	interceptor(obj);
	return obj;
}
```

### _.isMatch(object, properties)

这个方法能告诉你object中是否包含properties中的那些键值对

```javascript
var stooge = {name: 'moe', age: 32};
_.isMatch(stooge, {age: 32});
//true
```

```javascript
_.isMatch = function(object, attrs){
	var keys = _.keys(attrs), length = keys.length;
	if(object == null) return !length;
	var obj = Object(object);
	for(var i = 0; i < length; i++){
		var key = keys[i];
		if(attrs[key] !== obj[key] || !(key in obj)) return false;
	}
	return true;
}
```

### _.isEqual(object, other)

对两个对象进行深层比较，来绝对这两个对象是否是相等的。

```javascript
var stooge = {name: 'moe', luckyNumbers: [13, 27, 34]};
var clone  = {name: 'moe', luckyNumbers: [13, 27, 34]};
stooge == clone;
// false
_.isEqual(stooge, clone);
// true
```

```javascript
_.isEqual = function(a, b){
	return eq(a, b);
}
```

```javascript
// 内部函数eq
var eq = function(a, b, aStack, bStack){
	//0,-0,+0用三等号比较，返回的都是true，但是它们不是完全相等的，下面这一句就是判断这个的。
	if(a === b) return a !== 0 || 1 / a === 1 / b;
	//由于null == undefined，所以如果其中一项是null的话，那么必须要用三个等号进行比较
	if(a == null || b == null) return a === b;

	if(a instanceof _) a = a._wrapped;
	if(b instenceof _) b = b._wrapped;
	var className = toString.call(a);
	if(className !== toString.call(b)) return false;
	switch(className){
		//字符串、数字、正则、日期和布尔类型，都是用值来比较的
		case '[object RegExp]':
		//正则表达式也是用 '' + a === '' + b这一句来判断的。也就是说，正则是强制转化为字符串类型进行比较（注：'' + /a/i === '/a/i')
		case '[object String]':
		//基本类型和对应的包装类型是相等的，也就是说'5'等于new String('5')；/a/与new RegExp('a')也是相等的。
		//通过+操作符，将包装类型转成了基本类型
			return '' + a === '' + b;
		case '[object Number]':
			//下面这一句是判断NaN的，两个NaN在underscore中会判断为相等的；Object(NaN)与NaN也是相等的
			if(+a !== +a) return +b !== +b;
			//下面是排除0的干扰，通过+a可以将Number()保障类型转为基本类型。
			return +a === 0 ? 1 / +a === 1 / b : +a === +b;
		//一下对Date和Boolean的判断，使用+操作符可以转为基本类型的数字。
		//var a = new Boolean(true); 
		//+a; => 1
		//var b = new Boolean(false);
		//+b; => 0
		//var c = new Date();
		//+c; => 1491106648303

		//var x = new Date(NaN); var y = new Date(NaN);
		//+x === +y; => false
		case '[object Date]':
		case '[object Boolean]':
			return +a === +b;
	}

	var areArrays = className === '[object Array]';
	if(!areArrays){//如果a不是数组
		//如果a不是object或b不是object，就返回false
		if(typeof a != 'object' || typeof b != 'object') return false;
		//到这里，a和b都应该是object了
		var aCtor = a.constructor, bCtor = b.constructor;
		//如果a和b的构造函数不同，两个对象不一定不相等。比如a、b在不同的iframe中。
		//什么时候aCtor instanceof aCtor？举个例子：
		//var aCtor = function(){return{name: 'abc', age:123};};
		//var aP1 = new aCtor;
		//aP1.constructor instanceof aP1.constructor; => true

		//如果aCtor !== bCtor那么也不说明a、b两个对象不相等。有可能在两个frame里面。接下来判断，a和b的constructor是否都为函数，并且这两个构造函数都是直接返回了一个对象作为实例，就像上面注释举得例子这样。如果有一个其实是返回的指向自身内部的this，那么两者也就不全等了，因为两个的原型链是不一样的。接下来还要判断，是否两者都有constructor这个属性，要是万一一个是通过构造函数返回来的，另外一个不是通过构造函数，而直接是变量声明得来的，那么两者也不算做全等。

		//此处理解不够细致，需要再研究
		if(aCtor !== bCtor && !(_.isFunction(aCtor) && aCtor instanceof aCtor && _.isFunction(bCtor) && bCtor instanceof bCtor) && ('constructor' in a && 'constructor' in b)){
			return false;
		}
	}

	//创建了一个栈，用来放被遍历的数组。第一次调用eq()，没有传入这两个参数，之后递归的话，这两个参数就有了。
	aStack = aStack || [];
	bStack = bStack || [];
	var length = aStack.length;
	while(length--){
		if(aStack[length] === a) return bStack[length] === b;
	}
	aStack.push(a);
	bStack.push(b);

	//举个例子，在这个地方将aStack和bStack打印出来：
	//用_.isEqual([1, [2, [3, [4, 5]]]], [1, [2, [3, [4, 5]]]])举例
	/*
	aStack: [[1,[2,[3,[4,5]]]]] ; bStack: [[1,[2,[3,[4,5]]]]]
	aStack: [[1,[2,[3,[4,5]]]],[2,[3,[4,5]]]]; bStack: [[1,[2,[3,[4,5]]]],[2,[3,[4,5]]]];
	aStack: [[1,[2,[3,[4,5]]]],[2,[3,[4,5]]], [3,[4,5]]]; ; bStack: [[1,[2,[3,[4,5]]]],[2,[3,[4,5]]], [3,[4,5]]];
	aStack: [[1,[2,[3,[4,5]]]],[2,[3,[4,5]]], [3,[4,5]],[4, 5]]; ; bStack: [[1,[2,[3,[4,5]]]],[2,[3,[4,5]]], [3,[4,5]],[4, 5]];
	=>true
	最后一步，展开成那样，最后的[4, 5]遍历，判断出来相等，然后依次返回来，[4, 5]相等了，那么[3, [4, 5]]也相等，依次向前推（一层一层的true返回回来）。所以当嵌套数组拆解成最后一个那种的stack之后，一下子所有问题就解决了
	*/

	if(areArrays){ 
		//数组比较走这个分支
		length = a.length;
		if(length !== b.length) return false;
		while(length--){
			if(!eq(a[length], b[length], aStack, bStack)) return false;
		}
	} else {
		//对象比较走这个分支
		var keys = _.keys(a), key;
		length = keys.length;

		if(_.keys(b).length !== length) return false;
		while(length--){
			key = keys[length];
			if(!(_.has(b, key) && eq(a[key], b[key], aStack, bStack))) return false;
		}
	}

	//与aStack.push相对应
	aStack.pop();
	bStack.pop();
	return true;
}
```

### _.isEmpty(object)

判断对象中是不是有own properties。对数组来说，length为0就是空的。

```javascript
_.isEmpty = function(obj){
	if(obj == null) return true;
	if(isArrayLike(obj) && (_.isArray(obj) || _.isString(obj) || _.isArguments(obj))) return obj.length === 0;
	return _.keys(obj).length === 0;
}
```

### _.isElement(object)

如果object是个dom元素，就返回true

```javascript
_.isEmpty(jQuery('body')[0]);
//true
```

```javascript
_.isElement = function(obj){
	return !!(obj && obj.nodeType === 1);
}
```

### _.isArray(object)

```javascript
_.isArray = nativeIsArray || function(obj){
	return toString.call(obj) === '[object Array]';
}
```

### _.isObject(object)

```javascript
_.isObject = function(obj){
	var type = typeof obj;
	return type === 'function' || type === 'object' && !!obj;
}
```

//注意看上面源码，如果传入的是null，经过_.isObject来判断，会返回false

```javascript
typeof null
//'object'
_.isObject(null);
//false
```

### _.isArguments, _.isFunction, _.isString, _.isNumber, _.isDate, _.isRegExp, _.isError

```javascript
_.each(['Arguments', 'Function', 'String', 'Number', 'Date', 'RegExp', 'Error'], function(name){
	_['is' + name] = function(obj){
		return toString.call(obj) === '[object ' + name + ']';
	}
})
```

```javascript
//兼容IE9以下浏览器，在IE9下，对arguments调用Object.prototype.toString.call，返回[object Object]，并非我们想要的。这里用了判断是否存在arguments.callee来进行兼容
if(!_.isArguments(arguments)){
	_.isArguments = function(obj){
		return _.has(obj, 'callee');
	}
}
```

☆☆☆ 下面的兼容没看明白：参考[链接](https://github.com/jashkenas/underscore/issues/1621)

```javascript
//以下对_.isFunction进行了优化，这里没有看明白为什么
if(typeof /./ != 'function' && typeof Int8Array != 'object'){
	_.isFunction = function(obj){
		return typeof obj == 'function' || false;
	}
}
```

### _.isFinite(object)

```javascript
_.isFinite = function(obj){
	return isFinite(obj) && !isNaN(parseFloat(obj));
}
//js自带的isFinite返回false会有两种情况，一种是正负Infinity，第二种就是NaN
```

### _.isNaN(object)

```javascript
_.isNaN = function(obj){
	return _.isNumber(obj) && obj != +obj;
	//NaN是唯一不等于自己的数字类型
}
```

### _.isBoolean(object)

```javascript
_.isBoolean = function(obj){
	return obj === true || obj === false || toString.call(obj) === '[object Boolean]';
}
```

### _.isNull(object)

```javascript
_.isNull = function(obj){
	return obj === null;
}
```

### _.isUndefined(object)

```javascript
_.isUndefined = function(obj){
	return obj === void 0;
}
```

### _.has(object, key)

如果object自己拥有这个key属性，就返回true

```javascript
_.has = function(obj, key){
	return obj != null && hasOwnProperty.call(obj, key);
}
```

## 其他一些实用方法

### _.noConflict()

该方法将`_`归还给之前的所有者，然后返回一个对象作为Underscore对象的引用。

```javascript
var underscore = _.noConflict();
```

```javascript
_.noConflict = function(){
	root._ = previousUnderscore;
	return this;
}
```

### _.identity(value)

value是什么，就原样返回去

```javascript
_.identity = function(value){
	return value;
}
```

### _.constant(value)

创建了一个函数这个函数返回与value同样的值

```javascript
_.constant = function(value){
	return function(){
		return value;
	}
}
```

### _.noop()

不管传了什么参数进去，都返回undefined。这个方法的用处就是在一些可以选择性的传入回掉函数的时候，可以用这个来占位。

```javascript
_.noop = function(){};
```

### _.property(key)

```javascript
var stooge = {name: 'moe'};
'moe' === _.property('name')(stooge);
//true
```

```javascript
var property = function(key){
	return function(obj){
		return obj == null ? void 0 : obj[key];
	}
}

_.property = property
```

### _.propertyOf(object)

就是_.property反过来。接收一个object作为参数，返回一个函数，那个函数能够返回某个属性的值。

```javascript
var stooge = {name: 'moe'};
_.propertyOf(stooge)('name');
//'moe'
```

```javascript
_.propertyOf = function(obj){
	return obj == null ? function(){} : function(key){
		return obj[key];
	}
}
```

### _.matcher(attrs)

返回一个predicate函数，这个函数会告诉你：如果你传给它一个对象，它是否包含所有attrs中的键值对。

```javascript
var ready = _.matcher({selected: true, visible: true});
var readyToGoList = _.filter(list, ready);
```

```javascript
_.matcher = _.matches = function(attrs){
	attrs = _.extendOwn({}, attrs);
	return function(obj){
		return _.isMatch(obj, attrs);
	}
}
```

### _.times(n, iteratee, [context])

触发给定的iteratee函数n次。每一次触发iteratee都有一个index参数。产生的是返回的值所构成的一个数组。

```javascript
_.times = function(n, iteratee, context){
	var accum = Array(Math.max(0, n));
	iteratee = optimizeCb(iteratee, context, 1);
	for(var i = 0; i < n; i++) accum[i] = iteratee(i);
	return accum;
}
```

```javascript
_.times(3, function(i){ return i*i; })
//[0, 1, 4]
```

### _.random(min, max)

返回min和max之间的随机的一个整数。如果你只传入了一个数值，那么就会返回0和那个数值之间的一个随机整数。

```javascript
_.random = function(min, max){
	if(max == null){
		max = min;
		min = 0;
	}
	return min + Math.floor(Math.random() * (max - min + 1));
}
```

### _.now()README.md

返回表示当前时间戳的整数。

```javascript
_.now = Date.now || function(){
	return new Date().getTime();
}
```

### _.escape(string), _.unescape(string)

`_.escape`将一串要插入HTML中的字符串，该转义的转义出来。

```javascript
_.escape('Curly, Larry & Moe');
//'Curly, Larry &amp; Moe'
```

`_.unescape`正好是`_.escape`反过来。

```javascript
_.unescape('Curly, Larry &amp; Moe');
//'Curly, Larry & Moe'
```

源码：

```javascript
var escapeMap = {
	'&': '&amp;',
	'<': '&lt;',
	'>': '&gt;',
	'"': '&quote;',
	"'": '&#x27;',
	'`': '&#x60;'
};
var unescapeMap = _.invert(escapeMap);
var createEscaper = function(map){
	var escaper = function(match){
		return map[match];
	}

	var source = '(?:' + _.keys(map).join('|') + ')';
	//(?:x) 匹配'x'但不记住匹配项。这叫做非捕获括号，使你能够定义为正则表达式运算符一起使用的子表达式。
	var testRegexp = RegExp(source);
	var replaceRegexp = RegExp(source, 'g');
	return function(string){
		string = string == null ? '' : '' + string;
		return testRegexp.test(string) ? string.replace(replaceRegexp, escaper) : string;
	}
}

_.escape = createEscaper(escapeMap);
_.unescape = createEscaper(unescapeMap);
```

### _.result(object, property, [defaultValue])

如果这里传入的property是个函数，那么就以object为context来运行它，否则就返回object以property为键名的值。入股哟提供了defaultValue，并且property在object中并不存在或者是undefined，那么就返回default。如果defaultValue是函数的话，跟property的处理方式是一样的。

```javascript
var object = {cheese: 'crumpets', stuff: function(){ return 'nonsense'; }};
_.result(object, 'cheese');
// "crumpets"
_.result(object, 'stuff');
// "nonsense"
_.result(object, 'meat', 'ham');
// "ham"
```

```javascript
_.result = function(object, property, fallback){
	var value = object == null ? void 0 : object[property];
	if(value === void 0){
		value = fallback;
	}
	return _.isFunction(value) ? value.call(object) : value;
}
```

### _.uniqueId([prefix])

为客户端的model或dom元素产生一个全局唯一的id。如果传入了prefix，那么id一开始就会有这个作为前缀。

```javascript
_.uniqueId('contact_');
//'contact_104'
```

```javascript
var idCounter = 0;
_.uniqueId = function(prefix){
	var id = ++idCounter + '';
	return prefix ? prefix + id : id;
}
```

### _.template(templateString, [settings])

将javascript模板编译成函数，这些函数求值之后可用于页面渲染。该方法对于从JSON数据源渲染HTML比较复杂的部分很有用。模板函数可以进行值的插入（使用`<%= ... %>`），也可以执行javascript代码（使用`<% ... %>`)。如果你希望插入一个值，但是还要给HTML转义，那么就使用`<%- ... %>`。当你在给模板函数求值的时候，传入一个数据对象，这个对象的属性与模板的自由变量相对应。`settings`参数应该是一个哈希表，里面包含的任何需要被重写的_.templateSettings。

```javascript
var compiled = _.template("hello: <%= name %>");
compiled({name: 'moe'});
//"hello: moe"

var template = _.template("<b><%- value %></b>");
template({value: '<script>'});
//"<b>&lt;script&gt;</b>"
```

在javascript代码中，你也可以使用print。这样做有时候比用`<%= ... %>`更方便。

```javascript
var compiled = _.template("<% print('Hello ' + epithet); %>");
compiled({epithet: "stooge"});
//"Hello stooge"
```

`<% ... %>`这种模板风格被称为ERB风格的分隔符。如果你不喜欢，也可以换成其他分隔符。定义一个叫`interpolate`的属性，这个属性是一个正则表达式，它会匹配哪些东西是要原样插入进模板的。有一个`escape`属性，也是正则表达式，匹配的是经过HTML转义之后，应当插入的表达式；还有一个`evaluate`正则表达式，用于匹配应该需要被求值的，而不是插入结果字符串的表达式。你可以定义或者略去这三者的任何一个或几个。例如，如果想要用小胡子风格的模板，那么可以这样定义：

```javascript
_.templateSettings = {
	interpolate: /\{\{(.+?)\}\}/g
}

var template = _.template("Hello {{ name }}!");
template({name: "Mustache"});
//"Hello Mustache!"
```

默认情况下，模板通过`with`将你传入的数据放在局部作用域中。但是，你可以在`variable`设置中专门给指定一个变量名。这样可以显著提升渲染模板的速度。

```javascript
_.template("Using 'with': <%= data.answer %>", {variable: 'data'})({answer: 'no'});
//"Using 'with': no"
```

预编译模板对于调试你无法重现的错误非常有帮助。这是因为预编译的模板可以提供行号和堆栈追踪，这是你在客户端编译模板的时候不可能做到的。在编译好的模板函数上，有一个`source`属性可以提供简单的预编译功能

```javascript
<script>
	JST.project = <%= _.template(jstText).source %>;
</script>
```

举个例子：

```javascript
compiled = _.template("interpolate: <%= name %>; evaluate: <% 1 + 2 %>");

compiled.source;
/*
"function(obj){
var __t,__p='',__j=Array.prototype.join,print=function(){__p+=__j.call(arguments,'');};
with(obj||{}){
__p+='interpolate: '+
((__t=( name ))==null?'':__t)+
'; evaluate: ';
 1 + 2 
__p+='';
}
return __p;
}"
*/
```

源码：

```javascript
_.templateSettings = {
	evaluate: /<%([\s\S]+?)%>/g,
	interpolate: /<%=([\s\S]+?)%>/g,
	escape: /<%-([\s\S]+?)%>/g
};

var noMatch = /(.)^/;

var escapes = {
	"'": "'",
	'\\': '\\',
	'\r': 'r',
	'\n': 'n',
	'\u2028': 'u2028',
	'\u2029': 'u2029'
} 
var escaper = /\\|'|\r|\n|\u2028|\u2029/g;
var escapeChar = function(match){
	return '\\' + escapes[match];
}

_.template = function(text, settings, oldSettings){
	if(!settings && oldSettings) settings = oldSettings;
	settings = _.defaults({}, settings, _.templateSettings);

	var matcher = RegExp([
		(settings.escape || noMatch).source,
		(settings.interpolate || noMatch).source,
		(settings.evaluate || noMatch).source
	].join('|') + '|$', 'g');
	//regExp.source 这个正则表达式的source属性返回的是正则对象中的文本。但是不包含两个斜线以及任何g这样的flag。
	//如果没有改过settings，那么最终的matcher就会变成：
	///<%-([\s\S]+?)%>|<%=([\s\S]+?)%>|<%([\s\S]+?)%>|$/g

	var index = 0;
	var source = "__p+='";
	text.replace(matcher, function(match, escape, interpolate, evaluate, offset){
		source += text.slice(index, offset).replace(escaper, escapeChar);
		index = offset + match.length;

		if(escape){
			source += "'+\n((__t=(" + escape + "))==null?'':_.escape(__t))+\n'";
		} else if (interpolate){
			source += "'+\n((__t=(" + interpolate + "))==null?'':__t)+\n'";
		} else if(evaluate){
			source += "';\n" + evaluate + "\n__p+='";
		}
		return match;
	});
	source += "';\n";

	if(!settings.variable) source = 'with(obj||{}){\n' + source + '}\n';
	
	//下面写的是print功能
	source = "var __t,__p='',__j=Array.prototype.join," + "print=function(){__p+=__j.call(arguments,'');};\n" + source + 'return __p;\n';

	try{
		//obj为传入的数据，传入_是为了函数内部使用用到了_.escape函数
		var render = new Function(settings.variable || 'obj', '_', source);
	} catch(e){
		e.source = source;
		throw e;
	}

	var template = function(data){
		return render.call(this, data, _);
	}

	var argument = settings.variable || 'obj';
	template.source = 'function(' + argument + '){\n' + source + '}';

	return template;
}
```

### _.chain(obj), _.chain(obj).value()

返回被包装的（wrapped）那个对象。然后再在这个对象上执行方法，又会返回保障的函数，直到value方法执行。

```javascript
var stooges = [{name: 'curly', age: 25}, {name: 'moe', age: 21}, {name: 'larry', age: 23}];
var youngest = _.chain(stooges)
				.sortBy(function(stooge){ return stooge.age; })
				.map(function(stooge){ return stooge.name + ' is ' + stooge.age; })
				.first()
				.value();
//"moe is 21"
```

`_.chain(obj).value()`可以将包装对象的值抽出来

```javascript
_.chain([1, 2, 3]).reverse().value();
//[3, 2, 1]
```

```javascript
var lyrics = [
  {line: 1, words: "I'm a lumberjack and I'm okay"},
  {line: 2, words: "I sleep all night and I work all day"},
  {line: 3, words: "He's a lumberjack and he's okay"},
  {line: 4, words: "He sleeps all night and he works all day"}
];

_.chain(lyrics)
  .map(function(line) { return line.words.split(' '); })
  .flatten()
  .reduce(function(counts, word) {
    counts[word] = (counts[word] || 0) + 1;
    return counts;
  }, {})
  .value();
//Object {I'm: 2, a: 2, lumberjack: 2, and: 4, okay: 2…}
```

从上例中可以看到，数组原型方法也支持underscore的链式操作。具体原因，可以看下面关于OOP的源码。

源码：

```javascript
_.chain = function(obj){
	var instance = _(obj);
	instance._chain = true; //属性_chain为true就是可以继续链式操作
	return instance;
}
//使用_.chain之后，之所以可以这样一直调用下去，要看后面的OOP的相关代码
```

## 有关OOP的一些代码

```javascript
var result = function(instance, obj){
	return instance._chain ? _(obj).chain() : obj;
}
//辅助函数，用于持续的链式操作
```

### _.mixin(object)

允许你扩展Underscore。传入哈希值{name: function}这样的定义就可以让你将自己的方法添加到Underscore对象中以及OOP包装对象中。

```javascript
_.mixin({
	capitalize: function(string){
		return string.charAt(0).toUpperCase() + string.substring(1).toLowerCase();
	}
});

_("fabio").capitalize();
//"Fabio"
_.capitalize("fabio");
//"Fabio"
```

源码：

```javascript
_.mixin = function(obj){
	//_.functions方法返回的是对象上的所有方法的名称组成的数组
	_.each(_.functions(obj), function(name){
		var func = _[name] = obj[name]; //添加到underscore对象上了
		_.prototype[name] = function(){ //添加到了_的实例的原型上去了，那么每个原型都具备了这些方法
			var args = [this._wrapped]; 
			//this指的就是包装对象，this._wrapped就是原本的纯对象
			push.apply(args, arguments); 
			//arguments就是除了第一个obj以外的其他所有的参数，现在通过push，将调用underscore方法的所有参数都凑齐了。
			return result(this, func.apply(_, args)); //注意，一旦链式操作开始，那么每次链式操作返回的都是经过这个result函数的结果，(可以参考一下result函数的源码)也就是返回的是_(obj).chain()这个包装对象。要得到最终值的话，需要用.value()将包装在里面的那个对象抽出来。 
		}
	})
}

_.mixin(_); //将_上的所有方法都添加到underscore对象上了。
```

同时，underscore还将所有Array.prototype上有的方法也都加到包装对象上了。但是要注意，这些方法没有加到underscore对象上，所以，例如你调用`_.pop([1, 2, 3])`是不行的，`_`上没有这个方法。但是用链式调用是可以的。例如：

```javascript
_.chain([1, 2, 3]).pop().value();
//[1, 2]
```

源码：

```javascript
//将Array.prototype上的所有的mutator方法加到包装对象上
_.each(['pop', 'push', 'reverse', 'shift', 'sort', 'splice', 'unshift'], function(name){
	var method = ArrayProto[name];
	_.prototype[name] = function(){
		var obj = this.wrapped;
		method.apply(obj, arguments);
		//下面这一句，参考这个链接：http://stackoverflow.com/questions/24725560/javascript-why-need-to-delete-the-0-index-of-an-array
		if((name === 'shift' || name === 'splice') && obj.length === 0) delete obj[0];
		return result(this, obj);
	}
})
```

源码：

```javascript
//将所有Array上的accessor的函数加到包装对象上
_.each(['concat', 'join', 'slice'], function(name){
	var method = ArrayProto[name];
	_.prototype[name] = function(){
		return result(this, method.apply(this._wrapped, arguments));
	}
})
```

抽取包装对象中所包装的对象：value方法

```javascript
_.prototype.value = function(){
	return this._wrapped;
}
```

提供解包的代理，这是为了一些引擎操作的方法，比如算数方法或JSON的字符串化。

```javascript
_.prototype.valueOf = _.prototype.toJSON = _.prototype.value;
_.prototype.toString = function(){
	return '' + this._wrapped;
}
```

最后兼容AMD规范

```javascript
if(typeof define === 'function' && define.amd){
	define('underscore', [], function(){
		return _;
	})
}
```



























