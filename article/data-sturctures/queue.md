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

## 击鼓传花游戏
由于队列经常被应用在计算机领域和我们的现实生活中，就出现了一些队列的修改版本。这其中的一种叫作循环队列。循环队列的一个例子就是击鼓传花游戏（hot potato）。在这个游戏中，孩子们围成一个圆圈，把花尽快地传递给旁边的人。某一时刻传花停止，这个时候花在谁手里，谁就退出圆圈、结束游戏。重复这个过程，直到只剩一个孩子（胜者）。

算法实现如下：
```javascript
function hotPotato(elements, num) {
  const queue = new Queue()
  const eliminated = []

  for(let i = 0; i < elements.length; i++) {
    queue.enqueue(elements[i])
  }

  while(queue.size() > 1) {
    for(let i = 0; i < num; i++) {
      queue.enqueue(queue.dequeue())
    }
    eliminated.push(queue.dequeue())
  }

  return {
    eliminated,
    winner: queue.dequeue()
  }
}
```

我们可以使用下面的代码来尝试 `hotPotato` 算法：
```javascript
const names = ['John', 'Jack', 'Camila', 'Ingrid', 'Carl'];
const result = hotPotato(names, 7);
result.eliminated.forEach(name => {
  console.log(`${name}在击鼓传花游戏中被淘汰。`);
});

console.log(`胜利者： ${result.winner}`);
```

结果如下：
```
Camila在击鼓传花游戏中被淘汰。
Jack在击鼓传花游戏中被淘汰。
Carl在击鼓传花游戏中被淘汰。
Ingrid在击鼓传花游戏中被淘汰。
胜利者：John
```

下图模拟了这个输出过程：
![](https://s2.loli.net/2023/01/10/NbtsvmFVUd9DTjl.jpg)
