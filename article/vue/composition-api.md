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
近年来，越来越多的前端开发人员开始采用`TypeScript`，因为它可以帮助我们编写更健壮的代码，更有信心地进行更改，并在IDE支持下提供出色的开发体验。然而，最初构思于2013年的`Options API`在设计时并没有考虑到类型推断。我们必须实现一些非常复杂的类型体操，才能使类型推断与`Options API`一起工作。即使做了这些工作，`Options API`的类型推断仍然可以分解为`mixin`和`依赖注入`。

这导致许多想要使用`Vue`和`TypeScript`的开发人员倾向于使用`vue-class-component`支持的`class API`。然而，基于class的API严重依赖`ES装饰器`，这一语言特性在2019年`Vue3`开发时只是第二阶段的提议。我们觉得把官方API建立在一个不稳定的提案上太冒险了。从那以后，装修器的提案又经历了一次彻底的修改，最终在2022年进入了第三阶段。此外，与`Options API`类似，基于class的API也存在逻辑重用和组织限制。

相比之下，`Composition API`主要使用简单的变量和函数，它们自然是类型友好的。用`Composition API`编写的代码可以享受完全的类型推断，几乎不需要手动类型提示。大多数时候，`Composition API`代码在`TypeScript`和普通`JavaScrip`t中看起来基本相同。这也使得普通`JavaScript`用户可以从部分类型推断中受益。

### 更小的包体积和更少的开销
用`Composition API`和`<script setup>`编写的代码也比`Options API`更高效、更精简。这是因为`<script setup>`组件中的`template`被编译为内联在`<script setup>`代码相同作用域中的函数。与使用`this`访问属性不同，编译后的模板代码可以直接访问`<script setup>`中声明的变量，中间不需要实例代理。这也导致了更好的缩小，因为所有的变量名都可以安全地缩短。

## 与Options API的关系
### 权衡
一些从`Options API`迁移过来的用户发现他们的`Composition API`代码组织更差，并得出结论说`Composition API`在代码组织方面“更糟糕”。我们建议有这种意见的用户从不同的角度来看待这个问题。

`Composition API`不再提供引导你将代码放入各自桶中的“guard rails”。您可以像编写普通`JavaScript`一样编写组件代码。这意味着你可以也应该将任何代码组织的最佳实践应用到你的`Composition API`代码中，就像你在编写普通`JavaScript`时一样。如果您能够编写组织良好的`JavaScript`，那么您也应该能够编写组织良好的`Composition API`代码。

在编写组件代码时，`Options API`确实允许您“想得更少”，这就是许多用户喜欢它的原因。然而，为了减少精神开销，它还将您锁定在规定的代码组织模式中，这可能会使重构或提高大型项目中的代码质量变得困难。在这方面，`Composition API`提供了更好的长期可伸缩性。

## Composition API是否涵盖了所有的用例?
是的，就有状态逻辑而言。在使用`Composition API`时，可能只需要几个选项：`props`、`emit`、`name`和`inheritAttrs`。如果使用`<script setup>`，那么`inheritAttrs`通常是需要单独的`<script>`块的选项。

如果您打算专门使用`Composition API`(以及上面列出的选项)，您可以通过一个编译时标志从`Vue`中删除`Options API`相关代码，从而从您的生产包中减少一些体积。注意，这也会影响依赖项中的Vue组件。