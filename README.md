# vue-router-node
## Table of Content
- [实例](#实例)
- [动态路由匹配](#动态路由匹配)
- [嵌套路由](#嵌套路由)
- [编程式的导航](#编程式的导航)
- [命名路由](#命名路由)
- [命名视图](#命名视图)
- [重定向](#重定向)
- [别名](#别名)
- [导航钩子](#导航钩子)
  - [全局钩子](#全局钩子)
  - [某个路由独享的钩子](#某个路由独享的钩子)
  - [组件内的钩子](#组件内的钩子)
- [路由元信息](#路由元信息)
- [过渡动效](#过渡动效)
- [数据获取](#数据获取)
- [滚动行为](#滚动行为)
- [路由懒加载](#路由懒加载)
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
    '$route' (to, from) {
      //to : current router message  {name: "index", meta: {}, path: "/index", hash: "", query: {}, …}
      //from : old router message  {name: "login", meta: {}, path: "/", hash: "", query: {}, …}

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

## 重定向
当用户访问 /a时，URL 将会被替换成 /b，然后匹配路由为 /b。

配置：
```js

const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' }
  ]
})

const router = new VueRouter({
  routes: [
    { path: '/a', redirect: { name: 'foo' }}
  ]
})


const router = new VueRouter({
  routes: [
    { path: '/a', redirect: to => {
      // 方法接收 目标路由 作为参数
      // return 重定向的 字符串路径/路径对象
    }}
  ]
})


```

官方完整例子

```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const Home = { template: '<router-view></router-view>' }
const Default = { template: '<div>default</div>' }
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }
const Baz = { template: '<div>baz</div>' }
const WithParams = { template: '<div>{{ $route.params.id }}</div>' }

const router = new VueRouter({
  mode: 'history',
  base: __dirname,
  routes: [
    { path: '/', component: Home,
      children: [
        { path: '', component: Default },
        { path: 'foo', component: Foo },
        { path: 'bar', component: Bar },
        { path: 'baz', name: 'baz', component: Baz },
        { path: 'with-params/:id', component: WithParams },
        // relative redirect to a sibling route
        { path: 'relative-redirect', redirect: 'foo' }
      ]
    },
    // absolute redirect
    { path: '/absolute-redirect', redirect: '/bar' },
    // dynamic redirect, note that the target route `to` is available for the redirect function
    { path: '/dynamic-redirect/:id?',
      redirect: to => {
        const { hash, params, query } = to
        if (query.to === 'foo') {
          return { path: '/foo', query: null }
        }
        if (hash === '#baz') {
          return { name: 'baz', hash: '' }
        }
        if (params.id) {
          return '/with-params/:id'
        } else {
          return '/bar'
        }
      }
    },
    // named redirect
    { path: '/named-redirect', redirect: { name: 'baz' }},

    // redirect with params
    { path: '/redirect-with-params/:id', redirect: '/with-params/:id' },

    // catch all redirect
    { path: '*', redirect: '/' }
  ]
})

new Vue({
  router,
  template: `
    <div id="app">
      <h1>Redirect</h1>
      <ul>
        <li><router-link to="/relative-redirect">
          /relative-redirect (redirects to /foo)
        </router-link></li>
        <li><router-link to="/relative-redirect?foo=bar">
          /relative-redirect?foo=bar (redirects to /foo?foo=bar)
        </router-link></li>
        <li><router-link to="/absolute-redirect">
          /absolute-redirect (redirects to /bar)
        </router-link></li>
        <li><router-link to="/dynamic-redirect">
          /dynamic-redirect (redirects to /bar)
        </router-link></li>
        <li><router-link to="/dynamic-redirect/123">
          /dynamic-redirect/123 (redirects to /with-params/123)
        </router-link></li>
        <li><router-link to="/dynamic-redirect?to=foo">
          /dynamic-redirect?to=foo (redirects to /foo)
        </router-link></li>
        <li><router-link to="/dynamic-redirect#baz">
          /dynamic-redirect#baz (redirects to /baz)
        </router-link></li>
        <li><router-link to="/named-redirect">
          /named-redirect (redirects to /baz)
        </router-link></li>
        <li><router-link to="/redirect-with-params/123">
          /redirect-with-params/123 (redirects to /with-params/123)
        </router-link></li>
        <li><router-link to="/not-found">
          /not-found (redirects to /)
        </router-link></li>
      </ul>
      <router-view class="view"></router-view>
    </div>
  `
}).$mount('#app')
```

## 别名
/a 的别名是 /b，意味着，当用户访问 /b 时，URL 会保持为 /b，但是路由匹配则为 /a，就像用户访问 /a 一样
```js

const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' },
    // absolute alias
    { path: 'foo', component: Foo, alias: '/foo' },
    // relative alias (alias to /home/bar-alias)
    { path: 'bar', component: Bar, alias: 'bar-alias' },
    // multiple aliases
    { path: 'baz', component: Baz, alias: ['/baz', 'baz-alias'] }
  ]
})

```

官方完整例子：

```js

import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const Home = { template: '<div><h1>Home</h1><router-view></router-view></div>' }
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }
const Baz = { template: '<div>baz</div>' }

const router = new VueRouter({
  mode: 'history',
  base: __dirname,
  routes: [
    { path: '/home', component: Home,
      children: [
        // absolute alias
        { path: 'foo', component: Foo, alias: '/foo' },
        // relative alias (alias to /home/bar-alias)
        { path: 'bar', component: Bar, alias: 'bar-alias' },
        // multiple aliases
        { path: 'baz', component: Baz, alias: ['/baz', 'baz-alias'] }
      ]
    }
  ]
})

new Vue({
  router,
  template: `
    <div id="app">
      <h1>Route Alias</h1>
      <ul>
        <li><router-link to="/foo">
          /foo (renders /home/foo)
        </router-link></li>
        <li><router-link to="/home/bar-alias">
          /home/bar-alias (renders /home/bar)
        </router-link></li>
        <li><router-link to="/baz">
          /baz (renders /home/baz)</router-link>
        </li>
        <li><router-link to="/home/baz-alias">
          /home/baz-alias (renders /home/baz)
        </router-link></li>
      </ul>
      <router-view class="view"></router-view>
    </div>
  `
}).$mount('#app')

```

## 导航钩子

### 全局钩子
```js
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  //to:即将要进入的目标 路由对象
  //from:当前导航正要离开的路由
  //next:Function 一定要调用该方法来 resolve 这个钩子。执行效果依赖 next 方法的调用参数。
      //next(): 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 confirmed （确认的）。
      //next(false): 中断当前的导航。如果浏览器的 URL 改变了（可能是用户手动或者浏览器后退按钮），那么 URL 地址会重置到 from 路由对应的地址。
      //next('/') 或者 next({ path: '/' }): 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。
  // ...
})
```


### 某个路由独享的钩子
```js
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
         //to:即将要进入的目标 路由对象
         //from:当前导航正要离开的路由
         //next:Function 一定要调用该方法来 resolve 这个钩子。执行效果依赖 next 方法的调用参数。
              //next(): 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 confirmed （确认的）。
              //next(false): 中断当前的导航。如果浏览器的 URL 改变了（可能是用户手动或者浏览器后退按钮），那么 URL 地址会重置到 from 路由对应的地址。
              //next('/') 或者 next({ path: '/' }): 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。
         // ...
      }
    }
  ]
})
```

### 组件内的钩子

```js
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当钩子执行前，组件实例还没被创建
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
  }
}
```
## 路由元信息
```js
// 配置 meta 字段
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      children: [
        {
          path: 'bar',
          component: Bar,
          // a meta field
          meta: { requiresAuth: true }
        }
      ]
    }
  ]
})
```

```js
//全局导航钩子中检查 meta 字段
router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requiresAuth)) {
    // this route requires auth, check if logged in
    // if not, redirect to login page.
    if (!auth.loggedIn()) {
      next({
        path: '/login',
        query: { redirect: to.fullPath }
      })
    } else {
      next()
    }
  } else {
    next() // 确保一定要调用 next()
  }
})
```

## 过渡动效
官方完整例子：
```js
mport Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const Home = {
  template: `
    <div class="home">
      <h2>Home</h2>
      <p>hello</p>
    </div>
  `
}

const Parent = {
  data () {
    return {
      transitionName: 'slide-left'
    }
  },
  // dynamically set transition based on route change
  watch: {
    '$route' (to, from) {
      const toDepth = to.path.split('/').length
      const fromDepth = from.path.split('/').length
      this.transitionName = toDepth < fromDepth ? 'slide-right' : 'slide-left'
    }
  },
  template: `
    <div class="parent">
      <h2>Parent</h2>
      <transition :name="transitionName">
        <router-view class="child-view"></router-view>
      </transition>
    </div>
  `
}

const Default = { template: '<div class="default">default</div>' }
const Foo = { template: '<div class="foo">foo</div>' }
const Bar = { template: '<div class="bar">bar</div>' }

const router = new VueRouter({
  mode: 'history',
  base: __dirname,
  routes: [
    { path: '/', component: Home },
    { path: '/parent', component: Parent,
      children: [
        { path: '', component: Default },
        { path: 'foo', component: Foo },
        { path: 'bar', component: Bar }
      ]
    }
  ]
})

new Vue({
  router,
  template: `
    <div id="app">
      <h1>Transitions</h1>
      <ul>
        <li><router-link to="/">/</router-link></li>
        <li><router-link to="/parent">/parent</router-link></li>
        <li><router-link to="/parent/foo">/parent/foo</router-link></li>
        <li><router-link to="/parent/bar">/parent/bar</router-link></li>
      </ul>
      <transition name="fade" mode="out-in">
        <router-view class="view"></router-view>
      </transition>
    </div>
  `
}).$mount('#app')
```


## 数据获取
### 导航完成后获取数据
```js
<template>
  <div class="post">
    <div class="loading" v-if="loading">
      Loading...
    </div>

    <div v-if="error" class="error">

    </div>

    <div v-if="post" class="content">
      <h2></h2>
      <p></p>
    </div>
  </div>
</template>
export default {
  data () {
    return {
      loading: false,
      post: null,
      error: null
    }
  },
  created () {
    // 组件创建完后获取数据，
    // 此时 data 已经被 observed 了
    this.fetchData()
  },
  watch: {
    // 如果路由有变化，会再次执行该方法
    '$route': 'fetchData'
  },
  methods: {
    fetchData () {
      this.error = this.post = null
      this.loading = true
      // replace getPost with your data fetching util / API wrapper
      getPost(this.$route.params.id, (err, post) => {
        this.loading = false
        if (err) {
          this.error = err.toString()
        } else {
          this.post = post
        }
      })
    }
  }
}
```

### 在导航完成前获取数据
```js
export default {
  data () {
    return {
      post: null,
      error: null
    }
  },
  beforeRouteEnter (to, from, next) {
    getPost(to.params.id, (err, post) => 
      if (err) {
        // display some global error message
        next(false)
      } else {
        next(vm => {
          vm.post = post
        })
      }
    })
  },
  // 路由改变前，组件就已经渲染完了
  // 逻辑稍稍不同
  watch: {
    $route () {
      this.post = null
      getPost(this.$route.params.id, (err, post) => {
        if (err) {
          this.error = err.toString()
        } else {
          this.post = post
        }
      })
    }
  }
}
```

## 滚动行为
注意: 这个功能只在 HTML5 history 模式下可用
当创建一个 Router 实例，你可以提供一个 scrollBehavior 方法：   
``` js
const router = new VueRouter({
  routes: [...],
  scrollBehavior (to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition // 模拟浏览器 后退 或 前进 后页面浏览器所save的位置
    } else {
      return { x: 0, y: 0 } // return 期望滚动到哪个的位置   { x: number, y: number }  or { selector: string }
    }
  }
})
```
配合[路由元信息](#路由元信息)官网完整例子：
```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const Home = { template: '<div>home</div>' }
const Foo = { template: '<div>foo</div>' }
const Bar = {
  template: `
    <div>
      bar
      <div style="height:500px"></div>
      <p id="anchor">Anchor</p>
    </div>
  `
}

// scrollBehavior:
// - only available in html5 history mode
// - defaults to no scroll behavior
// - return false to prevent scroll
const scrollBehavior = (to, from, savedPosition) => {
  if (savedPosition) {
    // savedPosition is only available for popstate navigations.
    return savedPosition
  } else {
    const position = {}
    // new navigation.
    // scroll to anchor by returning the selector
    if (to.hash) {
      position.selector = to.hash
    }
    // check if any matched route config has meta that requires scrolling to top
    if (to.matched.some(m => m.meta.scrollToTop)) {
      // cords will be used if no selector is provided,
      // or if the selector didn't match any element.
      position.x = 0
      position.y = 0
    }
    // if the returned position is falsy or an empty object,
    // will retain current scroll position.
    return position
  }
}

const router = new VueRouter({
  mode: 'history',
  base: __dirname,
  scrollBehavior,
  routes: [
    { path: '/', component: Home, meta: { scrollToTop: true }},
    { path: '/foo', component: Foo },
    { path: '/bar', component: Bar, meta: { scrollToTop: true }}
  ]
})

new Vue({
  router,
  template: `
    <div id="app">
      <h1>Scroll Behavior</h1>
      <ul>
        <li><router-link to="/">/</router-link></li>
        <li><router-link to="/foo">/foo</router-link></li>
        <li><router-link to="/bar">/bar</router-link></li>
        <li><router-link to="/bar#anchor">/bar#anchor</router-link></li>
      </ul>
      <router-view class="view"></router-view>
    </div>
  `
}).$mount('#app')
```

## 路由懒加载
当打包构建应用时，Javascript 包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。  

结合 Vue 的 异步组件 和 Webpack 的 code splitting feature, 轻松实现路由组件的懒加载。  

我们要做的就是把路由对应的组件定义成异步组件：  
```js
const Foo = resolve => {
  // require.ensure 是 Webpack 的特殊语法，用来设置 code-split point
  // （代码分块）
  require.ensure(['./Foo.vue'], () => {
    resolve(require('./Foo.vue'))
  })
}
```
这里还有另一种代码分块的语法，使用 AMD 风格的 require，于是就更简单了： 
```js
const Foo = resolve => require(['./Foo.vue'], resolve)
```

不需要改变任何路由配置，跟之前一样使用 Foo：

```js
const router = new VueRouter({
  routes: [
    { path: '/foo', component: Foo }
  ]
})
```

### 把组件按组分块

有时候我们想把某个路由下的所有组件都打包在同个异步 chunk 中。只需要 给 chunk 命名，提供 require.ensure 第三个参数作为 chunk 的名称:
```js
const Foo = r => require.ensure([], () => r(require('./Foo.vue')), 'group-foo')
const Bar = r => require.ensure([], () => r(require('./Bar.vue')), 'group-foo')
const Baz = r => require.ensure([], () => r(require('./Baz.vue')), 'group-foo')
```

Webpack 将相同 chunk 下的所有异步模块打包到一个异步块里面 —— 这也意味着我们无须明确列出 require.ensure 的依赖（传空数组就行）。