# 散列表

散列表（HashTable）是字典（Dictionary）的一种散列表实现方式。

散列算法的作用是尽可能快地在数据结构中找到一个值。在之前的章节中，你已经知道如果要在数据结构中获得一个值（使用`get`方法），需要迭代整个数据结构来找到它。如果使用**散列函数**，就知道值的具体位置，因此能够快速检索到该值。**散列函数**的作用是给定一个键值，然后返回值在表中的地址。

## 方法
- `put(key, value)`：向散列表增加一个新的项（也能更新散列表）。
- `delete(key)`：根据键值从散列表中移除值。
- `get(key)`：返回根据键值检索到的特定的值。

## 散列表结构
```javascript
function defaultToStrFn(value) {
  if (value === null) return 'NULL'
  if (value === void 0) return 'UNDEFINED'
  if (typeof value === 'string' || value instanceof String) return `${value}`
  return value.toString()
}

class ValuePair {
  constructor(key, value) {
    this.key = key
    this.value = value
  }

  toString() {
    return `[#${this.key}: ${this.value}]`;
  }
}

class HashTable {
  constructor(toStrFn = defaultToStrFn) {
    this.toStrFn = toStrFn
    this.table = {}
  }

  djb2HashCode(key) {
    const tableKey = this.toStrFn(key);
    let hash = 5381;
    for (let i = 0; i < tableKey.length; i++) {
      hash = (hash * 33) + tableKey.charCodeAt(i);
    }
    return hash % 1013;
  }

  hashCode(key) {
    return this.djb2HashCode(key);
  }


  put(key, value) {
    if (key !== void 0 && key !== null) {
      const hashKey = this.hashCode(key)
      this.table[hashKey] = new ValuePair(key, value)
      return true
    }
    return false
  }

  delete(key) {
    const hashKey = this.hashCode(key)
    const data = this.table[hashKey]
    if (data !== void 0) {
      Reflect.deleteProperty(this.data, hashKey)
      return true
    }
    return false
  }

  get(key) {
    const data = this.table[this.hashCode(key)]
    return data === void 0 ? void 0 : data.value
  }
}

```

## 散列表冲突
有时候，一些键会有相同的散列值。不同的值在散列表中对应相同位置的时候，我们称其为冲突。例如，我们看看下面的代码会得到怎样的输出结果。

```javascript
const hash = new HashTable();
hash.put('Ygritte', 'ygritte@email.com');
hash.put('Jonathan', 'jonathan@email.com');
hash.put('Jamie', 'jamie@email.com');
hash.put('Jack', 'jack@email.com');
hash.put('Jasmine', 'jasmine@email.com');
hash.put('Jake', 'jake@email.com');
hash.put('Nathan', 'nathan@email.com');
hash.put('Athelstan', 'athelstan@email.com');
hash.put('Sue', 'sue@email.com');
hash.put('Aethelwulf', 'aethelwulf@email.com');
hash.put('Sargeras', 'sargeras@email.com');
```

打印生成的hash值 和 key 值
```
4 - Ygritte
5 - Jonathan
5 - Jamie
7 - Jack
8 - Jasmine
9 - Jake
10 - Nathan
7 - Athelstan
5 - Sue
5 - Aethelwulf
10 - Sargeras
```

我们发现上面不同的key值生成的hash值可能相同，所以如果存在多个 key 对应的 hash 相同的情况下，put 方法会把之前的值覆盖掉。

## 解决冲突的方法

### 分离链接法
分离链接法包括为散列表的每一个位置创建一个链表并将元素存储在里面。它是解决冲突的最简单的方法，但是在HashTable实例之外还需要额外的存储空间。

例如，我们在之前的测试代码中使用分离链接并用图表示的话，输出结果将会是如下这样（为了简化，图表中的值被省略了）。

![](https://s2.loli.net/2023/01/12/W4Ra2yXgYxDGN7m.jpg)

对于分离链接来说，只需要重写三个方法：`put`、`get`和`delete`。这三个方法在每种技术实现中都是不同的

```javascript
class HashTable {
...

put(key, value) {
  if (key !== void 0 && key !== null) {
    const hashKey = this.hashCode(key)
    if (this.table[hashKey] === void 0) {
      this.table[hashKey] = new LinkedList()
      return true
    }
    this.table[hashKey].push(new ValuePair(key, value))
  }
  return false
}

delete(key) {
  const hashKey = this.hashCode(key)
  const linkedList = this.table[hashKey]
  if (linkedList !== void 0 && !linkedList.isEmpty()) {
    let current = linkedList.getHead()
    while(current !== void 0) {
      if (current.element.key === key) {
        linkedList.remove(current.element)
        if (linkedList.isEmpty()) {
          Reflect.deleteProperty(this.table, hashKey)
        }
        return true
      }
      current = current.next
    }
  }
  return false
}

get(key) {
  const hashKey = this.hashCode(key)
  const linkedList = this.table[hashKey]
  if (linkedList !== void 0 && !linkedList.isEmpty()) {
    let current = linkedList.getHead()
    while(current !== void 0) {
      if (current.element.key === key) {
        return current.element.value
      }
      current = current.next
    }
  }
}

...
}
```
其中 `LinkedList` 的实现可参考[链表结构](/article/data-sturctures/linked-list.md?id=链表结构)


### 线性探查法

另一种解决冲突的方法是线性探查。之所以称作线性，是因为它处理冲突的方法是将元素直接存储到表中，而不是在单独的数据结构中。

当想向表中某个位置添加一个新元素的时候，如果索引为 `position` 的位置已经被占据了，就尝试 `position + 1`的位置。如果 `position + 1`的位置也被占据了，就尝试 `position + 2`的位置，以此类推，直到在散列表中找到一个空闲的位置。想象一下，有一个已经包含一些元素的散列表，我们想要添加一个新的键和值。我们计算这个新键的 `hash` ，并检查散列表中对应的位置是否被占据。如果没有，我们就将该值添加到正确的位置。如果被占据了，我们就迭代散列表，直到找到一个空闲的位置。

下图展现了这个过程。

![](https://s2.loli.net/2023/01/12/B7S4TCOPMGjLtbY.jpg)

当我们从散列表中移除一个键值对的时候，仅将本章之前的数据结构所实现位置的元素移除是不够的。如果我们只是移除了元素，就可能在查找有相同hash（位置）的其他元素时找到一个空的位置，这会导致算法出现问题。

线性探查技术分为两种。第一种是软删除方法。我们使用一个特殊的值（标记）来表示键值对被删除了（惰性删除或软删除），而不是真的删除它。经过一段时间，散列表被操作过后，我们会得到一个标记了若干删除位置的散列表。这会逐渐降低散列表的效率，因为搜索键值会随时间变得更慢。能快速访问并找到一个键是我们使用散列表的一个重要原因。

下图展示了这个过程。

![](https://s2.loli.net/2023/01/12/2rEgSsVGthf8cyY.jpg)

第二种方法需要检验是否有必要将一个或多个元素移动到之前的位置。当搜索一个键的时候，这种方法可以避免找到一个空位置。如果移动元素是必要的，我们就需要在散列表中挪动键值对。

下图展现了这个过程。

![](https://s2.loli.net/2023/01/12/yZ2LY8qHPuDhAIX.jpg)

两种方法都有各自的优缺点。本章会实现第二种方法（移动一个或多个元素到之前的位置）。

对于线性探查来说，只需要重写三个方法：`put`、`get`和`delete`。这三个方法在每种技术实现中都是不同的

```javascript
class HashTable {
...

verifyRemoveSideEffect(key, removedPosition) {
  const hash = this.hashCode(key);
  let index = removedPosition + 1;
  while (this.table[index] !== void 0) {
    const posHash = this.hashCode(this.table[index].key);
    if (posHash <= hash || posHash <= removedPosition) {
      this.table[removedPosition] = this.table[index];
      Reflect.deleteProperty(this.table, index)
      delete this.table[index];
      removedPosition = index;
    }
    index++;
  }
}

put(key, value) {
  if (key !== void 0 && key !== null) {
    const hashKey = this.hashCode(key)
    if (this.table[hashKey] === void 0) {
      this.table[hashKey] = new ValuePair(key, value)
      return true
    } else {
      let index = hashKey + 1
      while(this.table[index] !== void 0) {
        index++
      }
      this.table[index] = new ValuePair(key, value)
    }
    return true
  }
  return false
}

delete(key) {
  const hashKey = this.hashCode(key)
  const data = this.table[hashKey]
  if (data !== void 0) {
    if (data.key === key) {
      Reflect.deleteProperty(this.table, hashKey)
      this.verifyRemoveSideEffect(key, hashKey)
      return true
    }
    let index = hashKey + 1 
    while(this.table[index] !== void 0 && this.table[index].key !== key) {
      index++
    }
    if (this.table[key] !== void 0 && this.table[index].key === key) {
      Reflect.deleteProperty(this.table, index)
      this.verifyRemoveSideEffect(key, index)
      return true
    }
  }
  return false
}

get(key) {
  const hashKey = this.hashCode(key)
  const data = this.table[hashKey]
  if (data !== void 0) {
    if (data.key === key) {
      return data.value
    }
    let index = hashKey + 1
    while(this.table[index] !== void 0 && this.table[index].key !== key) {
      index++
    }
    if (this.table[index] !== void 0 && this.table[index].key === key) {
      return this.table[index].value
    }
  }
}
...
}
```

## ES6 Map类
[📖说明文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map)
