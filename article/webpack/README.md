# 基础配置

## 5大核心概念
1. entry（入口）
   指定Webpack从哪个文件开始大包，可以支持一个或多个文件入口。
2. output（输出）
   指定Webpack打包完的文件输出到哪里，如何命名等。
3. loader（加载器）
   Webpack本身只能处理js、json等资源，其他资源需要借助loader才能解析。
4. plugins（插件）
   扩展Webpack的功能。
5. mode（模式）
   - 生产环境：development
   - 开发环境：production

## 准备webpack配置文件
在根目录新建`webback.config.js`文件

基本配置：
```javascript
const { resolve } = require('path')

module.exports = {
  // 入口
  entry: resolve(__dirname, '../demo/basic/index.js'),

  // 输出
  output: {
    // 文件输出目录
    path: resolve(__dirname, 'dist'),
    // 文件输出名称
    filename: 'js/[name].[chunkhash:8].js',
  },

  // 加载器
  module: {
    rules: [
      // loader配置
    ]
  },

  // plugins
  plugins: [
    // 插件配置列表
  ],

  // 模式
  mode: 'development'
}
```

## 处理样式资源
> webpack 本身是不识别样式资源的，我们需要借助loader帮助webpack解析样式资源。
> **loader加载顺序为从右到左，从下到上依次处理。**
> [Loader官方文档]()

### 处理CSS资源
1. 安装依赖
  ```bash
  yarn add -D css-loader style-loader
  ```
2. 在相应的文件引入CSS资源
  ```javascript
  import 'path/to/index.css'
  ``` 
3. 配置：
  ```javascript
  module.exports = {
    module: {
      rules: [
        // loader配置
        {
          test: /\.css$/,
          use: [
            'style-loader',
            'css-loader'
          ]
        }
      ]
    }
  }
  ```

### 处理Less资源
1. 安装依赖
  ```bash
  yarn add -D less less-loader css-loader style-loader
  ```
2. 在相应的文件引入CSS资源
  ```javascript
  import 'path/to/index.css'
  ``` 
3. 配置：
  ```javascript
  module.exports = {
    module: {
      rules: [
        // loader配置
        {
          test: /\.less$/,
          use: [
            'style-loader',
            'css-loader',
            'less-loader',
          ]
        }
      ]
    }
  }
  ```

### 处理Sass资源
1. 安装依赖
  ```bash
  yarn add -D sass sass-loader css-loader style-loader
  ```
2. 在相应的文件引入CSS资源
  ```javascript
  import 'path/to/index.css'
  ``` 
3. 配置：
  ```javascript
  module.exports = {
    module: {
      rules: [
        // loader配置
        {
          test: /\.s(c|a)ss$/,
          use: [
            'style-loader',
            'css-loader',
            'sass-loader',
          ]
        }
      ]
    }
  }
  ```

### 提取css文件
1. 安装依赖
  ```bash
  yarn add -D mini-css-extract-plugin
  ```
2. 配置：
  ```javascript
  const MiniCssExtractPlugin = require("mini-css-extract-plugin");

  module.exports = {
    module: {
      rules: [
        // loader配置
        {
          test: /\.s(c|a)ss$/,
          use: [
            MiniCssExtractPlugin.loader,
            'css-loader',
            'sass-loader',
          ]
        }
      ]
    },
  
    plugins: [
      new MiniCssExtractPlugin({
        filename: 'styles/[name].[contenthash:8].css'
      })
    ]
  }
  ```

### 样式兼容性
针对市面上不同浏览器样式的兼容性问题，我们需要借助`postcss`工具来提高我们编写样式的兼容性程度。

在 `Webpack` 中我们需要配置 `postcss-loader` 对样式文件进行处理。

1. 安装依赖
  ```bash
  yarn add -D postcss-loader postcss postcss-preset-env
  ```
2. 配置
  `postcss-loader`需要在`css-loader`之前执行，在其他预处理`loader`（`less-loader`, `sass-loader`）后执行
  ```javascript
    module.exports = {
      module: {
        rules: [
          // loader配置
          {
            test: /\.s(c|a)ss$/,
            use: [
              MiniCssExtractPlugin.loader,
              'css-loader',
              'postcss-loader',
              'sass-loader',
            ]
          }
        ]
      },
    }
  ```
3. 新建`.browserlistrc`文件配置指定`postcss`要兼容的程度
  ```yml
  # 支持最新的两个版本的浏览器
  last 2 version
  # 覆盖99%的浏览器
  > 1%
  # 不包括已经死掉的浏览器
  not dead
  ```
### 样式压缩
1. 安装依赖
  ```bash
  yarn add -D css-minimizer-webpack-plugin
  ```
2. 配置
  ```javascript
  const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");
  
  module.exports = {
    optimization: {
      minimizer: [
        new CssMinimizerPlugin(),
      ],
    },
  }
  ```
这将仅在生产环境开启 CSS 优化。

如果还想在开发环境下启用 CSS 优化，请将 `optimization.minimize` 设置为 `true`:
```javascript
module.exports = {
    optimization: {
      // [...]
      minimize: true,
    },
}
```

## 处理图片资源
过去在`Webpack4`中，我们需要使用`file-loader`和`url-loader`处理图片资源。

但是在`Webpack5`中，已经将这两个`loader`内置到`Webpack`中，我们只需要简单配置即可处理图片资源。

配置：
```javascript
module.exports = {
  module: {
    rules: [
      // loader配置
      {
        test: /\.(png|jpe?g|gif|webp|svg)$/,
        type: 'asset',
        parser: {
          defaultUrlCondition: {
            // 小于10kb会对图片进行处理
            maxSize: 10 * 1024
          }
        },
        generator: {
          // 文件输出目录
          filename: 'images/[name]_[hash:6][ext][query]',
        }
      }
    ]
  }    
}
```

## 处理其他资源
在`Webpack5`中，我们可以通过loader的`type: 'asset/resource'`，原封不动输出资源，所以我们可以利用这个属性处理不需要进行处理的资源。

配置：
```javascript
...
  module: {
    rules: [
      // loader配置
      {
        test: /\.(ttf|woff2?|mp4|mp3|avi)$/,
        type: 'asset/resource',
        generator: {
          // 文件输出目录
          filename: 'media/[name]_[hash:6][ext][query]',
        }
      }
    ]
  }
...
```

## 自动清空上次打包资源
设置`Webpack`内容的`clean: true`则可以清空上次打包内容。

配置：
```javascript
module.exports = {
    output: {
      // 清空上一次打包内容
      clean: true,
    }
}
```

## 处理js文件
`Webpack` 对 `js` 处理是有限的，只能编译 `js` 中的 `ES` 模块化语法，不能编译其他语法，导致 `js` 不能在 IE 等浏览器运行，所以我们希望做一些兼容处理。

另外，团队对代码格式是有严格要求的，我们不能由肉眼去检测代码格式，需要使用专业的工具来检测。

- 针对 `js` 兼容性处理，我们使用 `Babel` 来完成。
- 针对代码格式，我们需要使用 `ESLint` 来完成

### ESLint
使用 ESLint 关键是设置 ESLint 配置文件，在配置文件里面配置各种 rules 规则，将来运行 ESLint 时就这些规则检测代码规范。

安装依赖：
```bash
yarn add -D eslint @babel/eslint-parser eslint-webpack-plugin
```

配置文件：
- 在项目根目录新建配置文件。
- 文件格式：`.eslitrc` 或 `.eslintrc.js` 或 `.eslintrc.json`

ESLint 会自动查找和读取上述的配置文件
    
#### 具体配置
以 `.eslintrc.js` 为例：
```javascript
module.exports = {
    // 解析选项
    parserOptions: {},
    // 具体的检查规则
    rules: {},
    // 继承其他规则
    extends: []
    // ...
}
```

1. `parser`和`parserOptions` 解析选项
  ```javascript
  parser: "@babel/eslint-parser", 
  parserOptions: {
      ecmaVersion: 8, // ES 语法版本
      sourceType: 'module', // ES 模块化
      // ... 
  }
  ```
2. `rules` 具体规则
   - `"off" | 0`： 关闭规则
   - `"warm" | 1`： 开启规则，使用警告级别的错误，不会导致程序退出
   - `"error" | 2`： 开启规则，使用错误级别的错误，触发时回导致程序退出

  ```javascript
  rules: {
      semi: "error", // 禁止使用分号
      // ...
  }
  ```
  [具体rules规则](https://eslint.bootcss.com/docs/rules/)    

3. `extends` 继承其他规则
- ESLint 官方规则：`eslint:recommended`
- Vue Cli 官方规则：`plugin:vue/essential`
- React Cli 官方规则：`react-app`

4. 在根目录新增`.eslintignore` 跳过不需要检查的文件
5. VSCode 安装 ESLint 插件

#### 在Webpack中使用
- 安装依赖
```bash
yarn add -D eslint-webpack-plugin eslint
```
- 在 `Webpack` 配置文件引入插件
```javascript
const ESLintWebpackPlugin = require('eslint-webpack-plugin')

module.exports = {
    plugins: [
     new ESLintWebpackPlugin({
       context: resolve(__dirname, 'src')
     }),
     // ...
    ]
}
```
- 新建`.eslintrc.js`
```javascript
module.exports = {
  extends: ['eslint:recommended'],
  env: {
    es6: true, // 开启es6语法
    node: true, // 开启node全局变量
    browser: true, // 开启浏览器全局变量
  },
  parserOptions: {
    ecmaVersion: 8,
    sourceType: 'module',
  },
  rules: {
    "no-var": 2, // 不能使用 var 定义变量
    // ...
  }
}
```

### Babel
`JavaScript`编辑器。

主要用于将 `ES6` 语法编译成向后兼容的`JavaScript`语法，以便能够运行在当前和旧版本的浏览器上。

#### 配置文件
- 在项目根目录新建配置文件。
- 文件格式：`.babelrc` 或 `.babelrc.js` 或 `.babelrc.json`或`babel.config.js` 或 `babel.config.json`

Babel 会自动查找和读取上述的配置文件

#### 具体配置
以 `babel.config.js` 为例：
```javascript
module.exports = {
    // 预设
    presets: [],
}
```
`presets` 预设是一组 `Babel` 插件，扩展 `Babel` 功能
- `@babel/preset-env`：智能预设，允许使用最新的 `JavaScript`
- `@babel/preset-react`：用来编译 `React jsx` 语法
- `@babel/preset-typescript`：用来编译 `TypeScript` 语法

#### 在Webpack使用
- 安装依赖
```bash
yarn add -D @babel/core @babel/preset-env babel-loader
```
- 在 `Webpack` 配置文件配置 `loader`
```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.jsx$/,
                exclude: /node_modules/,
                use: {
                  loader: 'babel-loader'
                }
            }
        ]
    }
}
```
- 在根目录新建 `babel.config.js` 文件
```javascript
module.exports = {
  presets: ['@babel/preset-env']
}
```

## 处理HTML
- 安装依赖
```bash
yarn add -D html-webpack-plugin
```
- 在 `Webpack` 配置文件引入插件
```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')
    
module.exports = {
    plugins: [
        new HtmlWebpackPlugin({
          template: resolve(__dirname, 'public/index.html')
        }),
    ]
}
```

## 开发服务器&自动化
- 安装依赖
```bash
yarn add -D webpack-dev-server
```
- 在 `Webpack` 配置文件配饰 `devServer`
```javascript
module.exports = {
    devServer: {
      host: 'localhost',
      port: '3000',
      open: true,
      hot: true,
    }
}
```
- 启动命令
```bash
webpack serve
```

# 高级配置
## 提高开发体验
### SourceMap
SourceMap （源代码映射）是一个用来生成源码与构建后的代码一一映射的文件的方案。

它会生产一个xxx.map文件，里面包含源代码和构建后代码的每一行，每一列的映射关系，当构建后代码出错了，会通过xxx.map文件从构建后的代码出错位置找到映射后源代码出错位置，从而让浏览器提示源代码文件出错的位置，帮助我们更快的定位到问题。

主要分为下面其中模式：
| 模式                    | 说明                                                                                                           |
| ----------------------- | -------------------------------------------------------------------------------------------------------------- |
| eval                    | 每个 module 会封装到 eval 里包裹起来执行，并且会在末尾追加注释 //@ sourceURL.                                  |
| source-map              | 生成一个 SourceMap 文件.                                                                                       |
| hidden-source-map       | 和 source-map 一样，但不会在 bundle 末尾追加注释.                                                              |
| inline-source-map       | 生成一个 DataUrl 形式的 SourceMap 文件.                                                                        |
| eval-source-map         | 每个 module 会通过 eval() 来执行，并且生成一个 DataUrl 形式的 SourceMap                                        |
| cheap-source-map        | 生成一个没有列信息（column-mappings）的 SourceMaps 文件，不包含 loader 的 sourcemap（譬如 babel 的 sourcemap） |
| cheap-module-source-map | 生成一个没有列信息（column-mappings）的 SourceMaps 文件，同时 loader 的 sourcemap 也被简化为只包含对应行的。   |


实际开发时我们主要关心两种情况就行：
- 开发模式：`cheap-module-source-map`
  - 优点：打包编译速度快，只包含行映射
  - 缺点：没有列映射
  ```javascript
  module.exports = {
    mode: 'development',
    devtool: 'cheap-module-source-map'
  }
  ```
- 生产模式：`source-map`
  - 优点：包含行/列映射
  - 缺点：打包编译速度更慢
  ```javascript
  module.exports = {
    mode: 'production',
    devtool: 'source-map'
  }
  ```

## 提升打包构建速度
### HMR 热模块替换
开发时我们修改一个模块代码，Webpack 默认会将所有模块全部重新打包编译，速度很慢。

所以我们需要做到修改某个模块，就只有这个模块需要重新打包编译，其他模块不需要更新，这样打包就会快很多。

HMR 就能在程序运行的时候，替换、更新或者删除模块，无需重新加载整个页面。

基本配置：
```javascript
module.exports = {
  devServer: {
    hot: true, // 开启热更新
    ...
  }
}
```
简单的js模块需要手动添加热模块更新代码才能实现，在对应的js模块下面添加如下代码
```javascript
if (module.hot) {
    module.hot.accept('./js/count');
    module.hot.accept('./js/sum');
}
```
其中`module.hot.accept`接收的就是我们希望热替换的模块。

如果是在 `vue` 或 `react` 项目，它们分别在`vue-loader`和`react-hot-loader`内部实现了热更新。

对于样式模块，`style-loader`已经实现了热模块更新的功能。

### oneOf
使用 `oneOf` 根据文件类型加载对应的`loader`，只要能匹配一个即可退出，
对于同一类型文件，比如处理js，如果需要多个`loader`，可以单独抽离js处理，确保`oneOf`里面一个文件类型对应一个`loader`。

配置：
```javascript
module.exports = {
    // 加载器
  module: {
    rules: [
      {
        oneOf: [
          // less-loader配置
          {
            test: /\.less$/,
            use: getStyleLoader('less-loader'),
          },
          // 处理图片资源
          {
            test: /\.(png|jpe?g|gif|webp|svg)$/,
            type: 'asset',
            parser: {
              dataUrlCondition: {
                // 小于10kb会对图片转为base64格式
                maxSize: 10 * 1024,
              },
            },
          },
          // 处理其他资源
          {
            test: /\.(ttf|woff2?|mp4|mp3|avi)$/,
            type: 'asset/resource',
          },
          // 处理js资源
          {
            test: /\.jsx?$/,
            exclude: /node_modules/,
            use: 'babel-loader',
          }
        ]
      }
    ]
  }
}
```

### Include/Exclude
我们在开发过程中可能会引入其他第三方的库和插件，所有文件都下载到 `node_modules` 中，而这些文件是不需要编译可以直接使用的。

所以我们在对 `js` 文件处理的时候，要排除 `node_modules` 下的文件

- include：只编译指定文件
- exclude：排除文件不需要编译

⚠️注意：`include`和`exclude`不能同时使用

配置
```javascript
const ESLintWebpackPlugin = require('eslint-webpack-plugin');

module.exports = {
    module: {
        rules: [
            {
              test: /\.jsx?$/,
              include: resolve(__dirname, '../src'),
              // exclude: /node_modules/,
              use: {
                loader: 'babel-loader'
              }
            }
        ]    
    },

    plugins: [
        // ESLint 代码检验
        new ESLintWebpackPlugin({
          context: resolve(__dirname, '../demo'),
          exclude: "node_modules" // 默认值
        }),
    ],
}
```

### Cache 缓存
每次打包时 `js` 文件都需要经过 `ESLint` 检查和 `Babel` 编译，速度比较慢。

我们可以缓存之前的 `ESLint` 检查和 `Babel` 编译的结果，这样就可以在后面打包时速度更快了。

配置：
```javascript
const ESLintWebpackPlugin = require('eslint-webpack-plugin');

module.exports = {
    module: {
        rules: [
            {
              test: /\.jsx?$/,
              include: resolve(__dirname, '../src'),
              loader: 'babel-loader',
              options: {
                cacheDirectory: true, // 开启babel缓存
                cacheCompression: false, // 关闭缓存压缩
              }
            }
        ]    
    },

    plugins: [
        // ESLint 代码检验
        new ESLintWebpackPlugin({
          context: resolve(__dirname, '../demo'),
          exclude: "node_modules"， // 默认值
          cache: true, // 开启ESLint缓存
        }),
    ]
}
```

### Thread 多进程
当项目越来越大时，打包速度越来越慢。

我们想继续提升打包速度，其实就是提升 js 的打包速度，因为其他文件都比较少，

而对 js 文件处理主要就是处理 ESLint、Babel、Terser 三个工具，所以我们要提升他们的运行速度。

我们可以开启多进程同时处理 js 文件，这样速度就比之前单进程的打包更快了。

⚠️注意：仅在特别耗时的操作中使用，因为每个进程的启动都需要有一定的时间开销。

安装依赖：
```bash
yarn add -D thread-loader
```

获取CPU核数：
```javascript
const os = require('os')
// CPU核数
const threads = os.cups().length
```

配置：
```javascript
const ESLintWebpackPlugin = require('eslint-webpack-plugin');
const TerserWebpackPlugin = require('terser-webpack-plugin');

module.exports = {
    module: {
        rules: [
            {
              test: /\.jsx?$/,
              include: resolve(__dirname, '../src'),
              use: [
                // 开启多进程
                {
                  loader: 'thread-loader',
                  options: {
                    workers: threads,
                  },
                },
                {
                  loader: 'babel-loader',
                  options: {
                    cacheDirectory: true, // 开启babel缓存
                    cacheCompression: false, // 关闭缓存压缩
                  }
                }
              ]
            }
        ]    
    },

    plugins: [
        // ESLint 代码检验
        new ESLintWebpackPlugin({
          context: resolve(__dirname, '../demo'),
          exclude: "node_modules"， // 默认值
          cache: true, // 开启ESLint缓存
            threads, // 开启多进程
        }),
    ],

    optimization: {
      minimizer: [
        new TerserWebpackPlugin({ parallel: threads })
      ],
    },
}
```



## 减少打包体积
### Tree Shaking
开发时，我们第一个一些工具函数，或者引用其他第三方的库。

如果没有特殊处理的话，我们打包时会全量引入，但是实际我们用到的就只有其他少部分的模块，这样会导致我们项目打包体积过大。

`Tree Shaking`就是用于描述移除 `js` 中没有使用上的代码。

⚠️注意：`Tree Shaking` 依赖 `ESModule`。

默认 Webpack 已经开启了这个功能，不需要额外配置。

### 较少Babel生成代码的体积
Babel 为编译的每个文件都插入了辅助代码，使代码体积过大。

Babel 对一些公共的方法使用了非常小的辅助代码，比如 `_extend`等。默认情况下会被添加到每一个使用到它的文件中。

我们可以将这些辅助代码作为一个独立的模块，来避免重复引入。

- `@babel/plugin-transform-runtime`：禁用 Babel 自动对每一个文件的runtime注入，而是引入`@babel/plugin-transform-runtime`，并且使所有辅助代码从这里引用。

安装依赖：
```bash
yarn add -D @babel/plugin-transform-runtime
```

在`babel.config.js`中配置`@babel/plugin-transform-runtime`：
```javascript
module.exports = {
  plugins: ['@babel/plugin-transform-runtime'], // 减少代码体积
  presets: ['@babel/preset-env'],
};
```

### Image Minimizer 图片压缩
如果项目引入了较多图片，而且图片的提交比较大，将来请求速度比较慢。

我们可以对图片进行压缩，减少图片体积。

安装依赖：
```bash
yarn add -D image-minimizer-webpack-plugin imagemin imagemin-gifsicle imagemin-jpegtran imagemin-optipng imagemin-svgo
```

配置：
```javascript
const ImageMinimizerPlugin = require("image-minimizer-webpack-plugin");

module.exports = {
    // 优化配置
  optimization: {
    minimizer: [
      // 图片压缩
      new ImageMinimizerPlugin({
        minimizer: {
          implementation: ImageMinimizerPlugin.imageminMinify,
          options: {
            plugins: [
              ["gifsicle", { interlaced: true }],
              ["jpegtran", { progressive: true }],
              ["optipng", { optimizationLevel: 5 }],
              [
                "svgo",
                {
                  plugins: [
                    {
                      name: "preset-default",
                      params: {
                        overrides: {
                          removeViewBox: false,
                          addAttributesToSVGElement: {
                            params: {
                              attributes: [
                                { xmlns: "http://www.w3.org/2000/svg" },
                              ],
                            },
                          },
                        },
                      },
                    },
                  ],
                },
              ],
            ],
          },
        },
      }),
    ]
  }
}
```

## 优化代码运行性能
### Code Split 代码分割
打包代码时会将所有 js 代码都打包到一个文件中，体积太大了，如果需要渲染首页，就要加载首页的 js 文件，其他文件不应该加载。

所以我们需要将打包生成的文件进行切割，生成多个小的 js 文件，渲染哪个页面就加载对应的 js 文件，这样每次加载的资源少了，访问页面的速度自然就变快了。

代码分割主要完成下面工作：
- 分割文件：将打包生成的文件进行切割，生成多个 js 文件
- 按需加载：需要哪个文件加载哪个文件

配置：
```javascript
module.exports = {
    optimization: {
        splitChunks: {
            chunks: 'all', // 对所有模块都进行分割
            // 以下为默认配置
            // chunks: 'async', // 需分割的模块
            // minSize: 20000, // 代码分割最小的大小
            // minRemainingSize: 0, // 类似minSize，最后确保提出的文件大小不能为0
            // minChunks: 1, // 至少被引用的次数，满足条件才会被分割
            // maxAsyncRequests: 30, // 按需加载时并行加载的文件最大数量
            // maxInitialRequests: 30, // 入口js 文件最大并行数量
            // enforceSizeThreshold: 50000, // 超过500kb一定要单独打包
            // cacheGroups: {
            //   defaultVendors: {
            //     test: /[\\/]node_modules[\\/]/,
            //     priority: -10, // 权重，越大越高
            //     reuseExistingChunk: true, // 如果当前chunk包含已从主 bundle 中拆分出的模块，则它将被重用，而不是重新生成新的模块
            //   },
            //   default: {
            //     minChunks: 2, // 这里的minChunks权重最高
            //     priority: -20,
            //     reuseExistingChunk: true,
            //   },
            // },
        }
    }
}
```

如果希望自定义chunk的名称，可以使用魔法注释 `webpackChunkName` 来指定：
```javascript
document.addEventListener('click', () => {
  import(/* webpackChunkName: "count" */'./count').then(res => {
    console.log(res.count());
  });
});
```
同时需要配置 `webpack.config.js` ：
```javascript
module.exports = {
  // 输出
  output: {
    // chunk的名称
    chunkFilename: 'js/[name].[chunkhash:8].chunk.js',
  }
}
```
那么打包后的文件就是我们配置`chunkFilename`规则格式决定。

### Preload 和 Prefetch
我们已经做了代码分割，同时实现了按需加载，但是速度上还是不能带到我们的预期，比如，点击按钮才加载资源，如果这个资源体积很大，那么用户就会感觉卡顿。

如果想在浏览器空闲的时间，加载后续需要使用的资源，我们可以使用 `Preload` 和 `Prefetch` 技术。

- `Preload`：告诉浏览器立即加载资源。
- `Prefetch`：告诉浏览器在空闲时才加载资源。

共同点：
- 只加载资源，但不执行。
- 都有缓存。

区别：
- `Preload` 加载优先级高，`Prefetch` 加载优先级低。
- `Preload` 只能加载当前页面需要使用的资源，`Prefetch` 可以加载当前页面资源，也可以加载下一个页面需要使用的资源。

总结：
- 当前页面优先级高的资源用 `Preload` 加载。
- 下一个页面需要使用的资源使用 `Prefetch` 加载。

安装依赖：
```bash
yarn add -D @vue/preload-webpack-plugin
```

### Network Cache 缓存
- 对打包资源使用 `contenthash`
- 配置 `runtimeChunk`

配置：
```javascript
module.exports = {
  // 输出
  output: {
    // 文件输出名称
    filename: 'js/[name].[chunkhash:8].js',
    // chunk的名称
    chunkFilename: 'js/[name].chunk.[contenthash:8].js',
    // 静态资源打包路径
    assetModuleFilename: 'assets/[name].[contenthash:8][ext][query]',
  },
    
  optimization: {
    runtimeChunk: {
      name: (entrypoint) => `runtime.${entrypoint.name}`
    }
  }
}

```

### 解决js兼容性问题 core-js
我们使用babel 对 js 进行兼容性处理，其实使用 @babel/preset-env 智能预设来处理兼容性问题。

它将ES6的一些语法进行编译转化，比如箭头函数，但是如果是ES6+以上的语法，例如async/await, Promise,  includes等，它是没有办法处理的。

所以这个时候我们的 js 代码还是存在兼容性问题。

`core-js` 是专门用来做 ES6 及以上的API的兼容。


安装依赖
```bash
yarn add -D core-js
```

配置 `babel.config.js`
```javascript
module.exports = {
  plugins: ['@babel/plugin-transform-runtime'],
  presets: [
    ['@babel/preset-env', { useBuiltIns: 'usage', corejs: 3 }]
  ],
};
```
