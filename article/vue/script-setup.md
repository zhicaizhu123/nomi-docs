# `<script setup>`
` <script setup>`是一个编译时语法糖，用于在单文件组件(`SFC`)中使用`Composition API`。如果同时使用`SFC`和`Composition API`，则推荐使用这种语法。与常规的`<script>`语法相比，它提供了许多优点:

- 更简洁的代码，更少的样板
- 能够使用纯`TypeScript`声明`props`和触发事件
- 更好的运行时性能(`template`被编译到相同作用域的`render`函数中，没有中间代理)
- 更好的IDE类型推断性能(减少语言服务器从代码中提取类型的工作量)

## 基本语法
```vue
<script setup>
console.log('hello script setup')
</script>
```
内部代码被编译为组件的`setup()`函数的内容。这意味着，与普通的`<script>`(只在第一次导入组件时执行一次)不同，`<script setup>`中的代码将在每次创建组件的实例时执行。

**顶级绑定暴露给模板使用**
当使用`<script setup>`时，在`<script setup>`中声明的任何顶级绑定(包括变量、函数声明和导入)都可以直接在模板中使用:
```vue
<script setup>
// variable
const msg = 'Hello!'

// functions
function log() {
  console.log(msg)
}
</script>

<template>
  <button @click="log">{{ msg }}</button>
</template>
```

导入的内容也以同样的方式暴露。这意味着你可以直接在模板表达式中使用导入的`helper`函数，而不必通过`methods`选项暴露它:
```vue
<script setup>
import { capitalize } from './helpers'
</script>

<template>
  <div>{{ capitalize('hello') }}</div>
</template>
```

## 响应式
响应式状态需要使用`Reactivity API`显式地创建。类似于`setup()`函数返回的值，在模板中引用时会自动`unwrapped`:
```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)
</script>

<template>
  <button @click="count++">{{ count }}</button>
</template>
```

## 使用组件
`<script setup>`范围内的值也可以直接用作自定义组件标签:
```vue
<script setup>
import MyComponent from './MyComponent.vue'
</script>

<template>
  <MyComponent />
</template>
```

把`MyComponent`看作是一个变量。如果您使用过`JSX`，那么这里的思维模型是类似的。`kebab-case`的等效`<my-component>`也可以在`template`中使用-但是强烈建议使用`PascalCase`组件标记以保持一致性。它还有助于区别于本地定制元素。

## 动态组件
由于组件被定义为变量，而不是通过字符串注册，所以在`<script setup>`使用内部动态组件时应该使用`:is`绑定：

```vue
<script setup>
import Foo from './Foo.vue'
import Bar from './Bar.vue'
</script>

<template>
  <component :is="Foo" />
  <component :is="someCondition ? Foo : Bar" />
</template>
```
!> 注意如何将组件用作三元表达式中的变量。

## 递归组件
`SFC`可以通过文件名隐式地引用自己。例如，一个名为`FooBar`的文件。`vue`可以在它的模板中引用自己为`<FooBar/>`。  注意，它的优先级低于导入的组件。如果你有一个命名导入与组件的推断名称冲突，你可以别名导入:
 ```vue
import { FooBar as FooBarChild } from './components'
 ```

 ## 命名空间组件
 你可以使用带有`<Foo.Bar>`引用嵌套在对象属性下的组件。当你从一个文件导入多个组件时，这很有用:
 ```vue
<script setup>
import * as Form from './form-components'
</script>

<template>
  <Form.Input>
    <Form.Label>label</Form.Label>
  </Form.Input>
</template>
 ```

 ## 使用自定义指令
全局注册的自定义指令正常工作。本地自定义指令不需要显式注册`<script setup>`，但它们必须遵循命名方案`vNameOfDirective`：
```vue
<script setup>
const vMyDirective = {
  beforeMount: (el) => {
    // do something with the element
  }
}
</script>

<template>
  <h1 v-my-directive>This is a Heading</h1>
</template>
```

如果你从其他地方导入一个指令，它可以被重命名以适应所需的命名方案:
```vue
<script setup>
import { myDirective as vMyDirective } from './MyDirective.js'
</script>
```

## defineProps() 和 defineEmits()
要声明具有完全类型推断支持的`props`和`emit`等选项，我们可以使用`defineProps`和`defineEmit` API，它们在`<script setup>`中自动可用:
```vue
<script setup>
const props = defineProps({
  foo: String
})

const emit = defineEmits(['change', 'delete'])
// setup code
</script>
```

- `defineProps`和`defineEmit`是编译器宏，只能在`<script setup>`中使用。它们不需要被导入，并且在`<script setup>`被处理时被编译掉。
- `defineProps`接受与`props`选项相同的值，而`defineEmit`接受与`emits`选项相同的值。
- `defineProps`和`defineEmit`根据传递的选项提供适当的类型推断。
- 传递给`defineProps`和`defineEmit`的选项将从设置提升到模块范围。因此，选项不能引用在设置范围中声明的局部变量。这样做将导致编译错误。但是，它可以引用导入的绑定，因为它们也在模块作用域中。

## defineExpose()
默认情况下，使用`<script setup>`的组件是关闭的——也就是说，组件的公共实例(通过`refs`或`$parent`链检索)将不会公开`<script setup>`中声明的任何绑定。
 
要显式公开`<script setup>`组件中的属性，使用`defineExpose`编译器宏:

```vue
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

defineExpose({
  a,
  b
})
</script>
```

当父组件通过`refs`获得该组件的实例时，检索到的实例将具有`{ a: number, b: number }`的结构(`refs`会像普通实例一样自动展开)。

## useSlots() 和 useAttrs()
`<script setup>`中`slots`和`attrs`的使用应该相对较少，因为您可以在`template`中直接以`$slots`和`$attrs`的形式访问它们。在极少数情况下你确实需要它们，分别使用`useSlots`和`useAttrs`函数:

```vue
<script setup>
import { useSlots, useAttrs } from 'vue'

const slots = useSlots()
const attrs = useAttrs()
</script>
```
`useSlots`和`useAttrs`是实际的运行时函数，返回`setupContext.slots`和`setupContext.attrs`。它们也可以用在普通的组合API函数中。

## 在顶级使用await
顶层await可以在`<script setup>`中使用。生成的代码将被编译为`async setup()`:
```vue
<script setup>
const post = await fetch(`/api/post/1`).then((r) => r.json())
</script>
```
此外，被等待的表达式将被自动编译为在`await`之后保留当前组件实例上下文的格式。

## TypeScript-only 特性
### props/emit 声明
`Props`和`Emit`也可以使用纯类型语法来声明，方法是将一个字面类型参数传递给d`efineProps`或`defineEmit`:
```typescript
const props = defineProps<{
  foo: string
  bar?: number
}>()

const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()
```
- `defineProps`或`defineEmit`只能使用运行时声明或类型声明。同时使用两者将导致编译错误。
- 当使用类型声明时，从静态分析中自动生成等价的运行时声明，以消除双重声明的需要，并仍然确保正确的运行时行为。
  - 在开发模式下，编译器将尝试从类型推断出相应的运行时验证。例如，这里的`foo: string`是从`foo: String`类型推断出来的。如果该类型是对导入类型的引用，则推断结果将是`foo: null`(等于任何类型)，因为编译器没有外部文件的信息。
  - 在生产模式下，编译器将生成数组格式声明以减少包的大小(这里的`props`将被编译为`['foo'， 'bar']`)
  - `emit`的代码仍然是具有有效类型的`TypeScript`，可以由其他工具进一步处理。
- 到目前为止，类型声明参数必须是以下之一，以确保正确的静态分析:
  - `type`
  - 同一文件中对`interface`或`type`的引用
  - 
目前不支持复杂类型和从其他文件导入的类型。将来有可能支持类型导入。

### 默认值
纯类型`defineProps`声明的一个缺点是它没有办法为这些`props`提供默认值。为了解决这个问题，还提供了`withDefaults`编译器宏:
```typescript
export interface Props {
  msg?: string
  labels?: string[]
}
  
const props = withDefaults(defineProps<Props>(), {
  msg: 'hello',
  labels: () => ['one', 'two']
})
```
这将被编译为等效的运行时`props` `default`选项。此外，`withDefaults`帮助器为默认值提供类型检查，并确保返回的`props`类型删除了声明了默认值的属性的可选标志。

## 局限性
由于模块执行语义的不同，`<script setup>`中的代码依赖于`SFC`的上下文，当移动到外部的`.js`或`.ts`文件时，可能会导致开发人员和工具的混淆。因此，`<script setup>`不能与`src`属性一起使用。