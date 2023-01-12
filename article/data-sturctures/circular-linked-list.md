# 单向循环链表
单向循环链表和链表之间唯一的区别在于，最后一个元素的 `next` 指向第一个元素（`head`），如下图所示。

![](https://s2.loli.net/2023/01/12/LyP6wGzKqd8uYpD.jpg)

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
[Node](/article/data-sturctures/linked-list.md?id=元素节点node)

## 比较两个节点是否相同
```javascript
// 用于比较两个元素是否相等
function defaultEquals(a, b) {
  return a === b
}
```

## 单向循环链表结构
继承 [LinkedList](/article/data-sturctures/linked-list.md?id=链表结构)
```javascript
class CircularLinkedList extends LinkedList {
  constructor(equalsFn = defaultEquals) {
    super(equalsFn)
  }

  insert(element, index) {
    if (index >= 0 && index <= this.count) {
      const node = new Node(element)
      if (index === 0) {
        if (this.head === void 0) {
          this.head = node
          node.next = this.head
        } else {
          const current = this.head
          node.next = current
          current = this.getElementAt(this.size())
          current.next = this.head
        }
      } else {
        const previous = this.getElementAt(index - 1)
        node.next = previous.next
        previous.next = node
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
        if (this.size() === 1) {
          this.head = void 0
        } else {
          const removed = this.head
          current = this.getElementAt(this.size())
          this.head = this.head.next
          current.next = this.head
          current = removed
        }
      } else {
        const previous = this.getElementAt(index - 1)
        current = previous.next
        previous.next = current.next
      }
      this.count--
      return current.element
    }
  }

  toString() {
    if (this.head === void 0) {
      return '';
    }
    let objString = `${this.head.element}`;
    let current = this.head;
    while(current.next && current.next !== this.head) {
      current = current.next
      objString = `${objString},${current.element}`;
    }
    return objString;
  }
}
```