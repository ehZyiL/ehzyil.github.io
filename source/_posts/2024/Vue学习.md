---
title: Vue学习
author: ehzyil
tags:
  - Vue
categories:
  - 记录
date: 2024-03-11 20:22:46
headimg:
---

## Vue学习

### Vite目录 & Vue单文件组件 & npm run dev 详解

#### Vite目录

public 下面的不会被编译 可以存放静态资源

assets 下面可以存放可编译的静态资源

components 下面用来存放我们的组件

App.vue 是全局组件

main ts 全局的ts文件

index.html 非常重要的入口文件 （webpack，rollup 他们的入口文件都是enrty input 是一个js文件 而Vite 的入口文件是一个html文件，他刚开始不会编译这些js文件 只有当你用到的时候 如script src="xxxxx.js" 会发起一个请求被vite拦截这时候才会解析js文件）

vite config ts 这是vite的配置文件具体配置项 后面会详解

VsCode Vue3 插件推荐 Vue Language Features (Volar)

#### SFC 语法规范

*.vue 件都由三种类型的顶层语法块所组成：<template>、<script>、<style>

<template>
每个 *.vue 文件最多可同时包含一个顶层 <template> 块。

其中的内容会被提取出来并传递给 @vue/compiler-dom，预编译为 JavaScript 的渲染函数，并附属到导出的组件上作为其 render 选项。

<script>
每一个 *.vue 文件可以有多个 <script> 块 (不包括<script setup>)。


该脚本将作为 ES Module 来执行。

其默认导出的内容应该是 Vue 组件选项对象，它要么是一个普通的对象，要么是 defineComponent 的返回值。

<script setup>
每个 *.vue 文件最多只能有一个 <script setup> 块 (不包括常规的 <script>)

该脚本会被预处理并作为组件的 setup() 函数使用，也就是说它会在每个组件实例中执行。<script setup> 的顶层绑定会自动暴露给模板。更多详情请查看 <script setup> 文档。

<style>
一个 *.vue 文件可以包含多个 <style> 标签。

<style> 标签可以通过 scoped 或 module attribute (更多详情请查看 SFC 样式特性) 将样式封装在当前组件内。多个不同封装模式的 <style> 标签可以在同一个组件中混

#### npm run dev 详解

在我们执行这个命令的时候会去找 package json 的scripts 然后执行对应的dev命令

```
  "scripts": {
    "dev": "vite",
    "build": "run-p type-check \"build-only {@}\" --",
    "preview": "vite preview",
    "build-only": "vite build",
    "type-check": "vue-tsc --build --force"
  },
```

 **那为什么我们不直接执行**vite 命令不是更方便吗

因为在我们的电脑上面并没有配置过相关命令 所以无法直接执行

 其实在我们执行npm install 的时候（包含vite） 会在node_modules/.bin/ 创建好可执行文件

.bin 目录，这个目录不是任何一个 npm 包。目录下的文件，表示这是一个个软链接，打开文件可以看到文件顶部写着 #!/bin/sh ，表示这是一个脚本 

在我们执行npm run xxx  npm 会通过软连接 查找这个软连接存在于源码目录node_modules/vite

 所以npm run xxx 的时候，就会到 node_modules/bin中找对应的映射文件，然后再找到相应的js文件来执行

1.查找规则是先从当前项目的node_modlue /bin去找,

2.找不到去全局的node_module/bin 去找

3.再找不到 去环境变量去找

 node_modules/bin中 有三个vite文件。为什么会有三个文件呢？

```
# unix Linux macOS 系默认的可执行文件，必须输入完整文件名

vite

# windows cmd 中默认的可执行文件，当我们不添加后缀名时，自动根据 pathext 查找文件

vite

# Windows PowerShell 中可执行文件，可以跨平台

vite
```

[Vite目录 & Vue单文件组件 & npm run dev 详解](https://xiaoman.blog.csdn.net/article/details/122771007)



### 模板方法

```

```

**第一种方法（选项式 API）：**

* 使用 `data` 和 `methods` 选项来定义组件的状态和方法。

* `data` 返回一个对象，其中包含组件的状态。

* `methods` 返回一个对象，其中包含组件的方法。

  ```
  export default {
   data() {
      return {
        message: 'Welcome to Your Vue.js App'
      }
    },methods: {
      // ...methods
    }
  }
  ```

**第二种方法（Composition API 的 `setup` 函数）：**

* 使用 `setup` 函数来定义组件的状态和方法。
* `setup` 函数返回一个对象，其中包含组件的状态和方法。
* 不需要使用 `data` 或 `methods` 选项。

```
export default {
  setup(){
    const a="Hello"
    return{a}
  }
}
```

**第三种方法（Composition API 的 `<script setup>`）：**

* 使用 `<script setup>` 块来定义组件的状态和方法。

* `<script setup>` 块中的代码直接在组件模板中执行。

* 不需要使用 `setup` 函数或 `data` 和 `methods` 选项。

  ```
  <script setup lang="ts">
  const message:string='ehzyil is here'
  </script>
  ```

**主要区别：**

* **状态定义：**第一种方法使用 `data` 选项定义状态，而第二和第三种方法使用 `setup` 函数或 `<script setup>` 块。
* **方法定义：**第一种方法使用 `methods` 选项定义方法，而第二和第三种方法使用 `setup` 函数或 `<script setup>` 块中的代码。
* **语法：**第一种方法使用选项式 API 语法，而第二和第三种方法使用 Composition API 语法。
* **代码组织：**Composition API 的方法可以更好地组织和重用代码，因为它们与组件模板分离。

### Vue 指令



### Ref



### Reactive





### toRef toRefs toRaw



### computed



```vue
<template>
  <div>
    <p>Name: {{ name }}</p>
    <p>Reversed Name: {{ reversedName }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      name: 'John Doe'
    }
  },
  computed: {
    reversedName() {
      return this.name.split('').reverse().join('');
    }
  }
}
</script>
```

代码讲解

1. 模板

```html
<template>
  <div>
    <p>Name: {{ name }}</p>
    <p>Reversed Name: {{ reversedName }}</p>
  </div>
</template>
```

* 模板是 Vue.js 组件的 HTML 结构。
* 在模板中，我们使用双大括号 `{{ }}` 来插值数据。
* `name` 是一个数据属性，它在组件的 `data` 对象中定义。
* `reversedName` 是一个计算属性，它在组件的 `computed` 对象中定义。

2. 脚本

```javascript
<script>
export default {
  data() {
    return {
      name: 'John Doe'
    }
  },
  computed: {
    reversedName() {
      return this.name.split('').reverse().join('');
    }
  }
}
</script>
```

* 脚本是 Vue.js 组件的 JavaScript 代码。
* 在脚本中，我们使用 `export default` 来导出组件。
* `data` 对象用于定义组件的数据属性。
* `computed` 对象用于定义组件的计算属性。

3. 数据属性

```javascript
data() {
  return {
    name: 'John Doe'
  }
}
```

* `data` 对象是一个函数，它返回一个对象。
* 这个对象包含了组件的数据属性。
* 在这个例子中，我们定义了一个名为 `name` 的数据属性，它的值为 `John Doe`。

4. 计算属性

```javascript
computed: {
  reversedName() {
    return this.name.split('').reverse().join('');
  }
}
```

* `computed` 对象是一个对象，它包含了组件的计算属性。
* 计算属性是根据其他数据属性计算出来的属性。
* 在这个例子中，我们定义了一个名为 `reversedName` 的计算属性，它的值是 `name` 属性值的反转。

5. 渲染

当 Vue.js 组件被渲染时，它会先执行 `data` 函数来获取数据属性，然后执行 `computed` 函数来计算计算属性。最后，它会使用模板来渲染组件。

在上面的例子中，组件会被渲染成如下 HTML：

```html
<div>
  <p>Name: John Doe</p>
  <p>Reversed Name: eoD nhoJ</p>
</div>
```



#### computed购物车案例

```
<template>
  <div>
    <input placeholder="请输入关键字" type="text" v-model="keyWord">
    <table style="margin-top:10px;" width="500" cellspacing="0" cellpadding="0" border>
      <thead>
        <tr>
          <th>物品</th>
          <th>单价</th>
          <th>数量</th>
          <th>总价</th>
          <th>操作</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in searchData">
          <td align="center">{{ item.name }}</td>
          <td align="center">{{ item.price }}</td>
          <td align="center">
            <button @click="item.num > 0 ? item.num-- : null">-</button>
            {{ item.num }}
            <button @click="item.num++">+</button>
          </td>
          <td align="center">{{ item.num * item.price }}</td>
          <td align="center"><button @click="del(index)">删除</button></td>

        </tr>
      </tbody>
      <tfoot>
        <tr>
          <td colspan="5" align="right">
            <span>总价：{{ total }}</span>
          </td>

        </tr>
      </tfoot>
    </table>

  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from 'vue'
let keyWord = ref<string>('')
interface data {
  name: string
  price: number
  num: number
}
const data = reactive<data[]>([
  {
    name: "小满的绿帽子",
    price: 100,
    num: 1,
  },
  {
    name: "小满的红衣服",
    price: 200,
    num: 1,
  },
  {
    name: "小满的黑袜子",
    price: 300,
    num: 1,
  }

])

let searchData = computed(() => {
  return data.filter(item => item.name.includes(keyWord.value))
})


let total = computed(() => {
  return searchData.value.reduce((pre, cur) => {
    console.log('pre', pre, 'cur', cur)
    return pre + cur.num * cur.price
  }, 0)
})

const del = (index: number) => {
  console.log(index)
  data.splice(index, 1)
}


</script>

```



### watch侦听器

watch 需要侦听特定的数据源，并在单独的回调函数中执行副作用

watch第一个参数监听源

watch第二个参数回调函数cb（newVal,oldVal）

watch第三个参数一个options配置项是一个对象{

immediate:true //是否立即调用一次

deep:true //是否开启深度监听



监听Ref 案例

```
import { ref, watch } from 'vue'

let message = ref({
    nav:{
        bar:{
            name:""
        }
    }
})

watch(message, (newVal, oldVal) => {
    console.log('新的值----', newVal);
    console.log('旧的值----', oldVal);
},{
    immediate:true,
    deep:true
})
```

监听多个ref 注意变成数组啦

```
import { ref, watch ,reactive} from 'vue'

let message = ref('')
let message2 = ref('')

watch([message,message2], (newVal, oldVal) => {
    console.log('新的值----', newVal);
    console.log('旧的值----', oldVal);
})
```

监听Reactive

使用reactive监听深层对象开启和不开启deep 效果一样

```
import { ref, watch ,reactive} from 'vue'

let message = reactive({
    nav:{
        bar:{
            name:""
        }
    }
})

watch(message, (newVal, oldVal) => {
    console.log('新的值----', newVal);
    console.log('旧的值----', oldVal);
})
```

案例2 监听reactive 单一值

```
import { ref, watch ,reactive} from 'vue'

let message = reactive({
    name:"",
    name2:""
})

watch(()=>message.name, (newVal, oldVal) => {
    console.log('新的值----', newVal);
    console.log('旧的值----', oldVal);
})
```

### watchEffect高级侦听器



**Vue 3 中 watchEffect 函数**

watchEffect 函数是一个响应式函数，它会在其依赖项发生变化时执行。

**语法**

```javascript
watchEffect(() => {
  // 回调函数
})
```

**参数**

* 回调函数：一个将在依赖项发生变化时执行的函数。

**返回值**

* 取消监视函数：一个可以用来停止监视依赖项的函数。

**示例**

```javascript
import { watchEffect } from 'vue'

const count = ref(0)

watchEffect(() => {
  console.log(`count is now ${count.value}`)
})

count.value++
```

在上面的示例中，我们使用 watchEffect 函数来监视 `count` 变量的变化。当 `count` 变量发生变化时，watchEffect 函数中的回调函数就会执行，并会将 `count` 变量的值打印到控制台。

**依赖项**

watchEffect 函数的依赖项是指在回调函数中使用的任何响应式变量。在上面的示例中，`count` 变量是 watchEffect 函数的依赖项。

**取消监视**

要取消监视依赖项，可以使用 watchEffect 函数返回的取消监视函数。在上面的示例中，我们可以使用以下代码来取消监视 `count` 变量：

```javascript
const stopWatching = watchEffect(() => {
  console.log(`count is now ${count.value}`)
})

stopWatching()
```

**何时使用 watchEffect 函数**

watchEffect 函数可以用来监视任何响应式变量的变化。它通常用于在响应式变量发生变化时执行一些操作，例如更新 UI 或执行异步操作。

**与 computed 函数的区别**

watchEffect 函数与 computed 函数非常相似。它们都是响应式函数，都会在依赖项发生变化时执行。但是，watchEffect 函数和 computed 函数之间存在一些关键的区别：

* watchEffect 函数的回调函数会在每次依赖项发生变化时执行，而 computed 函数的回调函数只会在依赖项发生变化时执行一次。
* watchEffect 函数没有返回值，而 computed 函数有返回值。
* watchEffect 函数可以用来执行异步操作，而 computed 函数不能。

**结论**

watchEffect 函数是一个强大的工具，可以用来监视任何响应式变量的变化。它通常用于在响应式变量发生变化时执行一些操作，例如更新 UI 或执行异步操作。

**其他示例**

* 使用 watchEffect 函数来更新 UI：

```javascript
import { watchEffect } from 'vue'

const message = ref('Hello, world!')

watchEffect(() => {
  document.getElementById('app').innerHTML = message.value
})

message.value = 'Hello, Vue!'
```

* 使用 watchEffect 函数来执行异步操作：

```javascript
import { watchEffect } from 'vue'

const count = ref(0)

watchEffect(async () => {
  const data = await fetch('https://example.com/api/data')
  console.log(data)
})

count.value++
```

**注意**

* watchEffect 函数是 Vue 3 中的新特性。在 Vue 2 中，可以使用 `watch` 选项来监视响应式变量的变化。
* watchEffect 函数是异步的。这意味着它不会立即执行。相反，它会在下一次事件循环中执行。
* watchEffect 函数可以用来监视多个响应式变量。只需在回调函数中使用这些变量即可。

### 





### **Vue 3 生命周期**

**1. 创建阶段**

- `beforeCreate`：在实例创建之前调用。
- `created`：在实例创建之后调用。
- `beforeMount`：在实例挂载之前调用。

**2. 挂载阶段**

- `mounted`：在实例挂载之后调用。

**3. 更新阶段**

- `beforeUpdate`：在实例更新之前调用。
- `updated`：在实例更新之后调用。

**4. 卸载阶段**

- `beforeDestroy`：在实例销毁之前调用。
- `destroyed`：在实例销毁之后调用。

**5. 错误处理阶段**

- `errorCaptured`：在捕获错误时调用。

**6. 其他生命周期钩子**

- `activated`：在组件激活时调用。
- `deactivated`：在组件停用时调用。
- `renderTriggered`：在重新渲染触发时调用。
- `renderTracked`：在重新渲染被跟踪时调用。

**示例**

```javascript
<template>
  <div v-if="showComponent" style="background-color: bisque;">
    <span @click="num++" v-if="num >= 0 && num < 3">{{ num }}</span>

  </div>
  <button @click="change">button</button>
</template>

<script setup lang="ts">

import { ref, onBeforeMount, onMounted, onBeforeUpdate, onUpdated, onBeforeUnmount, onUnmounted, before } from 'vue'
var num = ref(0)
let showComponent = ref(true);
const change = () => {
  showComponent.value = !showComponent.value;
}
onBeforeMount(() => {
  console.log(`the component is to mounted.`)
})
onMounted(() => {
  console.log(`the component is now mounted.`)
})

onBeforeUpdate(() => {
  console.log(`the component is about to update.`)
})
onUpdated(() => {
  console.log(`the component is now updated.`)
})
onBeforeUnmount(() => {
  console.log(`the component is to unmounted.`)
})
onUnmounted(() => {
  console.log(`the component is now unmounted.`)
})

</script>
   
```

**输出**

```

```



### **组件**

代码组件是 Vue 中可重用的组件，它们可以包含自己的模板、脚本和样式。

每一个.vue 文件呢都可以充当组件来使用

每一个组件都可以复用

**代码示例**

```vue
<template>
  <HelloWorld></HelloWorld>
</template>

<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue';

</script>

```



### bem架构

他是一种css架构 oocss 实现的一种 （面向对象css） ，BEM实际上是block、element、modifier的缩写，分别为块层、元素层、修饰符层，element UI 也使用的是这种架构

BEM 命名约定的模式是：

```
.block {}

.block__element {}

.block--modifier {}
```

#### 编写bem架构

bem.scss

```
$block-sel: "-" !default;
$element-sel: "__" !default;
$modifier-sel: "--" !default;
$namespace:'xm' !default;
@mixin bfc {
    height: 100%;
    overflow: hidden;
}
 
//混入
@mixin b($block) {
   $B: $namespace + $block-sel + $block; //变量
   .#{$B}{ //插值语法#{}
     @content; //内容替换
   }
}
 
@mixin flex {
    display: flex;
}
 
@mixin e($element) {
    $selector:&;
    @at-root {
        #{$selector + $element-sel + $element} {
            @content;
        }
    }
}
 
@mixin m($modifier) {
    $selector:&;
    @at-root {
        #{$selector + $modifier-sel + $modifier} {
            @content;
        }
    }
}
```

全局扩充sass:

vite.config.ts

```
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
 
// https://vitejs.dev/config/
export default defineConfig({
    plugins: [vue()],
    css: {
        preprocessorOptions: {
            scss: {
                additionalData: "@import './src/bem.scss';"
            }
        }
    }
})
```

Vue 组件用法

app.vue

```
<template>
  <div>
    <div class="xm-test">ehzyil-test
      <br>
      <div class="xm-test__inner">ehzyil-test__inner</div>
      <dir class="xm-test--super">super</dir>
    </div>
  </div>

  <Layout></Layout>
</template>

<script lang="ts" setup>
import Layout from './Layout/index.vue'

</script>

<style  lang="scss">
@include b(test) {
  background-color: red;

  @include e(inner) {
    background-color: rgba(0, 34, 255, 0.37);
  }

  @include m(super) {
    background-color: yellow;
  }

  @include flex;
}

#app {
  @include bfc;
}
</style>
```

#### bem架构实现Layout布局

 文件树：在app.vue中引入Layout.vue

```
├─.vscode
├─public
└─src
    ├─assets
    ├─components
    └─Layout
        ├─Content
        ├─Header
        └─Menu
        
```



在index.html中添加样式

```
 <style>
      * {
        margin: 0;
        padding: 0;
      }
      html, body{
        height: 100%;
     overflow: hidden;
      }
    </style>
```

App.vue

```
<template>
  <Layout></Layout>
</template>

<script lang="ts" setup>
import Layout from './Layout/index.vue'

</script>

<style  lang="scss">
#app {
  @include bfc;
}
</style>
```

Layout/index.vue

```
<template>
    <div class="xm-wraps">
        <div>
            <Menu></Menu>
        </div>
        <div class="xm-wraps__right">
            <Header></Header>
            <Content></Content>
        </div>
    </div>
</template>
 
<script lang="ts" setup>
import { ref, reactive } from "vue"
import Menu from './Menu/index.vue'
import Content from './Content/index.vue'
import Header from './Header/index.vue'
</script>
 
<style lang="scss" scoped>
@include b('wraps') {
    @include bfc;
    @include flex;

    @include e(right) {
        flex: 1;
        display: flex;
        flex-direction: column;
    }
}
</style>
```



/Menu/index.vue

```
<template>
    <div class="xm-menu">
        menu
    </div>
</template>
 
<script lang="ts" setup>
import type { ref, reactive } from 'vue';

</script>
 
<style scoped lang="scss">
@include b(menu) {
    min-width: 200px;
    border-right: 1px solid #e6e6e6;
    height: 100%;

}
</style>
```

/Content/index.vue

```
<template>
    <div class="xm-content">
        Content
        <div class="xm-content__items" c- v-for="(item, index) in 50" :key="index">
            {{ index }}
        </div>
    </div>
</template>
 
 
<script lang="ts" setup>
import type { ref, reactive } from 'vue';

</script>
 
<style scoped lang="scss">
@include b(content) {
    flex: 1;
    overflow: auto;

    @include e(items) {
        padding: 10px;
        border: solid;
        margin: 10px;
        border-radius: 10px;
    }
}
</style>
```

/Header/index.vue

```
<template>
    <div class="xm-header">
        header
    </div>
</template>
 
 
<script lang="ts" setup>
import type { ref, reactive } from 'vue';

</script>
 
<style scoped lang="scss">
@include b(header) {
    height: 50px;
    border-bottom: 1px solid red;
}
</style>
```



### 父子组件传参

#### 父传值

字符串类型是不需要v-bind/:

```
<Menu  title="我是标题"></Menu>
```

传递非字符串类型需要加v-bind/简写 冒号

```
<template>
    <div class="layout">
        <Menu v-bind:data="data"  title="我是标题"></Menu>
        <div class="layout-right">
            <Header></Header>
            <Content></Content>
        </div>
    </div>
</template>
 
<script setup lang="ts">
import Menu from './Menu/index.vue'
import Header from './Header/index.vue'
import Content from './Content/index.vue'
import { reactive } from 'vue';
 
const data = reactive<number[]>([1, 2, 3])
</script>
```

若非字符串类型不加v-bind则**传递的字符串**

```
<Menu title="这是title" :arr="data" arr1=data></Menu>
```

#### 子组件接受值

通过defineProps 来接受 defineProps是无须引入的直接使用即可

如果我们使用的TypeScript,可以使用传递字面量类型的纯类型语法做为参数

```
const values = defineProps<{
    title: string
    arr: number[]
    arr1: number[]
}>();
```

如果你使用的不是TS

```
defineProps({
    title: {
        type: String,
        default: 'menu'
    },
    arr: {
        type: Array,
        default: () => [1, 2, 3]
    },
    arr1: {
        type: Array,
        default: () => [1, 2, 3]
    }
})
```

TS 特有的默认值方式

```
type props = {
    title?: string
    arr?: number[]
    arr1?: number[]
    arr2?: number[]
}
withDefaults(defineProps<props>(), {
    //设置默认值
    title: 'menu',
    arr: () => [1, 2, 3],
    arr1: () => [1, 2, 3],
    arr2: () => [1, 2, 3]
})
```

#### 代码

index.vue

```
<template>
    <div class="xm-wraps">
        <div>
            <Menu title="这是title" :arr="data" arr1=data></Menu>
        </div>
        <div class="xm-wraps__right">
            <Header></Header>
            <Content></Content>
        </div>
    </div>
</template>
 
<script lang="ts" setup>
import { ref, reactive } from "vue"
import Menu from './Menu/index.vue'
import Content from './Content/index.vue'
import Header from './Header/index.vue'

var data = [1, 2, 3];

</script>
 
<style lang="scss" scoped>
@include b('wraps') {
    @include bfc;
    @include flex;

    @include e(right) {
        flex: 1;
        display: flex;
        flex-direction: column;
    }
}
</style>
```



Menu/index.vue

```
<template>
    <div class="xm-menu">
        menu
        <hr>
        {{ title }}
        <hr>
        {{ values }}
        <hr>
        {{ arr }}
        <hr>
        {{ arr1 }}
        <hr>
        {{ arr2 }}
        <hr>
    </div>
</template>
 
<script lang="ts" setup>
import type { ref, reactive } from 'vue';

// const values = defineProps<{
//     title: string
//     arr: number[]
//     arr1: number[]
// }>();

// defineProps({
//     title: {
//         type: String,
//         default: 'menu'
//     },
//     arr: {
//         type: Array,
//         default: () => [1, 2, 3]
//     },
//     arr1: {
//         type: Array,
//         default: () => [1, 2, 3]
//     }
// })

type props = {
    title?: string
    arr?: number[]
    arr1?: number[]
    arr2?: number[]
}
withDefaults(defineProps<props>(), {
    //设置默认值
    title: 'menu',
    arr: () => [1, 2, 3],
    arr1: () => [1, 2, 3],
    arr2: () => [1, 2, 3]
})

const fun = () => {
    // console.log(title)  //找不到名称“title”。
    console.log(values.title)
}

</script>
 
<style scoped lang="scss">
@include b(menu) {
    min-width: 200px;
    border-right: 1px solid #e6e6e6;
    height: 100%;
}
</style>
```



#### 子组件给父组件传参

是通过defineEmits派发一个事件

```
<template>
    <div class="menu">
        {{ title }}
        <hr>
        <!-- 3.触发事件 -->
        <button @click="clickTap">派发给父组件</button>
    </div>
</template>
 
<script setup lang="ts">
import { reactive } from 'vue'
defineProps({
    title: {
        type: String,
        default: '啥都没传'
    },

})

const list = reactive<number[]>([4, 5, 6])
// 1.定义一个名为 on-click 的自定义事件。
// const emit = defineEmits(['on-click'])

// 如果用了ts可以这样
const emit = defineEmits<{
    (e: 'on-click', list: number[]): void
}>()

// 2.在 clickTap 方法中，使用 emit 方法派发 on-click 事件，并传递 list 数组作为参数
const clickTap = () => {
    emit('on-click', list)
}

</script>
```

我们在子组件绑定了一个click 事件 然后通过defineEmits 注册了一个自定义事件

点击click 触发 emit 去调用我们注册的事件 然后传递参数

#### 父组件接受子组件的事件

```
<template>
    <div class="xm-wraps">
        <div>
            <Menu @on-click="handleClick" title="Menu"></Menu>
        </div>
        <div class="xm-wraps__right">
            <Header></Header>
            <Content></Content>
        </div>
    </div>
</template>
 
<script lang="ts" setup>
import { ref, reactive } from "vue"
import Menu from './Menu/index.vue'
import Content from './Content/index.vue'
import Header from './Header/index.vue'

var data = [1, 2, 3];
const handleClick = (list: number[]) => {
    console.log(list, '父组件接受子组件');
}
</script>
 
<style lang="scss" scoped>
@include b('wraps') {
    @include bfc;
    @include flex;

    @include e(right) {
        flex: 1;
        display: flex;
        flex-direction: column;
    }
}
</style>
```

我们从Menu 组件接受子组件派发的事件on-click 后面是我们自己定义的函数名称handleClick

会把参数返回过来

#### 子组件暴露给父组件内部属性

子组件使用defineExpose暴露属性或方法

```
<script setup lang="ts">
import { reactive } from 'vue'

const aa = reactive({ "aa": "1" })

const list = reactive<number[]>([4, 5, 6])

const clickTap = () => {
 
}

defineExpose({
    clickTap, list, aa,
})
</script>
```

父组件使用`const MenuVal = ref<InstanceType<typeof Menu>>();`接受

```
<script lang="ts" setup>
import { ref, reactive } from "vue"
import Menu from './Menu/index.vue'

const handleClick = (list: number[]) => {
    console.log(list, '父组件接受子组件');
    console.log(MenuVal.value?.aa);
    console.log(MenuVal.value?.list);
}
const MenuVal = ref<InstanceType<typeof Menu>>();
</script>
```

### 组件

vite-demo\src\components\Card\index.vue

```
<template>
    <div class="card">
        <div class="card-header">
            <div>标题</div>
            <div>副标题</div>
        </div>
        <div v-if='content' class="card-content">
            {{ content }}
        </div>
    </div>
</template>
   
<script setup lang="ts">
type Props = {
    content: string
}
defineProps<Props>()

</script>
   
<style scoped lang='less'>
@border: #ccc;

.card {
    width: 300px;
    border: 1px solid @border;
    border-radius: 3px;

    &:hover {
        box-shadow: 0 0 10px @border;
    }

    &-content {
        padding: 10px;
    }

    &-header {
        display: flex;
        justify-content: space-between;
        padding: 10px;
        border-bottom: 1px solid @border;
    }
}</style>
```



#### 全局组件

**组件注册方法**

在main.ts 引入我们的组件跟随在createApp(App) 后面 切记不能放到mount 后面这是一个链式调用用

其次调用 component 第一个参数组件名称 第二个参数组件实例

```typescript
import './assets/main.css'

import { createApp } from 'vue'
import App from './App.vue'
import Card from './components/Card/index.vue'

createApp(App).component('Card', Card).mount('#app')

```

使用方法

直接在其他vue页面 立即使用即可 无需引入

```
<template>
    <div class="xm-content">
        Content
        <div class="xm-content__items">
            <Card content="这是一个卡片"></Card>
        </div>
    </div>
</template>
 
```

**批量注册组件**

例如：

需要从 @element-plus/icons-vue 中导入所有图标并进行全局注册。

 main.ts

```

import * as ElementPlusIconsVue from '@element-plus/icons-vue'

const app = createApp(App)
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
  app.component(key, component)
}
```



#### 局部组件

就是在一个组件内（A） 通过import 去引入别的组件(B) 称之为局部组件

应为B组件只能在A组件内使用 所以是局部组件

如果C组件想用B组件 就需要C组件也手动import 引入 B 组件

例子：如`父组件接受子组件的事件`



#### 递归组件

组件定义名称的方式：

1.在增加一个script 通过 export 添加name

```
<script lang="ts">
export default {
  name:"TreeItem"
}
</script>
```

2.直接使用文件名当组件名

3.使用插件 

[vue-macros](https://github.com/vue-macros/vue-macros)



**案例递归树**

在父组件配置数据结构 数组对象格式 传给子组件

```cobol
type TreeList = {
  name: string;
  icon?: string;
  children?: TreeList[] | [];
};
const data = reactive<TreeList[]>([
  {
    name: "no.1",
    children: [
      {
        name: "no.1-1",
        children: [
          {
            name: "no.1-1-1",
          },
        ],
      },
    ],
  },
  {
    name: "no.2",
    children: [
      {
        name: "no.2-1",
      },
    ],
  },
  {
    name: "no.3",
  },
]);
```

如：

```
<template>
  <Tree :data="data"></Tree>
</template>

<script lang="ts" setup>
import { reactive } from 'vue';
import Layout from './Layout/index.vue'
import Tree from './components/Tree/Tree.vue';

type TreeList = {
  name: string;
  icon?: string;
  children?: TreeList[] | [];
};
const data = reactive<TreeList[]>([
  {
    name: "no.1",
    children: [
      {
        name: "no.1-1",
        children: [
          {
            name: "no.1-1-1",
          },
        ],
      },
    ],
  },
  {
    name: "no.2",
    children: [
      {
        name: "no.2-1",
      },
    ],
  },
  {
    name: "no.3",
  },
]);

</script>

```



子组件接收值

```
type TreeList = {
  name: string;
  icon?: string;
  children?: TreeList[] | [];
};
 
type Props<T> = {
  data?: T[] | [];
};
 
defineProps<Props<TreeList>>();
const clickItem = (item: TreeList) => {
  console.log(item)
}
```

如：

```
<template>
    <div style="margin-left:10px;" class="tree">
        <div :key="index" v-for="(item, index) in data">
            <div @click.stop='clickItem(item)'>
                {{ item.name }}
            </div>
            <TreeItem @click.stop='clickItem(item)' v-if='item?.children?.length' :data="item.children"></TreeItem>
        </div>
    </div>
</template>
<script lang="ts" setup>
import type { ref, reactive } from 'vue';
type TreeList = {
    name: string;
    icon?: string;
    children?: TreeList[] | [];
};

type Props<T> = {
    data?: T[] | T;
}
defineProps<Props<TreeList>>();

const clickItem = (item: TreeList) => {
    console.log(item);
}
</script>

<script lang="ts">
export default {
    name: "TreeItem"
}
</script>

<style scoped lang="scss"></style>
```

页面展示如下：

```
no.1
	no.1-1
		no.1-1-1
no.2
	no.2-1
no.3
```

### 动态组件

动态组件允许你根据一个动态值来渲染不同的组件。这对于创建可重用且灵活的组件非常有用。

要创建动态组件，可以使用 `is` 属性：

```html
<component :is="value"></component>
```

其中 `componentName` 是一个动态值，它将被求值为一个组件名称。

例如，以下代码将根据 `currentComponent` 的值渲染不同的组件：

```html
<template>
 <div class="box" v-for="(item, index) in componentData">
    <div class="tabs" :class="[index == active ? 'active' : '']" @click="value = item.com"> {{ item.name }}
    </div>
  </div>
  <hr>
  <component :is="value"></component>
</template>

<script lang="ts" setup>
import { ref, reactive } from 'vue';
import Layout from './Layout/index.vue'
import V1 from './components/V1.vue'
import V2 from './components/V2.vue'

const value = ref(V1)
const active = ref(0)
var componentData = reactive([{
  name: 'v1',
  com: V1
},
{
  name: 'v2',
  com: V2
}])

const change = () => {

}
</script>

<style  lang="less">
//并列
.box {
  display: inline-block;
}

.active {
  background-color: blue;
}

.tabs {
  border: 1px solid;
  padding: 10px 10px;
  margin: 20px;
  background-color: red;
}
</style>
```

### 插槽slot



#### 匿名插槽

1.在子组件放置一个插槽

```
<template>
    <div>
        <slot></slot>
    </div>
</template>
```

2.在父组件给这个插槽填充内容

```
<template >
            <Header></Header>
            <Content>
            
                <template v-slot>
                    <p>这是子组件的内容</p>
                </template>
              
            </Content>
</template>
```

#### 具名插槽

具名插槽其实就是给插槽取个名字。一个子组件可以放多个插槽，而且可以放在不同的地方，而父组件填充内容时，可以根据这个名字把内容填充到对应插槽中

1.在子组件放置一个插槽

```cobol
<template>
    <div>
        <slot name="c"></slot>
    </div>
</template>
```

2.在父组件给这个插槽填充内容

```
<template >
    <Header></Header>
    <Content>

        <template v-slot:c>
            <p>这是子组件的内容</p>
        </template>

    </Content>
</template>
```

插槽简写

```
<template >
    <Header></Header>
    <Content>

        <template #c>
            <p>这是子组件的内容</p>
        </template>

    </Content>
</template>
```

#### 作用域插槽

在子组件动态绑定参数派发给父组件的slot去使用

```
<template>
    <div>
        <div v-for="item in 100">
            <slot :data="item"></slot>
        </div>
    </div>
</template>
```

子组件

```
        <template v-slot="data">
            <p>{{ data }}</p>
        </template>

```

#### 动态组件



子组件：

```
<template>
    <div>
        <slot name="c"></slot>
    </div>
</template>
```

父组件：

```
<template >
    <Header></Header>
    <Content>
        <template #[val]>
            <p>这是子组件的内容</p>
        </template>
    </Content>
</template>
 
<script lang="ts" setup>
import { ref, reactive } from "vue"
import Menu from './Menu/index.vue'
import Content from './Content/index.vue'
import Header from './Header/index.vue'
const val = ref('c')
</script>
```

