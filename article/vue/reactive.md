# 响应式系统的作用和实现
## 响应式数据和副作用函数
副作用函数是的是会产生副作用的函数，如：
```javascript
function effect() {
    document.body.innerText = 'hello vue'
}
```
当 `effect` 执行时，它会修改body的文本内容，但除了 `effect` 外的任何函数都可以读取和设置body的文本内容，所以 `effect` 的执行会直接或者间接的影响其他函数的执行，这个时候我们就认为 `effect` 产生了副作用。

那什么时响应式数据，假如我们在一个副函数中读取某个对象的属性
```javascript
const obj = { text: 'hello vue' }
effect() {
    document.body.innerText = obj.text
}
```
当改变 `obj.text` 的值时，我们希望副作用函数会自动执行，如果能够实现这个目标，那么对象 `obj` 就是一个响应式数据。

## 响应式数据的实现
如何实现响应式数据呢？我们可以发现两点线索：
- 当副作用函数`effect`执行时，会触发`obj.text`的**读取**操作;
- 当修改`obj.text`时，会触发字段`obj.text`的**设置**操作。

如果能够拦截`obj.text`的读取和设置操作，事情就变得简单。
- 当读取`obj.text`时，我们把副作用函数存储在一个桶里；
- 当设置`obj.text`时，我们把桶存储的副作用函数取出并执行。
  
我们可以使用Proxy来实现我们上面的思路
```javascript
// 存储副作用函数的桶
const bucket = new Set()

// 原始数据
const data = { text: 'hello vue' }

// 对原始数据的代理
const obj = new Proxy(data, {
  get(target, key) {
    // 读取操作时将副作用添加到桶里
    bucket.add(effect)
    return target[key]
  },

  set(target, key, value) {
    target[key] = value
    // 设置操作时将副作用取出并执行
    bucket.forEach(effect => effect())
    return true
  }
})

function effect() {
  document.body.innerText = obj.text
}

// 执行副作用触发读取操作
effect()

setTimeout(() => {
  // 修改响应式数据
  obj.text = 'hello nomi'
}, 5000)
```
- 先创建一个存储副作用函数的桶 `bucket`;
- `obj` 对原始数据 `data` 进行代理，分别对读取和设置操作进行拦截；
- 读取时将副作用函数添加到 bucket 中，设置时将副作用函数从 bucket 取出并执行； 
