# 基础知识
> 使用nuxt3实现服务端渲染

## 安装
1. 使用脚手架创建nuxt项目
```bash
npx nuxi init nuxt-project
```
2. 进入项目跟目录安装依赖
```bash
yarn install
```
3. 本地启动项目
```bash
yarn dev -o
```

## 配置
### nuxt.config.ts
`nuxt.config.ts`位于根目录，可以通过配置覆盖和扩展当前应用的一些行为。
```javascript
export default defineNuxtConfig({
    ...
})
```
#### 运行时环境变量和私钥
我们可以通过设置`nuxt.config.ts`中的`runtimeConfig`属性，供整个应用使用这些信息。

```javascript
export default defineNuxtConfig({
    runtimeConfig: {
        // 私钥只提供给服务端使用
        apiSecret: '123',
        // public下的信息提供给客户端使用
        public: {
          apiBase: '/api'
        }
    }
})
```
然后可以在应用中使用这些信息
```vue
<script setup>
const runtimeConfig = useRuntimeConfig()
</script>
```

### app.config.ts
位于根目录，可以通过`app.config.ts`配置覆盖和扩展构建时需要的一些信息，但是不能获取到环境变量信息。
```javascript
export default defineAppConfig({
    title: 'Hello Nomi',
    theme: {
        colors: {
            primary: '#c00'
        }
    },
    ...
})
```
然后就可以在应用中获取这些信息
```vue
<script setup>
const appConfig = useAppConfig()
</script>
```


### nuxt.config.ts 和 app.config.ts 区别
- nuxt.config.js: 在使用环境变量构建之后需要指定的私有或公共信息。
- app.config.ts： 在构建时确定的公共信息、网站配置(如主题变体、标题)和任何不敏感的项目配置

## Views
- app.vue： 入口文件
- components目录：存放全局组件，它们将在应用中自动引用，不需要显式地通过`import`引入
- pages：页面表示用于每个特定路由模式的视图。`pages`目录中的每个文件都表示显示其内容的不同路由
- layouts：布局模板目录


## 资源文件
### public
`public`目录被用作静态资产的公共服务器，这些静态资产可以在应用的一个已定义的URL上公开使用。

在应用中，我们可以直接使用`/`路径引用位于`public`目录中的文件。
```vue
<template>
<img src="/images/avatar.png" alt="用户头像" />
</template>
```
### assets
`assets`目录存放需要打包构建工具处理的资源，如样式文件，图片，字体等资源，他们会经过打包构建工具处理后再输出。

在应用中，我们可以可以使用`~/assets/`路径引用位于`assets`目录中的文件。
<template>
  <img src="~/assets/images/avatar.png" alt="用户头像" />
</template>

## 全局样式
要全局引入样式，我们可以在`nuxt.config.ts`中使用`vite`选项进行配置。
假如需要引入全局样式`color.scss`：
```css
$primary-color: #c00;
$danger-color: red;
```
配置`nuxt.config.ts`的`vite`选项：
```javascript
export default defineNuxtConfig({
    vite: {
        css: {
            preprocessorOptions: {
                scss: {
                    additionalData: '@use "@/assets/colors.scss" as *;'
                }
            }
        }
    }
})
```


## 路由
### pages
`Nuxt`的路由通能通过`vue-router`实现的，nuxt的路由表是从`pages`目录中文件自动生成。

它使用命名约定来动态创建嵌套路由，假如存在以下目录结构
```
pages/
--| about.vue
--| posts/
----| [id].vue
```
那么`Nuxt`会自动生成以下路由表
```json
{
    "routes": [
        {
            "path": "/about",
            "component": "pages/about.vue"
        },
        {
            "path": "/posts/:id",
            "component": "pages/posts/[id].vue"
        }
    ]
}
```


### 导航
使用`<NuxtLink>`组件实现，它实际渲染一个`a`标签， 当`<NuxtLink>`被渲染到页面时，`Nuxt`会自动预加载链接对应页面的组件和有信心，从而实现更快的导航。
```html
<ul>
    <li><NuxtLink to="/about">About</NuxtLink></li>
    <li><NuxtLink to="/posts/1">Post 1</NuxtLink></li>
    <li><NuxtLink to="/posts/2">Post 2</NuxtLink></li>
</ul>
```

### 路由参数
我们可以通过`useRoute()`hook函数获取当前路由信息
```javascript
const route = useRoute()

console.log(route.params.id, route,query.name)
```

### 路由中间件
- 内联中间件:（直接声明在页面中）
```vue
<script setup>
definePageMeta({
    middleware: (from, to) => {
        const auth = useState('auth')
        if (!auth.value.authenticated) {
            return navigateTo('/login')
        }
        return navigateTo('/checkout')
    },
    // 或者通过数组方式包含多个中间件
    middleware: [(from, to) => {
        const auth = useState('auth')
        if (!auth.value.authenticated) {
            return navigateTo('/login')
        }
        return navigateTo('/checkout')
    }],
})
</script>
```
- 普通中间件（声明在`middleware`目录下）
```javascript
export default defineNuxtRouteMiddleware((to, from) => {
    if (isAuthenticated() === false) {
        return navigateTo('/login')
    }
}) 
```
假设在`pages/dashboard.vue`使用该中间件
```vue
<script setup>
definePageMeta({
    middleware: 'auth',
    // 或者通过数组方式包含多个中间件
    middleware: ['auth'],
})
</script>
```

- 全局中间件（声明在`middleware`目录下，命名加载`.global`后缀）
假设存在`middleware`目录下存在日志中间件文件`log.global.ts`
```javascript
export default defineNuxtRouteMiddleware((to, from) => {
    console.log("to:"+to.path)
    console.log("from:"+from.path)
})
```
那么在每个路由都会自动执行这个全局中间件。

⚠️注意：路由中间件名称被规范化为`kebab-case`，因此`someMiddleware`变成了`some-middleware`

### 路由验证
Nuxt通过`definePageMeta`参数中的`validate`方法提供路由验证, 该属性接收的参数为route，通过返回布尔值来决定要不要渲染当前页面，如果返回false，则会触发404错误
```vue
<script setup>
definePageMeta({
  validate: async (route) => {
    const nuxtApp = useNuxtApp()
    // Check if the id is made up of digits
    return /^\d+$/.test(route.params.id)
  }
})
</script>
```

## 配置头部信息
我们可以通过头部信息配置，`useHead`，头部组件来提高`Nuxt`的SEO

### 全局配置
配置`nuxt.config.ts`的`app.head`选项

默认值：
- charset: utf-8
- viewport: width=device-width, initial-scale=1

```javascript
export default defineNuxtConfig({
    app: {
        head: {
            title: 'NOMI',
            meta: [
                { name: 'description', content: 'nomi document' }
            ],
        }
    }
})
```

### useHead配置
与其他hook一样，`useHead`只能与组件设置和生命周期钩子一起使用。
```vue
<script setup lang="ts">
useHead({
    title: 'NOMI',
    meta: [
        { name: 'description', content: 'nomi document' }
    ],
    bodyAttrs: {
        class: 'app'
    },
    script: [ { children: 'console.log(\'Hello NOMI\')' } ]
})
</script>
```

### 组件配置
`Nuxt`提供`<Title>`， `<Base>`， `<Script>`， `<NoScript>`， `<Style>`， `<Meta>`， `<Link>`， `<Body>`， `<Html>`和`<Head>`组件，我们可以在组件的`template`中直接与元数据交互。

```vue
<script setup>
const title = ref('Hello World')
</script>

<template>
<div>
    <Head>
        <Title>{{ title }}</Title>
        <Meta name="description" :content="title" />
        <Style type="text/css" children="body { background-color: green; }" />
    </Head>
    <h1>{{ title }}</h1>
</div>
</template>
```

### 特性
#### 响应式
所有属性都支持响应性，如computed、computed getter引用和reactive。
- `useHead`配置
```vue
<script setup lang="ts">
    const desc = ref('nomi document')
    useHead({
        meta: [
            { name: 'description', content: desc }
        ],
    })
</script>
```
- 组件配置
```vue
<script setup>
const desc = ref('nomi document')
</script>
 
<template>
<div>
    <Meta name="description" :content="desc" />
</div>
</template>
```

#### `titleTemplate`选项
我们可以使用`titleTemplate`选项为自定义站点标题提供一个动态模板。

`titleTemplate`可以是一个字符串，其中`%s`被替换为标题，也可以是一个函数。

```vue
<script setup lang="ts">
  useHead({
    titleTemplate: (titleChunk) => {
      return titleChunk ? `${titleChunk} - NOMI` : 'NOMI';
    },
    // 或者
    titleTemplate: `%s - NOMI`
  })
</script>
```

#### 在`body`标签元素内容后插入
- `useHead`配置
```vue
<script setup lang="ts">
useHead({
    script: [
      {
          src: 'https://third-party-script.com',
          body: true
      }
    ]
})
</script>
```
- 组件配置
```vue
<template>
<div>
    <Script src="https://third-party-script.com" body="true"></Script>
</div>
</template>
```

#### 与`definePageMeta`配合使用
在`pages`目录中，可以使用`definePageMeta`和`useHead`来基于当前路由设置元数据。

假如在某个页面配置
```vue
<script setup>
definePageMeta({
    title: 'Page Title'
})
</script>
```
然后在布局组件使用上述的配置
```vue
<script setup>
const route = useRoute()
useHead({
    meta: [{ name: 'og:title', content: `NOMI - ${route.meta.title}` }]
})
</script>
```

#### 添加样式
```vue
<script setup lang="ts">  
useHead({
  link: [
    { 
      rel: 'stylesheet', 
      href: 'https://fonts.googleapis.com/css2?family=Roboto&display=swap', 
      crossorigin: '' 
    }
  ]
})
</script>
```

## 动画
`Nuxt`利用`Vue`的`<Transition>`组件在页面和布局之间实现过渡。

### 页面动画
我们可以在`nuxt.config.ts`启用`app.pageTransition`，为所有页面切换时添加过渡效果。
```javascript
export default defineNuxtConfig({
  app: {
    pageTransition: { name: 'page', mode: 'out-in' }
  },
})
```
在`app.vue`设置过渡动画样式
```vue
<template>
  <NuxtPage />
</template>
 
<style>
.page-enter-active,
.page-leave-active {
    transition: all 0.4s;
}
.page-enter-from,
.page-leave-to {
  opacity: 0;
  filter: blur(1rem);
}
</style>
```
如果要为某些页面指定特定的动画，我们可以设置`definePageMeta`方法中的`pageTransition`进行定制
```vue
<script setup lang="ts">
definePageMeta({
  pageTransition: {
    name: 'rotate'
  }
})
</script>
```

### 布局动画
我们可以在`nuxt.config.ts`启用`app.layoutTransition`，为所有布局模版切换时添加过渡效果。
```javascript
export default defineNuxtConfig({
  app: {
    layoutTransition: { name: 'layout', mode: 'out-in' }
  },
})
```
在`app.vue`设置过渡动画样式
```vue
<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>

<style>
.layout-enter-active,
.layout-leave-active {
  transition: all 0.4s;
}
.layout-enter-from,
.layout-leave-to {
  filter: grayscale(1);
}
</style>
```
然后在切换布局模板时就会使用我们设置的动画效果进行过渡。

如果要为某些模版切换时指定特定的动画，我们可以设置`definePageMeta`方法中的`layoutTransition`进行定制
```vue
<script setup lang="ts">
definePageMeta({
  layout: 'orange',
  layoutTransition: {
    name: 'slide-in'
  }
})
</script>
```
如果我们希望在特定页面禁用过渡动画效果
```vue
<script setup lang="ts">
definePageMeta({
  pageTransition: false
  layoutTransition: false
})
</script>
```
当然也可以在`nuxt.config.ts`全局设置，让所有页面切换时禁用过渡动画效果
```javascript
defineNuxtConfig({
  app: {
    pageTransition: false,
    layoutTransition: false
  }
})
```

### 动态设置过渡效果
我们可以利用在内联中间件中根据条件设置`to.meta.pageTransition`实现不同的过渡效果。
```vue
<script setup lang="ts">
definePageMeta({
  pageTransition: {
    name: 'slide-right',
    mode: 'out-in'
  },
  middleware (to, from) {
    to.meta.pageTransition.name = +to.params.id > +from.params.id ? 'slide-left' : 'slide-right'
  }
})
</script>
 
<template>
  <h1>#{{ $route.params.id }}</h1>
</template>
 
<style>
.slide-left-enter-active,
.slide-left-leave-active,
.slide-right-enter-active,
.slide-right-leave-active {
  transition: all 0.2s;
}
.slide-left-enter-from {
  opacity: 0;
  transform: translate(50px, 0);
}
.slide-left-leave-to {
  opacity: 0;
  transform: translate(-50px, 0);
}
.slide-right-enter-from {
  opacity: 0;
  transform: translate(-50px, 0);
}
.slide-right-leave-to {
  opacity: 0;
  transform: translate(50px, 0);
}
</style>
```

### NuxtPage组件配置动画
```vue
<template>
  <div>
    <NuxtLayout>
      <NuxtPage :transition="{
        name: 'bounce',
        mode: 'out-in'
      }" />
    </NuxtLayout>
  </div>
</template>
```
!> 不能在各个页面上使用definePageMeta重写此页面转换。

## 状态管理
`Nuxt`提供了`useState`，可以跨组件创建响应性的、对`SSR`友好的共享状态。

`useState`是一个`SSR` `ref` 替换。它的值将在服务器端渲染后保存(在客户端激活期间)，并使用唯一的键在所有组件之间共享。

!>  useState只在`setup`或生命周期钩子间工作。

### 最佳实践
!> 永远不要在`<script setup>`或`setup()`函数之外定义c`onst state = ref()`。 这样的状态将被所有访问您的网站的用户共享，并可能导致内存泄漏！而是使用`const useX = () => useState('x')`。

 ### 基础使用
 ```vue
<script setup>
const counter = useState('counter', () => Math.round(Math.random() * 1000))
</script>

<template>
  <div>
    Counter: {{ counter }}
    <button @click="counter++">
      +
    </button>
    <button @click="counter--">
      -
    </button>
  </div>
</template>
 ```

### 数据共享
假设我们在`composables/states.ts`下定义了以下状态
```javascript
export const useCounter = () => useState<number>('counter', () => 0)
export const useColor = () => useState<string>('color', () => 'pink')
```
那么我们可以在组件内使用最新状态
```vue
<script setup>
const color = useColor() // 和 useState('color') 作用一样
</script>

<template>
  <p>Current color: {{ color }}</p>
</template>
```

## 错误处理
`Nuxt`是一个全栈框架，这意味着在不同的上下文中，有几种不可避免的用户运行时错误来源: 
- `Vue`渲染生命周期中的错误(`SSR` + `SPA`) 
- `API`或`Nitro`服务器生命周期中的错误 
- 服务器和客户端启动错误(`SSR` + `SPA`)

### `Vue`渲染生命周期中的错误(`SSR` + `SPA`) 
我们可以使用`onErrorCaptured`来处理异常


### `API`或`Nitro`服务器生命周期中的错误 

### 服务器和客户端启动错误(`SSR` + `SPA`)