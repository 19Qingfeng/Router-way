# Router-way
前端页面三种路由实现方式。
1. 传统路由实现。
> window.location.href = 'https://baidu.com' 跳转。刷新页面。
> history.back()回退。
整个页面重新加载，浏览器历史可以显示每一个地址。考虑到安全性但是JS代码中是无法操作的。
---
2. Hash路由方式。
>  window.href.href = '#hash' localhost:9000#test。并不刷新页面。
#后跟的就是页面Hash，同样hash的改变也会推进浏览器历史记录中。
支持后退前进。
```
window.onhashchange = function () {
    console.log('current Hash:',window.location.hash)
}
```
---
3. H5 Router。
> history.pushState(state, title[, url]) 推进路由。增加历史盏中的一条。
> history.replaceState(state,title[, url]) 替换路由。在历史记录中替换当前记录。

> 可以改变网址(存在跨域限制)而不刷新页面，这个强大的特性后来用到了单页面应用如：vue-router，react-router-dom中。
>> 仅改变网址,网页不会真的跳转,也不会获取到新的内容,本质上网页还停留在原页面。
+ 状态对象：传给目标路由的信息,可为空
+ 页面标题：目前所有浏览器都不支持,填空字符串即可
+ 可选url：目标url，不会检查url是否存在，且不能跨域。如不传该项,即给当前url添加data
> popstate事件会在点击后退、前进按钮(或调用history.back()、history.forward()、history.go()方法)时触发。前提是不能真的发生了页面跳转,而是在由history.pushState()或者history.replaceState()形成的历史节点中前进后退

> 注意:用history.pushState()或者history.replaceState()不会触发popstate事件。

+ history.state
> 当前URL下对应的状态信息。如果当前URL不是通过pushState或者replaceState产生的，那么history.state是null。history.state可以保存当前页面的信息，通过pushState或者replaceState传递onpopstate中改变时候获得（history.state也可以获取）。
+ history.pushState(state, title, url)
    1. state：与要跳转到的URL对应的状态信息。
    2. title：不知道干啥用，传空字符串就行了。
    3. url：要跳转到的URL地址，不能跨域。
> 将当前URL和history.state加入到history中，并用新的state和URL替换当前。不会造成页面刷新。
+ history.replaceState
    1. state：与要跳转到的URL对应的状态信息。
    2. title：不知道干啥用，传空字符串就行了。
    3. url：要跳转到的URL地址，不能跨域。
> 用新的state和URL替换当前。不会造成页面刷新。
+ window.onpopstate
> history.go和history.back（包括用户按浏览器历史前进后退按钮）触发，并且页面无刷的时候（由于使用pushState修改了history）会触发popstate事件，事件发生时浏览器会从history中取出URL和对应的state对象替换当前的URL和history.state。通过event.state也可以获取history.state。

```
window.onpopstate = function(event) {
  console.log(event.state); // 当前页面相关的history路由信息
  console.log(window.history.state;); // 当前页面相关的history路由信息
  console.log(window.location.hash) // hash路径
  console.log(window.location.pathname) // 绝对路径
  console.log(window.location.href) // 全部路径
};
```

### 引入Vue中两种路由模式的区别。
---
#### Hash模式
> hash模式背后的原理是onhashchange事件,可以在window对象上监听这个事件:
```
window.onhashchange = function(event){
    // 打印旧的url和新的url
    console.log(event.oldURL, event.newURL);
    // 相当与跳转页面的时候通过hash区别页面以及传递参数
    let hash = location.hash.slice(1);
    document.body.style.color = hash;
}
```
上面的代码可以通过改变hash来改变页面字体颜色，虽然没什么用，但是一定程度上说明了原理。

> 更关键的一点是，因为hash发生变化的url都会被浏览器记录下来，从而你会发现浏览器的前进后退都可以用了，同时点击后退时，页面字体颜色也会发生变化。这样一来，浏览器不会发起请求，但是页面状态和url关联了起来，url改变页面可以根据url进行相应逻辑变化。这就是hash路由。
---
#### History模式
> history api，H5的history api给了前端路由充分的自由。相对于hash路由来讲前端只能控制#后的url地址，而history api允许在同源策略下进行任意的自由路由设置而不刷新页面。

> 需要额外注意：

history api可以分为两大部分，切换和修改，参考MDN，切换历史状态包括back、forward、go 三个方法，对应浏览器的前进，后退，跳转操作：
```
history.go(-2);//后退两次
history.go(2);//前进两次
history.back(); //后退
hsitory.forward(); //前进
```

###### 修改历史状态包括了pushState,replaceState <br>
> 两个方法,这两个方法接收三个参数:stateObj,title,url

```
history.pushState({color:'red'}, 'red', 'red')
history.back();
setTimeout(function(){
     history.forward();
 },0)
window.onpopstate = function(event){
     console.log(event.state)
     if(event.state && event.state.color === 'red'){
           document.body.style.color = 'red';
      }
}
```
###### history模式配置问题
> vue-router官方文档：不过这种模式要玩好，还需要后台配置支持。因为我们的应用是个单页客户端应用，如果后台没有正确的配置，当用户在浏览器直接访问 http://oursite.com/user/id 就会返回 404，这就不好看了。

+ 只配置前端的情况
首先，我们将mode设置为history，但不配置后端。然后，假如我们的路由是长这个样子的：
```
const routes = [
    {path: '/home', component: Home},
    {path: '/', redirect: '/home'}
];
```
我们用nginx部署项目，然后在地址栏输入 http://localhost:8080 （这里配置的端口是8080），你会发现地址栏之后会变为http://localhost:8080/home，并且看起来一切正常，似乎路由也可以正常切换而不会发生其他问题（实际上会发生问题，后面会进行讨论）。看起来好像不需要按官网告诉我们的那样配置后端也能实现history模式，但如果你直接在地址栏输入http://localhost:8080/home，你会发现你获得了一个404页面。

那么http://localhost:8080为什么可以（部分）正常显示呢？道理其实很简单，你访问 http://localhost:8080时 ，静态服务器（这里是nginx）会默认去目标目录（这里为location中root所指定的目录）下寻找index.html（这是nginx在端口后没有额外路径时的默认行为），目标目录下有这个文件吗？有！然后静态服务器返回给你这个文件，配合vue-router进行转发，自然可以（部分）正常显示。
但如果直接访问http://localhost:8080/home，静态服务器会去目标目录下寻找home文件，目标目录下有这个文件吗？没有！所以自然就404了。

+ 配置后端

为了达到直接访问http://localhost:8080/home也可以成功的目的，我们需要对后端（这里即nginx）进行一些配置。

首先想想，要怎样才能达到这个目的呢？

在传统的hash模式中 http://localhost:8080#home ,即使不需要配置，静态服务器始终会去寻找index.html并返回给我们，然后vue-router会获取#后面的字符作为参数，对前端页面进行变换。

类比一下，在history模式中，我们所想要的情况就是：输入http://localhost:8080/home，但最终返回的也是index.html，然后vue-router会获取home作为参数，对前端页面进行变换。那么在nginx中，谁能做到这件事呢？答案就是try_files。

关于nginx配置以及history模式可能遇到的问题可以参考这篇文章[Vue Router history模式的配置方法及其原理](https://segmentfault.com/a/1190000019391139)



#### Vue对比两种模式
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;已经有 hash 模式了，而且 hash 能兼容到IE8， history 只能兼容到 IE10，为什么还要搞个 history 呢？
+ 首先，hash 本来是拿来做页面定位的，如果拿来做路由的话，原来的锚点功能就不能用了。其次，hash 的传参是基于 url 的，如果要传递复杂的数据，会有体积的限制，而 history 模式不仅可以在url里放参数，还可以将数据存放在一个特定的对象中。
最重要的一点：

+ 如果不想要很丑的 hash，我们可以用路由的 history 模式 —— 引用自 vueRouter文档
