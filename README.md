#vue-router-node
## Table of Content
- [实例](#实例)
- [动态路由匹配](#动态路由匹配)
- [嵌套路由](#嵌套路由)
- [编程式的导航](#编程式的导航)
- [命名路由](#命名路由)
- [命名视图](#命名视图)

## 实例
``` js

// file: router/login
const login = require('../views/login');
export default {
    path:'/login',
    name:'login',
    components:login
}


// file: router/index
const Vue = require('vue');
const Router = require('vue-router');
const index = require('../views/index');
// login router
const login = require('./login');

// load Router
Vue.use('Router');

export default new Router({
  routes: [
    {
      path: '/',
      name: 'index',
      component: index
    },
    login
    
    
  ]
})


// file: App.js
const router = require('../router');
const app = new Vue({
    el:'#app',
    router,
})
```

## 动态路由匹配  

:id为动态路由参数  
在模版字符串中使用 {{ $route.params.id }} 获取id  
在非模版字符串中使用 this.$route.params.id 获取id  

<table>
    <tr>
        <th>模式</th>
        <th>匹配路径</th>
        <th>$route.params</th>
    </tr>
    <tr>
        <td>/user/:username</td>
        <td>/user/evan </td>
        <td>{ username: 'evan' }</td>
    </tr>
    <tr>
        <td>/user/:username/post/:post_id</td>
        <td>/user/evan/post/123</td>
        <td>{ username: 'evan', post_id: 123 }</td>
    </tr>
</table>

```js

const User = {
  template: '<div>User {{ $route.params.id }}</div><button @click="clickHandle">click me</button>'
  methods:{
      clickHandle(){
        console.log(this.$route.params.id)
      }
  }
}

const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User }
  ]
})

```

### 相应路由器变化
```js
const User = {
  template: '...',
  watch: {
    //to : current router message
    //{name: "index", meta: {}, path: "/index", hash: "", query: {}, …}
    //from : old router message
    //{name: "login", meta: {}, path: "/", hash: "", query: {}, …}
    '$route' (to, from) {
      // 对路由变化作出响应...
    }
  }
}
```
注意：使用路由器参数从 /user/foo 导航到 user/bar 并不会触发组件的生命周期钩子。

### 匹配的优先级
谁先定义的，谁的优先级就最高。

## 嵌套路由
```js

<div id="app">
  <router-view></router-view>
</div>

const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `
}

const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        {
          // 当 /user/:id 匹配成功，
          // UserHome 会被渲染在 User 的 <router-view> 中 
          path: '', 
          component: UserHome 
        },
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})

```
注意：以 / 开头的嵌套路径会被当作根路径。 

## 编程式的导航

### ```router.push(location)```
导航至不同的URL
这个方法会向 history 栈添加一个新的记录，所以，当用户点击浏览器后退按钮时，则回到之前的 URL。

<table>
    <tr>
        <th>声明式</th>
        <th>编程式</th>
    </tr>
    <tr>
        <td>&lt;router-link :to="..."&gt;</td>
        <td>router.push(...)</td>
    </tr>
</table>

```js
// 字符串 相当于 调用 { path:'home' }
router.push('home');
// js中准确的跳转url
this.$router.push('home');

// 对象
router.push({
    path:'home'
});
// js中准确的跳转url
this.$router.push({
    path:'home'
})

// 命名的路由（在定义的路由中寻找name为user的路由 如果寻找的路由的 path:'/user/:userId'，则最重解析的URL为 /user/123）
router.push({
    name:'user',
    params:{
        userId:123
    }
})
// js中准确的跳转url
this.$router.push({
    name:'user',
    params:{
        userId:123
    }
})
```
注意：本节所提及的``this.$router``与[动态路由匹配](#动态路由匹配)中所提及的``this.$route``不同。  
```js
this.$router // VueRouter {app: Vue$3, apps: Array, options: Object, beforeHooks: [], afterHooks: [], …} 
this.$route  // {name: "user", meta: {}, path: "/user/123", hash: "", query: {}, …}
```
### ```router.replace(location)```
导航至不同的URL，但是它不会向 history 添加新记录，而是替换掉当前的 history 记录。 

<table>
    <tr>
        <th>声明式</th>
        <th>编程式</th>
    </tr>
    <tr>
        <td>&lt;router-link :to="..." replace &gt;</td>
        <td>router.replace(...)</td>
    </tr>
</table>

### ```router.go(n)```
n:整数，在 history 记录中向前或者后退多少步。  
n>0:向前  
n<0:向后

```js

// 在浏览器记录中前进一步，等同于 history.forward()
router.go(1)
// 准确写法
this.$router.go(1);

// 后退一步记录，等同于 history.back()
router.go(-1)
// 准确写法
this.$router.go(-1);

// 前进 3 步记录
router.go(3)
// 准确写法
this.$router.go(3);

// 如果 history 记录不够用，那就默默地失败呗
router.go(-100)
router.go(100)
// 准确写法
this.$router.go(-100);
this.$router.go(100);

```
## 命名路由
```js
const userRoute = {
    path:'/user/:userId',
    name:'user', //设置路由名称
    component:User
}

const router = new VueRouter({
    routes:[
        userRoute
    ]
})

// How to use ?
// 1. <router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
// 2. router.push({ name: 'user', params: { userId: 123 }})

```
官方完整例子  
```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const Home = { template: '<div>This is Home</div>' }
const Foo = { template: '<div>This is Foo</div>' }
const Bar = { template: '<div>This is Bar {{ $route.params.id }}</div>' }

const router = new VueRouter({
  mode: 'history',
  base: __dirname,
  routes: [
    { path: '/', name: 'home', component: Home },
    { path: '/foo', name: 'foo', component: Foo },
    { path: '/bar/:id', name: 'bar', component: Bar }
  ]
})

new Vue({
  router,
  template: `
    <div id="app">
      <h1>Named Routes</h1>
      <p>Current route name: {{ $route.name }}</p>
      <ul>
        <li><router-link :to="{ name: 'home' }">home</router-link></li>
        <li><router-link :to="{ name: 'foo' }">foo</router-link></li>
        <li><router-link :to="{ name: 'bar', params: { id: 123 }}">bar</router-link></li>
      </ul>
      <router-view class="view"></router-view>
    </div>
  `
}).$mount('#app')
```

## 命名视图
有时候想同时（同级）展示多个视图，而不是嵌套展示，例如创建一个布局，有 sidebar（侧导航） 和 main（主内容） 两个视图，这个时候命名视图就派上用场了。你可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果 router-view 没有设置名字，那么默认为 default。
```html
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```
一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。确保正确使用 components 配置（带上 s）：

```js
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```