> 在半年前尤大就不推荐使用vue-resource了，好像我这么没安全感的人，没人维护的东西不敢碰。

首先是简介：

那么axios这个是什么呢？是一个国外友人开发的基于Promise 用于浏览器和 nodejs 的 HTTP 客户端。它有什么用法呢：
从浏览器中创建 XMLHttpRequest

从 node.js 发出 http 请求

支持 Promise API

拦截请求和响应

转换请求和响应数据

取消请求

自动转换JSON数据

客户端支持防止 ![CSRF/XSRF](http://baike.baidu.com/link?* url=iUceAfgyfJOacUtjPgT4ifaSOxDULAc_MzcLEOTySflAn5iLlHfMGsZMtthBm5sK4y6skrSvJ1HOO2qKtV1ej_)

# 安装
```
npm install axios
```
(我最后用的是cnpm...)
如果不用npm，可以通过cdn引入
```
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```
# axios 基本用法
这里笔者使用的是es6，由于标题是要结合vue，因此将vue、axios以及vue-axios引入。
```
import Vue from 'vue'
import axios from 'axios'
import VueAxios from 'vue-axios'
#通过use方法加载axios插件
Vue.use(VueAxios，axios)
```
基本用法和模板：
```
axios.request(config)

axios.get(url[, config])

axios.delete(url[, config])

axios.head(url[, config])

axios.post(url[, data[, config]])

axios.put(url[, data[, config]])

axios.patch(url[, data[, config]])

```
# GET用法
```
 var vm = this
  this.$http.get(vm.testUrl).then((response)=>{
                    alert(response.data.msg);
   })
```
这里我们通过this.$http去调用axios，如果之前你的vue-resourse也是这么写的话，那简直可以无缝切换。当然你你换成this.axios也是没有问题的，但扩展性就不好了。

# POST用法（贼坑）
.vue模板：
```
  var vm = this
  this.$http.post(vm.testUrl,{"name":"痞子达"})
  .then((response)=>{
          alert(response.data.msg);
  })
  
```
testUrl:
```
public function test(){
        return json(array('statys'=>1,'msg'=>$_POST['name']));
    }
```
正常应该弹出“痞子达”，但是并没有，还报了500错误。
接口提示未定义数组索引: name
![image](https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/A80D99337B13446FA0504625A62B5E93/7684)

抓包看了看，是以Request Payload的形式传送了参数。
不是我们熟悉的form-data形式，看看api：
```
axios.post(url[, data[, config]])
```
第三个参数是config配置，这个配置应该可以做点事儿。这个config的参数有很多，先看看（随便瞅下就行）：

url —— 用来向服务器发送请求的url

method —— 请求方法，默认是GET方法

baseURL —— 基础URL路径，假如url不是绝对路径，如https://some-domain.com/api/v1/login?name=jack,那么向服务器发送请求的URL将会是baseURL + url。

transformRequest —— transformRequest方法允许在请求发送到服务器之前修改该请求，此方法只适用于PUT、POST和PATCH方法中。而且，此方法最后必须返回一个string、ArrayBuffer或者Stream
。
transformResponse —— transformResponse方法允许在数据传递到then/catch之前修改response数据。此方法最后也要返回数据。

headers —— 发送自定义Headers头文件，头文件中包含了http请求的各种信息。

params —— params是发送请求的查询参数对象，对象中的数据会被拼接成url?param1=value1&param2=value2。

paramsSerializer —— params参数序列化器。

data —— data是在发送POST、PUT或者PATCH请求的数据对象。

timeout —— 请求超时设置，单位为毫秒

withCredentials —— 表明是否有跨域请求需要用到证书

adapter —— adapter允许用户处理更易于测试的请求。返回一个Promise和一个有效的response

auth —— auth表明提供凭证用于完成http的身份验证。这将会在headers中设置一个Authorization授权信息。自定义Authorization授权要设置在headers中。

responseType —— 表示服务器将返回响应的数据类型，有arraybuffer、blob、document、json、text、stream这6个类型，默认是json类似数据。

xsrfCookieName —— 用作 xsrf token 值的 cookie 名称

xsrfHeaderName —— 带有 xsrf token 值 http head 名称

onUploadProgress —— 允许在上传过程中的做一些操作

onDownloadProgress —— 允许在下载过程中的做一些操作

maxContentLength —— 定义了接收到的response响应数据的最大长度。

validateStatus —— validateStatus定义了根据HTTP响应状态码决定是否接收或拒绝获取到的promise。如果 validateStatus 返回 true (或设置为 null 或 undefined ),promise将被接收;否则,promise将被拒绝。

maxRedirects —— maxRedirects定义了在node.js中redirect的最大值，如果设置为0，则没有redirect。

httpAgent —— 定义在使用http请求时的代理

httpsAgent —— 定义在使用https请求时的代理

proxy —— proxy定义代理服务器的主机名和端口，auth

cancelToken —— cancelToken定义一个 cancel token 用于取消请求
```
var vm = this
 this.$http.post(
 vm.testUrl,
 {"name":"痞子达"},
 {headers:{'Content-Type':'application/x-www-form-urlencoded'}})
 .then((response)=>{
         alert(response.data.msg);
})

```
加入{headers:{'Content-Type':'application/x-www-form-urlencoded'}}

但这个时候还是没能够正确弹框，因为我们传过去的是key字符串,并且vlue是没有值的。

后端打印出来是这样的
```
array(1) { ["{"name":"痞子达"}"]=> string(0) "" }
```

这必须获取不到啊，那我们尝试将其转换为query参数。
引入Qs，这个库是axios里面包含的，不需要再下载了。
```
import Qs from 'qs'
 var vm = this
var data = Qs.stringify({"name":"痞子达"});
 this.$http.post(vm.testUrl,data, {headers:{'Content-Type':'application/x-www-form-urlencoded'}}).then((response)=>{
     alert(response.data.msg);
})

```

![image](https://note.youdao.com/yws/public/resource/bb50e794c49c05e7417f6cdb0b9dfd1f/xmlnote/009906EFFD874A88914CCE7B2F09A7FC/7706)
这才是我们熟悉的样子。

但是我们不能每次请求都写一遍config，太麻烦了。
在入口文件main.js修改：
```
#创建一个axios实例
var axios_instance = axios.create({
#config里面有这个transformRquest，这个选项会在发送参数前进行处理。
#这时候我们通过Qs.stringify转换为表单查询参数
    transformRequest: [function (data) {
        data = Qs.stringify(data);
        return data;
    }],
#设置Content-Type
    headers:{'Content-Type':'application/x-www-form-urlencoded'}
})
Vue.use(VueAxios, axios_instance)
```
ok，以后发起http请求，如下即可：
```
var vm = this
this.$http.post(vm.url,data).then((response)=>{
    alert(response.data.msg);
})
```



补充：
1.get请求只需拼接在接口后边即可
```
 _that.$http.get('/syncResourceService/getSyncConfig?serviceIndexCode='+this.serviceIndexCode).then(function(res) {
      if (res && res.data && res.data.code == 0) {
           let data = res.data.data;
           _that.supportList = data.supportList;
           _that.typeCodes = data.configList;
       } else {
           _that.$message.error(res.data.msg);
       }
   }, function() {});
   
   
   axios.get('/user', {  //params参数必写 , 如果没有参数传{}也可以
    params: {  
       id: 12345，
       name: user
    }
})
.then(function (res) {
    console.log(res);
})
.catch(function (err) {
    console.log(err);
});

```
2.post请求及需要对象方式传参
```
params = {
   typeCodes:_that.typeCodes,
    serviceIndexCode:_that.serviceIndexCode
};
_that.$http.post('/syncResourceService/updateSyncConfig',params).then(function(res) {
    if (res && res.data && res.data.code == 0) {
        _that.$message({
            message: '自动同步设置修改成功',
            type: 'success'
        });
        _that.dialogVisible = false;
    } else {
        _that.$message.error(res.data.msg);
    }
}, function() {});
```

参考网址：
[1](https://www.jianshu.com/p/b22d03dfe006)
[2](https://blog.csdn.net/wopelo/article/details/78783442)
[axios四种数据格式以及各种方法](https://blog.csdn.net/qq_31837621/article/details/80688854)
[为什么放弃用resouce](https://link.jianshu.com/?t=https://github.com/vuefe/vuefe.github.io/issues/186)





