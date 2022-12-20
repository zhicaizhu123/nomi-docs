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
- 读取时将副作用函数添加到 `bucket` 中，设置时将副作用函数从 `bucket` 取出并执行； 

## 完善响应式系统
上述的响应式数据的实现，副作用函数 `effect` 是一个硬编码，导致一旦函数名称不是 `effect` 就无法运行，如果我们希望副作用函数足够灵活，我们还需要提供一个用来注册副作用的函数的机制。

```javascript
let activeEffect // 用于临时存储即将当前运行的副作用函数
function effect(fn) {
    // 存储将要运行的副作用函数
    activeEffect = fn
    // 运行副作用函数
    fn()
}
```
上述代码中，我们对 `effect` 进行了改造，它将变成一个用于注册副作用函数的函数，其接受一个副作用函数 `fn` 作为参数，其中使用全局变量 `activeEffect` 来存储来当前将要运行的副作用函数 `fn`，接着执行当前副作用函数。

然后我们对副作用的收集逻辑也做下优化。
```javascript
// 对原始数据的代理
const obj = new Proxy(data, {
  get(target, key) {
    // 将 activeEffect 存储的副作用函数添加到桶里
    if (activeEffect) {
      bucket.add(activeEffect)
    }
    return target[key]
  },

  set(target, key, value) {
    target[key] = value
    // 设置操作时将副作用取出并执行
    bucket.forEach(effect => effect())
    return true
  }
})
```

我们使用改造后的 `effect` 注册副作用函数
```javascript
effect(() => {
    document.body.innerText = obj.text
})
```

5秒后这个 `obj.text`
```javascript
setTimeout(() => {
  // 修改响应式数据
  obj.text = 'hello nomi'
}, 5000)
```

经过上面的改造我们就不需要依赖副作用函数的名字了。

但是当我们运行以下的代码是，我们发现以下的 'effect' 打印了两次
```javascript
effect(() => {
    console.log('effect')
    document.body.innerText = obj.text
})

setTimeout(() => {
  // 修改响应式数据
  obj.nomi = 'hello nomi'
}, 5000)
```
我们可以看到，匿名副作用函数内部对去了 `obj.text` 的值，于是匿名副作用函数与字段 `obj.text`之间建立响应联系，接着我们设置新的属性 `obj.nomi`。我们知道，在匿名副作用函数中没有读取 `obj.nomi` 属性的值 ，所以理论上，字段 `obj.nomi` 并没有与副作用函数建立响应联系，所以定时器内语句的执行不应该触发匿名副作用函数的重新执行。为了解决这个问题，我们需要重新设计桶的数据结构。

之前我们使用 `Set` 数据结构存储副作用函数，导致这个问题的根本原因是，我们**没有在副作用函数与被操作的目标字段直接建立明确的联系**。当读取属性时，无论读取的是哪一个属性，都会把副作用函数收集到桶里，当设置属性时，无论设置哪一个属性，也都会把桶里的副作用全部取出并执行。解决的方法很简单，只需要在副作用函数与被操作的字段之间建立联系即可。

那我们应该如何设计数据结构呢？观察以下代码
```javascript
effect(function effectFn() {
    document.body.innerText = obj.text
})
```
这里有三个角色
- 被操作（读取）的代理对象 obj
- 被操作（读取）的字段名 text
- 使用 effect 函数注册的副作用函数 effectFn

如果用 target 来表示一个代理对象所代理的原始对象，用 key 表示被操作的字段名，用 effectFn 表示被注册的副作用函数，那么可以为这三个角色建立以下关系
```
target
  ⎿ key
      ⎿ effectFn
```
这是一个属性结构。

我们使用代码来实现这个新的桶，首先我们需要使用 WeakMap 代替 Set 作为桶的数据结构
```javascript
// 存储副作用函数的桶
const bucket = new WeakMap()
    
// 原始数据
const data = { text: 'hello vue' }
    
// 存储副作用函数的桶
const bucket = new WeakMap()
    
// 对原始数据的代理
const obj = new Proxy(data, {
  get(target, key) {
    // 将 activeEffect 存储的副作用函数添加到桶里
    if (!activeEffect) return
    // 根据 target 从桶中取出 depsMap, 它是一个 Map 类型
    let depsMap = bucket.get(target)
    // 如果不存在depsMap，那么新建一个 Map 并与 target 关联
    if (!depsMap) {
      bucket.set(target, (depsMap = new Map()))
    }    
    // 根据 key 从桶中取出 deps, 它是一个 Set 类型
    // 里面存储所有与当前 key 相关联的副作用函数
    let deps = depsMap.get(key)
    // 如果不存在deps，那么新建一个 Set 并与 key 关联
    if (!deps) {
      depsMap.set(key, (deps = new Set()))
    }
    // 将当前激活的副作用函数存储到桶
    deps.add(activeEffect)
    return target[key]
  },
    
  set(target, key, value) {
    target[key] = value
    // 根据 target 从桶中取出 depsMap, 它是一个 Map 类型
    const depsMap = bucket.get(target)
    if (!depsMap) return
    // 根据 key 从桶中取出 deps, 它是一个 Set 类型
    const effects = depsMap.get(key)
    if (!effects) return
    effects.forEach(effect => effect())
    // 设置操作时将副作用取出并执行
    return true
  }
})
```
从上面代码我们可以看到，我们分别使用了`WeakMap`、 `Map`、 `Set` 三个数据结构
- `WeakMap` 由 target -> Map 构成；
- `Map` 由 key -> Set 构成。

经过上面的改造，之前问题就解决了。

我们对代码做下封装，我们把 `get` 拦截函数里把副作用函数收集到桶的逻辑单独封装到一个 `track` 函数中，把触发副作用函数重新执行的逻辑封装到 `trigger` 函数中。
``` javascript
// 存储副作用函数的桶
const bucket = new WeakMap()

// 原始数据
const data = { text: 'hello vue' }
    
// 对原始数据的代理
const obj = new Proxy(data, {
  get(target, key) {
    // 将 activeEffect 存储的副作用函数添加到桶里
    track(target, key)
    return target[key]
  },

  set(target, key, value) {
    target[key] = value
    // 把副作用从桶取出并执行
    trigger(target, key)
    return true
  }
})
    
// 在 get 拦截函数内调用 track 函数追踪变化
function track(target, key) {
  // 将 activeEffect 存储的副作用函数添加到桶里
  if (!activeEffect) return
  // 根据 target 从桶中取出 depsMap, 它是一个 Map 类型
  let depsMap = bucket.get(target)
  // 如果不存在depsMap，那么新建一个 Map 并与 target 关联
  if (!depsMap) {
    bucket.set(target, (depsMap = new Map()))
  }    
  // 根据 key 从桶中取出 deps, 它是一个 Set 类型
  // 里面存储所有与当前 key 相关联的副作用函数
  let deps = depsMap.get(key)
  // 如果不存在deps，那么新建一个 Set 并与 key 关联
  if (!deps) {
    depsMap.set(key, (deps = new Set()))
  }
  // 将当前激活的副作用函数存储到桶
  deps.add(activeEffect)
}
    
// 在 set 拦截函数内调用 trigger 函数触发改变
function trigger(targe, key) {
  target[key] = value
  // 根据 target 从桶中取出 depsMap, 它是一个 Map 类型
  const depsMap = bucket.get(target)
  if (!depsMap) return
  // 根据 key 从桶中取出 deps, 它是一个 Set 类型
  const effects = depsMap.get(key)
  if (!effects) return
  // 设置操作时将副作用取出并执行
  effects.forEach(effect => effect())
}
```
把逻辑封装到 track 和 trigger 中，能为我们带来极大的便利性。
