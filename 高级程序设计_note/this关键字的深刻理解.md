- 环境
- 关键字this

# 1.环境


**总结的话**

---

this在函数中所指的环境是一开始就是全局，如果是全局的化就是windows如果是箭头函数的话不会改变this的所指向的this(一直是一开始引用的那个),因为箭头函数本身并没有this，要在函数外面拿this,如果是箭头函数嵌套一个普通函数的化要看是几层嵌套然后再判断。但是如果是普通函数有可能会更改，如果是付给一个对象的属性的话，环境this就会更改为这个对象里面的环境this，还有如果时普通函数里面附一个return arguments的话也会更改

用例子解释一下环境的含义：

我们最为熟悉的的环境就是全局环境 window。当你在全局声明一个变量时，你可以通过 window 找到你声明的变量。如：

```
var a = "hello world!";
window.a  //"hello world!"
typeof window   //"object"
```
这时候，我们可以发现，其实所谓环境就是对象。它可以只有一个对象，也可以包含多个对象，而环境变量就是这些对象里的属性罢了。如：
```
var age = 20;
var k = {
    name: 'k',
    fn: function(){
        return { 
            name: this.name,
            age: age
        }
    }
}
k.fn()  // { name: "k", age: 20 }
```
函数 k.fn 所在的环境就是 k 对象和 window 对象。
总的来说，环境就是函数访问到的作用域。

## 二.关键字
this 是一个指针，一般指向其所在函数的执行环境（作用域链）的第二个环境对象（或者说是指向[[scope]]的第一个环境对象），好像说是指向定义环境，什么是定义环境？


*补充知识*

1. 根据函数的内、外部，可以分为：函数的内部环境、函数的外部环境。


2. 根据函数的创建过程和执行过程，可以将函数环境分为两种：函数的定义环境、函数的执行环境。


3. 函数的执行环境就是我们通常说的 “上下文”。

4. 更改环境（即更改 this 的指向）的两类方法：**显式更改** 和 **隐式更改**。（在箭头函数里，this 的指向无法更改）

5. 执行环境(作用域链)=函数内部环境+定义环境[[scope]]


### 1.函数的定义环境
函数的定义环境举例：
```
ar a = 'window';
function say(){
    return a
}
say()   // "window"

function shout(){
    var a = 'fn';
    return say();
}
shout()  // "window"
```
我们可以看到无论我们什么时候调用say函数，返回结果都是"window".
函数在定义的过程中，会将需要访问的环境对象按优先顺序记录到函数的[[scope]]属性里面，而这个[[scope]]就是定义环境。

### 2.函数的执行环境


函数的定义环境好理解，但函数的执行环境就稍微复杂了一点。

函数在执行的过程中，会创建自己的内部环境对象，再与 [[scope]] 所记录的环境对象按优先顺序排列，形成作用域链，这就是执行环境。

以上面的 shout、say 嵌套函数为例：
```
var a = 'window';
function say(){
    var a = 'say';
    return this.a     // 原来的 "return a" 变成 "return this.a"
}
say()   // "window"

function shout(){
    var a = 'fn';
    return say();
}
shout()  // "window"
```
> 分析：
say 函数创建时，其 [[scope]] 为 [ 全局环境window ]
say 函数被 shout 调用时，say 形成的执行环境（作用域链）为 [ say函数的内部环境 , 全局环境window ]
所以，this 指向全局环境window

再看一个例子：

```
var a = 'window';
function say(){
    return this.a
}
var k = {
    a: 'k',
    shout: function shout(){
        console.log("shout函数内部的this.a => "+ this.a)
        return say();
    }
}
k.shout()   
// shout函数内部的this.a => k
// "window"
```
通过上面两个例子，我们可以知道，无论 say被哪一个函数调用，放回结果都是一样的。
### 4. this 指向的深度理解

接下来会将之前所说所说的全部推翻：

例1：
```
var x = 1
function a(){
    var x = 'a'
    function b(){
        var x = 'b'
        return this.x
    }
    return b()
}
a()
```
例2：
```
var x = 1
var obj = {
    x: 'obj',
    a: function a(){
        var x = 'a'
        function b(){
            var x = 'b'
            return this.x
        }
        return b()
    }
}
obj.a()
```
执行过上面的代码后，发现不对劲了，到底怎么回事？ 这个问题的解答，我们将在关键字this（填坑篇之函数创建）解决。

看完补坑篇的人可以回过头来想想，之前挖的坑究竟坑在哪了？哪里是坑？哪里不是坑？如果你分清楚了，就说明你理解的差不多了。

### 4.函数环境的更改(更改this的指向)
- 显式更改
```
var a = 1;
function say(){
    return this.a;
}

var second = {
    a: 2,
}

say.call(second)  // 2
say.apply(second) // 2
say.bind(second)()  // 2
say()     // 1     
```
- 隐式更改
```
var a = 1;
function say(){
    return this.a;
}

var second = {
    a: 2,
    say: say
}
second.say()  // 2
say()         // 1
```

---


