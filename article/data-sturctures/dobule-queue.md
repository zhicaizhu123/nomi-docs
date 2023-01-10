# 双端队列
> 双端队列是一种允许我们同时从前端和后端添加和移除元素的特殊队列

双端队列的一个常见应用是存储一系列的撤销操作。每当用户在软件中进行了一个操作，该操作会被存在一个双端队列中（就像在一个栈里）。当用户点击撤销按钮时，该操作会被从双端队列中弹出，表示它被从后面移除了。在进行了预先定义的一定数量的操作后，最先进行的操作会被从双端队列的前端移除。由于双端队列同时遵守了先进先出和后进先出原则，可以说它是把队列和栈相结合的一种数据结构。


## 方法
- `addFront(element)`：在双端队列前端添加新的元素；
- `addBack()`：在双端队列后端添加新的元素；
- `removeFront()`：从双端队列前端移除第一个元素；
- `removeBack()`：从双端队列后端移除第一个元素；
- `peekFront()`：返回双端队列前端的第一个元素；
- `peekBack()`：返回双端队列后端的第一个元素；
- `isEmpty()`：如果队列中不包含任何元素，返回 `true`，否则返回 `false`；
- `clear()`：移除队列里的所有元素；
- `size()`：返回队列包含的元素个数。

## 实现
```javascript
// 为了保护数据内部元素
const items = new WeakMap()

class Dequeue {
  constructor() {
    items.set(this, {
      count: 0, 
      lowestCount: 0,
      queue: {}
    })
  }

  // 在双端队列前端添加新的元素
  addFront(element) {
    const target = items.get(this)
    if (this.isEmpty()) {
      this.addBack(element)
    } else if(target.lowestCount > 0) {
      target.lowestCount--
      target.queue[target.lowestCount] = element
    } else {
      for(let i = target.count; i > 0; i--) {
        target.queue[i] = target.queue[i - 1]
      }
      target.count++
      target.lowestCount = 0
      target.queue[target.lowestCount] = element
    }
  }

  // 在双端队列后端添加新的元素
  addBack(element) {
    const target = items.get(this)
    target.queue[target.count] = element
    target.count++
  }

  // 从双端队列前端移除第一个元素
  removeFront() {
    const target = items.get(this)
    if (this.isEmpty()) {
      return void 0
    }
    const result = target.queue[target.lowestCount]
    Reflect.deleteProperty(target.queue, target.lowestCount)
    target.lowestCount++
    return result
  }

  // 从双端队列后端移除第一个元素
  removeBack() {
    if (this.isEmpty()) {
      return void 0
    }
    const target = items.get(this)
    target.count--
    const result = target.queue[target.count]
    Reflect.deleteProperty(target.queue, target.count)
    return result
  }

  // 返回双端队列前端的第一个元素
  peekFront() {
    const target = items.get(this)
    if (this.isEmpty()) {
      return void 0
    }
    const result = target.queue[target.lowestCount]
    return result
  }

  // 返回双端队列后端的第一个元素
  peekBack() {
    if (this.isEmpty()) {
      return void 0
    }
    const target = items.get(this)
    const result = target.queue[target.count - 1]
    return result
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

## 回文检查
> 回文是正反都能读通的单词、词组、数或一系列字符的序列，例如 `madam` 或 `racecar`。

有不同的算法可以检查一个词组或字符串是否为回文。最简单的方式是将字符串反向排列并检查它和原字符串是否相同。如果两者相同，那么它就是一个回文。我们也可以用栈来完成，但是利用数据结构来解决这个问题的最简单方法是使用双端队列。

算法实现：
```javascript
function palindromeChecker(str) {
  if (typeof str !== 'string' || !str.length) {
    return false;
  }
  const dequeue = new Dequeue();
  const lowerString = str.toLocaleLowerCase().split(' ').join('');
  let isEqual = true;
  let firstChar, lastChar;

  for (let i = 0; i < lowerString.length; i++) {
    dequeue.addBack(lowerString.charAt(i));
  }

  while (dequeue.size() > 1 && isEqual) {
    firstChar = dequeue.removeFront();
    lastChar = dequeue.removeBack();
    if (firstChar !== lastChar) {
      isEqual = false;
    }
  }

  return isEqual;
}
```

我们可以使用下面的代码来尝试 `palindromeChecker` 算法：
```javascript
console.log('abcba', palindromeChecker('abcba')) // true
console.log('racecar', palindromeChecker('racecar')) // true
console.log('Was it a car or a cat I saw', palindromeChecker('Was it a car or a cat I saw')) // true
console.log('hello world', palindromeChecker('hello world')) // false
```