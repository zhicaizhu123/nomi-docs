# TS/JS规范

## 变量

【强制】命名方法: 小驼峰式命名法 命名规范：前缀为形容词 （函数前缀为动词, 以此来区分函数和变量）

反例：

```typescript
let setConut = 10;
let getTitle = '表格标题';
```

正例：

```typescript
let maxCount = 10;
let tableTitle = '表格标题';
```

**说明：这里的变量包含块作用域和函数作用域中定义的常量**

## 常量

【强制】命名方法：名词全部大写 命名规范：使用大写字母和下划线来组合命名，下划线用来分割单词。

反例：

```text
const maxCount = 10;
const homeUrl = 'https://www.lingshi.com';
```

正例：

```typescript
const MAX_COUNT = 10;
const HOME_URL = 'https://www.lingshi.com';
```

**说明：这里的常量指在全局或者某个模块类使用，不包含块作用域和函数作用域中定义的常量**

## 函数 & 方法

【强制】命名方法： 小驼峰式命名法 命名规范： 前缀应该为动词 命名建议：常用动词约定

```typescript
// 是否可阅读
function canRead() {}
// 获取名称
function getName() {}
// 设置名称
function setName() {}
```

## 类（ES6） & 构造函数（ES6之前）

【强制】命名方法：大写驼峰式命名法，首字母大写。 命名规范：前缀为名称。

```typescript
// ES6 类
class Person {
  constructor(name) {
	this.name = name
  }

  getName() {
	return this.name
  }
}
const person = new Person('lingshi admin');

// ES6之前构造函数
function Animal(type) {
	this.type = type
}
Animal.prototype.getType = function () {
  return this.type
}
const animal = new Animal('cat')
```

## 函数参数默认值

【推荐】函数参数设置默认值在参数列表完成，可提高代码可读性

不推荐：

```typescript
function initData(name, sex) {
	sex = sex || 'male'
	...
}

function initInfo(params) {
	const id = params.id || ''
    const name = params.sex || 'male'
	...
}

```

推荐：

```typescript
function initData(name, sex = 'male') {
	...
}

function initInfo({id = '', sex = 'mail'} = {}) {
	...
}

```

## 解构赋值

【推荐】使用`...` 操作符对数据进行提取或者合并

不推荐：

```typescript
// 数组
const position = [1, 2]
const x = position[0]
const y = position[1]

// 对象
const userInfo = {
  id: 1,
  name: 'lingshi',
  sex: 'male',
}
const userInfoWithoutId = {
  name: userInfo.name,
  sex: userInfo.sex,
}
const userFullInfo = userInfo
userInfo.email = 'lingshi@xxx.com'
userInfo.address = 'Guang Zhou'

```

推荐：

```typescript
// 数组
const position = [1, 2]
const [x, y] = position

// 对象
const userInfo = {
  id: 1,
  name: 'lingshi',
  sex: 'male',
}
const {id, ...userInfoWithoutId} = userInfo
const newUserInfo = {
	...userInfo,
	sex: 'female',
	email: 'lingshi@xxx.com',
	address: 'Guang Zhou'
}   
```

## 避免魔法数字或字符串

【强制】在编程的领域中指莫名其妙出现的数字，其意义需要通过详细的阅读代码才能理解。一般魔法数字都是需要使用枚举变量来替换的。

反例：

```typescript
const worksheetStatus = 1;
const homeworkStatus = 'completed'
```

正例：

```typescript
enum WorksheetStatusEnum {
 	unstart = 1;
  ...
}

enum HomeworkStatusEnum {
 	completed = 'completed';
  ...
}

const worksheetStatus: WorksheetStatusEnum = WorksheetStatusEnum.unstart;
const homeworkStatus: HomeworkStatusEnum = HomeworkStatusEnum.completed
```

## 注释风格

### 单行注释

【推荐】使用`// xxx` 或则 `/** xxx */`，注意其中`//`后有一个空格，`/** xxx */` 注释文本xxx前后各有一个空格

```typescript
function handleUser(userInfo: UserInfo) {
  // 待处理姓名
	const name = userInfo.name || ''
  /** 待处理邮箱 */
  const email = userInfo.email || ''
}
```

### 多行注释

```text
/**
 * 这是多行注释
 */
```

### 函数 & 方法注释

【强制】函数 & 方法如果需要注释则使用[JSDoc](https://jsdoc.app/about-getting-started.html)或[TSDoc](https://tsdoc.org/) 风格注释，便于编辑提示和自动生成说明文档

格式：

```typescript
/**
 * 方法名描述
 *
 * @param {DataType} info 参数说明
 */
method(info: DataType) {

}
```

例子：

```typescript
/**
 * 处理用户信息
 *
 * @param {UserInfo} userInfo 用户信息
 */
function handleUser(userInfo: UserInfo) {
  // 待处理姓名
	const name = userInfo.name || ''
  /** 待处理邮箱 */
  const email = userInfo.email || ''
}
```

### 接口注释

【强制】接口注释使用[TSDoc](https://tsdoc.org/)风格注释，便于编辑器回显

反例：

```typescript
interface TransitionSetting {
  // 是否打开切换页面的动画
  enable: boolean;
  // 路由基本切换动画
  basicTransition: RouterTransitionEnum;
  // 是否打开页面切换加载
  openPageLoading: boolean;
  // 是否打开顶部进度条
  openNProgress: boolean;
}
```

正例：

```typescript
interface TransitionSetting {
  /**
   * 是否打开切换页面的动画
   */
  enable: boolean;
  /**
   * 路由基本切换动画
   */
  basicTransition: RouterTransitionEnum;
  /**
   * 是否打开页面切换加载
   */
  openPageLoading: boolean;
  /**
   * 是否打开顶部进度条
   */
  openNProgress: boolean;
}
```



