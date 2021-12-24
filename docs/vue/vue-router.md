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

 