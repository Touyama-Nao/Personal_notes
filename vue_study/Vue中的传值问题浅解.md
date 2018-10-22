# Vue v2.4中新增的$attrs及$listeners属性使用教程

> 这篇文章主要给大家介绍了关于Vue v2.4中新增的$attrs及$listeners属性的使用方法，文中通过示例代码介绍的非常详细，对大家的学习或者工作具有一定的参考学习价值，需要的朋友们下面随着小编来一起学习学习吧。

## 前言

多级组件嵌套需要传递数据时，通常使用的方法是通过vuex。如果仅仅是传递数据，而不做中间处理，使用 vuex 处理，未免有点杀鸡用牛刀。Vue 2.4 版本提供了另一种方法，使用 v-bind=”$attrs”, 将父组件中不被认为 props特性绑定的属性传入子组件中，通常配合 interitAttrs 选项一起使用。之所以要提到这两个属性，是因为两者的出现使得组件之间跨组件的通信在不依赖 vuex 和事件总线的情况下变得简洁，业务清晰。

### A 组件与 B 组件之间的通信： （父子组件）

![image](https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/AAB36F6EAFBE48F7862823E987E9F8A0/9353)

如上图所示，A、B、C三个组件依次嵌套，按照 Vue 的开发习惯，父子组件通信可以通过以下方式实现：
- A to B 通过props的方式向子组件传递，B to A 通过在 B 组件中 $emit, A 组件中 v-on 的方式实现
- 通过设置全局Vuex共享状态，通过 computed 计算属性和 commit mutation的方式实现数据的获取和更新，以达到父子组件通信的目的。
- Vue Event Bus，使用Vue的实例，实现事件的监听和发布，实现组件之间的传递。


### A 组件与 C 组件之间的通信： （跨多级的组件嵌套关系）

如上图，A 组件与 C 组件之间属于跨多级的组件嵌套关系，以往两者之间如需实现通信，往往通过以下方式实现：

- 借助 B 组件的中转，从上到下props依次传递，从下至上，$emit事件的传递，达到跨级组件通信的效果
- 借助Vuex的全局状态共享
- Vue Event Bus，使用Vue的实例，实现事件的监听和发布，实现组件之间的传递。


很显然，第一种通过props和$emit的方式，使得组件之间的业务逻辑臃肿不堪，B组件在其中仅仅充当的是一个中转站的作用。如使用第二种 Vuex的方式，某些情况下似乎又有点大材小用的意味，（仅仅是想实现组件之间的一个数据传递，并非数据共享的概念）。第三种情况的使用在实际的项目操作中发现，如不能实现很好的事件监听与发布的管理，往往容易导致数据流的混乱，在多人协作的项目中，不利于项目的维护。

$attrs以及$listeners的出现解决的就是第一种情况的问题，B 组件在其中传递props以及事件的过程中，不必在写多余的代码，仅仅是将$attrs以及$listeners向上或者向下传递即可。

A组件（App.vue）
```
<template>
 <div id="app">
 <child1
 :p-child1="child1"
 :p-child2="child2"
 v-on:test1="onTest1" //此处监听了两个事件，可以在B组件或者C组件中直接触发
 v-on:test2="onTest2"> 
 </child1>
 </div>
</template>
<script>
 import Child1 from './Child1.vue';
 export default {
 data () {
 return {};
 },
 components: { Child1 },
 methods: {
 onTest1 () {
 console.log('test1 running...');
 },
 onTest2 () {
 console.log('test2 running');
 }
 }
 };
</script>
```

B组件（Child1.vue）
```
<template>
 <div class="child-1">
 <p>in child1:</p>
 <p>props: {{pChild1}}</p>
 <p>$attrs: {{$attrs}}</p>
 <hr>
 <!-- C组件中能直接触发test的原因在于 B组件调用C组件时 使用 v-on 绑定了$listeners 属性 -->
 <!-- 通过v-bind 绑定$attrs属性，C组件可以直接获取到A组件中传递下来的props（除了B组件中props声明的） -->
 <child2 v-bind="$attrs" v-on="$listeners"></child2>
 </div>
</template>
<script>
 import Child2 from './Child2.vue';
 export default {
 props: ['pChild1'],
 data () {
 return {};
 },
 inheritAttrs: false,
 components: { Child2 },
 mounted () {
 this.$emit('test1');
 }
 };
</script>
```

结果：

in child1:

props: v_child1

$attrs: { “p-child2”: “v_child2”}

C 组件 (Child2.vue)


```
<template>
 <div class="child-2">
 <p>in child2:</p>
 <p>props: {{pChild2}}</p>
 <p>$attrs: {{$attrs}}</p>
 <hr>
 </div>
</template>
<script>
 export default {
 props: ['pChild2'],
 data () {
 return {};
 },
 inheritAttrs: false,
 mounted () {
 this.$emit('test2');
 }
 };
</script>
```
结果：

in child2:

props: v_child2

$attrs: {}

## 知识点总结

==$attrs==

 包含了父作用域中不被认为 (且不预期为) props 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 props 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind=”$attrs” 传入内部组件——在创建更高层次的组件时非常有用。
 
==$listeners==

包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on=”$listeners” 传入内部组件——在创建更高层次的组件时非常有用

==inheritAttrs==

默认情况下父作用域的不被认作 props 的特性绑定 (attribute bindings) 将会“回退”且作为普通的 HTML 特性应用在子组件的根元素上。当撰写包裹一个目标元素或另一个组件的组件时，这可能不会总是符合预期行为。通过设置 inheritAttrs 到 false，这些默认行为将会被去掉。而通过 (同样是 2.4 新增的) 实例属性 $attrs 可以让这些特性生效，且可以通过 v-bind 显性的绑定到非根元素上。

上述特性的使用完全可以降低在不使用Vuex以及事件总线的情况下，组件跨级props以及事件传递的复杂度。

## $attr 与 interitAttrs 之间的关系
==interitAttrs :==
默认情况下父作用域的不被认作 props 的特性绑定 (attribute bindings) 将会“回退”且作为普通的 HTML 特性应用在子组件的根元素上。当撰写包裹一个目标元素或另一个组件的组件时，这可能不会总是符合预期行为。

通过设置 inheritAttrs 到 false，这些默认行为将会被去掉。而通过 (同样是 2.4 新增的) 实例属性 $attrs 可以让这些特性生效，且可以通过 v-bind 显性的绑定到非根元素上。


注意：这个选项不影响 class 和 style 绑定。

### 子组件
```
<template>   <div>      {{first}}   </div> </template> <script> export default {   name: 'demo',   props: ['first'] } </script>
```
### 父组件
```
<template>  
<div class="hello">  
<demo :first="firstMsg" :second="secondMessage"></demo> </div> </template> <script> 
import Demo from './Demo.vue' export default { name: 'hello', components: { Demo }, data () {  return {irstMsg: 'first props',secondMessage: 'second props' } }, } </script>

```
父组件在子组件中进行传递 firstMsg 和 secondMsg 两个数据，在子组件中，应该有相对应的 Props 定义的接收点。

如果在 props 中定义了，你会发现无论是 firstMsg 和 secondMsg 都成了子组件的接收来的数据了，可以用来进行数据展示和行为操作。

虽然在父组件中在子组件模版上通过 props 定义了两个数据，但是子组件中的 Props 只接收了一个，只接收了 firstMsg，并没有接收 secondMsg，没有进行接收的此时就会成为子组件根无素的属性节点。


## interitAttrs = false 发生了什么 ？
```
<template>  <div>{{first}}  </div> </template> <script> export default { name: 'demo',  props: ['first'],  inheritAttrs: false, } </script>

```
对子组件进行一个改动，我们加上 inheritAttrs: false，从字面上的翻译的意思，取消继承的属性，然而 props 里仍然没有接收 seconed，发现就算 Props 里没有接收 seconed，在子组件的根元素上并没有绑定任何属性。

> $attrs

包含了父作用域中不被认为 (且不预期为) props 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 props 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind=”$attrs“ 传入内部组件——在创建更高层次的组件时非常有用。

在前面的例子中，子组件 props 中并没有接受 seconed，设置选项 inheritAttrs: false，同样也不会作为根元素的属性节点，整个没有接收的数据都被 $attr实例属性给接收，里面包含着所有父组件传入而子组件并没有在 Props里显示接收。


为了验证事实，可以在子组件中加上：
```
created () {       console.log(this.`$attrs`)    }
```
打印出来则是一个对象：{second: “secondMsg”, third: “thirdMsg”}

==注意==

想要通 $attr 接收，但必须要保证设置选项 inheritAttrs: false，不然会默认变成根元素的属性节点。

开头说了，最有用的情况则是在深层次组件运用的时候。创建第三层组件，作为第二层组件的子组件，在子组件引入的孙子组件，在模版上把整个 $attr 当数作数据传递下去，中间则并不用通过任何方法去手动转换数据。

子组件:
```
<template>   <div>      <next-demo v-bind="`$attrs`"></next-demo>   </div> </template>
```
孙子组件：
```
<template> <div>{{second}}{{third}} </div> </template> <script> export default { props : [ 'second' , 'third'] } </script>

```
孙子组件在 props 接收子组件中通过 $attr 包裹传来的数据，同样是通过父组件传来的数据，只是在子组件用了 $attrs 进行了统一接收，再往下传递，最后通过孙子组件进行接收。

依次类推孙子组件仍然不想接收，再传入下级组件，我们仍然需要对孙子组件实力选项进行设置选项 inheritAttrs: false，否则仍然会成为孙子组件根元素的属性节点。

从而利用 $attrs 来接收 props 为接收的数据再次向下传递是一件很方便的事件，深层次接收数据我们理解了，那从深层次向层请求改变数据如何实现。意思就是让顶层数据和最底层数据进行一个双向绑定。

### $listeners
listeners 可以认为是监听者。

向下如何容易的传递数据已经了解了,面临的问题是如何向顶层的组件如何改变数据，父子组件可以通过 v-model，.sync,v-on 等一系列方法，深层及的组件可以通过 $listeners 去管理。

$listeners 和 $attrs 两者表面层都是一个意思，$attrs 是向下传递数据，$listeners 是向下传递方法，通过手动去调用 $listeners 对象里的方法，则原理就是 $emit 监听事件，$listeners 也可以看成一个包裹监听事件的一个对象。

父组件：
```
<template>  <div class="hello">     {{firstMsg}}     <demo v-on:changeData="changeData" v-on:another = 'another'></demo>  </div> </template> <script>  import Demo from './Demo.vue'  export default {    name: 'hello',    components: {      Demo    },    data () {      return { firstMsg: '父组件', }    },    methods: {      changeData (params) {         this.firstMsg = params      },      another () {alert(2)      }    }  } </script>


```
在父组件中引入子组件，在子组件模板上面进行 changeData 和 another 两个事件监听，其它这两个监听事件并不打算被触发，而是直接被调用,再简单的理解则是向下传递两个函数。

```
<template>   <div>      <p @click="$emit('another')">子组件</p>        <next-demo  v-on='`$listeners`'></next-demo>   </div> </template> <script> import NextDemo from './nextDemo.vue' export default {   name: 'demo',   components: { NextDemo   },   created () {     console.log(this.`$listeners`)   }, } </script>

```
在子组件中，引入孙子组件 nextDemo，在子组件中，像 $attrs 一样,可以用 $listeners 去整体接收监听的事件，{changeData: ƒ, another: ƒ} 以一个对象去接收。

此时在父组件中在子组件模板上监听的两个事件不但可以被子组件实例属性 $listeners 去整体接收，并且同时可以在子组件进行触发。

同样的在孙子 nextDemo 组件中，继续向下传递,通过 v-on 把整个 $listeners 所接收的事件传递到孙子组件中，只是通过 $listeners 把其所有在父组件拿到监听事件一并通过 $listeners 一起传递到孙子组件里。

#### 孙子组件

```
<template>  <div class="hello">      <p @click='`$listeners`.changeData("change")'>孙子组件</p>  </div> </template> <script> export default {  name: 'demo',  created () {      console.log(this.`$listeners`)  }, } </script>

```
依然能拿到从子组中传递过来的 $listeners 所有的监听事件,此时并不是通过 emit 只是针对于父子组件的双向通信，$listeners 包了一个对象，分别是 changeData 和 another，通过 $listeners.changeData(‘change’) 等于直接触发了事件，执行监听后的回调函数,就是通过函数的传递，调用了父组件的函数。


### mixin

对于 mixin 这个混合来说，用一句话来总结，无论运用到组件上还是路由页面里，样式不同，而所有的操作都逻辑都 一样，就可以公用一个混 合，这个不但可以节省代码。

而且也可以维护一种 mixin，云客官有上下模式和左右布局模式，除了 better-scroll 针对于滑动模式的不同，其余的业务和用户操作都是一样，所以可以大量的采用 mixin。


### Vuex 如何正确使用

如果数据传递和共享不进行跨路由或者有着大量的牵扯，建议用 $attrs 和 $listeners 进行数据和行为的交互，因为这样能更加直观和明确的在组件业务中操作，毕镜 vuex 属于数据共享的原则，只要你保证正确的数据单向流动，而 vuex 在跨路由的情况下就能显示出自己的优势。


> vuex 跨路由操作，必然只能在子路由操作

在 h5 页面中，当使用 vuex 进行跨路由操作时，会导致当用户刷新页面的时候，vuex 同样的也会初始化，同时就会导致报错。

那如果是子路由的话，必然所有数据都是通过父路由进行数据操作完毕之后，放入数据共享，然后再由子路去获取，这样就算刷新页面不会导致数据错误。

