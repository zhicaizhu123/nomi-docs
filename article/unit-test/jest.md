# Jest

## 特点
- 开箱即用，零配置
- 快
- 内置代码覆盖率
- Mocking很容易

## 断言
```javascript
test('test common matcher', () => {
    expect(2 + 2).toBe(4);
    expect(2 + 2).not.toBe(5);
})

test('test to be true or false', () => {
    expect(1).toBeTruthy()
    expect(0).toBeFalsy()
})

test('test number', () => {
    expect(4).toBeGreaterThan(3)
    expect(2).toBeLessThan(3)
})

test('test object', () => {
    expect({ name: 'Nomi' }).toEqual({ name: 'Nomi' })
})
```

## 支持异步
### 回调方式
```javascript
const fetchUser = (cb) => {
    setTimeout(() => {
          cb('nomi')
    })
}

it('test callback', (done) => {
    fetchUser((data) => {
      expect(data).toBe('nomi')
      expect(data).not.toBe('hello')
      done()
    })
})
```
其中 `done` 是告诉 `jest` 单元测试已经完成。

### `Promise` 方式
```javascript
const userPromise = () => Promise.resolve({ username: 'nomi' })
it('test promise', () => {
    return userPromise().then(data => {
      expect(data).toEqual({ username: 'nomi' })
    })
})
```
⚠️注意：需要返回一个 `Promise` 对象

### `async/await` 方式
```javascript
const userPromise = () => Promise.resolve({ username: 'nomi' })
it('test promise', async () => {
    const data = await userPromise()
    expect(data).toEqual({ username: 'nomi' })
})
```

### `expect`方式

```javascript
const userPromise = () => Promise.resolve({ username: 'nomi' })
it('test promise', async () => {
    return expect(userPromise()).resolves.toEqual({ username: 'nomi' })
})


const userPromise2 = () => Promise.reject('error')
it('test promise', async () => {
    return expect(userPromise2()).rejects.toEqual('error')
})
```
resolves： 为决策结果，rejects：为异常结果。
   
## 实现Mock
为什么需要Mock?
- 前端需要有网络请求。
- 后端依赖数据库等模块。
- 局限性：依赖其他的模块。

Mock 解决方案
- 测试替换，将真实代码替换成替代代码。

Mock 两大功能
- 创建 mock function，在测试中使用，用来测试回调。
- 手动 mock ，覆盖第三方实现。

### 创建mock function
> 使用`jest.fn()`创建mock function

- 默认参数

```javascript
function mockTest(shouldCall, cb) {
    shouldCall && cb('nomi')
}

it('test with mock function', () => {
    const mockCb = jest.fn()
    mockTest(true, mockCb)
    expect(mockCb).toHaveBeenCalled()
    expect(mockCb).toHaveBeenCalledWith('nomi')
    expect(mockCb).toBeCalledTimes(1)
})
```

- 自定义参数

```javascript
it('test mock with implementation', () => {
    const mockCb = jest.fn(x => 'hello ' + x)
    mockTest(true, mockCb)
    expect(mockCb).toHaveBeenCalled()
    expect(mockCb).toBeCalledTimes(1)
    console.log(mockCb.mock.results)
})
```

### Mock 第三方模块实现
> 下面以 axios 举例

覆盖第三方模块的实现
```javascript
const axios = require('axios')
jest.mock('axios')
// 使用mock来模拟axios.get实现
// axios.get.mockImplementation(() => {
//   return Promise.resolve({ data: { username: 'nomi' } })
// })

// 使用mock来模拟axios.get返回数据
// axios.get.mockReturnValue(Promise.resolve({ data: { username: 'nomi' } }))

// 使用mock来模拟axios.get返回的Promise的决策值
axios.get.mockResolvedValue({ data: { username: 'nomi' } })
// 获取用户信息
function getUserName(id) {
    return axios.get(`https://jsonplaceholder.typicode.com/users/${id}`).then((res) => {
      return res.data.username
    })
}

it('test with mock modules', () => {
    return getUserName(1).then(name => {
      console.log(name)
    })
})
```
jest mock 提供了一系列的方法：
- `mockImplementation`：模拟实现
- `mockReturnValue`：模拟响应的Promise
- `mockResolvedValue`：模拟响应的Promise的决策值
- 更多方法可以查看[mock function](https://jestjs.io/docs/mock-functions)

另外，我们还是在根目录下创建 `__mocks__`目录，里面创建我们要模拟的第三方模块文件，例如我们需要mock axios，那么文件名就是`axios.js`。
```
|-- __mocks__
    |-- axios.js  
```
以下模拟一下axios get方法
```javascript
const axios = {
    get: jest.fn(() => Promise.resolve({ data: {  username: 'nomi' } })),
}

module.exports = axios
```
单元测试如下
```javascript
// 获取用户信息
function getUserName(id) {
    return axios.get(`https://jsonplaceholder.typicode.com/users/${id}`).then((res) => {
      return res.data.username
    })
}

it('test with mock modules', () => {
  return getUserName(1).then(name => {
    console.log(name)
  })
})
```
所以使用__mock__的方式可以让我们的单元测试代码简洁舒服很多。

### Timers Mock
> Jest Timers Mock 可以处理setTimeout、setInterval...进行特殊处理，将实现和测试分离开来

**三大API完成不同颗粒度的时间控制**
- [jest.runAllTimers](https://jestjs.io/docs/timer-mocks#run-all-timers)：完成所有定时器
- [jest.runOnlyPendingTimers](https://jestjs.io/docs/timer-mocks#run-pending-timers)：只执行未完成的定时器
- [jest.advanceTimersByTime](https://jestjs.io/docs/timer-mocks#advance-timers-by-time)：控制前进时间


**使用方式**
- 开启Timers Mock
```javascript
jest.useFakeTimers()
```

- jest.runAllTimers
> 一次性执行全部定时器
```javascript
const fetchUser = (cb) => {
  setTimeout(() => {
    setTimeout(() => {
        cb('nomi')
    }, 1000)
  }, 1000)
}
it('test runOnlyPendingTimers', () => {
    const callback = jest.fn()
    fetchUser(callback)
    expect(callback).not.toHaveBeenCalled()
    expect(callback).toHaveBeenCalledTimes(0)
    jest.runOnlyPendingTimers()
    expect(callback).toHaveBeenCalled()
    expect(callback).toHaveBeenCalledTimes(1)
})
```

- jest.runOnlyPendingTimers
> 一步步执行定时器
```javascript
it('test runOnlyPendingTimers', () => {
    const callback = jest.fn()
    fetchUser2(callback)
    expect(callback).not.toHaveBeenCalled()
    expect(callback).toHaveBeenCalledTimes(0)
    jest.runOnlyPendingTimers()
    expect(callback).toHaveBeenCalledWith('nomi 1')
    expect(callback).toHaveBeenCalledTimes(1)
    jest.runOnlyPendingTimers()
    expect(callback).toHaveBeenCalledWith('nomi 2')
    expect(callback).toHaveBeenCalledTimes(2)
})
```

- jest.advanceTimersByTime
> 自定义前进的时间
```javascript
it('test advanceTimersByTime', () => {
    const callback = jest.fn()
    fetchUser2(callback)
    expect(callback).not.toHaveBeenCalled()
    expect(callback).toHaveBeenCalledTimes(0)
    jest.advanceTimersByTime(500)
    jest.advanceTimersByTime(500)
    expect(callback).toHaveBeenCalledWith('nomi 1')
    expect(callback).toHaveBeenCalledTimes(1)
    jest.advanceTimersByTime(800)
      jest.advanceTimersByTime(200)
    expect(callback).toHaveBeenCalledWith('nomi 2')
    expect(callback).toHaveBeenCalledTimes(2)
})
```





