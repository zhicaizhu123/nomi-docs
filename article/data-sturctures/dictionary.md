# 字典

在字典中，存储的是 [键，值] 对，其中键名是用来查询特定元素的。字典和集合很相似，集合以 [值，值] 的形式存储元素，字典则是以 [键，值] 的形式来存储元素。字典也称作映射、符号表或关联数组。

## 方法
- `set(key, value)`：向字典中添加新元素。如果 `key` 已经存在，那么已存在的 `value` 会被新的值覆盖。
- `delete(key)`：通过使用键值作为参数来从字典中移除键值对应的数据值。
- `has(key)`：如果某个键值存在于该字典中，返回 `true` ，否则返回 `false` 。
- `get(key)`：通过以键值作为参数查找特定的数值并返回。
- `clear()`：删除该字典中的所有值。
- `size()`：返回字典所包含值的数量。与数组的 `length` 属性类似。
- `isEmpty()`：在 `size` 等于零的时候返回 `true` ，否则返回 `false` 。
- `keys()`：将字典所包含的所有键名以数组形式返回。
- `values()`：将字典所包含的所有数值以数组形式返回。
- `entries()`：将字典中所有 [键，值] 对返回。
- `forEach(cb)`：迭代字典中所有的键值对。`cb` 有两个参数： `key` 和 `value` 。


## 转化key函数
```javascript
function defaultToStrFn(value) {
  if (value === null) return 'NULL'
  if (value === void 0) return 'UNDEFINED'
  if (typeof value === 'string' || value instanceof String) return `${value}`
  return value.toString()
}
```

## 字典元素存储类
```javascript
class ValuePair {
  constructor(key, value) {
    this.key = key
    this.value = value
  }

  toString() {
    return `[#${this.key}: ${this.value}]`;
  }
}
```

## 字典结构
```javascript
class Dictionary {
  constructor(toStrFn = defaultToStrFn) {
    this.toStrFn = toStrFn
    this.table = {}
  }

  set(key, value) {
    if (key !== null && key !== void 0) {
      const tableKey = this.toStrFn(key)
      this.table[tableKey] = new ValuePair(key, value)
      return true
    }
    return false
  }

  get(key) {
    const data = this.table[this.toStrFn(key)]
    return data === void 0 ? void 0 : data.value
  }

  delete(key) {
    if (this.has(key)) {
      Reflect.deleteProperty(this.table, this.toStrFn(key))
      return true
    }
    return false
  }

  has(key) {
    return this.table[this.toStrFn(key)] !== void 0
  }

  clear() {
    this.table = {}
  }

  isEmpty() {
    return this.size() === 0
  }

  size() {
    return this.entries().length
  }

  keys() {
    return this.entries().map(item => item.key)
  }

  values() {
    return this.entries().map(item => item.value)
  }

  entries() {
    return Object.values(this.table)
  }

  forEach(cb) {
    this.entries.forEach(item => {
      cb(item.key, item.value)
    })
  }
}
```