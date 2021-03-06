
# 你不知道JS: ES6与超越

# 第5章藏品

结构化收集和数据访问是任何js程序的关键组成部分. 从语言开始到现在,数组和对象一直是我们创建数据结构的主要机制. 当然,许多高级数据结构已经建立在这些之上,如用户地库. 

截至6,一些最有用的(和性能优化!)数据结构抽象已作为语言的本机组件添加. 

我们先看这一章开始. _typedarrays_在当代,ES5前几年的努力,但只有规范作为同伴的WebGL和JavaScript本身不. 截至6,这些已被语言规范直接采用,这给他们一流的地位. 

映射就像对象(键/值对),但不只是键的字符串,您可以使用任何值ℴℴ甚至是另一个对象或映射!集合类似于数组(值列表),但值是唯一的;如果您添加了一个副本,则它将被忽略. 也有弱(相对于内存/垃圾收集)同行: WeakMap和weakset. 

## typedarrays

正如我们在_类型与语法_这个系列的名称,js确实有一组内置类型,比如`数`和`一串`. 看看一个名为"类型数组"的特性,并假定它是一个特定类型的值数组,就像一个只有字符串的数组一样,是很诱人的. 

但是,类型数组实际上更多地是使用数组语义(索引访问等)提供对二进制数据的结构化访问. 名称中的"类型"指的是位位桶类型上的一种"视图",它本质上是一种映射,即比特是否应该被看作是一个8位有符号整数数组ㄡ16位有符号整数等. 

你怎么建造这样一个桶?它被称为"缓冲区",而你用它最直接地构造它. `ArrayBuffer(..)`构造函数: 

```js
var buf = new ArrayBuffer( 32 );
buf.byteLength;							// 32
```

`buf`现在是一个二进制缓冲区,长度为32字节(256位),这是预先初始化为所有的. `零`一个缓冲区本身不允许你进行任何交互,除了检查它. `bytelength`财产. 

**提示: **几个Web平台具有使用或返回数组缓冲区的功能,如`有# readasarraybuffer(..)`,`#发送XMLHttpRequest(..)`,和`图像数据`(画布数据). 

但在这个数组缓冲区之上,您可以将一个"视图"分层,它是以类型数组的形式出现的. 考虑: 

```js
var arr = new Uint16Array( buf );
arr.length;							// 16
```

`ARR`是在256位上映射的16位无符号整数的类型数组. `buf`缓冲区,意思是你得到16个元素. 

### ..

理解这一点是非常重要的. `ARR`映射使用端设置(Big-Endian或Little-Endian)的平台上运行的JS. 这是一个问题如果二进制数据创建一个..但解释平台上与之相反的... 

Endian表示,如果低字节(收集8)一个多字节数,如16位的无符号整型我们在早段创造--在右边或左的字节数. 

例如,让我们想象一下10号`三千零八十五`以16位表示. 如果只有一个16位的数字容器,它将用二进制表示. `0000110000001101`(十六进制`0c0d`)无论... 

但如果`三千零八十五`为两个8位数字的排列顺序,将内存中的存储的影响: 

-   `0000110000001101`/`0c0d`(大端)
-   `0000110100001100`/`0d0c`(小端)

如果你收到了`三千零八十五`作为`0000110100001100`从一个小端字节序的系统,但你的上一层这一种观点,大端系统,你会看见价值`三千三百四十`(10)和`0d0c`(16进制). 

小端是这些天网络上最常见的表现,但有一定的浏览器,那不是真的. 你明白两者的生产者和消费者的一块二进制数据格式是很重要的. 

从MDN,这里是一个快速的方法来测试你的JavaScript的..: 

```js
var littleEndian = (function() {
	var buffer = new ArrayBuffer( 2 );
	new DataView( buffer ).setInt16( 0, 256, true );
	return new Int16Array( buffer )[0] === 256;
})();
```

`littleendian`将`真正的`或`假`对于大多数浏览器,它应该返回`真正的`. 本试验采用`DataView(..)`,它允许更低级的ㄡ细粒度的控制访问(设置/获取)从缓冲区层上放置的视图的比特. 的第三个参数`setint16(..)`前一段代码的方法是告诉`数据视图`你想它使用操作什么... 

**警告: **不要混淆阵列缓冲底层二进制存储端与给定数量的代表暴露在一个js程序时. 例如,`(3085)说明(2). `返回`"110000001101"`假设四领先`"0"`的出现是大端表示. 事实上,这种表示基于单个16位视图,而不是两个8位字节的视图. 这个`数据视图`以上的测试是确定你的JS环境设计的最佳方式. 

### 多个视图

单个缓冲区可以附加多个视图,例如

```js
var buf = new ArrayBuffer( 2 );

var view8 = new Uint8Array( buf );
var view16 = new Uint16Array( buf );

view16[0] = 3085;
view8[0];						// 13
view8[1];						// 12

view8[0].toString( 16 );		// "d"
view8[1].toString( 16 );		// "c"

// swap (as if endian!)
var tmp = view8[0];
view8[0] = view8[1];
view8[1] = tmp;

view16[0];						// 3340
```

类型化数组构造函数具有多个签名变体. 到目前为止,我们只向他们传递了一个现有的缓冲区. 但是,该表单还需要两个额外参数: `byteoffset`和`长度`. 换句话说,您可以在一个位置以外的位置启动类型数组视图. `零`您可以使它小于缓冲区的整个长度. 

如果二进制数据的缓冲区包含非均匀大小/位置的数据,这种技术是非常有用的. 

例如,考虑一个二进制的缓冲区,2字节数(又名"字")开头,后面跟着两个字节数,后面跟着一个32位浮点数. 下面是如何在同一缓冲区ㄡ偏移量和长度上使用多个视图访问该数据的方法: 

```js
var first = new Uint16Array( buf, 0, 2 )[0],
	second = new Uint8Array( buf, 2, 1 )[0],
	third = new Uint8Array( buf, 3, 1 )[0],
	fourth = new Float32Array( buf, 4, 4 )[0];
```

### 中的TypedArray构造函数

除了`(缓冲器,[偏移量,长度] ]`在上一节中检查的表单中,类型化数组构造函数也支持这些表单: 

-   [构造函数]`(长度)`: 在一个新缓冲区上创建一个新视图`长度`字节
-   [构造函数]`(typedarr)`: 创建一个新视图和缓冲区,并从`typedarr`看法
-   [构造函数]`(obj)`: 创建一个新的视图和缓冲,遍历数组或对象`obj`复制内容

以下类型的数组构造器可以ES6: 

-   `int8array`(8位有符号整数),`uint8array`(8位无符号整数)`ℴ`uint8clampedarray`8位无符号整数,每个值在设置为`零`ℴ`二百五十五
-   `范围)`int16array`(16位有符号整数),`uint16array
-   `(16位无符号整数)`int32array`(32位有符号整数),`uint32array
-   `(32位无符号整数)`float32array
-   `(32位浮点IEEE-754)`float64array

(64位浮点,IEEE-754)

类型化数组构造函数的实例与普通本机数组几乎相同. 有些差异包括一个固定的长度和值都是相同的"类型". `然而,它们共享的大部分是相同的. `原型

方法.因此,您可能会将它们作为常规数组使用,而无需转换. 

```js
var a = new Int32Array( 3 );
a[0] = 10;
a[1] = 20;
a[2] = 30;

a.map( function(v){
	console.log( v );
} );
// 10 20 30

a.join( "-" );
// "10-20-30"
```

**例如:**警告: `你不能用某些`array.prototype`与typedarrays没有意义的方法,如应用程序(`拼接(ⅆ)`,`推(ⅆ)`等等`concat(..)

. `注意,在typedarrays元素真的是不得不宣布比特大小. 如果你有`uint8array

并尝试将大于8位值的某个元素分配到它的一个元素中,该值被环绕,以便保持在位长度内. 

```js
var a = new Uint8Array( 3 );
a[0] = 10;
a[1] = 20;
a[2] = 30;

var b = a.map( function(v){
	return v * v;
} );

b;				// [100, 144, 132]
```

这可能导致的问题,如果你想,例如,广场中的值中的TypedArray. 考虑: `这个`二十`和`三十`值,当平方时,会导致位溢出. 为了绕过这种限制,您可以使用`从中的TypedArray #(..)

```js
var a = new Uint8Array( 3 );
a[0] = 10;
a[1] = 20;
a[2] = 30;

var b = Uint16Array.from( a, function(v){
	return v * v;
} );

b;				// [100, 400, 900]
```

功能: `看到"`数组(ⅆ)`"静态函数"第6章中的章节,了解更多有关`数组(ⅆ)

这是与TypedArrays在一起. 具体来说,"映射"部分解释了作为第二个参数被接受的映射函数. `一个有趣的现象是,TypedArrays有一个`排序(ⅆ)

```js
var a = [ 10, 1, 2, ];
a.sort();								// [1,10,2]

var b = new Uint8Array( [ 10, 1, 2 ] );
b.sort();								// [1,2,10]
```

就像普通阵列的方法,但这一默认数值排序比较代替强迫值字符串字典比较. 例如:`这个`中的TypedArray #排序(..)`接受一个可选的比较函数参数`阵列#排序(..)

## ,它的工作方式完全相同. 

地图

如果您有很多js经验,那么您知道对象是创建无序键/值对数据结构的主要机制,也就是所谓的映射. 然而,对象作为映射的主要缺点是不能使用非字符串值作为键. 

```js
var m = {};

var x = { id: 1 },
	y = { id: 2 };

m[x] = "foo";
m[y] = "bar";

m[x];							// "bar"
m[y];							// "bar"
```

例如,考虑: `到底是为什么呢两个对象`X`和`Y`对两stringify`"\[对象对象]`所以只有一把钥匙在里面`M

. 

```js
var keys = [], vals = [];

var x = { id: 1 },
	y = { id: 2 };

keys.push( x );
vals.push( "foo" );

keys.push( y );
vals.push( "bar" );

keys[0] === x;					// true
vals[0];						// "foo"

keys[1] === y;					// true
vals[1];						// "bar"
```

有些人通过在值的数组旁边维护一个非字符串键的并行数组来实现假映射,例如: 

当然,您不希望自己管理这些并行数组,因此您可以定义一个数据结构,该方法具有在覆盖下自动执行管理的方法. 除了要自己做这项工作之外,主要的缺点是访问不再是O(1)时间复杂度,而是O(n). `但截至6,没有必要再做这个!只使用`地图(ⅆ)

```js
var m = new Map();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );
m.set( y, "bar" );

m.get( x );						// "foo"
m.get( y );						// "bar"
```

: `唯一的缺点是你不能使用`\[ ]`用于设置和检索值的括号访问语法. 但`得到(ⅆ)`和`设置(ⅆ)

代替适当地工作. `若要从映射中删除元素,请不要使用`删除`运算符,但使用`删除(..)

```js
m.set( x, "foo" );
m.set( y, "bar" );

m.delete( y );
```

方法: `您可以清除整个地图的内容`clear()`. 要获得映射的长度(即键的数量),请使用`大小`财产(不`长度

```js
m.set( x, "foo" );
m.set( y, "bar" );
m.size;							// 2

m.clear();
m.size;							// 0
```

): `这个`地图(ⅆ)`构造函数也可以得到一个(见"迭代器"3章),它必须产生一个列表的数组,其中每个阵列中的第一个项目是关键,第二项是值. 这种迭代格式与`entries()

```js
var m2 = new Map( m.entries() );

// same as:
var m2 = new Map( m );
```

方法,在下一个解释`entries()`第二种较短的形式更可取. 

当然,您可以手动指定一个_条目_列表中的数组(键/值数组)`地图(ⅆ)`构造函数的形式: 

```js
var x = { id: 1 },
	y = { id: 2 };

var m = new Map( [
	[ x, "foo" ],
	[ y, "bar" ]
] );

m.get( x );						// "foo"
m.get( y );						// "bar"
```

### 地图的价值

要从一个映射中获取值列表,请使用`值(..)`,它返回一个迭代器. 在第2章和第3章中,我们介绍了顺序处理迭代器(如数组)的各种方法,如`ⅆ`扩展运算符和`对..`环. 此外,第6章中的"数组"包括`数组(ⅆ)`详细方法. 考虑: 

```js
var m = new Map();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );
m.set( y, "bar" );

var vals = [ ...m.values() ];

vals;							// ["foo","bar"]
Array.from( m.values() );		// ["foo","bar"]
```

如前一节所讨论的,您可以使用`entries()`(或者默认的map迭代器). 考虑: 

```js
var m = new Map();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );
m.set( y, "bar" );

var vals = [ ...m.entries() ];

vals[0][0] === x;				// true
vals[0][1];						// "foo"

vals[1][0] === y;				// true
vals[1][1];						// "bar"
```

### 地图键

要获取密钥列表,请使用`keys()`,它在map中的键上返回一个迭代器: 

```js
var m = new Map();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );
m.set( y, "bar" );

var keys = [ ...m.keys() ];

keys[0] === x;					// true
keys[1] === y;					// true
```

若要确定映射是否具有给定的键,请使用`有(ⅆ)`: 

```js
var m = new Map();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );

m.has( x );						// true
m.has( y );						// false
```

映射本质上允许您将一些额外的信息(值)与一个对象(键)相关联,而无需将这些信息实际地放在对象本身上. 

虽然您可以使用任何类型的值作为映射的键,但通常您将使用对象,因为字符串和其他原语已经可以作为普通对象的键. 换句话说,您可能希望继续使用普通对象来映射,除非某些或所有键都是对象,在这种情况下,map更合适. 

**警告: **如果你使用一个对象作为地图的关键,对象就被丢弃(所有引用设置)以垃圾收集(GC)回收其内存,地图本身仍将保留其入境. 您需要从映射中删除条目,使其具有GC资格. 在下一节中,我们将看到WeakMaps为对象的键和GC一个更好的选择. 

## WeakMaps

WeakMaps是地图上的变化,其中有大部分相同的外部行为,但在如何不同的内存分配(特别是其GC)作品. 

WeakMaps带(只)对象作为键. 这些物品被拿着. _弱_,这意味着如果对象本身是GC会在weakmap入口也被取消了. 这不是可观察的行为,但是,作为一个对象可以被GC会的唯一方法是如果没有更多的提及它,一旦没有更多的提及它,你没有对象引用来检查它是否存在于weakmap. 

否则,对于weakmap API类似,但更有限: 

```js
var m = new WeakMap();

var x = { id: 1 },
	y = { id: 2 };

m.set( x, "foo" );

m.has( x );						// true
m.has( y );						// false
```

WeakMaps没有`大小`财产或`clear()`方法,也不会在其键ㄡ值或条目上公开任何迭代器. 所以,即使你取消`X`引用,将删除其条目`M`在GC,没有办法告诉. 你只需要用JavaScript的话就可以了!

就像地图,让你有一个对象WeakMaps软关联信息. 但是如果对象不是您完全控制的对象,比如DOM元素,它们特别有用. 如果你使用的是一个地图键可以删除,应该是GC资格当对象,然后weakmap是更合适的选择. 

重要的是要注意,一个weakmap仅持有其_钥匙_弱,而不是它的值. 考虑: 

```js
var m = new WeakMap();

var x = { id: 1 },
	y = { id: 2 },
	z = { id: 3 },
	w = { id: 4 };

m.set( x, y );

x = null;						// { id: 1 } is GC-eligible
y = null;						// { id: 2 } is GC-eligible
								// only because { id: 1 } is

m.set( z, w );

w = null;						// { id: 4 } is not GC-eligible
```

因为这个原因,WeakMaps在我看来是"weakkeymaps更好. "

## 集

集合是唯一值的集合(重复被忽略). 

集合的API类似于map. 这个`添加(..)`方法代替`设置(ⅆ)`方法(有点讽刺意味),没有`得到(ⅆ)`方法. 

考虑: 

```js
var s = new Set();

var x = { id: 1 },
	y = { id: 2 };

s.add( x );
s.add( y );
s.add( x );

s.size;							// 2

s.delete( y );
s.size;							// 1

s.clear();
s.size;							// 0
```

这个`设置(ⅆ)`构造函数形式类似于`地图(ⅆ)`,在能获得一个像另一套,或只是一个数组的值. 然而,不同于`地图(ⅆ)`预计_条目_列表(键/值数组),`设置(ⅆ)`预计_价值观_列表(值数组): 

```js
var x = { id: 1 },
	y = { id: 2 };

var s = new Set( [x,y] );
```

一套不需要`得到(ⅆ)`因为不从一个集合中检索值,而是测试它是否存在,使用`有(ⅆ)`: 

```js
var s = new Set();

var x = { id: 1 },
	y = { id: 2 };

s.add( x );

s.has( x );						// true
s.has( y );						// false
```

**注: **中的比较算法`有(ⅆ)`几乎相同`对象是(ⅆ)`(见第6章),除此之外`- 0`和`零`被看作是相同的而不是不同的. 

### 集合的迭代器

集合具有与地图相同的迭代器方法. 它们的行为不同于集合,但与map迭代器的行为是对称的. 考虑: 

```js
var s = new Set();

var x = { id: 1 },
	y = { id: 2 };

s.add( x ).add( y );

var keys = [ ...s.keys() ],
	vals = [ ...s.values() ],
	entries = [ ...s.entries() ];

keys[0] === x;
keys[1] === y;

vals[0] === x;
vals[1] === y;

entries[0][0] === x;
entries[0][1] === x;
entries[1][0] === y;
entries[1][1] === y;
```

这个`keys()`和`values()`迭代器都生成集合中唯一值的列表. 这个`entries()`迭代器生成一个条目数组列表,其中数组的两个项都是唯一的集合值. 一个集合的默认迭代器是`values()`迭代器. 

集合的固有唯一性是它最有用的特性. 例如:

```js
var s = new Set( [1,2,3,4,"1",2,4,"5"] ),
	uniques = [ ...s ];

uniques;						// [1,2,3,4,"1","5"]
```

集合唯一性不允许强制,所以`一`和`"1"`被视为不同的值. 

## weaksets

而weakmap持有其键弱(但其价值观强),一个weakset持有其价值弱(真的没有钥匙). 

```js
var s = new WeakSet();

var x = { id: 1 },
	y = { id: 2 };

s.add( x );
s.add( y );

x = null;						// `x` is GC-eligible
y = null;						// `y` is GC-eligible
```

**警告: **weakset值必须是对象,不是原始值是允许集. 

## 回顾

6定义了一些有用的集合,使以结构化的方式更加高效和有效的数据工作. 

typedarrays提供"观"的二进制数据缓冲区,不同的整数类型对齐,如8位无符号整数和32浮. 数组访问二进制数据使操作m

映射是键值对,其中键可以是对象,而不是字符串/原语. 集合是唯一的值列表(任何类型). 

WeakMaps地图是其中的关键(对象)是弱,所以GC是免费的收集条目如果它的最后一个对象引用. weaksets是集值弱举行,又使GC可以如果这是最后的参考对象删除条目. 
