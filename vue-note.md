### Vue3



#### 基础

##### 应用 & 组件实例

```vue
<div id="event-handling">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">反转 Message</button>
</div>
<script>
const EventHandling = {
  data() {
    return {
      message: 'Hello Vue.js!'
    }
  },
  methods: {
    reverseMessage() {
      this.message = this.message
        .split('')
        .reverse()
        .join('')
    }
  }
}

// 创建 Vue 组件
const app = Vue.createApp({
  components: {
    TodoItem // 注册一个新组件
  },
  ... // 组件的其它 property
})

Vue.createApp(EventHandling).mount('#event-handling')
</script>
```

指令 `v-`

v-bind 绑定元素

v-on 添加事件监听器

v-model 双向绑定表单输入 和 应用状态

v-if 控制标签显示

v-for 绑定数组循环渲染



**创建应用实例**

**根组件**

```javascript
// 可以链式调用，也可以直接用 app 调用
let app = Vue.createApp({}) // 传递给 createApp 的选项用于配置根组件，该组件被作为渲染的起点
  .component('SearchInput', SearchInputComponent)
  .directive('focus', FocusDirective)
app.use(LocalePlugin)
const vm = app.mount('#app')
```

`mount` 不返回应用本身。相反，它返回的是根组件实例。

**组件实例 property**

在 `data` 中定义的 property 是通过组件实例暴露的

```javascript
const app = Vue.createApp({
  data() {
    return { count: 4 }
  }
})

const vm = app.mount('#app')

console.log(vm.count) // => 4 直接获取 data 中的数据
```

**生命周期钩子**

每个组件在被创建时都要经过一系列的初始化过程，在这个过程中也会运行一些叫做**生命周期钩子**的函数，这给了用户在不同阶段添加自己的代码的机会

不要在选项 property 或 回调函数 使用箭头函数 (没有 this 会一直向上查找)

**生命周期**

![实例的生命周期](./src/assets/lifecycle.svg)

##### 模板语法

1. **插值**

   两个大括号 `{{ }}` 文本插值 (即使文本是html，可不会被渲染)，可以放置**一个**表达式

   v-once 一次性插值，数据更新时不会改变

   v-html 输出真正的 html (可以配置 style)

   ```html
   <p>Using v-html directive: <span v-html="rawHtml"></span></p>
   ```

   使用 v-bind 给 标签绑定属性

   ```html
   <div v-bind:id="dynamicId"></div>
   <!-- 如果绑定的值是 `null` 或 `undefined`，那么该 attribute 将不会被包含在渲染的元素上 -->
   
   <!-- 8个falsy: false、0、-0、0n、""、null、undefined 和 NaN  -->
   <!-- 除8个falsy之外全为truthy-->
   
   <button v-bind:disabled="isButtonDisabled">按钮</button>
   <!-- 布尔类型的属性 truthy + 空字符串 会被包含在内，其他falsy会被省略 -->
   ```

   

2. **指令**

   带有 `v-` 前缀的特殊属性

   接收参数的指令，在指令后用 `:` 表示

   ```html
   <!-- href 可以接收一个参数，使用 v-bind 相应实更新 href attribute -->
   <a v-bind:href="url"> ... </a>
   
   <!-- v-on 监听dom事件-->
   <a v-on:click="doSomething"> ... </a>
   ```

   动态参数

   ```html
   <!--  -->
   <!-- [] 内会被作为表达式进行求值，求值结果作为参数使用 -->
   <a v-bind:[attributeName]="url"> ... </a>
   <a v-on:[eventName]="doSomething"> ... </a>
   ```

   修饰符

   使用 `·` 指明的特殊后缀，用来指出一个指令应该以特殊方式绑定

   `.prevent` 告诉 `v-on` 对于触发的事件调用 `event.preventDefault()` 方法

   

3. **缩写**

   v-bind  -->  :

   v-on  --> @

   ```html
   <a v-bind:href="url"> ... </a>
   <a :href="url"> ... </a>
   
   <a v-on:click="doSomething"> ... </a>
   <a @click="doSomething"> ... </a>
   ```

   模板表达式都被放在沙盒中，只能访问一个受限的全局变量列表，如 Math 和 Date。你不应该在模板表达式中试图访问用户定义的全局变量

   ```javascript
   const GLOBALS_WHITE_LISTED =
     'Infinity,undefined,NaN,isFinite,isNaN,parseFloat,parseInt,decodeURI,' +
     'decodeURIComponent,encodeURI,encodeURIComponent,Math,Number,Date,Array,' +
     'Object,Boolean,String,RegExp,Map,Set,JSON,Intl,BigInt'
   ```

   

##### Data Property 和 方法

**Data Property**

组件的 data 选项是一个函数。Vue 会在创建新组件实例的过程中调用此函数。它应该返回一个对象，然后 Vue 会通过响应性系统将其包裹起来，并以 $data 的形式存储在组件实例中。

该对象的任何顶级 property 也会直接通过组件实例暴露出来



**method**

包含方法的**对象**

Vue 自动为 `methods` 绑定 `this`，避免使用 箭头函数

从模板调用的方法 ( `{{}}`中调用方法 ) 不应该有任何副作用，比如更改数据或触发异步进程



**防抖和节流**

lodash



##### 计算属性和侦听器



**计算属性**

对于任何包含响应式数据的复杂逻辑，应该使用**计算属性**

**计算属性将基于它们的响应依赖关系缓存**

计算属性只会在相关响应式依赖发生改变时重新求值，依赖不改变，多个访问返回之前的计算结果

```javascript
// 这个计算属性永远不会更新
computed: {
  now() {
    return Date.now() // 不是响应式依赖
  }
}
```

计算属性默认只有 getter，可以自己添加 setter



**侦听器**

watch

```javascript
data() {
  return {
    question: '',
    answer: 'Questions usually contain a question mark. ;-)'
  }
},
watch: {
  // 每当 question 发生变化时，该函数将会执行
  question(newQuestion, oldQuestion) {
    if (newQuestion.indexOf('?') > -1) {
      this.getAnswer()
    }
  }
},
```



##### css 与 style 绑定



**css 绑定**

```html
<div :class="{ active: isActive }"></div>
<!--  active 这个 class 存在与否将取决于 data property isActive 的 truthiness -->
<!-- :class 可以与普通 class 共存 -->
<!-- 可以在这里绑定一个返回对象的计算属性 -->
```

数组语法

```html
<div :class="[{ active: isActive }, errorClass]"></div>
```

在带有单个根元素的自定义组件上使用 `class` attribute 时，这些 class 将被添加到该元素中。此元素上的现有 class 将**不会被覆盖**

如果组件有多个根元素，需要定义哪些部分将接收这个 class。可以使用 `$attrs` 组件 property 执行此操作

```html
<div id="app">
  <my-component class="baz"></my-component>
</div>
<script>
const app = Vue.createApp({})

app.component('my-component', {
  template: `
    <p :class="$attrs.class">Hi!</p>
    <span>This is a child component</span>
  `
})
</script>
```



**style 绑定**

```html
<div :style="styleObject"></div>

<script>
data() {
  return {
    styleObject: {
      color: 'red',
      fontSize: '13px'
    }
  }
}
</script>
```



##### 条件渲染

v-if: 惰性的（条件成立才会渲染）

v-else-if

v-else

v-show: 简单的切换元素 display 属性（开始就会被渲染，即使不显示）

v-if + v-for 同时使用，v-if 优先级更高



##### 列表渲染

```html
<li v-for="(item, index) in items"> 遍历数组
<div v-for="item of items"></div> 遍历数组

<li v-for="(value, name) in myObject"> 遍历对象
<li v-for="(value, name, index) in myObject"> 遍历对象
```

更新使用 `v-for` 渲染的元素列表时，它默认使用“**就地更新**”的策略。如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序，而是就地更新每个元素，并且确保它们在每个索引位置正确渲染。



**数组变更检测**

如下方法会触发视图更新 `push, pop, shift, unshif, splice, sort, reverse`

**替换数组**

`filter, concat, slice` 不变更原始数组，返回**新数组**

**显示过滤排序后的结果**

使用计算属性

**v-for + v-if**

v-if 优先级更高，v-if 不能访问 v-for 中的变量

**组件上使用 v-for**

数据不会自动传递到组件中，需要使用 props



##### 事件处理



**监听事件**

v-on:click --> @click

传入 `$event` 访问原始 dom

可以同时有多个方法，逗号分割



**事件修饰符**

- .stop 阻止事件冒泡
- .prevent 提交事件不再重载页面
- .capture 添加事件监听器时使用事件捕获模式，即内部元素触发的事件现在此处理，再交给内部元素
- .self 当 evett.target 是元素自身时触发元素，即事件不是从元素内部触发的
- .once 点击事件只触发一次
- .passive 滚动事件的默认行为会立刻触发，不会等待 onScroll 完成

**按键修饰符**

- .enter
- .tab
- .delete
- .esc
- .space
- .up
- .down
- .left
- .right

```html
<input @keyup.page-down="onPageDown" />
<!-- 只有 $event.key 等于 'PageDown' 时被调-->
```

**系统修饰键**

- .ctrl
- .alt
- .shift
- .meta

仅在按下相应按见时才出发鼠标或键盘事件的监听器

```html
<!-- Alt + Enter 组合按键 -->
<input @keyup.alt.enter="clear" />
```

**.exact 修饰符**

精确控制系统修饰符

**鼠标按钮修饰符**

- .left
- .right
- .middle

##### 表单输入绑定

- v-bind
- v-model

修饰符

- .lazy 将 input 事件触发后输入框的值和数据进行同步 转为 change 事件(失去焦点)后同步
- .number 将输入转为数值类型
- .trim 自动过滤首尾空白字符



##### 组件基础

**组件复用**

**组件组织**

- 全局注册 `app.component("component-name", {})`

- 局部注册

**prop 向子组件传递数据**

**监听子组件事件**

父组件通过 `@method-name='methodName'` 传入事件，通过 $event 访问子组件提供的值，如果是一个方法子组件提供的值会被作为第一个参数传进去

子组件通过 `$emit('methodName', value)`传入事件名称触发事件，第二个参数可以提供一个值

**组件上使用 v-model**

```html
<input v-model="searchText" />
<input :value="searchText" @input="searchText = $event.target.value" />
```

**通过插槽分发内容**

子组件中使用 `<slot>` 作为占位符，放置父组件传过来的数据

**动态组件**

`<component>` 标签加 `is` 完成组件间的动态切换

**解析dom时的问题**

元素位置受限

对于元素出现位置有严格限制的元素(比如 table 下面时 tr)，可以用 `is` 处理

```html
<!-- 解析错误 -->
<table>
  <blog-post-row></blog-post-row>
</table>

<!-- 使用 is 进行变通，需要以 .vue 作为前缀，避免和原生自定义原生混淆-->
<table>
  <tr is="vue:blog-post-row"></tr>
</table>
```



#### 深入组件



##### 组件注册

```javascript
// 全局注册
app.component('component-a', {
  /* ... */
})

// 局部注册
const ComponentA = {
  /* ... */
}
const app = Vue.createApp({
  components: {
    'component-a': ComponentA,
  }
})
```

**局部注册的组件在其子组件中不可用**



##### props

##### 传递静态或动态 prop

```html
<blog-post title="My journey with Vue"></blog-post>
<!-- 动态赋予一个变量的值 -->
<blog-post :title="post.title"></blog-post>
```

传入其他类型的值时需要用 v-bind 告诉 vue 这是一个js 表达式不是字符串

将一个对象所有的属性都作为 prop 传入，可以使用不带参数的 v-bind

```html
<blog-post v-bind="post"></blog-post>
<blog-post v-bind:id="post.id" v-bind:title="post.title"></blog-post>
```

**单向数据流**

父子组件之间会形成 **单向下行绑定**，父组件 prop 的更新会向下流动到子组件，返回来不行

防止子组件意外变更父组件的状态，不应该在子组件中变更 prop 的值

- 这个 prop 用来传递一个初始值；这个子组件接下来希望将其作为一个本地的 prop 数据来使用

  ```javascript
  props: ['initialCounter'],
  data() {
    return {
      counter: this.initialCounter
    }
  }
  ```

-  prop 以一种原始的值传入且需要进行转换

  ```javascript
  props: ['size'],
  computed: {
    normalizedSize() {
      return this.size.trim().toLowerCase()
    }
  }
  ```

在 JavaScript 中对象和数组是通过引用传入的，所以对于一个数组或对象类型的 prop 来说，在子组件中改变这个对象或数组本身**将会**影响到父组件的状态，且 Vue 无法为此发出警告。作为一个通用规则，应该避免修改任何 prop，包括对象和数组，因为这种做法无视了单向数据绑定，且可能会导致意料之外的结果。



##### prop 验证

- String
- Number
- Boolean
- Array
- Object
- Date
- Function
- Symbol: ES6 引入一种新的原始数据类型 Symbol ，表示独一无二的值，最大的用法是用来定义对象的唯一属性名。 // TODO



##### 非 prop 的 Attribute

`class, style, id` 通过 `$attrs` 访问

**Attribute 继承**

当组件返回单个根节点时，非 prop 的 attribute 和 事件监听器 将自动添加到根节点的 attribute 中

**禁用 Attribute 继承**

禁用 attribute 继承的常见场景是需要将 attribute 应用于根节点之外的其他元素。

在组件定义的时候设置 `inheritAttrs: false`

通过将 `inheritAttrs` 选项设置为 `false`，你可以使用组件的 `$attrs` property 将 attribute 应用到其它元素上，该 property 包括组件 `props` 和 `emits` property 中未包含的所有属性 (例如，`class`、`style`、`v-on` 监听器等)。

```html
<!-- 
如果需要将所有非 prop 的 attribute 应用于 input 元素而不是根 div 元素，可以使用 v-bind 缩写来完成
有了这个新配置，data-status attribute 将应用于 input 元素！
-->
<date-picker data-status="activated"></date-picker>

<script>
app.component('date-picker', {
  inheritAttrs: false, // 不继承属性
  template: `
    <div class="date-picker">
      <input type="datetime-local" v-bind="$attrs" />
    </div>
  `
})
</script>

```

**多个根节点的 Attribute 继承**

如果未显式绑定 `$attrs`，将发出运行时警告。



##### 自定义事件



**定义自定义事件**

通过 `emits` 选项在组件上定义发出的事件，如果定义了原生事件(clikc)，将使用组件中的替换原生的

**验证抛出的事件**

要添加验证，请为事件分配一个函数，该函数接收传递给 `$emit` 调用的参数，并返回一个布尔值以指示事件是否有效。

**v-model 参数** // TODO

默认情况下，组件上的 `v-model` 使用 `modelValue` 作为 prop 和 `update:modelValue` 作为事件。我们可以通过向 `v-model` 传递参数来修改这些名称

**自定义v-model修饰符**

添加到组件 `v-model` 的修饰符将通过 `modelModifiers` prop 提供给组件



##### 插槽



**插槽内容**

如果 template 中**没有**包含一个 `<slot>` 元素，则该组件起始标签和结束标签之间的任何内容都会被抛弃。



**渲染作用域**

想在一个插槽中使用数据时，该插槽可以访问与模板其余部分相同的实例 property (即相同的“作用域”)

父级模板里的所有内容都是在父级作用域中编译的；子模板里的所有内容都是在子作用域中编译的



**备用内容**

父组件不提供任何插槽内容时，备用内容会被渲染；如果父组件提供了，直接使用父组件的

```html
<button type="submit">
  <slot>Submit</slot>
</button>
```



**具名插槽**

多个插槽

一个不带 `name` 的 `<slot>` 出口会带有隐含的名字“default”。

在向具名插槽提供内容的时候，我们可以在一个 `<template>` 元素上使用 `v-slot` 指令，并以 `v-slot` 的参数的形式提供其名称

```html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>

<base-layout>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  <template v-slot:default>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </template>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```



**作用域插槽**

https://www.jianshu.com/p/0c9516a3be80

根据自己的需要将任意数量的 attribute 绑定到 `slot` 上

在父级作用域中，我们可以使用带值的 `v-slot` 来定义我们提供的插槽 prop 的名字

```html
<ul>
  <li v-for="( item, index ) in items">
    <slot :item="item" :index="index" :another-attribute="anotherAttribute"></slot>
  </li>
</ul>

<todo-list>
  <template v-slot:default="slotProps">
    <i class="fas fa-check"></i>
    <span class="green">{{ slotProps.item }}</span>
  </template>
</todo-list>
```



**独占插槽的缩写**

在上述情况下，当被提供的内容*只有*默认插槽时，组件的标签才可以被当作插槽的模板来使用。这样我们就可以把 `v-slot` 直接用在组件上

不带参数的 `v-slot` 被假定对应默认插槽

```html
<!-- todo-list 是一个子组件，在标签上直接使用 v-slot -->
<todo-list v-slot:default="slotProps">
  <i class="fas fa-check"></i>
  <span class="green">{{ slotProps.item }}</span>
</todo-list>
```

**解构插槽prop**

```html
<!-- 可以重命名，提供默认值-->
<todo-list v-slot="{ item: todo, another = 'default string' }">
  <i class="fas fa-check"></i>
  <span class="green">{{ todo }}</span>
</todo-list>
```

**具名插槽缩写**

v-slot --> #

v-slot:header --> #header 没有参数的时候会才能直接用

v-slot:header --> #header = "{ item }" 如果有参数需要这样处理



##### **Provide / Inject**

子组件依赖父组件的部分数据，但是传递层级太深

父组件有一个 `provide` 选项来提供数据，子组件有一个 `inject` 选项来开始使用这些数据

类似于：长距离的 prop

- 父组件不需要知道哪些子组件使用了它 provide 的 property
- 子组件不需要知道 inject 的 property 来自哪里

默认不是相应式的，数据的更改不会反映在 inject 的 properties 中

- 通过传递 ref 或 reactive 给 provide 使其变成相应式的

- 为 provide 的 `todoLength` 分配一个组合式 API `computed` property

```javascript
app.component('todo-list', {
  data() {
    return {
      todos: ['Feed a cat', 'Buy tickets']
    }
  },
  provide() {
    return {
      todoLength: this.todos.length,
      // todoLength: Vue.computed(() => this.todos.length) // 变成响应式
    }
  },
  template: `
    ...
  `
})

app.component('todo-list-statistics', {
  inject: ['todoLength'],
  created() {
    console.log(`Injected property: ${this.todoLength}`) // > 注入的 property: John Doe
  }
})
```



##### **动态组件 & 异步组件**

**动态组件**

使用 `is` 来切换不同的组件，想保持这些组件的状态，以避免反复渲染导致的性能问题，可以用一个 `<keep-alive>` 元素将其动态组件包裹起来

**异步组件**

在大型应用中，我们可能需要将应用分割成小一些的代码块，并且只在需要的时候才从服务器加载一个模块

```javascript
const { createApp, defineAsyncComponent } = Vue

const app = createApp({})

const AsyncComp = defineAsyncComponent(
  () =>
    new Promise((resolve, reject) => {
      resolve({
        template: '<div>I am async!</div>'
      })
    })
)

app.component('async-example', AsyncComp)
```



##### 模板引用

尽管存在 prop 和事件，但有时你可能仍然需要在 JavaScript 中直接访问子组件。为此，可以使用 `ref` attribute 为子组件或 HTML 元素指定引用 ID

`$refs` 只会在组件渲染完成之后生效。

```html
<input ref="input" />

<script>
const app = Vue.createApp({})

app.component('base-input', {
  template: `
    <input ref="input" />
  `,
  methods: {
    focusInput() {
      this.$refs.input.focus()
    }
  },
  mounted() {
    this.focusInput()
  }
})
</script>
```



##### **处理边界情况**

- 控制更新

- 强制更新 $forceUpdate

- 低级静态组件 和 v-once

  有一个包含**很多**静态内容的组件。在这些情况下，可以通过向根元素添加 `v-once` 指令来确保只对其求值一次，然后进行缓存





#### 过渡 & 动画



#### 可复用 & 组合



##### 组合式 API

碎片化使得理解和维护复杂组件变得困难。选项的分离掩盖了潜在的逻辑问题。此外，在处理单个逻辑关注点时，我们必须不断地“跳转”相关代码的选项块。

组合式 API：将同一个逻辑关注点相关代码收集在一起



**setup 组件选项**

新的 `setup` 选项在组件创建**之前**执行，在 `setup` 中你应该避免使用 `this`，因为它不会找到组件实例；。`setup` 的调用发生在 `data` property、`computed` property 或 `methods` 被解析之前，所以它们无法在 `setup` 中被获取

`setup` 选项是一个接收 `props` 和 `context` 的函数； `setup` 返回的所有内容都暴露给组件的其余部分 (计算属性、方法、生命周期钩子等等) 以及组件的模板



**带 ref 的响应式变量**

通过一个新的 `ref` 函数使任何响应式变量在任何地方起作用，创建了一个**响应式引用**。

`ref` 接收参数并将其包裹在一个带有 `value` property 的对象中返回，然后可以使用该 property 访问或更改响应式变量的值 (将所有数据类型都封装成对象(引用传递)，避免失去响应性)

```javascript
import { ref } from 'vue'

const counter = ref(0)

console.log(counter) // { value: 0 }
console.log(counter.value) // 0

counter.value++
console.log(counter.value) // 1
```



**在 setup 内注册生命周期钩子**

组合式 API 上的生命周期钩子与选项式 API 的名称相同，但前缀为 `on`：即 `mounted` 看起来会像 `onMounted`

这些函数接受一个回调，当钩子被组件调用时，该回调将被执行。

```javascript
// 在组件中
setup (props) {
  onMounted(getUserRepositories) // 在 `mounted` 时调用 `getUserRepositories`
  return {  }
}
```



**watch 响应式更改**



- 一个想要侦听的**响应式引用**或 getter 函数
- 一个回调
- 可选的配置选项

```javascript
setup (props) {
  const counter = ref(0)
  watch(counter, (newValue, oldValue) => {
    console.log('The new counter value is: ' + counter.value)
  })
}
```

`toRefs` 确保侦听器能够根据 `user` prop 的变化做出反应。



**独立的 computed 属性**

这里给 `computed` 函数传递了第一个参数，它是一个类似 getter 的回调函数，输出的是一个*只读*的**响应式引用**。为了访问新创建的计算变量的 **value**，我们需要像 `ref` 一样使用 `.value` property

```javascript
import { ref, computed } from 'vue'

const counter = ref(0)
const twiceTheCounter = computed(() => counter.value * 2)

counter.value++
console.log(counter.value) // 1
console.log(twiceTheCounter.value) // 2
```



##### setup

setup 函数接收两个参数 props, context

- **props**

  `setup` 函数中的 `props` 是响应式的，当传入新的 prop 时，它将被更新

  因为 `props` 是响应式的，你**不能使用 ES6 解构**，它会消除 prop 的响应性

  ```javascript
  // 避免解构导致响应式丢失
  import { toRefs } from 'vue'
  
  setup(props) {
    const { title } = toRefs(props)
  
    console.log(title.value)
  }
  ```

  toRefs(obj)  将整个对象变成响应式

  toRef(obj, 'item')  为对象添加响应式

- **context**

  非响应式的(可以解构)，暴露了其他能在 setup 中使用的值 

  `attrs, slots, emit, expose`



**结合模板使用**

如果 `setup` 返回一个对象，那么该对象的 property 以及传递给 `setup` 的 `props` 参数中的 property 就都可以在模板中访问到

从 setup 返回的 refs 在模板中访问时是被自动浅解包的，因此不应在模板中使用 .value



**使用渲染函数**

`setup` 还可以返回一个渲染函数，该函数可以直接使用在同一作用域中声明的响应式状态

返回一个渲染函数将阻止我们返回任何其它的东西。

但当我们想要将这个组件的方法通过模板 ref 暴露给父组件时，可以通过调用 `expose` 来解决这个问题，给它传递一个对象，其中定义的 property 将可以被外部组件实例访问



##### 生命周期钩子

这些函数接受一个回调函数，当钩子被组件调用时将会被执行

| 选项式 API        | Hook inside `setup` |
| ----------------- | ------------------- |
| `beforeCreate`    | Not needed*         |
| `created`         | Not needed*         |
| `beforeMount`     | `onBeforeMount`     |
| `mounted`         | `onMounted`         |
| `beforeUpdate`    | `onBeforeUpdate`    |
| `updated`         | `onUpdated`         |
| `beforeUnmount`   | `onBeforeUnmount`   |
| `unmounted`       | `onUnmounted`       |
| `errorCaptured`   | `onErrorCaptured`   |
| `renderTracked`   | `onRenderTracked`   |
| `renderTriggered` | `onRenderTriggered` |
| `activated`       | `onActivated`       |
| `deactivated`     | `onDeactivated`     |



##### Provide / Inject

在组合式 API 中使用 provide/inject。两者都只能在当前活动实例的 setup() 期间调用

从 `vue` 显式导入 `provide` 方法，将变量改成响应式

**建议尽可能将对响应式 property 的所有修改限制在\*定义 provide 的组件\*内部**。

```html
<script>
import { provide, reactive, ref } from 'vue'
import MyMarker from './MyMarker.vue'

export default {
  components: {
    MyMarker
  },
  setup() {
    const location = ref('North Pole')
    const geolocation = reactive({
      longitude: 90,
      latitude: 135
    })

    const updateLocation = () => {
      location.value = 'South Pole'
    }

    provide('location', readonly(location)) // 只读
    provide('geolocation', geolocation) // 响应式
    provide('updateLocation', updateLocation) // 提供一个方法，让子组件修改数据
  }
}
</script>

<script>
import { inject } from 'vue'

export default {
  setup() {
    const userLocation = inject('location', 'The Universe')
    const userGeolocation = inject('geolocation')
    const updateUserLocation = inject('updateLocation') // 通过方法修改组件数据

    return {
      userLocation,
      userGeolocation,
      updateUserLocation
    }
  }
}
</script>
```

如果要确保通过 `provide` 传递的数据不会被 inject 的组件更改，我们建议对提供者的 property 使用 `readonly`。



**模板应用**

在使用组合式 API 时，响应式引用和模板引用的概念是统一的。为了获得对模板内元素或组件实例的引用，我们可以像往常一样声明 ref 并从 setup() 返回

`watch()` 和 `watchEffect()` 在 DOM 挂载或更新*之前*运行副作用，所以当侦听器运行时，模板引用还未被更新，因此，使用模板引用的侦听器应该用 `flush: 'post'` 选项来定义，这将在 DOM 更新*后*运行副作用，确保模板引用与 DOM 保持同步，并引用正确的元素。



##### Mixin

分发 Vue 组件中的可复用功能，一个 mixin 对象可以包含任意组件选项。当组件使用 mixin 对象时，所有 mixin 对象的选项将被“混合”进入该组件本身的选项。

**选项合并**

当组件和 mixin 对象含有同名选项时，当组件和 mixin 对象含有同名选项时，会以组件自身的数据为优先。

同名钩子函数将合并为一个**数组**，因此都将被调用。另外，mixin 对象的钩子将在组件自身钩子**之前**调用。



**全局mixin**

**自定义选项合并策略**

```javascript
const app = Vue.createApp({})

app.config.optionMergeStrategies.customOption = (toVal, fromVal) => {
  // return mergedVal
}
```



##### 自定义指令

```javascript
const app = Vue.createApp({})
// 注册一个全局自定义指令 `v-focus`
app.directive('focus', {
  // 当被绑定的元素挂载到 DOM 中时……
  mounted(el) {
    // 聚焦元素
    el.focus()
  }
})

// 注册局部指令，组件中放入选项
directives: {
  focus: {
    // 指令的定义
    mounted(el) {
      el.focus()
    }
  }
}
```

**钩子函数**

// TODO



##### Teleport

// TODO



##### 渲染函数

向组件中传递不带 `v-slot` 指令的子节点时，比如 `anchored-heading` 中的 `Hello world!` ，这些子节点被存储在组件实例中的 `$slots.default` 中



**h()**

接收三个参数：

- html 标签名
- props
- 包含子节点的数组



**用javascript代码模板功能**

// TODO



##### 插件

// TODO





#### 高阶



##### Vue 与 Web Componens



##### 响应性

1. **当一个值被读取时进行追踪**
2. **当某个值改变时进行检测**
3. **重新运行代码来读取原始值**







##### 渲染机制和优化



##### Vue2 中更改检测





### API 参考



#### 应用配置



#### 应用API



#### 选项

##### data

该函数返回组件实例的 data 对象

实例创建之后，可以通过 `vm.$data` 访问原始数据对象。组件实例也代理了 data 对象上所有的 property，因此访问 `vm.a` 等价于访问 `vm.$data.a`



##### props

用于从父组件接收数据的数组或对象。



##### computed

计算属性将被混入到组件实例中



##### methods

methods 将被混入到组件实例中。

**不应该使用箭头函数来定义 method 函数**



##### watch

一个对象，键是要侦听的响应式 property——包含了 data 或 computed property，而值是对应的回调函数。值也可以是方法名，或者包含额外选项的对象。

可以提供三个选项 `deep, immediate, flush`



##### emits

从组件触发自定义事件，用来定义一个组件可以向其父组件触发的事件



##### expose

一个将暴露在公共组件实例上的 property 列表

// TODO



#### DOM

##### template

##### render



#### 生命周期钩子

所有生命周期钩子的 `this` 上下文将自动绑定至实例中，因此你可以访问 data、computed 和 methods。

**不应该使用箭头函数来定义一个生命周期方法** 



#### 选项/资源

- directives：声明一组可用于组件实例中的指令

- components：声明一组可用于组件实例中的组件



#### 组合

##### mixins  extends

##### provide/inject

这对选项需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。

`provide` 和 `inject` 绑定并不是响应式的。这是刻意为之的。然而，如果你传入了一个响应式的对象，那么其对象的 property 仍是响应式的。

##### setup

是组件内部使用组合式 API 的入口点

在创建组件实例时，在初始 prop 解析之后(pmdcw)立即调用 setup。在生命周期方面，它是在 beforeCreate 钩子之前调用的。

如果 `setup` 返回一个对象，则该对象的属性将合并到组件模板的渲染上下文中

从 setup 返回的 refs 在模板中访问时会自动解包，因此模板中不需要 .value

```javascript
// props 不可解构
setup(props, { attrs, slots, emit, expose }) {
  // 稍后可能会调用的函数
  function onClick() {
    console.log(attrs.foo) // 保证是最新引用
  }
}
```



#### 实例 property

- $data：组件实例正在侦听的数据对象。
- $props：当前组件接收到的 props 对象。
- $el：组件实例正在使用的根 DOM 元素。
- $options：用于当前组件实例的初始化选项。
- $parent：父实例，如果当前实例有的话。
- $root：当前组件树的根组件实例。
- $slots：用来以编程方式访问通过插槽分发的内容。
- $refs：一个对象，持有注册过 ref attribute 的所有 DOM 元素和组件实例。
- $attrs：包含了父作用域中不作为组件 props 或自定义事件的 attribute 绑定和事件。



#### 实例方法

- $watch：侦听组件实例上的响应式 property 或函数计算结果的变化。当侦听的值是一个对象或者数组时，对其属性或元素的任何更改都不会触发侦听器，因为它们引用相同的对象/数组，返回一个取消侦听函数，用来停止触发回调
- $emit：触发当前实例上的事件。附加参数都会传给监听器回调。
- $forceUpdate：迫使组件实例重新渲染。
- $nextTick：将回调延迟到下次 DOM 更新循环之后执行。



#### 指令

- v-text

  ```html
  <span v-text="msg"></span>
  <!-- 等价于 -->
  <span>{{msg}}</span>
  ```

- v-html：内容按普通 HTML 插入 - 不会作为 Vue 模板进行编译(css是有用的)

- v-show

- v-if / v-else-if / v-esle

- v-for

  ```html
  <div v-for="(item, index) in items"></div>
  <div v-for="(value, key, index) in object"></div>
  ```

- v-on：事件绑定 `@`(可以添加修饰符)

- v-bind 属性绑定 `:`  (可以添加修饰符)

  - `.camel` - 将 kebab-case attribute 名转换为 camelCase。
  - `.prop` - 将一个绑定强制设置为一个 DOM property。3.2+
  - `.attr` - 将一个绑定强制设置为一个 DOM attribute。3.2+

- v-model：表单绑定

- v-slot：插槽 `#` 提供具名插槽或需要接收 prop 的插槽。

- v-pre：跳过这个元素和它的子元素的编译过程。可以用来显示原始 Mustache 标签。

- v-cloak：这个指令保持在元素上直到关联组件实例结束编译，可以隐藏未编译的 Mustache 标签直到组件实例准备完毕。

- v-once：只渲染元素和组件一次

- v-memo：记住一个模板的子树。元素和组件上都可以使用。该指令接收一个固定长度的数组作为依赖值进行记忆比对。如果数组中的每个值都和上次渲染的时候相同，则整个该子树的更新会被跳过。

- ~~v-is~~



#### 特殊 attribute

- key
- ref：用来给元素或子组件注册引用信息
- is：使用动态组件



#### 内置组件

- component：和上面的 is 属性使用，动态决定渲染哪个组件
- transition
- transition-group
- keep-alive：缓存不活动的组件实例，而不是销毁他
- slot：具名插槽
- teleport：// TODO



#### 响应式 API



##### 响应性基础 API

- reactive：返回对象的响应式副本
  - reactive 将解包所有深层的 refs，同时维持 ref 的响应性。
  - 当将 ref 分配给 reactive property 时，ref 将被自动解包。
- readonly：接受一个对象 (响应式或纯对象) 或 ref 并返回原始对象的只读代理。只读代理是深层的：任何被访问的嵌套 property 也是只读的
- isProxy：检查对象是否是由 reactive 或 readonly 创建的 proxy
- isReactive：检查对象是否是由 reactive 创建的响应式代理
- isReadOnly：检查对象是否是由 readonly 创建的只读代理
- toRaw：返回 reactive 或 readonly 代理的原始对象
- markRaw：标记一个对象，使其永远不会转换为 proxy。返回对象本身
- shallowReactive：创建一个响应式代理，它跟踪其自身 property 的响应性，但不执行嵌套对象的深层响应式转换，当从组合式函数返回响应式对象时，toRefs 非常有用，这样消费组件就可以在不丢失响应性的情况下对返回的对象进行解构/展开
- shallowReadonly：创建一个 proxy，使其自身的 property 为只读，但不执行嵌套对象的深度只读转换



##### Refs

- ref：接受一个内部值并返回一个响应式且可变的 ref 对象。ref 对象仅有一个 `.value` property，指向该内部值。如果将对象分配为 ref 值，则它将被 reactive 函数处理为深层的响应式对象
- unref：如果参数是一个 ref，则返回内部值，否则返回参数本身
- toRef：可以用来为原响应式对象上的某个 property 新创建一个 ref
- toRefs：将响应式对象转换为普通对象，其中结果对象的每个 property 都是指向原始对象相应 property 的 ref
- isRef：检查值是否为一个 ref 对象
- customRef
- shallowRef
- triggerRef



##### computed watch

- computed：接受一个 getter 函数，并根据 getter 的返回值返回一个不可变的响应式 ref 对象
- watchEffect：立即执行传入的一个函数，同时响应式追踪其依赖，并在其依赖变更时重新运行该函数
- watchPostEffect：
- watchSyncEffect：
- watch：watch 需要侦听特定的数据源，并在单独的回调函数中执行副作用。
  - 侦听单一源：侦听器数据源可以是一个具有返回值的 getter 函数，也可以直接是一个 ref
  - 侦听多个源：使用数组以同时侦听多个源



#### 组合式 API



##### setup

所有声明了的 prop，不管父组件是否向其传递了，都将出现在 `props` 对象中。其中未被传入的可选的 prop 的值会是 `undefined`

如果返回了渲染函数，则不能再返回其他 property。如果需要将 property 暴露给外部访问，比如通过父组件的 `ref`，可以使用 `expose`

expose 只能被调用一次。如果需要暴露多个 property，则它们必须全部包含在传递给 expose 的对象中。



##### 生命周期钩子

##### provide/inject

启用依赖注入



#### 单文件组件

##### 规范

vue文件的组成 <template>, <script>, <style>  三个标签

<script setup>

该脚本会被预处理并作为组件的 setup() 函数使用，也就是说它会在每个组件实例中执行

