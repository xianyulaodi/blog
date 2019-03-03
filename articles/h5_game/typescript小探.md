> 最近项目中可能会用白鹭引擎开发h5小游戏，而该引擎的h5小游戏需要基于typescript，而且typescript应该是未来的趋势，让我们来学习之

## typescript简介

#### 1. 什么是typescript
官方解释：TypeScript是Javascript类型的超集，它可以编译成纯javascript。可以在任何浏览器、任何计算机和任何操作系统上运行，而且是开源的。
通俗一点的解释：TypeScript 是 JavaScript 的强类型版本，本质上是类似于css的less、sass，都是为了易于维护、开发，最后还是编译生成纯粹的 JavaScript 代码。
由于它最终在浏览器中运行的仍然是 JavaScript，所以 TypeScript 并不依赖于浏览器的支持，也并不会带来兼容性问题。

#### 2. 为什么要用typescript
javascript属于弱类型语言，什么意思呢，打个比方
```javascript
var test = 1;
test = 'aaa';
```
上述代码中，test刚开始是一个number，后来存储为string，它的类型由当前的值来决定，这就是弱类型语言。

我们来看下面代码：
```javascript
jquery.ajax(url,setting);
```
使用jq的ajax，传入两个参数，那么参数是什么类型呢？如果不往下继续看代码，我们只能靠猜，url可能是一个字符串，setting它可能是一个对象，也可能是一个数组。
要知道这两个参数是什么类型，我们只能继续读里面的代码，或者看注释

而如果用typescript
```javascript
jquery.ajax(url:string, setting?: JQAjaxSetting): JQXHR;

interface JQAjaxSetting {
    sync?: boolean;
    cache?: boolean;
    contentType?: any
    ...
}

interface JQXHR {
    responseJSON?: any;
    ...
}
```
通过typescript，我们可以知道url是一个字符串，setting设置参数是可选的，而且还知道setting参数里面的类型等等，更加的清晰明了，而不必深入到实现或读取文档中。

又打个比方：比如一个字段的类型，我们一开始定义为number类型，突然某一天，后端同学抽风了，说都要换成string，我们需要改动他的类型，如果此时用typescript强大的强类型编译器检验，那些不传string的参数变便会报了一堆错，而我们此时只要根据报错改掉相应的参数即可。

也就是说，typescript可以帮助我们更快地阅读类型化代码，更好的理解代码、更严格的类型检查，它使我们的开发更加的严谨和自由（因为你再需要向别人解析需传什么类型的参数）


## typescript的使用

#### 安装typescript
`npm install -g typescript`，需cmd管理员身份运行安装

#### 编写typescript文件
新建`hello_world.ts`文件，并写入如下代码：
```javascript
function hello(text: string) {
    return "Hello, " + text;
}

let text = "world";

document.body.innerHTML = hello(text);
```
命令行输入`tsc hello_world.ts`，我们可以看到，输出了一个名为hello_world.js文件，里面是经过编译后的typescript，就是普通的js代码

## typescript的一些语法
## typescript一些常用的工具
## 基于typescript的项目实战


参考：
TypeScript - 不止稳，而且快
https://segmentfault.com/a/1190000010391598

浅谈TypeScript
https://www.cnblogs.com/lovesong/p/4947919.html

typescript中文网
https://www.tslang.cn
http://www.typescriptlang.org/ 官网
https://www.cnblogs.com/Leo_wl/p/5751187.html