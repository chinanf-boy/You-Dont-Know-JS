
# 你不知道JS：ES6与超越

# 第7章：元编程

元编程是编程的，操作的目标是程序本身的行为。换句话说，它是对程序编程的编程。是的，一口，嗯？

例如，如果您探测一个对象之间的关系。`一`和另一个`B`——他们`[原型]`有关系吗？-使用`a.isprototypeof（B）`这通常被称为内省，是元编程的一种形式。宏（在js中还不存在）——代码在编译时修改自己——是元编程的另一个明显的例子。用对象枚举对象的键`对..`循环或检查对象是否为_实例_“类构造函数”是其他常见的元编程任务。

元编程关注以下一个或多个：代码检查自身、代码修改本身或修改默认语言行为的代码，从而影响其他代码。

元编程的目标是利用语言自身的固有功能，使代码的其余部分更具描述性、表达性和/或灵活性。因为这个_元_元编程的本质，要比它更精确地定义它有点困难。理解元编程的最好方法是通过示例看到它。

6添加一些新的形式/功能的元编程上什么JS已经。

## 函数名

有些情况下，代码可能需要反省自身，并询问某些函数的名称是什么。如果你问函数的名字是什么，答案是出人意料的有些模棱两可。考虑：

```js
function daz() {
	// ..
}

var obj = {
	foo: function() {
		// ..
	},
	bar: function baz() {
		// ..
	},
	bam: daz,
	zim() {
		// ..
	}
};
```

在前面的片段中，“什么是`foo() OBJ。`“有点细微差别。它是`“foo”`，`“”`，或`未定义`？和什么有关`bar() OBJ。`-它是命名的吗？`“酒吧”`或`“巴兹”`？是`bam() OBJ。`命名`“砰”`或`“哈根达斯”`？怎么样`zim() OBJ。`？

此外，什么是通过回调函数，如：

```js
function foo(cb) {
	// what is the name of `cb()` here?
}

foo( function(){
	// I'm anonymous!
} );
```

函数在程序中可以表达的方法有很多种，但函数的“名称”不总是清晰清楚的。

更重要的是，我们需要区分函数的“名称”是否表示它的名字。`名称`属性——是的，函数有一个名为`名称`——或者它是否引用词汇绑定名，例如`酒吧`在里面`功能bar() { ..}`。

词汇绑定名称是用于递归之类的东西：

```js
function foo(i) {
	if (i < 10) return foo( i * 2 );
	return i;
}
```

这个`名称`属性是用于元编程目的的，因此我们将集中讨论这个问题。

混淆是因为默认情况下，函数（如果有的话）的词法名称也被设置为`名称`财产。其实没有官方要求，行为的ES5（和以前）规格。的设置`名称`物业不标准，但还算可靠。截至6，它规范了。

**提示：**如果函数有一个`名称`值赋值，这通常是开发工具中堆栈跟踪中使用的名称。

### 推论

但是发生了什么`名称`属性，如果函数没有词汇名？

截至6，现在可以确定一个合理的推理规则`名称`属性值来分配一个函数，即使该函数没有使用词汇名称。

考虑：

```js
var abc = function() {
	// ..
};

abc.name;				// "abc"
```

如果我们赋予函数一个词汇名，比如`ABC =功能def() { ..}`，的`名称`财产当然是`“def”`。但是在没有词汇名的情况下，直观地`“ABC”`名字似乎合适。

这里有其他形式，将推断出一个名字（或没有）在6：

```js
(function(){ .. });					// name:
(function*(){ .. });				// name:
window.foo = function(){ .. };		// name:

class Awesome {
	constructor() { .. }			// name: Awesome
	funny() { .. }					// name: funny
}

var c = class Awesome { .. };		// name: Awesome

var o = {
	foo() { .. },					// name: foo
	*bar() { .. },					// name: bar
	baz: () => { .. },				// name: baz
	bam: function(){ .. },			// name: bam
	get qux() { .. },				// name: get qux
	set fuz() { .. },				// name: set fuz
	["b" + "iz"]:
		function(){ .. },			// name: biz
	[Symbol( "buz" )]:
		function(){ .. }			// name: [buz]
};

var x = o.foo.bind( o );			// name: bound foo
(function(){ .. }).bind( o );		// name: bound

export default function() { .. }	// name: default

var y = new Function();				// name: anonymous
var GeneratorFunction =
	function*(){}.__proto__.constructor;
var z = new GeneratorFunction();	// name: anonymous
```

这个`名称`属性在默认情况下是不可写的，但它是可配置的，这意味着您可以使用。`对象。defineproperty（..）`如果需要，手动更改它。

## 元属性

在“`new.target`“第一章3节中，我们介绍了一个新概念：JS ES6元财产。顾名思义，元属性旨在以不可能的属性访问的形式提供特殊的元信息。

在`new.target`，关键字`新的`用作属性访问的上下文。明确`新的`它本身不是一个对象，它使这个功能变得特别。然而，当`new.target`在构造函数调用（用于调用的函数/方法）中使用。`新的`），`新的`变成虚拟上下文，因此`new.target`可以引用目标构造函数`新的`调用。

这是元编程操作的一个明显例子，其目的是从构造函数调用中确定原始内容。`新的`目标通常是为了自省（检查类型/结构）或静态属性访问。

例如，您可能希望在构造函数中有不同的行为，这取决于它是否直接通过子类调用或调用：

```js
class Parent {
	constructor() {
		if (new.target === Parent) {
			console.log( "Parent instantiated" );
		}
		else {
			console.log( "A child instantiated" );
		}
	}
}

class Child extends Parent {}

var a = new Parent();
// Parent instantiated

var b = new Child();
// A child instantiated
```

这里有一点细微的差别，那就是`constructor()`里面的`起源`类定义实际上是给定类的词汇名称（`起源`，尽管语法意味着类是构造函数的独立实体。

**警告：**与Al

## 众所周知的符号

在2章的“符号”一节中，我们覆盖的新的6原始类型`符号`。除了符号，你可以定义你自己的程序，JS预定义了一些内置的符号，称为_众所周知的符号_（周）。

这些符号值的定义主要是为了暴露对JS程序的特殊元属性，从而使您对JS的行为有更多的控制权。

我们将简要介绍每一个并讨论它们的用途。

### `symbol.iterator`

在第2章和第3章中，我们介绍并使用了`@ @迭代器`符号，自动使用`…`利差和`对..`环。我们也看到了`@ @迭代器`定义在新的ES6收藏在5章定义。

`symbol.iterator`表示在任何对象上的特殊位置（属性），在该对象中，语言机制自动查找要构造用于消耗该对象值的迭代器实例的方法。许多对象都带有默认的定义。

但是，我们可以通过设置`symbol.iterator`属性，即使这覆盖了默认迭代器。元编程方面是我们定义了JS的其他部分（即操作符和循环结构）在处理我们定义的对象值时使用的行为。

考虑：

```js
var arr = [4,5,6,7,8,9];

for (var v of arr) {
	console.log( v );
}
// 4 5 6 7 8 9

// define iterator that only produces values
// from odd indexes
arr[Symbol.iterator] = function*() {
	var idx = 1;
	do {
		yield this[idx];
	} while ((idx += 2) < this.length);
};

for (var v of arr) {
	console.log( v );
}
// 5 7 9
```

### `symbol.tostringtag`和`symbol.hasinstance`

最常见的元编程任务之一是对一个值进行反省以找出什么是_友善的_通常是决定什么操作适合于它执行。对于对象，两种最常见的检查技术是`tostring()`和`运算符`。

考虑：

```js
function Foo() {}

var a = new Foo();

a.toString();				// [object Object]
a instanceof Foo;			// true
```

截至6，你可以控制这些操作的行为：

```js
function Foo(greeting) {
	this.greeting = greeting;
}

Foo.prototype[Symbol.toStringTag] = "Foo";

Object.defineProperty( Foo, Symbol.hasInstance, {
	value: function(inst) {
		return inst.greeting == "hello";
	}
} );

var a = new Foo( "hello" ),
	b = new Foo( "world" );

b[Symbol.toStringTag] = "cool";

a.toString();				// [object Foo]
String( b );				// [object cool]

a instanceof Foo;			// true
b instanceof Foo;			// false
```

这个`@ @ tostringtag`原型（或实例本身）上的符号指定要使用的字符串值。`___ [对象]`字符串化所带来的。

这个`@ @ hasinstance`符号是构造函数函数的一种方法，它接收实例对象值，并通过返回决定。`真正的`或`假`如果值应该被视为实例。

**注：**设置`@ @ hasinstance`在函数上，必须使用`对象。defineproperty（..）`作为默认的`function.prototype`是`写：假`。看到_对象原型_此系列的标题以获取更多信息。

### `symbol.species`

在第3章的“类”中，我们介绍了`@ @物种`符号，它控制需要生成新实例的类的内置方法使用哪一个构造函数。

最常见的例子是在子类化`阵列`想定义哪一个构造函数（`数组（..）`或子类）继承的方法`切片（..）`应该使用。默认情况下，`切片（..）`调用一个子类的实例`阵列`将生成这个子类的一个新实例，坦白地说，您可能经常需要它。

但是，可以通过重写类的默认值来进行元编程。`@ @物种`定义：

```js
class Cool {
	// defer `@@species` to derived constructor
	static get [Symbol.species]() { return this; }

	again() {
		return new this.constructor[Symbol.species]();
	}
}

class Fun extends Cool {}

class Awesome extends Cool {
	// force `@@species` to be parent constructor
	static get [Symbol.species]() { return Cool; }
}

var a = new Fun(),
	b = new Awesome(),
	c = a.again(),
	d = b.again();

c instanceof Fun;			// true
d instanceof Awesome;		// false
d instanceof Cool;			// true
```

这个`symbol.species`将内置的本地构造函数设置为`返回此`前面的代码段中所示的行为`酷`定义。它在用户类上没有默认值，但正如所示，行为很容易模拟。

如果需要定义生成新实例的方法，则使用`新的构造函数（符号，物种）（..）`模式代替硬接线`新建这个构造函数（..）`或`新XYZ（…）`。派生类将能够自定义。`symbol.species`控制这些实例构造函数声明。

### `symbol.toprimitive`

在_类型与语法_本系列的标题，我们讨论了`toprimitive`抽象强制操作，当对象必须被强制为某个操作的原始值时使用（例如`= =`比较或`+`此外）。前6，没有办法控制这种行为。

截至6，的`@ @ toprimitive`符号作为任何对象值上的属性可以自定义`toprimitive`通过指定方法来强制。

考虑：

```js
var arr = [1,2,3,4,5];

arr + 10;				// 1,2,3,4,510

arr[Symbol.toPrimitive] = function(hint) {
	if (hint == "default" || hint == "number") {
		// sum all numbers
		return this.reduce( function(acc,curr){
			return acc + curr;
		}, 0 );
	}
};

arr + 10;				// 25
```

这个`symbol.toprimitive`方法将提供_暗示_属于`“字符串”`，`“数”`，或`“默认”`（应解释为`“数”`），取决于调用操作的类型。`toprimitive`期待。在前一段代码中，添加程序`+`操作没有提示（`“默认”`通过）。一个乘法`*`操作提示`“数”`和一个`字符串（ARR）`会提示`“字符串”`。

**警告：**这个`= =`操作员将调用`toprimitive`没有提示的操作——`@ @ toprimitive`方法，如果有提示调用`“默认”`-如果一个对象被比较的其他值不是一个对象。但是，如果两个比较值都是对象，则`= =`是相同的`= = =`这是直接比较引用本身。在这种情况下，`@ @ toprimitive`根本不调用。看到_类型与语法_本系列的标题，了解有关强制和抽象操作的更多信息。

### 正则表达式符号

有四个众所周知的符号可以被重写为正则表达式对象，它控制如何`string.prototype`同名函数：

-   `@ @比赛`：的`symbol.match`正则表达式的值是用于匹配字符串值的全部或部分与给定正则表达式的方法。它被用来`原型。匹配（…）`如果传递了模式匹配的正则表达式。

    匹配的默认算法是奠定在6规范第21.2.5.6（[HTTP：/ / www.ecma-international。org / ECMA-262 / 6 / #秒regexp匹配。原型- @ @](http://www.ecma-international.org/ecma-262/6.0/#sec-regexp.prototype-@@match)）。你可以重写此默认算法和正则表达式提供额外的功能，如看后面的断言。

    symbol.match`也被`isregexp`抽象操作（参见第6章中“字符串检查函数”中的注释），以确定对象是否用作正则表达式。要强制该检查失败，因此它不作为正则表达式处理，请设置`symbol.match`价值`假`（或者falsy）。`@ @取代

-   `：的`symbol.replace`正则表达式的值是所使用的方法。`原型。替换（…）`在字符串中替换与给定的正则表达式模式匹配的字符序列的一个或全部出现。`更换默认算法是奠定在6规范第21.2.5.8（

    HTTP：/ / www.ecma-international。org / ECMA-262 / 6 / #秒@代替原型的regexp。[）。](http://www.ecma-international.org/ecma-262/6.0/#sec-regexp.prototype-@@replace)重写默认算法的一个很酷的用法是提供额外的

    替代品`参数选项，如支持`“芭蕉”取代（/ / g，\[1,2,3]）`生产`“1b2c3”`吃个逐次替换值。`@ @搜索

-   `：的`symbol.search`正则表达式的值是所使用的方法。`原型。搜索（..）`在给定的正则表达式匹配的情况下搜索另一个字符串中的子字符串。`默认的搜索算法是奠定在6规范第21.2.5.9（

    HTTP：/ / www.ecma-international。org / ECMA-262 / 6 / #秒regexp。原型- @ @搜索[）。](http://www.ecma-international.org/ecma-262/6.0/#sec-regexp.prototype-@@search)@ @分裂

-   `：的`symbol.split`正则表达式的值是所使用的方法。`原型。拆分（..）`在给定的正则表达式匹配的分隔符的位置将字符串分割成子字符串。`分裂的默认算法是奠定在6规范第21.2.5.11（

    HTTP：/ / www.ecma-international。org / ECMA-262 / 6 / #秒regexp。原型- @ @分裂[）。](http://www.ecma-international.org/ecma-262/6.0/#sec-regexp.prototype-@@split)重写内置正则表达式算法并不适用于心脏的微弱！js附带一个高度优化的正则表达式引擎，所以您自己的用户代码可能会慢很多。这种元编程既简洁又强大，但只在真正必要或有益的情况下使用。

symbol.isconcatspreadable

### `这个`

@ @ isconcatspreadable`符号可以定义为布尔属性（`symbol.isconcatspreadable`）对任何对象（如数组或其他迭代器）以指示它是否应该`摊开_如果传递给数组_concat（..）`。`考虑：

symbol.unscopables

```js
var a = [1,2,3],
	b = [4,5,6];

b[Symbol.isConcatSpreadable] = false;

[].concat( a, b );		// [1,2,3,[4,5,6]]
```

### `这个`

@ @ unscopables`符号可以定义为对象属性（`symbol.unscopables`）在任何对象上指示哪些属性可以且不能作为`具有`声明。`考虑：

一

```js
var o = { a:1, b:2, c:3 },
	a = 10, b = 20, c = 30;

o[Symbol.unscopables] = {
	a: false,
	b: true,
	c: false
};

with (o) {
	console.log( a, b, c );		// 1 20 3
}
```

真正的`在`@ @ unscopables`对象表示属性应该是`unscopable_从而从词法范围变量中过滤出来。_假`意味着被包含在词法范围变量中是可以的。`警告：

**这个**具有`声明是不完全`严格的`模式，因此应该考虑从语言的使用。不要用它。看到`范围和关闭_此系列的标题以获取更多信息。因为_具有`应避免`@ @ unscopables`符号也是无意义的。`代理

## 一个最明显的元编程的功能添加到6是

代理`特征。`代理是一种特殊的对象，您创建的对象是“包装”，或者位于另一个正常对象的前面。您可以注册特殊处理程序（又名

陷阱_）在代理对象上，当对代理执行各种操作时调用。这些处理程序有机会执行额外的逻辑除了_转发_对原始目标/包装对象的操作。_这类的一个例子

陷阱_可以在代理上定义的处理程序是_得到`拦截`\[得到]`操作——在尝试访问对象上的属性时执行。考虑：`我们声明一个

```js
var obj = { a: 1 },
	handlers = {
		get(target,key,context) {
			// note: target === obj,
			// context === pobj
			console.log( "accessing: ", key );
			return Reflect.get(
				target, key, context
			);
		}
	},
	pobj = new Proxy( obj, handlers );

obj.a;
// 1

pobj.a;
// accessing: a
// 1
```

得到（…）`作为命名方法的处理程序`处理程序_对象（第二个参数到_代理（..）`），它接收到`目标_对象（_obj`），这`钥匙_属性名称（_“一个”`）以及`自己`/接收方/代理（`pobj`）。`后

控制台（…）`跟踪语句，我们将操作向前推进到`obj`通过`反射，得到（…）`。我们将负责`反映`下一节中的API，但是请注意每个可用的代理陷阱都有相应的API`反映`同名函数。`这些映射是对称的。当执行相应的元编程任务时，代理处理程序各自截取。

这些映射是对称的。当执行各自的元编程任务时，代理处理程序各自截取，以及`反映`实用程序在对象上分别执行各自的元编程任务。每个代理处理程序都有一个自动调用相应的默认定义。`反映`效用。你几乎肯定会同时使用两者。`代理`和`反映`在串联。

下面是一个可以在代理上定义的处理程序列表_目标_对象/函数，以及它们被触发的方式：

-   `得到（…）`：通过`[得到]`在代理上访问属性。`反射，得到（…）`，`。`property operator, or`【..]`房地产商）
-   `设置（…）`：通过`[ ]`属性值设置在代理上（`反射（…）`，的`=`赋值操作符或解构分配如果目标对象属性）
-   `deleteproperty（..）`：通过`[删除]`属性将从代理中删除（`反映。deleteproperty（..）`或`删除`）
-   `应用（…）`（如果_目标_是一个函数）：通过`[ [调用] ]`，作为正常函数/方法调用代理。`反射，应用（…）`，`打电话（…）`，`应用（…）`，或`（..）`调用操作符）
-   `建造（…）`（如果_目标_是构造函数）：通过`[构造]`作为构造函数函数调用代理。`反射（构造）`或`新的`）
-   `getownpropertydescriptor（..）`：通过`getownproperty [ [ ] ]`从代理检索属性描述符（`对象。getownpropertydescriptor（..）`或`反映。getownpropertydescriptor（..）`）
-   `defineproperty（..）`：通过`defineownproperty [ [ ] ]`在代理上设置属性描述符（`对象。defineproperty（..）`或`反映。defineproperty（..）`）
-   `getprototypeof（..）`：通过`getprototypeof [ [ ] ]`，的`[原型]`检索代理的内容（`对象。getprototypeof（..）`，`反映。getprototypeof（..）`，`__proto__`，`对象# isprototypeof（..）`，或`运算符`）
-   `setprototypeof（..）`：通过`setprototypeof [ [ ] ]`，的`[原型]`代理的设置（`对象。setprototypeof（..）`，`反映。setprototypeof（..）`，或`__proto__`）
-   `preventextensions（..）`：通过`preventextensions [ [ ] ]`代理是不可扩展的。`对象。preventextensions（..）`或`反映。preventextensions（..）`）
-   `扩展性（..）`：通过`[ [ ] ]扩展性`对代理的可扩展性进行了探讨。`对象。扩展性（..）`或`反映。扩展性（..）`）
-   `ownkeys（..）`：通过`ownpropertykeys [ [ ] ]`检索代理的拥有属性和/或拥有的符号属性集（`对象（键）`，`对象。getownpropertynames（..）`，`对象。getownsymbolproperties（..）`，`反映。ownkeys（..）`，或`JSON。stringify（..）`）
-   `列举（…）`：通过`[ [列举]`，一个迭代器要求代理的可枚举所有和“遗传”的特性（`思考，列举（…）`或`对..`）
-   `有（…）`：通过`hasproperty [ [ ] ]`对代理进行检查，以查看它是否拥有或继承的属性（`反射……（…）`，`对象# hasownproperty（..）`，或`“支柱”的目标`）

**提示：**有关这些元编程任务的更多信息，请参见`反映`API“本章后面的章节。

除了前一个列表中有关触发各种陷阱的操作之外，有些陷阱是由另一个陷阱的默认动作间接触发的。例如:

```js
var handlers = {
		getOwnPropertyDescriptor(target,prop) {
			console.log(
				"getOwnPropertyDescriptor"
			);
			return Object.getOwnPropertyDescriptor(
				target, prop
			);
		},
		defineProperty(target,prop,desc){
			console.log( "defineProperty" );
			return Object.defineProperty(
				target, prop, desc
			);
		}
	},
	proxy = new Proxy( {}, handlers );

proxy.a = 2;
// getOwnPropertyDescriptor
// defineProperty
```

这个`getownpropertydescriptor（..）`和`defineproperty（..）`处理程序由默认值触发。`设置（…）`设置属性值时的处理程序步骤（不管是新添加还是更新）。如果你也定义你自己的`设置（…）`处理程序，您可以对相应的调用进行调用，也可以不调用`语境`（不`目标`！）这将触发这些代理陷阱。

### 代理的局限性

这些元编程处理程序捕获了一系列可以针对对象执行的基本操作。然而，有一些操作（至少还不能）截取。

例如，这些操作中没有一个是被捕获和转发的。`pobj`代理`obj`目标：

```js
var obj = { a:1, b:2 },
	handlers = { .. },
	pobj = new Proxy( obj, handlers );

typeof obj;
String( obj );
obj + "";
obj == pobj;
obj === pobj
```

也许在未来，在语言的底层基本操作这些将截取的，给我们更多的力量来扩展JavaScript本身。

**警告：**有一定的_不变性_——不能被覆盖的行为——适用于代理处理程序的使用。例如，从`扩展性（..）`处理程序总是被强制为`布尔`。这些不变量限制了您使用代理定制行为的一些能力，但它们只是为了防止您创建奇怪的、不寻常的（或不一致的）行为。这些不变量的条件是复杂的，所以我们不完全在这里讨论，但这篇文章。[HTTP：/ / www.2ality。COM / 2014 / 12 / 6代理HTML #不变量。](http://www.2ality.com/2014/12/es6-proxies.html#invariants)做一个很好的遮盖他们的工作。

### 可撤销的代理

正规的代理总圈闭为目标对象，而且无法修改后的创造--只要参考保持到代理，代理仍然是可能的。但是，有些情况下您希望创建一个代理，当您希望停止代理时，它可以被禁用。解决方案是创建一个_Rev_：

```js
var obj = { a: 1 },
	handlers = {
		get(target,key,context) {
			// note: target === obj,
			// context === pobj
			console.log( "accessing: ", key );
			return target[key];
		}
	},
	{ proxy: pobj, revoke: prevoke } =
		Proxy.revocable( obj, handlers );

pobj.a;
// accessing: a
// 1

// later:
prevoke();

pobj.a;
// TypeError
```

可以创建一个可撤销的代理`代理，可撤销（..）`，这是一个正则函数，而不是构造函数。`代理（..）`。否则，它将使用相同的两个参数：_目标_和_处理程序_。

返回值`代理，可撤销（..）`不是代理本身`新代理（..）`。相反，它是一个具有两个属性的对象：_代理_和_撤销_我们使用对象解构（见“解构”2章）来指定这些属性`pobj`和`prevoke()`变量分别。

一旦一个可撤销的代理被撤销，任何试图访问它的尝试（触发它的任何陷阱）都会抛出一个`TypeError`。

使用可撤销代理的一个示例可能是在应用程序中向另一方分发代理，该应用程序管理模型中的数据，而不是给它们引用真正的模型对象本身。如果您的模型对象更改或被替换，您希望使您分发的代理无效，以便另一方知道（通过错误！）请求对模型的更新引用。

### 使用代理服务器

这些代理处理程序的元编程好处应该是显而易见的。我们几乎可以完全拦截（并因此覆盖）对象的行为，这意味着我们可以以非常强大的方式扩展核心JS以外的对象行为。我们将看几个示例模式来探索这些可能性。

#### 代理第一，代理最后

正如我们前面提到的，您通常认为代理是“包装”目标对象。从这个意义上说，代理成为代码与之接口的主要对象，而实际的目标对象仍然是隐藏的/受保护的。

您可能会这样做，因为您希望将对象传递给不能完全信任的对象，因此需要在访问附近强制执行特殊规则，而不是传递对象本身。

考虑：

```js
var messages = [],
	handlers = {
		get(target,key) {
			// string value?
			if (typeof target[key] == "string") {
				// filter out punctuation
				return target[key]
					.replace( /[^\w]/g, "" );
			}

			// pass everything else through
			return target[key];
		},
		set(target,key,val) {
			// only set unique strings, lowercased
			if (typeof val == "string") {
				val = val.toLowerCase();
				if (target.indexOf( val ) == -1) {
					target.push(val);
				}
			}
			return true;
		}
	},
	messages_proxy =
		new Proxy( messages, handlers );

// elsewhere:
messages_proxy.push(
	"heLLo...", 42, "wOrlD!!", "WoRld!!"
);

messages_proxy.forEach( function(val){
	console.log(val);
} );
// hello world

messages.forEach( function(val){
	console.log(val);
} );
// hello... world!!
```

我把这称为_代理第一_设计时，我们首先与代理进行交互（主要是完全）。

我们执行一些特殊的规则`messages_proxy`那不是强制执行的`信息`本身。我们只添加元素，如果值是字符串，也是唯一的；我们也小写值。当从中检索值时`messages_proxy`我们过滤掉字符串中的任何标点符号。

或者，我们可以完全反转这个模式，在这个模式中，目标与代理交互，而不是与目标交互的代理。因此，代码实际上只与主对象交互。实现此回退的最简单方法是在`[原型]`主要对象链。

考虑：

```js
var handlers = {
		get(target,key,context) {
			return function() {
				context.speak(key + "!");
			};
		}
	},
	catchall = new Proxy( {}, handlers ),
	greeter = {
		speak(who = "someone") {
			console.log( "hello", who );
		}
	};

// setup `greeter` to fall back to `catchall`
Object.setPrototypeOf( greeter, catchall );

greeter.speak();				// hello someone
greeter.speak( "world" );		// hello world

greeter.everyone();				// hello everyone!
```

我们直接互动`迎宾员`而不是`包罗万象的`。我们打电话的时候`说话（..）`它在…上被发现`迎宾员`直接使用。但是当我们尝试访问一种方法时`everyone()`这个函数不存在于`迎宾员`。

默认的对象属性行为是检查`[原型]`链（见）_对象原型_本系列的标题），所以`包罗万象的`征求意见`每个人`财产。代理`get()`然后处理程序启动并返回调用的函数。`说话（..）`具有正在访问的属性的名称（`“每个人”`）。

我称这种模式为_代理上_，因为代理只被用作最后手段。

#### “没有这样的财产/方法”

关于JS的一个常见的抱怨是，在试图访问或设置不存在的属性的情况下，对象默认不是非常防御性的。你可以预先定义的所有属性和方法的对象，并有一个错误，如果一个不存在的属性名称，随后抛出。

我们可以用代理来完成这个任务。_代理第一_或_代理上_设计。让我们考虑两者。

```js
var obj = {
		a: 1,
		foo() {
			console.log( "a:", this.a );
		}
	},
	handlers = {
		get(target,key,context) {
			if (Reflect.has( target, key )) {
				return Reflect.get(
					target, key, context
				);
			}
			else {
				throw "No such property/method!";
			}
		},
		set(target,key,val,context) {
			if (Reflect.has( target, key )) {
				return Reflect.set(
					target, key, val, context
				);
			}
			else {
				throw "No such property/method!";
			}
		}
	},
	pobj = new Proxy( obj, handlers );

pobj.a = 3;
pobj.foo();			// a: 3

pobj.b = 4;			// Error: No such property/method!
pobj.bar();			// Error: No such property/method!
```

两`得到（…）`和`设置（…）`如果目标对象的属性已经存在，我们只转发该操作；否则抛出错误。代理对象（`pobj`主要对象代码应该与它交互，因为它拦截这些动作来提供保护。

现在，让我们考虑用_代理上_设计：

```js
var handlers = {
		get() {
			throw "No such property/method!";
		},
		set() {
			throw "No such property/method!";
		}
	},
	pobj = new Proxy( {}, handlers ),
	obj = {
		a: 1,
		foo() {
			console.log( "a:", this.a );
		}
	};

// setup `obj` to fall back to `pobj`
Object.setPrototypeOf( obj, pobj );

obj.a = 3;
obj.foo();			// a: 3

obj.b = 4;			// Error: No such property/method!
obj.bar();			// Error: No such property/method!
```

这个_代理上_这里的设计对于处理程序是如何定义的比较简单。而不是需要拦截`[得到]`和`[ ]`操作，只有在目标属性存在时才转发它们，而不是依赖于一个事实，即如果`[得到]`或`[ ]`到了我们`pobj`回退，动作已经遍历整个。`[原型]`链并没有找到匹配的属性。在这一点上我们是自由的，无条件地抛出错误。很酷，是吧？

#### 代理黑客`[原型]`链

这个`[得到]`操作是主要的渠道。`[原型]`机制被调用。当在直接对象上找不到属性时，`[得到]`自动关闭操作到`[原型]`对象。

这意味着您可以使用`得到（…）`代理的陷阱来模仿或扩展这个概念`[原型]`机制.

我们首先考虑的是创建两个通过循环链接的对象。`[原型]`（或者，至少看起来是这样的！）实际上你不能创建一个真正的循环。`[原型]`链条，因为发动机会抛出一个

考虑：

```js
var handlers = {
		get(target,key,context) {
			if (Reflect.has( target, key )) {
				return Reflect.get(
					target, key, context
				);
			}
			// fake circular `[[Prototype]]`
			else {
				return Reflect.get(
					target[
						Symbol.for( "[[Prototype]]" )
					],
					key,
					context
				);
			}
		}
	},
	obj1 = new Proxy(
		{
			name: "obj-1",
			foo() {
				console.log( "foo:", this.name );
			}
		},
		handlers
	),
	obj2 = Object.assign(
		Object.create( obj1 ),
		{
			name: "obj-2",
			bar() {
				console.log( "bar:", this.name );
				this.foo();
			}
		}
	);

// fake circular `[[Prototype]]` link
obj1[ Symbol.for( "[[Prototype]]" ) ] = obj2;

obj1.bar();
// bar: obj-1 <-- through proxy faking [[Prototype]]
// foo: obj-1 <-- `this` context still preserved

obj2.foo();
// foo: obj-2 <-- through [[Prototype]]
```

**注：**我们不需要代理/转发。`[ ]`在这个例子中，我们使事情更简单。要充分`[原型]`仿真兼容，您想要实现一个`设置（…）`搜索程序的处理程序`[原型]`匹配属性的链，并尊重其描述符行为（例如，集合、可写）。看到_对象原型_本系列的标题。

在前一段代码中，`obj2`是`[原型]`联系`obj1`凭借`对象创建（..）`声明。但是要创建反向（圆形）链接，我们需要创建属性`obj1`在符号位置`符号。用于（[ [原型] ]）`（见第2章中的“符号”）。这个符号可能看起来有点特殊/神奇，但它不是。它只允许我一个方便命名的钩子，语义上与我所执行的任务相关。

然后，代理服务器的`得到（…）`处理程序先查看是否有请求。`钥匙`在代理上吗？。如果没有，操作将手动传递到存储在`符号。用于（[ [原型] ]）`位置`目标`。

这种模式的一个重要优点是`obj1`和`obj2`它们之间并没有建立这种循环关系。尽管前面的代码片段为简洁起见，所有的步骤都是相互交织的，但是如果仔细查看，代理处理程序逻辑是完全通用的（不知道`obj1`或`obj2`具体）。因此，这种逻辑可以被拉出到一个简单的助手，把它们连接起来，就像`setcircularprototypeof（..）`例如.我们将把它作为读者的练习。

现在我们已经知道如何使用`得到（…）`模拟`[原型]`链接，让我们把牛车远一点。代替循环`[原型]`那么多重呢？`[原型]`联系（又名“多重继承”）？这结果相当简单：

```js
var obj1 = {
		name: "obj-1",
		foo() {
			console.log( "obj1.foo:", this.name );
		},
	},
	obj2 = {
		name: "obj-2",
		foo() {
			console.log( "obj2.foo:", this.name );
		},
		bar() {
			console.log( "obj2.bar:", this.name );
		}
	},
	handlers = {
		get(target,key,context) {
			if (Reflect.has( target, key )) {
				return Reflect.get(
					target, key, context
				);
			}
			// fake multiple `[[Prototype]]`
			else {
				for (var P of target[
					Symbol.for( "[[Prototype]]" )
				]) {
					if (Reflect.has( P, key )) {
						return Reflect.get(
							P, key, context
						);
					}
				}
			}
		}
	},
	obj3 = new Proxy(
		{
			name: "obj-3",
			baz() {
				this.foo();
				this.bar();
			}
		},
		handlers
	);

// fake multiple `[[Prototype]]` links
obj3[ Symbol.for( "[[Prototype]]" ) ] = [
	obj1, obj2
];

obj3.baz();
// obj1.foo: obj-3
// obj2.bar: obj-3
```

**注：**正如前面的通告后面提到的那样`[原型]`例如，我们没有实现`设置（…）`处理程序，但是需要一个完整的解决方案来模拟`[ ]`行为是正常的`[原型]`的表现。

`OBJ3`设置为多个委托`obj1`和`obj2`。在`baz() OBJ3。`，的`这foo()。`呼叫结束`foo()`从`obj1`（先到先得，即使有）`foo()`在`obj2`）。如果我们重新排序的联系`obj2，以此`，的`foo() obj2。`会被发现和使用。

但事实是，`这bar()。`打电话找不到`bar()`在`obj1`所以它落在支票上`obj2`在那里找到一根火柴。

`obj1`和`obj2`代表两平行`[原型]`链`OBJ3`。`obj1`和/或`obj2`自己能正常吗？`[原型]`对其他对象的代理，或者可以自己是一个代理（如`OBJ3`）可以是多个委托。

就像圆形一样`[原型]`前面的例子`obj1`，`obj2`，和`OBJ3`几乎完全不同于处理多个委托的通用代理逻辑。定义一个实用程序是很简单的。`setprototypesof（..）`（注意“s”！）这需要一个主对象和一个对象列表来伪装多个对象。`[原型]`联动。再次，我们将把它作为一个练习给读者。

希望通过这些不同的例子，代理的力量现在变得越来越清晰。有许多其他强大的元编程任务，代理启用。

## `反映`美国石油学会

这个`反映`对象是一个普通对象（如`数学`，不像其他内置的本地人那样是一个函数/构造函数。

它拥有静态函数，与您可以控制的各种元编程任务相对应。这些函数与处理程序方法一一对应（_陷阱_代理可以定义。

有些函数看起来像是相同名称的函数。`对象`：

-   `反映。getownpropertydescriptor（..）`
-   `反映。defineproperty（..）`
-   `反映。getprototypeof（..）`
-   `反映。setprototypeof（..）`
-   `反映。preventextensions（..）`
-   `反映。扩展性（..）`

这些公用事业的行为一般与他们的相同。`*对象。`相对应的人.然而，一个不同之处在于`*对象。`对应者试图将第一个参数（目标对象）强制强制给对象，如果对象还没有的话。这个`*反映。`方法在这种情况下只抛出一个错误。

可以使用这些工具访问/检查对象的键：

-   `反映。ownkeys（..）`：返回所有拥有的键（不是“继承的”）列表，由两个键返回`对象。getownpropertynames（..）`和`对象。getownpropertysymbols（..）`。有关键的顺序的信息，请参见“属性枚举顺序”一节。
-   `思考，列举（…）`：返回一个迭代器，该迭代器生成所有非符号键（拥有和继承）_枚举_（见_对象原型_本系列的标题）。本质上，这组键与由a处理的键相同。`对..`环。有关键的顺序的信息，请参见“属性枚举顺序”一节。
-   `反射……（…）`基本上是一样的`在里面`歌剧`[原型]`链。例如,`反射。有（o，“富”）`基本上执行`O中的“富”`。

函数调用的构造函数的调用，可以手动执行，正常的语法分开（例如，`（..）`和`新的`）使用这些实用程序：

-   `反射，应用（…）`例如，`反映。申请（Foo，thisobj，[ 42，“酒吧”]）`调用`富（…）`功能`thisobj`作为其`这`并通过`四十二`和`“酒吧”`争论。
-   `反射（构造）`例如，`反射（构造，（42，“条”））`基本要求`新富（42，“条”）`。

可以使用这些实用工具手动执行对象属性访问、设置和删除操作：

-   `反射，得到（…）`例如，`反射。获取（o，“富”）`检索`o.foo`。
-   `反射（…）`例如，`反射集（O，“富”，42）`基本上执行`o.foo = 42`。
-   `反映。deleteproperty（..）`例如，`反映。deleteproperty（O，“foo”）`基本上执行`删除o.foo`。

元编程能力`反映`给你编程等价物来模拟各种句法特征，暴露以前隐藏的抽象操作。例如，您可以使用这些功能来扩展功能和API。_领域特定语言_（DSL）。

### 属性排序

前6，使用的顺序列出对象的键/属性的规范实施依赖未定义。一般来说，大多数引擎都把它们列在创建顺序中，尽管强烈鼓励开发人员不要依赖这种排序。

截至6，上市国有性质的订单现在定义（ES6规范，部分9.1.12）的`ownpropertykeys [ [ ] ]`算法，产生集体所有制性质（字符串或符号），不可数。此订单仅保证`反映。ownkeys（..）`（推而广之），`对象。getownpropertynames（..）`和`对象。getownpropertysymbols（..）`）。

顺序是：

1.  首先，以升序数字顺序枚举所有拥有整数索引的属性。
2.  接下来，按创建顺序枚举其余的字符串属性名称。
3.  最后，枚举创建顺序中拥有的符号属性。

考虑：

```js
var o = {};

o[Symbol("c")] = "yay";
o[2] = true;
o[1] = true;
o.b = "awesome";
o.a = "cool";

Reflect.ownKeys( o );				// [1,2,"b","a",Symbol(c)]
Object.getOwnPropertyNames( o );	// [1,2,"b","a"]
Object.getOwnPropertySymbols( o );	// [Symbol(c)]
```

另一方面，`[ [列举]`算法（ES6规范，部分9.1.11）只产生可枚举属性，从目标对象以及其`[原型]`链。两者都使用。`思考，列举（…）`和`对..`。可观察的顺序是依赖于实现的，而不是由规范控制的。

通过对比，`对象（键）`调用`ownpropertykeys [ [ ] ]`获取所有拥有密钥列表的算法。然而，它过滤掉非可枚举属性然后重新排序列表相匹配的遗产实施相关的行为，特别是`JSON。stringify（..）`和`对..`。因此，通过扩展排序_也_火柴，`思考，列举（…）`。

换言之，所有四种机制（`思考，列举（…）`，`对象（键）`，`对..`，和`JSON。stringify（..）`）将与相同的实现依赖排序相匹配，尽管它们在技术上有不同的实现方式。

实现允许将这四个匹配到`ownpropertykeys [ [ ] ]`但不要求。不过，您可能会从他们身上观察到以下排序行为：

```js
var o = { a: 1, b: 2 };
var p = Object.create( o );
p.c = 3;
p.d = 4;

for (var prop of Reflect.enumerate( p )) {
	console.log( prop );
}
// c d a b

for (var prop in p) {
	console.log( prop );
}
// c d a b

JSON.stringify( p );
// {"c":3,"d":4}

Object.keys( p );
// ["c","d"]
```

沸腾这一切归结：为6，`反映。ownkeys（..）`，`对象。getownpropertynames（..）`，和`对象。getownpropertysymbols（..）`所有产品都有可预测和可靠的订货。因此，构建依赖于这种排序的代码是安全的。

`思考，列举（…）`，`对象（键）`，和`对..`（以及`JSON。stringify（..）`通过扩展，继续像彼此一样共享一个可观察的排序。但这种排序不一定与`反映。ownkeys（..）`。仍然应该依赖依赖于实现的排序。

## 功能测试

什么是特征测试？这是一个测试，你运行，以确定是否有一个功能是否可用。有时，测试不仅仅是为了生存，而是为了符合指定的行为——特性可以存在，但却有缺陷。

这是一种元编程技术，用于测试程序运行的环境，然后确定程序应该如何运行。

功能测试的JS最常见的用途是检查一个API的存在，如果它是不存在的，定义一个polyfill（见1章）。例如:

```js
if (!Number.isNaN) {
	Number.isNaN = function(x) {
		return x !== x;
	};
}
```

这个`如果`这个片段中的语句是元编程：我们正在探索程序及其运行时环境，以确定是否应该以及如何进行。

但是如何测试涉及新语法的特性呢？

你可以试试类似的东西：

```js
try {
	a = () => {};
	ARROW_FUNCS_ENABLED = true;
}
catch (err) {
	ARROW_FUNCS_ENABLED = false;
}
```

不幸的是，这不起作用，因为我们的js程序是编译的。这样，发动机就要窒息了。`（）{ }`语法如果不是已经支持ES6箭头功能。程序中有语法错误阻止它运行，这会妨碍程序在支持或不支持功能时做出不同的响应。

元编程

你的头脑跳到使用了吗？`eval（..）`？

不这么快。看到_范围和关闭_本系列的标题为何`eval（..）`是个坏主意。但还有另一个选择较少的缺点：`函数（…）`构造函数。

考虑：

```js
try {
	new Function( "( () => {} )" );
	ARROW_FUNCS_ENABLED = true;
}
catch (err) {
	ARROW_FUNCS_ENABLED = false;
}
```

好了，现在我们通过确定像箭头函数这样的特性来进行元编程。_可以_在当前引擎中编译或不编译。你可能会想，我们该如何处理这些信息？

与API存在的检查，并确定polyfills回退API，有一个清晰的路径，该怎么处理试验成功或失败。但是我们能从我们得到的信息中得到什么呢？`arrow_funcs_enabled`存在`真正的`或`假`？

因为如果引擎不支持该特性，语法就不能出现在文件中，因此您不能在文件中定义不同的函数，而不带语法。

你可以做的是使用测试来确定你应该加载的一组js文件中的哪一个。例如，如果你在为你的js应用程序有一套这些功能测试，它可以测试环境来确定你的ES6代码可以直接加载运行，或者如果你需要加载transpiled版本的代码（见1章）。

这种技术叫做_分批交货_。

它承认现实，你写的程序会6 JS有时可以完全运行在“原生”6 +浏览器，但有时需要transpilation运行在pre-es6浏览器。如果你一直使用的transpiled代码，即使在新的6兼容的环境中，你至少有一些次优的代码运行时间。这不太理想。

拆分交付更为复杂和复杂，但它代表了一种更成熟、更健壮的方法来弥补您编写的代码和您的程序必须运行的浏览器之间的特性支持之间的差距。

### featuretests.io

定义所有6 +语法功能测试，以及语义的行为，是一项艰巨的任务，你可能不想自己处理。因为这些测试需要动态编译（`新功能（..）`）有一些糟糕的性能成本。

此外，每次运行应用程序时都运行这些测试可能是浪费的，因为用户的浏览器平均最多只能在几个星期内更新一次，即使如此，新的功能也不一定会出现在每次更新中。

最后，管理功能的测试，适用于您的特定代码库的列表，将你的程序很少使用ES6整体是不羁和容易出错的。

“<https://featuretests.io>“feature-tests-as-a-service提供这些挫折的解决方案。

您可以将服务库加载到页面中，并加载最新的测试定义并运行所有的功能测试。如果可能的话，它使用后台处理与Web工作者，以降低性能开销。它还使用localStorage的持久性的一种方式，可以在所有的网站你访问使用共享缓存的结果，从而大大降低了多久的测试需要运行在每个浏览器实例。

您可以在每个用户的浏览器中获得运行时功能测试，并且可以动态地使用这些测试结果为用户提供最合适的代码（不多也不少）。

此外，该服务还提供工具和API来扫描您的文件，以确定您需要什么功能，因此您可以完全自动化您的拆分交付构建过程。

FeatureTests.io使得它的实际使用功能测试的所有部分和超出6确保只有最好的代码被加载并运行在任何给定的环境。

## 尾调用优化（TCO）

通常，当函数调用是从另一个函数中进行时，第二个函数调用_栈帧_分配给单独管理其他函数调用的变量/状态。这种分配不仅花费了一些处理时间，而且占用了额外的内存。

调用堆栈链通常从一个函数到另一个函数至多有10-15次跳转。在这些场景中，内存使用不太可能是任何实际问题。

然而，当您考虑递归编程（一个调用它自己的函数）或两个或多个函数相互调用的递归时，调用堆栈可以很容易地达到几百个、数千个或更多个级别。如果内存使用变得无限，您可能会看到可能导致的问题。

JavaScript引擎必须设置任意限制，以防止这种编程技术因运行浏览器和设备内存不足而崩溃。这就是为什么我们得到了令人沮丧的“rangeerror：最大堆栈大小超过“如果被击中的限制抛。

**警告：**调用堆栈深度的限制不受规范控制。它依赖于实现，并且在浏览器和设备之间会有所不同。你永远不要用精确的假设来精确地观察极限，因为它们很可能从发布到发布。

调用函数的某些模式，称为_尾调用_可以以某种方式优化，以避免堆栈帧的额外分配。如果可以避免额外的分配，就没有理由任意限制调用堆栈深度，因此引擎可以让它们运行无界。

尾调用是`返回`具有函数调用的语句，在调用之后不发生任何事情，除非返回其值。

这种优化只能应用于`严格的`模式。还有另一个原因，总是编写所有代码`严格的`！

这里是一个函数调用_不_尾部位置：

```js
"use strict";

function foo(x) {
	return x * 2;
}

function bar(x) {
	// not a tail call
	return 1 + foo( x );
}

bar( 10 );				// 21
```

`1 + ..`必须在`富（x）`呼叫完成，所以状态`酒吧（…）`需要保存调用。

但是下面的代码片段演示了调用`富（…）`和`酒吧（…）`在那里_是_在尾部位置，因为它们是代码路径中最不可能发生的事情（除了`返回`）：

```js
"use strict";

function foo(x) {
	return x * 2;
}

function bar(x) {
	x = x + 1;
	if (x > 10) {
		return foo( x );
	}
	else {
		return bar( x + 1 );
	}
}

bar( 5 );				// 24
bar( 15 );				// 32
```

在这个程序中，`酒吧（…）`显然是递归的，但`富（…）`只是一个常规函数调用。在这两种情况下，函数调用都在_适当的尾部位置_。这个`x + 1`前评估`酒吧（…）`调用，每当调用结束时，所有发生的都是`返回`。

这些表单的适当尾部调用（PTC）可以被优化——称为尾部调用优化（TCO），这样就不需要额外的堆栈帧分配。而不是创造未来的函数调用一个新的栈帧，发动机就重用现有的堆栈帧。这样做是因为一个函数不需要保存任何当前状态，因为PTC之后没有任何情况发生。

TCO意味着调用堆栈的深度几乎没有限制。这个技巧稍微改善了正常程序中的正则函数调用，但更重要的是打开了使用递归进行程序表达式的大门，即使调用堆栈可能是数以万计的调用深度。

我们不再被简化为解决问题的递归理论，而是可以在真正的JavaScript程序中使用递归！

截至6，所有PTC应该这样被优化，递归或不。

### 尾调用重写

然而，问题是，只有PTC可以优化，非PTC当然仍然工作，但会导致堆栈帧分配，因为他们总是这样做。如果你想优化的话，你必须小心使用PTC来构建你的函数。

如果你有一个没有用PTC写的函数，你可能需要手动重新排列你的代码以符合TCO的要求。

考虑：

```js
"use strict";

function foo(x) {
	if (x <= 1) return 1;
	return (x / 2) + foo( x - 1 );
}

foo( 123456 );			// RangeError
```

调用`foo（X-1）`不是PTC，因为它的结果必须添加到`（x／2）`之前`返回`惯性导航与制导.

然而，为了使代码符合TCO在6引擎，我们可以把它改写如下：

```js
"use strict";

var foo = (function(){
	function _foo(acc,x) {
		if (x <= 1) return acc;
		return _foo( (x / 2) + acc, x - 1 );
	}

	return function(x) {
		return _foo( 1, x );
	};
})();

foo( 123456 );			// 3810376848.5
```

如果你在6引擎实现TCO运行前面的代码，你会得到的`三十八亿一千零三十七万六千八百四十八点五`如图所示回答。然而，它仍然会失败`rangeerror`在非TCO引擎中。

### 非TCO优化

还有其他重写代码的技术，这样每次调用都不会增加调用堆栈。

一种这样的技术被称为_蹦床_这意味着每一个部分结果都表示为返回另一个部分结果函数或最终结果的函数。然后你可以简单地循环直到你停止得到一个函数，你就会得到结果。考虑：

```js
"use strict";

function trampoline( res ) {
	while (typeof res == "function") {
		res = res();
	}
	return res;
}

var foo = (function(){
	function _foo(acc,x) {
		if (x <= 1) return acc;
		return function partial(){
			return _foo( (x / 2) + acc, x - 1 );
		};
	}

	return function(x) {
		return trampoline( _foo( 1, x ) );
	};
})();

foo( 123456 );			// 3810376848.5
```

这个返工需要最小的修改来将递归分解为循环。`蹦床（…）`：

1.  首先，我们包装`返回_foo ..`线在`返回partial() { ..`函数表达式。
2.  然后我们包装`_foo（1，X）`打电话的`蹦床（…）`呼叫。

这种技术不受调用堆栈限制的原因是每个内部的限制。`部分（…）`函数返回到`虽然`环`蹦床（…）`运行它，然后再次循环。换言之，`部分（…）`不递归调用自己，它只返回另一个函数。栈的深度保持不变，所以它可以运行，只要它需要。

蹦床这样表达采用封闭的内`partial()`功能已超过`X`和`ACC`保持状态从迭代到迭代的变量。其优点是将循环逻辑拉出到可重用性中。`蹦床（…）`实用函数，许多库提供的版本。你可以重复使用`蹦床（…）`在不同的trampolined算法程序多次。

当然，如果您真的想深入优化（重用性并不是一个问题），您可以放弃闭包状态并内联状态跟踪。`ACC`只包含一个函数的范围和一个循环。这种技术通常被称为_递归展开_：

```js
"use strict";

function foo(x) {
	var acc = 1;
	while (x > 1) {
		acc = (x / 2) + acc;
		x = x - 1;
	}
	return acc;
}

foo( 123456 );			// 3810376848.5
```

这个算法的表达式更容易阅读，而且可能对我们探索过的各种形式（严格地说）执行得最好（严格地说）。这似乎是一个明显的赢家，你可能想知道为什么你会尝试其他方法。

有一些原因，你可能不想总是手动打开你的递归：

-   而分解出蹦床（环）逻辑的可重用性，我们内联它
-   这里的例子非常简单，足以说明不同的形式。实际上，递归算法中有很多复杂的问题，比如递归（不仅仅是一个函数调用）。

    你越往下走这个兔子洞，越是手工和复杂的。_展开_优化。很快就会失去可读性的所有价值。递归的主要优势，即使在PTC的形式，是它保留了算法的可读性，并会对发动机的性能优化。

如果你写下你的算法与PTC，6发动机将应用TCO来让你的代码运行在恒定的堆栈深度（利用堆栈帧）。您可以获得递归的可读性，且具有大部分性能优势，并且没有运行长度的限制。

### 元？

TCO与元编程有什么关系？

正如我们在前面的“功能测试”部分中所介绍的，您可以在运行时确定引擎支持的特性。这包括TCO，尽管确定它是蛮力的。考虑：

```js
"use strict";

try {
	(function foo(x){
		if (x < 5E5) return foo( x + 1 );
	})( 1 );

	TCO_ENABLED = true;
}
catch (err) {
	TCO_ENABLED = false;
}
```

在非TCO引擎中，递归循环最终会失败，抛出由异常捕获的异常。`试试。赶上`。否则，由于TCO，循环很容易完成。

哎呀，好吗？

但是围绕着TCO特性（或者说是它的不足）如何进行元编程对我们的代码有好处呢？答案很简单，你可以使用这样的功能测试来决定加载使用递归的应用程序的代码版本，或替代一个已经转换/ transpiled不需要递归。

#### 自调整代码

但这是另一种看待问题的方法：

```js
"use strict";

function foo(x) {
	function _foo() {
		if (x > 1) {
			acc = acc + (x / 2);
			x = x - 1;
			return _foo();
		}
	}

	var acc = 1;

	while (x > 1) {
		try {
			_foo();
		}
		catch (err) { }
	}

	return acc;
}

foo( 123456 );			// 3810376848.5
```

该算法通过试图做同样的工作与递归的可能，但跟踪进度，通过限制了作用域的变量`X`和`ACC`。如果整个问题可以用递归而不犯错误来解决，那就太好了。如果引擎在某个时刻杀死递归，我们只需用`试试。赶上`然后再尝试，拿起我们离开的地方。

我认为这是元编程的一种形式，在运行时您正在探索引擎完全（递归）完成任务的能力，并围绕可能限制您的任何（非TCO）引擎限制工作。

刚开始（甚至是第二次）！看，我打赌这是代码看起来非常讨厌的你相比，一些较早的版本。它也运行得稍微慢一点（在非TCO环境下的较大运行）。

主要的优势，它比其他能够即使在非TCO引擎完成任何规模的任务，这样的“解决方案”的递归栈的限制是更灵活的比蹦床或手动展开技术证明以前。

基本上,`_foo()`在这种情况下，实际上是任何递归任务中的一种，甚至是相互递归。其余的是样板，应该只是任何算法。

只有“抓”的是，能够在一个递归限制被打事件的简历，递归的状态必须在作用域的变量之外存在的递归函数（S）。我们这样做是为了离开`X`和`ACC`外面的`_foo()`函数，而不是将它们作为参数传递给`_foo()`作为较早。

几乎所有的递归算法都能适应这样的工作方式。这意味着这是在程序中递归使用TCO的最广泛的应用方式，只需极少的重写。

这种方法仍然使用PTC，这意味着该代码将_逐步提高_从运行使用多次循环（递归批次）在一个旧的浏览器充分利用TCO会递归的ES6 +环境。我觉得很酷！

## 回顾

元编程就是当你把程序的逻辑转向自己（或者它的运行时环境），或者检查它自己的结构或者修改它。元编程的主要价值是扩展语言的正常机制以提供附加功能。

前6，JavaScript已经有相当多的元编程能力，但6明显坡道出几个新的特点。

从匿名函数的函数名推断到提供调用构造函数的信息的元属性，您可以在程序运行时检查程序结构。众所周知的符号允许您重写内在行为，如将对象强制为原始值。代理可以拦截和定制对象上的各种低级操作，以及`反映`提供实用程序来模拟它们。

特性测试，即使是像尾调用优化这样的细微语义行为，也会将元编程焦点从程序转移到js引擎本身。通过更多地了解环境可以做什么，您的程序可以在运行时自行调整到最适合的位置。

你应该元程序吗？我的建议是：首先要着重学习语言的核心机制是如何工作的。但一旦你完全知道js本身能做什么，就应该开始利用这些P了。
