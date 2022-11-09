# Typescript规范

## 命名规范

### interface接口

> `interface`以`XxxYyyModel`方式命名

```typescript
interface UserInfoModel {
	/**
	 * 用户id
     */
	id: string;
	/**
	 * 用户名称
     */
	name: string;
	/**
	 * 性别
     */
	gender: 0 | 1;
	...
}
```

### type类型别名

> `type`以`XxxYyyType`方式命名

```typescript
/**
 * 激活类型
 */
type EnableType = 0 | 1;
```

### enum枚举

> `enum`以`XxxYyyEnum`方式命名

```typescript
enum GenderEnum {
	/**
	 * 未知
     */
	unknown,
	/**
	 * 女
     */
	female,
	/**
	 * 男
     */
	male,
}
```

### class类

> class以`XxxYyy`方式命名

```typescript
class Point {
	x: number
	y: number
}
```



## 注释规范

统一使用`JSDoc comments`风格，安装VsCode插件Document This ，快捷键`Ctrl + Alt + D` 后再重新`Ctrl + Alt  + D` 为插入符号在或在其中的内容生成注释，这么做有以下优点：

- **降低成员阅读成本**；

- **编辑器友好提示**；

- **便于生成文档；**

🌰 ：

```typescript
/**
 * 员工列表元素信息
 */
interface StaffItemModel {
  /**
   * 企业id
   */
  companyId: string;
  /**
   * 部门id
   */
  departmentId: string;
  /**
   * 员工id
   */
  staffId: string;
  /**
   * 员工名称
   */
  staffName: string;
}

```



## 如何选择

### interface 和 type

1. **类型别名更通用（接口只能声明对象，不能重命名基本类型）**

类型别名的右边可以是任何类型，包括基本类型、元祖、类型表达式（`&`或`|`等类型运算符）；而在接口声明中，右边必须为结构；

```typescript
type A = number
type B = A | string
```

1. **扩展时表现不同**

扩展接口时，TS将检查扩展的接口是否可以赋值给被扩展的接口。🌰 ：

```typescript
interface A {
    good(x: number): string,
    bad(x: number): string
}
interface B extends A {
    good(x: string | number) : string,
    bad(x: number): number // Interface 'B' incorrectly extends interface 'A'.
                           // Types of property 'bad' are incompatible.
                           // Type '(x: number) => number' is not assignable to type '(x: number) => string'.
                           // Type 'number' is not assignable to type 'string'.
}
```

但使用交集类型时则不会出现这种情况。我们将上述代码中的接口改写成类型别名，把 `extends` 换成交集运算符 `&`，TS将尽其所能把扩展和被扩展的类型组合在一起，而不会抛出编译时错误。

```typescript
type A = {
    good(x: number): string,
    bad(x: number): string
}
type B = A & {
     good(x: string | number) : string,
     bad(x: number): number 
}
```

1. **多次定义时表现不同**

接口可以定义多次，多次的声明会合并。但是类型别名如果定义多次，会报错。

```typescript
interface Point {
    x: number
}
interface Point {
    y: number
}
const point: Point = { x:1 } // Property 'y' is missing in type '{ x: number; }' but required in type 'Point'.

const point: Point = { x:1, y:1 } // 正确

```

```typescript
type Point = {
    x: number // Duplicate identifier 'Point'.
}

type Point = {
    y: number // Duplicate identifier 'Point'.
}

```

1. **到底应该用哪个**

如果接口和类型别名都能满足的情况下，到底应该用哪个是我们关心的问题。感觉哪个都可以，但是强烈建议大家只要能用接口实现的就优先使用接口，接口满足不了的再用类型别名。

为什么会这么建议呢？其实在TS的wiki中有说明。具体的地址在[__这里__](https://github.com/microsoft/TypeScript/wiki/Performance#writing-easy-to-compile-code)

## interface 和 class

1. **接口只声明成员不做实现，而类声明并实现方法**

```typescript
interface ContentInterface {
    getContent(): string;
}

class Article implements ContentInterface {
    // 必须实现getContent方法
    public function getContent(): string {
        return 'I am an article.';
    } 
}

class News implements ContentInterface {
    // 没有实现getContent方法，编译器会报错
}
```

1. **到底应该用哪个**

如果要声明并实现的话可以结合类和接口声明完成，如果只是单纯的声明则推荐使用接口声明

### type 和 enum

1. **类型别名用来约束数据类型(数据结构，数据属性且可以是联合类型，元组类型)**

以下🌰 表示的就是性别的类型是0或1或2，而不是值是0或1或2

```typescript
type GenderType = 0 | 1 | 2
```

1. **枚举用来约束数据类型的值，而并非数据类型**

以下🌰 表示的就是性别的值是0或1或2，而不是类型是0或1或2

```typescript
enum GenderEnum {
	unknown, // 不指定默认值时从0开始递增
	male,
	female,
}
```

1. **到底应该选哪个**

如果我们需要用到定义的类型值的话，我们可以使用枚举，否则两者都可以，不过为了便于后期的扩展性，强烈推荐使用枚举，拿上面使用type和enum声明的性别来说，如果我们需要拓展性别列表或者性别的key-value格式对象或者用于比较，type是不能做到的，但是enum可以

```typescript
enum GenderEnum {
	unknown, // 不指定默认值时从0开始递增
	male,
	female,
}

// 拓展列表
const genderOptions = [
	{
		label: '未知',
		value: GenderEnum.unknown,
	},
	{
		label: '男',
		value: GenderEnum.male,
	},
	{
		label: '女',
		value: GenderEnum.female,
	},
]

// 拓展对象
const genderMap = {
	[GenderEnum.unknown]: '未知',
	[GenderEnum.male]: '男',
	[GenderEnum.female]: '女',
}

// 拓展比较
const isMale = gender === GenderEnum.male
```



## 全局配置

全局接口配置在`.d.ts` ，不同项目存放目录可能不一致，开发项目前优先熟悉这些全局接口避免后期重复造轮子。



## 技巧

### 继承

主要用于抽离公用的有逻辑或业务相关接口声明

如果两个接口直接存在相同的字段声明，但是它们直接是没有任何业务联系的，如果硬生生的抽离通用接口，如果后期某个接口的字段类型改变了就会增加后期维护成本

正🌰 ：封装分页逻辑的接口字段

```typescript
interface ResponsePaginationModel {
  /**
   * 当前页码
   */
  currentPage: number;
  /**
   * 页数
   */
  pageCount: number;
  /**
   * 每页条数
   */
  pageSize: number;
  /**
   * 总数
   */
  total: number;
}


interface ResponsePageListModel<T> extends ResponsePaginationModel {
	/**
	 * 数据列表
	 */
	items: T[];
}
```

反🌰 ：只是通过相同字段名称进行抽离

```typescript
interface CommanModel {
	id: number;
	name: string;
}

/**
 * 用户信息接口
 */
interface UserInfoModel extends CommonModel {
	gender: GenderType
	...
}

/**
 * 部门信息接口
 */

interface DeptModel extends CommonModel {
	parentId: number
	...
}

```

从反🌰 可以看出，其实id和name在用户信息和部门信息代表的是不同的含义，只是它们的字段名称是一样的而已，除此之外没有任何联系。如果后期用户信息的id 类型改成 string，或者说字段名都改了这样部门信息也会收到影响

### 范型

> 复用相同结构接口，类型别名，类，函数等

- 分页响应数据接口：

```typescript
/**
 * 分页响应数据
 */
interface ResponsePageListModel<T> extends ResponsePaginationModel {
	/**
	 * 数据列表
	 */
	items: T[];
}

```

对于分页响应数据，我们的数据结构是一致的，如下所示：

```typescript
{
	currentPage: 1,
	pageSize: 10,
	pageCount: 1,
	total: 100,
	items: [],
}
```

不同的是items数组元素值和结构不一样，所以我们可以通过范型来自定义不同类型的元素

```typescript
interface ItemModel {
	label: string
	value: string
}

type ListType = ResponsePageListModel<ItemModel>
```



- 表格元素信息接口：

```typescript
interface FieldColumnItemModel<T = any> {
  /**
   * 字段key
   */
  prop: string;
  /**
   * 字段label
   */
  label?: string | Fn<T>;
  /**
   * label 插槽
   */
  labelSlot?: string;
  /**
   * 格式化函数
   */
  format?: Fn<T, string | number> | string;
  /**
   * 内容插槽
   */
  slot?: string;
  /**
   * 栅格化配置
   */
  span?: number;
  /**
   * label文本对齐方式
   */
  labelAlign?: 'left' | 'right';
}


```

对于表格元素接口，对于不同的列字段我们需要自定义返回的数据，这个时候需要依赖实际的每个的数据，所以我们可以通过范型定制不同的接口。

```typescript
/**
 * 搜索线索列表
 */
export interface SearchLeadItemModel {
  /**
   * 跟进人id
   */
  staffId: string;
  /**
   * 跟进人姓名
   */
  staffName: string;
  /**
   * 广告渠道
   */
  channels: ItemOption[];
  /**
   * 创建时间
   */
  createDt: string;
  /**
   * 线索id
   */
  id: string;
  /**
   * 线索昵称
   */
  leadName: string;
  /**
   * 手机号
   */
  mobile: string;
  /**
   * 姓名
   */
  name: string;
  /**
   * 昵称
   */
  nickname: string;
  /**
   * 意向项目ID
   */
  productId: number;
  /**
   * 意向项目
   */
  productName: string;
  /**
   * 来源url
   */
  url: string;
  /**
   * 微信昵称
   */
  wxNickname: string;
  /**
   * 是否绑定
   */
  bindUnion: boolean;
  /**
   * unionId
   */
  unionId: string;
}



const columns: FieldColumnItemModel<SearchLeadItemModel>[] = [
  {
    label: 'ID',
    prop: 'id',
  },
  {
    label: '意向项目',
    prop: 'productName',
  },
  {
    label: '创建时间',
    prop: 'createdDt',
    format: 'date|YYYY-MM-DD HH:mm',
  },
  {
    label: '线索昵称',
    prop: 'wxNickname',
    format: (record) => record.wxNickname || record.nickname || record.name || record.mobile,
  },
  {
    label: '手机号',
    prop: 'mobile',
    slot: 'mobile',
  },
  {
    label: '广告渠道',
    prop: 'channels',
    format: (record) => record.channels?.map((item) => item.name).join('，'),
  },
  {
    label: '来源URL',
    prop: 'url',
    slot: 'url',
  },
];

```

上述的`format`函数的参数`record`对应的数据结构就是`SearchLeadItemModel`接口声明的结构了，对于代码校验和编辑器的提示都会非常友好。

### 常用内置接口

#### Partial<T>

将类型 T 的所有属性标记为可选属性

🌰 ：

```typescript
interface UserInfoModel {
  name: string 
  email: string 
  age: number 
}

type BaseInfoType = Partial<UserInfoModel>

/* 
结果：
{ 
  name?: string
  email?: stirng
  age?: number
}
*/
```

#### Required<T>

与 Partial 相反，Required 将类型 T 的所有属性标记为必选属性

🌰 ：

```typescript
interface UserInfoModel {
  name?: string 
  email?: string 
  age?: number 
}

type BaseInfoType = Required<UserInfoModel>
/* 
结果：
{ 
  name: string
  email: stirng
  age: number
}
*/

```

#### Readonly<T>

将所有属性标记为 readonly, 即不能修改

🌰 ：

```typescript
interface UserInfoModel {
  name: string 
  email: string 
  age: number 
}

type BaseInfoType = Required<UserInfoModel>
/* 
结果：
{ 
  readonly name: string
  readonly email: stirng
  readonly age: number
}
*/

```

#### Pick<T, K>

从 T 中过滤出属性 K

🌰 ：

```typescript
interface UserInfoModel {
  name: string 
  email: string 
  age: number 
}

type BaseInfoType = Pick<UserInfoModel, 'name' | 'email'>

/* 
结果：
{ 
  name: string
  email: stirng
}
*/
```

#### Record<K, T>

标记对象的 key value类型

🌰 ：

```typescript
// 定义 学号(key)-账号信息(value) 的对象
const accountMapModel: Record<number, UserInfoModel> = {
  10001: {
    name: 'xx',
    email: 'xxxxx',
    // ...
  }    
}
const user: Record<keyof UserInfoModel, string> = {
    name: '', 
    email: ''
}
```

#### Exclude<T, U>

移除 T 中的 U 属性

🌰 ：

```typescript
// 'a' | 'd'
type A = Exclude<'a'|'b'|'c'|'d' ,'b'|'c'|'e' >  
```

#### Omit<T, K>

移除 T 中的 K 属性

🌰 ：

```typescript
interface UserInfoModel {
  name: string 
  email: string 
  age: number 
}

type BaseInfoType = Omit<UserInfoModel, 'age'>

/* 
结果：
{ 
  name: string
  email: stirng
}
*/
```

#### Extract<T, U>

Exclude 的反操作，取 T，U两者的交集属性

🌰 ：

```typescript
// 'b'|'c'
type A = Extract<'a'|'b'|'c'|'d' ,'b'|'c'|'e' >  
```

#### NonNullable<T>

排除类型 T 的 `null` | `undefined` 属性

🌰 ：

```typescript
type A = string | number | undefined 
type B = NonNullable<A> // string | number
```

#### ReturnType<T>

获取函数类型 T 的返回类型

🌰 ：

```typescript
type TimeoutHandle = ReturnType<typeof setTimeout>;
type IntervalHandle = ReturnType<typeof setInterval>;

const timerId: TimeoutHandle = setTimeout(() => {})
const intervalId: IntervalHandle = setTimeout(() => {})
```

### 快速生成TS

- vscode安装`json2ts` ，复制json内容，然后快捷粘贴：`cmd+alt+V` 或 `ctrl+alt+V`

- 使用Apifox工具，快速生成Typescript，具体操作可参考Apifox文档



