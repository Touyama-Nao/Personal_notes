一.Function构造器
函数其实是一个对象：
```
function fn(){
    return 'Hello World!'
}
typeof fn // "function"
fn instanceof Object // true
```
既然函数是对象，那函数是谁创建的？
```
fn.constructor  // Function
```
函数是由 Function 构造函数创建的。也就是说，当我们声明一个函数时，就相当于使用 Function 构造函数实例化了一个函数对象。

---
## 二、函数创建过程
先看一段代码：
```
var x = 1
function a(){
    var y = x
    console.log(this.x, y)
}
// 等价与
var a = function(){
    var y = x
    console.log(this.x, y)
}
```
在创建函数 a 创建时，由于函数 a 是在全局声明的，所以，在全局先调用 Function 构造函数。然后，就传入参数一样传入了 x 的引用（不是 x 的副本），而 y 是在函数 a 执行时才创建的，此时并没有创建。因为 Function 构造函数是一个全局函数，当它实例化一个函数并且为该实例函数绑定 this 的时候，只能将实例函数的 this 指向全局环境，一个匿名函数就创建好了。然后，再将该匿名函数赋给了变量 a。由于赋值给 a 的过程中，并没有发生 this 的重定向，所以，最后的 this 依旧指向全局。x 所在的环境将被记录到新函数的 [[scope]] 中。



![image](https://note.youdao.com/yws/api/personal/file/76E972C9824B496DA5057281D81AC353?method=download&shareKey=afdfb245d6117126845c32554702648a)


 **解释**  这里属性a先传入x的引用然后用Function构造函数构造匿名函数fn这时候其本身的this默认指向全局，这时候函数创建的时侯将this这个环境对象丢进[[scope]]了形成这个[[scope]]的第一个对象那个环境，然后在进行下面的匿名函数赋给a,this指向发生了隐式更改，指向全局
 
 
 
 所有函数刚被创建出来的时候，this 都是指向全局的，在赋值的时候 this 的指向才有可能会被隐式更改。

再看回这个代码是不是已经变得很简单啦：
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
obj.a() // 1，这里调用的是a函数，不是a方法。
```
为什么这里没有被隐式更改？

> 隐式更改只有在先定义构造函数再赋给类的水性才可以被更改，如果实在类里面先创建那么他的this肯定就是全局


对比之前隐式更改：
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

### 2.箭头函数

箭头函数是 ES6 的语法。之前我们说过，箭头函数的 this 的指向是不能被更改的。 先看下面的代码：

```
var x = 1
var obj = {
    x: 'obj',
    a: function a(){
        var x = 'a'
        var b = ()=>{ // 注意这里
            var x = 'b'
            return this.x
        }
        return b()
    }
}
obj.a()  //"obj“
```
**解释**  执行完后是不是又懵逼了。箭头函数的创建跟普通函数稍微有点不一样。

首先，函数 a 执行，声明箭头函数，调用 Function 构造函数，将函数 a 的 this 的引用传入 Function，创建匿名函数，但不为匿名函数创建自己的 this，因此，该函数只能使用传入来的 this。

函数a的this传给匿名函数，因为匿名函数是箭头函数自己没有this的，但是函数a的this是从obj的类里面来的；主要看函数是从哪各类中来的，是调用那个函数就从这个函数最近的环境中找。

我们用一个更加简单的例子来说明箭头函数的创建过程：

```
var x = 1
var a = ()=>{
    var y = x
    console.log(this.x, y)
}
var b = function(){
    var y = x
    console.log(this.x, y)
}
var obj = {
    x: 2,
    a: a,
    b: b
}
obj.a()  // 1 1
obj.b()  // 2 1
```
![image](https://note.youdao.com/yws/api/personal/file/FE36A7923008450092060D3AA2D6B6B1?method=download&shareKey=73a8f9064e863ce0f1d36cef3c8f7041)

**解释**这里a函数直接在全局中定义而且是箭头函数自然是两个都是1,1
后面的b函数，因为不是箭头函数，再obj类中将函数b赋给方法b，就会将里面的this隐式更改，但是这里的y用的是函数创建的时候一开始的x：函数创建的顺序：先是创建函数，然后闯入默认全局this然后将this指向的x赋给y然后再借用方法obj修改this。所以this.x和y是不一样的

再看一个例子：

```
var x = 1
var fn = () => {
    return this.x
}
var obj = {
    x: 2
}
fn.apply(obj)
```
我们会发现，箭头函数的 this 是没有办法被更改的。 因为无论显示更改还是隐式更改，都只能更改函数自身的 this，而箭头函数的 this 不是自己的，而是像普通变量一样从外部引进来的。


---

## 三.例子
```
var length = 1
function fn(){
    console.log(this.length)
}
var obj = {
    length: 2,
    f: function(fn){
        fn()  //1
        arguments[0]()  //4
    }
}
obj.f(fn, 1, 2, 3)
```
这里函数fn()是被当作参数传进来的所以没有被赋值，所以没有更改，但是arguments的话这个数组第一项是执行这个函数this的参数个数的。。。

可以先看下面的解释：

---


```
var length = 10;
function fn(){
    alert(this.length)
}
var obj = {
    length: 5,
    method: function(fn) {
        arguments[0]()
    }
}
obj.method(fn)//输出1
```
这里没有输出5，也没有输出10，反而输出了1，有趣。这里arguments是javascript的一个内置对象（可以参见mdn：arguments - JavaScript），是一个类数组（就是长的比较像数组，但是欠缺一些数组的方法，可以用slice.call转换，具体参见上面的链接），其存储的是函数的参数。也就是说，这里arguments[0]指代的就是你method函数的第一个参数：fn，所以arguments[0]()的意思就是：fn()。

不过这里有个疑问，为何这里没有输出5呢？我method里面用this，不应该指向obj么，至少也会输出10呀，这个1是闹哪样？

实际上，这个1就是arguments.length，也就是本函数参数的个数。为啥这里的this指向了arguments呢？因为在Javascript里，数组只不过使用数字做属性名的方法，也就是说：arguments[0]()的意思，和arguments.0()的意思差不多（当然这么写是不允许的），你更可以这么理解：

```
arguments = {
    0: fn, //也就是 functon() {alert(this.length)} 
    1: 第二个参数, //没有 
    2: 第三个参数, //没有
    ..., 
    length: 1 //只有一个参数
}
```
所以这里alert出来的结果是1。

如果要输出5应该咋写呢？直接 method: fn 就行了。

---
好了理解完了，懂了吧？
例2：
```
var x = 1
var obj = {
    x: 2
}
obj.fn = ()=>{
    return this.x
}
obj.fn()
```
例3：
```
var length = 100
var a = [
    1,
    2,
    3,
    ()=>{ return this.length }, 
    function(){ return this.length }
]
a[3]()  //100,函数不会更改
a[4]()  //5
```
例4：
```
var x = 1
var obj = {
    x: 2,
    fn: function(){
        var x = 3
        return ()=>{
            var x = 4
            return function(){
                return this.x
            }.apply(this)
        }
    }
}
obj.fn()  //返回：  
            ()=>{
            var x = 4
            return function(){
                return this.x
            }.apply(this)
        }
obj.fn()()  //2
```







