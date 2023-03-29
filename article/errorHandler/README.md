# 异常采集

## 前端异常类型
> 在操作页面的过程中，触发的执行异常或加载异常，浏览器所抛出的报错信息

我们常遇见的异常可能有变量未声明、请求后端接口404，500、资源加载失败等。所以对于前端而言，异常主要分为两大类：前端异常和接口异常

## 捕获异常方式

### try...catch...
> 只能捕获同步代码运行的异常，无法捕获语法错误和异步错误。
```typescript
try {
	// 执行代码
} catch(err) {
	// 异常处理
}
```

### window.onerrror
> 捕获 `Javascript` 运行的同步错误和异步运行错误，无法捕获语法错误、静态资源异常、接口请求异常。

```typescript
/**
*
* @param message 错误信息文本
* @param source 错误文件
* @param lineNo 错误行数
* @param colNo 错误列数
* @param error 错误信息
*/
window.onerror = function (message: string | Event, source?: string, lineNo?: number, colNo?: number, error?: Error | undefined) {
	console.log('捕获异常', { message, source, lineNo, colNo, error })
	// 阻止异常向上抛出
	return true
}
```
`window.onerror` 在捕获到错误时，会向上抛出，为了使异常不向上抛出，可以在函数中返回 `true`。

### window.addEventListener('error')
> 捕获 `Javascript` 运行的同步错误、异步运行错误、静态资源异常和接口请求异常无法捕获 `Promise` 和 `async/await` 产生的未处理的异常。

```typescript
window.addEventListener('error', function (e: ErrorEvent) {
	if (e.target !== window) {
		// 静态资源加载错误
	} else {
		// 其他错误
	}
}, true)
```
因为资源加载的异常只会在当前元素触发，异常不会冒泡到 `window`，因此监听 `window` 上的异常是捕捉不到，既然冒泡阶段监听不到，我们可以在捕获阶段捕获。所以我们需要在 ` window.addEventListener` 函数添加第三个参数，设置为 `true` 表示监听函数会在捕获阶段执行。

### window.addEventListener('unhandledrejection')
> 捕获未处理的 Promise 异常。

```typescript
window.addEventListener('unhandledrejection', function (error: PromiseRejectionEvent) {
	// 打印异常原因
	console.log(error.reason);
	console.log(error);
	// 阻止控制台打印
	error.preventDefault();
});
```
默认情况下，`Promise` 发生异常且未被 `catch` 时，会在控制台打印异常。如果我们想阻止异常打印，可以用上面的 `error.preventDefault()` 方法。

## 异常处理
首先我们需要提供一个异常处理器 `errorHandler` ，所有的异常和上报逻辑都在这个函数处理。
```typescript
enum ErrorType {
	// 前端异常
	normal,
	// 接口异常
	api,
}

function errorHandler(error: any, type: ErrorType) {
	errorHandlers[type](error)
}
```
其中 `errorHanlders` 是包含不同异常处理器的 `hash` 结构
```typescript
const errorHandlers = {
	// 前端异常处理器
	[ErrorType.normal]: normalErrorHandler,
	// 接口异常处理器
	[Error.api]: apiErrorHandler
}
```
### 前端异常处理器
```typescript
interface NormalErrorInfo {
	// 出现错误所属的页面链接，只有在浏览器才收集
	url?: string
	// 错误类型
	type: string
	// 错误信息
	error: string
}

function normalErrorHandler(error: any) {
	const report = {} as NormalErrorInfo
	if (typeof window === 'undefined') {
		// 浏览器平台
		report.url = location.href
	}
	if (error instanceof Error) {
		// 常规错误
		const { name, message } = error
		report.type = name
		report.error = message
	} else {
		// 非常规错误
		report.type = 'OtherError'
		report.error = JSON.stringify(error)
	}
	console.log('report', error)
}
```

### 接口异常处理器
```typescript
interface ApiErrorInfo {
	// 请求接口完整path
	url: string
	// 请求方法，GET, POST, PUT, DELETE...
	method: string
	// GET, DELETE...请求参数
	params?: Record<string, any>,
	// POST, PUT...请求参数
	body?: Record<string, any>,
	// 错误信息
	error: string
}

function apiErrorHandler(error: ApiErrorInfo) {
	console.log('report', error)
}
```

