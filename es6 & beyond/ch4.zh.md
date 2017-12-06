
# 你不知道JS：ES6与超越

# 4章：异步流控制

如果您编写了大量JavaScript，异步编程是必需的技能，这就不是秘密了。管理异步的主要机制是函数回调。

然而，6增加了一个新的功能帮助解决的重大缺陷的唯一途径，异步回调：_承诺_。此外，我们可以重温发电机（从上一章）看那两向结合JavaScript异步流控制规划的重要一步模式。

## 承诺

让我们澄清一些误解：承诺是不更换回调。承诺提供一个值得信赖的中介，在你调用的代码和异步代码会执行任务——管理回调。

另一种考虑承诺的方式是作为事件侦听器，在该侦听器上，您可以注册侦听事件，让您知道任务何时完成。这是一个只有火一次的事件，但它可以被认为是一个事件，尽管如此。

承诺可以链接在一起，可以顺序的一系列步骤异步完成。再加上更高级别的抽象，比如`全部（…）`方法（经典术语，“门”）和`种族（…）`方法（经典术语中，“锁”），承诺链提供一个异步的流量控制机制。

另一种定义方式，这是一个承诺_未来价值_一个独立于时间的容器，围绕着一个值。这个容器可以完全相同地判断基础值是否为最终值。观察承诺的分辨率将提取该值。换句话说，一个承诺可以说是一个同步函数的返回值的异步版本。

承诺只能有两种可能的解决方案之一：满足或拒绝，具有可选的单个值。如果一个诺言实现了，最后的价值就叫做实现。如果被拒绝，最后的值被称为一个原因（如“拒绝原因”）。承诺只能得到解决（履行或拒绝）。_一旦_。任何进一步的实现或拒绝尝试都被忽略了。因此，一旦承诺得到解决，它是一个不变的价值，是无法改变的。

显然，有几种不同的方式来思考什么是承诺。没有单一的透视图是完全足够的，但是每一个透视图都提供了整体的一个单独的方面。最大的外卖是，他们提供了异步回调只有一个重大的改进，即他们提供订单，可预测性和信任。

### 制作和使用承诺

要构造一个允诺实例，使用`许诺（…）`构造函数：

```js
var p = new Promise( function pr(resolve,reject){
	// ..
} );
```

这个`许诺（…）`构造函数只接受一个函数（`公关（…）`，它立即被调用，并接收两个控制函数作为参数，通常称为`决心（…）`和`拒绝（…）`。它们被用作：

-   如果你打电话`拒绝（…）`承诺被拒绝，如果任何值被传递到`拒绝（…）`，它被设置为拒绝的原因。
-   如果你打电话`决心（…）`没有价值或任何不承诺价值，承诺就实现了。
-   如果你打电话`决心（…）`并且通过另一个承诺，这个承诺简单地采用了国家——无论是直接的还是最终的——对已通过的承诺（履行或拒绝）。

这里是你如何能保证重构通常使用回调函数调用的依赖。如果你开始使用`ajax(..)`希望能够调用错误第一样式回调的实用程序：

```js
function ajax(url,cb) {
	// make request, eventually call `cb(..)`
}

// ..

ajax( "http://some.url.1", function handler(err,contents){
	if (err) {
		// handle ajax error
	}
	else {
		// handle `contents` success
	}
} );
```

你可以把它转换成：

```js
function ajax(url) {
	return new Promise( function pr(resolve,reject){
		// make request, eventually call
		// either `resolve(..)` or `reject(..)`
	} );
}

// ..

ajax( "http://some.url.1" )
.then(
	function fulfilled(contents){
		// handle `contents` success
	},
	function rejected(reason){
		// handle ajax error reason
	}
);
```

有一种承诺`然后（…）`接受一个或两个回调函数的方法。如果成功地完成了承诺，第一个函数（如果存在）被视为处理程序来调用。如果显式拒绝了承诺，或者在解决过程中捕获了任何错误/异常，则第二个函数（如果存在）被视为处理程序来调用。

如果其中一个参数被省略或不是一个有效的函数——通常您将使用`无效的`而是使用默认占位符相等。默认成功回调将传递其执行值，默认错误回调将传播其拒绝原因。

打电话的速记法`然后（null，handlerejection）`是`抓住（handlerejection）`。

两`然后（…）`和`抓住（…）`自动构造并返回另一个承诺实例，该实例从接收到的承诺或拒绝处理程序（实际调用的任何一个）返回的值都是有线接收的。考虑：

```js
ajax( "http://some.url.1" )
.then(
	function fulfilled(contents){
		return contents.toUpperCase();
	},
	function rejected(reason){
		return "DEFAULT VALUE";
	}
)
.then( function fulfilled(data){
	// handle data from original promise's
	// handlers
} );
```

在这个片段中，我们将返回一个直接值。`满足（…）`或`拒绝（…）`然后在下一个事件的第二个事件中接收`然后（…）`的`满足（…）`。如果我们返回一个新的承诺，新的承诺将采用的分辨率：

```js
ajax( "http://some.url.1" )
.then(
	function fulfilled(contents){
		return ajax(
			"http://some.url.2?v=" + contents
		);
	},
	function rejected(reason){
		return ajax(
			"http://backup.url.3?err=" + reason
		);
	}
)
.then( function fulfilled(contents){
	// `contents` comes from the subsequent
	// `ajax(..)` call, whichever it was
} );
```

重要的是要注意第一个例外（或拒绝的承诺）。`满足（…）`将_不_结果在第一`rejected(..)`被调用，因为该处理程序只对第一个原始PR的解析作出响应。`然后（…）`被拒绝，受到拒绝。

在这段以前的片段中，我们并没有听到这种拒绝，这意味着它将被默默地保留下来以备将来观察。如果你从来没有观察过它`然后（…）`或`抓住（…）`，然后将未处理的。一些浏览器开发者控制台可以检测到这些未经处理的抑制和报告他们，但这是不可靠的保证；你应该遵守承诺的拒绝。

**注：**这只是对承诺理论和行为的简要概述。要进行更深入的探索，请参阅第3章_异步和性能_本系列的标题。

### thenables

承诺是真实的事例。`许诺（…）`构造函数。然而，有类似于承诺的对象被称为_thenables_一般可互操作的承诺机制。

任何对象（或函数）带有`然后（…）`功能上，它被认为是一个thenable。任何地方的承诺机制可以接受和采纳的一个真正的承诺的国家，他们也能处理thenable。

thenables基本上任何诺言的价值，可能有一些其他的系统比实际的通用标签`许诺（…）`构造函数。在这个角度来看，一个thenable比真正的承诺通常是不可信的。考虑这一行为不端的thenable，例如：

```js
var th = {
	then: function thener( fulfilled ) {
		// call `fulfilled(..)` once every 100ms forever
		setInterval( fulfilled, 100 );
	}
};
```

如果你收到了，并把它与thenable`然后（…）`你可能会惊讶的发现，你的满足处理程序被反复调用，当正常的承诺被认为只有一次被解决的时候。

一般来说，如果你收到什么据称是承诺或thenable从其他系统后，你不应该只是盲目的信任。在下一节中，我们会看到一个效用包括6承诺帮助解决这个信任问题。

但是，为了进一步了解这个问题的危险，请考虑_任何_对象_任何_被定义为在它上面有一个方法的一段代码`然后（…）`可作为一个thenable --如果用承诺潜在的困惑，当然，不管那东西是永远的目的甚至是承诺式异步编码相关。

前6，从来没有在任何特殊的保留方法`然后（…）`并且，正如你所想象的，至少有几个案例，在出现在雷达屏幕上的承诺之前，已经选择了方法名。的错误thenable最可能的情况将异步库使用`然后（…）`但这并不是严格遵守诺言——在野外有好几个例外。

责任在你防范直接使用价值的承诺机制，会被误认为是一个thenable。

### `承诺`美国石油学会

这个`承诺`API还提供了一些处理承诺的静态方法。

`承诺，决心（…）`创建一个解析为传递的值的承诺。让我们来比较它如何工作，以更手动的方法：

```js
var p1 = Promise.resolve( 42 );

var p2 = new Promise( function pr(resolve){
	resolve( 42 );
} );
```

`P1`和`P2`行为基本相同。用诺言解决同样的问题：

```js
var theP = ajax( .. );

var p1 = Promise.resolve( theP );

var p2 = new Promise( function pr(resolve){
	resolve( theP );
} );
```

**提示：**承诺，决心（…）`是的thenable信任问题在上一节中提出的解决方案。任何价值，你是不是已经确定是一个值得信赖的承诺——即使它可能是一个直接的价值可以通过归一化`承诺，决心（…）`。如果该值已经是一个公认的承诺或thenable，其状态/分辨率将被采纳，绝缘你行为不端。如果它不是立竿见影的价值，将它“包裹”在一个真正的承诺，从而规范其行为是异步的。`承诺，拒绝（…）

`创建一个立即被拒绝的承诺，和它的一样`许诺（…）`构造函数对应：`而

```js
var p1 = Promise.reject( "Oops" );

var p2 = new Promise( function pr(resolve,reject){
	reject( "Oops" );
} );
```

决心（…）`和`承诺，决心（…）`可以接受承诺并通过其状态/决议，`拒绝（…）`和`承诺，拒绝（…）`不要区分它们收到的值。所以，如果你拒绝承诺或thenable，诺言thenable本身将被设置为拒绝的理由，而不是它的潜在价值。`承诺（全部）]）

`接受一个或多个值的数组（例如，直接的价值承诺，thenables）。它返回一个承诺，如果所有的值完成，或者一旦它们中的第一个拒绝，立即返回。`从这些价值观/承诺开始：

让我们考虑一下如何

```js
var p1 = Promise.resolve( 42 );
var p2 = new Promise( function pr(resolve){
	setTimeout( function(){
		resolve( 43 );
	}, 100 );
} );
var v3 = 44;
var p4 = new Promise( function pr(resolve,reject){
	setTimeout( function(){
		reject( "Oops" );
	}, 10 );
} );
```

承诺（全部）]）`与这些值的组合一起工作：`而

```js
Promise.all( [p1,p2,v3] )
.then( function fulfilled(vals){
	console.log( vals );			// [42,43,44]
} );

Promise.all( [p1,p2,v3,p4] )
.then(
	function fulfilled(vals){
		// never gets here
	},
	function rejected(reason){
		console.log( reason );		// Oops
	}
);
```

承诺（全部）]）`等待所有的实践（或第一个拒绝），`承诺（赛跑）]）`只等待第一次实现或拒绝。考虑：`警告：

```js
// NOTE: re-setup all test values to
// avoid timing issues misleading you!

Promise.race( [p2,p1,v3] )
.then( function fulfilled(val){
	console.log( val );				// 42
} );

Promise.race( [p2,p4] )
.then(
	function fulfilled(val){
		// never gets here
	},
	function rejected(reason){
		console.log( reason );		// Oops
	}
);
```

**而**承诺（\[…]）`马上就完成（没有价值观），`承诺（赛跑）`将永远挂起。这是一个奇怪的矛盾，并建议您不要用空数组使用这些方法。`发电机+承诺

## 它

是_可能表达一系列链条中的承诺代表你的程序的异步流控制。考虑：_然而，有表达异步流量控制的一种更好的选择，它可能会更加preferab

```js
step1()
.then(
	step2,
	step1Failed
)
.then(
	function step3(msg) {
		return Promise.all( [
			step3a( msg ),
			step3b( msg ),
			step3c( msg )
		] )
	}
)
.then(step4);
```

然而，有表达异步流量控制的一种更好的选择，它可能是更可取的编码风格方面的承诺比长链。我们可以利用我们所学的3章发电机来表达我们的异步流控制。

认识到的重要模式：一个发电机可以产生一个承诺，然后这个承诺可以连接到恢复发电机的实现价值。

考虑前一段的异步流控制与发电机的表达：

```js
function *main() {

	try {
		var ret = yield step1();
	}
	catch (err) {
		ret = yield step1Failed( err );
	}

	ret = yield step2( ret );

	// step 3
	ret = yield Promise.all( [
		step3a( ret ),
		step3b( ret ),
		step3c( ret )
	] );

	yield step4( ret );
}
```

从表面上看，这个片段可能比前一段代码中的允诺链更为冗长。然而，它提供了一个更具吸引力，更重要的是，一个更易于理解和理智的-同步查找编码风格（与`=`赋值“返回值”等，尤其如此`试试。赶上`错误处理可以在那些隐藏的异步边界使用。

为什么我们要使用生成器的承诺呢？异步发电机做编码没有承诺这当然是可能的。

承诺是一个值得信赖的系统，uninverts的正常回调或反转控制（看到这个_异步和性能_本系列的标题）。因此，结合承诺和代码发电机同步有效地解决所有的回调的主要缺陷的信任。还有，公用事业`承诺（全部）]）`是一个很好的、干净的方法来表示生成器的并发性。`产量`步。

那么这个魔法是如何工作的呢？我们需要一个_跑步者_可以运行我们的发电机，接收`产量`ED承诺，并将其连接到以执行成功值恢复生成器，或以拒绝原因向生成器抛出错误。

许多异步能够事业/图书馆有这样一个“亚军”；例如，`Q.spawn（..）`我的asynquence`跑步者（…）`插件。但是这里有一个独立的跑步者来演示这个过程是如何工作的：

```js
function run(gen) {
	var args = [].slice.call( arguments, 1), it;

	it = gen.apply( this, args );

	return Promise.resolve()
		.then( function handleNext(value){
			var next = it.next( value );

			return (function handleResult(next){
				if (next.done) {
					return next.value;
				}
				else {
					return Promise.resolve( next.value )
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
			})( next );
		} );
}
```

**注：**一个更加多产，评论该实用程序的版本，看_异步和性能_本系列的标题。同时，运行公用事业提供各种异步库往往更有能力比我们这里显示。例如，asynquence的`跑步者（…）`可以处理`产量`ED的承诺，序列，这个，和直接（非承诺）值，给你最大的灵活性。

所以现在运行`* main()`正如前面的代码片段所列出的一样简单：

```js
run( main )
.then(
	function fulfilled(){
		// `*main()` completed successfully
	},
	function rejected(reason){
		// Oops, something went wrong
	}
);
```

本质上，在程序中有两个以上的流控制逻辑异步步骤的地方，您可以_并应_使用由运行工具驱动的生成器，以同步方式表达流控制。这将使理解和维护代码变得更加容易。

这yield-a-promise-resume-the-generator模式是如此普遍和强大，经过6几乎肯定是要引入一个新的功能，会自动无需运行实用JavaScript的下一个版本。我们将覆盖`异步函数`在第8章中（正如他们期望的那样）。

## 回顾

随着JavaScript在其广泛采用中的不断成熟和发展，异步编程越来越成为人们关注的焦点。回调没有完全满足这些任务，完全落在需要更复杂的。

值得庆幸的是，6增加了承诺的一个主要缺点回调地址：在可预见的行为缺乏信任。承诺代表未来完成价值从潜在的异步任务，规范行为，在同步和异步边界。

但它的发电机，充分实现了重新安排我们的异步流程控制代码来强调抽象掉那个难看的回调汤结合利益承诺（又名“地狱”）。

现在，我们能够对付各种异步图书馆者助手这些相互作用，但JavaScript最终会支持这种交互模式与专用的语法！
