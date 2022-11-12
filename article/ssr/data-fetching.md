# 数据请求

## 方式
### useFetch
> 一般情况，组件和插件可以使用useFetch从任何URL进行通用的获取
```html
<script setup>
  const { data } = await useFetch('/api/getData')
</script>
<template>
  title: {{ data.title }}
</template>
```

- useLazyFetch
- useAsyncData
- useLazyAsyncData


## 刷新数据
- refreshNuxtData
- clearNuxtData

## 最佳实践