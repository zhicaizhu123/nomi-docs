# 栈链表
> 使用[双端链表](/article/data-sturctures/double-linked-list.md)实现栈数据结构

## 方法
- `push(element)`：添加一个新元素到栈顶；
- `pop()`：移除栈顶的元素，同时返回被移除的元素；
- `peek()`：返回栈顶的元素，不对栈做任何修改（该方法不会移除栈顶的元素，仅仅返回它）；
- `isEmpty()`：如果栈里没有任何元素就返回 `true` ，否则返回 `false` ；
- `clear()`：移除栈里的所有元素；
- `size()`：返回栈里的元素个数。

## 栈链表结构
```javascript
class StackLinkedList {
  constructor() {
    this.items = new DoubleLinkedList()
  }

  push(element) {
    this.items.push(element)
  }

  pop() {
    return this.items.removeAt(this.items.size() - 1)
  }

  peek() {
    return this.items.getElementAt(this.items.size() - 1).element
  }

  isEmpty() {
    return this.items.isEmpty()
  }

  clear() {
    this.items.clear()
  }

  size() {
    return this.items.size()
  }
}
```
之所以使用双向链表而不是链表，是因为对栈来说，我们会向链表尾部添加元素，也会从链表尾部移除元素。[`DoublyLinkedList`](/article/data-sturctures/double-linked-list.md)类有列表最后一个元素（`tail`）的引用，无须迭代整个链表的元素就能获取它。双向链表可以直接获取头尾的元素，减少过程消耗，它的时间复杂度和原始的Stack实现相同，为`O(1)`