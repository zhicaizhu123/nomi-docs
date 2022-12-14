# 栈
> 栈遵循 LIFO (后进先出) 原则

## 方法
- `push(element)`：添加一个新元素到栈顶；
- `pop()`：移除栈顶的元素，同时返回被移除的元素；
- `peek()`：返回栈顶的元素，不对栈做任何修改（该方法不会移除栈顶的元素，仅仅返回它）；
- `isEmpty()`：如果栈里没有任何元素就返回 `true` ，否则返回 `false` ；
- `clear()`：移除栈里的所有元素；
- `size()`：返回栈里的元素个数。

## 基于数组的栈的实现
```javascript
// 为了保护数据内部元素
const items = new WeakMap()

class Stack {
  constructor() {
    // 为了保护数据内部元素
    items.set(this, [])
  }

  // 入栈
  push(data) {
    items.get(this).push(data)
  }

  // 出栈
  pop() {
    return items.get(this).pop()
  }

  // 查看栈顶元素
  peek() {
    const list = items.get(this)
    return list[list.length - 1]
  }

  // 检查栈是否为空
  isEmpty() {
    return this.size() === 0
  }

  // 检查栈是否为空
  clear() {
    items.set(this, [])
  }

  // 栈大小
  size() {
    return items.get(this).length
  }
}
```

## 基于普通对象的栈的实现
```javascript
// 为了保护数据内部元素
const items = new WeakMap()

class Stack {
  constructor() {
    // 为了保护数据内部元素
    items.set(this, {
      stack: {},
      count: 0,
    })
  }

  getData() {
    return items.get(this)
  }

  // 入栈
  push(data) {
    const target = items.get(this)
    target.stack[target.count] = data
    target.count++
  }

  // 出栈
  pop() {
    const target = items.get(this)
    target.count--
    const result = target.stack[target.count]
    Reflect.deleteProperty(target.stack, target.count)
    return result
  }

  // 查看栈顶元素
  peek() {
    const target = items.get(this)
    const result = target.stack[target.count - 1]
    return result
  }

  // 检查栈是否为空
  isEmpty() {
    return this.size() === 0
  }

  // 检查栈是否为空
  clear() {
    const target = items.get(this)
    target.stack = {}
    target.count = 0
  }

  // 栈大小
  size() {
    const target = items.get(this)
    return target.count
  }
}
```

## 进制转换器
进制转换器能把十进制转换成基数为 2～36 的任意进制。除了把十进制数除以 2 转成二进制数，还可以传入其他任意进制的基数为参数。

算法实现：
```javascript
function baseConverter(decNumber, base) {
  const remStack = new Stack();
  const digits = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ'; // {6}
  let number = decNumber;
  let rem;
  let baseString = '';

  if (! (base >= 2 && base <= 36)) {
      return '';
  }

  while (number > 0) {
      rem = Math.floor(number % base);
      remStack.push(rem);
      number = Math.floor(number / base);
  }
  while (! remStack.isEmpty()) {
    baseString += digits[remStack.pop()]; // {7}
  }

  return baseString;
}
```

我们可以使用下面的代码来尝试 `baseConverter` 算法：
```javascript
console.log(baseConverter(100345, 2)); // 11000011111111001
console.log(baseConverter(100345, 8)); // 303771
console.log(baseConverter(100345, 16)); // 187F9
console.log(baseConverter(100345, 35)); // 2BW0
```