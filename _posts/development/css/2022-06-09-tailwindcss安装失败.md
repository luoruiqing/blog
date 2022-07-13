---
title:        "tailwindcss 安装失败的问题"
subtitle:     "tailwindcss | vue2 | 安装 | postcss7"
layout:       post
author:       "luoruiqing"
header-style: text
tags:
  - CSS
---


Vue2 新安装 `tailwindcss` 经常会发生这个问题


```log
project:(master) ✗ vue add tailwind
project:(master) ✗ yarn serve
------------------------------------------------------------------------------------------------------------------------------------------------
yarn run v1.22.10
$ vue-cli-service serve
 INFO  Starting development server...
98% after emitting CopyPlugin

 ERROR  Failed to compile with 1 error                                                                                               

 error  in ./src/assets/tailwind.css

Syntax Error: Error: PostCSS plugin tailwindcss requires PostCSS 8.
Migration guide for end-users:
https://github.com/postcss/postcss/wiki/PostCSS-8-for-end-users


 @ ./src/assets/tailwind.css 4:14-166 15:3-20:5 16:22-174
 @ ./src/main.js
 @ multi (webpack)-dev-server/client?http://10.26.13.98:8080&sockPath=/sockjs-node (webpack)/hot/dev-server.js ./src/main.js

```

错误信息 **Syntax Error: Error: PostCSS plugin tailwindcss requires PostCSS 8.**


### 解决方法

```sh
yarn add -D tailwindcss@npm:@tailwindcss/postcss7-compat "postcss@^7" "autoprefixer@^9"
```


### CDN

还可以通过CDN完成引入，比安装更加轻量。


```html
<script src="https://cdn.tailwindcss.com"></script>
<script>
    tailwind.config = {
      prefix: 'tw-',
    }
</script>
```