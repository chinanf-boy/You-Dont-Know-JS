
# 你不知道JS: 异步性能

# 第4章发电机

在2章中,我们确定了两个关键缺点表示异步流控制与回调: 

-   基于异步不适合我们的大脑如何计划出一个任务步骤回调ㄢ
-   回调不可信或组合的原因_控制反转_ㄢ

在3章中,我们详细介绍uninvert的承诺_控制反转_回调,恢复信任/组合ㄢ

现在我们将目光转向在顺序表达异步流控制ㄡ同步看时尚ㄢ"魔术ℽ,使得它可能是6**发电机**ㄢ

## 中断运行至完成

在第1章中,我们解释了js开发人员在代码中几乎普遍依赖的一个期望: 一旦函数开始执行,它就一直运行到它完成,并且没有其他代码可以中断和运行ㄢ

离奇好像6介绍了一种新型的功能,不做与运行完成的行为ㄢ这种新的函数称为"生成器ℽㄢ

为了理解其中的含义,我们来考虑这个例子: 

```js
var x = 1;

function foo() {
	x++;
	bar();				// <-- what about this line?
	console.log( "x:", x );
}

function bar() {
	x++;
}

foo();					// x: 3
```

在这个例子中,我们肯定知道`bar()`运行在之间`X + +`和`控制台(日志)`ㄢ但如果`bar()`不是吗?显然,结果会是`二`而不是`三`ㄢ

现在让我们扭曲你的大脑ㄢ如果ⅆ怎么办`bar()`不在场,但它仍能在两者之间运行ㄢ`X + +`和`控制台(日志)`报表?怎么可能呢?

在**先发制人**多线程语言,本质上是可能的`bar()`在两个语句之间"打断ℽ并在适当的时间运行ㄢ但js不是抢占式的,也不是当前的多线程ㄢ然而,A**合作的**这种"中断ℽ(并发)的形式是可能的,如果`foo()`它本身可以指示代码中那个部分的"暂停ℽㄢ

**注: **我用的词是"合作ℽ,不仅是因为经典的并发性术语的联系(见1章),但因为你会看到一段代码,用于指示一个停顿点6语法`产量`ℴℴ礼貌地暗示_合作的_控制屈服ㄢ

这里的6代码去完成这样的合作并发: 

```js
var x = 1;

function *foo() {
	x++;
	yield; // pause!
	console.log( "x:", x );
}

function bar() {
	x++;
}
```

**注: **您可能会看到大多数其他js文档/代码,它们将格式化生成器声明为`功能* foo() { ..}`而不是我在这里做的`功能* foo() { ..}`-唯一的区别是风格的定位`*`ㄢ这两种形式的功能/语法相同,为第三`功能* foo() { ..}`(无空格)窗体ㄢ这两种风格都有争论,但我基本上更喜欢ㄢ`函数*ⅆ`因为当我用一个生成器引用一个生成器时,它就匹配了ㄢ`* foo()`ㄢ如果我只说`foo()`如果我说的是发电机或正则函数,你不会很清楚ㄢ纯粹是文体上的偏爱ㄢ

现在,我们如何运行前面的代码段中的代码,以便`bar()`在`产量`里面的`* foo()`?

```js
// construct an iterator `it` to control the generator
var it = foo();

// start `foo()` here!
it.next();
x;						// 2
bar();
x;						// 3
it.next();				// x: 3
```

好吧,这两个代码段中有相当多的新的和潜在的混乱的东西,所以我们有大量的经历ㄢ但在我们解释不同的力学/语法与6发电机,让我们通过行为流走: 

1.  这个`它foo()`手术_不_执行`* foo()`生成器还没有,但它仅仅构造了一个_迭代器_那将控制它的执行ㄢ更多关于_迭代器_有点ㄢ
2.  第一`next()它ㄢ`开始`* foo()`生成器,并运行`X + +`在第一行`* foo()`ㄢ
3.  `* foo()`停留在`产量`语句,在第一个点`next()它ㄢ`通话结束ㄢ此刻,`* foo()`仍在运行并处于活动状态,但处于暂停状态ㄢ
4.  我们检查一下`X`现在是`二`ㄢ
5.  我们的电话`bar()`,其中增量`X`再次与`X + +`ㄢ
6.  我们检查一下`X`又一次,现在`三`ㄢ
7.  最后的`next()它ㄢ`叫简历`* foo()`从暂停的生成器,并运行`控制台(ⅆ)`语句,它使用当前值`X`属于`三`ㄢ

明确,`* foo()`开始了,但是_不_运行至完成ℴℴ它停在`产量`ㄢ我们重新开始`* foo()`稍后,让它结束,但这甚至不是必需的ㄢ

因此,发电机是一种特殊的功能,可以启动和停止一个或多个时间,并不一定要完成ㄢ虽然它不会很明显,但为什么是如此的强大,我们在本章的其余部分,这将是一个我们用来构建发电机异步流控制为我们的代码模式的基本构建块ㄢ

### 输入和输出

生成器函数是一种特殊的函数,我们刚才提到了新的处理模型ㄢ但它仍然是一个函数,这意味着它仍然有一些基本原则没有改变ℴℴ也就是说,它仍然接受参数(又名"输入ℽ),并且它仍然可以返回一个值(又名"输出ℽ): 

```js
function *foo(x,y) {
	return x * y;
}

var it = foo( 6, 7 );

var res = it.next();

res.value;		// 42
```

我们通过争论ㄢ`六`和`七`到`*富(..)`作为参数`X`和`Y`,分别ㄢ和`*富(..)`返回值`四十二`回到调用代码ㄢ

我们n`foo(6,7)`看起来很眼熟ㄢ但微妙的是`*富(..)`生成器实际上还没有运行,因为它有一个函数ㄢ

相反,我们只是创建一个_迭代器_对象,我们将其赋给变量`它`,以控制`*富(..)`发电机ㄢ然后我们打电话`next()它ㄢ`,指示`*富(..)`生成器从当前位置前进,在下一个位置停止ㄢ`产量`或发电机的端部ㄢ

结果`下一个(..)`调用是一个带有`价值`属性保留任何值(如果有的话)返回`*富(..)`ㄢ换言之,`产量`在执行过程中,从生成器中发出一个值,类似于中间值ㄢ`返回`ㄢ

再说一遍,这还不清楚,为什么我们需要整个间接的_迭代器_对象来控制生成器ㄢ我们会到达那里的,我_承诺_ㄢ

#### 迭代消息

除了接受参数和返回值的生成器之外,还有更强大ㄡ更强大的输入/输出消息传递功能,它们通过`产量`和`下一个(..)`ㄢ

考虑: 

```js
function *foo(x) {
	var y = x * (yield);
	return y;
}

var it = foo( 6 );

// start `foo(..)`
it.next();

var res = it.next( 7 );

res.value;		// 42
```

首先,我们通过`六`作为参数`X`ㄢ然后我们打电话`next()它ㄢ`它开始了`*富(..)`ㄢ

里面`*富(..)`,的`varⅆ`语句开始处理,但随后运行`产量`表达ㄢ这时,它停了下来ㄢ`*富(..)`(在赋值语句的中间!)并且本质上要求调用代码为`产量`表达ㄢ接下来,我们调用`下一个(7)ㄢ`, which is passing the`七`值回到_是_暂停的结果`产量`表达ㄢ

所以,在这一点上,赋值语句本质上是`var = 6 * 7`ㄢ现在,`回到Y`返回`四十二`作为结果返回值`下一个(7)ㄢ`呼叫ㄢ

注意一些非常重要但也容易混淆的问题,甚至对经验丰富的JS开发人员来说: 根据你的观点,两者之间存在不匹配ㄢ`产量`和`下一个(..)`呼叫ㄢ总的来说,你还会有一个`下一个(..)`打电话比你有`产量`语句ℴℴ前面的代码段有一个`产量`两`下一个(..)`电话.

为什么不匹配?

因为第一`下一个(..)`总是启动一个生成器,然后运行到第一个`产量`ㄢ但这是第二次`下一个(..)`完成第一个暂停的调用`产量`表达式和第三`下一个(..)`将完成第二`产量`等等ㄢ

##### 两个问题的故事

实际上,你主要考虑的代码会影响是否存在不匹配ㄢ

只考虑生成器代码: 

```js
var y = x * (yield);
return y;
```

这**第一** `产量`基本上是_问一个问题_"我应该在这里插入什么值?ℽ

谁来回答这个问题?嗯,这**第一** `next()`已经运行到发电机到这一点,所以显然_它_不能回答这个问题ㄢ因此,该**第二** `下一个(..)`打电话一定要回答问题ㄢ_合影_由**第一** `产量`ㄢ

看不匹配ℴℴ第二个到第一个?

但是,让我们改变我们的观点ㄢ让我们从生成器的角度来看它,而不是从迭代器的角度来看它ㄢ

为了正确地说明这一观点,我们还需要解释消息可以双向传播ℴℴ`产量ㄢ`作为表达式,可以发送消息以响应`下一个(..)`电话,和`下一个(..)`可以将值发送到暂停`产量`表达ㄢ考虑一下这个稍微调整过的代码: 

```js
function *foo(x) {
	var y = x * (yield "Hello");	// <-- yield a value!
	return y;
}

var it = foo( 6 );

var res = it.next();	// first `next()`, don't pass anything
res.value;				// "Hello"

res = it.next( 7 );		// pass `7` to waiting `yield`
res.value;				// 42
```

`产量ㄢ`和`下一个(..)`作为双向消息传递系统组合在一起**在执行发电机的过程中**ㄢ

所以,只看_迭代器_代码: 

```js
var res = it.next();	// first `next()`, don't pass anything
res.value;				// "Hello"

res = it.next( 7 );		// pass `7` to waiting `yield`
res.value;				// 42
```

**注: **我们不会将值传递给第一个ㄢ`next()`打电话,这是故意的ㄢ只有停下来`产量`可以接受由a传递的值ㄢ`下一个(..)`当我们调用第一个程序时,在生成器的开头`next()`,有**没有停下来`产量`**接受这样的价值ㄢ规范和所有兼容浏览器只是默默地**丢弃**任何东西传给第一个`next()`ㄢ传递一个值仍然是个坏主意,因为你正在创建一个令人困惑的无声的"失败ℽ代码ㄢ所以,始终启动一个无参数生成器ㄢ`next()`ㄢ

第一`next()`基本上没有调用_问一个问题_: "什么_下一个_价值呢`*富(..)`发电机得给我吗?ℽ谁来回答这个问题?第一`屈服"你好ℽ`表达ㄢ

看到了吗?不存在不匹配ㄢ

根据_谁_你想想问这个问题,要么两者之间不匹配`产量`和`下一个(..)`打电话,或不打电话ㄢ

但等待!还有一个额外的`next()`与数量相比`产量`声明.那么,最后`下一个(7)ㄢ`电话又问了一个关于什么的问题ㄢ_下一个_发电机将产生的值ㄢ但已经没有了`产量`剩下的回答,是吗?那么谁来回答呢?

这个`返回`陈述回答问题!

如果有**没有`返回`**在你的发电机里---`返回`生成器中肯定没有比正则函数更需要的了ℴℴ总是有假设的/隐式的ㄢ`返回;`(又名`返回未定义;`),它默认的回答问题的目的_合影_到最后`下一个(7)ㄢ`呼叫ㄢ

这些问题和答案ℴℴ传递的双向信息`产量`和`下一个(..)`--真的很强大,但不是很明显,在所有这些机制如何连接异步流控制ㄢ我们到那里了!

### 多个迭代器

从语法用法可以看出,当您使用_迭代器_为了控制生成器,您正在控制声明的生成器函数本身ㄢ但有一个微妙之处很容易忽略: 每次构建一个_迭代器_,您正在隐式地构造生成器的实例ㄢ_迭代器_将控制ㄢ

可以同时运行同一个生成器的多个实例,它们甚至可以交互: 

```js
function *foo() {
	var x = yield 2;
	z++;
	var y = yield (x * z);
	console.log( x, y, z );
}

var z = 1;

var it1 = foo();
var it2 = foo();

var val1 = it1.next().value;			// 2 <-- yield 2
var val2 = it2.next().value;			// 2 <-- yield 2

val1 = it1.next( val2 * 10 ).value;		// 40  <-- x:20,  z:2
val2 = it2.next( val1 * 5 ).value;		// 600 <-- x:200, z:3

it1.next( val2 / 2 );					// y:300
										// 20 300 3
it2.next( val1 / 4 );					// y:10
										// 200 10 3
```

**警告: **同时运行的同一个生成器的多个实例最常见的用法不是这种交互,而是当生成器在不输入的情况下生成自己的值时,可能是来自一些独立连接的资源ㄢ我们将在下一节中更多地讨论价值生产ㄢ

让我们简要地浏览一下处理过程: 

1.  两实例`* foo()`是同时启动的,而且两者都同时启动ㄢ`next()`来电显示`价值`属于`二`从`产量2`分别陈述ㄢ
2.  `临床* 10`是`2 * 10`,它被发送到第一个生成器实例ㄢ`IT1`,所以`X`获取价值`二十`ㄢ`Z`是递增的`一`到`二`,然后`20 * 2`是`产量`退出,设置`val1`到`四十`ㄢ
3.  `val1 * 5`是`40 * 5`,它被发送到第二个生成器实例ㄢ`IT2`,所以`X`获取价值`二百`ㄢ`Z`再次递增,从`二`到`三`,然后`200 * 3`是`产量`退出,设置`val2`到`六百`ㄢ
4.  `临床/ 2`是`600 / 2`,它被发送到第一个生成器实例ㄢ`IT1`,所以`Y`获取价值`三百`然后打印出来`20 300 3`其`XㄡYㄡZ`值分别ㄢ
5.  `val1 / 4`是`40 / 4`,它被发送到第二个生成器实例ㄢ`IT2`,所以`Y`获取价值`十`然后打印出来`200 10 3`其`XㄡYㄡZ`值分别ㄢ

这是一个"有趣ℽ的例子贯穿在你的脑海里ㄢ你把它弄直了吗?

#### 交织

从第1章的"运行到完成ℽ部分回忆这个场景: 

```js
var a = 1;
var b = 2;

function foo() {
	a++;
	b = b * a;
	a = b + 3;
}

function bar() {
	b--;
	a = 8 + b;
	b = a * 2;
}
```

当然,使用普通js函数`foo()`可以完全运行第一,或`bar()`可以完全运行第一,但`foo()`不能将其个别语句与`bar()`ㄢ因此,前面的程序只有两种可能的结果ㄢ

然而,与生成器,显然交织(即使在中间的语句!)是可能的: 

```js
var a = 1;
var b = 2;

function *foo() {
	a++;
	yield;
	b = b * a;
	a = (yield b) + 3;
}

function *bar() {
	b--;
	yield;
	a = (yield 8) + b;
	b = a * (yield 2);
}
```

取决于各自的顺序_迭代器_控制`* foo()`和`* bar()`被调用,前面的程序可以产生不同的结果ㄢ换言之,我们可以用第1章中讨论的理论"线程竞争条件ℽ来说明在相同的共享变量上交错两个生成器迭代的情况ㄢ

首先,让我们创建一个助手`步骤(..)`控制_迭代器_: 

```js
function step(gen) {
	var it = gen();
	var last;

	return function() {
		// whatever is `yield`ed out, just
		// send it right back in the next time!
		last = it.next( last ).value;
	};
}
```

`步骤(..)`初始化生成器以创建其`它` _迭代器_然后返回一个函数,当调用时,函数将_迭代器_一步一步ㄢ此外,以前`产量`将输出值直接发送到_下一个_步ㄢ所以,`产量8`只会成为`八`和`产量B`只会`B`(无论是什么时候`产量`)ㄢ

现在,为了好玩,让我们来试验一下交错这些不同块的效果ㄢ`* foo()`和`* bar()`ㄢ我们先从这个无聊的基础案例开始,确定一下ㄢ`* foo()`完全结束之前`* bar()`(就像我们在第1章中所做的那样): 

```js
// make sure to reset `a` and `b`
a = 1;
b = 2;

var s1 = step( foo );
var s2 = step( bar );

// run `*foo()` completely first
s1();
s1();
s1();

// now run `*bar()`
s2();
s2();
s2();
s2();

console.log( a, b );	// 11 22
```

最终结果是`十一`和`二十二`就像第1章的版本一样ㄢ现在让我们混合交错排序,看看它是如何改变最终值的ㄢ`一`和`B`: 

```js
// make sure to reset `a` and `b`
a = 1;
b = 2;

var s1 = step( foo );
var s2 = step( bar );

s2();		// b--;
s2();		// yield 8
s1();		// a++;
s2();		// a = 8 + b;
			// yield 2
s1();		// b = b * a;
			// yield b
s1();		// a = b + 3;
s2();		// b = a * 2;
```

在我告诉你结果之前,你能想出什么吗?`一`和`B`是在前面的程序之后吗?没有欺骗!

```js
console.log( a, b );	// 12 18
```

**注: **作为读者的练习,试着看看有多少其他的结果组合可以让你重新排列`s1()`和`s2()`电话.别忘了你总是需要三个ㄢ`s1()`电话和四`s2()`电话.回想前面关于匹配的讨论`next()`具有`产量`由于原因ㄢ

你几乎肯定不会想刻意创造_这_交错混乱的水平,因为它会产生难以理解的代码ㄢ但是这个练习对于理解多个生成器如何在同一个共享范围内同时运行是非常有趣和有启发性的,因为会有一些功能非常有用的地方ㄢ

我们将在本章末更详细地讨论生成器并发性ㄢ

## generator'ing值

在上一节中,我们提到了生成器的一个有趣用法,它是生成值的一种方法ㄢ这是**不**本章的主要焦点,但我们如果我们不涉及基础是失职,特别是因为这个使用案例基本上是这个名字的由来: 发电机ㄢ

我们将稍微考虑一下这个话题ㄢ_迭代器_有一点,但我们会回到何处_生成_价值观ㄢ

### 生产者和迭代器

假设您正在生成一系列值,其中每个值与前面的值有一个可定义的关系ㄢ要做到这一点,您将需要一个有状态的生产者,它记得它给出的最后一个值ㄢ

你可以实现的东西一样,直接使用函数闭包(见_范围和关闭_本系列的标题): 

```js
var gimmeSomething = (function(){
	var nextVal;

	return function(){
		if (nextVal === undefined) {
			nextVal = 1;
		}
		else {
			nextVal = (3 * nextVal) + 6;
		}

		return nextVal;
	};
})();

gimmeSomething();		// 1
gimmeSomething();		// 9
gimmeSomething();		// 33
gimmeSomething();		// 105
```

**注: **这个`nextval`这里的计算逻辑可以简化,但从概念上讲,我们不想计算_下一个值_(又名`nextval`)直到_下一个_ `gimmesomething()`调用发生了,因为一般来说,对于资源更持久或资源有限的生产者来说,这可能是一种资源泄漏的设计,而不是简单的ㄢ`数`S.

生成任意数字序列不是一个非常现实的例子ㄢ但是,如果您从数据源生成记录呢?你可以想象很多相同的代码ㄢ

事实上,这个任务是一种非常常见的设计模式,通常由迭代器解决ㄢ一个_迭代器_是一个定义良好的接口,用于从生产者中逐步跟踪一系列值ㄢ在大多数语言中,迭代器的js接口是调用`next()`每次你想要生产者的下一个值时ㄢ

我们可以实现标准_迭代器_接口为我们的数字系列生产者: 

```js
var something = (function(){
	var nextVal;

	return {
		// needed for `for..of` loops
		[Symbol.iterator]: function(){ return this; },

		// standard iterator interface method
		next: function(){
			if (nextVal === undefined) {
				nextVal = 1;
			}
			else {
				nextVal = (3 * nextVal) + 6;
			}

			return { done:false, value:nextVal };
		}
	};
})();

something.next().value;		// 1
something.next().value;		// 9
something.next().value;		// 33
something.next().value;		// 105
```

**注: **我们将解释为什么我们需要`[符号迭代器]: ⅆ`这部分代码段中的"可迭代对象ℽ部分ㄢ语法虽然两ES6功能在起作用ㄢ首先,该`㄰..]`语法称为_计算属性名称_(见_对象原型_本系列的标题)ㄢ在对象字面定义中指定表达式并使用该表达式的结果作为属性的名称是一种方法ㄢ下一个,`symbol.iterator`是一种特殊的预定义ES6`符号`值(参见_6与超越_这本书系列的标题)ㄢ

这个`next()`调用返回具有两个属性的对象: `完成`是一个`布尔`价值信号_迭代器的_完整的状态;`价值`保存迭代值ㄢ

6还增加了`对..`循环,这意味着一个标准_迭代器_可以自动使用本机循环语法: 

```js
for (var v of something) {
	console.log( v );

	// don't let the loop run forever!
	if (v > 500) {
		break;
	}
}
// 1 9 33 105 321 969
```

**注: **因为我们的`某物` _迭代器_总是返回`做: 假`,这`对..`循环将永远运行,这就是为什么我们把`打破`有条件ㄢ迭代器完全是无止境的,但也有这样的情况ㄢ_迭代器_将在有限的值集合上运行,最终返回一个`做: 真的`ㄢ

这个`对..`回路自动调用`next()`对于每次迭代,它不将任何值传递到`next()`ℴℴ它会在接收一个消息时自动终止ㄢ`做: 真的`ㄢ在一组数据上循环非常方便ㄢ

当然,您可以手动遍历迭代器,调用`next()`并检查`做: 真的`知道何时停止的条件: 

```js
for (
	var ret;
	(ret = something.next()) && !ret.done;
) {
	console.log( ret.value );

	// don't let the loop run forever!
	if (ret.value > 500) {
		break;
	}
}
// 1 9 33 105 321 969
```

**注: **本手册`对于`方法当然比6丑`对..`循环语法,但它的优点是它为您提供了将值传递给`下一个(..)`必要时调用ㄢ

除了自己制作_迭代器_,JS许多内置的数据结构(如6),如`阵列`s,也有默认值_迭代器_: 

```js
var a = [1,3,5,7,9];

for (var v of a) {
	console.log( v );
}
// 1 3 5 7 9
```

这个`对..`环问`一`其_迭代器_并自动使用它迭代`一`价值观ㄢ

**注: **它可能看起来奇怪的ES6遗漏,但定期`对象`s故意不带违约_迭代器_的方式`阵列`做ㄢ原因比我们在这里要深入的多ㄢ如果您想要的只是遍历对象的属性(没有特定的排序保证),`对象(键)`返回一个`阵列`然后可以像`对(物体ㄢVAR K键(obj)){ ..`ㄢ这样的一个`对..`对对象键的循环类似于`对..`循环,除了`对象(键)`不包含来自`[原型]`链`对..`(见)_对象原型_本系列的标题)ㄢ

### 可迭代对象

这个`某物`我们运行示例中的对象称为_迭代器_因为它有`next()`界面法ㄢ但是一个密切相关的术语是_可迭代的_,这是一个`对象`那个**包含**一个_迭代器_可以迭代它的值ㄢ

截至6,如何检索_迭代器_从一个_可迭代的_是的_可迭代的_必须对它有一个功能,以名字的特殊符号价值6`symbol.iterator`ㄢ当调用这个函数时,它返回一个_迭代器_ㄢ虽然不是必需的,但通常每个调用都应该返回一个新的_迭代器_ㄢ

`一`在前一段代码中_可迭代的_ㄢ这个`对..`循环自动调用它`symbol.iterator`函数构造_迭代器_ㄢ但我们当然可以手动调用函数,并使用_迭代器_它返回: 

```js
var a = [1,3,5,7,9];

var it = a[Symbol.iterator]();

it.next().value;	// 1
it.next().value;	// 3
it.next().value;	// 5
..
```

在前面定义的代码清单中`某物`你可能注意到了这一行: 

```js
[Symbol.iterator]: function(){ return this; }
```

这一点令人困惑的代码使`某物`值ℴℴ接口`某物` _迭代器_还有一个_可迭代的_现在都是_可迭代的_和一个_迭代器_ㄢ然后`某物`到`对..`环: 

```js
for (var v of something) {
	..
}
```

这个`对..`环预计`某物`是一个_可迭代的_所以它寻找并调用它`symbol.iterator`功能ㄢ我们将函数定义为`返回此`所以它只是把自己还给自己,然后`对..`循环一点也不明智ㄢ

### 发电机的迭代器

让我们把注意力转回发电机,在_迭代器_ㄢ生成器可以被看作是一个值的生产者,我们每次都通过一个_迭代器_接口的`next()`电话.

所以,发电机本身并不是技术上的_可迭代的_尽管它非常相似ℴℴ当你执行生成器时,你得到一个_迭代器_回来: 

```js
function *foo(){ .. }

var it = foo();
```

我们可以实现`某物`从一个发电机,像这样的早期无穷级数生成器: 

```js
function *something() {
	var nextVal;

	while (true) {
		if (nextVal === undefined) {
			nextVal = 1;
		}
		else {
			nextVal = (3 * nextVal) + 6;
		}

		yield nextVal;
	}
}
```

**注: **一`而实现的ㄢ`循环通常是一个非常坏的事情,包括在一个真正的js程序,至少如果它没有一个`打破`或`返回`在它,因为它可能会永远运行,同步,并阻止/锁定浏览器的用户界面ㄢ然而,在一个发电机中,这样一个回路一般是完全可以的,如果它有一个`产量`在这个过程中,生成器在每次迭代时都会暂停,`产量`回到主程序和/或事件循环队列ㄢ把它满口,"发电机把`而实现的ㄢ`回到js编程中!ℽ

这样稍微干净一点,对吧?因为发电机每停一次ㄢ`产量`函数的状态(范围)`* something()`在,意味着没有必要关闭样板保存状态变量在调用ㄢ

它不仅是简单的代码,我们也不必自己做代码ㄢ_迭代器_接口ℴℴ实际上是更合理的代码,因为它更明确地表达了意图ㄢ例如,`而实现的ㄢ`循环告诉我们生成器是为了永远运行ℴℴ保持_生成_价值观,只要我们不断要求ㄢ

现在我们可以用我们闪亮的新`* something()`发电机可与`对..`循环,你会看到它基本上是相同的: 

```js
for (var v of something()) {
	console.log( v );

	// don't let the loop run forever!
	if (v > 500) {
		break;
	}
}
// 1 9 33 105 321 969
```

但不要跳过`对于(var v something())..`!我们不仅仅是参考`某物`作为一个值,如前面的示例中所示,而是调用`* something()`生成器获取其_迭代器_对于`对..`循环使用ㄢ

如果你密切关注,发电机和回路之间的相互作用可能会产生两个问题: 

-   为什么我们不能说`对于(某物的var)ⅆ`?因为`某物`这是一台发电机,不是_可迭代的_ㄢ我们必须打电话`something()`为`对..`循环迭代ㄢ
-   这个`something()`调用产生一个_迭代器_,但`对..`要一环_可迭代的_,对吗?是的.发电机_迭代器_也有一个`symbol.iterator`它上的功能,基本上做一个`返回此`就像`某物` _可迭代的_我们早就定义了ㄢ换句话说,发电机的_迭代器_又是一个_可迭代的_!

#### 停发电机

在前一个示例中,它将显示_迭代器_实例的`* something()`发电机基本上是处于悬浮状态后永远在`打破`在循环中被称为ㄢ

但是有一种隐藏的行为会帮你解决这个问题ㄢℽ"异常完成ℽ(即"提前终止ℽ)`对..`循环--通常由A引起`打破`,`返回`,或未捕获的异常,发出一个信号发生器_迭代器_让它终止ㄢ

**注: **技术上,该`对..`循环也将此信号发送到_迭代器_在正常完成循环时ㄢ对于发电机来说,这基本上是一个模拟的操作,就像发电机一样ㄢ_迭代器_必须先完成,所以`对..`循环完成ㄢ然而,自定义_迭代器_可能希望收到这个额外的信号`对..`环的消费者ㄢ

而`对..`循环将自动发送此信号,您可能希望手动发送信号到_迭代器_这是通过调用来实现的`return(..)`ㄢ

如果指定一个`最后尝试..`生成器中的子句,即使在生成器外部完成时,它也将始终运行ㄢ这是有用的,如果你需要清理资源(数据库连接等): 

```js
function *something() {
	try {
		var nextVal;

		while (true) {
			if (nextVal === undefined) {
				nextVal = 1;
			}
			else {
				nextVal = (3 * nextVal) + 6;
			}

			yield nextVal;
		}
	}
	// cleanup clause
	finally {
		console.log( "cleaning up!" );
	}
}
```

前面的例子`打破`在`对..`循环将触发`最后`条款.但您可以手动终止生成器的_迭代器_实例来自外部`返回(..)`: 

```js
var it = something();
for (var v of it) {
	console.log( v );

	// don't let the loop run forever!
	if (v > 500) {
		console.log(
			// complete the generator's iterator
			it.return( "Hello World" ).value
		);
		// no `break` needed here
	}
}
// 1 9 33 105 321 969
// cleaning up!
// Hello World
```

我们打电话的时候`它ㄢ返回(..)`它立即终止生成器,当然,它运行`最后`条款.而且,它设置返回的值ㄢ`价值`无论你走到什么地方`return(..)`这是怎么回事`"你好,世界ℽ`马上回来ㄢ我们也不需要包含一个`打破`现在因为发电机_迭代器_是集`做: 真的`,所以`对..`循环将在下一次迭代中终止ㄢ

发电机大多与此同名ㄢ_消费价值观_使用ㄢ但是,这只是发电机的用途之一,坦率地说,甚至不是我们在本书中所关心的主要问题ㄢ

但是现在我们更充分地理解了它们的工作原理,我们可以_下一个_把我们的注意

## 发电机异步迭代

没有什么发电机异步编码模式,固定的问题而回调,等等?让我们来回答那个重要的问题ㄢ

我们应该从第3章重新审视我们的场景ㄢ让我们回忆一下回调方法: 

```js
function foo(x,y,cb) {
	ajax(
		"http://some.url.1/?x=" + x + "&y=" + y,
		cb
	);
}

foo( 11, 31, function(err,text) {
	if (err) {
		console.error( err );
	}
	else {
		console.log( text );
	}
} );
```

如果我们想用生成器表达相同的任务流控制,我们可以做: 

```js
function foo(x,y) {
	ajax(
		"http://some.url.1/?x=" + x + "&y=" + y,
		function(err,data){
			if (err) {
				// throw an error into `*main()`
				it.throw( err );
			}
			else {
				// resume `*main()` with received `data`
				it.next( data );
			}
		}
	);
}

function *main() {
	try {
		var text = yield foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}

var it = main();

// start it all up!
it.next();
```

乍一看,这个片段比它前面的回调代码片段要长,而且可能更复杂一些ㄢ但不要让这种印象让你偏离轨道ㄢ生成器片段实际上是**许多的**更好的!但是我们有很多事情要解释ㄢ

首先,让我们看看代码的这一部分,这是最重要的部分: 

```js
var text = yield foo( 11, 31 );
console.log( text );
```

想一想代码是如何工作的ㄢ我们调用一个正常函数`富(ⅆ)`而且我们显然能找回`文本`从Ajax调用,即使它是异步的ㄢ

那怎么可能呢?如果您还记得第1章的开头,我们有几乎相同的代码: 

```js
var data = ajax( "..url 1.." );
console.log( data );
```

那个代码不起作用!你能找出区别吗?这是`产量`用于发电机ㄢ

这就是魔法!这就是允许我们出现阻塞ㄡ同步代码的原因,但它实际上并不能阻塞整个程序;它只在生成器本身中暂停/阻塞代码ㄢ

在`产量foo(11,31)`,第一`foo(11,31)`调用是不返回任何东西的(又名`未定义`)所以我们正在打电话请求数据,但实际上我们正在做ㄢ`产量未定义`ㄢ没关系,因为代码目前不依赖于`产量`做任何有趣的事情的价值ㄢ我们将在本章后面重新讨论这一点ㄢ

我们没有使用`产量`在这里传递消息的意义上,只有在流控制意义上才能暂停/阻止ㄢ事实上,它会有消息传递,但只能在一个方向上,在生成器恢复之后ㄢ

因此,发电机在`产量`,本质上问这个问题,"我应该返回什么值来赋值给变量?`文本`?ℽ谁来回答这个问题?

看看`富(ⅆ)`ㄢ如果Ajax请求成功,我们调用: 

```js
it.next( data );
```

这将用响应数据恢复生成器,这意味着我们暂停了ㄢ`产量`表达式直接接收该值,然后在重新启动生成器代码时,该值将被赋给本地变量ㄢ`文本`ㄢ

很酷吧?

退一步考虑一下其中的含义ㄢ我们在生成器内部有完全同步的代码(除了`产量`关键字本身,但隐藏在幕后,在`富(ⅆ)`操作可以异步完成ㄢ

**那是巨大的!**这就是我们前面提到的问题,回调不能够在连续的同步性表达了近乎完美的解决方案,同步的方式,我们的大脑能理解ㄢ

在本质上,我们提取了一个异步的实现细节,这样我们就可以同步/顺序原因对我们的流量控制: "让一个Ajax请求,当它完成打印出响应ㄢℽ当然,我们只是表达了对流量控制的两个步骤,但这一能力的延伸没有界限然而,让我们表达我们需要许多步骤ㄢ

**提示: **这是一个很重要的实现,回去读最后三段,让它下沉!

### 同步错误处理

但前面的生成器代码更适合于_产量_我们ㄢ让我们把注意力转向`试试ㄢ赶上`发电机内部: 

```js
try {
	var text = yield foo( 11, 31 );
	console.log( text );
}
catch (err) {
	console.error( err );
}
```

这是怎么工作的?这个`富(ⅆ)`调用是异步完成的,而不是`试试ㄢ赶上`未能捕获异步错误,正如我们在第3章中看到的那样?

我们已经看到了`产量`让赋值语句暂停等待`富(ⅆ)`完成,以便完成的响应可以分配给`文本`ㄢ令人敬畏的部分是`产量`暂停_也_允许发电机`抓住`一个错误ㄢ我们用前面的代码列表中的这一部分将错误抛出到生成器中: 

```js
if (err) {
	// throw an error into `*main()`
	it.throw( err );
}
```

这个`产量`-发电机的暂停性质意味着我们不仅可以同步查找ㄢ`返回`从异步调用的函数值,但我们也可以同步`抓住`从这些异步函数调用错误!

所以我们看到我们可以抛出错误_进入之内_发电机,但是扔错怎么办?_由于_发电机?和你预料的一模一样: 

```js
function *main() {
	var x = yield "Hello World";

	yield x.toLowerCase();	// cause an exception!
}

var it = main();

it.next().value;			// Hello World

try {
	it.next( 42 );
}
catch (err) {
	console.error( err );	// TypeError
}
```

当然,我们可以手动抛出一个错误`把..`而不是导致异常ㄢ

我们甚至可以`抓住`同样的错误,我们`投掷(ⅆ)`进入发电机,基本上给发电机一个机会来处理它,但如果不这样做,_迭代器_代码必须处理它: 

```js
function *main() {
	var x = yield "Hello World";

	// never gets here
	console.log( x );
}

var it = main();

it.next();

try {
	// will `*main()` handle this error? we'll see!
	it.throw( "Oops" );
}
catch (err) {
	// nope, didn't handle it!
	console.error( err );			// Oops
}
```

同步查找错误处理(通过`试试ㄢ赶上`)与异步代码是一个巨大的胜利,可读性和推理的能力ㄢ

## 发电机+承诺

在我们以前的讨论中,我们展示了如何可以迭代异步发电机,这是向前的顺序推理能力在回调意大利面乱糟糟的一大步ㄢ但是我们失去了一些非常重要的东西: 信任和承诺的可组合性(见3章)!

别担心,我们可以把它弄回来ㄢ

但如何?

回想第3章我们运行Ajax示例的基于承诺的方法: 

```js
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}

foo( 11, 31 )
.then(
	function(text){
		console.log( text );
	},
	function(err){
		console.error( err );
	}
);
```

在我们早期的用于运行Ajax示例的生成器代码中,`富(ⅆ)`返回什么(`未定义`以及我们的_迭代器_控制代码不在乎`产量`ED值ㄢ

但这里的承诺意识`富(ⅆ)`在Ajax调用之后返回一个承诺ㄢ这表明我们可以用`富(ⅆ)`然后`产量`它来自发电机,然后_迭代器_控制代码将收到这个承诺ㄢ

但应该怎样_迭代器_遵守诺言吗?

它应该侦听解决问题的承诺(实现或拒绝),然后要么以执行消息恢复生成器,要么以拒绝原因向生成器抛出错误ㄢ

让我重复一遍,因为它非常重要ㄢ获得承诺和发电机的最自然的方法是**到`产量`一个承诺**和承诺控制发电机的电线_迭代器_ㄢ

让我们试试看!首先,我们要把承诺意识到ㄢ`富(ⅆ)`和发电机一起`* main()`: 

```js
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}

function *main() {
	try {
		var text = yield foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}
```

在这一重构最强大的启示是,里面的代码`* main()` **根本不需要改变!**在生成器内部,无论值是什么`产量`ED只是一个不透明的实现细节,所以我们甚至不知道它正在发生,我们也不需要担心它ㄢ

但是我们该怎么办呢?`* main()`现在呢?我们仍然有一些实现管道工作要做,接收和连接`产量`艾德答应让它在发电机上重新启动ㄢ我们将从手动开始: 

```js
var it = main();

var p = it.next().value;

// wait for the `p` promise to resolve
p.then(
	function(text){
		it.next( text );
	},
	function(err){
		it.throw( err );
	}
);
```

事实上,根本就没有那么痛苦,是吗?

这个代码段应该非常类似于我们先前使用错误第一回调控制的手动有线生成器ㄢ而不是一个`如果(错误){它抛出ⅆ`承诺已经为我们带来成功(成功)和拒绝(失败),但在另一方面_迭代器_控件相同ㄢ

现在,我们已经掩盖了一些重要的细节ㄢ

最重要的是,我们利用了我们知道的事实ㄢ`* main()`只有一个承诺知道它的步骤ㄢ如果我们希望能够承诺驱动一个发电机,不管它有多少步骤?我们当然不想为每个生成器手动写出承诺链!更好的是,如果有一种重复(又名"循环ℽ)迭代控制的方法,每次承诺出现时,在继续之前等待它的分辨率ㄢ

另外,如果生成器在进程中抛出错误(有意或无意),该怎么办?`下一个(..)`电话吗?我们应该退出,还是应该`抓住`然后把它寄回去?同样,如果我们`扔(ⅆ)`一个承诺拒绝进入发电机,但它没有处理,并立即回来?

### 有意识的发电机转轮

你越是开始探索这条路,你就越会意识到: "哇,如果有一些实用工具能帮我,那就太好了ㄢℽ你绝对正确ㄢ这是一个非常重要的模式,你不想让它出错(或者反复重复一遍),所以最好的办法是使用一个专门设计的实用程序ㄢ_运行_承诺ℴ`产量`按照我们所演示的方式生成发电机ㄢ

几个承诺抽象库提供了这样一个实用程序,包括我的_asynquence_图书馆及其`跑步者(ⅆ)`这将在本书附录A中讨论ㄢ

但是为了学习和说明,让我们定义我们自己调用的独立实用程序ㄢ`跑步(ⅆ)`: 

```js
// thanks to Benjamin Gruenbaum (@benjamingr on GitHub) for
// big improvements here!
function run(gen) {
	var args = [].slice.call( arguments, 1), it;

	// initialize the generator in the current context
	it = gen.apply( this, args );

	// return a promise for the generator completing
	return Promise.resolve()
		.then( function handleNext(value){
			// run to the next yielded value
			var next = it.next( value );

			return (function handleResult(next){
				// generator has completed running?
				if (next.done) {
					return next.value;
				}
				// otherwise keep going
				else {
					return Promise.resolve( next.value )
						.then(
							// resume the async loop on
							// success, sending the resolved
							// value back into the generator
							handleNext,

							// if `value` is a rejected
							// promise, propagate error back
							// into the generator for its own
							// error handling
							function handleErr(err) {
								return Promise.resolve(
									it.throw( err )
								)
								.then( handleResult );
							}
						);
				}
			})(next);
		} );
}
```

正如你所看到的,它比你想编写的要复杂得多,而且你特别不想重复你使用的每个生成器的代码ㄢ因此,一个实用程序/库助手绝对是该走的路了ㄢ不过,我鼓励您花几分钟的时间研究代码清单,以便更好地了解如何管理生成器+承诺协商ㄢ

你将如何使用`跑步(ⅆ)`具有`* main()`在我们的_运行_Ajax的例子吗?

```js
function *main() {
	// ..
}

run( main );
```

是这么回事!我们连线的方式`跑步(ⅆ)`它会自动将发电机传递给它,异步地完成ㄢ

**注: **这个`跑步(ⅆ)`我们定义的回报承诺这是有线一旦发电机完全解决,或收到一个未捕获的异常如果发电机不处理它ㄢ我们在这里没有显示这种能力,但我们将在本章后面再介绍它ㄢ

#### ES7: `异步`和`等待`?

前一种模式生成器产生的承诺,然后控制发电机的ㄢ_迭代器_将它推进到完成ℴℴ这是一个非常强大和有用的方法,如果没有图书馆实用工具的帮助,我们可以做得更好ㄢ`跑步(ⅆ)`)ㄢ

前线可能有好消息ㄢ在写这篇文章的时候,有一个早期但坚强的在这个领域的更多post-es6语法添加计划的支持,ES7 ISH的时间表ㄢ显然,现在保证细节还为时过早,但有一个相当不错的机会,它将摆脱类似如下: 

```js
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}

async function main() {
	try {
		var text = await foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}

main();
```

一`跑步(ⅆ)`调用(意味着不需要库实用程序!)调用和驱动`main()`-它被称为正常函数ㄢ也,`main()`不再被声明为生成器函数,它是一种新函数: `异步函数`ㄢ最后,而不是`产量`我们许下诺言`等待`为它解决ㄢ

这个`异步函数`自动知道怎么做如果你`等待`一个承诺-它会暂停功能(就像发电机),直到承诺解决为止ㄢ我们没有说明这段代码,但调用异步函数`main()`自动返回当函数完全完成时已解决的承诺ㄢ

**提示: **这个`异步`/`等待`语法应该看起来很熟悉的读者在C #经验,因为它基本上是相同的ㄢ

建议基本上是将我们已经得到的模式支持,为句法机制: 承诺与同步看流量控制码组合ㄢ这是两全其美的结合,有效地解决了几乎所有我们概述了回调的主要问题ㄢ

这样的ES7 ISH建议已经存在,事实早有支持和热情是在异步模式的未来信心的主要投票的重要性ㄢ

### 生成器中的并发承诺

到目前为止,我们已经证明是一个单步异步流承诺+发电机ㄢ但现实世界中的代码通常会有很多个步骤ㄢ

如果你不小心,同步发电机的外观风格可能让你陷于自满,你如何构造你的异步并发,导致次优的性能模式ㄢ所以我们想花点时间探索一下选项ㄢ

设想一个场景,您需要从两个不同的源获取数据,然后将这些响应组合为一个第三请求,最后打印出最后一个响应ㄢ我们用第3章中的承诺探索了一个类似的场景,但是让我们在生成器的上下文中重新考虑它ㄢ

你的第一直觉可能是类似的: 

```js
function *foo() {
	var r1 = yield request( "http://some.url.1" );
	var r2 = yield request( "http://some.url.2" );

	var r3 = yield request(
		"http://some.url.3/?v=" + r1 + "," + r2
	);

	console.log( r3 );
}

// use previously defined `run(..)` utility
run( foo );
```

这个代码会有效,但在我们的场景中,它不是最优的ㄢ你能找出原因吗?

因为`R1`和`R2`请求可以ℴℴ出于性能原因,_应该_-同时运行,但在这段代码中,它们将按顺序运行;`"2ℽ`URL是不是Ajax获取到`"1ℽ`请求完成ㄢ这两个请求是独立的,所以更好的性能方法可能是让它们同时运行ㄢ

但是你怎么能用发电机和`产量`?我们知道,`产量`只是代码中的一个停顿点,所以你不能同时做两个停顿ㄢ

最自然ㄡ最有效的回答是基于异步流的承诺,特别是他们的能力,在一次独立的方式管理国家(见"未来价值ℽ3章)ㄢ

最简单的方法: 

```js
function *foo() {
	// make both requests "in parallel"
	var p1 = request( "http://some.url.1" );
	var p2 = request( "http://some.url.2" );

	// wait until both promises resolve
	var r1 = yield p1;
	var r2 = yield p2;

	var r3 = yield request(
		"http://some.url.3/?v=" + r1 + "," + r2
	);

	console.log( r3 );
}

// use previously defined `run(..)` utility
run( foo );
```

为什么这与以前的片段不同?看哪里`产量`是与不是ㄢ`P1`和`P2`是Ajax请求并发的承诺(又名"并行ℽ)ㄢ先完成哪一个并不重要,因为承诺会在必要的时候保持其已解决的状态ㄢ

然后我们使用两个后续`产量`从承诺中等待和收回决议的语句(进入`R1`和`R2`分别)ㄢ如果`P1`首先解决`产量P1`先恢复后再等待`产量P2`恢复ㄢ如果`P2`首先解析,它会耐心地保持这个分辨率值直到被问到,但是`产量P1`将坚持第一,直到`P1`解决了ㄢ

无论哪种方式,两者都`P1`和`P2`将同时运行,并且必须在两个顺序之前完成`屈服请求ⅆ`将进行Ajax请求ㄢ

如果流程控制处理模型听起来很熟悉,它基本上与我们在第3章中所确定的"门ℽ模式相同,由`承诺(全部)])`效用ㄢ因此,我们也可以像这样表达流量控制: 

```js
function *foo() {
	// make both requests "in parallel," and
	// wait until both promises resolve
	var results = yield Promise.all( [
		request( "http://some.url.1" ),
		request( "http://some.url.2" )
	] );

	var r1 = results[0];
	var r2 = results[1];

	var r3 = yield request(
		"http://some.url.3/?v=" + r1 + "," + r2
	);

	console.log( r3 );
}

// use previously defined `run(..)` utility
run( foo );
```

**注: **正如我们在3章讨论的,我们甚至可以使用6解构分配简化`varvarⅆ`作业,与`var,R2 =结果`ㄢ

换句话说,承诺的所有并发功能都可以在生成器+承诺方法中提供给我们ㄢ所以在任何地方你需要比顺序然后,异步流的控制措施,将可能是你最好的选择ㄢ

#### 承诺,隐藏

作为一个风格谨慎的词,要小心你有多少承诺逻辑ㄢ**在你的发电机**ㄢ我们描述的使用异步生成器的全部目的是创建简单的ㄡ连续的ㄡ同步查找的代码,并尽可能多地隐藏异步性的细节ㄢ

例如,这可能是一种更清洁的方法: 

```js
// note: normal function, not generator
function bar(url1,url2) {
	return Promise.all( [
		request( url1 ),
		request( url2 )
	] );
}

function *foo() {
	// hide the Promise-based concurrency details
	// inside `bar(..)`
	var results = yield bar(
		"http://some.url.1",
		"http://some.url.2"
	);

	var r1 = results[0];
	var r2 = results[1];

	var r3 = yield request(
		"http://some.url.3/?v=" + r1 + "," + r2
	);

	console.log( r3 );
}

// use previously defined `run(..)` utility
run( foo );
```

里面`* foo()`更干净ㄡ更清楚的是,我们所做的只是在问`酒吧(ⅆ)`给我们一些`结果`我们会`产量`-等着发生ㄢ我们不必在封面下关心这个问题ㄢ`承诺(全部)])`承诺作文将用于制作

**我们对待异步,实际上承诺,作为一个实现细节ㄢ**

如果你想做一个复杂的串行流控制,把你的承诺逻辑隐藏在你只从生成器调用的函数中是非常有用的ㄢ例如:

```js
function bar() {
	return	Promise.all( [
		  baz( .. )
		  .then( .. ),
		  Promise.race( [ .. ] )
		] )
		.then( .. )
}
```

这种逻辑有时是必需的,如果你直接将它转储到你的生成器中,你就已经打败了你首先要使用生成器的大部分原因ㄢ我们_应该_有意地将这些细节从生成器代码中抽象出来,这样它们不会弄乱更高级别的任务表达式ㄢ

除了创造,是功能性和高性能的代码,你也应该努力使这是合理的和可维护的代码ㄢ

**注: **抽象是不_总是_一个健康的事情ℴℴ多次编程可以提高交换的简洁的复杂性ㄢ但在这种情况下,我相信它会更健康你的发电机+保证异步代码比的选择ㄢ不过,和所有这些建议一样,注意你的具体情况,为你和你的团队做出正确的决定ㄢ

## 发电机代表团

在上一节中,我们发现从内部的发电机叫正则函数,以及如何保持抽象出来的实施细节的一个有用的技术(如异步承诺流量)ㄢ但是使用这个函数的一个普通函数的主要缺点是它必须遵循正常的函数规则,这意味着它不能用`产量`像发电机一样ㄢ

然后,您可能会想到,您可以尝试使用另一个生成器调用来自另一个生成器的生成器ㄢ`跑步(ⅆ)`助手,如: 

```js
function *foo() {
	var r2 = yield request( "http://some.url.2" );
	var r3 = yield request( "http://some.url.3/?v=" + r2 );

	return r3;
}

function *bar() {
	var r1 = yield request( "http://some.url.1" );

	// "delegating" to `*foo()` via `run(..)`
	var r3 = yield run( foo );

	console.log( r3 );
}

run( bar );
```

我们跑`* foo()`里面的`* bar()`通过使用我们的`跑步(ⅆ)`实用又ㄢ我们利用这里的事实`跑步(ⅆ)`我们定义了前面返回的一个承诺,当生成器运行完成(或错误)时,该承诺将得到解决,所以如果我们`产量`去`跑步(ⅆ)`从另一个承诺的例子`跑步(ⅆ)`呼叫,它自动暂停`* bar()`直到`* foo()`完成ㄢ

但是有一个更好的整合呼叫的方法`* foo()`进入之内`* bar()`这叫`产量`-代表团ㄢ特殊语法`产量`-代表团是: `* __产量`(注意额外的)`*`)ㄢ在我们在前面的示例中看到它之前,让我们来看一个更简单的场景: 

```js
function *foo() {
	console.log( "`*foo()` starting" );
	yield 3;
	yield 4;
	console.log( "`*foo()` finished" );
}

function *bar() {
	yield 1;
	yield 2;
	yield *foo();	// `yield`-delegation!
	yield 5;
}

var it = bar();

it.next().value;	// 1
it.next().value;	// 2
it.next().value;	// `*foo()` starting
					// 3
it.next().value;	// 4
it.next().value;	// `*foo()` finished
					// 5
```

**注: **与前面一章中的说明类似,我解释了为什么我喜欢`功能* foo() ..`而不是`功能* foo() ..`我更喜欢ℴℴ与本主题的大多数其他文档不同ℴℴ比如说`* foo()产量`而不是`* foo()产量`ㄢ的位置`*`完全是风格化的,符合你的最佳判断ㄢ但我觉得造型的一致性很吸引人ㄢ

如何`* foo()产量`代表团的工作吗?

首先,调用`foo()`创建一个_迭代器_正如我们已经看到的ㄢ然后,`产量*`委托/转让_迭代器_实例控制(目前的)`* bar()`发电机)`* foo()` _迭代器_ㄢ

那么,前两个`next()它ㄢ`呼叫控制`* bar()`但当我们创造第三个`next()它ㄢ`电话,现在`* foo()`开始,现在我们控制`* foo()`而不是`* bar()`ㄢ这就是为什么它被称为代表团ℴℴ`* bar()`将其迭代控制委派给`* foo()`ㄢ

一旦`它` _迭代器_控制耗尽整个`* foo()` _迭代器_它自动返回控制ㄢ`* bar()`ㄢ

现在回到前面的例子中,列出三个连续的Ajax请求: 

```js
function *foo() {
	var r2 = yield request( "http://some.url.2" );
	var r3 = yield request( "http://some.url.3/?v=" + r2 );

	return r3;
}

function *bar() {
	var r1 = yield request( "http://some.url.1" );

	// "delegating" to `*foo()` via `yield*`
	var r3 = yield *foo();

	console.log( r3 );
}

run( bar );
```

这个片段与前面使用的版本之间的唯一区别是`* foo()产量`而不是以前的`收益率运行(富)`ㄢ

**注: ** `产量*`产生迭代控制,而不是生成器控制;当您调用`* foo()`发电机,你现在`产量`-委托其_迭代器_ㄢ但实际上你可以`产量`-代表任何_可迭代的_;`产量×[1,2,3]`将消耗默认值_迭代器_对于`[1,2,3]`数组的值ㄢ

### 为什么代表团?

目的`产量`-委托主要是代码组织,这种方式与正常函数调用是对称的ㄢ

设想两个模块分别提供方法`foo()`和`bar()`,在那里`bar()`电话`foo()`ㄢ这两个是分开的原因通常是因为程序的适当组织要求它们处于不同的函数中ㄢ例如,可能有一些情况`foo()`被称为独立,和其他地方`bar()`电话`foo()`ㄢ

所有这些理由,保持发电机单独辅助程序的可读性,维护,和可调试性ㄢ在这方面,`产量*`是一种语法快捷方式,可以手动遍历`* foo()`而内部`* bar()`ㄢ

如果采取步骤,这种手工方法将特别复杂ㄢ`* foo()`是异步的,这就是为什么您可能需要使用它ㄢ`跑步(ⅆ)`效用去做ㄢ正如我们所展示的,`* foo()产量`消除了对子实例的需要ㄢ`跑步(ⅆ)`实用工具(如`运行(富)`)ㄢ

### 授权信息

你可能想知道这是怎么回事`产量`-授权不仅仅适用于_迭代器_控制,但用`产量`-代表团: 

```js
function *foo() {
	console.log( "inside `*foo()`:", yield "B" );

	console.log( "inside `*foo()`:", yield "C" );

	return "D";
}

function *bar() {
	console.log( "inside `*bar()`:", yield "A" );

	// `yield`-delegation!
	console.log( "inside `*bar()`:", yield *foo() );

	console.log( "inside `*bar()`:", yield "E" );

	return "F";
}

var it = bar();

console.log( "outside:", it.next().value );
// outside: A

console.log( "outside:", it.next( 1 ).value );
// inside `*bar()`: 1
// outside: B

console.log( "outside:", it.next( 2 ).value );
// inside `*foo()`: 2
// outside: C

console.log( "outside:", it.next( 3 ).value );
// inside `*foo()`: 3
// inside `*bar()`: D
// outside: E

console.log( "outside:", it.next( 4 ).value );
// inside `*bar()`: 4
// outside: F
```

以后要特别注意处理步骤ㄢ`下一个(3)ㄢ`呼叫: 

1.  这个`三`值被传递(通过`产量`-代表团`* bar()`进入等待`产量"Cℽ`表达内心的`* foo()`ㄢ
2.  `* foo()`然后调用`返回"Dℽ`但这个值不会一直返回到外部ㄢ`下一个(3)ㄢ`呼叫ㄢ
3.  相反,该`"Dℽ`值作为等待的结果发送ㄢ`* foo()产量`表达内心的`* bar()`ℴℴ这`产量`-委托表达式基本上已暂停,而所有`* foo()`筋疲力尽ㄢ所以`"Dℽ`在里面结束`* bar()`打印出来ㄢ
4.  `"Eℽ产量`被称为`* bar()`,和`"Eℽ`价值是从外部产生的ㄢ`下一个(3)ㄢ`呼叫ㄢ

从外部的角度看_迭代器_(`它`)控制初始生成器或委托生成器之间没有什么不同ㄢ

事实上,`产量`-授权甚至不需要直接指向另一个生成器,它可以直接指向非生成器,通用的ㄢ_可迭代的_ㄢ例如:

```js
function *bar() {
	console.log( "inside `*bar()`:", yield "A" );

	// `yield`-delegation to a non-generator!
	console.log( "inside `*bar()`:", yield *[ "B", "C", "D" ] );

	console.log( "inside `*bar()`:", yield "E" );

	return "F";
}

var it = bar();

console.log( "outside:", it.next().value );
// outside: A

console.log( "outside:", it.next( 1 ).value );
// inside `*bar()`: 1
// outside: B

console.log( "outside:", it.next( 2 ).value );
// outside: C

console.log( "outside:", it.next( 3 ).value );
// outside: D

console.log( "outside:", it.next( 4 ).value );
// inside `*bar()`: undefined
// outside: E

console.log( "outside:", it.next( 5 ).value );
// inside `*bar()`: 5
// outside: F
```

请注意在这个示例和前面的消息之间接收到的消息的差异ㄢ

最引人注目的是违约ㄢ`阵列` _迭代器_不关心发送的任何消息`下一个(..)`调用,所以值`二`,`三`,和`四`本质上被忽略ㄢ而且,因为_迭代器_没有显式的`返回`值(与以前使用的值不同)`* foo()`),这`产量*`表达得到`未定义`当它完成ㄢ

#### 委派的例外情况!

以同样的方式`产量`-授权透明地通过两个方向传递消息,错误/异常也通过两个方向传递: 

```js
function *foo() {
	try {
		yield "B";
	}
	catch (err) {
		console.log( "error caught inside `*foo()`:", err );
	}

	yield "C";

	throw "D";
}

function *bar() {
	yield "A";

	try {
		yield *foo();
	}
	catch (err) {
		console.log( "error caught inside `*bar()`:", err );
	}

	yield "E";

	yield *baz();

	// note: can't get here!
	yield "G";
}

function *baz() {
	throw "F";
}

var it = bar();

console.log( "outside:", it.next().value );
// outside: A

console.log( "outside:", it.next( 1 ).value );
// outside: B

console.log( "outside:", it.throw( 2 ).value );
// error caught inside `*foo()`: 2
// outside: C

console.log( "outside:", it.next( 3 ).value );
// error caught inside `*bar()`: D
// outside: E

try {
	console.log( "outside:", it.next( 4 ).value );
}
catch (err) {
	console.log( "error caught outside:", err );
}
// error caught outside: F
```

从这段代码中需要注意的事项: 

1.  我们打电话的时候`它扔(2)`它发送错误消息ㄢ`二`进入之内`* bar()`,这代表`* foo()`,然后`抓住`然后优雅地处理它ㄢ然后,该`产量"Cℽ`发送`"Cℽ`返回作为回报`价值`从`它扔(2)`呼叫ㄢ
2.  这个`"Dℽ`下一个值`扔`n从里面`* foo()`传播出去`* bar()`,这`抓住`然后优雅地处理它ㄢ然后`"Eℽ产量`发送`"Eℽ`返回作为回报`价值`从`下一个(3)ㄢ`呼叫ㄢ
3.  接下来,例外`扔`n`* baz()`没有被抓住`* bar()`ℴℴ虽然我们做到了`抓住`它在外面--所以都`* baz()`和`* bar()`设置为已完成状态ㄢ在这个片段之后,您将无法得到`"Gℽ`以任何后续价值`下一个(..)`打电话ℴℴ他们会回来的`未定义`对于`价值`ㄢ

### 授权不同步

让我们回到以前的生活吧ㄢ`产量`-多个连续Ajax请求的委托示例: 

```js
function *foo() {
	var r2 = yield request( "http://some.url.2" );
	var r3 = yield request( "http://some.url.3/?v=" + r2 );

	return r3;
}

function *bar() {
	var r1 = yield request( "http://some.url.1" );

	var r3 = yield *foo();

	console.log( r3 );
}

run( bar );
```

而不是打电话`收益率运行(富)`里面的`* bar()`我们只是打电话`* foo()产量`ㄢ

在本例的前一版本中,承诺机制(由`跑步(ⅆ)`用于将值从`返回R3`在里面`* foo()`到局部变量`R3`里面`* bar()`ㄢ现在,该值直接返回`产量*`力学ㄢ

否则,行为几乎完全相同ㄢ

### 委托"递归ℽ

当然,`产量`-当你接通时,代表团可以遵循许多代表团的步骤ㄢ你甚至可以使用`产量`-代表团能够"递归ℽℴℴ异步发电机发电机`产量`-授权给自己: 

```js
function *foo(val) {
	if (val > 1) {
		// generator recursion
		val = yield *foo( val - 1 );
	}

	return yield request( "http://some.url/?v=" + val );
}

function *bar() {
	var r1 = yield *foo( 3 );
	console.log( r1 );
}

run( bar );
```

**注: **我们的`跑步(ⅆ)`可以调用实用程序`运行(富,3)`因为它支持传递给生成器初始化的附加参数ㄢ但是,我们使用了一个参数ㄢ`* bar()`这里强调的灵活性`产量*`ㄢ

从代码中得到什么处理步骤?等一下,这将是相当复杂的详细描述: 

1.  `运行(酒吧)`启动`* bar()`发电机ㄢ
2.  `美孚(3)`创建一个_迭代器_对于`*富(..)`并通过`三`作为其`瓦尔`参数.
3.  因为`3 > 1`,`美孚(2)`创建另一个_迭代器_并通过`二`作为其`瓦尔`参数.
4.  因为`2 > 1`,`美孚(1)`创建另一个_迭代器_并通过`一`作为其`瓦尔`参数.
5.  `1 > 1`是`假`所以我们下一个电话`请求(ⅆ)`与`一`值,并为第一个Ajax调用获得一个承诺ㄢ
6.  这个承诺是`产量`退了出来,又回到了`*美孚(2)`发电机的实例ㄢ
7.  这个`产量*`把那个承诺传递给`*美孚(3)`发电机的实例ㄢ另一个`产量*`把诺言传给`* bar()`发电机的实例ㄢ又一次`产量*`把诺言传给`跑步(ⅆ)`实用程序,它将等待该承诺(第一个Ajax请求)继续进行ㄢ
8.  当承诺解决时,它的执行信息被发送到简历中ㄢ`* bar()`,它经过`产量*`进入`*美孚(3)`实例,然后通过`产量*`到`*美孚(2)`生成器实例,然后通过`产量*`到正常`产量`这是在等待`*美孚(3)`发电机的实例ㄢ
9.  第一个调用的Ajax响应现在立即生效ㄢ`返回`由`*美孚(3)`生成器实例,它将该值作为结果返回`产量*`表达`*美孚(2)`实例,并分配给它的本地`瓦尔`变量ㄢ
10. 里面`*美孚(2)`第二个Ajax请求是用`请求(ⅆ)`谁的诺言是`产量`返回到`*美孚(1)`实例,然后`产量*`传播到所有的方式`跑步(ⅆ)`(第7步)ㄢ当该承诺解析时,第二个Ajax响应会一直传播到`*美孚(2)`生成器实例,并分配给它的本地`瓦尔`变量ㄢ
11. 最后,第三个Ajax请求是用`请求(ⅆ)`它的承诺会实现`跑步(ⅆ)`然后,它的解析值又回到了原来的位置ㄢ`返回`让它回到等待中`产量*`表达`* bar()`ㄢ

唷!很多疯狂的脑力杂耍,嗯?你可能想多读几遍,然后去吃点小吃来清醒头脑!

## 发电机的并发

正如我们在本章第1章和前面所讨论的,两个同时运行的"进程ℽ可以合作地交错它们的操作,并且多次这样做可以_产量_(双关语)非常强大的异步表达式ㄢ

坦率地说,我们早期的多个生成器并发交织的例子展示了如何使它真正混淆ㄢ但我们暗示有一些地方,这种能力是非常有用的ㄢ

回想一下我们在第1章中看到的场景,其中两个不同的并发Ajax响应处理程序需要相互协调以确保数据通信不是一个竞争条件ㄢ我们把答复分成`物件`像这样的数组: 

```js
function response(data) {
	if (data.url == "http://some.url.1") {
		res[0] = data;
	}
	else if (data.url == "http://some.url.2") {
		res[1] = data;
	}
}
```

但是,我们如何能同时使用多个生成器呢?

```js
// `request(..)` is a Promise-aware Ajax utility

var res = [];

function *reqData(url) {
	res.push(
		yield request( url )
	);
}
```

**注: **我们将使用两个实例`* reqdata(..)`生成器在这里,但是运行两个不同的生成器的一个实例没有区别;这两种方法都是相同的ㄢ我们会看到两个不同的发电机协调在一点点ㄢ

而不是手动排序`RES [ 0 ]`和`RES [ 1 ]`赋值,我们将使用协调排序,以便`res.push(..)`正确地按预期和可预见的顺序存储值ㄢ因此,表达的逻辑应该有点干净ㄢ

但是我们将如何协调这种互动呢?首先,让我们用诺言手动完成它: 

```js
var it1 = reqData( "http://some.url.1" );
var it2 = reqData( "http://some.url.2" );

var p1 = it1.next().value;
var p2 = it2.next().value;

p1
.then( function(data){
	it1.next( data );
	return p2;
} )
.then( function(data){
	it2.next( data );
} );
```

`* reqdata(..)`两个实例都开始创建Ajax请求,然后暂停`产量`ㄢ然后我们选择恢复第一个实例ㄢ`P1`解析,然后`P2`该决议将重新启动第二个实例ㄢ通过这种方式,我们使用承诺编排来确保`RES [ 0 ]`会有第一反应和`RES [ 1 ]`会有第二个反应ㄢ

但坦率地说,这是非常手工的,它并不是真的让发电机协调自己,这才是真正的力量所在ㄢ让我们用另一种方法试试: 

```js
// `request(..)` is a Promise-aware Ajax utility

var res = [];

function *reqData(url) {
	var data = yield request( url );

	// transfer control
	yield;

	res.push( data );
}

var it1 = reqData( "http://some.url.1" );
var it2 = reqData( "http://some.url.2" );

var p1 = it1.next().value;
var p2 = it2.next().value;

p1.then( function(data){
	it1.next( data );
} );

p2.then( function(data){
	it2.next( data );
} );

Promise.all( [p1,p2] )
.then( function(){
	it1.next();
	it2.next();
} );
```

好的,这有点好(尽管仍然是手动的!)因为现在的两个实例`* reqdata(..)`独立运行,至少(第一部分)独立运行ㄢ

在前一段代码中,直到第一个实例完全完成后,第二个实例才得到它的数据ㄢ但在这里,两个实例一旦各自的响应返回,就接收数据,然后每个实例执行另一个实例ㄢ`产量`用于控制转移目的ㄢ然后,我们选择什么顺序来恢复它们在`承诺(全部)])`处理程序ㄢ

可能不那么明显的是,由于对称性,这种方法暗示了一种更易于重用的实用程序形式ㄢ我们可以做得更好ㄢ让我们假设使用一个实用程序`奔跑(..)`: 

```js
// `request(..)` is a Promise-aware Ajax utility

var res = [];

runAll(
	function*(){
		var p1 = request( "http://some.url.1" );

		// transfer control
		yield;

		res.push( yield p1 );
	},
	function*(){
		var p2 = request( "http://some.url.2" );

		// transfer control
		yield;

		res.push( yield p2 );
	}
);
```

**注: **我们不包括一个代码清单`奔跑(..)`因为它不仅足够长,可以把文本压缩下来,而且是我们已经实现的逻辑的扩展ㄢ`跑步(ⅆ)`早期的.因此,作为对读者的一个很好的补充练习,请尝试从代码演变`跑步(ⅆ)`像想象中的那样工作`奔跑(..)`ㄢ另外,我_asynquence_库提供了前面提到的`跑步者(ⅆ)`具有这种功能的实用程序已经内置,并将在本书附录A中讨论ㄢ

下面是内部的处理过程`奔跑(..)`会操作: 

1.  第一个生成器得到了第一个Ajax响应的承诺ㄢ`"1ℽ`,然后`产量`s控制回`奔跑(..)`效用ㄢ
2.  第二个生成器运行并执行相同的操作ㄢ`"2ℽ`,`产量`ING控制回到`奔跑(..)`效用ㄢ
3.  第一个生成器恢复,然后`产量`履行诺言`P1`ㄢ这个`奔跑(..)`在本例中,实用程序与前面的相同ㄢ`跑步(ⅆ)`因为它等待那个承诺去解决,然后恢复相同的生成器(没有控制转移!)ㄢ当`P1`解决了,`奔跑(..)`用该分辨率值再次启动第一个生成器,然后`RES [ 0 ]`被赋予它的价值ㄢ当第一个生成器完成时,这是一个隐式的控制转移ㄢ
4.  第二台发电机恢复运行,`产量`履行诺言`P2`等着它去解决ㄢ一旦它做到了,`奔跑(..)`用该值恢复第二个生成器,并`RES [ 1 ]`是集ㄢ

在这个运行中`物件`要存储两种不同Ajax响应的结果ℴℴ这是我们的并发协调使之成为可能ㄢ

但进一步扩展可能会有很大帮助ㄢ`奔跑(..)`为多个生成器实例提供一个内部变量空间_分享_比如我们要调用的空对象`数据`下面ㄢ此外,它可以接受非承诺值ㄢ`产量`把它们交给下一个发电机ㄢ

考虑: 

```js
// `request(..)` is a Promise-aware Ajax utility

runAll(
	function*(data){
		data.res = [];

		// transfer control (and message pass)
		var url1 = yield "http://some.url.2";

		var p1 = request( url1 ); // "http://some.url.1"

		// transfer control
		yield;

		data.res.push( yield p1 );
	},
	function*(data){
		// transfer control (and message pass)
		var url2 = yield "http://some.url.1";

		var p2 = request( url2 ); // "http://some.url.2"

		// transfer control
		yield;

		data.res.push( yield p2 );
	}
);
```

在这一表述中,这两个生成器不仅协调控制转移,而且实际上相互沟通,两者都通过ㄢ`data.res`和`产量`贸易信息`url1`和`url2`价值观ㄢ难以置信的强大!

这种实现还可以作为更复杂的异步技术CSP(通信顺序进程)的概念基础,我们将在本书附录B中介绍ㄢ

## 这个

到目前为止,我们已经假定`产量`发出一个来自生成器的承诺,并承诺通过一个助手实用程序恢复生成器`跑步(ⅆ)`ℴℴ最好的方法是用生成器管理异步ㄢ很明显,它是ㄢ

但我们跳过了另一种模式,它有一些广泛的采用,所以为了完整性,我们将简要地看一下它ㄢ

在一般的计算机科学,有一个古老的概念称为"预JS咚ㄢℽ没有陷入历史的性质,在JS的thunk的狭窄的表达式是一个函数,没有参数连接到调用另一个函数ㄢ

换句话说,您在函数调用周围包装了一个函数定义ℴℴ用它所需的任何参数_推迟_那叫执行,并且包装函数是一个thunkㄢ当你以后执行thunk,你最终调用原函数ㄢ

例如:

```js
function foo(x,y) {
	return x + y;
}

function fooThunk() {
	return foo( 3, 4 );
}

// later

console.log( fooThunk() );	// 7
```

所以,一个同步的thunk很简单ㄢ但对于异步咚?我们可以扩大狭窄的thunk的定义包括它接收回调ㄢ

考虑: 

```js
function foo(x,y,cb) {
	setTimeout( function(){
		cb( x + y );
	}, 1000 );
}

function fooThunk(cb) {
	foo( 3, 4, cb );
}

// later

fooThunk( function(sum){
	console.log( sum );		// 7
} );
```

正如你所看到的,`foothunk(..)`唯一的希望`CB(..)`参数,因为它已经有值`三`和`四`(为`X`和`Y`分别)预先指定并准备传递给`富(ⅆ)`ㄢ一个thunk是耐心等待完成它需要的最后一件: 回调ㄢ

你不想让这个手动,虽然ㄢ所以,让我们发明一个实用工具,为我们做这个包装ㄢ

考虑: 

```js
function thunkify(fn) {
	var args = [].slice.call( arguments, 1 );
	return function(cb) {
		args.push( cb );
		return fn.apply( null, args );
	};
}

var fooThunk = thunkify( foo, 3, 4 );

// later

fooThunk( function(sum) {
	console.log( sum );		// 7
} );
```

**提示: **在这里我们假设原文(`富(ⅆ)`函数签名期望它的回调在最后一个位置,任何其他参数都会出现在它前面ㄢ这是异步js功能标准相当普遍存在的"标准ℽㄢ你可以称之为"回调最后一种风格ℽㄢ如果出于某种原因,你需要处理"回调第一样式ℽ签名,你只需要做一个实用程序ㄢ`argsㄢUnshift(..)`而不是`argsㄢ推(..)`ㄢ

前面的公式`thunkify(..)`兼顾`富(ⅆ)`函数的引用,以及任何参数需要,返回thunk本身(`foothunk(..)`)ㄢ然而,这不是你会发现这个JS的典型方法ㄢ

而不是`thunkify(..)`使觉得本身,通常如果不是令人困惑的`thunkify(..)`效用会产生作用,产生这个ㄢ

嗯ⅆ是啊.

考虑: 

```js
function thunkify(fn) {
	return function() {
		var args = [].slice.call( arguments );
		return function(cb) {
			args.push( cb );
			return fn.apply( null, args );
		};
	};
}
```

这里的主要区别是额外的ㄢ`返回function() { ..}`层ㄢ下面是它用法的不同之处: 

```js
var whatIsThis = thunkify( foo );

var fooThunk = whatIsThis( 3, 4 );

// later

fooThunk( function(sum) {
	console.log( sum );		// 7
} );
```

显然,这个片段暗示的一个大问题是什么?`是什么`所谓的正确吗?这是不是想到,这是将要发生的事情从这个`富(ⅆ)`电话.这就像一个"工厂ℽ的"这个ℽ,似乎没有任何一种标准协议命名这样的事ㄢ

所以,我的建议是"thunkoryℽ("咚ℽ+"工厂ℽ)ㄢ所以,`thunkify(..)`产生一个thunkory,和thunkory产生这个ㄢ这种推理是对称的我的建议"promisoryℽ3章: 

```js
var fooThunkory = thunkify( foo );

var fooThunk1 = fooThunkory( 3, 4 );
var fooThunk2 = fooThunkory( 5, 6 );

// later

fooThunk1( function(sum) {
	console.log( sum );		// 7
} );

fooThunk2( function(sum) {
	console.log( sum );		// 11
} );
```

**注: **运行`富(ⅆ)`示例希望回调类型不是"错误第一样式ℽㄢ当然,"错误优先样式ℽ更为常见ㄢ如果`富(ⅆ)`有某种合法的错误产生期望,我们可以将它更改为期望并使用错误第一回调函数ㄢ没有后续的`thunkify(..)`机器在乎回调是什么样的?ㄢ唯一不同的用法是`foothunk1(功能(呃,总和){ ..`ㄢ

暴露thunkory方法--而不是如何早期`thunkify(..)`隐藏中间步骤ℴℴ似乎是不必要的并发症ㄢ但总的来说,在你的程序将现有的API方法开始让thunkories很有用,然后可以通过在调用这些thunkories当你需要这个ㄢ这两个截然不同的步骤保持了更干净的能力分离ㄢ

说明: 

```js
// cleaner:
var fooThunkory = thunkify( foo );

var fooThunk1 = fooThunkory( 3, 4 );
var fooThunk2 = fooThunkory( 5, 6 );

// instead of:
var fooThunk1 = thunkify( foo, 3, 4 );
var fooThunk2 = thunkify( foo, 5, 6 );
```

不管你是否喜欢处理的thunkories明确与否的thunk的用法`foothunk1(..)`和`foothunk2(..)`保持不变ㄢ

### S /承诺/咚/

那么所有这些东西都与发电机的thunk?

相比这个承诺一般: 他们不直接转化为他们的

但另一方面,它们都可以被视为一种价值的要求,这可能是异步的回答ㄢ

回顾一下第3章我们promisifying函数定义的效用,我们称之为`承诺,包装(ⅆ)`ℴℴ我们可以叫它`promisify(..)`,太!这一承诺包装效用不产生的承诺;它产生promisories反过来产生的承诺ㄢ这是完全对称的,目前正在讨论这个thunkoriesㄢ

为了说明对称性,让我们首先改变跑步ㄢ`富(ⅆ)`从以前的例子中假设"错误第一样式ℽ回调: 

```js
function foo(x,y,cb) {
	setTimeout( function(){
		// assume `cb(..)` as "error-first style"
		cb( null, x + y );
	}, 1000 );
}
```

现在,我们将比较使用`thunkify(..)`和`promisify(..)`(又名`承诺,包装(ⅆ)`从第3章): 

```js
// symmetrical: constructing the question asker
var fooThunkory = thunkify( foo );
var fooPromisory = promisify( foo );

// symmetrical: asking the question
var fooThunk = fooThunkory( 3, 4 );
var fooPromise = fooPromisory( 3, 4 );

// get the thunk answer
fooThunk( function(err,sum){
	if (err) {
		console.error( err );
	}
	else {
		console.log( sum );		// 7
	}
} );

// get the promise answer
fooPromise
.then(
	function(sum){
		console.log( sum );		// 7
	},
	function(err){
		console.error( err );
	}
);
```

无论是thunkory和promisory基本上是问一个问题(一个值),分别与一`foothunk`并承诺`foopromise`代表将来对这个问题的答案ㄢ在这种情况下,对称性是清楚的ㄢ

考虑到这一点,我们可以看到发电机`产量`对异步的承诺可以代替`产量`这个异步ㄢ我们需要的只是一个更聪明的人`跑步(ⅆ)`实用程序(类似于以前),它不仅可以查找和连接到`产量`ED的承诺,但也提供一个回调到`产量`ED咚ㄢ

考虑: 

```js
function *foo() {
	var val = yield request( "http://some.url.1" );
	console.log( val );
}

run( foo );
```

在这个例子中,`请求(ⅆ)`可以是一个promisory返回一个承诺,或thunkory返回咚ㄢ从生成器代码逻辑内部的角度来看,我们并不关心实现细节,它非常强大!

所以,`请求(ⅆ)`可能是: 

```js
// promisory `request(..)` (see Chapter 3)
var request = Promise.wrap( ajax );

// vs.

// thunkory `request(..)`
var request = thunkify( ajax );
```

最后,作为一个thunk补丁我们早知道`跑步(ⅆ)`实用,我们需要像这样的逻辑: 

```js
// ..
// did we receive a thunk back?
else if (typeof next.value == "function") {
	return new Promise( function(resolve,reject){
		// call the thunk with an error-first callback
		next.value( function(err,msg) {
			if (err) {
				reject( err );
			}
			else {
				resolve( msg );
			}
		} );
	} )
	.then(
		handleNext,
		function handleErr(err) {
			return Promise.resolve(
				it.throw( err )
			)
			.then( handleResult );
		}
	);
}
```

现在,我们可以叫promisories到发电机`产量`承诺,或打电话给thunkories`产量`这个,在任何情况下,`跑步(ⅆ)`将处理该值,并使用它等待完成恢复生成器ㄢ

对称性,这两种方法看起来完全相同ㄢ然而,我们应该指出这是真的只有从这个承诺或表示发电机的未来价值的延续ㄢ

从大的角度来看,这个不在和自己几乎没有任何的信任或承诺性保证了ㄢ使用一个thunk是站在在这个特殊的发电机异步模式的承诺是可行的但应被视为不理想时相比,所有的好处,承诺提供(见3章)ㄢ

如果你有选择,请选择`产量的公关`而不是`屈服了`ㄢ但是拥有一个`跑步(ⅆ)`可以同时处理两种值类型的实用程序ㄢ

**注: **这个`跑步者(ⅆ)`它在我的_asynquence_库,将在附录A中讨论,句柄`产量`的承诺,这个和_asynquence_序列ㄢ

## pre-es6发电机

希望你现在认为发电机的异步编程工具箱中一个非常重要的补充ㄢ但它在6新语法,这意味着你不能只是polyfill发电机能像你的承诺(这是一个新的API)ㄢ所以我们能做些什么使发电机我们的浏览器js如果我们没有忽略pre-es6浏览器的奢侈品?

在6所有新的语法扩展,有工具--他们最为常见的术语transpilers,跨编译器,可以把你的6语法和转化(但显然更难看!)pre-es6代码ㄢ因此,发电机可以transpiled成代码,也会有同样的行为,但在ES5和下面的工作ㄢ

但如何?"魔法ℽ`产量`不明显的声音像代码,很容易transpileㄢ实际上,我们在前面关于闭包的讨论中暗示了一种解决方案ㄢ_迭代器_ㄢ

### 手动转换

在我们讨论的transpilers,让我们得到transpilation如何手动将发电机的情况下工作ㄢ这不仅仅是一个学术活动,因为这样做实际上有助于进一步加强他们的工作方式ㄢ

考虑: 

```js
// `request(..)` is a Promise-aware Ajax utility

function *foo(url) {
	try {
		console.log( "requesting:", url );
		var val = yield request( url );
		console.log( val );
	}
	catch (err) {
		console.log( "Oops:", err );
		return false;
	}
}

var it = foo( "http://some.url.1" );
```

首先要注意的是,我们仍然需要一个正常的`foo()`可以调用的函数,它仍然需要返回一个_迭代器_ㄢ那么,让我们勾勒出非生成器转换: 

```js
function foo(url) {

	// ..

	// make and return an iterator
	return {
		next: function(v) {
			// ..
		},
		throw: function(e) {
			// ..
		}
	};
}

var it = foo( "http://some.url.1" );
```

接下来要注意的是,一个生成器通过暂停它的范围/状态来实现它的"魔力ℽ,但是我们可以用函数闭包来模拟它(参见_范围和关闭_本系列的标题)ㄢ为了理解如何编写这样的代码,我们将首先用状态值注释生成器的不同部分: 

```js
// `request(..)` is a Promise-aware Ajax utility

function *foo(url) {
	// STATE *1*

	try {
		console.log( "requesting:", url );
		var TMP1 = request( url );

		// STATE *2*
		var val = yield TMP1;
		console.log( val );
	}
	catch (err) {
		// STATE *3*
		console.log( "Oops:", err );
		return false;
	}
}
```

**注: **为了更准确地说明,我们将`屈服请求ⅆ`语句分为两部分,使用临时`TMP1`变量ㄢ`请求(ⅆ)`发生在状态`* 1 *`以及其完成值的赋值`瓦尔`发生在状态`* 2 *`ㄢ我们将把那个中间的东西除掉ㄢ`TMP1`当我们将代码转换为它的非生成器等价物时ㄢ

换言之,`* 1 *`是开始状态,`* 2 *`国家是否`请求(ⅆ)`成功,和`* 3 *`国家是否`请求(ⅆ)`失败.你可以想象一下额外的费用ㄢ`产量`步骤将被编码为前

回到我们的transpiled发生器,让我们定义一个变量`状态`在闭包中,我们可以用来跟踪状态: 

```js
function foo(url) {
	// manage generator state
	var state;

	// ..
}
```

现在,我们定义一个名为`过程(ⅆ)`在处理每个状态的闭包中,使用`转换`声明: 

```js
// `request(..)` is a Promise-aware Ajax utility

function foo(url) {
	// manage generator state
	var state;

	// generator-wide variable declarations
	var val;

	function process(v) {
		switch (state) {
			case 1:
				console.log( "requesting:", url );
				return request( url );
			case 2:
				val = v;
				console.log( val );
				return;
			case 3:
				var err = v;
				console.log( "Oops:", err );
				return false;
		}
	}

	// ..
}
```

生成器中的每个状态都是由自己表示的ㄢ`案例`在`转换`声明ㄢ`过程(ⅆ)`每次我们需要处理一个新的状态时都会调用它ㄢ我们会在一瞬间回到这个过程中ㄢ

对于任何生成器范围变量声明(`瓦尔`),我们将这些移动到`VaR`声明之外`过程(ⅆ)`这样他们就可以在多次通话中存活`过程(ⅆ)`ㄢ但"块域ℽ`犯错`变量只需要`* 3 *`状态,所以我们把它放在适当的位置ㄢ

在状态`* 1 *`,而不是`屈服请求(ⅆ)`,我们做了`返回请求(..)`ㄢ在终端状态`* 2 *`没有明确的`返回`所以我们只做一个`返回;`这是一样的`返回未定义`ㄢ在终端状态`* 3 *`有一个`返回false`所以我们保留ㄢ

现在我们需要在_迭代器_他们调用的函数`过程(ⅆ)`适当的: 

```js
function foo(url) {
	// manage generator state
	var state;

	// generator-wide variable declarations
	var val;

	function process(v) {
		switch (state) {
			case 1:
				console.log( "requesting:", url );
				return request( url );
			case 2:
				val = v;
				console.log( val );
				return;
			case 3:
				var err = v;
				console.log( "Oops:", err );
				return false;
		}
	}

	// make and return an iterator
	return {
		next: function(v) {
			// initial state
			if (!state) {
				state = 1;
				return {
					done: false,
					value: process()
				};
			}
			// yield resumed successfully
			else if (state == 1) {
				state = 2;
				return {
					done: true,
					value: process( v )
				};
			}
			// generator already completed
			else {
				return {
					done: true,
					value: undefined
				};
			}
		},
		"throw": function(e) {
			// the only explicit error handling is in
			// state *1*
			if (state == 1) {
				state = 3;
				return {
					done: true,
					value: process( e )
				};
			}
			// otherwise, an error won't be handled,
			// so just throw it right back out
			else {
				throw e;
			}
		}
	};
}
```

这个代码是如何工作的?

1.  第一次打电话给_迭代器_的`next()`呼叫将发电机从初始化状态`一`然后叫`process()`处理那个国家ㄢ返回值从`请求(ⅆ)`这是Ajax响应的承诺,返回为`价值`房地产从`next()`呼叫ㄢ
2.  如果Ajax请求成功,则第二次调用`下一个(..)`应该发送Ajax响应值,它将我们的状态移动到`二`ㄢ`过程(ⅆ)`再次调用(此时使用传入的Ajax响应值),并且`价值`属性返回的`下一个(..)`将`未定义`ㄢ
3.  但是,如果Ajax请求失败,`投掷(ⅆ)`应该用错误调用,它将从状态移动`一`到`三`(而不是`二`)ㄢ阿盖恩`过程(ⅆ)`这一次被称为错误值ㄢ那`案例`返回`假`,它被设置为`价值`返回的属性`投掷(ⅆ)`呼叫ㄢ

从外部ℴℴ也就是说,只与_迭代器_ℴℴ这`富(ⅆ)`正常函数的工作原理与`*富(..)`发电机会工作的ㄢ所以我们已经有效地"transpiledℽ我们6发电机pre-es6兼容性!

然后我们可以手动实例化生成器并控制它的迭代器ℴℴ调用`var =ⅆ`和`下一个(..)`这样或者更好,我们可以把它传递给我们先前定义的`跑步(ⅆ)`效用`run(foo,"..")`ㄢ

### 自动transpilation

手动导出我们6发电机改造pre-es6等效前运动教我们如何工作,发电机的概念ㄢ但是这种转换非常复杂,在我们的代码中对其他生成器非常不可移植ㄢ这将是相当不切实际的用手做这项工作,并将完全排除所有的发电效益ㄢ

但幸运的是,一些工具已经存在,可以自动转换成6发电机有什么喜欢的东西我们得到在前面的部分ㄢ他们不仅为我们做繁重的工作,但他们也处理一些并发症,我们掩饰ㄢ

一个这样的工具是再生器(<https://facebook.github.io/regenerator/>)来自脸谱网的聪明人ㄢ

如果我们使用再生器transpile我们以前的发电机,这里的代码产生(在撰写本文时): 

```js
// `request(..)` is a Promise-aware Ajax utility

var foo = regeneratorRuntime.mark(function foo(url) {
    var val;

    return regeneratorRuntime.wrap(function foo$(context$1$0) {
        while (1) switch (context$1$0.prev = context$1$0.next) {
        case 0:
            context$1$0.prev = 0;
            console.log( "requesting:", url );
            context$1$0.next = 4;
            return request( url );
        case 4:
            val = context$1$0.sent;
            console.log( val );
            context$1$0.next = 12;
            break;
        case 8:
            context$1$0.prev = 8;
            context$1$0.t0 = context$1$0.catch(0);
            console.log("Oops:", context$1$0.t0);
            return context$1$0.abrupt("return", false);
        case 12:
        case "end":
            return context$1$0.stop();
        }
    }, foo, this, [[0, 8]]);
});
```

在我们的手工推导中,有一些明显的相似之处,例如`转换`/`案例`声明,我们甚至看到`瓦尔`像我们一样从封闭处撤出ㄢ

当然,一个权衡,再生器的transpilation需要辅助库`regeneratorruntime`它包含管理通用生成器的所有可重用逻辑ㄢ_迭代器_ㄢ很多的样板期待不同,我们的版本,但即使这样,这个概念可以看出,像`背景1美元0.next = 4`跟踪发电机的下一个状态ㄢ

主要结论是,发电机不限于可在6 +环境ㄢ一旦理解了这些概念,就可以在代码中使用这些概念,并使用工具将代码转换为与旧环境兼容的代码ㄢ

这不仅仅是使用一个`承诺`对于pre-es6承诺polyfill API,但努力是完全值得的,因为发电机更好表达异步流量控制在合理的ㄡ理智的ㄡ同步的,顺序的方式ㄢ

一旦你迷上的发电机,你不会想回到意大利面异步回调的地狱!

## 回顾

发电机是一种新的6函数类型不运行完成正常的功能ㄢ相反,生成器可以在完成时暂停(完全保存它的状态),然后可以从它停止的地方恢复ㄢ

这种暂停/恢复交换是合作的,而不是先发制人的,这意味着生成器具有唯一的暂停能力,使用`产量`关键字,但_迭代器_控制发电机`下一个(..)`恢复发电机ㄢ

这个`产量`/`下一个(..)`二元性不仅仅是一种控制机制,它实际上是一种双向消息传递机制ㄢ一`产量ㄢ`表达式本质上是等待值的停顿,然后是下一个值ㄢ`下一个(..)`调用传递一个值(或隐式)`未定义`)回到那个停顿`产量`表达ㄢ

对异步发电机的流量控制相关的关键的好处是,在一个源代码表示序列的一种自然同步/顺序的任务步骤ㄢ诀窍在于我们本质上隐藏了潜在的异步ㄢ`产量`关键字ℴℴ将异步移动到生成器的代码中_迭代器_控制ㄢ

换句话说,发电机保护的顺序,同步,阻塞的异步代码模式,它可以让我们的大脑思考的代码更自然,解决一个基于异步回调的两个关键问题ㄢ
