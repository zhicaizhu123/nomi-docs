# 有序链表

有序链表是指保持元素有序的链表结构。除了使用排序算法之外，我们还可以将元素插入到正确的位置来保证链表的有序性。

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

## 两个节点排序逻辑
```javascript
const Compare = {
  lt: -1,
  gt: 1,
  qe: 0
}

function defaultCompare(a, b) {
  if (a === b) return Compare.qe
  return a < b ? Compare.lt : Compare.gt
}
```

## 有序链表结构 
继承 [LinkedList](/article/data-sturctures/linked-list.md?id=链表结构)
```javascript
class SortedLinkedList extends LinkedList {
  constructor(equalsFn = defaultEquals, compareFn = defaultCompare) {
    super(equalsFn)
    this.compareFn = compareFn
  }

  insert(element, index = 0) {
    if (this.isEmpty()) {
      return super.insert(element, 0)
    }
    const position = this.getIndexNextSortedElement(element)
    return super.insert(element, position)
  }

  getIndexNextSortedElement(element) {
    let current = this.head
    let i = 0
    while(this.size() && current) {
      const comp = this.compareFn(element, current.element)
      if (comp === Compare.lt) {
        return i
      }
      current = current.next
      i++
    }
    return i
  }
}
```
