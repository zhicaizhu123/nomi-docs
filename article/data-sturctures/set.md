# 集合

集合是由一组无序且唯一（即不能重复）的项组成的。该数据结构使用了与有限集合相同的数学概念，但应用在计算机科学的数据结构中。

在数学中，集合也有并集、交集、差集等基本运算。

## 方法
- `add(element)`：向集合添加一个新元素。
- `delete(element)`：从集合移除一个元素。
- `has(element)`：如果元素在集合中，返回true，否则返回false。
- `clear()`：移除集合中的所有元素。
- `size()`：返回集合所包含元素的数量。它与数组的length属性类似。
- `values()`：返回一个包含集合中所有值（元素）的数组。

## 自定义集合结构
```javascript
class Set {
  constructor() {
    this.items = {}
  }

  add(element) {
    if (!this.items[element]) {
      this.items[element] = element
    }
    return false
  }

  delete(element) {
    if (this.has(element)) {
      Reflect.deleteProperty(this.items, element)
      return true
    }
    return false
  }

  has(element) {
    return Object.prototype.hasOwnProperty.call(this.items, element)
  }

  clear() {
    this.items = {}
  }

  size() {
    return Object.keys(this.items).length
  }

  values() {
    return Object.values(this.items)
  }
}
```

### 并集
```javascript
union(otherSet) {
  const unionSet = new Set()
  this.values().forEach(value => unionSet.add(value))
  otherSet.values.forEach(value => unionSet.add(value))
  return unionSet
}

```

### 交集
```javascript
intersection(otherSet) {
  const intersectionSet = new Set()
  const values = this.values()
  const otherValues = otherSet.values()
  let biggerSet = values
  let smallerSet = otherValues
  if (otherValues.length - values.length > 0) {
    biggerSet = otherValues
    smallerSet = values
  }
  smallerSet.forEach(value => {
    if (biggerSet.includes(value)) {
      intersectionSet.add(value)
    }
  });
  return intersectionSet;
}
```

### 差集
```javascript
difference(otherSet) {
  const differenceSet = new Set()
  this.values().forEach(value => {
    if (!otherSet.has(value)) {
      differenceSet.add(value)
    }
  })
  return differenceSet
}
```

### 子集
```javascript
isSubsetOf(otherSet) {
  if (this.size() > otherSet.size()) {
    return false
  }
  const isSubset = this.values().every(value => {
    return otherSet.has(value)
  });
  return isSubset
}
```

## ES6 Set类
[说明文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set)

### 并集
```javascript
const union = (setA, setB) => {
  const unionAb = new Set();
  setA.forEach(value => unionAb.add(value));
  setB.forEach(value => unionAb.add(value));
  return unionAb;
};
```

### 交集
```javascript
const intersection = (setA, setB) => {
  const intersectionSet = new Set();
  setA.forEach(value => {
    if (setB.has(value)) {
      intersectionSet.add(value);
    }
  });
  return intersectionSet;
};
```

### 差集
```javascript
const difference = (setA, setB) => {
  const differenceSet = new Set();
  setA.forEach(value => {
    if (! setB.has(value)) { // {1}
      differenceSet.add(value);
    }
  });
  return differenceSet;
};
```

### 子集
```javascript
const isSubsetOf = (setA, setB) => {
  if (setA.size > setB.size) {
    return false
  }
  const isSubset = [...setA].every(value => {
    return otherSet.has(value)
  });
  return isSubset
}
```