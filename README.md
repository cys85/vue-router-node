#vue-router-node
## Table of Content
- [实例](#实例)
- [动态路由匹配](#动态路由匹配)

## 实例
``` javascript

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

```javascript

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
```javascript
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
```javascript

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