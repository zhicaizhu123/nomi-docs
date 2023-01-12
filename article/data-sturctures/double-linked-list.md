# 双向链表

双向链表和普通链表的区别在于，在链表中，一个节点只有链向下一个节点的链接；而在双向链表中，链接是双向的：一个链向下一个元素，另一个链向前一个元素，如下图所示。

![](https://s2.loli.net/2023/01/12/R7wrZd86SAVsGXv.jpg)

## 方法
- `push(element)`：向链表尾部添加一个新元素。
- `insert(element, index)`：向链表的特定位置插入一个新元素。
- `getElementAt(index)`：返回链表中特定位置的元素。如果链表中不存在这样的元素，则返回 `undefined`。
- `remove(element)`：从链表中移除一个元素。
- `indexOf(element)`：返回元素在链表中的索引。如果链表中没有该元素则返回 -1。
- `removeAt(index)`：从链表的特定位置移除一个元素。
- `getHead()`：获取链头。
- `getTail`：获取链尾。
- `isEmpty()`：如果链表中不包含任何元素，返回`true`，如果链表长度大于 0 则返回`false`。
- `clear()`：清空链表。
- `size()`：返回链表包含的元素个数，与数组的 `length` 属性类似。
- `toString()`：返回表示整个链表的字符串。由于列表项使用了`Node`类，就需要重写继承自 `JavaScript` 对象默认的 `toString` 方法，让其只输出元素的值。


## 元素节点Node
继承 [Node](/article/data-sturctures/linked-list.md?id=元素节点node)
```javascript
class DoubleNode extends Node {
  constructor(element) {
    super(element)
    this.prev = void 0
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

## 双向列表结构 
继承 [LinkedList](/article/data-sturctures/linked-list.md?id=链表结构)
```javascript
class DoubleLinkedList extends LinkedList {
  constructor(equalsFn = defaultEquals) {
    super(equalsFn)
    this.tail = void 0
  }

  insert(element, index) {
    if (index >= 0 && index <= this.count) {
      const node = new DoubleNode(element)
      let current = this.head
      if (index === 0) {
        if (this.head === void 0) {
          this.head = node
          this.tail = node
        } else {
          node.next = this.head
          current.prev = node
          this.head = node
        }
      } else if (index === this.count) {
        current = this.tail
        current.next = node
        node.prev = current
        this.tail = node
      } else {
        const previous = this.getElementAt(index - 1)
        const current = previous.next
        node.next = current
        previous.next = node
        node.prev = previous
      }
      this.count++
      return true
    }
    return false
  }

  removeAt(index) {
    if (index >= 0 && index < this.count) {
      let current = this.head
      if (index === 0) {
        this.head = current.next
        if (this.count === 1) {
          this.tail = void 0
        } else {
          this.head.prev = void 0
        }
      } else if (index === this.count - 1) {
        current = this.tail
        this.tail = current.prev
        this.tail.next = void 0
      } else {
        current = this.getElementAt(index)
        const previous = current.prev
        previous.next = current.next
        current.next.prev = previous
      }
      this.count--
      return current.element
    }
  }

  getTail() {
    return this.tail
  }
}
```