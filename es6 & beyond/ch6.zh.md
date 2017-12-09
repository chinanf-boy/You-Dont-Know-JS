
# 你不知道JS: ES6与超越

# 第6章: api添加

从转换值的数学计算,6增添了许多静态的属性和方法的各种内置的本地人和物来帮助共同的任务ㄢ此外,一些本地的实例通过各种新的原型方法具有新的功能ㄢ

**注: **大多数的这些特点可以忠实地polyfilledㄢ我们不会进入这样的细节在这里,但看看"6垫片ℽ(<https://github.com/paulmillr/es6-shim/>符合标准的垫片/ polyfills)ㄢ

## `阵列`

各种用户库中js最常用的扩展特性之一是数组类型ㄢ应该是6增添了一些帮手来阵不足为奇,静态和原型(实例)ㄢ

### `数组(ⅆ)`静态函数

有一个众所周知的问题了`数组(..)`构造函数,也就是说,如果只有一个参数通过,而这个参数是一个数字,而不是用它中的一个值创建一个数组的元素,它就用`长度`与数字相等的属性ㄢ这个动作产生的不幸和离奇的"空槽ℽ的行为,对JS数组的辱骂ㄢ

`数组(ⅆ)`取代`数组(..)`作为数组的首选函数构造函数,因为`数组(ⅆ)`没有特殊的单号参数情况ㄢ考虑: 

```js
var a = Array( 3 );
a.length;						// 3
a[0];							// undefined

var b = Array.of( 3 );
b.length;						// 1
b[0];							// 3

var c = Array.of( 1, 2, 3 );
c.length;						// 3
c;								// [1,2,3]
```

你想在什么情况下使用?`数组(ⅆ)`而不是只创建一个带有字面语法的数组,比如`C = [1,2,3]`?有两种可能的情况ㄢ

如果你有一个回调,它应该包装数组中传递给它的参数,`数组(ⅆ)`完美地吻合帐单ㄢ这可能不是很常见,但可能会让你发痒ㄢ

另一种情况是如果您子类ㄢ`阵列`(参见第3章中的"类ℽ),并希望能够在子类的实例中创建和初始化元素,例如: 

```js
class MyCoolArray extends Array {
	sum() {
		return this.reduce( function reducer(acc,curr){
			return acc + curr;
		}, 0 );
	}
}

var x = new MyCoolArray( 3 );
x.length;						// 3 -- oops!
x.sum();						// 0 -- oops!

var y = [3];					// Array, not MyCoolArray
y.length;						// 1
y.sum();						// `sum` is not a function

var z = MyCoolArray.of( 3 );
z.length;						// 1
z.sum();						// 3
```

不能轻易地创建一个构造函数`mycoolarray`这覆盖了`阵列`父构造函数,因为该构造函数实际上需要创建一个行为良好的数组值(初始化`这`)ㄢ"继承ℽ的静态`(ⅆ)`方法对`mycoolarray`子类提供了一个很好的解决方案ㄢ

### `数组(ⅆ)`静态函数

JavaScript中的"类数组对象ℽ是一个具有`长度`属性,特别是整数值为零或更高ㄢ

在js中使用这些值是众所周知的令人沮丧的;需要将它们转换成实际数组是很常见的,因此各种各样的`array.prototype`方法(`地图(ⅆ)`,`IndexOf(..)`等)可供使用ㄢ这个过程通常看起来像: 

```js
// array-like object
var arrLike = {
	length: 3,
	0: "foo",
	1: "bar"
};

var arr = Array.prototype.slice.call( arrLike );
```

另一个常见的任务`切片(..)`常用的是复制一个真正的数组: 

```js
var arr2 = arr.slice();
```

在这两种情况下,新的ES6`Array.from(..)`方法可以是一种更易于理解和优美的(也不太冗长)的方法: 

```js
var arr = Array.from( arrLike );

var arrCopy = Array.from( arr );
```

`数组(ⅆ)`看看第一个参数是一个(见"迭代器ℽ3章),如果是这样的话,它使用的迭代器来产生价值的"复制ℽ到返回的数组ㄢ由于实数组具有这些值的迭代器,所以该迭代器是自动使用的ㄢ

但是如果将数组对象作为第一个参数传递给`数组(ⅆ)`它的行为基本上是一样的ㄢ`slice()`(没有论点!)或`应用(ⅆ)`是,它只是在值上循环,从属性中访问数值命名的属性ㄢ`零`不论什么价值`长度`是.

考虑: 

```js
var arrLike = {
	length: 4,
	2: "foo"
};

Array.from( arrLike );
// [ undefined, undefined, "foo", undefined ]
```

因为位置`零`,`一`,和`三`不存在于`arrlike`结果是`未定义`每个槽的值ㄢ

你可以产生类似的结果: 

```js
var emptySlotsArr = [];
emptySlotsArr.length = 4;
emptySlotsArr[2] = "foo";

Array.from( emptySlotsArr );
// [ undefined, undefined, "foo", undefined ]
```

#### 避免空槽

在前一段代码之间有一个微妙但重要的区别ㄢ`emptyslotsarr`的结果`Array.from(..)`呼叫ㄢ`数组(ⅆ)`永不产生空槽ㄢ

前6,如果你想产生一个数组的初始化与实际有一定的长度`未定义`每个槽中的值(没有空槽!)你得做额外的工作: 

```js
var a = Array( 4 );								// four empty slots!

var b = Array.apply( null, { length: 4 } );		// four `undefined` values
```

但`数组(ⅆ)`现在更容易了: 

```js
var c = Array.from( { length: 4 } );			// four `undefined` values
```

**警告: **使用空槽数组`一`在前一段代码中,某些数组函数可以工作,但有些则忽略空槽(如`地图(ⅆ)`等等)ㄢ您不应该故意使用空槽,因为它几乎肯定会导致程序中出现奇怪的/不可预知的行为ㄢ

#### 映射

这个`数组(ⅆ)`实用工具还有另一个有用的窍门ㄢ第二个参数,如果提供的话,是一个映射回调(几乎与正则表达式相同)ㄢ`阵列#地图(..)`期望)它被用来映射从源到返回目标的每个值ㄢ考虑: 

```js
var arrLike = {
	length: 4,
	2: "foo"
};

Array.from( arrLike, function mapper(val,idx){
	if (typeof val == "string") {
		return val.toUpperCase();
	}
	else {
		return idx;
	}
} );
// [ 0, 1, "FOO", 3 ]
```

**注: **与其他阵列的方法,采取回调,`Array.from(..)`接受一个可选的第三参数,如果设置将指定`这`作为第二个参数传递的回调绑定ㄢ否则,`这`将`未定义`ㄢ

看到"typedarraysℽ5章的例子`数组(ⅆ)`在将值从一个8位数组转换为

### 创建数组和子类型

在最后几节中,我们已经讨论过了ㄢ`数组(ⅆ)`和`数组(ⅆ)`它们都以类似于构造函数的方式创建一个新数组ㄢ但是它们在子类中做什么呢?它们创建基的实例吗?`阵列`或者派生的子类?

```js
class MyCoolArray extends Array {
	..
}

MyCoolArray.from( [1, 2] ) instanceof MyCoolArray;	// true

Array.from(
	MyCoolArray.from( [1, 2] )
) instanceof MyCoolArray;							// false
```

两`(ⅆ)`和`来自ⅆⅆ`使用从中访问的构造函数来构造数组ㄢ所以如果你使用基地`数组(ⅆ)`you'll get an`阵列`实例,但如果您使用`MyCoolArray,(ㄢ)`你会得到一个`mycoolarray`实例ㄢ

在第3章的"类ℽ中,我们介绍了`@ @物种`设置所有内置类(如`阵列`已定义,如果创建新实例,则由任何原型方法使用ㄢ`切片(..)`就是一个很好的例子: 

```js
var x = new MyCoolArray( 1, 2, 3 );

x.slice( 1 ) instanceof MyCoolArray;				// true
```

一般来说,默认行为可能是需要的,但是正如我们在第3章中讨论过的,你_可以_如果你想覆盖: 

```js
class MyCoolArray extends Array {
	// force `species` to be parent constructor
	static get [Symbol.species]() { return Array; }
}

var x = new MyCoolArray( 1, 2, 3 );

x.slice( 1 ) instanceof MyCoolArray;				// false
x.slice( 1 ) instanceof Array;						// true
```

重要的是要注意`@ @物种`设置仅用于原型方法,如`切片(..)`ㄢ它不被`(ⅆ)`和`来自ⅆⅆ`他们都只是用`这`绑定(用于引用的构造函数)ㄢ考虑: 

```js
class MyCoolArray extends Array {
	// force `species` to be parent constructor
	static get [Symbol.species]() { return Array; }
}

var x = new MyCoolArray( 1, 2, 3 );

MyCoolArray.from( x ) instanceof MyCoolArray;		// true
MyCoolArray.of( [2, 3] ) instanceof MyCoolArray;	// true
```

### `copywithin(..)`原型法

`阵列# copywithin(..)`是一种新的赋值方法适用于所有的阵列(包括类型的数组;见5章)ㄢ`copywithin(..)`在同一个数组拷贝到另一个位置的数组的一部分,无论是在覆盖ㄢ

参数_目标_(要复制的索引),_开始_(从复制开始的包容索引)和可选的_结束_(禁止复制的唯一索引)ㄢ如果其中任一参数为负,则它们与数组的结尾相对ㄢ

考虑: 

```js
[1,2,3,4,5].copyWithin( 3, 0 );			// [1,2,3,1,2]

[1,2,3,4,5].copyWithin( 3, 0, 1 );		// [1,2,3,1,5]

[1,2,3,4,5].copyWithin( 0, -2 );		// [4,5,3,4,5]

[1,2,3,4,5].copyWithin( 0, -2, -1 );	// [4,2,3,4,5]
```

这个`copywithin(..)`方法不扩展数组的长度,如前一段代码中的第一个示例所示ㄢ复制仅在数组的结尾到达时停止ㄢ

与你所想的相反,复制并不总是从左到右(升序)顺序ㄢ这可能会导致重复复制已经复制的值,如果从和目标范围重叠,这大概不是期望的行为ㄢ

所以在这种情况下,该算法避免了采用逆序复制避免了ㄢ考虑: 

```js
[1,2,3,4,5].copyWithin( 2, 1 );		// ???
```

如果算法严格地从左向右移动,那么`二`应被复制以覆盖`三`,然后_那个_复制`二`应被复制以覆盖`四`,然后_那个_复制`二`应被复制以覆盖`五`最后你会`[ 1,2,2,2,2 ]`ㄢ

相反,复制算法反转方向和副本ㄢ`四`覆盖`五`,然后复制`三`覆盖`四`,然后复制`二`覆盖`三`最后的结果是`[ 1,2,2,3,4 ]`ㄢ从期望的角度来说,这可能更"正确ℽ,但如果你只考虑一种天真的"左对右ℽ的复制算法,它可能会令人困惑ㄢ

### `填充(ⅆ)`原型法

填充现有数组完全(或部分)与指定的值为6的原生支持`#填充阵列(..)`方法: 

```js
var a = Array( 4 ).fill( undefined );
a;
// [undefined,undefined,undefined,undefined]
```

`填充(ⅆ)`可以有选择地将_开始_和_结束_参数,表示要填充的数组的子集部分,例如: 

```js
var a = [ null, null, null, null ].fill( 42, 1, 3 );

a;									// [null,42,42,null]
```

### `发现(ⅆ)`原型法

在数组中搜索值最常用的方法通常是`IndexOf(..)`方法,返回位于或位于该值的索引ㄢ`- 1`如果没有找到: 

```js
var a = [1,2,3,4,5];

(a.indexOf( 3 ) != -1);				// true
(a.indexOf( 7 ) != -1);				// false

(a.indexOf( "2" ) != -1);			// false
```

这个`IndexOf(..)`比较需要严格`= = =`匹配,所以搜索`"2ℽ`找不到值`二`反之亦然ㄢ没有办法覆盖匹配算法ㄢ`IndexOf(..)`ㄢ这也是不幸的/不必手动的比较`- 1`价值ㄢ

**提示: **看到_类型与语法_一个有趣的系列文章的标题(和有争议的混乱)技术来解决`- 1`丑陋的`~`算子ㄢ

由于ES5,对匹配逻辑控制的最常见的解决方案是`一些(ⅆ)`方法ㄢ它通过为每个元素调用一个函数回调来工作,直到其中一个调用返回一个`真正的`/一些值,然后停止ㄢ因为您可以定义回调函数,所以可以完全控制如何匹配: 

```js
var a = [1,2,3,4,5];

a.some( function matcher(v){
	return v == "2";
} );								// true

a.some( function matcher(v){
	return v == 7;
} );								// false
```

但是这种方法的缺点是你只能得到`真正的`/`假`指示是否找到适当匹配的值,而不是实际匹配的值ㄢ

6的`发现(ⅆ)`地址本ㄢ它的工作原理基本相同ㄢ`一些(ⅆ)`除了回调返回一个`真正的`/一些值,实际的数组返回值: 

```js
var a = [1,2,3,4,5];

a.find( function matcher(v){
	return v == "2";
} );								// 2

a.find( function matcher(v){
	return v == 7;					// undefined
});
```

使用一个自定义的`匹配(..)`函数还可以与对象等复杂值匹配: 

```js
var points = [
	{ x: 10, y: 20 },
	{ x: 20, y: 30 },
	{ x: 30, y: 40 },
	{ x: 40, y: 50 },
	{ x: 50, y: 60 }
];

points.find( function matcher(point) {
	return (
		point.x % 3 == 0 &&
		point.y % 4 == 0
	);
} );								// { x: 30, y: 40 }
```

**注: **与其他阵列的方法,采取回调,`发现(ⅆ)`接受一个可选的第二个参数,如果设置将指定`这`作为第一个参数传递的回调绑定ㄢ否则,`这`将`未定义`ㄢ

### `findIndex(..)`原型法

前一节说明了`一些(ⅆ)`生成数组搜索的布尔结果,以及`发现(ⅆ)`从数组搜索中产生匹配值本身,还有一个东东ㄢ

`IndexOf(..)`这样做,但无法控制它的匹配逻辑,它总是使用`= = =`严格的平等ㄢ所以ES6的`findIndex(..)`是答案吗?: 

```js
var points = [
	{ x: 10, y: 20 },
	{ x: 20, y: 30 },
	{ x: 30, y: 40 },
	{ x: 40, y: 50 },
	{ x: 50, y: 60 }
];

points.findIndex( function matcher(point) {
	return (
		point.x % 3 == 0 &&
		point.y % 4 == 0
	);
} );								// 2

points.findIndex( function matcher(point) {
	return (
		point.x % 6 == 0 &&
		point.y % 7 == 0
	);
} );								// -1
```

不要使用`findIndex(..)!= 1`(一直以来都是这样`IndexOf(..)`)从搜索中获得布尔值,因为`一些(ⅆ)`已经产生`真正的`/`假`你想要的ㄢ不要这样做`一个[ a.findindex(..)]`为了得到匹配的值,因为这是什么`发现(ⅆ)`完成ㄢ最后,使用`IndexOf(..)`如果需要严格匹配的索引,或者`findIndex(..)`如果需要更定制的匹配项的索引ㄢ

**注: **与其他阵列的方法,采取回调,`findIndex(..)`接受一个可选的第二个参数,如果设置将指定`这`作为第一个参数传递的回调绑定ㄢ否则,`这`将`未定义`ㄢ

### `entries()`,`values()`,`keys()`原型方法

在第3章中,我们演示了数据结构如何通过迭代器提供一个有模式的逐项枚举它们的值ㄢ然后阐述了这种方法在5章中,我们探讨了如何在新的6系列(地图,设置,等)提供生产各种迭代的几种方法ㄢ

因为它不是新的6,`阵列`可能不被传统上认为是"集合ℽ,但在某种意义上,它提供了相同的迭代器方法: `entries()`,`values()`,和`keys()`ㄢ考虑: 

```js
var a = [1,2,3];

[...a.values()];					// [1,2,3]
[...a.keys()];						// [0,1,2]
[...a.entries()];					// [ [0,1], [1,2], [2,3] ]

[...a[Symbol.iterator]()];			// [1,2,3]
```

就像`集`,默认`阵列`迭代器与`values()`返回ㄢ

在本章前面的"避免空槽ℽ中,我们演示了如何`数组(ⅆ)`将数组中的空槽当作当前插槽使用ㄢ`未定义`在他们ㄢ这实际上是因为在套子下,数组迭代器的行为是这样的: 

```js
var a = [];
a.length = 3;
a[1] = 2;

[...a.values()];		// [undefined,2,undefined]
[...a.keys()];			// [0,1,2]
[...a.entries()];		// [ [0,undefined], [1,2], [2,undefined] ]
```

## `对象`

添加了一些额外的静态助手ㄢ`对象`ㄢ传统上,这种功能被视为关注对象值的行为/能力ㄢ

然而,从6,`对象`静态函数也将用于其他类型的通用全局API,这些API在其他一些位置不太自然,`数组(ⅆ)`)ㄢ

### `对象是(ⅆ)`静态函数

这个`对象是(ⅆ)`静态函数比一个更严格的方式对值进行比较ㄢ`= = =`比较ㄢ

`对象是(ⅆ)`调用底层`相同的价值`算法(ES6规格,第7.2.9)ㄢ这个`相同的价值`算法基本上与`= = =`严格的等式比较算法(ES6规格,部分7.2.13),有两个重要的例外情况ㄢ

考虑: 

```js
var x = NaN, y = 0, z = -0;

x === x;							// false
y === z;							// true

Object.is( x, x );					// true
Object.is( y, z );					// false
```

你应该继续使用`= = =`用于严格的相等比较;`对象是(ⅆ)`不应该被看作是接线员的替代品ㄢ然而,在您试图严格确定一个`南`或`- 0`价值,`对象是(ⅆ)`现在是首选ㄢ

**注: **6还增加了一个`数ㄢisnan(..)`实用程序(本章稍后讨论),这可能是一个稍微方便一些的测试;您可能喜欢`isnan(x)的数量ㄢ`结束`对象是(x,楠)`ㄢ你_可以_准确测试`- 0`一个笨拙的`x=0和1`但在这种情况下`对象是(x,- 0)`好多了ㄢ

### `对象ㄢgetownpropertysymbols(..)`静态函数

2章中的"符号ℽ部分讨论了6的新象征原始值类型ㄢ

符号可能主要用作对象上的特殊(元)属性ㄢ所以`对象ㄢgetownpropertysymbols(..)`实用程序被引入,它只在对象上直接检索符号属性: 

```js
var o = {
	foo: 42,
	[ Symbol( "bar" ) ]: "hello world",
	baz: true
};

Object.getOwnPropertySymbols( o );	// [ Symbol(bar) ]
```

### `对象ㄢsetprototypeof(..)`静态函数

在第2章中,我们提到了`对象ㄢsetprototypeof(..)`实用程序,这(不足为奇)设置`[原型]`用于某事物的目的_行为代表团_(见_对象原型_本系列的标题)ㄢ考虑: 

```js
var o1 = {
	foo() { console.log( "foo" ); }
};
var o2 = {
	// .. o2's definition ..
};

Object.setPrototypeOf( o2, o1 );

// delegates to `o1.foo()`
o2.foo();							// foo
```

另外: 

```js
var o1 = {
	foo() { console.log( "foo" ); }
};

var o2 = Object.setPrototypeOf( {
	// .. o2's definition ..
}, o1 );

// delegates to `o1.foo()`
o2.foo();							// foo
```

在前两段中,`O2`和`O1`出现在结尾处`O2`定义ㄢ更常见的是,一个`O2`和`O1`在顶部指定`O2`定义,因为它是类,也与`__proto__`在对象文本中(见"设置ℽ)`[原型]`"在第2章中)ㄢ

**警告: **设置`[原型]`在对象创建之后是合理的,如图所示ㄢ但后来改变它通常不是一个好主意,通常会导致比清晰更混乱ㄢ

### `对象(ⅆ)`静态函数

许多JavaScript库/框架提供了用于将一个对象的属性复制到另一个对象的实用程序(例如jQuery的)ㄢ`扩展(ⅆ)`)ㄢ这些不同的实用程序之间存在着各种细微差别,例如是否具有值的属性ㄢ`未定义`被忽略或不被忽略ㄢ

6加`对象(ⅆ)`,这是这些算法的简化版本ㄢ第一个论点是_目标_以及传递的任何其他参数都是_来源_按清单顺序处理ㄢ对于每个源,其可枚举和自身(例如,不是"继承ℽ)键,包括符号,复制如平原`=`分配ㄢ`对象(ⅆ)`返回目标对象ㄢ

欺骗

```js
var target = {},
	o1 = { a: 1 }, o2 = { b: 2 },
	o3 = { c: 3 }, o4 = { d: 4 };

// setup read-only property
Object.defineProperty( o3, "e", {
	value: 5,
	enumerable: true,
	writable: false,
	configurable: false
} );

// setup non-enumerable property
Object.defineProperty( o3, "f", {
	value: 6,
	enumerable: false
} );

o3[ Symbol( "g" ) ] = 7;

// setup non-enumerable symbol
Object.defineProperty( o3, Symbol( "h" ), {
	value: 8,
	enumerable: false
} );

Object.setPrototypeOf( o3, o4 );
```

只有性能`一`,`B`,`C`,`E`,和`符号(g)`将被复制到`目标`: 

```js
Object.assign( target, o1, o2, o3 );

target.a;							// 1
target.b;							// 2
target.c;							// 3

Object.getOwnPropertyDescriptor( target, "e" );
// { value: 5, writable: true, enumerable: true,
//   configurable: true }

Object.getOwnPropertySymbols( target );
// [Symbol("g")]
```

这个`D`,`F`,和`符号("hℽ)`省略的属性复制;不可枚举属性和非国有性质都排除在分配ㄢ也,`E`作为正常属性赋值复制,而不是作为只读属性复制ㄢ

在前面的一节中,我们展示了`setprototypeof(..)`设立一个`[原型]`之间的关系`O2`和`O1`对象ㄢ还有另一种形式利用`对象(ⅆ)`: 

```js
var o1 = {
	foo() { console.log( "foo" ); }
};

var o2 = Object.assign(
	Object.create( o1 ),
	{
		// .. o2's definition ..
	}
);

// delegates to `o1.foo()`
o2.foo();							// foo
```

**注: ** `对象创建(..)`是ES5标准工具,创建一个空的对象,`[原型]`-链接ㄢ看到_对象原型_此系列的标题以获取更多信息ㄢ

## `数学`

6添加一些新的数学工具,填孔或援助与常用操作ㄢ这些都可以通过手工计算的,但现在大部分都是定义本身,在某些情况下,JS引擎可以更优化地执行计算,或进行更好的小数精度比他们的手工ㄢ

很可能asm.js/transpiled JS代码(见_异步和性能_这一系列的标题)是许多实用工具的消费者,而不是直接的开发者ㄢ

Trigonometry: 

-   `双曲余弦(..)`双曲余弦
-   `ACOSH(..)`- Hyperbolic arccosine
-   `双曲正弦(..)`- Hyperbolic sine
-   `反(..)`- Hyperbolic arcsine
-   `tanh(..)`双曲正切
-   `ATANH(..)`双曲正切函数
-   `最(..)`的平方和的平方根(即广义勾股定理)

算术: 

-   `银行(..)`-立方根
-   `clz32(..)`-在32位二进制表示中计数前导零
-   `expm1(..)`-同`(x)- 1`
-   `log2(..)`-二进制对数(日志基数2)
-   `log10(..)`-日志基10
-   `log1p(..)`-同`日志(x + 1)`
-   `IMul(..)`两位数的32位整数乘法

元:

-   `符号(..)`-返回数字的符号
-   `trunc(..)`-只返回数字的整数部分ㄢ
-   `发现(..)`-舍入到最近的32位(单精度)浮点值

## `数`

重要的是,为了您的程序正常工作,它必须准确地处理数字ㄢ6添加一些额外的特性和功能来帮助普通数字运算ㄢ

两个补充`数`只是引用已有的全局变量: `数ㄢparseInt(..)`和`数ㄢparseFloat(..)`ㄢ

### 静态特性

增加了一些有用的数字常量ES6静态特性: 

-   `number.epsilon`-两个数之间的最小值: `2 - 52`(见第2章)_类型与语法_关于使用这个值作为浮点运算不精确性公差的系列文章的标题)
-   `number.max_safe_integer`-可以安全地在js数字值中明确表示的最高整数: `2 - 53 - 1`
-   `number.min_safe_integer`-可以安全地在js数字值中明确地表示的最小整数: `-(2 - 53 - 1)`或`(- 2)53 + 1`ㄢ

**注: **见第2章_类型与语法_关于安全整数的更多信息ㄢ

### `数ㄢisnan(..)`静态函数

全球标准`isnan(..)`自从它开始以来,公用事业就已经被打破了ㄢ`真正的`对于那些不是数字的东西,不只是实际的`南`值,因为它强制参数数字型(可虚导致南)ㄢ6添加一个固定的效用`数ㄢisnan(..)`那样就可以了: 

```js
var a = NaN, b = "NaN", c = 42;

isNaN( a );							// true
isNaN( b );							// true -- oops!
isNaN( c );							// false

Number.isNaN( a );					// true
Number.isNaN( b );					// false -- fixed!
Number.isNaN( c );					// false
```

### `数量有限(..)ㄢ`静态函数

查看函数名是一种诱惑ㄢ`需(..)`假设它是"不是无限的ℽㄢ但那并不完全正确ㄢ还有更多的细微差别,这一新的ES6效用ㄢ考虑: 

```js
var a = NaN, b = Infinity, c = 42;

Number.isFinite( a );				// false
Number.isFinite( b );				// false

Number.isFinite( c );				// true
```

全球标准`需(..)`强迫它的参数,但`数量有限(..)ㄢ`省略强制行为: 

```js
var a = "42";

isFinite( a );						// true
Number.isFinite( a );				// false
```

您可能仍然喜欢强制,在这种情况下使用全局ㄢ`需(..)`是一个有效的选择ㄢ或者,也许更明智,你可以使用`数量有限(X)ㄢ`,其中明确胁迫`X`一个数字,然后传入它(见第4章)_类型与语法_本系列的标题)ㄢ

### 整数相关静态函数

JavaScript的数值总是浮点(IEEE-754)ㄢ因此,决定一个数是否是一个整数的概念并不是检查它的类型,因为JS没有这样的区分ㄢ

相反,您需要检查值是否有非零小数部分ㄢ做到这一点最简单的方法通常是: 

```js
x === Math.floor( x );
```

6添加`数ㄢ霍耳(..)`助手实用程序可能更有效地确定这种质量: 

```js
Number.isInteger( 4 );				// true
Number.isInteger( 4.2 );			// false
```

**注: **在JavaScript中,`四`,`4ㄢ`,`四`,或`四`ㄢ所有这些都被认为是一个"整数ℽ,并由此产生ㄢ`真正的`从`数ㄢ霍耳(..)`ㄢ

此外,`数ㄢ霍耳(..)`过滤掉一些显然不是整数值的`数学=地板(x)`可能会混淆: 

```js
Number.isInteger( NaN );			// false
Number.isInteger( Infinity );		// false
```

使用"整数ℽ有时是一个重要的信息位,因为它可以简化某些类型的算法ㄢjs代码本身不会因为过滤而运行得更快ㄢ

因为`数ㄢ霍耳(..)`的处理方法`南`和`无穷`定义一个值`isfloat(..)`实用性不会像`!数ㄢ霍耳(..)`ㄢ你需要做点什么: 

```js
function isFloat(x) {
	return Number.isFinite( x ) && !Number.isInteger( x );
}

isFloat( 4.2 );						// true
isFloat( 4 );						// false

isFloat( NaN );						// false
isFloat( Infinity );				// false
```

**注: **这似乎很奇怪,但无穷大既不应被看作整数,也不应该是浮点数ㄢ

6还定义了一个`数ㄢissafeinteger(..)`实用程序,它检查以确保该值既是整数又在`number.min_safe_integer`ℴ`number.max_safe_integer`(包括)ㄢ

```js
var x = Math.pow( 2, 53 ),
	y = Math.pow( -2, 53 );

Number.isSafeInteger( x - 1 );		// true
Number.isSafeInteger( y + 1 );		// true

Number.isSafeInteger( x );			// false
Number.isSafeInteger( y );			// false
```

## `字符串`

弦已经有了几个助手6之前,更已被添加到混合ㄢ

### Unicode功能

第2章讨论的"Unicode感知字符串操作ℽ`字符串ㄢfromcodepoint(..)`,`字符串# codepointat(..)`,和`字符串#规范(..)`详细ㄢ它们已经添加到js字符串值中,以改进unicode支持ㄢ

```js
String.fromCodePoint( 0x1d49e );			// "𝒞"

"ab𝒞d".codePointAt( 2 ).toString( 16 );		// "1d49e"
```

这个`规范化(ⅆ)`字符串原型法进行规范,结合字符Unicode相邻ℽ结合的标记ℽ或分解组合特征ㄢ

通常,标准化不会对字符串的内容产生可见的影响,但会改变字符串的内容,这会影响字符串的内容ㄢ`长度`属性以及位置如何访问字符的行为: 

```js
var s1 = "e\u0301";
s1.length;							// 2

var s2 = s1.normalize();
s2.length;							// 1
s2 === "\xE9";						// true
```

`规范化(ⅆ)`接受一个可选参数,该参数指定要使用的规范化窗体ㄢ此参数必须是以下四个值之一: `"NFCℽ`(默认),`"NFDℽ`,`"nfkcℽ`,或`"nfkdℽ`ㄢ

**注: **规范化形式及其对字符串的影响远远超出我们在这里讨论的范围ㄢ参见"Unicode标准化表单ℽ(<http://www.unicode.org/reports/tr15/>)获取更多信息ㄢ

### `字符串(ⅆ)`静态函数

这个`字符串(ⅆ)`实用程序被提供为内置的标记函数,用于与模板字符串字面量(见第2章)一起获取未经转义序列处理的原始字符串值ㄢ

这个函数几乎不会手动调用,但将与标记的模板文本一起使用: 

```js
var str = "bc";

String.raw`\ta${str}d\xE9`;
// "\tabcd\xE9", not "	abcdé"
```

在结果字符串中,`\`和`T`是单独的原始字符,而不是转义字符ㄢ`T`ㄢUnicode转义序列也是如此ㄢ

### `重复(ⅆ)`原型函数

在Python和露比这样的语言中,可以重复使用字符串作为: 

```js
"foo" * 3;							// "foofoofoo"
```

这在js中不起作用,因为`*`乘法仅定义为数字,因此`"fooℽ`强迫的`南`数ㄢ

然而,6定义一个字符串原型法`重复(ⅆ)`完成任务: 

```js
"foo".repeat( 3 );					// "foofoofoo"
```

### 检查字符串函数

除了`字符串# indexOf(..)`和`#字符串的字符串(..)`从之前的6,搜索/检查三的新方法已被添加: `从(..)`,`EndsWith(..)`,和`包括(ⅆ)`ㄢ

```js
var palindrome = "step on no pets";

palindrome.startsWith( "step on" );	// true
palindrome.startsWith( "on", 5 );	// true

palindrome.endsWith( "no pets" );	// true
palindrome.endsWith( "no", 10 );	// true

palindrome.includes( "on" );		// true
palindrome.includes( "on", 6 );		// false
```

对于所有字符串搜索/检查方法,如果您查找空字符串`"ℽ`它可以在字符串的开头或结尾找到ㄢ

**警告: **默认情况下,这些方法不会接受搜索字符串的正则表达式ㄢ有关第7章的信息,请参见"正则表达式符号ℽㄢ`isregexp`检查在第一个参数上执行的操作ㄢ

## 回顾

6添加许多额外的API助手的各种内置的本地对象: 

-   `阵列`增加了`(ⅆ)`和`来自ⅆⅆ`静态函数,以及原型函数`copywithin(..)`和`填充(ⅆ)`ㄢ
-   `对象`添加静态函数`是(ⅆ)`和`分配(ⅆ)`ㄢ
-   `数学`添加静态函数`ACOSH(..)`和`clz32(..)`ㄢ
-   `数`添加静态属性`number.epsilon`以及静态函数,比如`数量有限(..)ㄢ`ㄢ
-   `字符串`添加静态函数`字符串ㄢfromcodepoint(..)`和`字符串(ⅆ)`以及原型函数`重复(ⅆ)`和`包括(ⅆ)`ㄢ

大多数这些添加剂可以polyfilled(见6垫片),并受到普遍的JS库/框架工具ㄢ
