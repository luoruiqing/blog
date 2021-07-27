---
title: "vue3的vue-grid-layout支持"
subtitle: "vue-grid-layout ｜ vue3 ｜ 支持"
layout: post
author: "luoruiqing"
header-style: text
tags:
  - Vue
---


vue-grid-layout 包用于自定义布局使用，但是目前只支持Vue2， 对Vue3也只是试验性的例子，按照官方的安装方式在Vue3下依然会有错误的提醒:


```log
Uncaught (in promise) TypeError: external_commonjs_vue_commonjs2_vue_root_Vue_default.a is not a constructor
at Proxy.created
```

首先应该尝试安装官方版本的vue3分支的包， 安装命令如下：

```sh
yarn add https://github.com/jbaysolutions/vue-grid-layout.git#vue3
```

若还是有相同的错误, 可以尝试另一个版本， 使用过官方库fork出来的版本。

```sh
yarn add https://github.com/lvjunhao/vue-grid-layout
```