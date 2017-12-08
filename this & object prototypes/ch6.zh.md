
# 你不认识JS：_这_与对象的原型

# 第6章：行为授权

在第5章中，我们讨论了`[原型]`机制，以及_为什么_这是令人困惑和不恰当的（尽管经过了二十多年的无数尝试）把它描述为“类”或“继承”。我们一起走过的不仅是相当冗长的语法（`原型。`乱扔垃圾的代码），但各种问题（如惊讶`构造函数。`分辨率或丑陋伪多态语法）。我们探索了“混合”的方式的变化，许多人试图平息这种使用粗糙区。

这是一个常见的反应，在这一点上想知道为什么它必须是如此复杂，做一些看似简单的事情。现在我们已经拉开窗帘，看到的只是如何肮脏的这一切得到的，它不是一个惊喜，大多数开发商不会潜水深JS，而是把这些乱七八糟的一个“类”的图书馆为他们处理它。

我希望现在你不满足于仅仅把这些细节掩盖起来，把这些细节留给一个“黑盒子”图书馆。现在让我们来探究一下我们如何_可以而且应该是_关于对象的思考`[原型]`js中的机制**更简单、更直接的方法**比阶级混乱。

作为对我们从第5章得出的结论的简要回顾，`[原型]`机制是在一个对象上引用另一个对象的内部链接。

当对第一个对象使用属性/方法引用时，该链接将被执行，并且没有这样的属性/方法存在。在这种情况下，`[原型]`链接告诉引擎查找链接到对象的属性/方法。反过来，如果该对象不能完成查找，则`[原型]`被跟踪，等等。这一系列的对象之间的联系形成了所谓的“原型链”。

换句话说，实际机制是JavaScript中可以利用的功能的本质，它是**所有与其他对象链接的对象。**

这一观察对理解本章其余部分的动机和方法是至关重要的！

## 面向委托的设计

正确地思考如何使用`[原型]`以最直接的方式，我们必须认识到它代表了一种与类完全不同的设计模式（参见第4章）。

**注：**一些_面向类的设计原则仍然非常有效，所以不要扔掉你所知道的一切（只是大部分）！例如，_封装_功能强大，并与委托兼容（虽然不常见）。_我们需要尝试将我们的思维从类/继承设计模式转变为行为委托设计模式。如果你在课堂上做了大部分或全部的教育/职业思考，你可能会感到不舒服或感到不自然。你可能需要试几次这种心理练习来了解这种非常不同的思维方式。

首先，我将介绍一些理论练习，然后我们将在更具体的示例中并排查看，以便为您自己的代码提供实际的上下文。

阶级理论

### 假设我们有几个类似的任务（“XYZ”、“ABC”等），我们需要在软件中进行建模。

对于类，设计场景的方式是：定义一个通用的父类（基）类

任务`定义所有类似任务的共享行为。然后，定义子类`XYZ`和`基础知识`两者都继承自`任务`每一个都添加专门的行为来处理各自的任务。`更重要的是，

**类设计模式将鼓励您从继承中获得最大值，您将希望使用方法重写（多态），在这里您覆盖某些通用的定义。**任务`方法在你的`XYZ`任务，也许甚至利用`超级的`在向它添加更多行为时调用该方法的基本版本。`你可能会找到不少地方。**在这里，您可以将一般行为抽象为父类，并在子类中专门化（重写）。**下面是该场景的一些松散伪代码：

现在，您可以实例化一个或多个

```js
class Task {
	id;

	// constructor `Task()`
	Task(ID) { id = ID; }
	outputTask() { output( id ); }
}

class XYZ inherits Task {
	label;

	// constructor `XYZ()`
	XYZ(ID,Label) { super( ID ); label = Label; }
	outputTask() { super(); output( label ); }
}

class ABC inherits Task {
	// ...
}
```

副本**的**XYZ`子类，并使用这些实例执行任务“XYZ”。这些实例`份**一般的**任务`定义的行为以及特定的行为`XYZ`定义的行为。同样，`基础知识`类将具有`任务`行为与具体`基础知识`行为。在构建之后，一般只与这些实例（而不是类）交互，因为实例中每个都有您需要执行的任务的所有副本。`授权理论

### 但是现在让我们来考虑同一个问题域，但是使用

行为代表团_而不是_类_。_首先定义一个

对象**（不是一个类，也不是一个**功能`作为最js'rs会导致你相信）称为`任务`它将有具体的行为，其中包括各种任务可以使用的实用方法。`它将有具体的行为，包括各种任务可以使用的实用方法（读）：_代表_！）然后，对于每个任务（“XYZ”，“ABC”），定义一个**对象**持有特定于任务的数据/行为。你**链接**您的特定于任务的对象`任务`实用工具对象，允许它们在需要时委派给它。

基本上，您考虑将任务“XYZ”作为从两个同级/对等对象需要的行为来执行（`XYZ`和`任务`完成它。但是我们不需要通过类拷贝把它们组合在一起，我们可以把它们放在各自的对象中，我们可以允许`XYZ`对象**代表**任务`在需要的时候。`这里有一些简单的代码来说明你是如何做到的：

在此代码中，

```js
var Task = {
	setID: function(ID) { this.id = ID; },
	outputID: function() { console.log( this.id ); }
};

// make `XYZ` delegate to `Task`
var XYZ = Object.create( Task );

XYZ.prepareTask = function(ID,Label) {
	this.setID( ID );
	this.label = Label;
};

XYZ.outputTaskDetails = function() {
	this.outputID();
	console.log( this.label );
};

// ABC = Object.create( Task );
// ABC ... = ...
```

任务`和`XYZ`不是类（或函数），它们是`只是对象**。**XYZ`是通过`对象创建（..）`到`\[原型]`委托给`任务`对象（见第5章）。`与类面向（又名，面向对象-面向对象）相比，我称这种风格为代码。

“oloo”**（与其他对象链接的对象）。所有的我们**真正地_在乎的是_XYZ`对象委托给`任务`对象（与`基础知识`对象）。`在JavaScript中

\[原型]`的联系机制`物体**其他**物体**。没有像“类”这样的抽象机制，不管你如何说服自己。就像在上游划独木舟一样：你**可以_做吧，但你是_选择_违背自然潮流，所以很明显。_要去你要去的地方更难。**需要注意的其他一些差异**

oloo代码风格**：**两

1.  身份证件`和`标签`上一个类示例的数据成员直接是数据属性。`XYZ`（两者都不是`任务`）。一般来说`\[原型]`代表团，`你想要的状态是危险的**（**XYZ`，`基础知识`），不在委托人身上。`任务`）。`有了班级设计模式，我们有意命名。
2.  outputtask`对双亲也一样。`任务`）和孩子（`XYZ`），这样我们就可以利用覆盖（多态）。在行为授权中，我们做相反的事情：`我们尽可能避免将事物命名相同。**在不同级别的**\[原型]`链（称为阴影，见5章），因为有这些名字冲突造成的尴尬和脆性的句法歧义的引用（见4章），我们要避免的，如果我们能。`这种设计模式要求较少使用一般方法名，而这些名称往往是重写的，而不是更多的描述性方法名，

    具体的_对于每个对象正在做的行为类型。_这实际上可以创建更易于理解/维护的代码。**因为方法的名称（不仅在定义位置，而且散布在其他代码中）更为明显（自文档化）。**这个压缩文件SetID（ID）；

3.  `里面的一个方法`XYZ`对象首先查看`XYZ`对于`压缩文件SetID（..）`但是，因为它没有找到这个名字的方法`XYZ`，`\[原型]`代表团`意味着它可以链接到_任务_寻找`压缩文件SetID（..）`当然，它发现。此外，由于隐式调用站点`这`绑定规则（见第2章）`压缩文件SetID（..）`运行，即使找到了方法`任务`，的`这`该函数调用的绑定是`XYZ`完全符合我们的期望和要求。我们看到同样的事情`这outputid()。`稍后在代码清单中。`换句话说，存在的通用实用方法`任务

    与我们交互时可用`XYZ`，因为`XYZ`可以代表`任务`。`行为代表团`意思是：让一些对象（

**XYZ**）提供代表团（`任务`）如果对象上没有找到属性或方法引用（`XYZ`）。`这是一个`非常强大的

设计模式非常不同于父类和子类、继承、多态等概念，而不是垂直地组织对象在脑海中，与父母流到孩子身上，认为对象是并排的，作为对等的，在对象之间有任何代表性的联系。_注：_委托更恰当地用作内部实现细节，而不是直接暴露在API设计中。在上面的例子中，我们不一定

**打算**用我们的API设计让开发者打电话_setid() XYZ。_（当然可以！）我们有点。`隐藏`代表团作为我们API的内部细节，在哪里_XYZ。preparetask（..）_代表`任务。压缩文件SetID（..）`。看到“联系Fallbacks？”第5章详细讨论。`双方代表团（无效）`你不能创建一个

#### 周期

两个或多个对象相互委托（双向）的。如果你让_B_联系`一`然后试着链接`一`到`B`你会得到一个错误。`这是一个耻辱（不是很令人惊讶，但这是不允许的，轻度烦人）。如果您引用了两个地方都不存在的属性/方法，那么`\[原型]

环。但如果所有的参考资料都严格存在，那么`B`能代表`一`反之亦然。`可`反之亦然。_能够_工作。这意味着您可以使用任何一个对象来委托另一个对象进行各种任务。有几个利基用例，这可能是有益的。

但它不允许因为引擎实现观察，检查它的性能（和拒绝！）每次在查找对象上的属性时，不需要在每次检查属性的情况下都有性能检查的无限循环引用。

#### 调试

我们将简要介绍一个可能给开发人员带来困惑的细微细节。一般来说，js规范不控制浏览器开发工具应该如何向开发人员表示特定的值/结构，因此每个浏览器/引擎可以自由地解释他们认为合适的东西。因此，浏览器/工具_不要总是同意_。具体来说，我们现在要研究的行为目前只在Chrome的开发者工具中观察到。

考虑这个传统的“类构造函数”样式的js代码，因为它会出现在_慰问_Chrome开发者工具：

```js
function Foo() {}

var a1 = new Foo();

a1; // Foo {}
```

让我们看一看该代码段的最后一行：评估结果的输出`A1`表达，打印`foo { }`。如果您在Firefox中尝试相同的代码，您可能会看到`对象{ }`。为什么会有差异？这些输出意味着什么？

Chrome本质上是说“{”是一个空的对象，它是由一个名为“富”的函数构造的。Firefox说“{是对象的一般构造的空对象”。微妙的区别是，Chrome正在积极跟踪，作为一个_内部属性_，实际上是构建这个函数的名称，而其他浏览器不跟踪附加信息。

试图用JavaScript机制解释这一点是很有诱惑力的：

```js
function Foo() {}

var a1 = new Foo();

a1.constructor; // Foo(){}
a1.constructor.name; // "Foo"
```

那么，Chrome是如何通过简单地检查对象的“输出”来输出`constructor.name。`？令人困惑的是，答案是“是”和“不是”。

考虑这个代码：

```js
function Foo() {}

var a1 = new Foo();

Foo.prototype.constructor = function Gotcha(){};

a1.constructor; // Gotcha(){}
a1.constructor.name; // "Gotcha"

a1; // Foo {}
```

即使我们改变`a1.constructor.name`要被别的东西（“疑难杂症”），Chrome的控制台仍然使用“foo”的名字。

因此，它似乎是对前面问题的答案（它使用了吗？）`constructor.name。`？）是**不**它必须在其他地方跟踪，内部。

但是，不是那么快！让我们来看看这种行为与oloo代码风格：

```js
var Foo = {};

var a1 = Object.create( Foo );

a1; // Object {}

Object.defineProperty( Foo, "constructor", {
	enumerable: false,
	value: function Gotcha(){}
});

a1; // Gotcha {}
```

啊啊啊啊啊啊！**疑难杂症！**这里，Chrome的控制台**做**查找并使用`constructor.name。`。实际上，在写这本书的时候，这个确切的行为被标识为Chrome中的一个bug，当你读到这一点时，它可能已经被修复了。所以你可能已经看到更正了。`对象/对象{ }`。

除了这个bug外，内部跟踪（显然只用于调试输出目的）是Chrome所做的“构造函数名称”（在前面的代码段中显示），这是一个有意的Chrome扩展，超出了js规范的要求。

如果你不使用“构造函数”使你的对象，当我们沮丧的oloo风格的代码在本章中，你会得到Chrome对象_不_跟踪内部构造函数名称”，这样的对象只有正确输出为“对象{ }”，意思是“对象产生object()建设”。

**不要以为**这是一个缺点oloo编码风格。当你oloo行为代表团作为你的设计模式的代码，_谁_“构造”（即，_它的功能_被称为`新的`？）有些对象是不相关的细节。Chrome的具体内部构造函数的名字“跟踪是有用的如果你完全接受“编码类风格，但也不是如果你不是拥抱oloo代表团。

### 心理模型的比较

现在，您可以看到“类”和“委托”设计模式之间的区别，至少从理论上来说，让我们看看这些设计模式对我们用来分析代码的心理模型所产生的影响。

我们将探讨一些理论（“foo”、“酒吧”）代码，并比较两种方法（OO与oloo）执行代码。第一段采用经典（“原型”）的面向对象的风格：

```js
function Foo(who) {
	this.me = who;
}
Foo.prototype.identify = function() {
	return "I am " + this.me;
};

function Bar(who) {
	Foo.call( this, who );
}
Bar.prototype = Object.create( Foo.prototype );

Bar.prototype.speak = function() {
	alert( "Hello, " + this.identify() + "." );
};

var b1 = new Bar( "b1" );
var b2 = new Bar( "b2" );

b1.speak();
b2.speak();
```

父类`Foo`由子类继承`酒吧`然后实例化两次`B1`和`B2`。我们有的是`B1`委托`bar.prototype`这代表`foo.prototype`。在这一点上，你看起来应该很熟悉。没有什么比这更糟糕的了。

现在，让我们实现**完全相同的功能**使用_oloo_代码风格：

```js
var Foo = {
	init: function(who) {
		this.me = who;
	},
	identify: function() {
		return "I am " + this.me;
	}
};

var Bar = Object.create( Foo );

Bar.speak = function() {
	alert( "Hello, " + this.identify() + "." );
};

var b1 = Object.create( Bar );
b1.init( "b1" );
var b2 = Object.create( Bar );
b2.init( "b2" );

b1.speak();
b2.speak();
```

我们完全相同的优势`[原型]`代表团`B1`到`酒吧`到`Foo`正如我们在前一段代码中所做的那样`B1`，`bar.prototype`，和`foo.prototype`。**我们仍然有相同的3个对象连接在一起。**。

但是，重要的是，我们已经大大简化了。_所有其他的东西_继续，因为我们刚刚建立起来**物体**互相链接，而不需要所有的那些东西，那些看起来混乱（但不听话！）像类一样，具有构造函数和原型`新的`电话.

问问自己：如果我能得到相同的功能与oloo风格代码为我做“类”的代码风格，但oloo简单和H**不oloo更好**？

让我们来研究这两个片段之间的心理模型。

首先，类样式代码片段意味着实体及其关系的心智模型：

<img src="fig4.png">

事实上，这有点不公平/误导，因为它显示了很多额外的细节，你没有。_技术上_需要随时知道（尽管你）_做_需要理解它！）一个问题是，这是一系列复杂的关系。但另一个带走：如果你花时间跟随这些关系箭头，**内部一致性令人吃惊。**JS的机制。

例如，js函数访问的能力。`打电话（…）`，`应用（…）`，和`绑定（…）`（见第2章）是因为函数本身是对象，函数对象也有一个`[原型]`联动，到`function.prototype`对象，它定义了任何函数对象都可以委托的默认方法。js可以做那些事情，_你也可以！_。

好，让我们来看一个_轻微地_该图的简化版本比较“公平”，它只显示了_相关的_实体与关系。

<img src="fig5.png">

还挺复杂的，嗯？虚线是在设置“继承”之间描述隐含关系的。`foo.prototype`和`bar.prototype`and haven't yet_固定的_这个**丢失的**构造函数。`属性的引用（见“构造函数归来”5章）。即使删除了这些虚线，每次使用对象链接时，心理模型仍然是非常糟糕的。`现在，让我们在oloo风格代码的心智模式看：

正如你可以看到的比较，很明显oloo风格代码

<img src="fig6.png">

很少的东西_担心的，因为oloo风格的代码包含了_事实**我们唯一真正关心的是**与其他对象链接的对象**。**所有其他的“类”的缺陷是一个复杂的方式获得相同的结果。去掉那些东西，事情变得简单多了（不会失去任何能力）。

类与对象

## 我们刚刚看到了“阶级”与“行为代表”的各种理论探索和心理模式。但是，让我们来看看更具体的代码场景，以展示您是如何实际使用这些想法的。

我们将首先检查前端Web开发中的一个典型场景：创建UI小部件（按钮、下拉等）。

小部件“类”

### 因为您可能仍然如此习惯于OO设计模式，所以您可能会立即考虑到父类的问题域（也许是调用）

小装置`）使用所有公共基小部件行为，然后为特定的小部件类型派生子类（如`按钮`）。`注：

**我们将在这里使用jQuery来进行DOM和CSS操作，只因为它是我们目前讨论中不真正关心的细节。这个代码不关心它的js框架（jQuery，Dojo，由比，等等），如果有的话，你可能会解决这种无聊的任务。**让我们研究一下如何在没有任何“类”帮助库或语法的情况下，在经典风格的纯js中实现“类”设计：

OO设计模式告诉我们声明一个基

```js
// Parent class
function Widget(width,height) {
	this.width = width || 50;
	this.height = height || 50;
	this.$elem = null;
}

Widget.prototype.render = function($where){
	if (this.$elem) {
		this.$elem.css( {
			width: this.width + "px",
			height: this.height + "px"
		} ).appendTo( $where );
	}
};

// Child class
function Button(width,height,label) {
	// "super" constructor call
	Widget.call( this, width, height );
	this.label = label || "Default";

	this.$elem = $( "<button>" ).text( this.label );
}

// make `Button` "inherit" from `Widget`
Button.prototype = Object.create( Widget.prototype );

// override base "inherited" `render(..)`
Button.prototype.render = function($where) {
	// "super" call
	Widget.prototype.render.call( this, $where );
	this.$elem.click( this.onClick.bind( this ) );
};

Button.prototype.onClick = function(evt) {
	console.log( "Button '" + this.label + "' clicked!" );
};

$( document ).ready( function(){
	var $body = $( document.body );
	var btn1 = new Button( 125, 30, "Hello" );
	var btn2 = new Button( 150, 40, "World" );

	btn1.render( $body );
	btn2.render( $body );
} );
```

渲染（…）`在父类中，然后在子类中重写它，而不是本身替换它，而是使用按钮特定的行为来增强基本功能。`注意丑陋

明确的伪基因多态性_（见第4章）_widget.call`和`widget.prototype.render.call`从“类”方法调用“超级”调用的引用返回到“父”类基类方法。讨厌.`6

#### 班`糖`我们覆盖的ES6

班`附录A中详细介绍了语法糖，但是让我们简要说明如何使用相同的代码实现相同的代码`班`：`毫无疑问，一些以前的经典方法的语法丑陋已平整了6的

```js
class Widget {
	constructor(width,height) {
		this.width = width || 50;
		this.height = height || 50;
		this.$elem = null;
	}
	render($where){
		if (this.$elem) {
			this.$elem.css( {
				width: this.width + "px",
				height: this.height + "px"
			} ).appendTo( $where );
		}
	}
}

class Button extends Widget {
	constructor(width,height,label) {
		super( width, height );
		this.label = label || "Default";
		this.$elem = $( "<button>" ).text( this.label );
	}
	render($where) {
		super.render( $where );
		this.$elem.click( this.onClick.bind( this ) );
	}
	onClick(evt) {
		console.log( "Button '" + this.label + "' clicked!" );
	}
}

$( document ).ready( function(){
	var $body = $( document.body );
	var btn1 = new Button( 125, 30, "Hello" );
	var btn2 = new Button( 150, 40, "World" );

	btn1.render( $body );
	btn2.render( $body );
} );
```

班`。A的存在`超级（…）`特别是看起来很不错（虽然你深入挖掘它，它不是所有的玫瑰！）。`尽管语法改进，

这些都不是**真实的_类_，因为它们仍然在**\[原型]`机制.他们遭受了我们在第4, 5章和本章迄今所探讨的所有心理模型的不匹配。附录A将阐述ES6`班`语法及其在细节中的含义。我们将看到为什么解决语法打嗝不基本解决JS我们班的混乱，虽然它使一个英勇的努力伪装成一个解决方案！`你是否使用经典的原型语法或新6糖，你还是做了

选择_用“类”建模问题域（UI小部件）。正如前几章试图证明的那样_选择_在JavaScript中，你会选择额外的头痛和精神税。_将控件对象

### 以下是我们的简单

小装置`/`按钮`例如，使用`oloo风格代表团**：**这oloo式的做法，我们不认为

```js
var Widget = {
	init: function(width,height){
		this.width = width || 50;
		this.height = height || 50;
		this.$elem = null;
	},
	insert: function($where){
		if (this.$elem) {
			this.$elem.css( {
				width: this.width + "px",
				height: this.height + "px"
			} ).appendTo( $where );
		}
	}
};

var Button = Object.create( Widget );

Button.setup = function(width,height,label){
	// delegated call
	this.init( width, height );
	this.label = label || "Default";

	this.$elem = $( "<button>" ).text( this.label );
};
Button.build = function($where) {
	// delegated call
	this.insert( $where );
	this.$elem.click( this.onClick.bind( this ) );
};
Button.onClick = function(evt) {
	console.log( "Button '" + this.label + "' clicked!" );
};

$( document ).ready( function(){
	var $body = $( document.body );

	var btn1 = Object.create( Button );
	btn1.setup( 125, 30, "Hello" );

	var btn2 = Object.create( Button );
	btn2.setup( 150, 40, "World" );

	btn1.build( $body );
	btn2.build( $body );
} );
```

小装置`作为父母和`按钮`作为一个孩子。而，`小装置`只是一个物体`并且是一种实用程序集合，任何特定类型的小部件都可能想要删除。**只是一个物体**并且是一种实用程序集合，任何特定类型的小部件都可能希望将其委托给`按钮`也是一个独立的对象。**（与代表团联系在一起**小装置`当然可以！）。`从设计模式的角度来看，我们

没有**共享相同的方法名**渲染（…）`在两个对象中，类建议的方式，但我们选择不同的名称（`插入（…）`和`建造（…）`）更具体地描述每个任务的具体任务。这个`初始化_方法被称为_init（..）`和`设置（..）`，出于同样的原因。`这不仅代表团设计模式提出了不同的、描述性的名字（而不是共享和更通用的名称），但这样做oloo恰好避免显式伪多态调用丑（

widget.call`和`widget.prototype.render.call`），正如您可以通过简单的、相对的、委派的调用看到的那样`这个init（..）`和`这个…插入（..）`。`在语法上，我们也没有任何构造函数，

原型。`或`新的`目前，他们是，事实上，只是不必要的缺陷。`现在，如果你密切关注，你可能会注意到以前只是一个电话。

VaR BTN1 =新按钮（..）`现在是两个电话（`VaR BTN1 =对象创建（按钮）。`和`BTN1。安装程序（..）`）。起初这似乎是一个缺点（更多的代码）。`然而，即使这是

亲oloo代码风格**与经典原型风格代码相比。怎么用？**对于类构造函数，在同一步骤中进行“构造”和“初始化”是“强制的”（实际上不是强建议的）。然而，在许多情况下，能够做这两个步骤分别（为你做oloo！）更灵活。

例如，假设您在程序开始时在池中创建所有实例，但您等待用特定的设置初始化它们，直到它们从池中取出并使用。我们展示了两个调用，它们发生在彼此的旁边，但当然，它们可以在非常不同的时间，在我们的代码非常不同的部分，根据需要发生。

oloo

**支持**更好的_关注分离的原则，在创建和初始化不一定合并为同一操作。_简单的设计

## 除了oloo提供表面上简单的（和更灵活的！）代码、行为委托作为模式实际上可以导致更简单的代码体系结构。让我们看看最后一个例子，说明如何oloo简化你的整体设计。

我们将研究的场景是两个控制器对象，一个用于处理Web页面的登录表单，另一个用于实际处理服务器的身份验证（通信）。

我们需要一个实用工具帮助服务器进行Ajax通信。我们将使用jQuery（尽管任何框架都很好），因为它不仅为我们处理Ajax，而且还返回一个类似于承诺的响应，这样我们就可以在调用代码中侦听响应。

然后（…）`。`注：

**我们这里不包括承诺，但我们将在未来的标题中涵盖它们。**“你不知道JS”_系列。_按照典型的类设计模式，我们将把任务分解为一个类中的基本功能。

控制器`然后，我们将派生两个子类，`logincontroller`和`authcontroller`两者都继承自`控制器`并专门研究一些基本行为。`我们有所有控制器共享的基本行为，这些行为是

```js
// Parent class
function Controller() {
	this.errors = [];
}
Controller.prototype.showDialog = function(title,msg) {
	// display title & message to user in dialog
};
Controller.prototype.success = function(msg) {
	this.showDialog( "Success", msg );
};
Controller.prototype.failure = function(err) {
	this.errors.push( err );
	this.showDialog( "Error", err );
};
```

```js
// Child class
function LoginController() {
	Controller.call( this );
}
// Link child class to parent
LoginController.prototype = Object.create( Controller.prototype );
LoginController.prototype.getUser = function() {
	return document.getElementById( "login_username" ).value;
};
LoginController.prototype.getPassword = function() {
	return document.getElementById( "login_password" ).value;
};
LoginController.prototype.validateEntry = function(user,pw) {
	user = user || this.getUser();
	pw = pw || this.getPassword();

	if (!(user && pw)) {
		return this.failure( "Please enter a username & password!" );
	}
	else if (pw.length < 5) {
		return this.failure( "Password must be 5+ characters!" );
	}

	// got here? validated!
	return true;
};
// Override to extend base `failure()`
LoginController.prototype.failure = function(err) {
	// "super" call
	Controller.prototype.failure.call( this, "Login invalid: " + err );
};
```

```js
// Child class
function AuthController(login) {
	Controller.call( this );
	// in addition to inheritance, we also need composition
	this.login = login;
}
// Link child class to parent
AuthController.prototype = Object.create( Controller.prototype );
AuthController.prototype.server = function(url,data) {
	return $.ajax( {
		url: url,
		data: data
	} );
};
AuthController.prototype.checkAuth = function() {
	var user = this.login.getUser();
	var pw = this.login.getPassword();

	if (this.login.validateEntry( user, pw )) {
		this.server( "/check-auth",{
			user: user,
			pw: pw
		} )
		.then( this.success.bind( this ) )
		.fail( this.failure.bind( this ) );
	}
};
// Override to extend base `success()`
AuthController.prototype.success = function() {
	// "super" call
	Controller.prototype.success.call( this, "Authenticated!" );
};
// Override to extend base `failure()`
AuthController.prototype.failure = function(err) {
	// "super" call
	Controller.prototype.failure.call( this, "Auth Failed: " + err );
};
```

```js
var auth = new AuthController(
	// in addition to inheritance, we also need composition
	new LoginController()
);
auth.checkAuth();
```

成功（..）`，`失败（..）`和`ShowDialog（..）`。我们的子类`logincontroller`和`authcontroller`重写`失败（..）`和`成功（..）`增加默认基类行为。还注意到，`authcontroller`需要一个实例`logincontroller`与登录表单交互，从而成为成员数据属性。`另一件事是我们选择了一些。

作文_撒在继承的顶端。_authcontroller`需要了解`logincontroller`所以我们实例化它（`新的logincontroller()`并保留一个名为`this.login`参考它，以便`authcontroller`可以调用行为`logincontroller`。`注：

**那里**可以_有一点小小的诱惑_authcontroller`继承`logincontroller`反之亦然，如我们所拥有的`虚拟成分_通过继承链。但这是类继承错误的明显例子。_这个_问题域的模型，因为两者都不_authcontroller`也没有`logincontroller`它们是另一个的基类行为，所以如果类是惟一的设计模式，那么它们之间的继承就没有什么意义了。相反，我们在一些简单的层次`作文_现在他们可以合作，同时仍然受益于母公司的继承。_控制器`。`如果您熟悉面向类（OO）的设计，这一切看起来应该非常熟悉和自然。

de分类

### 但，

我们真的需要对这个问题进行建模吗？**与父母**控制器`类，两个子类，`和**有些成分**？有没有办法利用oloo风格行为代表团和有_许多的_简单的设计吗？**对!**

```js
var LoginController = {
	errors: [],
	getUser: function() {
		return document.getElementById( "login_username" ).value;
	},
	getPassword: function() {
		return document.getElementById( "login_password" ).value;
	},
	validateEntry: function(user,pw) {
		user = user || this.getUser();
		pw = pw || this.getPassword();

		if (!(user && pw)) {
			return this.failure( "Please enter a username & password!" );
		}
		else if (pw.length < 5) {
			return this.failure( "Password must be 5+ characters!" );
		}

		// got here? validated!
		return true;
	},
	showDialog: function(title,msg) {
		// display success message to user in dialog
	},
	failure: function(err) {
		this.errors.push( err );
		this.showDialog( "Error", "Login invalid: " + err );
	}
};
```

```js
// Link `AuthController` to delegate to `LoginController`
var AuthController = Object.create( LoginController );

AuthController.errors = [];
AuthController.checkAuth = function() {
	var user = this.getUser();
	var pw = this.getPassword();

	if (this.validateEntry( user, pw )) {
		this.server( "/check-auth",{
			user: user,
			pw: pw
		} )
		.then( this.accepted.bind( this ) )
		.fail( this.rejected.bind( this ) );
	}
};
AuthController.server = function(url,data) {
	return $.ajax( {
		url: url,
		data: data
	} );
};
AuthController.accepted = function() {
	this.showDialog( "Success", "Authenticated!" )
};
AuthController.rejected = function(err) {
	this.failure( "Auth Failed: " + err );
};
```

自`authcontroller`只是一个对象（是这样的）`logincontroller`），我们不需要实例化（如`新的authcontroller()`执行我们的任务。我们所需要做的就是：

```js
AuthController.checkAuth();
```

当然，与oloo，如果你需要创建一个或多个代表团中链额外的对象，这很容易，而且还不需要什么类的实例化：

```js
var controller1 = Object.create( AuthController );
var controller2 = Object.create( AuthController );
```

行为委托，`authcontroller`和`logincontroller`是**只是对象**，_水平的_彼此不相关，也不作为父母和孩子在课堂上的安排。我们有点武断地选择了`authcontroller`代表`logincontroller`——代表团走相反的方向也是一样有效的。

从第二个代码清单中得到的主要结果是，我们只有两个实体（`logincontroller`和`authcontroller`），**不是三**如前。

我们不需要一个基地`控制器`类之间的“共享”行为，因为授权是一种强大的机制，可以提供我们需要的功能。我们也像前面提到的那样，不需要实例化我们的类来处理它们，因为没有类，**只是物体本身。**此外，没有必要_作文_正如代表团给予这两个对象合作的能力一样。_差别地_如需要。

最后，我们避免了由于没有名称而导致面向类设计的多态性陷阱。`成功（..）`和`失败（..）`在这两个对象是相同的，这就要求丑显假多形。相反，我们打电话给他们`accepted()`和`拒绝（…）`在`authcontroller`-对其特定任务稍微有描述性的名称。

**底线**我们最终拥有相同的功能，但是（非常）简单的设计。这是oloo风格代码的权力及权力_行为代表团_设计模式。

## 良好的语法

一个更美好的东西，让6的`班`所以看似诱人的（见附录A为什么避免它！）是用于声明类方法的简短语法：

```js
class Foo {
	methodName() { /* .. */ }
}
```

我们得放弃这个词`功能`从宣言，这使JS开发商到处欢呼！

你可能已经注意到，受挫，建议oloo语法上面有很多`功能`出场，这似乎有点像一个小人在oloo简化目标。**但不一定是那样的！**

截至6，我们可以使用_简洁的方法声明_在任何对象的文字，所以在oloo样式对象可以宣布这种方式（相同的短手糖与`班`正文语法）：

```js
var LoginController = {
	errors: [],
	getUser() { // Look ma, no `function`!
		// ...
	},
	getPassword() {
		// ...
	}
	// ...
};
```

唯一不同的是对象文本仍然需要。`，`元素之间的逗号分隔符`班`语法没有。在整个方案中有相当小的让步。

此外，截至6，需求你使用（如语法`authcontroller`定义）在您单独分配属性而不使用对象文字的情况下，可以使用对象文字重新编写（这样您就可以使用简洁的方法），并且您只需修改对象的`[原型]`具有`对象。setprototypeof（..）`像这样：

```js
// use nicer object literal syntax w/ concise methods!
var AuthController = {
	errors: [],
	checkAuth() {
		// ...
	},
	server(url,data) {
		// ...
	}
	// ...
};

// NOW, link `AuthController` to delegate to `LoginController`
Object.setPrototypeOf( AuthController, LoginController );
```

oloo风格为6，用简洁的方法，**友好得多**比以前（甚至在那时，它比经典原型代码更简单和更好）。**你不必选择上课。**（复杂度）获得好的干净对象语法！

### unlexical

那里_是_简明方法的一个缺点是微妙但很重要。考虑这个代码：

```js
var Foo = {
	bar() { /*..*/ },
	baz: function baz() { /*..*/ }
};
```

这里的语法去糖表示如何将操作代码：

```js
var Foo = {
	bar: function() { /*..*/ },
	baz: function baz() { /*..*/ }
};
```

看到区别了吗？这个`bar()`短手成了_匿名函数的表达式_（`function() ..`附加到`酒吧`属性，因为函数对象本身没有名称标识符。将其与手动指定的比较_命名函数表达式_（`功能baz() ..`）它有一个词汇名标识符。`巴兹`除了依附于`。巴兹`财产。

还等什么？在_“范围和闭包”_标题本_“你不知道JS”_本书系列，我们覆盖的三个主要缺点_匿名函数表达式_详细。我们只需简短地重复它们，这样我们就可以与简洁的方法进行比较。

缺乏一个`名称`匿名函数上的标识符：

1.  使调试堆栈更难跟踪
2.  使自引用（递归、事件（UN）绑定等）更加困难。
3.  使代码（一点点）更难理解

项目1和3不适用于简明方法。

即使去糖利用_匿名函数的表达式_通常没有`名称`在堆栈跟踪中，指定了用于设置内部的简明方法。`名称`相应的函数对象的属性，因此堆栈跟踪应该能够使用它（虽然依赖于实现，所以不能保证）。

不幸的是，项目2，**仍然是简洁方法的一个缺点**。它们将没有一个用作自引用的词汇标识符。考虑：

```js
var Foo = {
	bar: function(x) {
		if (x < 10) {
			return Foo.bar( x * 2 );
		}
		return x;
	},
	baz: function baz(x) {
		if (x < 10) {
			return baz( x * 2 );
		}
		return x;
	}
};
```

手动`foo。B`在这个例子中，上面的引用是足够的，但是在很多情况下，函数不一定能够做到这一点，例如函数在不同对象之间的委托共享中，使用`这`绑定等，您希望使用真正的自引用，而函数对象的`名称`标识符是实现这一目标的最佳方式。

只是要注意这个简洁方法的警告，如果你遇到这样的问题，缺乏自我参照，请务必放弃简明的方法语法。**只是为了那个宣言**赞成手册_命名函数表达式_申报表：`记者：功能baz() {等}`。

## 反思

如果您花了很多时间在面向类的编程（以js或其他语言），您可能很熟悉_式的反思_检查一个实例找出什么_友善的_对象是。主要目标_式的反思_类实例是基于对象的结构/功能的原因。_它是如何产生的_。

考虑使用这个代码`运算符`（见5章）反思一个对象`A1`推断其能力：

```js
function Foo() {
	// ...
}
Foo.prototype.something = function(){
	// ...
}

var a1 = new Foo();

// later

if (a1 instanceof Foo) {
	a1.something();
}
```

因为`foo.prototype`（不`Foo`！）在`[原型]`链（见第5章）`A1`，的`运算符`算子（混淆）假装告诉我们，`A1`是一个实例`Foo`“类”。有了这些知识，我们就假设`A1`具有以下描述的功能`Foo`“类”。

当然，没有`Foo`类，只有普通的普通函数`Foo`这恰好是对任意对象的引用（`foo.prototype`），`A1`碰巧代表团与。它的语法，`运算符`假装在检查两者之间的关系`A1`和`Foo`但它实际上告诉我们是否`A1`和（引用的任意对象）`foo.prototype`是相关的。

语义混乱（或间接）的`运算符`语法意味着使用`运算符`询问对象是否基于内省`A1`与所讨论的功能对象相关，您_不得不_拥有一个对该对象持有引用的函数——您不能直接询问这两个对象是否相关。

回忆摘要`Foo`/`酒吧`/`B1`在本章前面的例子，我们会把这里：

```js
function Foo() { /* .. */ }
Foo.prototype...

function Bar() { /* .. */ }
Bar.prototype = Object.create( Foo.prototype );

var b1 = new Bar( "b1" );
```

对于_式的反思_对该示例中的实体的用途，使用`运算符`和`原型。`语义，这里是您可能需要执行的各种检查：

```js
// relating `Foo` and `Bar` to each other
Bar.prototype instanceof Foo; // true
Object.getPrototypeOf( Bar.prototype ) === Foo.prototype; // true
Foo.prototype.isPrototypeOf( Bar.prototype ); // true

// relating `b1` to both `Foo` and `Bar`
b1 instanceof Foo; // true
b1 instanceof Bar; // true
Object.getPrototypeOf( b1 ) === Bar.prototype; // true
Foo.prototype.isPrototypeOf( b1 ); // true
Bar.prototype.isPrototypeOf( b1 ); // true
```

可以公平地说，其中有些很糟糕。例如，直观地（使用类），您可能希望能够说出类似的内容。`酒吧是Foo`（因为很容易混淆“实例”意味着认为它包含“继承”），但JS中的这种比较是不明智的。你必须做`bar.prototype instanceof Foo`相反。

另一种常见但可能不那么健壮的模式_式的反思_，许多开发者似乎比较喜欢`运算符`被称为“鸭子打字”。这个词来自格言，“如果它看起来像鸭子，它叫起来像鸭子，它必须是一个鸭子”。

例子:

```js
if (a1.something) {
	a1.something();
}
```

而不是检查两者之间的关系`A1`和一个对象拥有可授权`something()`函数，我们假设测试`a1.something`传递手段`A1`有呼叫的能力`something()。`（不管它是否直接找到了方法）`A1`或委托给其他对象）。就其本身而言，这种假设并不那么危险。

但是“鸭子打字”常常被用来制作。**关于对象能力的其他假设**除了测试之外，这当然会给测试带来更多的风险（又名脆性设计）。

一个著名的例子“duck typing”配备6承诺（作为早期的注意，解释不被包含在这本书）。

由于各种原因，需要确定是否有任意对象引用。_是一个承诺_但是测试的方法是检查对象是否有一个`then()`函数存在于。换言之，**如果任何对象**碰巧有`then()`方法6承诺将无条件地承担，该对象**是一个“thenable”**因此，指望它表现conformantly对承诺所有标准的行为。

如果你有任何不承诺的对象，无论什么原因发生`then()`方法上，强烈建议你远离6承诺机制避免坏的假设。

这个例子清楚地说明了“鸭子打字”的风险。只有在控制条件下，才能使用这种方法。

把我们的注意力再次回到oloo风格代码在本章介绍了，_式的反思_结果是更干净。让我们回想（，缩写）的`Foo`/`酒吧`/`B1`oloo例子在本章前面：

```js
var Foo = { /* .. */ };

var Bar = Object.create( Foo );
Bar...

var b1 = Object.create( Bar );
```

使用这种oloo方法，在大家都是普通的对象，相关的经`[原型]`代表团，这里是相当简化的。_式的反思_我们可以用：

```js
// relating `Foo` and `Bar` to each other
Foo.isPrototypeOf( Bar ); // true
Object.getPrototypeOf( Bar ) === Foo; // true

// relating `b1` to both `Foo` and `Bar`
Foo.isPrototypeOf( b1 ); // true
Bar.isPrototypeOf( b1 ); // true
Object.getPrototypeOf( b1 ) === Bar; // true
```

我们没有使用`运算符`了，因为_一_我的原型？”没有更多的间接必要的东西一样`foo.prototype`或是痛苦的冗长`foo。原型。isprototypeof（..）`。

我认为公平地说，这些检查比以前的自省检查要复杂得多。**又一次，我们看到，oloo比较简单（但所有相同的功率）类JavaScript编码风格。**

## 回顾（TL；DR）

类和继承是一种设计模式，您可以_选择_，或_没有选择_在软件体系结构中。大多数开发人员想当然地认为类是组织代码的唯一（适当）方式，但在这里我们看到了另一个不太常见的模式，它实际上非常强大：**行为代表团**。

行为委托将对象看作是彼此之间的对等物，它们彼此委托，而不是父类和子类之间的关系。JavaScript的`[原型]`机制本身就是一种行为委托机制。这意味着我们可以选择在js之上实现类力学（参见第4章和第5章），或者我们可以只接受`[原型]`作为授权机制。

当您只使用对象设计代码时，不仅简化了使用的语法，而且实际上可以导致更简单的代码体系结构设计。

**oloo**（与其他对象链接的对象）是一种代码样式，它直接创建和关联对象而不需要抽象类。oloo很自然地实现了`[原型]`基于行为的委托。
