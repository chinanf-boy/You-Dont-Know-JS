
# 你不知道JS：异步性能

# 附录A：_asynquence_图书馆

1章和2章进入相当典型的异步编程模式的细节，以及它们如何与回调通常解决。但我们也看到为什么回调是致命的能力有限，从而导致我们的章节3和4，承诺和发电机提供一个更坚实，可靠，合理的基础来建立你的不同步。

我引用了我自己的异步库。_asynquence_（<http://github.com/getify/asynquence>）--“异步”+“序列”=“asynquence”--在这几次书，我想现在简要解释它是如何工作的，为什么它的独特设计是重要的和有帮助的。

在接下来的附录中，我们将探讨一些先进的异步模式，但你可能想要一个图书馆让那些可口的很有用。我们将使用_asynquence_为了表达这些模式，你需要花点时间来了解图书馆。

_asynquence_显然不是唯一选择好的异步编码；当然还有在这个空间中许多伟大的图书馆。但_asynquence_的最佳结合所有这些模式为单库提供了一个独特的视角，而且是建立在一个基本的抽象的（异步）序列。

我的前提是，复杂的js程序常常需要各种不同的异步模式交织在一起，而这通常完全由每个开发人员来决定。而不是把两种或两种以上不同的异步库，侧重于不同方面的不同步性，_asynquence_结合成变异序列的步骤，只有一个核心库的学习和部署。

我相信价值是足够强大的_asynquence_以保证风格语义超级容易做到异步流控制编程，所以我们只专注于图书馆。

首先，我将解释一下后面的设计原则。_asynquence_然后，我们将说明它的API是如何与代码示例一起工作的。

## 序列抽象设计

理解_asynquence_从理解一个基本抽象开始：一个任务的任何一系列步骤，不管它们是同步的还是异步的，都可以被看作是一个“序列”。换句话说，一个序列是一个容器，代表一个任务，是由个人（可能是异步的）步骤完成任务。

序列中的每一步都是根据承诺控制的（参见第3章）。也就是说，在序列中添加的每一步都隐式地创建了连接到序列的前一个末端的承诺。由于承诺的语义，即使在同步完成步骤时，序列中的每个单步推进都是异步的。

此外，一个序列总是从一步到一步线性地进行，这意味着步骤2总是在步骤1结束之后出现，等等。

当然，一个新的序列可以从现有的序列中分离出来，这意味着只要主序列到达流中的那个点，就只发生分叉。序列也可以结合不同的方式，包括有一个序列被另一个序列在流中的某一点。

一个序列有点像一个承诺链。然而，对于有希望的链，没有“句柄”来抓取引用整个链。任何一个你有一个引用的承诺只代表了链中的当前步骤，还有其他步骤。本质上，除非您对链中的第一个承诺持有引用，否则不能持有对承诺链的引用。

很多情况下，使用一个句柄来引用整个序列是非常有用的。这些情况中最重要的是顺序中止/取消。正如我们3章涵盖广泛，承诺自己绝不可能被取消，因为这违反了基本的设计势在必行：外部性。

但序列没有这样的不变性原理设计，主要是因为序列不是传递未来价值的容器，需要不可变的值语义。因此，序列是处理中止/取消行为的适当抽象级别。_asynquence_序列可`abort()`在任何时候，序列都会停在那个点上，不去任何理由。

有更多的理由喜欢在承诺链的顶部进行序列抽象，用于流控制目的。

首先，保证链接是一个相当的手工工艺，可以得到相当乏味的一旦你开始创建和链接将横跨广袤的程序--这乏味可counterproductively劝阻开发商从哪里比较合适的场所使用的承诺。

抽象是为了减少样板和乏味，所以序列抽象是很好的解决了这个问题。有了承诺，你的注意力就放在个人的步骤上，而且几乎没有假设你会维持这个链条。机智

当您开始考虑高阶承诺模式时，这种抽象复杂性降低特别强大。`种族（[…]）`和`全部（[…] ]`。

例如，在序列的中间，您可能希望表达一个概念上类似于`试试。赶上`在这一步总是会带来成功，无论是预期的主要成功分辨率或正无差信号的捕获错误。或者，您可能想要表达一个类似于重试/循环的步骤，在这个循环中，它不断地尝试相同的步骤，直到成功发生为止。

这些类型的抽象非常简单，只使用承诺原语来表达，在现有的承诺链中间这样做并不漂亮。但是，如果你把你的思维抽象成一个序列，并把一步作为一个包装物来包装一个承诺，那么这个步骤包装器可以隐藏这样的细节，让你以最合理的方式思考流程控制，而不会被细节所打扰。

其次，也许更重要的是，在一个序列中的步骤异步流控制思想可以抽象出什么类型的异步性的细节所涉及的每个步骤。在覆盖层下，承诺总是控制步骤，但在覆盖层上，该步骤可以看起来像是一个连续回调（简单的默认），或者像一个真正的承诺，或者作为一个运行到完成生成器，或者…希望你能明白。

第三，序列更容易被扭曲，以适应不同的思维方式，如事件、流或基于反应的编码。_asynquence_提供了一种模式，我称之为“无序列”（我们将在以后）在RxJS对“无明显变化”思想（“反应扩展”），让一个重复事件火了一个新的序列，那么每次。承诺是只有一个镜头，所以很尴尬的表示重复的承诺就不同步。

另一种思维方式将分辨率/控制能力的一个模式，我称之为“迭代序列”。而不是每个单独的步骤内部控制自己的完成（从而提高序列），序列是倒置的，所以进步控制是通过一个外部迭代器，每一步在_迭代序列_只是回应`下一个（..）`迭代器_控制。_我们将在附录的其余部分中探究所有这些不同的变化，所以不要担心，如果我们刚才把这些位跑得太快了。

外卖是序列比复杂的异步更强大和明智的抽象比承诺（承诺链）或只是发电机，和

asynquence_的设计与恰到好处的糖使异步编程更容易理解和更愉快的表达抽象。_asynquence

## _美国石油学会_首先，您创建序列的方式（a

asynquence_实例）与_ASQ（..）`功能。一个`asq()`没有参数调用创建一个空的初始序列，而传递一个或多个值或函数`ASQ（..）`设置每个参数代表序列的初始步骤的序列。`注：

**为了这里所有代码示例的目的，我将使用**asynquence_全局浏览器使用中的顶级标识符：_ASQ`。如果包括和使用`asynquence_通过一个模块系统（浏览器或服务器），你当然可以定义你喜欢的符号，_asynquence_不在乎！_这里讨论的许多API方法都是建立在

asynquence_，但其他人都是通过包括可选的“贡献”插件包提供。请参见文档_asynquence_对于一个方法是否通过插件构建或定义：_http&#x3A;//github.com/getify/asynquence[步骤](http://github.com/getify/asynquence)

### 如果函数代表序列中的正常步骤，则调用该函数，第一个参数是连续回调，而后面的任何参数都是从上一步传递的任何消息。在调用连续回调之前，步骤将不完成。调用它后，传递给它的任何参数都会作为消息发送到序列的下一个步骤。

若要向序列添加额外的正常步骤，请调用

然后（…）`（基本上与`ASQ（..）`呼叫）：`注：

```js
ASQ(
	// step 1
	function(done){
		setTimeout( function(){
			done( "Hello" );
		}, 100 );
	},
	// step 2
	function(done,greeting) {
		setTimeout( function(){
			done( greeting + " World" );
		}, 100 );
	}
)
// step 3
.then( function(done,msg){
	setTimeout( function(){
		done( msg.toUpperCase() );
	}, 100 );
} )
// step 4
.then( function(done,msg){
	console.log( msg );			// HELLO WORLD
} );
```

**虽然这个名字**然后（…）`与本地承诺API相同，此`然后（…）`不同的是。您可以将少数或多个函数或值传递给`然后（…）`如你所愿，每一步都是单独的一步。没有两个回调实现/拒绝语义涉及。`不同于承诺，在哪里把一个承诺链到下一个你必须创造和

返回`从一个承诺`然后（…）`执行处理程序`asynquence_您所需要做的就是调用连续回调——我一直称之为_done()`但您可以将它命名为适合您的任何名称，并可以将它作为参数传递给完成消息。`定义的每一步

然后（…）`被假定为异步的。如果你有一个步骤T`被假定为异步的。如果你有一个同步的步骤，你可以直接调用`完成（…）`马上，或者你可以用更简单的。`瓦尔（…）`步帮手：

```js
// step 1 (sync)
ASQ( function(done){
	done( "Hello" );	// manually synchronous
} )
// step 2 (sync)
.val( function(greeting){
	return greeting + " World";
} )
// step 3 (async)
.then( function(done,msg){
	setTimeout( function(){
		done( msg.toUpperCase() );
	}, 100 );
} )
// step 4 (sync)
.val( function(msg){
	console.log( msg );
} );
```

正如你所看到的，`瓦尔（…）`调用的步骤不会收到连续回调，因为您假定了该部分——结果列表中的参数列表不那么混乱！要将消息发送到下一个步骤，您只需使用`返回`。

想想`瓦尔（…）`表示同步的“只值”步骤，它对同步值操作、日志记录等非常有用。

### 错误

一个重要的区别是_asynquence_与承诺相比，错误处理。

有了承诺，链中的每个个体承诺（步骤）都会有自己的独立错误，并且每个后续步骤都有处理错误的能力。这种语义的主要原因（再次）来自于对单个承诺的关注，而不是对整个链（序列）的关注。

我相信，大多数情况下，序列中某一部分的错误通常不可恢复，因此序列中的后续步骤是没有实际意义的，应该跳过。因此，默认情况下，一个序列的任何一个步骤的错误都会将整个序列抛出错误模式，而其余的正常步骤将被忽略。

如果你_做_需要有一个步骤，在它的错误是可恢复的，有几种不同的API方法，可以容纳，如`尝试（…）`-以前提到的一种`试试。赶上`一步或`直到（…）`——重试循环，直到尝试成功或手动完成为止。`break()`环。_asynquence_甚至有`p然后（..）`和`pcatch（..）`方法与正常承诺的方法相同`然后（…）`和`抓住（…）`工作（见第3章），如果您选择，可以进行本地化的中间序列错误处理。

重点是，您有两种选择，但在我的经验中更常见的是默认值。有了承诺，一旦出现错误，就可以得到一系列的步骤来忽略所有步骤，您必须注意不要在任何步骤中注册拒绝处理程序；否则，该错误将被接受处理，序列可能继续（可能出乎意料）。这种理想的行为在正确可靠地处理上有点尴尬。

注册序列错误通知处理程序，_asynquence_提供了一个`或（…）`序列方法，也有一个别名`OnError（..）`。可以在序列中任何地方调用此方法，并且可以注册任意多个处理程序。这使得多个不同的消费者很容易在一个序列中监听，知道它是否失败；在这方面它有点像一个错误事件处理程序。

就像承诺一样，所有js异常都会成为序列错误，或者您可以以编程方式发出一个序列错误：

```js
var sq = ASQ( function(done){
	setTimeout( function(){
		// signal an error for the sequence
		done.fail( "Oops" );
	}, 100 );
} )
.then( function(done){
	// will never get here
} )
.or( function(err){
	console.log( err );			// Oops
} )
.then( function(done){
	// won't get here either
} );

// later

sq.or( function(err){
	console.log( err );			// Oops
} );
```

另一个与错误处理非常重要的区别_asynquence_相比于本土的承诺是默认的行为“未处理的例外”。正如我们在第3章中详细讨论过的，没有注册拒绝处理程序的拒绝承诺只会默默地持有（也就是燕子）的错误；您必须记住始终以一个终结结束一个链。`抓住（…）`。

在_asynquence_这个假设是相反的。

如果序列上出现错误，则**在那一刻**没有注册错误处理程序，则将错误报告给`慰问`。换句话说，未经处理的拒绝都是默认总是报道以免被吞下去的，错过了。

只要你登记对序列错误处理程序，它选择序列出这样的报告，以避免重复的噪音。

事实上，在您有机会注册处理程序之前，您可能希望创建一个序列，该序列可能进入错误状态。这并不常见，但有时会发生。

在这种情况下，你也可以**选择序列实例**通过调用进行错误报告`defer()`按顺序。如果您确信最终将处理此类错误，则只能选择不提交错误报告：

```js
var sq1 = ASQ( function(done){
	doesnt.Exist();			// will throw exception to console
} );

var sq2 = ASQ( function(done){
	doesnt.Exist();			// will throw only a sequence error
} )
// opt-out of error reporting
.defer();

setTimeout( function(){
	sq1.or( function(err){
		console.log( err );	// ReferenceError
	} );

	sq2.or( function(err){
		console.log( err );	// ReferenceError
	} );
}, 100 );

// ReferenceError (from sq1)
```

这是比承诺本身更好的错误处理行为，因为它是成功的陷阱，而不是失败的陷阱（参见第3章）。

**注：**如果一个序列是输送进（又名归入）另一个序列——见“组合序列”一个完整的描述，那么源序列选择了错误报告，但现在的靶序列的错误报告或缺乏必须考虑。

### 并行的步骤

不是所有的你的序列步骤将只有一个（异步）任务执行；一些需要执行多个步骤的“并行”（兼）。一步一个序列中的多个步骤处理的同时被称为`大门（…）`-有一个`全部（…）`别名，如果您喜欢-直接与本地对称`承诺，一切（[…]）`。

如果所有的步骤`大门（…）`成功完成，所有成功消息将传递给下一个序列步骤。如果其中任何一个产生错误，整个序列立即进入错误状态。

考虑：

```js
ASQ( function(done){
	setTimeout( done, 100 );
} )
.gate(
	function(done){
		setTimeout( function(){
			done( "Hello" );
		}, 100 );
	},
	function(done){
		setTimeout( function(){
			done( "World", "!" );
		}, 100 );
	}
)
.val( function(msg1,msg2){
	console.log( msg1 );	// Hello
	console.log( msg2 );	// [ "World", "!" ]
} );
```

为了说明，让我们比较一下

```js
new Promise( function(resolve,reject){
	setTimeout( resolve, 100 );
} )
.then( function(){
	return Promise.all( [
		new Promise( function(resolve,reject){
			setTimeout( function(){
				resolve( "Hello" );
			}, 100 );
		} ),
		new Promise( function(resolve,reject){
			setTimeout( function(){
				// note: we need a [ ] array here
				resolve( [ "World", "!" ] );
			}, 100 );
		} )
	] );
} )
.then( function(msgs){
	console.log( msgs[0] );	// Hello
	console.log( msgs[1] );	// [ "World", "!" ]
} );
```

讨厌.承诺需要更多的样板的开销来表达相同的异步流控制。这是一个很好的例证。_asynquence_API和抽象使处理承诺的步骤变得更好。改进程度越高，你的异步性就越复杂。

#### 步的变化

有几个变化的贡献插件_asynquence_的`大门（…）`可以非常有用的步骤类型：

-   `任何（…）`就像`大门（…）`除了一个片段必须最终成功地继续主序。
-   `首先（…）`就像`任何（…）`除了任何片段成功后，主序列继续进行（忽略其他片段的后续结果）。
-   `种族（…）`（对称`承诺，比赛（[…]）`）像`首先（…）`除主序列一段完成后（成功或失败）。
-   `最后（…）`就像`任何（…）`除了成功完成的最新部分外，它还将消息发送到主序列中。
-   `没有（…）`是逆的`大门（…）`主序列仅在所有段失败时才进行（所有段错误消息被转换为成功消息，反之亦然）。

让我们先定义一些帮助者使插图更清晰：

```js
function success1(done) {
	setTimeout( function(){
		done( 1 );
	}, 100 );
}

function success2(done) {
	setTimeout( function(){
		done( 2 );
	}, 100 );
}

function failure3(done) {
	setTimeout( function(){
		done.fail( 3 );
	}, 100 );
}

function output(msg) {
	console.log( msg );
}
```

现在，让我们演示一下`大门（…）`步的变化：

```js
ASQ().race(
	failure3,
	success1
)
.or( output );		// 3


ASQ().any(
	success1,
	failure3,
	success2
)
.val( function(){
	var args = [].slice.call( arguments );
	console.log(
		args		// [ 1, undefined, 2 ]
	);
} );


ASQ().first(
	failure3,
	success1,
	success2
)
.val( output );		// 1


ASQ().last(
	failure3,
	success1,
	success2
)
.val( output );		// 2

ASQ().none(
	failure3
)
.val( output )		// 3
.none(
	failure3
	success1
)
.or( output );		// 1
```

另一步变化是`地图（…）`，它允许异步地将数组元素映射到不同的值，并且在所有映射完成之前不进行该步骤。`地图（…）`非常相似`大门（…）`除非它从数组中获取初始值，而不是从单独指定的函数中获取值，还因为定义了一个函数回调函数来对每个值进行操作：

```js
function double(x,done) {
	setTimeout( function(){
		done( x * 2 );
	}, 100 );
}

ASQ().map( [1,2,3], double )
.val( output );					// [2,4,6]
```

也,`地图（…）`可以从上一步传递的消息中接收其中一个参数（数组或回调）：

```js
function plusOne(x,done) {
	setTimeout( function(){
		done( x + 1 );
	}, 100 );
}

ASQ( [1,2,3] )
.map( double )			// message `[1,2,3]` comes in
.map( plusOne )			// message `[2,4,6]` comes in
.val( output );			// [3,5,7]
```

另一个变化是`瀑布（..）`这有点像混合在一起。`大门（…）`的消息收集行为，但`然后（…）`顺序处理。

步骤1首先执行，然后从步骤1向步骤2提供成功消息，然后两个成功消息转到步骤3，然后所有三个成功消息转到步骤4，等等，以便消息收集并级联到“瀑布”。

考虑：

```js
function double(done) {
	var args = [].slice.call( arguments, 1 );
	console.log( args );

	setTimeout( function(){
		done( args[args.length - 1] * 2 );
	}, 100 );
}

ASQ( 3 )
.waterfall(
	double,					// [ 3 ]
	double,					// [ 6 ]
	double,					// [ 6, 12 ]
	double					// [ 6, 12, 24 ]
)
.val( function(){
	var args = [].slice.call( arguments );
	console.log( args );	// [ 6, 12, 24, 48 ]
} );
```

如果在“瀑布”中的任何一个点出现错误，则整个序列立即进入错误状态。

#### 容错

有时您希望在步骤级管理错误，而不是让它们将整个序列发送到错误状态。_asynquence_为此提供两个步骤的变体。

`尝试（…）`尝试的一步，如果它成功了，序列正常的收益，但如果一步失败，失败变成成功消息格式为`{ catch：…}`填充错误信息：

```js
ASQ()
.try( success1 )
.val( output )			// 1
.try( failure3 )
.val( output )			// { catch: 3 }
.or( function(err){
	// never gets here
} );
```

您可以使用`直到（…）`，试的步骤，如果失败，重试的步骤在下次事件循环滴答，等等。

这个重试循环可以无限期继续，但是如果你想跳出循环，你可以调用`break()`完成触发器上的标志，它将主序列发送到错误状态：

```js
var count = 0;

ASQ( 3 )
.until( double )
.val( output )					// 6
.until( function(done){
	count++;

	setTimeout( function(){
		if (count < 5) {
			done.fail();
		}
		else {
			// break out of the `until(..)` retry loop
			done.break( "Oops" );
		}
	}, 100 );
} )
.or( output );					// Oops
```

#### 承诺式的步骤

如果您希望在序列中内联，请像承诺一样保证样式语义`然后（…）`和`抓住（…）`（见第3章），您可以使用`p然后`和`pcatch`插件：

```js
ASQ( 21 )
.pThen( function(msg){
	return msg * 2;
} )
.pThen( output )				// 42
.pThen( function(){
	// throw an exception
	doesnt.Exist();
} )
.pCatch( function(err){
	// caught the exception (rejection)
	console.log( err );			// ReferenceError
} )
.val( function(){
	// main sequence is back in a
	// success state because previous
	// exception was caught by
	// `pCatch(..)`
} );
```

`p然后（..）`和`pcatch（..）`设计为按顺序运行，但表现为它是一个正常的承诺链。因此，你既可以解决真正的承诺，也可以_asynquence_传递到“完成”处理程序的序列`p然后（..）`（见第3章）。

### 分叉序列

对承诺非常有用的一个特性是可以附加多个。`然后（…）`处理程序注册到相同的承诺，有效的“分叉”，保证流量控制：

```js
var p = Promise.resolve( 21 );

// fork 1 (from `p`)
p.then( function(msg){
	return msg * 2;
} )
.then( function(msg){
	console.log( msg );		// 42
} )

// fork 2 (from `p`)
p.then( function(msg){
	console.log( msg );		// 21
} );
```

同样的“分叉”是容易的_asynquence_具有`fork()`：

```js
var sq = ASQ(..).then(..).then(..);

var sq2 = sq.fork();

// fork 1
sq.then(..)..;

// fork 2
sq2.then(..)..;
```

### 组合序列

反向的`fork()`，你可以通过归并到另一个相结合的两个序列，用`序列（..）`实例方法：

```js
var sq = ASQ( function(done){
	setTimeout( function(){
		done( "Hello World" );
	}, 200 );
} );

ASQ( function(done){
	setTimeout( done, 100 );
} )
// subsume `sq` sequence into this sequence
.seq( sq )
.val( function(msg){
	console.log( msg );		// Hello World
} )
```

`序列（..）`可以接受一个序列本身，如图所示，或函数。如果一个函数，预期函数在调用时将返回一个序列，所以前面的代码可以用：

```js
// ..
.seq( function(){
	return sq;
} )
// ..
```

而且，这一步可以用A完成。`烟斗（…）`：

```js
// ..
.then( function(done){
	// pipe `sq` into the `done` continuation callback
	sq.pipe( done );
} )
// ..
```

当一个序列将其成功消息流和错误流管道。

**注：**如前所述，管道（手动）`烟斗（…）`或自动`序列（..）`）选择源序列的错误报告，但并不影响错误报告目标序列的状态。

## 值和错误序列

如果一个序列的任何一步仅仅是一个正常值，那么这个值只映射到该步骤的完成消息：

```js
var sq = ASQ( 42 );

sq.val( function(msg){
	console.log( msg );		// 42
} );
```

如果你想使一个序列的自动错误：

```js
var sq = ASQ.failed( "Oops" );

ASQ()
.seq( sq )
.val( function(msg){
	// won't get here
} )
.or( function(err){
	console.log( err );		// Oops
} );
```

您还可能希望自动创建一个延迟值或一个延迟的错误序列。使用`之后`和`failafter`有插件，这是很容易的：

```js
var sq1 = ASQ.after( 100, "Hello", "World" );
var sq2 = ASQ.failAfter( 100, "Oops" );

sq1.val( function(msg1,msg2){
	console.log( msg1, msg2 );		// Hello World
} );

sq2.or( function(err){
	console.log( err );				// Oops
} );
```

还可以在序列中间插入一个延迟。`在…之后`：

```js
ASQ( 42 )
// insert a delay into the sequence
.after( 100 )
.val( function(msg){
	console.log( msg );		// 42
} );
```

## 承诺和回调

我想_asynquence_序列在原生承诺之上提供了大量的价值，在大多数情况下，您会发现在抽象层次上工作更愉快、更强大。然而，整合_asynquence_与其他非—_asynquence_代码将成为现实。

你可以很容易地把一个承诺（例如，thenable——见3章）为序列使用`许诺（…）`实例方法：

```js
var p = Promise.resolve( 42 );

ASQ()
.promise( p )			// could also: `function(){ return p; }`
.val( function(msg){
	console.log( msg );	// 42
} );
```

去相反的方向和叉/鬻从序列的承诺的一个步骤，使用`为了保证`有插件：

```js
var sq = ASQ.after( 100, "Hello World" );

sq.toPromise()
// this is a standard promise chain now
.then( function(msg){
	return msg.toUpperCase();
} )
.then( function(msg){
	console.log( msg );		// HELLO WORLD
} );
```

适应_asynquence_系统使用回调函数，有几个辅助设施。要自动从序列中生成一个“错误的第一样式”回调，以将其连接到回调导向的实用程序中，使用`errfcb`：

```js
var sq = ASQ( function(done){
	// note: expecting "error-first style" callback
	someAsyncFuncWithCB( 1, 2, done.errfcb )
} )
.val( function(msg){
	// ..
} )
.or( function(err){
	// ..
} );

// note: expecting "error-first style" callback
anotherAsyncFuncWithCB( 1, 2, sq.errfcb() );
```

你也可能想要创建一个序列包版本比较实用--“promisory”在3章和4章的“thunkory”--_asynquence_提供`ASQ。包（..）`为此目的：

```js
var coolUtility = ASQ.wrap( someAsyncFuncWithCB );

coolUtility( 1, 2 )
.val( function(msg){
	// ..
} )
.or( function(err){
	// ..
} );
```

**注：**为了清晰起见（为了好玩！）让我们再给另一个词，一个序列生成函数，来自`ASQ。包（..）`，像`coolutility`在这里.我提出“sequory”（“序列”+“工厂”）。

## 迭代序列

序列的范式是每一步都负责完成它自己，这就是推进序列的原因。承诺同样的方式。

不幸的是，有时你需要对诺言/步骤进行外部控制，这会导致尴尬的“能力提取”。

考虑这个承诺的例子：

```js
var domready = new Promise( function(resolve,reject){
	// don't want to put this here, because
	// it belongs logically in another part
	// of the code
	document.addEventListener( "DOMContentLoaded", resolve );
} );

// ..

domready.then( function(){
	// DOM is ready!
} );
```

有承诺的“能力提取”反模式看起来像这样：

```js
var ready;

var domready = new Promise( function(resolve,reject){
	// extract the `resolve()` capability
	ready = resolve;
} );

// ..

domready.then( function(){
	// DOM is ready!
} );

// ..

document.addEventListener( "DOMContentLoaded", ready );
```

**注：**在我看来，这种反模式是一种笨拙的代码味道，但一些开发人员喜欢它，原因是我无法掌握。

_asynquence_提供了一个倒置的序列类型，我称之为“迭代序列”，外部的控制能力（这是非常有用的在使用的情况下，如`Domready`）：

```js
// note: `domready` here is an *iterator* that
// controls the sequence
var domready = ASQ.iterable();

// ..

domready.val( function(){
	// DOM is ready
} );

// ..

document.addEventListener( "DOMContentLoaded", domready.next );
```

有个序列比我们所看到的这种情况更。我们将在附录B中回到他们那里。

## 运行的发电机

在第4章中，我们推导了一个名为`跑步（…）`它可以运行发电机完成，监听`产量`ED的承诺和使用它们来异步恢复发电。_asynquence_有这么一个公用设施，叫做`跑步者（…）`。

让我们首先设置一些帮助说明：

```js
function doublePr(x) {
	return new Promise( function(resolve,reject){
		setTimeout( function(){
			resolve( x * 2 );
		}, 100 );
	} );
}

function doubleSeq(x) {
	return ASQ( function(done){
		setTimeout( function(){
			done( x * 2)
		}, 100 );
	} );
}
```

现在，我们可以使用`跑步者（…）`作为序列中间的一个步骤：

```js
ASQ( 10, 11 )
.runner( function*(token){
	var x = token.messages[0] + token.messages[1];

	// yield a real promise
	x = yield doublePr( x );

	// yield a sequence
	x = yield doubleSeq( x );

	return x;
} )
.val( function(msg){
	console.log( msg );			// 84
} );
```

### 用发电机

您还可以创建一个自打包生成器，也就是一个正常的函数，它运行指定的生成器并返回一个完成的序列`ASQ。包（..）`它：

```js
var foo = ASQ.wrap( function*(token){
	var x = token.messages[0] + token.messages[1];

	// yield a real promise
	x = yield doublePr( x );

	// yield a sequence
	x = yield doubleSeq( x );

	return x;
}, { gen: true } );

// ..

foo( 8, 9 )
.val( function(msg){
	console.log( msg );			// 68
} );
```

还有很多可怕的东西`跑步者（…）`有能力，但我们会回到附录B中。

## 回顾

_asynquence_是一个简单的抽象——序列是一系列的（异步）--上承诺的步骤，旨在使各种异步模式更容易的工作，没有任何妥协的能力。

还有其他好吃的东西_asynquence_核心API和库插件以后我们在本附录中所看到的，但我们会离开，作为读者去检查其他的功能锻炼。

你现在看到了本质和精神_asynquence_。关键的一步是，一个序列由步骤组成，这些步骤可以是承诺上的几十种不同的变体，也可以是生成器运行，或者…选择了你，你把所有的自由编织在一起，不管异步控制流逻辑是适合你的工作。没有更多的图书馆不同的异步模式切换到。

如果这些_asynquence_片段对你来说是有意义的，你现在在图书馆的速度很快；实际上，学习并不需要那么多！

如果你仍然对它的工作方式有点模糊（或者为什么）！你会花更多的时间来检查以前的例子和玩。_asynquence_在下一个附录之前。附录B将推动_asynquence_几个先进和强大的异步模式。
