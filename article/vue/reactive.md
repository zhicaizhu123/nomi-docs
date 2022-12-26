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
因为上面的语句中，我们在副作用函数中，既执行了读取操作，也执行了设置操作，读取时触发 `track`，把当前的副作用收集到桶中，设置时触发 `trigger`，把桶中的副作用函数取出并执行，但是该副作用函数还正在执行中，就要开始下一次的执行了，这样就导致了无限递归循环调用自己，而从产生栈溢出错误。

解决上面问题也很简单，就是判断读取和设置的操作是在同一个副作用函数中进行的话，`trigger` 和 `track` 中执行的副作用函数都是 `activeEffect`，所以如果 `trigger` 触发执行副作用函数与当前执行的副作用函数相同时，则不触发执行。这样就可以避免无限递归调用了。
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

## 调度执行
可调度性是响应系统非常重要的特性。可调度性指的是当 `trigger` 动作触发副作用函数重新执行的时候，有能力决定副作用函数执行的时机、次数以及方式。

我们可以很清楚的知道下面代码打印的结果顺序为 `1`, `2`, `end`
```javascript
const data = { count: 1 }
const obj = new Proxy(data, ...)

effect(() => {
    console.log(obj.count)
})

obj.count++

console.log('end')
```
但是如果我们想不改变代码的编写顺序，打印的结果顺序为 `1`, `end`, `2`话，我们之前实现的代码是不能满足这样的场景的。这时我们就需要响应系统支持调度。

我们可以为 `effect` 函数设计一个选项参数 `options` ，允许指定调度器：
```javascript
effect(() => {
  // 语句
  console.log(obj.count)
}, {
  // ✅✅调度器 scheduler 是一个函数
  scheduler(fn) {
    // ...
  }
})
```
上面代码，我们传递了第二个参数 `options` ，其中它允许制定 `scheduler` 调度函数，同时在 `effect` 函数需要把 `options` 选项挂载到对应的副作用函数上：
```javascript
function effect(fn, options = {}) {
  const effectFn = () => {
    cleanup(effectFn)
     // fn赋值给activeEffect
    activeEffect = effectFn
    // 在调用副作用之前将当前副作用函数压入栈
    effectStack.push(effectFn)
    // 执行副作用函数
    fn()
    // 当副作用执行完毕后，将当前副作用函数弹出栈，并把 active Effect 还原为之前的值
    effectStack.pop()
    activeEffect = effectStack[effectStack.length - 1]
  }
  // ✅✅将options 挂载到 effectFn 上
  effectFn.options = options
  // 用来存储所有与该副作用函数相关联的集合    
  effectFn.deps = []
  effectFn()
}
```
有了调度函数后，我们在 `trigger` 函数中触发副作用函数重新执行的时候，就可以直接调用传递过来的调度函数，从而把控制权交给用户：
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
    // 如果trigger 触发执行副作用函数与当前正在执行的副作用函数相同，则不触发执行
    if (activeEffect !== effect) {
      effectsToRun.add(effect)
    }
  })
  // 设置操作时将副作用取出并执行
  effectsToRun.forEach(effect => {
    // ✅✅如果副作用函数存在调度器，则调用该调度器，并将副作用函数作为参数
    if (typeof effect.options.scheduler === 'function') {
      
      effect.options.scheduler(effect)
    } else {
      // 否则直接执行副作用函数
      effect() 
    }
  })
}
```
如果我们 `effect` 有传递 `options.scheduler` 时，则直接调用调度器函数，并把当前副作用函数作为参数传递过去，用用户自己控制如何执行，否则直接执行副作用函数。

有了上面的逻辑后，我们要实现之前的例子就很简单了
```javascript
effect(() => {
  // 语句
  console.log(obj.count)
}, {
  // ✅✅调度器 scheduler 是一个函数
  scheduler(fn) {
    // ...
    setTimeout(fn)
  }
})

obj.count++ 

console.log('end')
```
上面代码执行的结果顺序就是我们期望的 `1`, `end`, `2`了。

除了控制副作用的执行顺序外，调度器还可以做到控制它的执行次数，这一点很重要，我们思考下下面的例子：
```javascript
effect(() => {
  // 语句
  console.log(obj.count)
})

obj.count++
obj.count++
obj.count++
```
我们很清楚代码打印的结果为 `1`, `2`, `3`, `4`。

我们知道，`obj.count` 一定会从 `1` 自增到 `4`， 如果我们只关心结果不关心过程的话，我们期望只打印 `1` 和 `4`。

如果基于调度器的话，我们很容易实现这个功能：
```javascript
// 任务列表
const jobQueue = new Set()
// 使用 Promise.resolve 将一个任务添加到微任务队列中
const p = Promise.resolve()
// 是否正在刷新队列的标记
let isFlushing = false
function flushJob() {
  if (isFlushing) return
  isFlushing = true
  p.then(() => {
    // 在微任务队列中刷新 jobQueue 队列
    jobQueue.forEach(job => job())
  }).finally(() => {
    // 结束后重置 isFlushing
    isFlushing = false
  })
}


effect(() => {
  // 语句
  console.log(obj.count)
}, {
  scheduler(fn) {
    // 每次调度，将副作用函数添加到 jobQueue 队列中
    jobQueue.add(fn)
    // 调用 flushJob 刷新队列
    flushJob()
  }
})

obj.count++
obj.count++
obj.count++
```
从上面代码我么可以知道，我们定义了一个任务列表 `jobQueue`，它是一个 `Set` 数据结构，目的是利用 `Set` 数据结构的自动去重能力。然后在每次调度执行的时候，先将当前的副作用函数添加到 `jobQueue` 队列中，在调用 `flushJob` 函数刷新队列，在一个周期内，无论调用多少次 `flushJob` `函数，flushJob` 函数都会只执行一次。

## 计算属性 computed 与 lazy
有些场景下，我们并不希望 `effect` 立即执行副作用函数，而是希望它在需要的时候才执行，例如计算属性。这个时候我们可以通过 `options.lazy` 属性达到目的：
```javascript
function effect(fn, options = {}) {
  const effectFn = () => {
    // 从依赖集合中删除副作用函数
    cleanup(effectFn)
    // fn赋值给activeEffect
    activeEffect = effectFn
    // 在调用副作用之前将当前副作用函数压入栈
    effectStack.push(effectFn)
    // 执行副作用函数
    fn()
    // 当副作用执行完毕后，将当前副作用函数弹出栈，并把 active Effect 还原为之前的值
    effectStack.pop()
    activeEffect = effectStack[effectStack.length - 1]
  }
  // 将options 挂载到 effectFn 上
  effectFn.options = options
  // 用来存储所有与该副作用函数相关联的集合
  effectFn.deps = []
  // ✅✅只有非 lazy 的时候才执行
  if (!options.lazy) {
    // 执行副作用函数
    effectFn()
  }
  // ✅✅将副作用函数作为返回值返回
  return effectFn
}
```
通过 options.lazy 的判断后，我们就实现了副作用不立即执行的功能，如果 options.lazy 为真是，我们调用 effect 时，将对应的副作用函数返回，这样就能手动的执行该副作用函数了：
```javascript
const effectFn = effect(() => {
  console.log(obj.count)
}, {
  lazy: true
})

// 手动执行副作用函数
effectFn()
```
如果只是仅仅手动执行副作用函数，意义不大，但是如果把传递给 effect 的副作用函数看作一个 getter ，那么这个 getter 函数可以返回任何值，例如：
```javascript
const effectFn = effect(() => obj.text + ':' + obj.count, {
  lazy: true
})

const value = effectFn()
```
为了实现这个目标，我们还需要对 `effect` 函数做下改造：
```javascript
function effect(fn, options = {}) {
  const effectFn = () => {
    // 从依赖集合中删除副作用函数
    cleanup(effectFn)
    // fn赋值给activeEffect
    activeEffect = effectFn
    // 在调用副作用之前将当前副作用函数压入栈
    effectStack.push(effectFn)
    // ✅✅执行副作用函数
    const res = fn()
    // 当副作用执行完毕后，将当前副作用函数弹出栈，并把 active Effect 还原为之前的值
    effectStack.pop()
    activeEffect = effectStack[effectStack.length - 1]
    // ✅✅将 res 作为 effectFn 的返回值
    return res
  }
  // 将options 挂载到 effectFn 上
  effectFn.options = options
  // 用来存储所有与该副作用函数相关联的集合
  effectFn.deps = []
  // 只有非 lazy 的时候才执行
  if (!options.lazy) {
    // 执行副作用函数
    effectFn()
  }
  // 将副作用函数作为返回值返回
  return effectFn
}
```
通过上面代码的实现我们可以看出，传递给 `effect` 函数的参数 `fn` 才是真正的副作用函数，而 `effectFn` 是我们包装后的副作用函数。为了通过 `effectFn` 得到真正的副作用函数的执行结果，我们需要保存到 `res` 变量中，然后将其作为 `effectFn` 函数的返回值。

现在我们已经能够实现懒执行的副作用函数了，并且可以拿到副作用函数执行的记过，接下来就可以实现计算属性了：
```javascript
function computed(getter) {
  const effectFn = effect(getter, { lazy: true })

  const obj = {
    get value() {
      // 读取value 时才执行 effectFn
      return effectFn()
    }
  }
  return obj
}
```
我们自定义 `computed` 函数，它接收一个 `getter` 函数作为参数，我们把 `getter` 函数作为副作用函数，用它创建一个 `lazy` 的 `effect` ，`computed` 函数返回一个对象，该对象的 `value` 属性是一个访问器属性，只有当读取 `value` 的时候，才会执行 `effectFn` 将结果作为返回值返回。

这样我们就可以使用 `computed` 函数创建一个计算属性了：
```javascript
const res = computed(() =>  obj.text + ':' + obj.count)

console.log(res.value)
```
上述代码可以正常工作，但是现在我们实现的计算属性只做到了懒属性，也就是说，只有当你真正读取 `res.value` 的值时，它才会进行计算并得出值，但是还做不到对值进行缓存，也就是说，就是我们多次访问 `res.value` ，都会导致 `effectFn` 多次计算，即使 `obj.text` 和 `obj.count` 没有改变。

为了解决这个问题，我们需要实现 `computed` 函数的时候，添加对值进行缓存的功能：
```javascript
function computed(getter) {
  // ✅✅用来缓存上一次计算的值
  let value

  // ✅✅用来标记是否需要重新计算值，为true时则需要计算
  let dirty = true

  const effectFn = effect(getter, { lazy: true })

  const obj = {
    get value() {
      if (dirty) {
        // ✅✅只有 dirty 为 true 时才需要计算值，并将得到的返回值存到value
        value = effectFn()
        // ✅✅将 dirty 设置为 false ，下一次访问使用缓存到 value 的值
        dirty = false
      }
      return value
    }
  }

  return obj
}
```
我们新增 `value` 和 `dirty` 两个变量，分别用户缓存上一次计算返回的值和标识是否需要重新计算。当 `dirty` 为 `true` 的时候我们才需要重新计算值，否则直接返回缓存到 `value` 的值。无论我们访问多少次 `res.value` 都会在第一次访问时进行真正的计算，后续的口试发直接读取缓存 `value` 的值。
```javascript
const res = computed(() =>  obj.text + ':' + obj.count)

console.log(res.value)
console.log(res.value)
console.log(res.value)
```

但是上面 computed 函数的实现存在一定的问题，例如下面代码：
```javascript
const res = computed(() =>  obj.text + ':' + obj.count)
console.log(res.value) // hello vue:1
obj.text = 'hello nomi'
console.log(res.value) // hello vue:1
```
因为第一次访问 `res.value` 时，数据就缓存在了 `value` 中，同时 `dirty` 后续都为 `false`，所以即使我们后面怎么修改 `obj.text`，返回的值还是 `value` 缓存的那个值。

很明显，上面的结果不是我们需要的。解决办法也很简单，就是当 `obj.text` 或 `obj.count` 发生变化时，只要设置 `dirty` 为 `true` 就可以了。这个时候就用到了我们之前介绍的 `scheduler` 调度器选项了：
```javascript
function computed(getter) {
  // 用来缓存上一次计算的值
  let value

  // 用来标记是否需要重新计算值，为true时则需要计算
  let dirty = true

  const effectFn = effect(
    getter, 
    { 
        lazy: true,
        // ✅✅ 添加调度器，在调度器中将 dirty 重置为 true
        scheduler() {
            dirty = true
        }
    }
  )

  const obj = {
    get value() {
      if (dirty) {
        // 只有 dirty 为 true 时才需要计算值，并将得到的返回值存到value
        value = effectFn()
        // 将 dirty 设置为 false ，下一次访问使用缓存到 value 的值
        dirty = false
      }
      return value
    }
  }

  return obj
}
```
我们为 `effect` 添加了 `scheduler` 调度器函数，它会在 `getter` 函数中所依赖的响应式数据变化时执行，这样我们在 `scheduler` 函数将 `dirty` 设置为 `true`，在下一次访问 `res.value` 的时候，就回重新调用 `effectFn` 计算值，这样就能够达到预期的效果。

```javascript
const res = computed(() =>  obj.text + ':' + obj.count)
console.log(res.value) // hello vue:1
obj.text = 'hello nomi'
console.log(res.value) // hello nomi:1
```

现在上面设计的计算属性比较完美了，但是还是有一个权限，它体现在当我们在另外一个 `effect` 中读取计算属性的值时：
```javascript
const res = computed(() =>  obj.text + ':' + obj.count)

effect(() => {
    console.log(res.value)
})

obj.text = 'hello nomi'
```
上述代码， `res` 是一个计算属性，并且在另外一个 `effect` 的副作用函数中读取了 `res.value` 的值，如果此时修改了 `obj.text` 的值，我们期望副作用函数重新执行，但是如果尝试运行上面代码，会发现修改了 `obj.text` 并不会触发副作用函数的渲染，这个就是我们上面说的缺陷。

分析问题，我们发现，从本质上看就是一个典型的 `effect` 嵌套问题，一个计算属性内部拥有自己的 `effect` ， 并且它是懒执行的，只有当真正的读取计算属性的值时才会执行，对于计算属性的 `getter` 函数来说，它里面反问的响应式数据只会把 `computed` 函数内部的 `effect` 收集为依赖。而当把计算属性用于另外一个 `effect` 时，就会发生 `effect` 嵌套，外层的 `effect` 不会被内层的 `effect` 中的响应式数据收集。

解决办法很简单，当读取计算属性的值时，我们可以手动调用 `track` 函数进行追踪；当计算属性依赖的响应式数据发生变化时，我们可以手动调用 `trigger` 函数触发响应。
```javascript
function computed(getter) {
  // 用来缓存上一次计算的值
  let value

  // 用来标记是否需要重新计算值，为true时则需要计算
  let dirty = true

  const effectFn = effect(
    getter, 
    { 
      lazy: true, 
      scheduler() {
        dirty = true
        // ✅✅当计算属性依赖的响应式数据发生变化时，手动调用 trigger 函数触发响应
        trigger(obj, 'value')
      } 
    }
  )

  const obj = {
    get value() {
      if (dirty) {
        // 只有 dirty 为 true 时才需要计算值，并将得到的返回值存到value
        value = effectFn()
        // 将 dirty 设置为 false ，下一次访问使用缓存到 value 的值
        dirty = false
      }
      // ✅✅当读取 value 时，手动调用 track 函数进行追踪
      track(obj, 'value')
      return value
    }
  }

  return obj
}
```
当读取一个计算属性的 `value` 的值时，我们手动调用 `track` 函数，把计算属性返回的对象 `obj` 作为 `target` ， 同时作为第一个参数传递给 `track` 函数，当计算属性所依赖的响应式数据发生变化时，会执行调度器函数，在调度器函数里面我们手动调用 `trigger` 函数触发响应。

```javascript
effect(function effectFn() {
    console.log(res.value)
})
```
上面代码会建立一下的联系:
```
computed(obj)
    ⎿ value
         ⎿ effectFn
```

所谓的 `watch` ，其本质就是观测一个响应式数据，当数据发生变化时通知并执行响应的回调函数。
```javascript
watch(obj, () => {
    console.log('数据更新了')
})

// 修改响应数据的值，会导致回调函数执行
obj.text = 'hello nomi'
```

实际上，`watch` 的实现就是利用 `effect` 以及 `options.scheduler` 调用器选项。
```javascript
effect(() => {
    console.log(obj.text)
}, {
    scheduler() {
        console.log('数据更新了')
    }
})
```
上述代码中，在一个副作用函数中访问响应式数据 `obj.text` ，我们知道这会在副作用函数与响应式数据之间建立额联系，当响应式数据发生变化时，会触发副作用函数的执行，但是有一个例外，就是如果副作用函数存在 `scheduler` 选项，当响应式数据发生变化时，会触发 `scheduler` 调度函数的执行，而非直接触发副作用函数执行。从这个角度来看， 其实 `scheduler` 调度函数就相当于一个回调函数，而 `watch` 的实现就是利用了这个特点，下面是个简单的 `watch` 函数的实现：
```javascript
// source 为响应式数据，cb为回调函数
function watch(source, cb) {
  effect(
    // 触发读取操作，从而建立联系
    () => source.text,
    {
      scheduler() {
        // 当数据发生变化时，调用回调函数 cb
        cb()
      }
    }  
  )
}
```
下面我们使用 `watch` 函数：
```javascript
watch(obj, () => {
  console.log('数据更新了')
})

obj.text = 'hello, i am nomi'
```
上面的代码，在 `obj.text` 修改时，会触发回调函数的执行，但是我们注意到，`watch` 函数使用了硬编码对 `source.text` 进行读取操作，为了让 `watch` 更加通用，我们需要封装一个通用的读取函数：
```javascript
function watch(source, cb) {
  effect(
    // ✅✅触发读取操作，从而建立联系
    () => traverse(source),
    {
      scheduler() {
        // 当数据发生变化时，调用回调函数 cb
        cb()
      }
    }  
  )
}

function traverse(value, seen = new Set()) {
  // 使用seen 存储已经读取过的的数据，避免死循环
  if (typeof value !== 'object' || value === null || seen.has(value)) {
    return value
  }
  seen.add(value)
  // 暂时不考虑数据等其他结构
  // 假设 value 是一个对象，递归调用traverse进行处理
  for(const k in value) {
    traverse(value[k])
  }

  return value
}
```
调用 `traverse` 函数进行递归的读取操作，代替硬编码的方式，这样就能读取一个对象上任意属性了，从而当任意属性发生变化时都能够触发回调函数执行。
```javascript
watch(obj, () => {
  console.log('数据更新了')
})

obj.text = 'hello haha'

obj.count++

```
上述代码 `obj.text` 和 `obj.count` 都会触发回调函数的执行。

另外，`watch` 函数也可以接受一个 `getter` 函数：
```javascript
watch(() => obj.text, () => {
    console.log('obj.text 更新了')
})
```

为了满足上面的代码，我们需要改造下 `watch` 函数:
```javascript
function watch(source, cb) {
  // ✅✅定义getter
  let getter
  
  if (typeof source === 'function') {
    // ✅✅如果source 为函数
    getter = source
  } else {
    // ✅✅如果 source 不是函数
    getter = () => traverse(source)
  }

  effect(
    // ✅✅触发读取操作，从而建立联系
    getter,
    {
      scheduler() {
        // 当数据发生变化时，调用回调函数 cb
        cb()
      }
    }  
  )
}
```
上述代码，我们先判断 `source` 如果为函数时，则使用 `getter` 保存，如果 `source` 不为函数则保留之前的做法。

可能你已经注意到，上面实现的 `watch` 中的回调函数拿不到新旧值，那么我们如何在回调函数中获取到变化前后的值呢，这个时候需要充分利用 `effect` 函数 `options.lazy` 选项：
```javascript
function watch(source, cb) {
  // 定义getter
  let getter
  
  if (typeof source === 'function') {
    // 如果source 为函数
    getter = source
  } else {
    // 如果 source 不是函数
    getter = () => traverse(source)
  }
  
  // ✅✅存储新值 
  let newValue
  // ✅✅存储旧值
  let oldValue

  const effectFn = effect(
    // 触发读取操作，从而建立联系
    getter,
    {
      lazy: true,
      scheduler() {
        // ✅✅在 scheduler 中重新执行副作用函数，得到新值
        newValue = effectFn()
        // ✅✅当数据发生变化时，调用回调函数 cb，并传入新旧值
        cb(newValue, oldValue)
        // ✅✅更新旧值，不然下一次会得到错误的旧值
        oldValue = newValue
      }
    }  
  )

  // ✅✅手动调用副作用函数，拿到的值就是旧值
  oldValue = effectFn()
}
```
在这段代码中，最核心的改动是使用 `lazy` 选项创建了一个懒执行的 `effect`。注意上面代码中最下面的部分，我们手动调用 `effectFn` 函数得到的返回值就是旧值，即第一次执行得到的值。当变化发生并触发 `scheduler` 调度函数执行时，会重新调用 `effectFn` 函数并得到新值，这样我们就拿到了旧值与新值，接着将它们作为参数传递给回调函数 `cb` 就可以了。最后一件非常重要的事情是，不要忘记使用新值更新旧值：`oldValue = newValue`，否则在下一次变更发生时会得到错误的旧值。

