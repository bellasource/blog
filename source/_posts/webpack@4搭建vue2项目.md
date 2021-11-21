---
title: webpack@4搭建vue2项目
date: 2021/02/03
tags: ["webpack", "webpack4", "webpack5", "项目打包"]
---

{% blockquote %}
自动化构建工具。从入口开始，根据文件间的静态依赖关系进行静态分析，然后将这些模块（包含 js/json/css/less...） 生成静态资源
webpack4 构建项目 github 链接: https://github.com/bellasource/vue2_webpack_4_project
{% endblockquote %}

## 五个核心概念

### entry

入口起点，只 webpack 应该使用哪个模块，作为构建依赖图的开始，值：string | [string] | Object

### output

用于指定出口路径、命名等

### loader

对模块源代码进行转换，让 webpack 处理非 javascript/json 文件
loader 从右到左（或从下到上）执行(execute)，

### plugins

扩展功能，例如：打包优化，压缩代码等,值：Array

### mode

生产模式 production，开发模式 development,默认为生产模式

## 开启项目

- npm init -y //初始化包管理工具
- npm install webpack webpack-cli -g //全局安装,不推荐
- npm install webpack@4 webpack-cli@3 -D //本地安装,作为本地依赖使用

webpack 支持零配置打包，命令：npx webpack，默认以 ./src/index 文件为打包入口，输出到 ./dist 目录，
npx webpack 命令默认查找当前目录的 webpack.config.js 文件并运行打包

```js
--mode=development 指定打包模式
--output=dist/index.js_b 指定出口路径
npx webpack ./src/index1.js ./src/index2.js 打包多个文件用空格隔开
```

每次输入命名运行打包不方便，因此在 package.json 中配置 scripts，即可通过 npm run build 运行打包

```json
"scripts": {
    "build": "npx webpack ./src/index1.js ./src/index2.js --mode=development --output=dist/index.js",
  },
```

## 使用 webpack 配置文件

项目文件目录

```text
├─ dist
├─ package.json
├─ src
│  ├─ index.html
│  └─ js
│     ├─ index1.js
│     └─ index2.js
└─ webpack.config.js
```

### 配置 entry output

- 单/多入口，打包到一个出口文件。按下面的配置执行打包后，入口文件及其依赖文件都被打包至 bundle.js

```js
//webpack.config.js
const path = require("path");
module.exports = {
  mode: "development", //打包模式，默认production

  //打包入口配置,相对路径是相对于启动命名npx webpack时所在的目录
  // entry: "./src/js/index1.js", //单一入口
  entry: ["./src/js/index1.js", "./src/js/index2.js"], //多入口，值为数组

  //打包出口配置
  output: {
    filename: "bundle.js", //输出文件，默认main.js，
    //输出路径，必须绝对路径,否则报错
    // path: __dirname + "/dist",
    path: path.resolve(__dirname, "./dist"), //使用path模块，实现路径拼接
  },
};
```

- 多入口多出口打包配置

```js
//webpack.config.js
const { resolve } = require("path");
module.exports = {
  mode: "development",
  // 入口为对象
  entry: {
    one: "./src/js/index1.js",
    two: "./src/js/index2.js",
  },
  // 出口filename添加[name],即为enrty的key值
  output: {
    filename: "./[name]bundle.js",
    path: resolve(__dirname, "./dist"),
  },
};
//执行打包后的目录为：
├─ dist
│  ├─ onebundle.js
│  └─ twobundle.js
```

如果在编译时，不知道最终输出文件的 publicPath 是什么地址，则可以将其留空，并且在运行时通过入口起点文件中的 \_\_webpack_public_path\_\_ 动态设置。

### loader 打包 css、less、stylus

css、less、sass 文件不能被 webpack 解析需借助 loader，并通过 style-loader 插入到 header 标签内的 style 标签中
安装
npm install css-loader@3 style-loader@1 -D
npm i less@3 less-loader@7 -D
npm i node-sass@4 sass-loader@7 -D
注意：因为有的版本号不兼容，需要指定大版本号，否则报错：this.getOptions is not a function

- 引入 less 全局变量

```js
/*
/src/css目录下创建home.css文件,并在index1.js中 import '../css/home.css'
*/
module.exports = {
  ...
  module:{
    rules: [
      // 识别css文件
      {
        test: /\.css$/,
        //loader 从右到左（或从下到上）执行(execute)，
        // loader配置方式一：
        // loader: ["style-loader", "css-loader"],

        // loader配置方式二
        use: ["style-loader", "css-loader"],
      },

      // 识别less文件
      {
        test:/\.less$/,
        // loader配置方式三
        use:["style-loader","css-loader"
          {
            loader:"less-loader",
            options:{
              lessOptions: {
                javascriptEnabled: true,
                modifyVars: {
                  "@primary-color": "#ddd",//配置全局less变量
                },
              },
            }
          },
          // 引入less变量文件
          {
            loader: "style-resources-loader",
            options: {
              patterns:resolve(__dirname, "./src/less/variables.less"),
              //prepend/append， 表示资源导入的位置，在样式后导入的会覆盖前面导入的
              injector:"append",
              preProcessor:'less'
            },
          },
        ]
      },
      {
        test: /\.scss$/,
        use: [
          "style-loader",
          "css-loader",
          "sass-loader",
          // 引入scss变量(只能配置一次)
          {
            loader: "style-resources-loader",
            options: {
              patterns: resolve(__dirname, "./src/scss/variables.scss"),
            },
          },
        ],
      },

    ],
  }
};

/*
/less/variables.less文件代码
@font-color:#666;
@green-color:red;

/scss/variables.scss文件代码
$font-color:red;
 */
```

### loader postcss-loader 配置 css 兼容

不同版本的浏览器，可能需要 css 加前缀，

- 配置一 安装：npm i postcss-loader@3 autoprefixer@8 -D

```JS
{
  test: /\.css$/,
  use: [
    "style-loader",
    "css-loader",
    // postcss-loader需要在css-loader之前解析，所以必须配置在css-loader之后
    {
      loader: "postcss-loader",
      options: {
        plugins: [require("autoprefixer")],
      },
    },
  ],
},
// 与webpack.config.js同级目录新建文件：.browserslistrc;或者配置在package.json中
/* chrome 50
last 1 versions
ie 7
iOS 7 */
```

- 配置二 安装 npm i postcss-loader@4 postcss-preset-env@6 -D

```js
{
  loader: "postcss-loader",
  options: {
    postcssOptions: {
      plugins: [
        [
          "postcss-preset-env",
          {
            // 其他选项
          },
        ],
      ],
    },
  },
}
/* 通过postcss-preset-env插件，在package.json中配置兼容 */
"browserslist": [
  "chrome 50",
  "last 1 versions",
  "ie 7",
  "iOS 7",
]
```

### loader 配置 js 语法检查

使用 eslint 对 js 基本语法错误进行提前检查，不能识别运行中的错误
安装：npm i eslint eslint-loader -D

```js
// webpack.config.js
module.exports = {
  ...
  module:[
    ...
    {
        test: /\.js$/,
        exclude: /node_modules/, //排除
        enforce: "pre", //提前检查
        use: ["eslint-loader"],
      },
  ]
}
// 同目录下eslintrc.js文件配置检查规则
module.exports = {
  parserOptions: {
    ecmaVersion: 6, // 支持es6
    sourceType: "module", // 支持es6模块化
  },
  env: {
    // 设置环境
    browser: true, // 支持浏览器环境： 能够使用window上的全局变量
    node: true, // 支持服务器环境:  能够使用node上global的全局变量
  },
  globals: {
    // 声明使用的全局变量
    vari: "readonly", // 变量vari 可读，writable则为可写
  },
  rules: {
    // eslint检查的规则  0 忽略 1 警告 2 错误
    "no-console": 0, // 不检查console
    eqeqeq: 0, // 用 == 而不用 === 就报错
    "no-alert": 0, // 不能使用alert
  },
  extends: "eslint:recommended", // 使用eslint推荐的默认规则
};
```

- 扩展

另外也可借助 eslint-config-alloy 插件
npm install eslint @babel/eslint-parser vue-eslint-parser eslint-plugin-vue eslint-config-alloy -D

```js
// .eslintrc.js
module.exports = {
  extends: ["alloy", "alloy/vue"],
  env: {
    browser: true,
    node: true,
  },
  rules: {
    // 自定义规则
  },
};
// .prettierrc.js
module.exports = {
  // 一行最多 100 字符
  printWidth: 100,
  // 使用 4 个空格缩进
  // tabWidth: 4,
  // 不使用缩进符，而使用空格
  useTabs: false,
  // 行尾需要有分号
  semi: true,
  // 使用单引号
  singleQuote: true,
  // 对象的 key 仅在必要时用引号
  quoteProps: "as-needed",
  // jsx 不使用单引号，而使用双引号
  jsxSingleQuote: false,
  // 末尾不需要逗号
  // trailingComma: 'none',
  // 大括号内的首尾需要空格
  bracketSpacing: true,
  // jsx 标签的反尖括号需要换行
  jsxBracketSameLine: false,
  // 箭头函数，只有一个参数的时候，也需要括号
  arrowParens: "always",
  // 每个文件格式化的范围是文件的全部内容
  rangeStart: 0,
  rangeEnd: Infinity,
  // 不需要写文件开头的 @prettier
  requirePragma: false,
  // 不需要自动在文件开头插入 @prettier
  insertPragma: false,
  // 使用默认的折行标准
  proseWrap: "preserve",
  // 根据显示样式决定 html 要不要折行
  htmlWhitespaceSensitivity: "css",
  // 换行符使用 lf
  endOfLine: "lf",
};
```

### loader 配置 js 语法转换

使用 babel 对 js 新语法转换为浏览器支持的语法(ES6 模块化 const let 数组对象解构赋值 等等)
安装：npm install -D babel-loader @babel/core @babel/preset-env

```js
// 可以创建babel.config.js中配置babel的具体规则
{
  test:/\.js$/,
  exclude:/node_modules/,
  use:[
    {
      loader:'babel-loader',
      options:{
        preset:['@babel/preset-env'],
      }
    }
  ]

}
```

### loader js 兼容处理

用于实现浏览器不支持原生功能的代码，例如 Promise，async/await 等
安装：npm i @babel/polyfill -D
使用：方式一：在入口 js 文件中引入即可 import '@babel/polyfill'；方式二：在文件 babel.config.js 中配置

```js
module.exports={
  presets:[
    [
      '@babel/preset-env',
      {
        modules: false,
        useBuiltIns: 'usage',//自动进行polyfill,进行js兼容
        corejs: 3,
      },
    ],
  ];
  plugins:[]
}
```

### loader 打包图片资源

webpack 不能解析图片资源，需要借助 url-loader
安装：npm i file-loader url-loader -D
主要文件目录

```text
test_webpack
├─ .eslintrc.js
├─ package.json
├─ src
│  ├─ images
│  │  └─ 01.jpg
│  ├─ index.html
│  ├─ js
│  │  └─ index1.js
│  └─ less
│     └─ common.less
└─ webpack.config.js

```

```js
/*
common.less文件
.main {
  width: 200px;
  height: 200px;
  background: url("../images/01.jpg") no-repeat;
  background-size: cover;
}
*/

//webpack.config.js
{
  test: /\.(jpe?g|png|gif|svg)(\?.*)?$/,
  loader: "url-loader",
  options: {
    limit: 10000,//小于等于10000转base64
    // publicPath: "../dist",//设置图片资源url路径
    outputPath: 'static/images/',// 文件本地输出路径
    name: "[name].[hash:7].[ext]",//大于10000文件名称和后缀
  },
},
/*
打包后的资源./dist/static/images/01.9a3d046.jpg
html中引入的资源路径: ./static/images/01.9a3d046.jpg,导致资源404
因此可以配置publicPath，也可将index.html打包到dist目录下
 */
```

### loader 配置字体

同样的，借助 file-loader,在 module 中配置

```js
{
  test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
  loader: "file-loader",
  options: {
    outputPath: "static/fonts/",
    name: "[name].[hash:7].[ext]",
  },
},
```

### loader 识别 vue 文件

安装： npm i vue-loader@15 vue-template-compiler@2 -D

```js
{
  test: /\.vue$/,
  loader: "vue-loader",
  include: resolve(__dirname,'./src'),
},
```

### loader 打包 html 模板中的图片资源

安装：npm i html-loader@1 -D,注意版本，否则报错
效果：将 img 标签的图片地址自动替换为打包后的图片地址

```js
{
  test: /\.html$/,
  use: ["html-loader"],
},

```

### loader 分离 css 文件

使用 style-loader 对 css、less、sass 的配置，打包完成后，通过 js 创建 style 标签，将样式内容引入 html 中
安装：npm mini-css-extract-plugin@1 -D
作用：将 css，less，sass 提取为单独的文件，通过 link 引入样式文件，用来代替 style-loader

```js
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  module: {
    rules: [
      {
        test: /.css$/,
        use: [MiniCssExtractPlugin.loader, "css-loader"],
      },
      {
        test: /.less$/,
        use: [MiniCssExtractPlugin.loader, "css-loader", "less-loader"],
      },
      {
        test: /.scss$/,
        use: [MiniCssExtractPlugin.loader, "sass-loader",'],
      },
    ],
  },
  plugins: [
    new MiniCssExtractPlugin({ filename: "/css/[name].[hash:8].css" })//配置css输出路径
  ],
};
```

### optimization 压缩 css 文件

安装：npm i optimize-css-assets-webpack-plugin@5 -D

```js
optimization: {
    minimize: true,
    minimizer: [
      // 用于优化css文件
      new OptimizeCSSAssetsPlugin({
        assetNameRegExp: /\.css$/g,
        cssProcessorOptions: {
          autoprefixer: { disable: true },
          mergeLonghand: false,
          discardComments: {
            // 移除注释
            removeAll: true,
          },
        },
        canPrint: true,
      }),
    ],
  },
```

### optimization 优化 js 文件

安装：npm i terser-webpack-plugin@4 -D

```JS
optimization: {
    minimize: true,
    minimizer: [
      //js文件优化，有一些默认配置如移出注释，移出debugger等
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: false, //默认为false
            drop_debugger: false, //默认为true
          },
        },
      }),
    ],
  },
```

### optimization runtimeChunk

用来提取包含 chunks 映射关系的 list 为单独文件，避免子文件变更，导致浏览器缓存失效的问题

```js
/* single:创建一个在所有生成 chunk 之间共享的运行时文件，文件名默认为runtime
  multiple:每个入口添加一个只含有 runtime 的额外 chunk.
  为减少初始化请求runtime文件，通过inline-manifest-webpack-plugin，将生成的runtime文件内容直接插入html文件中
 */

runtimeChunk:'single',//默认值：false
```

### optimization splitChunks 拆包

将 js 文件拆分为多个 bundle，实现按需引入按需加载

```js
optimization: {
  splitChunks:{
    // 更多配置可查看文档
    automaticNameDelimiter: "-", //默认~，vendor和名称之间分割符
    chunks: "async",//默认值async，异步加载的模块拆包为单位bundle
    // 缓存组
    cacheGroups: {
        vendor: {
          // 分割第三方依赖
          priority: 1, // 设置优先级，首先抽离第三方模块
          name: 'vendor',
          test: /node_modules/,
          chunks: 'initial',//同步模块
          minSize: 0,
          minChunks: 1, // 最少引入了1次
        },

        common: {
          // 分割公共模块
          name: 'common',
          chunks: 'initial',
          minSize: 100, // 大小超过100个字节
          minChunks: 3, // 最少引入了3次
        },
      },
  }
},
```

### plugin 清空打包目录（生产模式）

引入插件 clean-webpack-plugin 能自动删除上一次打包生成的文件，
安装 npm i clean-webpack-plugin -D

```js
const CleanWebpackPlugin = require('clean-webpack-plugin')//引入插件
module.exports = {
  ...
  plugins: [new CleanWebpackPlugin()],//使用插件
};
```

### plugin 打包 html 文件

安装：npm i html-webpack-plugin@4 -D
注意：5 以上版本不兼容 webpack4，报错'tap'

```js
const htmlWebpackPlugin = require("html-webpack-plugin");
module.exports = {
  ...
  plugins: [
    new htmlWebpackPlugin({
      template: resolve(__dirname, "./src/index.html"),
      filename: "index.html",//出口路径
      title: "测试webpack", //在thml中配置<title><%= htmlWebpackPlugin.options.title %></title>
      favicon: "./src/images/01.jpg",//默认打包到出口根目录
      hash: true, //默认为false，给打包的js文件增加hash后缀
      chunks: ["one"], //多出口文件使用，仅插入入口key值为one的出口文件
      minify:true,//dev时为false，可配置移出注释，空格，引号等
    }),
  ],//使用插件
};
```

### plugin 优化 webpack 编译错误提示

配置在开发环境
安装：npm i friendly-errors-webpack-plugin@1 -D

```JS
const FriendlyErrorsWebpackPlugin = require("friendly-errors-webpack-plugin");
module.exports = {
  devServer:{
    ...
    quiet:true,
  }
  plugins:[
    ...
    new FriendlyErrorsWebpackPlugin()
  ]
}
```

### plugin 打包静态资源

安装：npm i copy-webpack-plugin@6 -D

```js
const CopyWebpackPlugin = require("copy-webpack-plugin");
module.export = {
  plugins: [
    new CopyWebpackPlugin({
      patterns: [
        {
          from: "./static",
          to: "./static", //相对于output
        },
      ],
    }),
  ],
};
```

### devServer 自动编译

效果：代码修改后，能自动运行打包，刷新页面
安装： npm i webpack-dev-server@3 -D
启动：npx webpack-dev-server

```js
module.exports={
  ...
  devServer:{
    hot:true,//js/css改变时，自动在浏览器中更新，模块热更新
    compress:true,//启用gzip压缩
    host:'0.0.0.0',//ip
    port:3000,//端口号
    openPage:'index.html',//打开文件名，默认index.html
    publicPath: "/", //打包资源的输出路径，没有配置时会拼output的publicPath，一般配置为当前路径
    proxy: {
      // 以/api开头的请求
      '/api': {
        target: `http://192.168.1.123:3000`,
        changeOrigin: true,
        pathRewrite:{//地址重写
          "^/api":"",
        }
      },
    },
  },
  plugins:[
    // hot:true即可开启模块热更新，内部会自动添加以下配置。
    new webpack.HotModuleReplacementPlugin(),
  ]

}
```

### resolve

resolve 选项中，可以配置模块被如何解析

```js
module.exports = {
  //...
  resolve: {
    extensions: [".js", ".vue", ".json"], //引入文件省略扩展名
    alias:{
      @src:resolve(__dirname, '../src'),//路径别名
    }
  },
};
```

通过 webpack 配置的路径别名，在使用路径别名时，编辑器不会有正确的路径提示，如果用的 vscode 编辑器可以在 jsconfig.json 中配置

```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"] //路径别名提示
    }
  },
  "exclude": ["node_modules", "dist"] // 除了这两个目录以外,其他的目录都使用上述的设置
}
```

### devtool 开发工具

```js
module.exports = {
  devtool: "source-map" | none, //精确到一行的具体部分的报错，一般用于生产环境
  devtool: "cheap-module-eval-source-map", //精确到行 一般用于开发环境
};
```

### cimmit 前进行语法检查

安装：npm i prettier husky lint-staged -D

```json
//package.json中配置
{
  "lint-staged": {
    "*": ["npm run prettier"]
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  }
}
```
