# package.json 字段作用

## name 包名
包名称，如果通过 `npm`/`yarn`/`pnpm` 安装时，就是使用这个字段

## main 模块入口
一般是使用打包成 `UMD`/`CommonJS` 格式的入口文件

## types 类型文件入口
定义 Typescript 类型声明入口文件

## module ESM 模块入口
它是 `rollup` 中最早就提出的概念 — `pkg.module`。在这之前 `npm` 包大都是基于 `CommonJS` 规范的。当我们当 `require` 引入包的时候，就会根据 `main` 字段去查找入口文件

在 `ES6` 规范出现后，`ES6` 定义了一套基于 `import`、`export` 操作符的模块规范。它与 `CommonJS` 规范最大的区别在于 `ES6` 中的 `import` 和 `export` 都是静态的。静态意味着一个模块要暴露或引入的所有方法在编译阶段就能全部确定，之后不能再改变。这样做的好处就是打包工具在打包阶段就可以分析出代码中用到了某个模块中的哪几个方法。其它没有用到的方法就可以从最终的 `bundle` 文件中剔除掉。这样既可以减少 `bundle` 文件的大小，又可以提高脚本的执行速度。这个机制被称为 `Tree Shaking`。在这个构建思想的基础上，开发基于 `ES Module` 规范的包是很有必要的。

之前我们说过 `CommonJS` 规范的包都是以 `main` 字段表示入口文件了，如果 `ES Module` 的也用 `main` 字段，就会对使用者造成困扰，如果他的项目不支持打包构建，比如大多数 `node` 项目(尽管 node9+ 支持 ES Module)，这时库开发者的模块系统跟项目构建的模块系统的冲突，更像是一种规范上的问题。况且目前大部分仍是采用 `CommonJS`，所以 rollup 便使用了另一个字段：`module`。

`webpack` 从版本 2 开始也可以识别 `pkg.module` 字段。打包时，如果存在 `module` 字段，会优先使用，如果没找到对应的文件，则使用 `main` 字段，并按照 `CommonJS` 规范打包。所以目前主流的打包工具（`webpack`、`rollup`）都是支持 `pkg.module` 的，鉴于其优点，`module` 字段很有可能加入 `package.json` 的规范之中。另外，越来越多的 `npm` 包已经同时支持两种模块，使用者可以根据情况自行选择，并且实现也比较简单，只是模块导出的方式。

## bin 命令行入口文件
很多包都有一个或多个可执行的文件希望被放到`PATH`中。npm让妈妈再也不用担心了（实际上，就是这个功能让`npm`可执行的）。

## 依赖的分类
- `dependencies`
  - 运行项目业务逻辑需要依赖的第三方库
  - `npm install '模块名称'` 的时候都会被解析，下载
- `devDependencies`
  - 开发模式工作流下依赖的第三方库
  - 单元测试、语法转换、lint工具、程序构建、本地开发等等
- `peerDependencies`
  - 需要核心依赖库，不能脱离依赖库单独使用

## version 版本号
> 遵循语义化版本 `semver`

版本格式：主版本号.次版本号.修订版本号（1.0.0），版本号递增规则如下：
- 主版本号：当你做了不兼容的API修改；
- 次版本号：当你做了向下兼容的功能性新增；
- 修订号：当你做了向下兼容的问题修复；

## files 字段
指定把什么文件上传到 `npm` 上

- 默认忽略掉 `.gitignore`和`.npmignore` 中的内容；
- 指示 `npm publish` 的时候需要上传的内容；
- `package.json` /` README.md` / `CHANGELOG.md` / `LICENSE` 都会包含在其中；
- 优先级：`files` > `.npmignore` > `.gitignore`

## publishConfig
这是一个在 `publish-time` 使用的配置集合。当你想设置 `tag` 或者 `registry` 的时候它非常有用，所以你可以确定一个给定的包没有打上 `"lastest"` 的 `tag` 或者被默认发布到全局的公开`registry`。

任何配置都可以被重写，但当然可能只有 `tag` 和 `registry` 与发布的意图有关。

## engines
指定工作的node的版本
```json
{ 
    "engines": { 
        "node": ">=14.0.0 <17.0.0"
    } 
}
```
也可以用来指定哪一个 `npm` 版本能更好地初始化程序
```json
{ 
    "engines": { 
        "npm": "~8.15.0"
    } 
}
```

## workspace
`workspace` 是除缓存外 `yarn` 区别于 `npm` 最大的优势。

- `workspace` 的作用
  - 能帮助你更好地管理多个子project的repo，这样你可以在每个子project里使用独立的package.json管理你的依赖，又不用分别进到每一个子project里去`yarn install`/`upfrade`安装/升级依赖，而是使用一条 `yarn` 命令去处理所有依赖就像只有一个 `package.json` 一样。通过这样设置后，会把所有的依赖提升到顶层的 `node_modules` 中。
  - yarn会根据就依赖关系帮助你分析所有子project的共用依赖，保证所有的project公用的依赖只会被下载和安装一次。
- `workspace` 的使用
  - `yarn workspace` 并不需要安装什么其他的包，只需要简单的更改 `package.json` 便可以工作。 首先我们需要确定` workspace root`，一般来说 `workspace root` 都会是repo的根目录。

根目录 package.json
```json
{
    //当private为true时workspace才会被启用
    "private": true,
    "workspace": ["workspace-a"]
} 
```

子包目录package.json
```json
{
    "name": "workspace-a",
    "version": "1.0.0",
    "dependencies": {
        "cross-env": "5.0.5"
    }
} 
```
