### Vue Router



#### 基础



##### 入门

使用一个自定义组件 `router-link` 来创建链接。这使得 Vue Router 可以在不重新加载页面的情况下更改 URL，处理 URL 的生成以及编码。

`router-view` 将显示与 url 对应的组件。

通过调用 `app.use(router)`，我们可以在任意组件中以 `this.$router` 的形式访问它，并且以 `this.$route` 的形式访问当前路由



##### 动态路由匹配

**路径参数**

```javascript
// /user/a   /user/b 会被映射到同一个路由
const routes = [
  // 动态段以冒号开始
  { path: '/users/:id', component: User },
]
```

当一个路由被匹配时，它的 *params* 的值将在每个组件中以 `this.$route.params` 的形式暴露出来

相同的组件实例将被重复使用，组件的生命周期钩子不会被调用。



要对同一个组件中参数的变化做出响应的话，你可以简单地 watch `$route` 对象上的任意属性

```javascript
const User = {
  template: '...',
  created() {
    // 监控路由参数
    this.$watch(
      () => this.$route.params,
      (toParams, previousParams) => {
        // 对路由变化做出响应...
      }
    )
  },
  // 使用导航守卫
  async beforeRouteUpdate(to, from) {
    // 对路由变化做出响应...
    this.userData = await fetchUser(to.params.id)
  },
}
```





##### 路由的匹配与语法



**嵌套路由**

以 / 开头的嵌套路径将被视为根路径。这允许你利用组件嵌套，而不必使用嵌套的 URL。

可以使用空的子路由，在没有匹配到是渲染一些东西

```javascript
const routes = [
  {
    path: '/user/:id',
    component: User,
    children: [
      {
        // 当 /user/:id/profile 匹配成功
        // UserProfile 将被渲染到 User 的 <router-view> 内部
        path: 'profile',
        component: UserProfile,
      },
      {
        // 当 /user/:id/posts 匹配成功
        // UserPosts 将被渲染到 User 的 <router-view> 内部
        path: 'posts',
        component: UserPosts,
      },
      // 当 /user/:id 匹配成功
      // UserHome 将被渲染到 User 的 <router-view> 内部
      { path: '', component: UserHome },
    ],
  },
]
```



##### 编程式导航

在 Vue 实例中，你可以通过 $router 访问路由实例。因此你可以调用 this.$router.push，这个方法会向 history 栈添加一个新的记录，所以，当用户点击浏览器后退按钮时，会回到之前的 URL。

name + params 或 path + query （~~path + params~~）



```javascript
// 命名的路由，并加上参数，让路由建立 url
router.push({ name: 'user', params: { username: 'eduardo' } })

// 带查询参数，结果是 /register?plan=private
router.push({ path: '/register', query: { plan: 'private' } })
```

`router.replace` 替换当前位置，在导航时不会向 history 添加新记录

也可以直接在传递给 router.push 的 routeLocation 中增加一个属性 replace: true 

```javascript
<router-link :to="..." replace>
router.push({ path: '/home', replace: true })
// 相当于
router.replace({ path: '/home' })
```

横跨历史 

```javascript
// 向前移动一条记录，与 router.forward() 相同
router.go(1)

// 返回一条记录，与router.back() 相同
router.go(-1)
```

篡改历史

// TODO



##### 命名路由

链接到一个命名的路由，可以向 `router-link` 组件的 `to` 属性传递一个对象

```javascript
const routes = [
  {
    path: '/user/:username',
    name: 'user',
    component: User
  }
]
router.push({ name: 'user', params: { username: 'erina' } })
```



##### 命名视图

**多个路由出口**，如果 `router-view` 没有设置名字，那么默认为 `default`

一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。确保正确使用 `components` 配置

```html
<router-view name="RightSidebar"></router-view>

const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    {
      path: '/',
      components: {
        default: Home,
        // LeftSidebar: LeftSidebar 的缩写
        // 与 `<router-view>` 上的 `name` 属性匹配
        LeftSidebar,
      },
    },
  ],
})
```



##### 重定向 和 别名

```javascript
// 重定向的目标是命名路由
const routes = [{ path: '/home', redirect: { name: 'homepage' } }]

// 也可以是方法动态返回重定向目标，完成相对重定向
const routes = [
  {
    path: '/users/:id/posts',
    redirect: to => {
      // 方法接收目标路由作为参数
      // return 重定向的字符串路径/路径对象
    },
  },
]
```

导航守卫并没有应用在跳转路由上，而仅仅应用在其目标上



**别名**

```javascript
//  访问 /home /homePage 都会被重定向到  /
const routes = [{ path: '/', component: Homepage, alias: ['/home', '/homePage'] }]
```



##### 路由组件传参

当 props 设置为 true 时，route.params 将被设置为组件的 props。

```javascript
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const routes = [{ path: '/user/:id', component: User, props: true }]
```

当 `props` 是一个对象时，它将原样设置为组件 props



##### 不同的历史记录模式





#### 进阶



##### 导航守卫

导航守卫主要用来通过跳转或取消的方式守卫导

- 全局的
- 单个路由独享的
- 组件级别的



**全局前置守卫**

```javascript
router.beforeEach(async (to, from) => {
  // canUserAccess() 返回 `true` 或 `false`
  return await canUserAccess(to)
})
```



**全局解析守卫**

 每次导航时都会触发，但是确保在导航被确认之前，同时在所有**组件内守卫**和**异步路由组件**被解析之后，解析守卫就被正确调用

// TODO



**全局后置钩子**

```javascript
router.afterEach((to, from) => {
  sendToAnalytics(to.fullPath)
})
```



**路由独享守卫**

在路由上直接配置 beforeEnter， 只在从一个不同的 路由导航 进入路由时触发



**组件内守卫**

- `beforeRouteEnter`
- `beforeRouteUpdate`
- `beforeRouteLeave`

```javascript
const UserDetails = {
  template: `...`,
  beforeRouteEnter(to, from, next) {
    // 在渲染该组件的对应路由被验证前调用
    // 不能获取组件实例 `this` ！
    // 因为当守卫执行时，组件实例还没被创建！
    // 可以通过传一个回调给 next 来访问组件实例。在导航被确认的时候执行回调，并且把组件实例作为回调方法的参数
    next(vm => {
      // 通过 `vm` 访问组件实例
    })
  },
  beforeRouteUpdate(to, from) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 `/users/:id`，在 `/users/1` 和 `/users/2` 之间跳转的时候，
    // 由于会渲染同样的 `UserDetails` 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 因为在这种情况发生的时候，组件已经挂载好了，导航守卫可以访问组件实例 `this`
  },
  beforeRouteLeave(to, from) {
    // 在导航离开渲染该组件的对应路由时调用
    // 与 `beforeRouteUpdate` 一样，它可以访问组件实例 `this`
    // 这个 离开守卫 通常用来预防用户在还未保存修改前突然离开。该导航可以通过返回 false 来取消。
  },
}
```



**导航解析流程**

1. 导航被触发。
2. 在失活的组件里调用 `beforeRouteLeave` 守卫。
3. 调用全局的 `beforeEach` 守卫。
4. 在重用的组件里调用 `beforeRouteUpdate` 守卫(2.2+)。
5. 在路由配置里调用 `beforeEnter`。
6. 解析异步路由组件。
7. 在被激活的组件里调用 `beforeRouteEnter`。
8. 调用全局的 `beforeResolve` 守卫(2.5+)。
9. 导航被确认。
10. 调用全局的 `afterEach` 钩子。
11. 触发 DOM 更新。
12. 调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数，创建好的组件实例会作为回调函数的参数传入。





##### 路由元信息

一个路由匹配到的所有路由记录会暴露为 `$route` 对象(还有在导航守卫中的路由对象)的`$route.matched` 数组。

 `$route.meta` 方法，它是一个非递归合并**所有 `meta`** 字段的（从父字段到子字段）的方法。

```javascript
router.beforeEach((to, from) => {
  // 而不是去检查每条路由记录
  // to.matched.some(record => record.meta.requiresAuth)
  if (to.meta.requiresAuth && !auth.isLoggedIn()) {
    // 此路由需要授权，请检查是否已登录
    // 如果没有，则重定向到登录页面
    return {
      path: '/login',
      // 保存我们所在的位置，以便以后再来
      query: { redirect: to.fullPath },
    }
  }
})
```



##### 数据获取

**导航完成后获取**

加载数据期间可以显示 loading

```javascript
created() {
  // watch 路由的参数，以便再次获取数据
  this.$watch(
    () => this.$route.params,
    () => {
      this.fetchData()
    },
    // 组件创建完后获取数据，
    // 此时 data 已经被 observed 了
    { immediate: true }
  )
}
```



**导航完成前获取**

 `beforeRouteEnter` 守卫中获取数据，当数据获取成功后只调用 `next` 方法

```javascript
beforeRouteEnter(to, from, next) {
  // 没有 this
  axios.get('...').then(data => {
    next(vm => vm.setData(data))
  })
    
},
```



##### 组合式API



**在 setup 中访问路由**

setup 中没有 this

```javascript
// 获取 $router $route 导航守卫 useLink
import { useRouter, useRoute, onBeforeRouteLeave, onBeforeRouteUpdate, RouterLink, useLink } from 'vue-router'
```



##### 过渡特效

##### 滚动行为

当切换到新路由时，想要页面滚到顶部，或者是保持原先的滚动位置，就像重新加载页面那样

```javascript
const router = createRouter({
  scrollBehavior(to, from, savedPosition) {
    // 始终在元素 #main 上方滚动 10px
    return {
      // 也可以这么写
      // el: document.getElementById('main'),
      el: '#main',
      top: -10,
    }
  },
})
```



##### 路由懒加载

```javascript
// 将
// import UserDetails from './views/UserDetails'
// 替换成
const UserDetails = () => import('./views/UserDetails')

const router = createRouter({
  // ...
  routes: [{ path: '/users/:id', component: UserDetails }],
})
```



##### 导航故障



**检测导航故障**

```javascript
// 需要 await 等到导航结果
const navigationResult = await router.push('/my-profile')

if (navigationResult) {
  // 导航被阻止
} else {
  // 导航成功 (包括重新导航的情况)
  this.isMenuOpen = false
}
```



**鉴别导航故障**

```javascript
// 
import { NavigationFailureType, isNavigationFailure } from 'vue-router'

// 试图离开未保存的编辑文本界面
const failure = await router.push('/articles/2')

// 判断是哪一种
if (isNavigationFailure(failure, NavigationFailureType.aborted)) {
  // 给用户显示一个小通知
  showToast('You have unsaved changes, discard and leave anyway?')
}
```

- `aborted`：在导航守卫中返回 `false` 中断了本次导航。
- `cancelled`： 在当前导航还没有完成之前又有了一个新的导航。比如，在等待导航守卫的过程中又调用了 `router.push`。
- `duplicated`：导航被阻止，因为我们已经在目标位置了。



**导航故障的属性**

```javascript
// 正在尝试访问 admin 页面
router.push('/admin').then(failure => {
  if (isNavigationFailure(failure, NavigationFailureType.redirected)) {
    failure.to.path // '/admin'
    failure.from.path // '/'
  }
})
```



##### 动态路由



**添加路由**

如果新增加的路由与当前位置相匹配

- 需要用 `router.push()` 或 `router.replace()` 来**手动导航**，才能显示该新路由

  ```javascript
  router.addRoute({ path: '/about', component: About })
  // 我们也可以使用 this.$route 或 route = useRoute() （在 setup 中）
  router.replace(router.currentRoute.value.fullPath)
  ```

- 如果在导航守卫内部添加或删除路由，不应该调用 `router.replace()`，而是通过返回新的位置来触发重定向

  ```javascript
  router.beforeEach(to => {
    // 进行判断避免无限重定向
    if (!hasNecessaryRoute(to)) {
      router.addRoute(generateRoute(to))
      // 触发重定向
      return to.fullPath
    }
  })
  ```



**删除路由**

添加重复名称的路由，之前的会被覆盖

当路由被删除时，所有的别名和子路由也会被**同时删除**

```javascript
const removeRoute = router.addRoute(routeRecord)
removeRoute() // 删除路由如果存在的话，主要对于没有名称的路由
```



**添加嵌套路由**

将路由的 *name* 作为第一个参数传递给 `router.addRoute()`，这将有效地添加路由，就像通过 `children` 添加的一样

```javascript
router.addRoute('admin', { path: 'settings', component: AdminSettings })
```



**查看现有路由**

- `router.hasRoute()`：检查路由是否存在。
- `router.getRoutes()`：获取一个包含所有路由记录的数组。





### API 参考



#### composition api



##### useRoute

返回当前路由地址。相当于在模板中使用 $route。必须在 setup() 中调用

##### useRouter

返回 router 实例。相当于在模板中使用 $router。必须在 setup() 中调



#### Router 属性



##### currentRoute

当前路由地址

##### options

创建 router 是传递的原始配置对象



#### Router 方法

- addRoute
- beforeEach
- forward：前进
- getRoutes：获取所有路由记录的完整列表
- go：在历史中前进或后退

