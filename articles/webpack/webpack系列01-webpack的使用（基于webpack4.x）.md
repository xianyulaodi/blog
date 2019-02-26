
> webpack在前端开发者的世界再熟悉不过了，网上也很多关于webpack的文章，为了让自己更好的巩固webpack，所以有了这个系列文章。

webpack 是js模块打包器，一直在更新,本系列是基于**webpack4.29.5**版本，将来的某一天，发觉本文章的一些配置用不了，那可能是webpack已经更新到更高的版本了

## 安装webpack

安装webpack和webpack-cli，由于webpack4中webpack-cli抽离了，所以需要分别安装，window就尽量全局安装了
```bash
npm install webpack webpack-cli -g
```
搞一个demo试试
```bash
mkdir webpack-demo && cd webpack-demo
npm init -y
```
新建一个index.js文件，./src/index.js
```tree
--package.json
--src
  -- index.js
```

## 零配置
webpack4支持0配置，默认`./src/index.js `为入口文件，webpack运行时，会根据mode的值采取不同的默认配置,mode两个可选值：`production` 和 `development`。没有传mode，会有一个警告
```bash
WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/
```
解决这个警告，修改`package.json`部分,传入mode即可
```bash
   "scripts": {
      "dev": "webpack --mode development",
      "build": "webpack --mode production"
   }
```
development和production的区别在于一个代码没压缩，一个有压缩和优化，执行 `npm run dev`,就会生产一个`./dist/main.js`文件。

如果我们只是想要一个简单的打包功能，使用默认配置就够用了。

简易webpack demo()[]

## 自定义webpack配置

当然，在我们的项目中，使用webpack的默认配置明显是不够用的,还是需要自定义我们的webpack配置

在我们的项目根目录，创建`webpack.config.js`，webpack配置的概况如下
```javascript
module.exports = {
    entry: '',               // 入口文件
    output: {},              // 出口文件
    module: {},              // 模块相关配置
    plugins: [],             // 插件相关配置
    resolve: { }             // 解析模块的可选项
    devServer: {},           // 开发服务器相关配置
    devtool: 'inline-source-map',  //开发工具，比如启动source-map
    mode: 'development'      // 模式配置 development/production
}
```
接下来，我们一点点往里面添加内容，以此更好的理解webpack

#### 单入口配置
一些项目中，只有一个入口文件，那么，入口文件和出口文件可以这样配置
```javascript
const path = require('path');

module.exports = {
    entry: './src/index.js',    
    output: {
        filename: 'bundle.js',    // 打包后的文件名称
        path: path.resolve(__dirname, 'dist')  // 打包后的目录
    }
}
```

#### 多入口配置
如果有多个入口，可以将entry配置成一个array或者object，如果是array，则是将多个入口文件最终生成**一个出口**文件，如果是object，则对应生成**多个文件**，如下：
entry为array
```javascript
const path = require('path');
module.exports = {
    entry: ['./src/a.js', './src/b.js'], 多个入口文件打包为一个js文件
    output: {
        filename: 'bundle.js', 
        path: path.resolve(__dirname, 'dist')
    }
}
```
entry为object
```javascript
const path = require('path');
module.exports = {
    entry: {
        a: './src/a.js',
        b: './src/b.js'
    },
    output: {
        filename: '[name].js',      // 打包后为a.js和b.js文件
        path: path.resolve(__dirname, 'dist')
    }
}
```

## 添加loader
由于webpack只能处理js，当我们需要处理其他非js文件时，我们需要引入对应的loader

#### 加载css
我们想从javaScript 模块中 import 一个 CSS 文件，需要在 module 配置中 安装并添加 style-loader 和 css-loader：

```bash
npm install --save-dev style-loader css-loader
```

```bash
const path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ['style-loader', 'css-loader']
            }
        ]
    }
};
```
src/css/style.css
```css
body {
    background: red;
}
```
src/index.js
```javascript
 import './css/style.css';
```
打开index.html，可以看到我们页面的背景色此时为红色。此时打开index.html源码，可以看到head中有我们刚刚添加的css，此时是内联进html页面的

#### 处理图片
加载图片，也需要安装对应的loader
```bash
npm install file-loader url-loader -save -dev
```
在css引入背景图片时，需要指定一下相对路径
```javascript
module: {
    rules: [
        {
            test: /\.css$/,
            use: [
                'style-loader',
                'css-loader'
            ]
        },
        {
            test: /\.(png|svg|jpg|gif)$/,
            use: [{
                'url-loader',
                options: {
                    limit: 8192,             // 小于8k的图片自动转成base64格式
                    outputPath: 'images/'   // 图片打包后存放的目录
                }

            }]
       }
    ]
}
```
#### 加载字体文件
```javascript
module: {
    rules: [
        {
            test: /\.(woff|woff2|eot|ttf|otf)$/,
            use: [
            'file-loader'
            ]
        }
    ]
}
    
```

## webpack插件

webpack提供了各种各样的插件，下面介绍几种常用的插件，只试过在webpack4.0以上的版本，其他版本没试过，所以如果有报错，那可能是版本不支持

使用webpack插件，我们要写在配置中的plugins里面，在使用插件之前，我们都需要先安装该插件。下面介绍几个常用的插件

#### html-webpack-plugin 创建 HTML 文件
改插件的作用为：生成html页面，自动引入js文件,自动消除src引入的缓存问题，上线之前压缩，配置minify压缩代码,生成的HTML文件引入各自的JS文件配置
首先安装该插件`npm install html-webpack-plugin --save -dev`;
```javascript
let HtmlWebpackPlugin = require('html-webpack-plugin');
module: {
    ...
    plugins:[
        new HtmlWebpakPlugin({
            minify:{
                collapseWhitespace: true,   // 折叠空白区域 也就是压缩代码
                removeAttributeQuotes: true // 移除双引号，
            },
            hash:true, //向html引入的src链接后面增加一段hash值,消除缓存
            template:'./src/index.html', // 模板地址
            title: 'webpack' // 标题
        })
    ]
}
```
关于该插件的更多使用方法，可以看(这里)[https://github.com/jantimon/html-webpack-plugin#options]

#### extract-text-webpack-plugin 拆分单独的css
安装`npm i extract-text-webpack-plugin@next --save -dev`，其中@next表示可以在webpack4中使用
```javascript
const ExtractTextPlugin = require("extract-text-webpack-plugin");

module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
        use: ExtractTextPlugin.extract({
            fallback: "style-loader",
          use: "css-loader"
        })
      }
    ]
  },
  plugins: [
      new ExtractTextPlugin("css/styles.css"), // 打包后的css文件
  ]
}
```
关于该插件的更多使用方法，可以看(这里)[https://www.webpackjs.com/plugins/extract-text-webpack-plugin/]


#### 添加css3前缀
网上很多的教程都是利用postcss中的autoprefixer，但它会有一个问题，就是你不得不维护一个postcss.config.js。或者是直接在webpack.config.js的 module.rules的  postcss-loader options 里添加，但是只能添加-webkit-前缀。
其实还有一种方法，可以`optimize-css-assets-webpack-plugin`和`cssnano`一起使用，如下:
首先安装： `npm install optimize-css-assets-webpack-plugin cssnano -D`;
```javascript
// 优化重复样式、添加样式浏览器前缀
const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin');
new OptimizeCssAssetsPlugin({
    cssProcessor: require('cssnano'),
    cssProcessorOptions: {
        autoprefixer: {
        add: true,
        browsers: ['> 0%', 'ie >= 8', 'op_mini > 0', 'op_mob > 0', 'and_qq > 0', 'and_uc > 0', 'Samsung > 0']
        },
        postcssDiscardComments: {
        removeAll: true
        }
    }
})
```
webpack提供的插件非常多，更多的插件，可以看(https://webpack.js.org/plugins/)[这里]


## 转义es6
安装`npm i -D @babel/core @babel/plugin-transform-runtime @babel/preset-env babel-loader`
在webpack.config.js中添加
```javascript
..略
module:{
    rulse:[
        ...略
        {
            test: /\.js$/,
            loader: 'babel-loader',
            exclude: /node_modules/,
        }
    ]
}
```
在根目录下，也就是和webpack.config.js同一个目录下，新增.babelrc文件
```javascript
{
    "presets": ["@babel/preset-env"], 
    "plugins": ["@babel/plugin-transform-runtime"] 
}
```


## webpack-dev-server 为webpack添加静态服务器
`webpack-dev-server`为自动刷新和模块热替换机制，装上它，可以我们的改动可以自动刷新
安装`npm install webpack-dev-server -D`
修改一下我们的webpack.config.js
```javascript
var path = require('path');

module.exports = {
  //...
  devServer: {
    contentBase: path.join(__dirname, 'dist'), // 服务器资源的根目录
    compress: true, // 服务器资源采用gzip压缩
    port: 9000,  // 运行的端口
    overlay: true  // 出错代码是否显示在html页面上
    hot: true //热加载
  }
};
```
再修改我们的package.json
```json
...
 "scripts": {
    "dev": "webpack-dev-server --mode development --open",
  }
...
```
执行`npm run dev`，就会自动打开浏览器，监听你的修改了
> 注意点：`webpack-dev-server`输出的文件只存在于内存中,不输出真实的文件，也就是说，你启动它，你的dist文件其实是没有生产新的文件的


##  resolve 设置模块如何被解析

resolve是webpack自带的，主要作用是设置模块如何被解析
主要介绍几个：
1. resolve.alias 配置别名
resolve.alias配置项通过别名来把原来导入路径映射成一个新的导入路径，例如：
```javascript
resolve: {
    alias: {
        compts: './src/components/'
    }
}
```
这样，我们原来`import Dialog from './src/components/dialog'`可以缩减为`import Dialog from 'compts/dialog'`;


2. resolve.extensions 自动解析扩展，意味着我们导入模块可以省略不写后缀名
```javascript
resolve: {
    extensions: ['.js', '.json']
}
```
当我们import一个js时，可以不带js后缀，如`import $js from './js/test`，引入的就是test.js。
解析过程为：当我们`import data from './data'`时，webpack就会依次寻找data.js是否存在，不存在继续寻找data.json是否存在，最后寻找data/index.js是否存在


3. resolve.modules 





















webpack4.0+vue+es6配置
https://juejin.im/post/5c68f4e9e51d454be11473b9

参考文章：

https://juejin.im/post/5adf017f518825671d203520#heading-16

webpack中文网
https://www.webpackjs.com/guides/asset-management/#%E5%8A%A0%E8%BD%BD%E5%AD%97%E4%BD%93

webpack详解
https://juejin.im/post/5aa3d2056fb9a028c36868aa

webpack4-用之初体验，一起敲它十一遍 ***
https://juejin.im/post/5adea0106fb9a07a9d6ff6de  

webpack最佳实践
https://juejin.im/post/5b304f1f51882574c72f19b0

初探webpack4
https://blog.csdn.net/qq_16339527/article/details/80641245