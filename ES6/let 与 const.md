# let
为es6新增命令，用来声明变量，类似于var,但是let所声明的变量，只在let命令所在的块级作用域内有效

块级作用域写法(ES6块级作用域允许任意嵌套)：
```
{
    let a = 10;
    var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```
上面代码在代码块之中，分别用let和var声明了两个变量。然后在代码块之外调用这两个变量，结果let声明的变量报错，var 声明的变量返回了正确的值。这表明，let 声明的变量只在它所在的代码块有效。

for循环的计数器，就很合适使用let命令。
```
for (let i = 0; i <10; i++) {}

console.log(i);
//ReferenceError: i is not defined
```
上面代码中,计数器只在for循环体内有效,在循环体外引用就会报错; 
下面的代码如果使用了var,最后输出的是10;

```
var a = [];
for (var i = 0; i <10; i++) {
  a[i] = function () {
  console.log(i);
};
}
a[6](); // 10
```
上面代码中，变量i是var声明的，在全局范围内都有效。所以每一次循环，新的i值都会覆盖旧值，导致最后输出的是最后一轮的i的值。

如果使用let，声明的变量仅在块级作用域内有效，最后输出的是6

```
var a = [];
for (let i = 0; i < 10; i++) {
    a[i] = function () {
        console.log(i);
    };
}
a[6](); // 6
```
上面代码中，变量i是let声明的，当前的i只在本轮循环有效，所以每一次循环的i其实都是一个新的变量，所以最后输出的是6。

下面讲一讲let的特性：

==let 不存在变量提升==
> 变量提升现象：浏览器在运行代码之前会进行预解析，首先解析函数声明，定义变量，解析完之后再对函数、变量进行运行、赋值等。
-不论var声明的变量处于当前作用域的第几行，都会提升到作用域的头部。 
-var 声明的变量会被提升到作用域的顶部并初始化为undefined，而let声明的变量在作用域的顶部未被初始化

let不像var那样会发生“变量提升”现象。所以，变量一定要在声明后使用，否则报错

```
console.log(foo); // 输出undefined
console.log(bar); // 报错ReferenceError
var foo = 2;
let bar = 2;
```

![image](https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/A13D8BBFB9234090882379BBF745B4E4/8047)

相当于：
```
// var 的情况
console.log(foo); // 输出undefined
var foo = 2;
//相当于
var foo;  //声明且初始化为undefined
console.log(foo);
foo=2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
//相当于在第一行先声明bar但没有初始化，直到赋值时才初始化

```
但是直接用let声明变量不赋值是会打印undefined，还是初始化了，只是let声明放在赋值之后，let声明会提前但不会初始化。
```
let a;
alert(a);//值为undefined

alert b;//会报错
let b;
```


上面代码中，变量foo用var命令声明，会发生变量提升，即脚本开始运行时，变量foo已经存在了，但是没有值，所以会输出undefined。变量bar用let命令声明，不会发生变量提升。这表示在声明它之前，变量bar是不存在的，这时如果用到它，就会抛出一个错误。

==暂时性死区==

只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

```
var tmp = 123;

if (true) {
tmp = 'abc'; // ReferenceError
let tmp;
}
```

![image](https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/2648E2E22B8344438E0E759430B9D3EA/8045)

上面代码中，存在全局变量tmp，但是块级作用域内let又声明了一个局部变量tmp，导致后者绑定这个块级作用域，所以在let声明变量前，对tmp赋值会报错。

ES6明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

总之，在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“++==暂时性死区==++”（temporal dead zone，简称TDZ）。
```
if (true) {
// TDZ开始
tmp = 'abc'; // ReferenceError
console.log(tmp); // ReferenceError

let tmp; // TDZ结束
console.log(tmp); // undefined

tmp = 123;
console.log(tmp); // 123
}
```
![image](https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/02616C52407F413B83BAF6C90FBD1E44/8044)
上面代码中，在let命令声明变量tmp之前，都属于变量tmp的“死区”。

“暂时性死区”也意味着typeof不再是一个百分之百安全的操作。
```
typeof x; // ReferenceError
let x;
```
面代码中，变量x使用let命令声明，所以在声明之前，都属于x的“死区”，只要用到该变量就会报错。因此，typeof运行时就会抛出一个ReferenceError。

作为比较，如果一个变量根本没有被声明，使用typeof反而不会报错。
```
typeof undeclared_variable // "undefined"
```
上面代码中，undeclared_variable是一个不存在的变量名，结果返回“undefined”。所以，在没有let之前，typeof运算符是百分之百安全的，永远不会报错。现在这一点不成立了。这样的设计是为了让大家养成良好的编程习惯，变量一定要在声明之后使用，否则就报错。

有些“死区”比较隐蔽，不太容易发现。
```
function bar(x = y, y = 2) {
return [x, y];
}

bar(); // 报错
```
上面代码中，调用bar函数之所以报错（某些实现可能不报错），是因为参数x默认值等于另一个参数y，而此时y还没有声明，属于”死区“。如果y的默认值是x，就不会报错，因为此时x已经声明了。
```
function bar(x = 2, y = x) {
return [x, y];
}
bar(); // [2, 2]
```
(假的，这两个都会报错的！)

ES6规定暂时性死区和let、const语句不出现变量提升，主要是为了减少运行时错误，防止在变量声明前就使用这个变量，从而导致意料之外的行为。这样的错误在ES5是很常见的，现在有了这种规定，避免此类错误就很容易了。

总之，暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。

不允许重复声明

let不允许在相同作用域内，重复声明同一个变量。
```
// 报错
function () {
let a = 10;
var a = 1;
}

// 报错
function () {
let a = 10;
let a = 1;
}
```
因此，不能在函数内部重新声明参数。
```
function func(arg) {
let arg; // 报错
}

function func(arg) {
{
let arg; // 不报错
}
}
```
## 块级作用域

为什么需要块级作用域？

ES5只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景。

第一种场景，内层变量可能会覆盖外层变量。
```
var tmp = new Date();

function f() {
console.log(tmp);
if (false) {
var tmp = "hello world";
}
}

f(); // undefined
```
上面代码中，函数f执行后，输出结果为undefined，原因在于变量提升，导致内层的tmp变量覆盖了外层的tmp变量。

内层函数变量声明提升！变成：

```
var tmp = new Date();
var tmp;
function f() {
console.log(tmp);
if (false) {
 tmp = "hello world";
}
}

f(); // undefined
```

[这里也有个例子](https://zhidao.baidu.com/question/1366893818772162859.html)

是这样的，在js的解释器(编译机制)里的规则是这样的

在作用域中的变量声明和方法声明都会呗编译器在编译的时候，

给强制挪到第一行，在开始执行,并且变量的默认值都是 `undefined
第二种场景，用来计数的循环变量泄露为全局变量。
```
var s = 'hello';

for (var i = 0; i < s.length; i++) {
console.log(s[i]);
}

console.log(i); // 5
```
上面代码中，变量i只用来控制循环，但是循环结束后，它并没有消失，泄露成了全局变量。
[这里也有个例子](https://blog.csdn.net/llsmingyi/article/details/82825124)

## ES6的块级作用域
let实际上为JavaScript新增了块级作用域。
```
function f1() {
let n = 5;
if (true) {
let n = 10;
}
console.log(n); 
}
f1(); // 5
```

![image](https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/E80C67C9CF1B41ABB26DC6957F54757A/8046)


上面的函数有两个代码块，都声明了变量n，运行后输出5。这表示外层代码块不受内层代码块的影响。如果使用var定义变量n，最后输出的值就是10。

ES6允许块级作用域的任意嵌套。

```
{{{{{let insane = 'Hello World'}}}}};
//上面代码使用了一个五层的块级作用域。外层作用域无法读取内层作用域的变量。

{{{{
{let insane = 'Hello World'}
console.log(insane); // 报错
}}}};
//内层作用域可以定义外层作用域的同名变量。

{{{{
let insane = 'Hello World';
{let insane = 'Hello World'}
}}}};
```
块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了。

块级作用域与函数声明

函数能不能在块级作用域之中声明，是一个相当令人混淆的问题。

ES5规定，函数只能在顶层作用域和函数作用域之中声明，不能在块级作用域声明。

```
// 情况一
if (true) {
function f() {}
}

// 情况二
try {
function f() {}
} catch(e) {
}
```
上面代码的两种函数声明，根据ES5的规定都是非法的。(为啥？)

但是，浏览器没有遵守这个规定，为了兼容以前的旧代码，还是支持在块级作用域之中声明函数，因此上面两种情况实际都能运行，不会报错。不过，“严格模式”下还是会报错。
```
// ES5严格模式
'use strict';
if (true) {
function f() {}
}
// 报错

//ES6 引入了块级作用域，明确允许在块级作用域之中声明函数。

// ES6严格模式
'use strict';
if (true) {
function f() {}
}
// 不报错
```
ES6 规定，块级作用域之中，函数声明语句的行为类似于let，在块级作用域之外不可引用。

```
function f() { console.log('I am outside!'); }
(function () {
if (false) {
// 重复声明一次函数f
function f() { console.log('I am inside!'); }
}

f();
}());
```
上面代码在 ES5 中运行，会得到“I am inside!”，因为在if内声明的函数f会被提升到函数头部，实际运行的代码如下。
```
// ES5版本
function f() { console.log('I am outside!'); }
(function () {
function f() { console.log('I am inside!'); }
if (false) {
}
f();
}());
//ES6 的运行结果就完全不一样了，会得到“I am outside!”。因为块级作用域内声明的函数类似于let，对作用域之外没有影响，实际运行的代码如下。

// ES6版本
function f() { console.log('I am outside!'); }
(function () {
f();
}());
```
很显然，这种行为差异会对老代码产生很大影响。为了减轻因此产生的不兼容问题，ES6在附录B里面规定，浏览器的实现可以不遵守上面的规定，有自己的行为方式。

允许在块级作用域内声明函数。 
函数声明类似于var，即会提升到全局作用域或函数作用域的头部。 
同时，函数声明还会提升到所在的块级作用域的头部。 
注意，上面三条规则只对ES6的浏览器实现有效，其他环境的实现不用遵守，还是将块级作用域的函数声明当作let处理。

[参考文献1](https://blog.csdn.net/zuiziyoudexiao/article/details/76890102)
[参考文献2](https://blog.csdn.net/qq_37133856/article/details/74359114/)



# CONST

内容，作用，含义：const是constant（常量）的缩写，const和 let一样，也是用来声明变量的，但是const是专门用于声明一个常量的，顾名思义，常量的值是不可改变的。

常量的特点：

1、不可修改

```
const c = 3.1415;
c=3;//错误，企图修改常量c
```
运行结果如图所示：

![image](https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/0CA44BE53F524B4D8512F2BB28C2E149/8071)

2、声明后必须要赋值。对于const来说，只声明不赋值，就会报错。

```
const c;
```
运行结果如图所示：

![image](https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/A2572754B39B41AFBAA0E4CAD90AED3E/8074)

3、 只在块级作用域起作用，这点与let命令一样。
```
if(true){
      const Name = "张三";
}
console.log(Name);//错误，在代码块{ }外，Name失效
```

![image](https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/F4E62FBE14A546168957D6E57088B98D/8072)

4、不存在变量提升，必须先声明后使用，这点也跟let命令一样。

```
alert(Name);//错误，使用前未声明
const Name = '张三';
```

![image](https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/EBA809CC57AA40999B51898FBE1A01F8/8075)

5、不可重复声明同一个变量。

```
var Name  = '张三';
const  Name = '李四';//错误，声明一个已经存在的变量Name
```

![image]https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/135C5EFC94524C9BB324486BEE356531/8073)


补充：


es6中新增了一个const。就是用来定义一个常量的。以前其实一直没有把这个放在 心上，觉得就是定义一个常量的，很easy,没有什么可以深入的。

问题来了：
```
let obj = {'num1' : 20, 'num2' : 30}
const obj1 = obj
const num = obj.num1 
obj.num1 = 40
```
那么，试问这时候如果输出obj1 和 num的值，分别是多少呢？让我们在谷歌浏览器中试验一下

![image](https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/35D447E67A3847C3A25B972002F9A613/8096)

答案很显而易见了。在obj的num1属性值改变了以后，obj1的值是随着对象的改变而改变了，但是num的值却并没有改变。也就是说：


const定义的对象，当对象改变了之后，const定义的值也会跟着改变。

cosnt定义的变量是一个对象的一个属性值，但是当对象属性值改变了以后，const定义的这个值并不会改变。

那么这是为了什么呢？

在计算机中，常量是放在栈中的，而对象是放在堆中的。对于对象赋值，const指向的仅仅是他的地址，cosnt仅仅是保证这个地址不改变，至于地址对应的数据，是可以进行改变的。举个栗子，现在可能在外工作很多人都是租的房子，假如你住在a公寓的a单元101，cosnt就仅仅是保证他指向的是这个地址，至于你房子里住的是哪些人，他是不关心的。~

而如果定义一个简单的数据类型，那这个数据他本身就是存在栈中的，所以不可以改变。
