＃你不知道JS:作用域和闭包
＃第3章:函数作用域与块级作用域

正如我们在第2章中探讨的那样,范围由一系列"气泡"组成,每个"气泡"都充当容器或者桶,其中声明了标识符(变量,函数)。这些气泡彼此整齐地嵌套在一起,这个嵌套是在作者时间定义的。

但究竟是一个新泡沫呢？它只是功能吗？JavaScript中的其他结构可以创建范围的气泡吗？

##函数作用域

这些问题最常见的答案是JavaScript具有基于功能的范围。也就是说,您声明的每个函数为自己创建一个气泡,但没有其他结构创建自己的范围气泡。正如我们稍后会看到的,这并不完全正确。

但首先,我们来探讨函数作用域及其含义。

考虑这个代码:

```js
function foo(a) {
  var b = 2;

  // some code

  function bar() {
    // ...
  }

  // more code

  var c = 3;
}
```

在这个代码段中,`foo(..)'的范围包括```````````````````````` **没有关系** *其中*在声明出现的范围内,变量或函数属于包含范围气泡,无论如何。我们将在下一章中探讨这个*的工作原理。

`bar(..)`有自己的范围泡沫。全球范围也只有一个标识符:foo。

因为`a``````````````````````````````````````````````````````````````````````````````````````````````````````````````` 也就是说,以下代码将导致"ReferenceError"错误,因为标识符不可用于全局范围:

```js
bar(); // fails

console.log( a, b, c ); // all 3 fail
```

但是,所有这些标识符(`a`,```````````foo`````````````````````````````````````````````````````````````````` ..)`(假设`bar(..)'内没有阴影标识符声明)。

函数范围鼓励所有变量属于函数,并且可以在整个函数(甚至可访问的嵌套范围)中使用和重用。这种设计方法可能是非常有用的,当然可以充分利用JavaScript变量的"动态"性质,根据需要承担不同类型的值。

另一方面,如果您不采取谨慎的预防措施,整个范围内存在的变量可能会导致一些意想不到的陷阱。

##隐藏在平原范围内

思考功能的传统方式是声明一个函数,然后在其中添加代码。但反向思维同样强大和有用:采取任何您编写的任何代码段,并在其周围包装一个函数声明,实际上"隐藏"代码。

实际的结果是围绕所讨论的代码创建一个范围泡沫,这意味着该代码中的任何声明(变量或函数)现在都将被绑定到新的包装函数的范围,而不是先前包含的范围。换句话说,您可以通过将变量和函数包含在函数的范围内来"隐藏"。

为什么"隐藏"变量和函数是有用的技术？

激励这种基于范围的隐藏有各种各样的原因。他们倾向于产生于软件设计原则"最低权限原则"[^ note-leastprivilege],有时也称为"最低权限"或"最低限度曝光"。这个原则指出,在软件设计(如模块/对象的API)中,您应该仅显示最低限度的必需内容,并"隐藏"所有其他内容。

这个原则扩展到选择哪个范围包含变量和函数。如果所有变量和函数都在全局范围内,则它们当然可以由任何嵌套的范围访问。但是这样做会违反"最不重要的"原则,因为你可能会暴露许多变数或函数,否则这些变量或函数应该保持私有,因为正确使用代码将阻止对这些变量/函数的访问。

例如:

```js
function doSomething(a) {
  b = a + doSomethingElse( a * 2 );

  console.log( b * 3 );
}

function doSomethingElse(a) {
  return a - 1;
}

var b;

doSomething( 2 ); // 15
```

在这段代码中,`b`变量和`doSomethingElse(..)'函数可能是"doSomething(..)"工作的"私有"细节。给"b"和"doSomethingElse(..)"的封闭范围"访问"不仅不必要,而且也可能是"危险的",因为它们可能会以意想不到的方式有意使用, `doSomething(..)`的条件假设。

一个更"适当的"设计将隐藏这些私有细节在"doSomething(..)"的范围内,如:

```js
function doSomething(a) {
  function doSomethingElse(a) {
    return a - 1;
  }

  var b;

  b = a + doSomethingElse( a * 2 );

  console.log( b * 3 );
}

doSomething( 2 ); // 15
```

现在,`b`和`doSomethingElse(..)`不能被任何外界的影响所接受,而只能由`doSomething(..)`控制。功能和最终结果没有受到影响,但是设计保持私有细节是私有的,这通常被认为是更好的软件。

###碰撞避免

在范围内"隐藏"变量和函数的另一个好处是避免两个不同标识符之间的意外冲突,这两个标识符的名称相同,但不同的用途。碰撞结果往往意外重写价值。

例如:

```js
function foo() {
  function bar(a) {
    i = 3; // changing the `i` in the enclosing scope's for-loop
    console.log( a + i );
  }

  for (var i=0; i<10; i++) {
    bar( i * 2 ); // oops, infinite loop ahead!
  }
}

foo();
```

`bar(..)'中的`i = 3`赋值意外地覆盖在for循环中在`foo(..)'中声明的`i`。在这种情况下,它将导致无限循环,因为`i`设置为固定值`3`,永远保持`<10`。

`bar(..)`中的赋值需要声明一个局部变量来使用,无论选择什么标识符名称。`var i = 3;`会解决问题(并为'i'创建前面提到的"shadowed变量"声明)。一个*附加的*,而不是替代的选项是完全选择另一个标识符名称,例如`var j = 3;`。但是,您的软件设计可能自然地要求相同的标识符名称,因此利用范围"隐藏"您的内部声明是您的最佳/唯一选项。

####全局"命名空间"

在全球范围内发生(可能)变量冲突的一个特别强大的例子。如果加载到程序中的多个库不能正确隐藏其内部/私有函数和变量,那么很容易相互碰撞。

这样的库通常将在全局范围内创建一个单一变量声明,通常是具有足够独特的名称的对象。然后,该对象被用作该库的"命名空间",其中所有特定的功能暴露都作为该对象(命名空间)的属性,而不是顶级的词法作用域标识符本身。

例如:

```js
var MyReallyCoolLibrary = {
  awesome: "stuff",
  doSomething: function() {
    // ...
  },
  doAnotherThing: function() {
    // ...
  }
};
```

####模块管理

避免碰撞的另一个选择是使用任何各种依赖管理器的更现代的"模块"方法。使用这些工具,任何库都不会向全局范围添加任何标识符,而是需要通过使用依赖管理器的各种机制将其标识符显式导入另一个特定的范围。

应该注意到,这些工具不具有免除词法范围规则的"魔术"功能。他们只是使用这里所述的范围规则来强制不将任何标识符注入到任何共享作用域中,而是保留在专用的,不受碰撞影响的范围内,从而防止任何意外的范围冲突。

因此,您可以防御性地进行编码,并实现与依赖关系管理器相同的结果,而不需要实际使用它们,如果您这样选择。有关模块模式的更多信息,请参阅第5章。

##作为范围

我们已经看到我们可以采取任何代码片段,并围绕它传递一个函数,并且有效地"隐藏"来自该函数内部范围内的外部范围的任何封闭的变量或函数声明。

例如:

```js
var a = 2;

function foo() { // <-- insert this

  var a = 3;
  console.log( a ); // 3

} // <-- and this
foo(); // <-- and this

console.log( a ); // 2
```

虽然这种技术"工作",但不一定非常理想。引入了一些问题。第一个是我们必须声明一个命名函数`foo()`,这意味着标识符名称`foo`本身会污染包围的范围(在这种情况下是全局的)。我们还必须通过名称(`foo()`)显式调用函数,以使包装的代码实际执行。

如果函数不需要名称(或者,名称不会污染封闭的范围),并且函数可以自动执行,这将是更理想的。

幸运的是,JavaScript为两个问题提供了一个解决方案。

```js
var a = 2;

(function foo(){ // <-- insert this

  var a = 3;
  console.log( a ); // 3

})(); // <-- and this

console.log( a ); // 2
```

让我们分解一下这里发生了什么。

首先,请注意,包装函数语句以`(function ...`而不仅仅是函数...)开头,虽然这可能看起来像一个细微的细节,但实际上是一个重大的变化,而不是把函数当作一个标准声明,该函数被视为一个函数表达式。

**注意:**区分声明与表达的最简单的方法是在语句中的"function"一词的位置(不仅仅是一行,而是一个不同的语句)。如果"function"是声明中的第一件事,那么它是一个函数声明。否则,它是一个函数表达式。

在函数声明和函数表达式之间我们可以观察到的关键区别在于其名称作为标识符绑定的位置。

比较前两个片段。在第一个代码片段中,名称"foo"被绑定在封闭的范围内,我们直接用`foo()`调用它。在第二个代码片段中,名称"foo"不是绑定在封闭的范围内,而是仅在其自己的函数内部绑定。

换句话说,`(function foo(){ .. })`作为表达式意味着在`..`表示的范围内只发现'foo`',而不在外部范围内。在其中隐藏名称"foo"意味着它不会不必要地污染封闭的范围。

###匿名与命名

你可能最熟悉函数表达式作为回调参数,如:

```js
setTimeout(function(){
  console.log('我等了1秒！'');
},1000);
```

这被称为"匿名函数表达式",因为`function()...`没有名称标识符。函数表达式可以是匿名的,但函数声明不能​​忽略名称 - 这将是非法的JS语法。

匿名函数表达式可以快速方便地输入,许多库和工具都倾向于鼓励这种惯用的代码风格。不过,他们有几个回避要考虑:

匿名函数在堆栈跟踪中没有显示有用的名称,这可以使调试更加困难。

2.没有名称,如果函数需要引用自身,对于递归等,不幸的是,不推荐使用**`arguments.callee`引用。需要自引用的另一个例子是当一个事件处理函数想要在它触发之后解除绑定。

匿名函数省略一个通常有助于提供更易读/可理解的代码的名称。描述性名称有助于自我记录相关代码。

**内联函数表达式**是强大而有用的 - 匿名和命名的问题不会减损。为您的函数表达式提供一个名称,相当有效地解决了所有这些引用,但没有明显的缺点。最好的做法是永远命名你的函数表达式:

```js
setTimeout(function timeoutHandler(){// < - 看,我有一个名字！
  console.log("我等了1秒！");
},1000);
```

###立即调用函数表达式

```js
var a = 2;

(function foo(){

  var a = 3;
  console.log(a); // 3

})();

console.log(a); // 2
```

现在我们有一个函数作为一个表达式,因为它被包含在一个`()`对中,我们可以通过在末尾添加另一个`()`来执行这个函数,比如`(function foo(){..}) ()`。第一个包含`()`对使得函数成为一个表达式,第二个`()`执行该函数。

这种模式是很常见的,几年前,社区同意了一个术语:** IIFE **,代表** I ** mmedial ** ** ** ** ** ** ** ** ** ** ** * *表示。

当然,IIFE不需要名字,必然 -  IIFE的最常见形式是使用匿名函数表达式。虽然不太常见,但命名为IIFE的所有上述优点都超过匿名函数表达式,所以采用这是一个很好的做法。

```js
var a = 2;

(function IIFE(){

  var a = 3;
  console.log( a ); // 3

})();

console.log( a ); // 2
```

传统的IIFE形式有一些微小的变化,有些人喜欢:(function(){..}())`。仔细观察看看差异。在第一种形式中,函数表达式被包装在`()`中,然后调用`()`对就在外面。在第二种形式中,调用的`()`对移动到外部`()'包装对的内部。

这两种形式的功能是相同的。**这纯粹是您喜欢的风格选择。**

非常普遍的IIFE的另一个变体是使用它们实际上只是函数调用,并传入参数的事实。

例如:

```js
var a = 2;

(function IIFE( global ){

  var a = 3;
  console.log( a ); // 3
  console.log( global.a ); // 2

})( window );

console.log( a ); // 2
```

我们传入`window`对象引用,但是我们命名参数`global`,这样我们可以对全局和非全局引用有一个清晰的描述。当然,您可以从所需的封闭范围传入任何内容,您可以将参数命名为适合您的任何内容。这大多只是风格的选择。

此模式的另一个应用程序解决了(次要的利基)关注,默认的"未定义"标识符可能会将其值错误地覆盖,从而导致意外的结果。通过命名"undefined"参数,但不传递该参数的任何值,我们可以保证"undefined"标识符实际上是代码块中的未定义的值:

```js
undefined = true; // setting a land-mine for other code! avoid!

(function IIFE( undefined ){

  var a;
  if (a === undefined) {
    console.log( "Undefined is safe here!" );
  }

})();
```

IIFE的另一个变体反转了事物的顺序,其中执行的功能被给予第二,*之后*的调用和参数传递给它。该模式用于UMD(通用模块定义)项目。有些人觉得它有点清洁,虽然它稍微更详细一些。

```js
var a = 2;

(function IIFE( def ){
  def( window );
})(function def( global ){

  var a = 3;
  console.log( a ); // 3
  console.log( global.a ); // 2

});
```

`def`函数表达式定义在片段的后半部分,然后作为参数(也称为"def")传递给代码片段前半部分定义的"IIFE"函数。最后,调用参数`````````````````````````````````

##块作为范围

虽然功能是最常见的范围单位,但肯定是大多数JS流通中最广泛的设计方法,其他单位的范围也是可能的,这些其他范围单位的使用可以导致更好的,更清洁以维护代码。

JavaScript以外的许多语言支持Block Scope,因此来自这些语言的开发人员习惯于思维方式,而那些主要在JavaScript中工作的人可能会发现这个概念稍微外观。

但即使您从未以块范围的方式编写单行代码,您仍然可能熟悉JavaScript中非常常见的成语:

```js
for(var i = 0; i <10; i ++){
  console.log(i);
}
```

我们直接在for循环头部声明变量`i`,很有可能是因为我们的*意图*只是在该for循环的上下文中使用'i',并且基本上忽略了这个变量实际上将它自己定位的事实封闭的范围(功能或全局)。

这就是范围界定的一切。将变量尽可能地尽可能地尽可能地声明为使用它们的位置。另一个例子:

```js
var foo = true;

if(foo){
  var bar = foo * 2;
  bar = something(bar);
  console.log(bar);
}
```

我们正在if语句的上下文中使用一个`bar`变量,所以它使我们在if-block中声明它的一种感觉。但是,当使用`var`时,我们声明变量不相关,因为它们将始终属于封闭的范围。这个片段本质上是"假的"块范围,因为文体的原因,并且依赖于自我执行,不要在该范围的另一个地方意外地使用"bar"。

阻止范围是将早期的"最低权限原则~~曝光"[^ note-leastprivilege]从功能中隐藏信息到我们的代码块中隐藏信息的工具。

再次考虑for循环示例:

```js
for(var i = 0; i <10; i ++){
  console.log(i);
}
```

为什么要用for循环使用的`i`变量(或者至少应该*)至少会污染一个函数的整个范围？

但更重要的是,开发人员可能更喜欢*自己检查*意外(重新)使用超出其预期目的之外的变量,例如发现一个未知变量的错误,如果您尝试使用错误的地方。`i`变量的块范围(如果可能的话)会使`i'只能用于for-loop,如果`i`在函数的其他地方访问,会导致错误。这有助于确保变量不会被混淆或难以维护的方式重新使用。

但是,可悲的现实是,从表面上看,JavaScript没有用于块范围的功能。

也就是说,直到你再深入一点。

###`with`

我们在第2章中了解了"with"。虽然它是一个皱眉的构造,它*是一个(一种形式)块范围的示例,因为从对象创建的范围仅在该生命周期内存在`with`语句,而不是在封闭的范围内。

###`try / catch`

这是一个非常不为人知的事实,ES3中的JavaScript将`try / catch'的`catch`子句中的变量声明指定为'catch`块的块范围。

例如:

```js
try {
  undefined(); // illegal operation to force an exception!
}
catch (err) {
  console.log( err ); // works!
}

console.log( err ); // ReferenceError: `err` not found
```

你可以看到,`err`只存在于`catch`子句中,当你尝试在别的地方引用它时会抛出一个错误。

**注意:**虽然这个行为已经被规定了,但实际上所有标准的JS环境(除了可能是旧的IE)都是真实的,但是如果你有两个或更多的"catch"子句在同一个范围内,那么许多linter似乎仍然抱怨,他们的错误变量具有相同的标识符名称。这实际上并不是一个重新定义,因为这些变量是安全的阻止范围的,但是短信似乎仍然抱怨这个事实。

为了避免这些不必要的警告,一些开发人员将命名他们的"catch"变量"err1","err2"等。其他开发人员将简单地关闭重复变量名称的linting检查。

"catch"的区块性质似乎是一个无用的学术事实,但是有关更多信息,请参见附录B。

###`let`

到目前为止,我们已经看到JavaScript只有一些奇怪的利基行为会暴露块范围的功能。如果这是我们所有的,并且*这是*许多年,那么块的范围对JavaScript开发人员来说将是非常有用的。

幸运的是,ES6改变了这一点,并引入了一个新的关键字`let`,它与`var`一起作为另一种声明变量的方式。

`let`关键字将变量声明附加到它包含的任何块(通常是一个`{..}`对)的范围内。换句话说,`let`隐式地劫持了任何块的变量声明的范围。

```js
var foo = true;

if(foo){
  let bar = foo * 2;
  bar = something(bar);
  console.log(bar);
}

console.log(bar); // ReferenceError
```

使用`let`将变量附加到现有块是有些隐含的。如果您没有密切关注哪些块具有范围的范围,并且在开发和演进代码时习惯于移动块,将其包装在其他块等中,这可能会让您感到困惑。

创建块范围界定的显式块可以解决其中一些问题,使得变量附加而不是更明显。通常,显式代码优于隐式或微妙的代码。这种显式的范围界定风格很容易实现,更适合其他语言中的块范围工作:

```js
var foo = true;

if (foo) {
  { // <-- explicit block
    let bar = foo * 2;
    bar = something( bar );
    console.log( bar );
  }
}

console.log( bar ); // ReferenceError
```

我们可以通过在语句有效的语法的任何地方简单地包含一个`{..}`对,为`let`绑定创建一个任意的块。在这种情况下,我们已经在* if语句中做了一个显式的block *,在整个块中可以更容易地在重构中移动,而不会影响包含if语句的位置和语义。

**注意:**另一种表达明确阻止范围的方式,请参见附录B.

在第四章中,我们将讨论吊装问题,它将声明作为现有的整个范围。

然而,使用`let`作出的声明将不会提升到它们出现的块的整个范围。在声明语句之前,这种声明不会显着地"存在"。

```js
{
   console.log( bar ); // ReferenceError!
   let bar = 2;
}
```

＃＃＃＃ 垃圾收集

阻止范围界定有用的另一个原因是关闭和垃圾收集以回收内存。我们将在这里简要说明一下,但关闭机制在第5章中有详细介绍。

考虑:

```js
function process(data) {
  // do something interesting
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
  console.log("button clicked");
}, /*capturingPhase=*/false );
```

`click`功能点击处理程序回调根本不需要* someReallyBigData`变量。这意味着理论上,在"进程(..)"运行之后,大内存重的数据结构可能被垃圾收集。然而,很可能(尽管依赖于实现)JS引擎仍然需要保持结构,因为`click`函数在整个范围内都有一个闭包。

块范围可以解决这个问题,使其更清晰的引擎,它不需要保持`someReallyBigData`:

```js
function process(data) {
  // do something interesting
}

// anything declared inside this block can go away after!
{
  let someReallyBigData = { .. };

  process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
  console.log("button clicked");
}, /*capturingPhase=*/false );
```

声明变量的显式块本地绑定是一个强大的工具,您可以添加到您的代码工具箱。

####`let`循环

一个特定的例子就是我们之前讨论过的`let`照亮在for循环的情况。

```js
for(let i = 0; i <10; i ++){
  console.log(i);
}

console.log(i); // ReferenceError
```

for循环头部中的`let`不仅将`i'绑定到for-loop体,而且实际上它**将其绑定到循环的每个*迭代*,确保重新 - 从上一个循环迭代结束的值分配。

这是说明发生的每次迭代绑定行为的另一种方法:

```js
{
  让j
  for(j = 0; j <10; j ++){
    让i = j; //重新绑定每个迭代！
    console.log(i);
  }
}
```

当我们讨论闭包时,第5章将会清楚这个每次迭代绑定的原因。

因为`let`声明附加到任意的块而不是封闭函数的范围(或全局),所以存在一些隐藏的依赖于函数范围的`var`声明的代码,并将`var`替换为`let "重构代码时可能需要额外的小心。

考虑:

```js
var foo = true,baz = 10;

if(foo){
  var bar = 3;

  if(baz> bar){
    console.log(baz);
  }

  // ...
}
```

这个代码很容易被重新定义为:

```js
var foo = true,baz = 10;

if(foo){
  var bar = 3;

  // ...
}

if(baz> bar){
  console.log(baz);
}
```

但是,使用块范围变量时,请注意这些更改:

```js
var foo = true, baz = 10;

if (foo) {
  let bar = 3;

  if (baz > bar) { // <-- don't forget `bar` when moving!
    console.log( baz );
  }
}
```

请参阅附录B以获得更为显着的块样式范围,这样可以提供更容易维护/重构代码,这些代码对这些情况更加强大。

###`const`

除了`let`之外,ES6引入了`const`,它也创建一个块范围的变量,但其值是固定的(constant)。任何尝试以后更改该值会导致错误。

```js
var foo = true;

if (foo) {
  var a = 2;
  const b = 3; // block-scoped to the containing `if`

  a = 3; // just fine!
  b = 4; // error!
}

console.log( a ); // 3
console.log( b ); // ReferenceError!
```

##评论(TL; DR)

函数是JavaScript中最常见的范围单元。在另一个函数中声明的变量和函数本质上是"隐藏"的任何封闭"范围",这是一个有意的设计原则的好软件。

但功能决不是唯一的范围。块范围是指变量和函数可以属于任意块(通常是任何`{..}对)代码,而不仅仅是封闭函数的想法。

```````````````````````````````````````````

在ES6中,引入`let`关键字(表示`var`关键字的表达式),以允许在任意代码块中声明变量。`if(..){let a = 2; ``将声明一个变量`a`,它基本上是劫持`if```````的范围,并附加在这里。

虽然有些似乎相信这一点,但是范围不应该被视为直接替换"var"函数范围。这两个功能共存,开发人员可以并且应该使用函数作用域和块范围技术,分别适合于生成更好,更可读/可维护的代码。

[^ note-leastprivilege]:[最低权限原则](http://en.wikipedia.org/wiki/Principle_of_least_privilege)