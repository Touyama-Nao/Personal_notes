# js函数与对象的关系

---
**明确概念**

==函数==函数是由事件驱动的或者当它被调用时执行的可重复使用的代码块。(百度)

==对象==JavaScript中，一切都是对象，函数也是对象，数组也是对象，但是数组是对象的子集。 函数实际上是对象，而且与其他引用类型一样具有属性和方法，由于函数是对象，因此函数名实际上也是一个指向函数对象的指针，不会与某个函数绑定。(MDN)

（高程）对象的定义：无需的集合。其属性可以包含基本值，对象或函数。严格来说对象是一组没有特定顺序的值。对象的每个属性和方法都有一个名字，而每个名字都映射一个值，其实就是一组名值对，其中值可以是函数或者数据。基本值类型不是对象（number、string、Boolean、Undefined），剩下的引用类型（函数、数组、null...）都是对象，也有人说对象是若干属性的集合。


1.函数是一种对象，但是函数有并不是对象的一种，因为他们两像蛋生鸡又像鸡生蛋。

2.对象都是由函数创建的
```
function test() {
    this.name="哈哈"
};
var test2=new test();
console.log(test2 instanceof Object);    //true
```
这个例子可以说明对象可以被函数创建。那为什么要说对象都是通过函数创建的，那对象字面量是不是也是通过函数来创建的，答案是肯定的，这是一种语法糖方式。举个简单的例子:

```
var obj={
    name:"哈哈",
    age:"18"
}
var obj=new Object()
obj.name="哈哈";
obj.age="18"
```
上面的对象字面量其实是通过下面的构造函数来创建的。而其中的Object是一种函数:：
```
console.log(typeof Object)    //function
```
通过上面的简单例子我们可以得出一个结论：对象是通过函数创建的，而函数又是一种对象。那么这是为什么呢？这就牵扯到prototype原型。


++一些有助于理解的说法++：

> avaScript中，一切都是对象，函数也是对象，数组也是对象，但是数组是对象的子集。 函数实际上是对象，而且与其他引用类型一样具有属性和方法，由于函数是对象，因此函数名实际上也是一个指向函数对象的指针，不会与某个函数绑定。

> 对象都是由函数创建的。 所有的对象都是由Object继承而来，而Object对象却是一个函数。

> 每一个函数实际上都是Function类型的实例

借助prototype,proto,constructor这三个属性，先从最简单的开始说起：

*补充概念*

1.构造函数（用于创造实例）prototype指向对象的原型，但是对象的原型有construtor指向构造函数，函数实例对象指向原型，注意并非指向构造函数。
![image](https://note.youdao.com/yws/api/personal/file/56F2AEC96D74454D9261CD1F91FD0A06?method=download&shareKey=01b2260a1d02775789c56e4a366b6187)


2.对于[[Prototype]]
每一个对象都有一个这样的隐藏属性(对象名字._proto_)，它引用了**创建这个对象的函数的prototype**==原型对象==（就是构造函数对象的原型），我们来看一张图：
![image](https://note.youdao.com/yws/api/personal/file/B3AD88B9C12440A6A41AF1E2A294132C?method=download&shareKey=d0c02df350ce6a82121649a37daa74cd)

注意：函数也是对象，自然它也有__proto__。 在控制台中，我们发现：
![image](https://note.youdao.com/yws/api/personal/file/4B33DFBEC0044C5F8CC26E0D598E8D50?method=download&shareKey=5cc10aea3d5eb898de6de920c2e6d30a)


即函数的__proto__是函数类型。（也就说函数的原型对象是函数，而函数也是对象，所以函数的原型还是对象）
![image](https://note.youdao.com/yws/api/personal/file/973FBD70EE344698911455FE072E592E?method=download&shareKey=b96fce7a94ec674df56faaea10c2d002)

3。*构造函数*

不同于其它的主流编程语言，JavaScript的构造函数并不是作为类的一个特定方法存在的；**当任意一个普通函数用于创建一类对象时，它就被称作构造函数，或构造器。**一个函数要作为一个真正意义上的构造函数，需要满足下列条件：

1、 在函数内部对新对象（this）的属性进行设置，通常是添加属性和方法。

2、 构造函数可以包含返回语句（不推荐），但返回值必须是this，或者其它非对象类型的值。


下面一步步讲解，明白了下面这个图就很好懂了：

## 1.1.原型创建对象机制

通过原型模式创建对象，我们接触到了对象创建的机制，如图：

1. 简单new 一个object的实例，然后再添加属性与方法：

```
var o1 = new Object()
var o2 = new Object()
```
![image](https://note.youdao.com/yws/api/personal/file/42B23F9A71EE4124A6F630612A3DAE19?method=download&shareKey=c1bea617e2eff194ff25aec7971f5f55)
**解释**左边两个小球是他的函数对象实例也就是o1,o2，对象那个里面有一个内置隐藏属性_proto_指向基本类型object的原型，object()类型类型本身有一个已经在js中配好的构造函数(中间那棵树function object())用来构造object类型.

接下来，再看另一种构造函数创造机制：
这个与上面的唯一区别就是将函数命名了：

```
function foo() {}
var f1 = new foo()
var f2 = new foo()
```
**解释**下面这个图是在上面这个图的基础上建立的：
![image](https://note.youdao.com/yws/api/personal/file/CFED8F79509D4E5F8706A0ECBDE1E7F5?method=download&shareKey=f5d2bdc2660ae99ee975e8a3ae654e8c)
> 下面这个图的下半部分就是上面这个图，上半部分是新增的foo()部分，跟上面的解释一样左边两个小球是新创建的对象，中间是创建对象foo()的构造函数，这个构造函数的prototype自然是指向原型foo.prototype，然后原型foo.prototype的constructor指向构造函数，但是顺着原型链原型foo.prototype又作为对象拥有_proto_指向所有原型的本源object.prototype;

但是本源object.prototype作为对象其内中置原型又是指向哪里呢？**竟然是null**

由于一切对象（除Object.prototype）都继承自Object,所以不难理解这里最大的boss是Object.prototype,而构造它的原型对象是null，这不是证明了它就是最大的boss吗？然而，并没有那么简单，那么，function()又是怎么回事呢？继续看：

![image](https://note.youdao.com/yws/api/personal/file/180D27A381E3415C9ADFB51C5F509E35?method=download&shareKey=3677f3cdd2995fe9b93f9b724d661fd9)


**首先解释这副图的最下面部分**中间的function Function是所有函数的构造函数，先说function object() (object的构造函数)是一个对象他的_proto_原本应该指向function Function但是function object()和function Function（函数的构造函数）本质是一样的都是 **对象(Function)**因此指向构造这个对象(函数都是Function对象)的函数（好像说的就是自己function Function其实也是Function对象）的protoytpe原型；

构造function Function这种函数的构造函数的其实也是function Function自己，也是Function对象

从另外一个角度说，function Function（函数作为Function对象的构造函数）的prototype是指向其对象原型Function.prototype（Function对象的原型），同时Function.prototype（Function对象的原型）的constructor也会指向他的对象构造器function Function（函数作为Function对象的构造函数）
但是

我们知道，一切函数都是Function的实例，而参考点击（指向MDN构造函数参考网页）这里 ，我们发现：
> Function 构造函数（简单来说就是函数的构造函数）创建一个新的Function对象（就是函数）。在JavaScript中,每个函数实际上都是一个Function对象。
> 全局的Function对象没有自己的属性和方法, 但是, 因为它本身也是函数，所以它也会通过原型链从Function.prototype上继承部分属性和方法。

这句话“Function从Function.prototype上继承部分属性和方法”，可以解释：

ps:（这里为什么说是部分？）
==原因==这个部分的属性和方法是作为函数的时候通过原型链在所有函数对象的原型本源Function.prototype继承回来的，但是Function.prototype本身是对象所以他会自己有方法和属性。
```
Function.__proto__==Function.prototype
```
即可以说函数是由Function创建的，那么Function由自身创建，所以Function.__proto__就指向了创建它的函数（也就是自己）的prototype。

最后，再来一张总的图：
![image](https://note.youdao.com/yws/api/personal/file/D2666B39F14D48AAB1023203F6D5A955?method=download&shareKey=4f27c6f99e5e928e1b7dade3596fd869)

通过这张图我们就可以轻松知道:
![image]https://note.youdao.com/yws/api/personal/file/4ADD0EA2D0AC483695997ACEDDAEE8C9?method=download&shareKey=b6dc3ba3f3967bfa5a14d71db30759fc)


再总结一下，函数与对象的关系，看来，最大的boss还是Object.prototype，因为从箭头的进出来看，Object.prototype是没有被谁创造出来的，而其他对象均可以找到创建它的原型对象。

而上面所说的
> 对象都是由函数创建的。
> 一切都是对象，函数也是对象



jq构建对象的机制：
```
function s(){
    this.a = 100;
}
s.prototype.fn = function(){
    this.a = 10000;
}
f(){
    this.a = 10000;
}
var x = new s();  //undefined

x
// s{a:100}
求jquery("#id");jq寻找对象的简写机制的实现方法
```

*以下为解答*
```

function S(){
this.age=1;
}
function newS(){
    return new S.prototype.init();
}
S.prototype.fn = function(){
    this.a = 1000;
}
s.prototype.init = function(){
    this.a = 100;
}  //s原本原型上定义方法
s.prototype.init.prototype = s.prototype  //这里用prototype是因为让方法的原型继承右边的原来对象的原型，原型式继承这样的话不会有继承s实例原来的方法

var x = newS()
//undefined
x
//s.init{a:100}
相当于重写原型
x.constructor
f s(){
    return new s.prototype.init()
}
```
下面的跟上面是一样的，只不过还是那个免得s函数不是写成构造函数的样子。

```
function s(){
this.age=1;
}//s函数实例原来的方法
function newS(){
    return new s.prototype.init();
}
s.prototype.fn = function(){
    this.a = 1000;
}//s函数式例上定义的原型方法，好像没用
s.prototype.init = function(){
    this.a = 100;
}//s函数式例上定义的原型方法
s.prototype.init = s//没有原型继承，知识引用同一个对象，所以没有继承原型，知识引用了相同的实例方法

var x = newS()
ƒ s(){
this.age=1;
}


x
s {age: 1}
```
```
function s(){
this.age=1;
}//s函数实例原来的方法
function newS(){
    return new s.prototype.init();
}
s.prototype.fn = function(){
    this.a = 1000;
}//s函数式例上定义的原型方法，好像没用
s.prototype.init = function(){
    this.a = 100;
}//s函数式例上定义的原型方法，好像没用
s.prototype.init.prototype = s.prototype//这里用prototype是因为让方法的原型继承右边的原来对象的原型，原型式继承这样的话不会有继承s实例原来的方法

var x = newS()
{fn: ƒ, init: ƒ, constructor: ƒ}
x
s.init {a: 100}a: 100__proto__: Objectfn: ƒ ()arguments: nullcaller: nulllength: 0name: ""prototype: {constructor: ƒ}__proto__: ƒ ()[[FunctionLocation]]: VM1376:7[[Scopes]]: Scopes[1]init: ƒ ()constructor: ƒ s()__proto__: Object


x.constructor
ƒ s(){
this.age=1;
}
```



