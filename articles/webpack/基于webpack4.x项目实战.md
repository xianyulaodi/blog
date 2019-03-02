
> webpack在前端开发者的世界再熟悉不过了，网上也很多关于webpack的文章，自己也写一下，加深印象

webpack 是js模块打包器，一直在更新,本文是基于**webpack4.29.5**版本，将来的某一天，发觉本文章的一些配置用不了，那可能是webpack已经更新到更高的版本了

## webpack4.0的零配置

安装webpack4和webpack-cli，由于webpack4中和webpack-cli抽离了，所以需要分别安装，我们全局安装一个：

`npm install webpack webpack-cli -g  `

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

如果我们只是想要一个简单的打包功能，使用默认配置就够用了。都不需要创建`webpack.config.js`



## 自定义webpack配置

在我们的项目中，使用webpack的默认配置明显是不够用的,还是需要自定义我们的webpack配置

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
        filename: 'main.js',    // 打包后的文件名称
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
        filename: 'main.js', 
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
我们想从javaScript 模块中 import 一个 CSS 文件，需要在 module 配置中 安装并添加 `style-loader` 和 `css-loader`：

`npm install style-loader css-loader -D`

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
打开index.html，可以看到我们页面的背景色此时为红色。执行编译，css文件将会打包到js文件当中

#### 处理图片
加载图片，也需要安装对应的loader
`npm install file-loader url-loader -D`
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
            test: /\.(png|svg|jpg|gif)$/, // 加载图片
            use: [{
                loader: 'url-loader',
                options: {
                    limit: 8192,  // 小于8k的图片自动转成base64格式
                    name: 'images/[name].[ext]?[hash]', // 图片打包后存放的目录
                    publicPath: '../'  // css图片引用地址，可修正打包后，css图片引用出错的问题
                }
            }]
        },
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

#### html-webpack-plugin： 创建 HTML 文件
该插件的作用为：生成html页面，自动引入js文件,自动消除src引入的缓存问题，上线之前压缩。

使用前安装该插件`npm install html-webpack-plugin --save -dev`;
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
关于该插件的更多使用方法，可以看[这里](https://github.com/jantimon/html-webpack-plugin#options)

#### extract-text-webpack-plugin：拆分单独的css
前面安装的`css-loader`在js中引入css文件时，打包后的css是和js混合在一起的，如果我们想打包后的css文件时单独的存在，需要引入这个插件。

安装`npm i extract-text-webpack-plugin@next -D`，其中@next表示可以在webpack4中使用
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
关于该插件的更多使用方法，可以看[这里](https://www.webpackjs.com/plugins/extract-text-webpack-plugin/)


#### 添加css3前缀
利用postcss中的autoprefixer
安装`npm i postcss-loader autoprefixer -D `
在根目录下新建`postcss.config.js`
里面写入
```javascript
module.exports = {
    plugins: [
        require('autoprefixer')
    ]
}
```
webpack.config.js里面，配置`postcss-loader`即可
```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ['style-loader', 'css-loader', 'postcss-loader']
            }
        ]
    }
}
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
            exclude: /node_modules/, // 忽略掉该文件下的js
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


## webpack-dev-server：为webpack添加静态服务器
`webpack-dev-server`为自动刷新和模块热替换机制，装上它，可以我们的改动可以自动刷新
安装`npm install webpack-dev-server -D`
修改一下我们的webpack.config.js
```javascript
var path = require('path');

module.exports = {
  //...
  devServer: {
    contentBase: path.join(__dirname, 'dist'), // 服务器资源的根目录，不写的话，默认为bundle.js
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
        components: './src/components/'
    }
}
```
这样，我们原来`import Dialog from './src/components/dialog'`可以缩减为`import Dialog from 'components/dialog'`;


2. resolve.extensions 自动解析扩展，意味着我们导入模块可以省略不写后缀名
```javascript
resolve: {
    extensions: ['.js', '.json']
}
```
当我们import一个js时，可以不带js后缀，如`import $js from './js/test`，引入的就是test.js。
解析过程为：假如我们用`import data from './data'`时，webpack就会依次寻找data.js是否存在，不存在继续寻找data.json是否存在，最后寻找data/index.js是否存在


3. resolve.modules 配置 Webpack 去哪些目录下寻找第三方模块
Webpack找第三方模块，默认是只会去node_modules目录下寻找。如果你的项目中有一些模块大量被其它模块依赖和导入，由于其它模块的位置分布不定，针对不同的文件都要去计算被导入模块文件的相对路径， 这个路径有时候会很长，比如我要在node_modules下面找dialog组件，那么可能这样写：`import '../../../components/dialog'`;如果利用`resolve.modules`去优化，假如那些被大量导入的模块都在./src/components  目录下，把  modules  配置成

modules:['./src/components','node_modules']
后，你可以简单通过  `import 'dialog'`  导入。


## webpack实践
通过以上的介绍，我们大概熟悉了webpack的一些基础配置，下面我们来进入实战

我们配置具有如下功能的webpack配置：
* 支持es6
* 支持css3自动前缀
* less、css单独存为css文件，而不是打包到js中
* 支持引入图片
* 支持热跟新项目
* 支持引用别名以及省略一些文件后缀

我们目录结构如下：
——src
    ├─components
    ├─css
    └─images
    ├─index.js   
——index.html
——.babelrc
——package.json
——postcss.config.js
——webpack.config.js

webpack.config.js的配置如下：
```javascript
const path = require('path');
const htmlWebpackPlugin = require("html-webpack-plugin");
const ExtractTextPlugin = require("extract-text-webpack-plugin");
const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin');

module.exports = {
    entry: './src/index.js', // 入口文件
    output: {
        filename: 'main.js',    // 打包后的文件名称
        path: path.resolve(__dirname, 'dist')  // 打包后的目录
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ExtractTextPlugin.extract({ // 拆分单独的css文件
                    fallback: "style-loader",
                    use: ['css-loader', 'postcss-loader'] // 加载css
                })
            },
            // 加载less
            {
                test: /\.less$/,
                use: ExtractTextPlugin.extract({
                    fallback: "style-loader",
                    use: ['css-loader', 'postcss-loader']
                })
            },
            {
                test: /\.(png|svg|jpg|gif)$/, // 加载图片
                use: [{
                    loader: 'url-loader',
                    options: {
                        limit: 8192,  // 小于8k的图片自动转成base64格式
                        name: 'images/[name].[ext]?[hash]', // 图片打包后的目录
                        publicPath: '../'  // css图片引用地址
                    },
                }]
            },
            {
                test: /\.(woff|woff2|eot|ttf|otf)$/, // 加载字体文件
                use: [
                    'file-loader'
                ]
            },
            // 转义es6
            {
                test: /\.js$/,
                loader: 'babel-loader',
                include: /src/,          // 只转化src目录下的js
                exclude: /node_modules/, // 忽略掉node_modules下的js
            }
        ]
    },
    resolve: {
        alias: {
            components: path.resolve(__dirname, 'src/components/') // 别名
        },
        extensions: ['.js', '.json'], // 忽略文件后缀
        modules: ['node_modules']
    },
    plugins: [
        new htmlWebpackPlugin({
            template: "./index.html",
            filename: "index.html",
            inject: true,
            hash: true,
            chunksSortMode: 'none' //如使用webpack4将该配置项设置为'none'
        }),
        new ExtractTextPlugin("css/styles.css"), 
        new OptimizeCssAssetsPlugin({ // 优化css
            cssProcessor: require('cssnano'), //引入cssnano配置压缩选项
            cssProcessorOptions: {
                discardComments: { removeAll: true }
            },
            canPrint: true //是否将插件信息打印到控制台
        })
    ],
    devServer: {
        hot: true,
        contentBase: path.join(__dirname, 'dist'),
        port: 3002,
    },
};
```
代码地址: https://github.com/xianyulaodi/demos/tree/master/webpack/webpack-basic

## 后记
目前vue、react都有自己的webpack配置，都已经配好了，直接拿来用即可。不过作为一个优秀的前端，我们也需要知道如何从零开始配置属于自己的webpack.config.js。 后续写一个只需配置一次，可以多个项目公共的webpack配置，敬请期待...




参考文章：
https://blog.csdn.net/qq_16339527/article/details/80641245
https://juejin.im/post/5aa3d2056fb9a028c36868aa
https://juejin.im/post/5adea0106fb9a07a9d6ff6de  
https://juejin.im/post/5b304f1f51882574c72f19b0
