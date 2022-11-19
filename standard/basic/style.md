# SCSS/LESS/CSS规范

## charset

【推荐】样式文件写上 `@charset` 规则，并且一定要在样式文件的第一行首个字符位置开始写，编码名用 “**UTF-8**”

```css
@charset "UTF-8";
...
```

## BEM 命名规范

【推荐】BEM 的名称来源于该方法学的三个组成部分的英文首字母，分别是块（Block）、元素（Element）和修饰符（Modifier）。关于`CSS BEM` 可以查看[官方文档](https://en.bem.info/methodology/css/)

格式：`block-name__element-name--modifier`

HTML：

```html
<!-- 'search-form' 模块 -->
<form class="search-form">
  <!-- 在 'search-form' 模块内的 'input' 元素 -->
  <input class="search-form__input" />
  <!-- 在 'search-form' 模块内的 'button' 元素 -->
  <button class="search-form__button"></button>
  <!-- 在 'search-form' 模块内的 'footer' 元素 -->
  <footer class="search-form__footer">
    <!-- 在 'search-form__footer' 元素内有 'submit' 操作 -->
  	<button class="search-form__footer--submit"></button>
  </footer>
</form>
```

SCSS/LESS样式：

```css
.search-form {
	&__input {

  }
  &__button {
  
  }
  &__footer {
    &--submit {
    
    }
  }
}  
```

## 代码风格

【强制】统一使用两个空格进行代码缩进

【强制】左括号与类名之间一个空格，冒号与属性值之间一个空格

【强制】`css`属性值需要用到引号时，统一使用单引号

【推荐】为单个`css`选择器或新申明开启新行

【推荐】属性值十六进制数值能用简写的尽量用简写

【推荐】颜色值 `rgb()` `rgba()` `hsl()` `hsla()` `rect()` 取值不要带有不必要的 `0`

【推荐】不要为 `0` 指明单位

反例：

```css
.main,.content{
  width:100px;
  height:200px;
  color:#000000;
  background-color:rgba(255,255,255,0.5);
  margin-left:0px;
  &:after{
  	content:"";
    display:block;
  }
}
```

正例：

```css
.main,
.content {
  width: 100px;
  height: 200px;
  color: #000;
  background-color: rgba(255,255,255,.5);
  margin-left: 0;
  &:after{
  	content: '';
    display: block;
  }
}
```

## 注释风格

### 单行注释

【推荐】注释内容第一个字符和最后一个字符都是一个空格字符，单独占一行，行与行之间相隔一行

不推荐：

```css
/*主体内容*/
.main {
	display: block;
}
```

推荐：

```css
/* 主体内容 */
.main {
	display: block;
}
```

### 模块注释

【推荐】注释内容第一个字符和最后一个字符都是一个空格字符，/* 与 模块信息描述占一行，多个横线分隔符-与*/占一行，行与行之间相隔两行

不推荐：

```css
/* Module A ---------------------------------------------------- */
.mod-a {}
/* Module B ---------------------------------------------------- */
.mod-b {}
```

推荐：

```css
/* Module A
---------------------------------------------------------------- */
.mod-a {}


/* Module B
---------------------------------------------------------------- */
.mod-b {}
```



