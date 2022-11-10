# HTML规范

## HTML代码大小写

【强制】HTML标签名、类名、标签属性和大部分属性值统一用小写，单词之间使用 `-` 或 `_` 分隔  

反例：

```html
<div class="LayoutContent">...</div>

<div class="Layout-Content">...</div>

<DIV class="LayoutContent">...</DIV>
```

正例：

```html
<div class="layout-content">...</div>
```

## 元素属性

【强制】元素属性值使用**双引号**语法

反例：

```html
<div class='layout-content'>...</div>

<input type='text'>
```

正例：

```html
<div class="layout-content">...</div>

<input type="text">
```

【强制】a标签 `href` 属性不能为空，如果没有给链接可写，强制使用`href="javascript: void(0);"`

## 图片

【推荐】img 标签alt建议不能为空

【推荐】img 标签设置固定宽高时，建议添加样式object-fit属性，确保图片不变形

> contain：被替换的内容将被缩放，以在填充元素的内容框时保持其宽高比。 整个对象在填充盒子的同时保留其长宽比，因此如果宽高比与框的宽高比不匹配，该对象将被添加“[黑边](https://zh.wikipedia.org/wiki/%E9%BB%91%E9%82%8A)”。

> cover：被替换的内容在保持其宽高比的同时填充元素的整个内容框。如果对象的宽高比与内容框不相匹配，该对象将被剪裁以适应内容框。

## 特殊字符使用

【强制】文本可以和字符引用混合出现。这种方法可以用来转义在文本中不能合法出现的字符。

在 HTML 中不能使用小于号 “<” 和大于号 “>”特殊字符，浏览器会将它们作为标签解析，若要正确显示，在 HTML 源代码中使用字符实体。

反例：

```html
<a href="#">more>></a>
```

正例：

```html
<a href="#">more&gt;&gt;</a>
```

HTML常用特殊字符

| HTML 原代码 | 显示结果 | 描述                   |
| ----------- | -------- | ---------------------- |
| \&lt;       | <        | 小于号或显示标记       |
| \&gt;       | >        | 大于号或显示标记       |
| \&amp;      | &        | 可用于显示其它特殊字符 |
| \&quot;     | “        | 引号                   |
| \&reg;      | ®        | 已注册                 |
| \&copy;     | ©        | 版权                   |
| \&trade;    | ™        | 商标                   |
| \&ensp;     |          | 半个空白位             |
| \&emsp;     |          | 一个空白位             |
| \&nbsp;     |          | 不断行的空白           |

## 代码缩进

【强制】统一使用两个空格进行代码缩进（各编辑器有相关配置）

```html
<section class="layout">
  <div class="layout-sider"></div>
  <div class="layout-content"></div>
</section>
```

## 代码嵌套

【推荐】元素嵌套规范，每个块状元素独立一行

不推荐：

```html
<div>
  <h1></h1><p></p>
</div>	
```

推荐：

```html
<div>
  <h1></h1>
  <p></p>
</div>
```

## 文件模板

【参考】移动端端模版

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, shrink-to-fit=no" >
  <meta name="format-detection" content="telephone=no" >
  <title>移动端HTML模版</title>
</head>
<body>

</body>
</html>
```

【参考】PC端模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="keywords" content="your keywords">
  <meta name="description" content="your description">
  <meta http-equiv="X-UA-Compatible" content="IE=Edge,chrome=1">
  <meta name="renderer" content="ie-stand">
	<title>PC端HTML模版</title>
</head>
<body>

</body>
</html>
```

## 注释风格

【强制】必须以4个有序字符开始：编码为 U+003C LESS-THAN SIGN 的小于号, 编码为 U+0021 EXCLAMATION MARK 的感叹号, 编码为 U+002D HYPHEN-MINUS 横线, 编码为 U+002D HYPHEN-MINUS横线 ，即 “<!–”

【强制】在此之后是注释内容，注释的内容有以下限制：

【强制】不能以单个 “>” (U+003E) 字符开始

【强制】不能以由 “-“（U+002D HYPHEN-MINUS）和 ”>” (U+003E) 组合的字符开始，即 “->”

【强制】不能包含两个连续的 U+002D HYPHEN-MINUS 字符，即 “–”

【强制】不能以一个 U+002D HYPHEN-MINUS 字符结束，即 “-”

【强制】必须以3个有序字符结束：U+002D HYPHEN-MINUS, U+002D HYPHEN-MINUS, U+003E GREATER-THAN SIGN，即 “–>”

反例：

```html
<!-->The Wrong Comment Text-->
```

正例：

```html
<!--Comment Text-->
```



