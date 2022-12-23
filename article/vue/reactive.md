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
- 被操作（读取）的代理对象 `obj`
- 被操作（读取）的字段名 `text`
- 使用 `effect` 函数注册的副作用函数 `effectFn`

如果用 `target` 来表示一个代理对象所代理的原始对象，用 `key` 表示被操作的字段名，用 `effectFn` 表示被注册的副作用函数，那么可以为这三个角色建立以下关系
```
target
  ⎿ key
      ⎿ effectFn
```
这是一个属性结构。

我们使用代码来实现这个新的桶，首先我们需要使用 `WeakMap` 代替 `Set` 作为桶的数据结构
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
function trigger(target, key) {
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
把逻辑封装到 `track` 和 `trigger` 中，能为我们带来极大的便利性。

## 分支切换和cleanup
观察以下的代码
```javascript
effect(() => {
    document.body.innerText = obj.ok ? obj.text : 'nomi'
})
```
当 `obj.ok` 设置为 `false` 后，副作用函数重新执行，`obj.text` 都不会被读取，所以理想状态下，副作用函数不应该被 `obj.text` 所对应的依赖集合收集。

按照之前的实现，我们即使把 `obj.ok` 设置为 `false` 并触发副作用函数重新执行后，`obj.text` 就不会再被读取， 但是后面我们修改 `obj.text` 时，副作用函数还是会被重新执行，因为 `obj.text` 关联的副作用函数还是存在，这是我们不需要看到的。

解决思路也很简单，在每次副作用执行前，可以先把它从所有与之关联的依赖集合中删除。在副作用执行完毕后，会重新建立联系，但是在新的联系中不会包含遗留的副作用函数了。

我们改造下注册副作用函数的函数 `effect`
```javascript
function effect(fn) {
    const effectFn = () => {
      // fn赋值给activeEffect
      activeEffect = effectFn
      // 执行副作用函数
      fn()
    }
    // 用来存储所有与该副作用函数相关联的集合
    effectFn.deps = []
      // 执行副作用函数
    effectFn()
}
```
在 `effect` 内部我们定义了新的 `effectFn` 函数，并为其添加 `effectFn.deps` 属性，用来存储所有包含当前副作用的函数的依赖集合。

那么 `effectFn.deps` 中的依赖集合是如何收集的？我们改造下 `track` 函数
```javascript
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
    // ✅✅ 添加到 activeEffect.deps 数组中
    activeEffect.deps.push(deps)
}
```

我们将执行当前的副作用函数 `activeEffect` 添加到依赖集合 `deps，说明` deps 就是一个与当前副作用函数存在联系的依赖集合，所以我们也把它添加到 `activeEffect.deps` 数组中，这样就完成了依赖集合到的收集。 

有了这个联系后，我们就可以在每次执行副作用函数执行时，根据 `effectFn.deps` 获取所有相关联的依赖集合，将副作用函数从依赖集合中移除。
```javascript
function effect(fn) {
    const effectFn = () => {
      // 完成清除工作
      cleanup(effectFn)
      // fn赋值给activeEffect
      activeEffect = effectFn
      // 执行副作用函数
      fn()
    }
    // 用来存储所有与该副作用函数相关联的集合
    effectFn.deps = []
      // 执行副作用函数
    effectFn()
}
```

`cleanup` 函数实现：

```javascript
function cleanup(effectFn) {
    // 遍历 effectFn.deps数组
    for(let i = 0; i < effectFn.deps.length; i++) {
      // deps 是依赖集合
      const deps = effectFn.deps[i]
      // 将 effectFn 从依赖集合中删除
      deps.delete(effectFn)
    }
    // 最后需要重置 effectFn.deps 数组
    effectFn.deps.length = 0
}
```

改造完成后，我们重新运行代码，发现目前的实现会导致无限循环执行，问题出在 `trigger` 函数中
```javascript
function trigger(target, key) {
    // 根据 target 从桶中取出 depsMap, 它是一个 Map 类型
    const depsMap = bucket.get(target)
    if (!depsMap) return
    // 根据 key 从桶中取出 deps, 它是一个 Set 类型
    const effects = depsMap.get(key)
    if (!effects) return
    // ⚠️⚠️ 设置操作时将副作用取出并执行
    effects.forEach(effect => effect())
}
```
`trigger` 内部，我们遍历 `effects` 集合，它是一个 `Set` 集合，当副作用函数执行时，会调用 `cleanup` 进行清除，实际就是从 `effects` 集合中将当前执行的副作用函数剔除，但是副作用函数的执行会导致其重新被收集到集合中，而此时对于 `effects` 集合遍历还在进行中，所以会被重新被访问。

要解决上面的问题也很简单，我们可以构造一个新的 `Set` 集合并遍历它:
```javascript
function trigger(target, key) {
    // 根据 target 从桶中取出 depsMap, 它是一个 Map 类型
    const depsMap = bucket.get(target)
    if (!depsMap) return
    // 根据 key 从桶中取出 deps, 它是一个 Set 类型
    const effects = depsMap.get(key)
    if (!effects) return
  
    // ✅✅ 避免无限执行
    const effectsToRun = new Set(effects)
    // 设置操作时将副作用取出并执行
    effectsToRun.forEach(effect => effect())
}
```
其中 `effectsToRun` 就是构建的新的集合，代替之间遍历 `effects` 集合，从而避免了无限执行。

## 嵌套的effect和effect栈
effect 可以嵌套
```javascript
effect(function effectFn1() {
    effect(function effectFn1() {
        // ...
    })
    // ...
})
```
上面的代码会存在什么问题呢？我们继续思考我们之前 `effect` 的实现
```javascript
let activeEffect
function effect(fn) {
    const effectFn = () => {
      // 完成清除工作
      cleanup(effectFn)
      // fn赋值给activeEffect
      activeEffect = effectFn
      // 执行副作用函数
      fn()
    }
    // 用来存储所有与该副作用函数相关联的集合
    effectFn.deps = []
      // 执行副作用函数
    effectFn()
}
```
`activeEffect` 是一个全局变量，执行一次 `effect` 就会被更新，如果是嵌套的 `effect`执行的话，当执行到内部的 `effect` 时，`activeEffect` 就会被更新，但是此时外层的 `effect` 还没有执行完，`activeEffect` 已更新成内部的副作用函数了。

如何解决上面的问题呢？我们可以是使用一个 **栈结构** 来存储副作用函数。
```javascript
// ✅✅effect 栈
const effectStack = []
function effect(fn) {
    const effectFn = () => {
      cleanup(effectFn)
        // fn赋值给activeEffect
      activeEffect = effectFn
      // ✅✅在调用副作用之前将当前副作用函数压入栈
      effectStack.push(effectFn)
      // 执行副作用函数
      fn()
      // ✅✅当副作用执行完毕后，将当前副作用函数弹出栈，并把 active Effect 还原为之前的值
      effectStack.pop()
      activeEffect = effectStack[effectStack.length - 1]
    }
    // 用来存储所有与该副作用函数相关联的集合    
    effectFn.deps = []
    effectFn()
}
```
其中 `effectStack` 即使我们定义存储副作用函数的栈，当副作用函数执行时，将单前副作用函数压入栈中，待副作用函数执行完毕后将其弹出，并始终让 `activeEffect` 指向栈顶的副作用函数。这样就能做到一个响应式数据只会收集直接读取其值的副作用函数，而不会出现互相影响的情况。

## 避免无限递归循环
假如我们运行下面的代码，会发现报栈溢出的错误
```javascript
effect(() => {
    obj.count = obj.count + 1
})
```
因为上面的语句中，我们在副作用函数中，既执行了读取操作，也执行了设置操作，读取时触发 track，把当前的副作用收集到桶中，设置时触发 trigger，把桶中的副作用函数取出并执行，但是该副作用函数还正在执行中，就要开始下一次的执行了，这样就导致了无限递归循环调用自己，而从产生栈溢出错误。

解决上面问题也很简单，就是判断读取和设置的操作是在同一个副作用函数中进行的话，trigger 和 track 中执行的副作用函数都是 activeEffect，所以如果 trigger 触发执行副作用函数与当前执行的副作用函数相同时，则不触发执行。这样就可以避免无限递归调用了。
```javascript
function trigger(target, key) {
    // 根据 target 从桶中取出 depsMap, 它是一个 Map 类型
    const depsMap = bucket.get(target)
    if (!depsMap) return
    // 根据 key 从桶中取出 deps, 它是一个 Set 类型
    const effects = depsMap.get(key)
    if (!effects) return
      
    const effectsToRun = new Set()
    effects.forEach(effect => {
      // ✅✅如果trigger 触发执行副作用函数与当前正在执行的副作用函数相同，则不触发执行
      if (activeEffect !== effect) {
        effectsToRun.add(effect)
      }
    })
    // 设置操作时将副作用取出并执行
    effectsToRun.forEach(effect => effect())
}
```

