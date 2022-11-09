# Vue规范

## 组件和页面文件目录

【强制】组件目录：使用大驼峰方式命名，如：`TablePagination`

【强制】页面目录：使用小驼峰方式命名，如：`systemManage`

## 组件名称

【强制】使用大驼峰方式命名，如：'SearchInput'

反例：

```typescript
export default defineComponent({
  name: 'searchInput',
  setup() {
    return {}
  },
});

```

正例：

```typescript
export default defineComponent({
  name: 'SearchInput',
  setup() {
    return {}
  },
});


```

## 组件内模块编写顺序

【强制】模板方式：template  ->  script  ->  style

【强制】JSX方式：script  ->  style

模板方式：

```html
<template>
  <div></div>
</template>

<script lang="ts">
  import { defineComponent } from 'vue';

  export default defineComponent({
    name: 'Redirect',
    setup() {
      return {};
    },
  });
</script>

<style lang="less" scoped></style>

```

JSX方式：

```html
<script lang="ts">
  import { defineComponent } from 'vue';

  export default defineComponent({
    name: 'Redirect',
    setup() {
      return (<div></div>)
    },
  });
</script>

<style lang="less" scoped></style>

```

## v-for

【强制】v-for 需要设置key属性

> key的特殊 attribute 主要用在 Vue 的虚拟 DOM 算法，在新旧 nodes 对比时辨识 VNodes。如果不使用 key，Vue 会使用一种最大限度减少动态元素并且尽可能的尝试就地修改/复用相同类型元素的算法。而使用 key 时，它会基于 key 的变化重新排列元素顺序，并且会移除 key 不存在的元素

【强制】v-for和v-if不建议一起使用

> v-for比v-if优先级高，所以使用的话，每次v-for都会执行v-if,造成不必要的计算，影响性能，尤其是当之需要渲染很小一部分的时候，解决方法的；**使用computed**

反例：

```html
<ul>
  <li v-for="user in users" v-if="user.isShow" :key="user.id">
    {{ user.name }}
  </li>
</ul>
```

正例：

```html
<div>
	<div v-for="(user,index) in activeUsers" :key="user.id" >
		{{ user.name }} 
	</div>
</div>
...
const activeUsers =  computed(() => {
  return this.users.filter((user) => {
	return user.isShow;
  })
}
```

## 样式作用域

【强制】除通用组件外，在普通组件或页面编写样式时，尽量加上`scoped`属性，避免造成全局污染，如果是需要修改特定组件下UI库组件样式，可以是使用`:deep(.className){}`方式来进行修改

例如自定义antv pagination组件的高亮分页样式，可以使用以下方式进行修改

```html
<template>
  <div class="content">
	<a-pagination v-model="current" :total="50" show-less-items />
  </div>
</template>

<script lang="ts">
  import { defineComponent } from 'vue';

  export default defineComponent({
    name: 'Redirect',
    setup() {
      return {};
    },
  });
</script>

<style lang="less" scoped>
.content {
  :deep(.ant-pagination-item-active) {
	border-color: red
  }
}
</style>

```

## hook

【强制】hook方法名使用use作为前缀，如：`useXxx`

反例：

```typescript
export search(params) {}
```

正例：

```typescript
export useSearch(params) {}
```

【参考】更多规范请参考官方[风格指南](https://cn.vuejs.org/v2/style-guide/index.html)