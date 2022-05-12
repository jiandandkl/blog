---
uuid: 6a4b3770-d200-11ec-b1cf-75316e66aa07
author: jiandandkl
email: emolingzhu@126.com
github: https://github.com/jiandandkl
avatar: https://avatars.githubusercontent.com/u/16009933?v=4
title: webpack热加载优化及vite踩坑记录
date: 2022-05-12 22:32:57
tags:
---

## 一、背景

三年前的老项目，技术栈是react + webpack4，融合了很多模块，项目巨大无比，每次更改代码都得等待20s左右，实在有点怀疑人生。接手项目的第一天就想有机会一定要优化，随着对项目的了解和工作节奏的适应，最近开始着手优化项目热加载。

## 二、webpack4的优化

已经做的优化

-   DllPlugin
-   babel-loader设置缓存

尝试做的优化

-   react-hot-loader
-   hard-source-webpack-plugin
-   react-refresh

试了若干种方案后，效果甚微；后来就某个问题场外求助时，被安利了esbuild和Vite，看了esbuild首页夸张的数据对比，抱着将信将疑的态度尝试了Vite（真香）。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce5b90db06a2497fb13c76732578348c~tplv-k3u1fbpfcp-zoom-1.image)

## 三、迁移至Vite

### 1. vite简介

Vite对于第三方库使用esbuild预构建依赖，Esbuild 使用 Go 编写，速度比以 JavaScript 编写的打包器预构建依赖快 10-100 倍；以原生ESM方式提供源码，没有了webpack的编译过程，对webpack可以说是降维打击...

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/382d39de7cb240deb5cef26cfb0af956~tplv-k3u1fbpfcp-zoom-1.image)\
webpack团队成员的中文回复..

### 2. 迁移步骤
2.1 根目录创建 Vite.config.js，完整配置：

```javascript

import { defineConfig } from 'vite';
import reactRefresh from '@vitejs/plugin-react-refresh';
import antdDayjs from 'antd-dayjs-vite-plugin';

function ttt (){
}

export default defineConfig({
  plugins: [
    reactRefresh(),
    // 处理antd日期组件只展示今天的问题
    antdDayjs({ preset: 'antdv3' })
  ],
  base: './',
  resolve: {
    extensions: ['.mjs', '.js', '.ts', '.jsx', '.tsx', '.json'],
    alias: [
      { find: '@src', replacement: ’xxxx },
      { find: 'react-native', replacement: 'react-native-web' },
      { find: '@ant-design/icons/lib/dist', replacement: '@ant-design/icons/lib/index.es.js' }
    ]
  },
  optimizeDeps: {
    // exclude: ['bizcharts']
  },
  css: {
    preprocessorOptions: {
      less: {
        javascriptEnabled: true
      }
    }
  },
  server: {
    port: 3000,
    host: 'localhost',
    open: '/',
    proxy: {
      '/api': {
        target: 'xxx',
        changeOrigin: true
      }
    }
  },
  define: {
    'process.env': {
      // 计划只在开发环境使用,写死了dev
      NODE_ENV: 'development'
    }
  }
});

```


2.2 复制index.html至根目录

添加 `<script type="module" src="/src/Main.jsx"></script> `  

2.3 package.json 添加启动命令

  `"start:Vite": "Vite"`

至此Vite配置已告一段落，可以尝试跑下 "start:Vite": "Vite"，会发现各种报错，下面开始填坑之旅。

> 2.4为开发环境为vite，线上保持webpack的兼容处理

2.4 让webpack处理 import.meta.globEager 和 import.meta.glob

```js
test: /.(js|mjs|jsx|ts|tsx)$/,
use: [
  { loader: require.resolve('@open-wc/webpack-import-meta-loader') },
  {
    loader: require.resolve('babel-loader'),
    options: {
      presets: ['babel-preset-vite'],
      plugins: [
        [require('@babel/plugin-syntax-import-meta')]
      ]
    }
 	}
]
```

### 3. 遇到的问题

##### 3.1 Cannot find module 'worker_threads'  

node ≥ 12

##### 3.2 Unknown theme type: undefined, name: undefined

```js
alias: {
    '@ant-design/icons/lib/dist': '@ant-design/icons/lib/index.es.js'
},
```

##### 3.3 require is not defined，

如backgroundimage:`url(${require('common/Asset/icon${item.index_page}.svg')})`

可以写个获取图片的工具方法


```js
const getImageUrl = imageName => {
    // 因为import.meta.url会加上本模块所在的目录 src/utils,所以加上../../
    return `${new URL(`../../src/${imagePath}`, import.meta.url).href}`;
};
backgroundimage:`url(${getImageUrl(`${item.index_page}.svg`)})`
```

##### 3.4 error: No matching export in "node_modules/@antv/util/esm/index.js" for import "max"

@antv/util 升级到 2.0.17

##### 3.5 The requested module 'xxx' does not provide an export named 'xx'

exports 改为export

##### 3.6 Failed to fetch dynamically imported module

改用import.meta.glob

##### 3.7  process is not defined

```js
// vite.config.js
define: {
  'process.env': {
    NODE_ENV: 'development'
  }
}
```

##### 3.8 antd的日期组件只显示了今天

Vite.config.js添加 plugins: [antdDayjs({ preset: 'antdv3' })]  (非v3版本不用preset参数）

##### 3.9 import.meta.glob webpack build 报错

```js
npm install --save-dev babel-preset-vite @babel/plugin-syntax-import-meta 
presets: ['babel-preset-vite'],
plugins: ['@babel/plugin-syntax-import-meta']
```

##### 3.10 import.meta.url webpack build 报错

添加`@open-wc/webpack-import-meta-loader`

##### 3.11 Dynamic require of "xxx.css" is not supported

改了 yimi-form-components 的样式。。。。

```js
// require("antd/lib/table/style/index.css");
import "antd/lib/table/style/index.css"
```

##### 3.12 The requested module '/node_modules/.vite/react-intl.js?v=94a2fd86' does not provide an export named 'intlShape'

这个问题很奇怪，'intlShape'是react-intl的一个interface，在vite编译后的文件里确实没有，于是想看是哪个组件使用了，一个个试了发现是有的组件引用了'intlShape'但是没有使用，把这样的'intlShape'删除后项目就可以启动了..

##### 3.13 GET http://localhost:9021/@id/dayjs/moment net::ERR_ABORTED 404 (Not Found)

组件中有 `import moment from 'moment/moment';`改为 `import moment from 'moment‘;`即可

##### 3.14 打开页面后发出了2000多个http请求，完全渲染完成依然需要9s

查看了下请求的文件，大多是swagger生成的services文件，因services是通过一个文件export出来的，所以使用的时候就是引用了所有的services文件，查看了services下确实有1800多个文件。

看了vite文档中有介绍：

> 一些包将它们的 ES 模块构建作为许多单独的文件相互导入。例如，[lodash-es 有超过 600 个内置模块](https://unpkg.com/browse/lodash-es/)！当我们执行 import { debounce } from 'lodash-es' 时，浏览器同时发出 600 多个 HTTP 请求！尽管服务器在处理这些请求时没有问题，但大量的请求会在浏览器端造成网络拥塞，导致页面的加载速度相当慢。通过预构建 lodash-es 成为一个模块，我们就只需要一个 HTTP 请求了！

所有可以把类似的模块加入预构建，比如这里的services，还有翻译相关的

```js
  optimizeDeps: {
    include: ['@src/services', '@src/Locales/zh_CN', '@src/Locales/en_US']
  }
```

##### 3.15 vite一直reload，new dependencies .. 的提示

由于动态import，预构建可能在项目启动后继续发生，但避免vite.config.js手动配置多个依赖，使用vite-plugin-optimize-persist插件可以把配置写到package.json\


```js
// vite.config.ts
import OptimizationPersist from 'vite-plugin-optimize-persist'
import PkgConfig from 'vite-plugin-package-config'

plugins: [
  PkgConfig(),
  OptimizationPersist()
]
```

## 四、效果

|       | webpack | Vite |
| ----- | ------- | ---- |
| 第一次启动 | 45s     | `20s`  |
| 第二次启动 | 45s     | `1s`   |
| 热加载    | 20s     | `1s`   |

## 五、总结

计划是对webpack做些优化的，但试了多种方法都没啥效果，所以不如直接上Vite，~~投入的时间差不多~~（后面陆续填了好多坑），但收获的开发体验是极致的。

## 六、参考

[玩转 webpack，使你的打包速度提升 90%
](https://juejin.cn/post/6844904071736852487)

[vite官方文档](https://cn.Vitejs.dev) 

[vue-cli3 vue2 保留 webpack 支持 vite 成功实践](https://www.cnblogs.com/EnSnail/p/15707038.html) 


[6年的老项目迁移vite2，提速几十倍，真香
](https://juejin.cn/post/7005479358085201957)

[vite 动态 import 引入打包报错解决方案
](https://juejin.cn/post/6951557699079569422) 

[浅谈 Vite 2.0 原理，依赖预编译，插件机制是如何兼容 Rollup 的？](https://ssh-blog.vercel.app/812799294)
