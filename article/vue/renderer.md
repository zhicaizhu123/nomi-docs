# Vue3 设计思路

## 渲染器
**渲染器其作用是将虚拟DOM渲染成真实DOM**

### 渲染标签
假设有一下虚拟DOM
```javascript
const vnode = {
  tag: 'div',
  props: {
    onClick: () => { alert('hello vue') }
  },
  children: [{ tag: 'button', props: {}, children: 'click me' }]
}
```
- `tag`： 描述标签名称
- `props`： 描述标签的属性、事件等内容
- `children`：描述标签的子节点

那么我们可以编译一个简单的渲染器，把上面的虚拟DOM渲染成真实DOM
```javascript
function renderer(vnode, container) {
  const el = document.createElement(vnode.tag);
    for (const key in vnode.props) {
      if (/^on/.test(key)) {
        // 事件处理
        const eventName = key.substring(2).toLocaleLowerCase()
        el.addEventListener(eventName, vnode.props[key])
      }
    }

    if(typeof vnode.children === 'string') {
      // 如果是字符串，只为text节点
      el.appendChild(document.createTextNode(vnode.children))
    } else if (Array.isArray(vnode.children)) {
      // 递归调用用renderer 函数渲染子节点
      vnode.children.forEach(item => renderer(item, el))
    }

    // 将元素添加到挂载元素下
    container.appendChild(el)
}
```
- `vnode`: 虚拟DOM对象
- `container`: 一个真实的DOM元素，作为挂载点

执行`renderer`函数
```javascript
renderer(vnode, document.body)
```

我们来分析`renderer`实现思路，总体分为3步：
- 创建元素：把`vnode.tag`作为标签来创建DOM元素；
- 为元素添加属性和事件：遍历`vnode.props`对象，并处理相应的属性；
- 处理children：如果`vnode.children`是一个数组，则递归调用`renderer`，否则为文本节点处理；

### 渲染组件
那么我们如何渲染组件，在编写代码之前，我们需要弄懂什么是组件

组件其实是一组DOM元素的封装，这组DOM元素就是组件需要渲染的内容，所以我们可以定义一个函数来代表组件，而函数的返回值就是一个虚拟DOM对象
```javascript
const MyComponent = () => {
  return {
    tag: 'div',
    props: {
      onClick: () => { alert('hello vue') }
    },
    children: [{ tag: 'button', props: {}, children: 'click me' }]
  }
}
```
我们可以使用对象的`tag`属性存储组件函数
```javascript
const vnode = {
    tag: MyComponent
}
```

所以我们可以完善上面的`renderer`函数渲染器，让其支持渲染组件函数
```javascript
function renderer(vnode, container) {
  if (typeof vnode.tag === 'string') {
    // vnode 描述的是标签元素
    mountElement(vnode, container)
  } else if (typeof vnode.tag === 'function') {
    // vnode 描述的是组件
    mountComponent(vnode, container)
  }
}
```
其中我们把之前渲染标签元素的逻辑抽离到`mountElement`函数，渲染组件的逻辑抽离到`mountComponent`函数

mountElement：
```javascript
function mountElement(vnode, container) {
  const el = document.createElement(vnode.tag);
    for (const key in vnode.props) {
      if (/^on/.test(key)) {
        // 事件处理
        const eventName = key.substring(2).toLocaleLowerCase()
        el.addEventListener(eventName, vnode.props[key])
      }
    }

    if(typeof vnode.children === 'string') {
      // 如果是字符串，只为text节点
      el.appendChild(document.createTextNode(vnode.children))
    } else if (Array.isArray(vnode.children)) {
      // 递归调用用renderer 函数渲染子节点
      vnode.children.forEach(item => renderer(item, el))
    }

    // 将元素添加到挂载元素下
    container.appendChild(el)
}
```

mountComponent：
```javascript
function mountComponent(vnode, container) {
  // 获取组件需要渲染的虚拟DOM结构
  const subtree = vnode.tag()
  // 递归调用renderer渲染subtree
  renderer(subtree, container)
}
```

那么除了以上述的函数表示组件，我们还是使用一个对象表示组件，其中改函数包含一个 `render` 属性，它是一个函数，返回的是虚拟DOM对象
```javascript
const MyComponent2 = {
  render() {
    return {
      tag: 'div',
      props: {
        onClick: () => { alert('hello vue 22') }
      },
      children: [{ tag: 'button', props: {}, children: 'click me 22' }]
    }
  }
}
```

使用对象的 `tag` 属性存储对象
```javascript
const vnode = {
    tag: MyComponent2
}
```

那么我们改造下 `mountComponent` 来满足这里组件的渲染
```javascript
function mountComponent(vnode, container) {
  if (typeof vnode.tag === 'function') {
    const subtree = vnode.tag()
    renderer(subtree, container)
  } else if (typeof vnode.tag === 'object' && typeof vnode.tag.render === 'function') {
    const subtree = vnode.tag.render()
    renderer(subtree, container)
  }
}
```

`renderer` 也做相应的修改
```javascript
function renderer(vnode, container) {
  if (typeof vnode.tag === 'string') {
    // vnode 描述的是标签元素
    mountElement(vnode, container)
  } else {
    // vnode 描述的是组件
    mountComponent(vnode, container)
  }
}
```

### 模板
在说模板之前，我们先简单了解下编译器

**编译器其作用是将模板编译成渲染函数**

例如以下模板
```html
<div @click="onClick">click me</div>
```

对于编译器来说，模板就是一串普通的字符串，它会分析该字符串并生成一个功能与之相同的渲染函数
```javascript
render() {
    return h('div', { onClick: onClick }, 'click me')
}
```

用我们熟悉的.vue文件为例
```html
<template>
    <div @click="onClick">click me</div>
</template>

<script>
    export default {
        data() {/** ... */},
        methods: {
            onClick() {
                alert('hello vue')
            }
        }
    }
</script>
```
其中 `template` 标签里的内容就是模板内容，编译器会把模板内容编译成渲染函数并添加到script标签的组件对象上，所以我们最终的运行代码如下
```javascript
 export default {
    data() {/** ... */},
    methods: {
        onClick() {
            alert('hello vue')
        }
    },
    render () {
        return h('div', { onClick: onClick }, 'click me')
    }
}
```

**所以无论是使用模板还是直接手写渲染函数，对于一个组件来说，它要渲染的内容最终都是通过渲染函数产生的，然后渲染器在把渲染函数返回的虚拟DOM对象渲染成真实的DOM，这就是模板的工作原理，也是Vue.js渲染页面的流程**