# 队列
> 队列遵循 FIFO (先进先出) 原则

## 队列方法
- enqueue(element)：向队列尾部添加新的项；
- dequeue()：移除队列的第一项（即排在队列最前面的项）并返回被移除的元素；
- peek()：返回队列中第一个元素——最先被添加，也将是最先被移除的元素。队列不做任何变动（不移除元素，只返回元素信息——）；
- isEmpty()：如果队列中不包含任何元素，返回true，否则返回false；
- clear()：移除队列里的所有元素；
- size()：返回队列包含的元素个数。

## 实现
```javascript
// 为了保护数据内部元素
const items = new WeakMap()

class Queue {
  constructor() {
    items.set(this, {
      count: 0, 
      lowestCount: 0,
      queue: {}
    })
  }

  enqueue(element) {
    const target = items.get(this)
    target.queue[target.count] = element
    target.count++
  }

  dequeue() {
    if (this.isEmpty()) {
      return void 0
    }
    const target = items.get(this)
    const result = target.queue[target.lowestCount]
    Reflect.deleteProperty(target.queue, target.lowestCount)
    target.lowestCount++
    return result
  }

  peek() {
    if (this.isEmpty()) {
      return void 0
    }
    const target = items.get(this)
    return target.queue[target.lowestCount]
  }

  isEmpty() {
    return this.size() === 0
  }

  clear() {
    const target = items.get(this)
    target.count = 0
    target.lowestCount = 0
    target.queue = {}
  }

  size() {
    const target = items.get(this)
    return target.count - target.lowestCount
  }
}
```