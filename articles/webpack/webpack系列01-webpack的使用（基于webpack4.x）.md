
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
        },
        {
            test: /\.(woff|woff2|eot|ttf|otf)$/,
            use: [
            'file-loader'
            ]
        }
    ]
}
    
```












参考文章：

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