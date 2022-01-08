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
```







#### 进阶

