### Vue-router引入及使用

1. 在index.js中通过import的方式引入
2. 使用Vue的use方法使用，说明vue-router是个插件
3. 最后将Router进行导出使用
4. router-view、router-link相当于是vue-router插件中的两个组件

> router/index.js

```js
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
// 使用use 说明是个插件
// 1.实现并声明了两个组件router-view & router-link
// 2.使用install this.$router.push()
Vue.use(Router)

export default new Router({
  // 路由配置表
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    }
  ]
})
```

> main.js

```js
import router from './router'
new Vue({
  el: '#app',
  router, // 添加到配置项中
  components: { App },
  template: '<App/>'
})
```

> App.vue

```html
<div id="nav">
  <router-link></router-link>
  <router-link></router-link>
</div>
<!-- 路由出口 -->
<!-- 利用vue响应式：当前地址发生变化 重新渲染 -->
<router-view/>
```

### Vue-router插件底层实现

先定义一个VueRouter类

```js
class VueRouter {
}
```

在Vue插件中需要有`install`方法，用于上文中`Vue.use()`调用，再将VueRouter导出

```js
VueRouter.install = function(_Vue) {
}
export default VueRouter
```

为了使用Vue但并不想打包的时候把Vue也打包进插件中，所以不要采用`import`的方式，而是定义一个变量

```js
let Vue;
```

在上文`index.js`中，创建Router实例是在`Vue.use()`之后的，所以我们要保证在代码中`$router`是可用的，就要在插件中使用全局混入的方法进行延迟，在每个组件的`beforecreated`生命周期时都会调用检查

```js
Vue.mixin({
  beforeCreate() {
    // 在每个组件创建实例时都会调用
    // 根实例才有该选项
    if (this.$options.router) {
      Vue.prototype.$router = this.$options.router;
    }
  }
})
```

实现两个组件

- 使用render函数的形式渲染dom，不采用jsx的原因是兼容性较差，如果浏览器不支持就不行
- 组件间的内容可以使用插槽的形式就行获取

```js
Vue.component('router-link', {
    props: {
      to: {
        type: String, // 正常可以是对象和字符串
        required: true
      }
    },
    render(h) {
      // <a href="to">aaa</a>
      // 也可以直接使用jsx方式去写，但是兼容性很差
      return h('a', {attrs: {href: '#' + this.to}}, this.$slots.default) // 获取默认插槽
    }
  })
  Vue.component('router-view', {
    render(h) {
      let component = null;
      // 获取当前路由所对应的组件
      const route = this.$router.$options.routes.find(route => route.path === this.$router.current)
      if (route) {
        component = route.component;
      }
      return h(component)
    }
  })
```

怎么获取当前页面的路由呢？

- 为了使一个数据成为响应式的，可以使用`Vue.util.defineReactive(this, name, init)`
- 监听页面路由变化

```js
class VueRouter {
  constructor(options) {
    this.$options = options;
    // 把current作为响应式数据
    const initial = window.location.hash.slice('#') || "/";
    Vue.util.defineReactive(this, 'current', initial) // Vue自带util方法中的
    this.current = '/';
    // 监听hash变化
    window.addEventListener('hashChange', ()=> {
      // 将当前路由
      this.current = window.location.hash.slice(1)
    })
  }
}
```

完整代码

```js
// 1.实现一个插件
// 2.实现两个组件

// vue插件：function / object 要求必须有一个install方法，会被use方法调用
let Vue; // 保存Vue的构造函数，在下面插件中使用，打包时候不要把Vue打包进去
class VueRouter {
  constructor(options) {
    this.$options = options;
    // 把current作为响应式数据
    const initial = window.location.hash.slice('#') || "/";
    Vue.util.defineReactive(this, 'current', initial) // Vue自带util方法中的
    this.current = '/';
    // 监听hash变化
    window.addEventListener('hashChange', ()=> {
      // 将当前路由
      this.current = window.location.hash.slice(1)
    })
  }
}

VueRouter.install = function(_Vue) {
  Vue = _Vue;
  // 1. 挂载$router属性
  // 使用全局混入: 延迟下面逻辑到router创建完毕并且附加到选项上时才执行
  Vue.mixin({
    beforeCreate() {
      // 在每个组件创建实例时都会调用
      // 根实例才有该选项
      if (this.$options.router) {
        Vue.prototype.$router = this.$options.router;
      }
    }
  })

  // 2.注册并实现两个组件
  Vue.component('router-link', {
    props: {
      to: {
        type: String, // 正常可以是对象和字符串
        required: true
      }
    },
    render(h) {
      // <a href="to">aaa</a>
      // 也可以直接使用jsx方式去写，但是兼容性很差
      return h('a', {attrs: {href: '#' + this.to}}, this.$slots.default) // 获取默认插槽
    }
  })
  Vue.component('router-view', {
    render(h) {
      let component = null;
      // 获取当前路由所对应的组件
      const route = this.$router.$options.routes.find(route => route.path === this.$router.current)
      if (route) {
        component = route.component;
      }
      return h(component)
    }
  })
}
// 导出
export default VueRouter

```

### 其他知识点

#### 路由守卫

有三种类型，分为全局的、单个路由独享的、组件级的。

##### 全局前置守卫

> router.beforeEach

```js
router.beforeEach((to, from, next) => {
  // ...
})
```

当一个导航触发时，全局前置守卫按照创建顺序调用。守卫是异步解析执行，此时导航在所有守卫 resolve 完之前一直处于 **等待中**。

- to: 即将要进入的目标路由对象
- from: 当前导航正要离开的路由
- next:  一定要调用该方法来 **resolve** 这个钩子。
  - **`next()`**: 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 **confirmed**
  - **`next(false)`**: 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 `from` 路由对应的地址。
  - **`next('/')` 或者 `next({ path: '/' })`**: 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。
  - **`next(error)`**: (2.4.0+) 如果传入 `next` 的参数是一个 `Error` 实例。

**tip：确保 `next` 函数在任何给定的导航守卫中都被严格调用一次。它可以出现多于一次，但是只能在所有的逻辑路径都不重叠的情况下，否则钩子永远都不会被解析或报错**。

通常用在检查是否登录：

```js
router.beforeEach((to, from, next) => {
  if (to.name !== 'Login' && !isAuthenticated) next({ name: 'Login' })
  // 如果用户未能验证身份，则 `next` 会被调用两次
  next()
})
```

##### 全局解析守卫

`router.beforeResolve`这和 `router.beforeEach` 类似，区别是在导航被确认之前，**同时在所有组件内守卫和异步路由组件被解析之后**，解析守卫就被调用。

##### 全局后置钩子

不会接受`next`函数也不会改变导航本身

```js
router.afterEach((to, from) => {
  // ...
})
```

##### 路由独享的守卫

在路由配置上直接定义`beforeEnter`，和全局前置守卫参数一样，可以在单独路由上作用。

```js
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})
```

##### 组件内的守卫

- `beforeRouteEnter`
- `beforeRouteUpdate` 
- `beforeRouteLeave`

```js
const Foo = {
  template: `...`,
  beforeRouteEnter(to, from, next) {
  },
  beforeRouteUpdate(to, from, next) {
  },
  beforeRouteLeave(to, from, next) {
  }
}
```

`beforeRouteEnter` 守卫 **不能** 访问 `this`，因为守卫是在导航确认前调用，那时候组件还没有被创建。但是在`next`中可以访问组件实例。并且只有它能够执行回调，其它两个调用的时候`this`已经有了。

其它两个守卫可以检测到用户对页面进行了什么变化和离开时如果没有保存禁止离开`next(false)`等操作。

#### 完整的导航解析流程

1. 导航被触发。
2. 在失活的组件里调用 `beforeRouteLeave` 守卫。
3. 调用全局的 `beforeEach` 守卫。
4. 在重用的组件里调用 `beforeRouteUpdate` 守卫。
5. 在路由配置里调用 `beforeEnter`。
6. 解析异步路由组件。
7. 在被激活的组件里调用 `beforeRouteEnter`。
8. 调用全局的 `beforeResolve` 守卫。
9. 导航被确认。
10. 调用全局的 `afterEach` 钩子。
11. 触发 DOM 更新。
12. 调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数，创建好的组件实例会作为回调函数的参数传入。

#### 路由跳转的方式

##### 声明式

`<router-link :to="...">`

##### 编程式

`router.push(...)`

```js
// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: '123' }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})
```

**注意：如果提供了 `path`，`params` 会被忽略**

#### `router.replace(location, onComplete?, onAbort?)`

跟 `router.push` 很像，唯一的不同就是，它不会向 history 添加新记录，而是跟它的方法名一样 —— 替换掉当前的 history 记录。

#### `router.go(n)`

这个方法的参数是一个整数，意思是在 history 记录中向前或者后退多少步

**Vue Router 的导航方法 (`push`、 `replace`、 `go`) 在各类路由模式 (`history`、 `hash` 和 `abstract`) 下表现一致。**

#### 重定向和别名

##### 重定向

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

##### 别名

“重定向”的意思是，当用户访问 `/a`时，URL 将会被替换成 `/b`，然后匹配路由为 `/b`，那么“别名”又是什么呢？

**`/a` 的别名是 `/b`，意味着，当用户访问 `/b` 时，URL 会保持为 `/b`，但是路由匹配则为 `/a`，就像用户访问 `/a` 一样。**

#### 路由懒加载

如果没有应用懒加载，运用webpack打包后的文件将会异常的大，造成进入首页时，需要加载的内容过多，时间过长，会出啊先长时间的白屏

##### vue异步组件

这种情况下一个组件生成一个js文件

```js
{ path: '/home', name: 'home', component: resolve => require(['@/components/home'],resolve) }
```

##### 使用import

```js
// 下面2行代码，没有指定webpackChunkName，每个组件打包成一个js文件。

/* const Home = () => import('@/components/home')

const Index = () => import('@/components/index')

const About = () => import('@/components/about') */

// 下面2行代码，指定了相同的webpackChunkName，会合并打包成一个js文件。

// 把组件按组分块

const Home = () => import(/* webpackChunkName: 'ImportFuncDemo' */ '@/components/home')

const Index = () => import(/* webpackChunkName: 'ImportFuncDemo' */ '@/components/index')

const About = () => import(/* webpackChunkName: 'ImportFuncDemo' */ '@/components/about')

 
{ path: '/about', component: About }, { path: '/index', component: Index }, { path: '/home', component: Home }
```

##### webpack提供的require.ensure()

这种情况下，多个路由指定相同的chunkName，会合并打包成一个js文件

```js
{ path: '/home', name: 'home', component: r => require.ensure([], () => r(require('@/components/home')), 'demo') },

{ path: '/index', name: 'Index', component: r => require.ensure([], () => r(require('@/components/index')), 'demo') },

{ path: '/about', name: 'about', component: r => require.ensure([], () => r(require('@/components/about')), 'demo-01') }
```

