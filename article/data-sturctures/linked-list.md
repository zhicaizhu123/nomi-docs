# 链表

要存储多个元素，数组（或列表）可能是最常用的数据结构。正如本书之前提到的，每种语言都实现了数组。这种数据结构非常方便，提供了一个便利的[]语法来访问其元素。然而，这种数据结构有一个缺点：（在大多数语言中）数组的大小是固定的，从数组的起点或中间插入或移除项的成本很高，因为需要移动元素。

链表存储有序的元素集合，但不同于数组，链表中的元素在内存中并不是连续放置的。每个元素由一个存储元素本身的节点和一个指向下一个元素的引用（也称指针或链接）组成。下图展示了一个链表的结构。

![](https://s2.loli.net/2023/01/11/eiRLU9YqC2uJrg3.jpg)

相对于传统的数组，链表的一个好处在于，添加或移除元素的时候不需要移动其他元素。然而，链表需要使用指针，因此实现链表时需要额外注意。在数组中，我们可以直接访问任何位置的任何元素，而要想访问链表中间的一个元素，则需要从起点（表头）开始迭代链表直到找到所需的元素。

## 方法
- `push(element)`：向链表尾部添加一个新元素。
- `insert(element, index)`：向链表的特定位置插入一个新元素。
- `getElementAt(index)`：返回链表中特定位置的元素。如果链表中不存在这样的元素，则返回 `undefined`。
- `remove(element)`：从链表中移除一个元素。
- `indexOf(element)`：返回元素在链表中的索引。如果链表中没有该元素则返回 -1。
- `removeAt(index)`：从链表的特定位置移除一个元素。
- `getHead()`：获取链头。
- `isEmpty()`：如果链表中不包含任何元素，返回`true`，如果链表长度大于 0 则返回`false`。
- `clear()`：清空链表。
- `size()`：返回链表包含的元素个数，与数组的 `length` 属性类似。
- `toString()`：返回表示整个链表的字符串。由于列表项使用了`Node`类，就需要重写继承自 `JavaScript` 对象默认的 `toString` 方法，让其只输出元素的值。

## 元素节点Node
```javascript
class Node {
  constructor(element) {
    this.element = element
    this.next = void 0
  }
}
```

## 比较两个节点是否相同
```javascript
// 用于比较两个元素是否相等
function defaultEquals(a, b) {
  return a === b
}
```

## 链表结构
```javascript
class LinkedList {
  constructor(equalsFn = defaultEquals) {
    this.head = void 0
    this.count = 0
    this.equalsFn = equalsFn
  }

  push(element) {
    this.insert(element, this.size())
  }

  insert(element, index) {
    if (index >= 0 && index <= this.count) {
      const node = new Node(element)
      if (index === 0) {
        const current = this.head
        node.next = current
        this.head = node
      } else {
        const previous = this.getElementAt(index - 1)
        const current = previous.next
        previous.next = node
        node.next = current
      }
      this.count++
      return true
    }
    return false
  }

  getElementAt(index) {
    if (index >= 0 && index < this.count) {
      let current = this.head
      for(let i = 0; i < index; i++) {
        current = current.next
      }
      return current
    } 
  }

  indexOf(element) {
    let current = this.head
    for(let i = 0; i < this.count && current !== void 0; i++) {
      if (this.equalsFn(element, current.element)) {
        return i
      }
      current = current.next
    }
    return -1
  }

  remove(element) {
    const index = this.indexOf(element)
    return this.removeAt(index)
  }

  removeAt(index) {
    if (index >= 0 && index < this.count) {
      let current = this.head
      if (index === 0) {
        this.head = current.next
      } else {
        const previous = this.getElementAt(index - 1)
        current = previous.next
        previous.next = current.next
      }
      this.count--
      return current.element
    }
  }

  getHead() {
    return this.head
  }

  
  isEmpty() {
    return this.size() === 0
  }

  clear() {
    const size = this.size()
    for(let i = size - 1; i >= 0; i--) {
      this.removeAt(i)
    }
  }

  size() {
    return this.count
  }

  toString() {
    if (this.head === void 0) {
      return '';
    }
    let objString = `${this.head.element}`;
    let current = this.head;
    while(current.next) {
      current = current.next
      objString = `${objString},${current.element}`;
    }
    return objString;
  }
}
```





