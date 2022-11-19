# Composition API
## 什么Composition API
- 响应式API：例如 `ref()`和`reactive()`，允许我们直接创建响应式属性、计算属性和watcher。
- 生命周期钩子函数： 例如`onMounted()`和`onUnmounted()`，允许我们以编程方式挂钩到组件生命周期
- 依赖注入：`provide()` 和 `inject()`，允许我们利用`Vue`的依赖注入系统，同时使用响应式API.

以下是使用Composition API编写组件的基础版本
```vue
<script setup>
import { ref, onMounted } from 'vue'

// reactive state
const count = ref(0)

// functions that mutate state and trigger updates
function increment() {
  count.value++
}

// lifecycle hooks
onMounted(() => {
  console.log(`The initial count is ${count.value}.`)
})
</script>
  
<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

## 为什么要使用Composition API
### 更好的逻辑复用
`Composition API`的主要优点是它支持以可组合函数的形式进行简洁、高效的逻辑复用。

 组合API的逻辑重用能力产生了很好的社区项目，如VueUse。它还可以作为一种干净的机制，方便地将有状态的第三方服务或库集成到Vue的响应式系统中。

### 更加灵活的代码组织
许多用户喜欢我们在默认情况下使用`Options API`编写有组织的代码：所有内容都有自己的位置，这取决于它所处的选项。然而，当单个组件的逻辑增长超过一定的复杂性阈值时，`Options API`会带来严重的限制。这种限制在需要处理多个逻辑问题的组件中尤其突出，我们已经在许多生产Vue 2应用程序中亲眼目睹了这一点。  以Vue CLI的GUI中的文件夹管理器组件为例：该组件负责以下逻辑问题:
- 跟踪当前文件夹状态并显示其内容
- 处理文件夹导航(打开、关闭、刷新…)
- 处理新文件夹的创建
- 正在切换仅显示收藏文件夹
- 切换显示隐藏文件夹
- 处理当前工作目录的更改

### 更好的类型推断

### 更小的包体积和更少的开销