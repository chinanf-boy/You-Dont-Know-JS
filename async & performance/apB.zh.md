
# 你不知道JS：异步性能

# 附录B：先进的异步模式

附录A介绍了_asynquence_序列型异步流控制库，主要是基于承诺和发电机。

现在我们将探索在现有的理解和功能之上构建的其他高级异步模式，并了解如何_asynquence_让那些复杂的异步技术容易混合和匹配在我们的程序不需要单独的图书馆很多。

## 迭代序列

我们介绍了_asynquence_在以往的附录的迭代序列，但我们想重温他们的更多细节。

刷新，回忆：

```js
var domready = ASQ.iterable();

// ..

domready.val( function(){
	// DOM is ready
} );

// ..

document.addEventListener( "DOMContentLoaded", domready.next );
```

现在，让我们定义一个序列的多步迭代序列：

```js
var steps = ASQ.iterable();

steps
.then( function STEP1(x){
	return x * 2;
} )
.then( function STEP2(x){
	return x + 3;
} )
.then( function STEP3(x){
	return x * 4;
} );

steps.next( 8 ).value;	// 16
steps.next( 16 ).value;	// 19
steps.next( 19 ).value;	// 76
steps.next().done;		// true
```

你可以看到，一个序列是一个符合标准的_迭代器_（见第4章）。因此，它可以用一个6迭代`对..`循环，就像发电机（或其他）_可迭代的_）可以：

```js
var steps = ASQ.iterable();

steps
.then( function STEP1(){ return 2; } )
.then( function STEP2(){ return 4; } )
.then( function STEP3(){ return 6; } )
.then( function STEP4(){ return 8; } )
.then( function STEP5(){ return 10; } );

for (var v of steps) {
	console.log( v );
}
// 2 4 6 8 10
```

在事件触发的例子显示在前面的附录，迭代序列是有趣的因为他们在本质上可以看作是一个站在发电机或承诺的枷锁，但更多的灵活性。

考虑多个Ajax请求的例子，我们看到在章节3和4相同的情况下，为保证链作为一个发电机，分别表示为一个序列：

```js
// sequence-aware ajax
var request = ASQ.wrap( ajax );

ASQ( "http://some.url.1" )
.runner(
	ASQ.iterable()

	.then( function STEP1(token){
		var url = token.messages[0];
		return request( url );
	} )

	.then( function STEP2(resp){
		return ASQ().gate(
			request( "http://some.url.2/?v=" + resp ),
			request( "http://some.url.3/?v=" + resp )
		);
	} )

	.then( function STEP3(r1,r2){ return r1 + r2; } )
)
.val( function(msg){
	console.log( msg );
} );
```

个序列表示一个连续的系列（同步或异步）步骤看起来非常类似于一个承诺链--换句话说，它非常干净看起来比只是简单的嵌套回调，但不是很好的`产量`生成器的序列语法。

但我们通过迭代序列`ASQ #转轮（..）`，它运行它完成相同的，如果它是一个发电机。事实上，一个序列的行为基本相同，发电机是显著的一对夫妇的原因。

第一个序列是一个pre-es6相当于6发电机的某个子集，这意味着你可以通过他们直接（在任何地方运行），或者你可以作者6发电机和transpile /转换成迭代序列（或为此事！保证链）。

思考一个异步运行完成发电机只是语法糖链是承诺他们的同构关系的一个重要的识别。

在继续之前，我们应该注意到前面的代码段可以用_asynquence_作为：

```js
ASQ( "http://some.url.1" )
.seq( /*STEP 1*/ request )
.seq( function STEP2(resp){
	return ASQ().gate(
		request( "http://some.url.2/?v=" + resp ),
		request( "http://some.url.3/?v=" + resp )
	);
} )
.val( function STEP3(r1,r2){ return r1 + r2; } )
.val( function(msg){
	console.log( msg );
} );
```

此外，第2步甚至可以被表示为：

```js
.gate(
	function STEP2a(done,resp) {
		request( "http://some.url.2/?v=" + resp )
		.pipe( done );
	},
	function STEP2b(done,resp) {
		request( "http://some.url.3/?v=" + resp )
		.pipe( done );
	}
)
```

那么，我们为什么要去表达我们的流量控制在一个序列的麻烦`ASQ #转轮（..）`当它看起来更简单/更平坦时_asyquence_这个工作链做得好吗？

因为迭代器序列形式具有重要的害处，给了我们更多的能力。读一读。

### 延伸的迭代序列

发电机，正常_asynquence_序列和承诺链都是**急切的评价**——无论最初的流量控制是如何表达的_是_将遵循的固定流程。

然而，迭代序列**懒洋洋地评价**，这意味着可迭代序列的执行过程中，您可以扩展序列步骤如果需要更多。

**注：**你只能附加到一个序列的末端，不注入到序列的中间。

让我们先看看这个功能的一个简单的（同步）例子，以便熟悉它：

```js
function double(x) {
	x *= 2;

	// should we keep extending?
	if (x < 500) {
		isq.then( double );
	}

	return x;
}

// setup single-step iterable sequence
var isq = ASQ.iterable().then( double );

for (var v = 10, ret;
	(ret = isq.next( v )) && !ret.done;
) {
	v = ret.value;
	console.log( v );
}
```

个序列开始时只有一个定义的步骤（`ISQ。然后（双）`）但序列在一定条件下不断扩展。`x＜500`）。两_asynquence_技术上的序列和承诺链_可以_做一些类似的事情，但我们会看到为什么他们的能力不足。

虽然这个例子相当琐碎，但也可以用`虽然`在生成器中循环，我们将考虑更复杂的情况。

例如，你可以从一个Ajax请求，如果发现需要更多的数据进行响应，你有条件地插入更多的步进迭代序列来实现额外的要求（S）。或者您可以有条件地添加一个值格式化步骤到Ajax处理结束。

考虑：

```js
var steps = ASQ.iterable()

.then( function STEP1(token){
	var url = token.messages[0].url;

	// was an additional formatting step provided?
	if (token.messages[0].format) {
		steps.then( token.messages[0].format );
	}

	return request( url );
} )

.then( function STEP2(resp){
	// add another Ajax request to the sequence?
	if (/x1/.test( resp )) {
		steps.then( function STEP5(text){
			return request(
				"http://some.url.4/?v=" + text
			);
		} );
	}

	return ASQ().gate(
		request( "http://some.url.2/?v=" + resp ),
		request( "http://some.url.3/?v=" + resp )
	);
} )

.then( function STEP3(r1,r2){ return r1 + r2; } );
```

你可以在两个不同的地方看到我们有条件的扩展。`步骤`具有`步骤。然后（…）`。然后运行这个`步骤`迭代序列，我们只是线到我们的主程序流程与_asynquence_序列（称为`主要的`这里）使用`ASQ #转轮（..）`：

```js
var main = ASQ( {
	url: "http://some.url.1",
	format: function STEP4(text){
		return text.toUpperCase();
	}
} )
.runner( steps )
.val( function(msg){
	console.log( msg );
} );
```

的灵活性（条件行为）`步骤`迭代序列与发电机的表达？有点，但我们必须以一种稍微笨拙的方式重新安排逻辑：

```js
function *steps(token) {
	// **STEP 1**
	var resp = yield request( token.messages[0].url );

	// **STEP 2**
	var rvals = yield ASQ().gate(
		request( "http://some.url.2/?v=" + resp ),
		request( "http://some.url.3/?v=" + resp )
	);

	// **STEP 3**
	var text = rvals[0] + rvals[1];

	// **STEP 4**
	// was an additional formatting step provided?
	if (token.messages[0].format) {
		text = yield token.messages[0].format( text );
	}

	// **STEP 5**
	// need another Ajax request added to the sequence?
	if (/foobar/.test( resp )) {
		text = yield request(
			"http://some.url.4/?v=" + text
		);
	}

	return text;
}

// note: `*steps()` can be run by the same `ASQ` sequence
// as `steps` was previously
```

撇开已经确定的序列生成器的同步、同步查找语法的好处（见第4章），`步骤`逻辑已经被记录在`* steps()`发电机的形式，假冒的可扩展的迭代序列的特性`步骤`。

那么，用承诺或序列表达功能如何呢？你_可以_做这样的事：

```js
var steps = something( .. )
.then( .. )
.then( function(..){
	// ..

	// extending the chain, right?
	steps = steps.then( .. );

	// ..
})
.then( .. );
```

这个问题很微妙，但很重要。所以，考虑把我们联系起来`步骤`承诺链进入我们的主体_asynquence_：

```js
var main = Promise.resolve( {
	url: "http://some.url.1",
	format: function STEP4(text){
		return text.toUpperCase();
	}
} )
.then( function(..){
	return steps;			// hint!
} )
.val( function(msg){
	console.log( msg );
} );
```

你现在能发现问题所在吗？仔细看！

序列阶序有一个竞争条件。当你`返回步骤`在那一刻`步骤`可以_是最初定义的承诺链，或者现在可以通过_步骤=步骤。然后（…）`根据事情发生的顺序调用。`以下是两种可能的结果：

如果

-   步骤`仍然是原来的承诺链，一旦它后来被“扩展”`步骤=步骤。然后（…）`这个链条末端的扩展承诺是`不**考虑的**主要的`流，因为它已经打开了`步骤`链。这是不幸的限制。`急功近利的评价**。**如果
-   步骤`已经是扩展的承诺链，它就像我们期望的那样工作，扩展的承诺是什么？`主要的`水龙头。`除了一个明显的事实，即种族条件是不能容忍的，第一个案件是关注，它说明

急功近利的评价**承诺链。通过对比，我们很容易扩展的迭代序列没有这样的问题，因为迭代序列**懒洋洋地评价**。**你需要更多的动态流量控制，多个序列会闪耀。

提示：

**检查出更多的信息和例子的迭代序列**asynquence_网站（_http：/ / GitHub。COM / getify / asynquence /斑点/硕士/博士#迭代序列的自述。[）。](https://github.com/getify/asynquence/blob/master/README.md#iterable-sequences)事件反应

## 这应该是显而易见的（至少是！）3章，承诺在你的异步工具箱的一个非常强大的工具。但有一件事显然缺乏，那就是他们处理事件流的能力，因为承诺只能解决一次。坦率地说，这个完全相同的弱点是真实的。

asynquence_序列，以及。_考虑一个场景，在每次触发某个事件时，您都希望触发一系列步骤。单一的承诺或序列不能代表该事件的所有事件。所以，你必须创建一个全新的承诺链（或序列）

每个_事件发生，例如：_我们需要的基本功能在这种方法中存在，但它远不是表达我们预期逻辑的理想方法。有两个独立的功能合并在这一范式：事件监听，并响应事件；关注分离恳求我们找出这些能力。

```js
listener.on( "foobar", function(data){

	// create a new event handling promise chain
	new Promise( function(resolve,reject){
		// ..
	} )
	.then( .. )
	.then( .. );

} );
```

认真细心的读者会看到这个问题有点对称的2章我们详细的回调的问题；它是一种控制反演问题。

想象uninverting这个范例，像这样：

这个

```js
var observable = listener.on( "foobar" );

// later
observable
.then( .. )
.then( .. );

// elsewhere
observable
.then( .. )
.then( .. );
```

观察`价值并不完全是承诺，但你可以`观察_这很像你能观察到的诺言，所以它是密切相关的。事实上，它可以观察很多次，并且每次事件都会发出通知（_“foobar”`）发生。`提示：

**我刚才说明的这个模式是**大量简化**反应式编程（又名RP）背后的概念和动机，已经被几个伟大的项目和语言实现/阐述了。RP的变化是功能反应式编程（FRP），它是指运用函数编程技术（永恒性、参照完整性等）的数据流。”反应性是指随着时间的推移将此功能传播出去以响应事件。有兴趣的读者应该考虑研究“无量”的神奇“反应扩展库（“RxJS”由微软（JavaScript）**http&#x3A;//rxjs.codeplex.com/[）它比我刚才展示的更为复杂和强大。另外，Andre Staltz有一个很好写了（](http://rxjs.codeplex.com/)https&#x3A;//gist.github.com/staltz/868e7e9bc2a7b8c1f754[），务实，奠定了RP的具体例子。](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)ES7的观测

### 在写这篇文章的时候，有一个新的数据类型称为“早期ES7建议观察”（

http：/ / GitHub。COM / jhusain / asyncgenerator #引入观测[，在精神上类似于我们在这里所做的，但绝对是更复杂的。](https://github.com/jhusain/asyncgenerator#introducing-observable)这种可观察的概念是，从流中订阅事件的方式是传递一个生成器——实际上

迭代器_是利害关系方吗？_下一个（..）`将为每个事件调用方法。`你可以想象这有点像：

你通过的发电机

```js
// `someEventStream` is a stream of events, like from
// mouse clicks, and the like.

var observer = new Observer( someEventStream, function*(){
	while (var evt = yield) {
		console.log( evt );
	}
} );
```

产量`暂停`虽然`循环等待下一个事件。这个`迭代器_附加到生成器实例将具有它的_下一个（..）`“每一次`someeventstream`发布一个新事件，以便事件数据恢复您的生成器/`迭代器_与_EVT`数据。`在订阅事件功能这里，它是

迭代器_重要的部分，而不是发电机。所以在概念上你可以在几乎任何个通行证，包括_iterable() ASQ。`迭代序列。`有趣的是，还提出了适配器，以便于从某些类型的流构造可观测值，例如

fromevent（..）`用于DOM事件。如果您查看一个建议的实现`fromevent（..）`在早些时候与ES7的建议，它看起来像一个可怕的很多`一`ASQ。反应（..）`我们将在下一节中看到。

当然，这些都是早期的建议，所以很有可能看起来和这里的表现不同。但是看到不同的库和语言方案的概念的早期对齐是令人兴奋的！

### 活性序列

随着对观测值（和RP）的疯狂简要总结，作为我们的灵感和动机，我现在将说明一个小的“反应观测值”的子集，我称之为“反应序列”。

首先，让我们从如何创建可观察的开始，使用_asynquence_插件的工具称为`反应（…）`：

```js
var observable = ASQ.react( function setup(next){
	listener.on( "foobar", next );
} );
```

现在，让我们看看如何定义一个“反应”序列——在F／RP中，这通常称为“订阅”。`观察`：

```js
observable
.seq( .. )
.then( .. )
.val( .. );
```

因此，只需通过将可见的链接链接来定义序列。那很简单，是吧？

在F／RP中，事件流通常通过一组功能转换来传输，比如`扫描（..）`，`地图（…）`，`减少（…）`等等。有了反应序列，每个事件都通过序列的一个新实例来传输。让我们看一个更具体的例子：

```js
ASQ.react( function setup(next){
	document.getElementById( "mybtn" )
	.addEventListener( "click", next, false );
} )
.seq( function(evt){
	var btnID = evt.target.id;
	return request(
		"http://some.url.1/?id=" + btnID
	);
} )
.val( function(text){
	console.log( text );
} );
```

反应序列的“反应”部分来自指定一个或多个事件处理程序来调用事件触发器（调用）。`下一个（..）`）。

反应序列的“序列”部分与我们已经探索过的序列完全相同：每一步都可以是任何异步技术都有意义的，从连续回调到承诺到生成器。

一旦设置了一个反应序列，只要事件继续触发，它将继续启动序列实例。如果要停止反应序列，可以调用`stop()`。

如果反应序列是`stop()`“D，你可能希望事件处理程序（S）未登记以及登记；可以用于此目的的拆解处理：

```js
var sq = ASQ.react( function setup(next,registerTeardown){
	var btn = document.getElementById( "mybtn" );

	btn.addEventListener( "click", next, false );

	// will be called once `sq.stop()` is called
	registerTeardown( function(){
		btn.removeEventListener( "click", next, false );
	} );
} )
.seq( .. )
.then( .. )
.val( .. );

// later
sq.stop();
```

**注：**这个`这`内部引用绑定`设置（..）`处理程序是相同的`平方`反应序列，所以可以使用`这`引用添加到反应序列定义中，调用方法如下`stop()`等等。

这是从Node.js的世界的一个例子，利用反应序列来处理传入的HTTP请求：

```js
var server = http.createServer();
server.listen(8000);

// reactive observer
var request = ASQ.react( function setup(next,registerTeardown){
	server.addListener( "request", next );
	server.addListener( "close", this.stop );

	registerTeardown( function(){
		server.removeListener( "request", next );
		server.removeListener( "close", request.stop );
	} );
});

// respond to requests
request
.seq( pullFromDatabase )
.val( function(data,res){
	res.end( data );
} );

// node teardown
process.on( "SIGINT", request.stop );
```

这个`下一个（..）`触发器也可以很容易地适应节点流，使用`投产（..）`和`上游（..）`：

```js
ASQ.react( function setup(next){
	var fstream = fs.createReadStream( "/some/file" );

	// pipe the stream's "data" event to `next(..)`
	next.onStream( fstream );

	// listen for the end of the stream
	fstream.on( "end", function(){
		next.unStream( fstream );
	} );
} )
.seq( .. )
.then( .. )
.val( .. );
```

还可以使用序列组合来组成多个反应序列流：

```js
var sq1 = ASQ.react( .. ).seq( .. ).then( .. );
var sq2 = ASQ.react( .. ).seq( .. ).then( .. );

var sq3 = ASQ.react(..)
.gate(
	sq1,
	sq2
)
.then( .. );
```

主要的外卖是`ASQ。反应（..）`这是一个轻量级的F／RP概念的适应，使事件流连接到一个序列，因此术语“反应序列”。

**注：**下面是一个使用的例子`ASQ。反应（..）`管理UI状态（[http://jsbin.com/rozipaki/6/edit？JS输出](http://jsbin.com/rozipaki/6/edit?js,output)）以及另一个处理HTTP请求/响应流的示例`ASQ。反应（..）`（<https://gist.github.com/getify/bba5ec0de9d6047b720e>）。

## 发电机协同

希望4章帮助你获得6发电机很熟悉。特别是，我们希望重新讨论“生成器并发”的讨论，并进一步推进它。

我们想象的`奔跑（..）`可以同时生成两个或多个生成器并同时运行它们的实用程序。`产量`控件从一个到另一个，可选的消息传递。

除了能够运行一台发电机完成，`ASQ #转轮（..）`我们在附录A中讨论的是类似概念的实现。`奔跑（..）`，它可以同时运行多个生成器完成。

让我们看看如何从第4章实现并发Ajax场景：

```js
ASQ(
	"http://some.url.2"
)
.runner(
	function*(token){
		// transfer control
		yield token;

		var url1 = token.messages[0]; // "http://some.url.1"

		// clear out messages to start fresh
		token.messages = [];

		var p1 = request( url1 );

		// transfer control
		yield token;

		token.messages.push( yield p1 );
	},
	function*(token){
		var url2 = token.messages[0]; // "http://some.url.2"

		// message pass and transfer control
		token.messages[0] = "http://some.url.1";
		yield token;

		var p2 = request( url2 );

		// transfer control
		yield token;

		token.messages.push( yield p2 );

		// pass along results to next sequence step
		return token.messages;
	}
)
.val( function(res){
	// `res[0]` comes from "http://some.url.1"
	// `res[1]` comes from "http://some.url.2"
} );
```

两者之间的主要区别`ASQ #转轮（..）`和`奔跑（..）`如下：

-   每个发生器（可选）提供了一种说法叫`令牌`，这是对`产量`当你想明确地转移到下一个访问控制。
-   `token.messages`是一个数组，用于保存从上一个序列步骤传入的任何消息。这也是一个数据结构，你可以使用共享的信息之间的协同。
-   `产量`一个承诺（或序列）值不转移控制，而是暂停协同处理直到值准备。
-   最后`返回`ED或`产量`ED值从协同处理运行将向前传递到序列中的下一步。

它也很容易在顶部的基础层助手。`ASQ #转轮（..）`适合不同用途的功能。

### 状态机

许多程序员可能熟悉的一个例子是状态机。你可以借助一个简单的化妆工具，创建一个易于表达的状态机处理器。

让我们想象这样一个工具。我们称之为`国家（…）`并将传递两个参数：一个状态值和一个处理该状态的生成器。`国家（…）`将完成创建和返回适配器生成器传递给`ASQ #转轮（..）`。

考虑：

```js
function state(val,handler) {
	// make a coroutine handler for this state
	return function*(token) {
		// state transition handler
		function transition(to) {
			token.messages[0] = to;
		}

		// set initial state (if none set yet)
		if (token.messages.length < 1) {
			token.messages[0] = val;
		}

		// keep going until final state (false) is reached
		while (token.messages[0] !== false) {
			// current state matches this handler?
			if (token.messages[0] === val) {
				// delegate to state handler
				yield *handler( transition );
			}

			// transfer control to another state handler?
			if (token.messages[0] !== false) {
				yield token;
			}
		}
	};
}
```

如果你仔细看，你就会看到`国家（…）`返回一个接受`令牌`然后设置一个`虽然`环将运行至状态机达到其最终状态（我们任意`假`值），这正是我们要传递的生成器类型。`ASQ #转轮（..）`！

我们也任意保留`令牌[消息] [ 0 ]`插槽作为我们的状态机的当前状态将被跟踪的地方，这意味着我们甚至可以将初始状态作为从序列的前一步传递的值。

我们如何使用`国家（…）`帮手随着`ASQ #转轮（..）`？

```js
var prevState;

ASQ(
	/* optional: initial state value */
	2
)
// run our state machine
// transitions: 2 -> 3 -> 1 -> 3 -> false
.runner(
	// state `1` handler
	state( 1, function *stateOne(transition){
		console.log( "in state 1" );

		prevState = 1;
		yield transition( 3 );	// goto state `3`
	} ),

	// state `2` handler
	state( 2, function *stateTwo(transition){
		console.log( "in state 2" );

		prevState = 2;
		yield transition( 3 );	// goto state `3`
	} ),

	// state `3` handler
	state( 3, function *stateThree(transition){
		console.log( "in state 3" );

		if (prevState === 2) {
			prevState = 3;
			yield transition( 1 ); // goto state `1`
		}
		// all done!
		else {
			yield "That's all folks!";

			prevState = 3;
			yield transition( false ); // terminal state
		}
	} )
)
// state machine complete, so move on
.val( function(msg){
	console.log( msg );	// That's all folks!
} );
```

重要的是要注意`* stateone（..）`，`* statetwo（..）`，和`* statethree（..）`发电机本身重新调用每次进入状态，他们就结束了`过渡（…）`另一种价值。虽然这里没有显示，当然这些状态生成器处理程序可以异步地暂停。`产量`ING承诺/序列/这个。

下面隐藏的发电机所产生的`国家（…）`助手并实际传递给`ASQ #转轮（..）`是继续运行的状态机的长度，并且它们各自协同处理。`产量`ING控制下一个，等等。

**注：**看这个“乒乓球”的例子（[http://jsbin.com/qutabu/1/edit？JS输出](http://jsbin.com/qutabu/1/edit?js,output)）更多地说明如何使用由驱动的`ASQ #转轮（..）`。

## 通信顺序进程（CSP）

“通信顺序进程”（CSP）是由1978的学术论文，首次描述托尼·霍尔（[http://dl.acm.org/citation.cfm？变化呈现单向= 359576.359585](http://dl.acm.org/citation.cfm?doid=359576.359585)），后来在1985本书中。<http://www.usingcsp.com/>）同名的。CSP描述了一种在处理过程中并发的进程交互（也称为“通信”）的形式化方法。

您可能记得我们在第1章回顾了并发的“过程”，因此我们对CSP的探索将建立在这种理解之上。

像计算机科学中的大多数伟大概念一样，CSP大量沉浸在学术形式主义中，用过程代数表示。然而，我怀疑符号代数定理不会对读者造成太大的影响，所以我们希望找到另一种包装我们大脑的方法。

我会多给霍尔的写作形式化描述和证明CSP离开，和许多其他精彩的作品自。相反，我们将尽可能简单地解释CSP的概念，作为联合国学术界，并尽可能直观地理解一种可能的方式。

### 消息传递

CSP的核心原则是所有独立进程之间的所有通信/交互都必须通过正式的消息传递。也许与您的期望相反，CSP消息传递被描述为同步动作，发送方进程和接收方进程必须相互准备好传递消息。

这样的同步消息怎么可能与JavaScript中的异步编程相关呢？

关系的具体性来自于自然的ES6生成器用来产生同步动作，在被子下的确可以是同步或异步（更可能）。

换句话说，两个或多个并发运行的生成器可以同时同步消息，同时保持系统的基本异步性，因为等待异步动作恢复时，每个生成器的代码被暂停（也称为“阻塞”）。

这是怎么工作的？

设想一个称为“A”的生成器（也称为“进程”），希望将消息发送给生成器“B”，首先，“a”`产量`当“B”准备好并接收消息时，消息（暂停“A”）被发送到“B”，然后“A”被恢复（畅通无阻）。

对称地想象一个生成器“A”想要一个消息。**从**“B”“A”`产量`s的请求（因此暂停“A”）来自“B”的消息，一旦“B”发送消息，“A”接收消息并恢复。

这一消息传递理论CSP更受欢迎的表达来自clojurescript的core.async图书馆，也从_去_语言。这些连接CSP体现了在一个称为“通道”的进程之间打开的管道中所描述的通信语义。

**注：**术语_通道_部分使用，因为有一种模式，其中一个以上的值可以立即发送到通道的“缓冲区”；这与您所认为的流类似。我们不会深入研究它，但它可以是一种非常强大的数据流管理技术。

在CSP最简单的概念中，我们在“A”和“B”之间创建的通道将有一个称为“方法”的方法。`take(..)`用于阻塞接收值和调用方法`放（…）`用于阻塞发送值。

这可能看起来像：

```js
var ch = channel();

function *foo() {
	var msg = yield take( ch );

	console.log( msg );
}

function *bar() {
	yield put( ch, "Hello World" );

	console.log( "message sent" );
}

run( foo );
run( bar );
// Hello World
// "message sent"
```

将这种结构化的、同步的（查看）消息传递交互与非正式的和非结构化的消息共享进行比较。`ASQ #转轮（..）`提供了通过`token.messages`阵列和合作`产量`惯性导航与制导.在本质上，`产量（…）`是一个单独的操作，它都发送值并暂停执行转移控制，而在前面的例子中，我们将它们作为单独的步骤进行操作。

此外，CSP强调你并没有明确地“传递控制”，而是说

**注：**公平警告：这种模式是非常强大的，但它也有点扭曲，以适应在第一次。您将需要练习这一点，以适应这种关于协调并发性的新思路。

有几个伟大的库在JavaScript中实现了这种CSP的味道，尤其是“js CSP”（<https://github.com/ubolonton/js-csp>）这是James Long（<http://twitter.com/jlongster>）叉（<https://github.com/jlongster/js-csp>并广泛著述（[http://jlongster.com/taming-the-asynchronous-beast-with-csp-in-javascript](http://jlongster.com/Taming-the-Asynchronous-Beast-with-CSP-in-JavaScript)）。另外，不能强调不够多么神奇的许多著作的David Nolen（<http://twitter.com/swannodette>）是适应clojurescript去风格core.async CSP为JS发电机的话题（<http://swannodette.github.io/2013/08/24/es6-generators-and-csp>）。

### asynquence CSP仿真

因为我们已经讨论了异步模式在这里在我的语境_asynquence_库，您可能会感兴趣地看到，我们可以很容易地添加一个仿真层的顶部`ASQ #转轮（..）`生成器处理作为CSP和行为的近乎完美的移植。这种仿真层船作为一个可选的部分“asynquence contrib包旁边_asynquence_。

非常相似`国家（…）`早期助手，`ASQ。CSP。去（..）`以发电机--去/ core.async而言，它被称为一个goroutine里--适应使用`ASQ #转轮（..）`通过返回一个新的生成器。

而不是通过`令牌`，你收到最初创建通道（goroutine里`中国`下面，在运行）概念将分享。您可以创建更多的通道（这通常非常有用！）具有`ASQ。CSP。陈（..）`。

在CSP模型，我们在阻塞信道信息方面都不同步，而不是阻塞等待一个诺言/序列/咚完成。

因此，而不是`产量`这个诺言从`请求（…）`，`请求（…）`应该返回你的频道`拿（…）`值从。换句话说，在这个上下文/用法中，一个单值通道大致相当于一个承诺/序列。

让我们先做一个通道感知的版本`request(..)`：

```js
function request(url) {
	var ch = ASQ.csp.channel();
	ajax( url ).then( function(content){
		// `putAsync(..)` is a version of `put(..)` that
		// can be used outside of a generator. It returns
		// a promise for the operation's completion. We
		// don't use that promise here, but we could if
		// we needed to be notified when the value had
		// been `take(..)`n.
		ASQ.csp.putAsync( ch, content );
	} );
	return ch;
}
```

3章，“promisory”的承诺产生效用，“thunkory”4章是一个thunk的生产工具，最后，在附录A中我们发明了“sequory“序列产生效用。

当然，我们需要在这里为一个信道生成实用程序提供一个对称项。所以我们也叫它“chanory”（“通道”+“工厂”）。作为读者的练习，请尝试定义一个`渠化（..）`类似的工具`承诺，包装（…）`/`promisify（..）`（第3章），`thunkify（..）`（第4章）和`ASQ。包（..）`（附录A）。

现在考虑使用Ajax的并发示例_asyquence_加味CSP：

```js
ASQ()
.runner(
	ASQ.csp.go( function*(ch){
		yield ASQ.csp.put( ch, "http://some.url.2" );

		var url1 = yield ASQ.csp.take( ch );
		// "http://some.url.1"

		var res1 = yield ASQ.csp.take( request( url1 ) );

		yield ASQ.csp.put( ch, res1 );
	} ),
	ASQ.csp.go( function*(ch){
		var url2 = yield ASQ.csp.take( ch );
		// "http://some.url.2"

		yield ASQ.csp.put( ch, "http://some.url.1" );

		var res2 = yield ASQ.csp.take( request( url2 ) );
		var res1 = yield ASQ.csp.take( ch );

		// pass along results to next sequence step
		ch.buffer_size = 2;
		ASQ.csp.put( ch, res1 );
		ASQ.csp.put( ch, res2 );
	} )
)
.val( function(res1,res2){
	// `res1` comes from "http://some.url.1"
	// `res2` comes from "http://some.url.2"
} );
```

消息传递交易两概念之间的URL字符串是很简单的。第一个goroutine里发出一个Ajax请求的第一个URL，并放置在响应`中国`通道。第二个goroutine里发出一个Ajax请求第二的URL，然后第一反应`RES1`关闭`中国`通道。在这一点上，两种反应`RES1`和`RES2`完成并准备好。

如果在`中国`在这个goroutine就结束运行的通道，他们将被传递到序列中的下一步。因此，通过消息（S）从最终的goroutine里，`放（…）`他们进入`中国`。如图所示，要避免那些最后的阻碍。`放（…）`S，我们切换`中国`通过设置它的缓冲模式`buffer_size`到`二`（默认：`零`）。

**注：**参见更多的使用示例_asynquence_风味CSP在这里（<https://gist.github.com/getify/e0d04f1f5aa24b1947ae>）。

## 回顾

承诺和生成器提供了基本的构建模块，在这个基础上我们可以构建更加复杂和有能力的异步。

_asynquence_有实施的工具_迭代序列_，_活性序列_（又名“观测值”），_并行协同_，甚至_CSP的概念_。

这些模式，再加上连续回调和承诺功能，给予_asynquence_一个强大的组合不同的异步功能，全部集成在一个干净的异步流控制的抽象：序列。
