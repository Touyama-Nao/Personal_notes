# 基本示例

```
// 定义一个名为 button-counter 的新组件
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```
> 注意当点击按钮时，每个组件都会各自独立维护它的 count。因为你每用一次组件，就会有一个它的新实例被创建。

## data必须要是一个函数

```
data: {
  count: 0
}  //不是这一种

data: function () {
  return {
    count: 0
  }
}

```
### 组件的组织
组件通常会以一棵树来组织：

![image](https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/540152A078CB4495A481A84CF58837B9/6181)
#### 全局注册
```
Vue.component('my-component-name', {
  // ... options ...
})
```
全局注册的组件可以用在其被注册之后的任何 (通过 new Vue) 新创建的 Vue 根实例，也包括其组件树中的所有子组件的模板中。

父向子组件传值:
```
Vue.component('blog-post', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
})

<blog-post title="My journey with Vue"></blog-post>
<blog-post title="Blogging with Vue"></blog-post>
<blog-post title="Why Vue is so fun"></blog-post>

```

如上所示，你会发现我们可以使用 v-bind 来动态传递 prop。这在你一开始不清楚要渲染的具体内容，比如从一个 API 获取博文列表的时候，是非常有用的。
```
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
></blog-post>
```
==注意==每个组件碧血最外面要有且只有一个div（父元素）包裹

当要要传的值太复杂的时候可以直接将一个对象赋过去。

```
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
  v-bind:content="post.content"
  v-bind:publishedAt="post.publishedAt"
  v-bind:comments="post.comments"
></blog-post>
```
修改为：
```
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:post="post"
></blog-post>

Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <div v-html="post.content"></div>
    </div>
  `
})
```
## 子组件向父组件传递数据：
可以用$emit来通知
```
//子组件向父组件传递一个原生事件，通知父组件
<button v-on:click="$emit('enlarge-text')">
  Enlarge text
</button>

//父组件通过v-on监听来获取
<blog-post
  ...
  v-on:enlarge-text="postFontSize += 0.1"
></blog-post>
```
### 使用事件抛出一个值
方法：可以使用 $emit 的第二个参数来提供这个值：
```
<button v-on:click="$emit('enlarge-text', 0.1)">
  Enlarge text
</button>
```
然后当在父级组件监听这个事件的时候，我们可以通过 $event 访问到被抛出的这个值：
```
<blog-post
  ...
  v-on:enlarge-text="postFontSize += $event"
></blog-post>
```
或者，如果这个事件处理函数是一个方法：
```
<blog-post
  ...
  v-on:enlarge-text="onEnlargeText"
></blog-post>
```
那么这个值将会作为第一个参数传入这个方法：
```
methods: {
  onEnlargeText: function (enlargeAmount) {
    this.postFontSize += enlargeAmount
  }
}
```
## 在组件上使用v-model
自定义事件也可以用于创建支持 v-model 的自定义输入组件。记住：
```
<input v-model="searchText">
```
等价于：
```
<input
  v-bind:value="searchText"
  v-on:input="searchText = $event.target.value"
>
```
==解释==
> 1、将输入框的值绑定到searchText变量上，这个是单向绑定，意味着改变searchText变量的值可以改变input的value，但是改变value不能改变searchText
2、监听input事件（input输入框都有该事件，当输入内容时自动触发该事件），当输入框输入内容就单向改变searchText的值

那如果我们想在自定义组件中使用v-model呢？类似如下的写法:
```
<custom-input v-model="searchText"></custom-input>
```
等价于：
```
<custom-input
  v-bind:value="searchText"
  v-on:input="searchText = $event"
></custom-input>

```
我们怎么做？

（1）、首先我们需要将父组件的value值通过props传递给子组件

（2）、当子组件发现value值变化之后，需要通知父组件时候，触发该组件的input事件，并$emit给父组件

修改一下官方的代码：
```
Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)
    >
  `
})
```
只是在input事件触发之后，$emit事件出去，并将value值通知出去即可。
## 通过插槽分发内容

- 工具：Vue 自定义的 <slot> 元素
例子：
```
Vue.component('alert-box', {
  template: `
    <div class="demo-alert-box">
      <strong>Error!</strong>
      <slot></slot>
    </div>
  `
})
```
## 动态切换组件
方法：

上述内容可以通过 Vue 的 <component> 元素加一个特殊的 is 特性来实现：
```
<!-- 组件会在 `currentTabComponent` 改变时改变 -->
<component v-bind:is="currentTabComponent"></component>
```
在上述示例中，currentTabComponent 可以包括
> 已注册组件的名字，或
一个组件的选项对象


