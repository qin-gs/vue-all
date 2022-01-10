### Vuex



#### 介绍

**状态管理模式 + 库**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化



#### 核心概念



##### State

Vuex 使用**单一状态树**——用一个对象就包含了全部的应用层级状态。

从 store 实例中**读取**状态最简单的方法就是在**计算属性**中返回某个状态



**mapState 辅助函数**

该函数辅助将状态声明为计算属性

```javascript
// 在单独构建的版本中辅助函数为 Vuex.mapState
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',

    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```

当映射的计算属性的名称与 state 的子节点名称相同时，我们也可以给 `mapState` 传一个字符串数组。

```javascript
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
])
```

**与局部计算属性混用**

```javascript
computed: {
  localComputed () { /* ... */ },
  // 使用对象展开运算符将此对象混入到外部对象中
  ...mapState({
    // ...
  })
}
```



##### Getter

可以认为是 store 的计算属性

 接受 state 作为其第一个参数，接受其他 getter 作为第二个参数

也可以通过让 getter 返回一个函数，来实现给 getter 传参

```javascript
getters: {
  doneTodosCount (state, getters) {
    return getters.doneTodos.length
  },
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  } // store.getters.getTodoById(2)
}
```

`mapGetters` 辅助函数将 store 中的 getter 映射到局部计算属性，通过对象展开运算符混入组件的计算属性

使用对象给 getter 属性起别名



##### Mutation

更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。

每个 mutation 都有一个字符串的**事件类型 (type)\**和一个\**回调函数 (handler)**。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数

不能直接调用一个 mutation 处理函数，需要以相应的 type 调用 **store.commit** 方法

```javascript
store.commit('increment'， {count: 10})
```



**提交载荷**

可以是一个对象，可以包含多个字段



**对象风格的提交方式**

```javascript
store.commit({type: 'increment', amount: 10}) // 整个对象都交给 mutation 函数
```



**用常量代替 Mutation 事件类型**

```javascript
const SOME_MUTATION = 'SOME_MUTATION'
const store = createStore({
  state: { ... },
  mutations: {
    // 我们可以使用 ES2015 风格的计算属性命名功能来使用一个常量作为函数名
    [SOME_MUTATION] (state) {
      // 修改 state
    }
  }
})
```



**Mutation 必须是同步函数**

异步函数是状态无法跟踪



**组件中提交 Mutation**

`this.$store.commit('xxx')`

```javascript
export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

      // `mapMutations` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
  }
}
```



**mutation 都是同步事务**



##### Action

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含任意异步操作。

Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象

**调用异步 API** 和**分发多重 mutation**

```javascript
actions: {
  checkout ({ commit, state }, products) {
    // 把当前购物车的物品备份起来
    const savedCartItems = [...state.cart.added]
    // 发出结账请求，然后乐观地清空购物车
    commit(types.CHECKOUT_REQUEST)
    // 购物 API 接受一个成功回调和一个失败回调
    shop.buyProducts(
      products,
      // 成功操作
      () => commit(types.CHECKOUT_SUCCESS),
      // 失败操作
      () => commit(types.CHECKOUT_FAILURE, savedCartItems)
    )
  }
}
```



**组件中分发 Action**

组件中使用 `this.$store.dispatch('xxx')` 分发 action，或者使用 `mapActions` 辅助函数将组件的 methods 映射为 `store.dispatch` 调用

 `store.dispatch` 可以处理被触发的 action 的处理函数返回的 Promise，并且 `store.dispatch` 仍旧返回 Promise



##### Module

将 store 分割成**模块（module）**。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割

mutation 和 getter，接收的第一个参数是**模块的局部状态对象**

```javascript
const moduleA = {

  state: () => ({
    count: 0
  }),

  mutations: {
    increment (state) {
      // 这里的 `state` 对象是模块的局部状态
      state.count++
    }
  },

  getters: {
    sumWithRootCount (state, getters, rootState) {
      return state.count + rootState.count
    }
  }

  actions: {
    incrementIfOddOnRootSum ({ state, commit, rootState }) {
      if ((state.count + rootState.count) % 2 === 1) {
        commit('increment')
      }
    }
  }
}
```



**命名空间**

默认情况下，模块内部的 action 和 mutation 仍然是注册在**全局命名空间**的

可以通过添加 `namespaced: true` 的方式使其成为带命名空间的模块

当模块被注册后，它的所有 getter、action 及 mutation 都会自动根据模块注册的路径调整命名



在带命名空间的模块中访问全局内容

如果你希望使用全局 state 和 getter，`rootState` 和 `rootGetters` 会作为第三和第四参数传入 getter，也会通过 `context` 对象的属性传入 action。

若需要在全局命名空间内分发 action 或提交 mutation，将 `{ root: true }` 作为第三参数传给 `dispatch` 或 `commit` 即可

```javascript
modules: {
  foo: {
    namespaced: true,

    getters: {
      // 在这个模块的 getter 中，`getters` 被局部化了
      // 你可以使用 getter 的第四个参数来调用 `rootGetters`
      someGetter (state, getters, rootState, rootGetters) {
        getters.someOtherGetter // -> 'foo/someOtherGetter'
        rootGetters.someOtherGetter // -> 'someOtherGetter'
      },
      someOtherGetter: state => { ... }
    },

    actions: {
      // 在这个模块中， dispatch 和 commit 也被局部化了
      // 他们可以接受 `root` 属性以访问根 dispatch 或 commit
      someAction ({ dispatch, commit, getters, rootGetters }) {
        getters.someGetter // -> 'foo/someGetter'
        rootGetters.someGetter // -> 'someGetter'

        dispatch('someOtherAction') // -> 'foo/someOtherAction'
        dispatch('someOtherAction', null, { root: true }) // -> 'someOtherAction'

        commit('someMutation') // -> 'foo/someMutation'
        commit('someMutation', null, { root: true }) // -> 'someMutation'
      },
      someOtherAction (ctx, payload) { ... }
    }
  }
}
```



若需要在带命名空间的模块注册全局 action，你可添加 `root: true`，并将这个 action 的定义放在函数 `handler` 中

```javascript
modules: {
  foo: {
    namespaced: true,
    actions: {
      someAction: {
        root: true,
        handler (namespacedContext, payload) { ... } // -> 'someAction'
      }
    }
  }
}
```

当使用 `mapState`、`mapGetters`、`mapActions` 和 `mapMutations` 这些函数来绑定带命名空间的模块时，可以将模块的空间名称字符串作为第一个参数传递给上述函数，这样所有绑定都会自动将该模块作为上下文。

```javascript
computed: {
  ...mapState('some/nested/module', {
    a: state => state.a,
    b: state => state.b
  })
},
methods: {
  ...mapActions('some/nested/module', [
    'foo', // -> this.foo()
    'bar' // -> this.bar()
  ])
}
```



**模块动态注册**

使用 `store.registerModule` 方法注册模块

使用 `store.unregisterModule(moduleName)` 来动态卸载模块。注意，你不能使用此方法卸载静态模块（即创建 store 时声明的模块）



使得其他 Vue 插件可以通过在 store 中附加新模块的方式来使用 Vuex 管理状态。

嵌套模块应该以数组形式传递给 `registerModule` 和 `hasModule`，而不是以路径字符串的形式传递给 module

```javascript
// 注册模块 `myModule`
store.registerModule('myModule', {
  // ...
})
// 注册嵌套模块 `nested/myModule`
store.registerModule(['nested', 'myModule'], {
  // ...
})
```



**模块重用**

// TODO



#### 进阶



##### 项目结构

##### 组合式 API

```javascript
import { computed } from 'vue'
import { useStore } from 'vuex'

export default {
  setup () {
    const store = useStore()

    return {
      // 在 computed 函数中访问 state
      count: computed(() => store.state.count),

      // 在 computed 函数中访问 getter
      double: computed(() => store.getters.double)
       
      // 使用 mutation
      increment: () => store.commit('increment'),

      // 使用 action
      asyncIncrement: () => store.dispatch('asyncIncrement')
    }
  }
}
```



##### 插件

##### 严格模式

在创建 store 的时候传入 `strict: true`

发生了状态变更且不是由 mutation 函数引起的，将会抛出错误。这能保证所有的状态变更都能被调试工具跟踪到。

发布环境下不要使用



##### 表单处理

严格模式中 v-model 直接修改值会抛出错误

- 给 `<input>` 中绑定 value，然后侦听 `input` 或者 `change` 事件，在事件回调中调用一个方法
- 使用带有 setter 的双向绑定计算属性

