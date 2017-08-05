# 你不知道的JavaScript-上

#### topic
1.作用域和闭包
2.this和对象原型

#### 背景
javascript是一门很有意思的编程语言，融合了很多编程语言的特点。上手特别容易，使用javascript开发项目也是很轻松的一件事。但是当你深入其中，你会遇到很多不太好理解的概念，比如闭包、this指向引用、原型链之类。这些模糊的概念，隐藏着javascript微妙之处；真正搞懂这些知识，对于理解javascript之门语言有很大的帮助。本文不讲javascript如何使用，主要是关于javascript中一些难点的分析和理解。

#### 整体梳理
有人说javascript中的三座大山：作用域、原型、异步。本文主要总结下前两个概念的知识点和个人的理解；关于异步的概念，单独讲起来也是很长的一个篇幅，可以尝试从回调函数——jquery中的异步处理——ES6中的Promise——Generator——ES7中的async/await。
本文从：作用域和闭包+this和对象原型，这两部分入手。那作者又是如何展开的呢？
任何编程语言最基本的功能都离不开存储和访问变量。要想存储和访问变量那就离不开作用域。作用域说白了就是一套规则，一套存储变量，并可以访问变量的规则。作用域类型可以分为：词法作用域、函数作用域以及块作用域。涉及到词法作用域就不能不提闭包这个非常作用而又神秘的概念。当某个函数可以记住并访问所在的词法作用域，且在当前词法作用域之外执行时就产生了闭包。当你能真正理解了闭包之后，你慢慢就可以理解并实现模块机制。想必模块思想不用多说，不管你是用vue还是react这样的开发框架，模块化的思想已经深入人心。模块化思想具有清晰，简洁，可复用的优点。
javascript中的作用域就是词法作用域，javascript中的词法作用域是一种静态的概念，既在代码的书写阶段就已经确定（如果不使用eval、with这类改变词法作用域的方法）；同时javascript中还存在某种很像动态作用域的机制，它就是this。this是一个很有趣的概念，它的动态绑定的特性决定了它的指向是一个比较难以理解的概念。根据调用位置的不同，this会绑定到不同的对象。说到对象，就引出了javascript中的一个精髓所在，原型链、原型继承。这是一个容易混淆的概念，经常会跟面向对象的类设计模式混淆。说到底javascript中的针对的是对象，对象之间的关联是委托关系。这部分内容经常跟模拟实现的类模式混在一起，并且很多语法糖和使用方法都在造成一种javascript类模式的错觉。不管是模拟实现的类的设计模式，还是对象关联的委托设计模式，其本质都是基于对象原型prototype实现的。至于使用哪种设计模式，仁者见仁，智者见智。

#### 作用域和闭包
###### 作用域是什么
作用域是一套规则，用来存储和访问变量。任何编程语言都不开作用域，正是作用域这种存储和访问变量的能力将状态带给了程序，赋予了编程语言可以实现丰富功能的能力。讲到作用域就不得不提两个重要角色：引擎和编译器。引擎从头到尾负责整个javascript程序的编译和执行过程。编译器负责词法分析、语法分析、代码生成等脏活累活。javascript是一种编译型的语言，但它不是提前编译的，它的编译发生在在代码执行前的几微秒。传统的编译过程分为三个阶段：A.词法分析；B.语法分析；C.代码生成。词法分析阶段主要将程序代码拆分成一个个的词法单元。比如将var  a = 2；拆分为var、a、=、2、；语法分析阶段主要将词法单元按照程序中的嵌套逻辑转换成“抽象语法树”；代码生成就是将抽象语法树装换为可执行代码的过程。javascript的编译过程比上面稍微复杂点，会在语法分析和代码生成阶段进行性能优化；一边编译优化、一边执行。
当一段程序被编译器编译完生成可执行代码，然后引擎执行它时，会对其中的变量进行查询，这个查询过程在作用域协助下，会从当前作用域开始，冒泡向上查找，找到即停止；如果没有找到，会一层层的嵌套进行，直到全局作用域为止。这个过程中变量查询主要分为：LHS查询（赋值操作的目标）或RHS查询（复制操作的源头）；如果全局作用域中也没找到，会根据查询类型不同抛出不同错误。LHS查询在严格模式下抛出ReferenceError，非严格模式下隐式的创建全局变量；RHS抛出ReferenceError错误。

###### 词法作用域
javascript中的绝大部分作用域都是词法作用。词法作用域是在词法阶段定义的作用域，所以在书写代码阶段已经确定，是个静态的概念。（跟后面的this动态的概念形成对比）。有两种方法可以修改词法作用域：eval和with；
【eval】通过代码中某处插入新的代码并运行，从而在代码执行阶段改变所在的词法作用域。
function foo(str,a) {
    eval(str);
    console.log(a,b);
}
foo("var b = 2;", 1);  //1,2
【with】常被用作重复引用一个对象中多个属性的快捷方式。接受一个对象作为参数，从而将一个对象处理为词法作用，创建一个全新的词法作用域。
function foo(obj) {
    with(obj){
        a = 2;
    }
}   
var obj1 = {a: 3};
var obj2 = {b: 3};
foo(obj1);
console.log(obj1.a); //2
foo(obj2);
console.log(obj2.a); //undefined
console.log(a);  //2
不提倡使用这两种方法，会导致性能下降，使得代码执行变慢。因为引擎在优化代码的时候依赖于词法进行静态分析，而它们会在代码执行阶段改变词法作用域，导致在编译阶段引擎无法对代码进行优化，使得性能下降。同时严格模式下with的行为也会被禁止。综上，不建议使用eval和with这两种方法。

###### 函数作用域和块作用域
前一章说到了作用域，那么javascript中不仅有函数作用域还存在块作用域。
【函数作用域】函数作用域中属于这个函数的全部变量都可以在整个函数范围内使用及复用。除了正常的声明一个函数然后定义函数外，我们还可以使用函数来包裹一个代码块，从而实现将代码块中的变量隐藏起来实现最小暴露原则；从而只暴露那些必要的变量或是函数，从而规避一些命名冲突。这里有两个典型的做法，第一种就是全局空间命名，用一个复杂的名字来定义某个对象，然后将要暴露出来的变量作为该对象的属性，规避变量名的冲突；第二种就是模块管理，强制所有标识符都不能注入共享的作用域中去。利用函数包裹代码块的做法本来就会带来两个问题：一、函数的声明本身就是对所在作用域的一种污染；二、函数需要调用运行才行。于是我们就想到了使用立即执行的匿名函数表达式。匿名函数相比于具名函数存在三个缺点：A.难以调试，追踪栈中不显示有意义的名字；B.难以调用，没有名字无法直接调用；C.难以理解，没有可读性的名字。所以一般不建议使用匿名函数。关于立即执行函数表达式，A.可以被当做函数调用传递参数进去；B.可以用来倒置代码的运行顺序。具体是否使用，可以根据具体场景来定。
【块作用域】使用函数的时候，我们可以通过函数作用域来规避一些潜在的冲突。但是当我们使用代码块的时候，就要时刻注意在作用范围之外可能存在的变量冲突。典型的：
for(var i=0; i < 10; i++) {
    console.log(i);
}
这里的i会被定义到全局，导致让人意想不到的冲突。为了解决这样的问题，引入块作用域可以很好的解决这样的问题。js中本身不存在块作用域，我们可以使用一下四种方式模拟实现块作用域。
A.with：在传入的对象中创建了块作用域；
B.try/with：catch分句中可以创建块作用域。可以用来模拟实现ES6之前的环境作为块作用域的替代方案；
C.let：可以在任意代码块中隐式的创建或是劫持块作用域；
D.const：同样可以用来创造块作用域；
不管是函数作用域还是块作用域，任何声明在某个作用域内的变量，都将附属于这个作用域。

###### 提升
上一部分说到，任何声明在某个作用域的变量都将附属于这个作用域。但是变量在作用域中声明的位置与作用域存在微妙的联系。不管是变量赋值还是函数定义，所有声明都会提升到各自作用域最顶部。
我们编写的程序会经过编译器进行编译，然后由引擎执行。这里的提升发生在编译器的编译阶段，也就是说变量、函数声明都会进行提升，但是变量赋值、函数执行不会。并且提升会限制各自所在的作用域中进行。其中函数的提升优先于变量提升；同时针对于函数提升有三个注意点：1、函数声明可以提升，但是函数表达式不提升；同时具名的函数表达式的标识符也不会提升。2、同名的函数声明，后面的覆盖前面的。3、函数声明的提升，不受逻辑判断的控制；

###### 作用域闭包
谈完了作用域，那我们就不得不提基于作用域的一个特别重要的概念：闭包。函数可以记住并访问所在的词法作用域时，就产生了闭包。它是基于作用域这个概念的。很典型的一个列子：
function foo(){
    var a = 2;
    function test(){
        console.log(a);
    }
    return test;
}
var func = foo();
func();  //2 ---这里就是闭包的效果——引用了foo中词法作用域中的变量
当某个函数可以记住并访问所在的词法作用域，那么就可以在其他地方使用这个闭包；并且这个被记住的词法作用域不会被销毁，可以一直被引用。
同时闭包是javascript中一个很重要的概念，很多异步操作中都会使用到。比如：定时器、事件监听器、ajax请求等。只要使用了回调函数，就一定存在闭包。
一个很常见的关于闭包的误解经常发生在循环中。比如：
for(var i=0; i < 6; i++) {
    setTimeout(function(){
        console.log(i);
    },0);
}
这里只会输出6个6。（这里定时器会等到循环执行完才会执行内面的内容）
为什么会这样呢？这里我们使用了闭包+块代码，其中块代码的作用域是全局的，所以当执行完循环之后运行setTimeout中闭包之后，其中引用的i就是全局公共区域中的i，也就是6。所以最终输出6个6.那么如何达到输出1到6个效果呢？我们可以通过作用域+闭包，解决循环中存在的问题。这里作用域可以通过函数实现or块作用域实现。
函数作用域：
for(var i=0; i < 6; i++) {
    (function(j) {
        setTimeout(function(){
            console.log(j);
        },0)
    })(i);
}
块作用域：
for(let i=0; i < 6; i++) {
    setTimeout(function(){
        console.log(i);
    },0)
}
谈到闭包，就不提一个很新的概念——模块模式。模块模式其实就是借助了闭包的思想。要实现一个模块模式需要具备两个必要条件：A.外部包裹函数+函数至少被调用一次返回实例；B.至少返回一个内部函数，才能形成闭包。

###### 小结
好了，到这里基本上已经将第一部分的内容，作用域和闭包的部分串起来进行了详细的梳理，希望能够对你有所帮助。敬请期待后续的：this和对象原型部分~







