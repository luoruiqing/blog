---
title: "Webpack4 babel 配置"
subtitle: ""
layout: post
author: "luoruiqing"
header-style: text
tags:
  - JavaScript
  - Webpack
---

`Webpack4`使用`babel`将新特性的代码转换为旧版浏览器可以兼容的JS代码, 安装步骤如下:

- 安装`babel`及依赖
- 安装`babel-polyfill`包
- 配置`babel`项
- 打包测试

---

### 一、安装`babel`及依赖

```sh
npm install -D babel-loader @babel/core @babel/preset-env
```

输出如下:
```
➜  js git:(dev) ✗ npm install -D babel-loader @babel/core @babel/preset-env
+ @babel/core@7.9.6
+ @babel/preset-env@7.9.6
+ babel-loader@8.1.0
added 92 packages from 13 contributors and updated 2 packages in 10.698s
```

### 二、安装`babel-polyfill`包

如果出现 `regeneratorRuntime is not defined` 错误, 则需要此步骤, 否则可以忽略

```sh
npm i -D babel-polyfill
```
输出如下

```
➜  js git:(dev) ✗ npm i -D babel-polyfill
+ babel-polyfill@6.26.0
added 2 packages from 2 contributors in 11.161s
```


### 三、配置`babel`项

#### webpack.config.js

```js
module.exports = {
  // 多页面情况
  entry: {
      sdk: ['babel-polyfill', './src/main.js'], // 入口A
      iframe: ['babel-polyfill', './src/iframe.js'], // 入口B
  },
  // 单页情况
  // entry: ['babel-polyfill', './src/main.js'], // 入口
  module: {
      rules: [
          {
              test: /\.js$/,  // 只编译js文件
              exclude: /node_modules/, // 忽略安装包
              loader: "babel-loader" // 指定loader
          }
      ]
  }
}

```

#### package.json

注意`package.json`文件必须为标准的JSON文件

```json
{
  "babel": {
    "presets": [
      "@babel/preset-env"
    ]
  }
}
```

`.babelrc`配置形式的与`package.json`配置差别不大, 自行搜索对应


### 四、 打包测试

```sh
npm run build
```


### 查阅
- [https://www.cnblogs.com/herewego/p/9308067.html](https://www.cnblogs.com/herewego/p/9308067.html)
- [https://www.cnblogs.com/Joe-and-Joan/p/10335881.html](https://www.cnblogs.com/Joe-and-Joan/p/10335881.html)
- [https://blog.csdn.net/joyvonlee/article/details/96507604](https://blog.csdn.net/joyvonlee/article/details/96507604)
- [https://www.jianshu.com/p/ce28ceddda72](https://www.jianshu.com/p/ce28ceddda72)
- [https://www.jianshu.com/p/67a158088807](https://www.jianshu.com/p/67a158088807)
- [https://www.babeljs.cn/docs/babel-parser](https://www.babeljs.cn/docs/babel-parser)
