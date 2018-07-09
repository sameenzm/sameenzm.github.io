---
title: Code Splitting[译]
date: 2018-06-11 21:14:55
tags:
- webpack
---

## webpack的代码拆分

工作中，随着需求不断迭代，业务代码量也越来越大，明显感觉到开发过程中代码编译速度太慢啦! 就算改一点样式代码，编译也得六万多毫秒，一分多钟，简直忍不了┭┮﹏┭┮...所以研究一下改进webpack编译速度的方法，其中，看到webpack的文档有这么一篇指南：Code Splitting，故翻译学习。

[原文地址](https://webpack.js.org/guides/code-splitting)

### 正文开始
`本指南(guide)扩展(extends)了`[入门(Getting Started)](https://webpack.js.org/guides/getting-started)`和`[输出管理(Output Management)](https://webpack.js.org/guides/output-management)`中提供的示例。请确保你至少熟悉其中提供的示例。`
<!-- more -->

代码拆分是webpack最引人注目的特性(features)之一。这个特性允许你 将你的代码分成不同的bundle，然后可以按需(on demand)或并行(in parallel)加载。它可以用来实现更小的打包文件 和 控制资源负载的优先级(prioritization)，如果使用得当，可以对加载时间产生重大影响(major impact)。

这里有三种可用的 代码拆分 的一般方法(general approaches)：
- 入口点(Entry)：使用[entry](https://webpack.js.org/configuration/entry-context/)配置 手动地(Manually)拆分代码
- 防止重复(Duplication)： 使用 [SplitChunks](https://webpack.js.org/plugins/split-chunks-plugin/) 重复数据删除(dedupe) 和拆分 块(chunks)
- 动态导入(Dynamic Imports)： 通过模块内(within modules)的内联函数(inline function)调用来分割代码


### 入口点
这是迄今为止(by far)最简单、最直观(intuitive)的分解代码的方式。但是，这更多的是手动的，并且有一些我们会经历的陷阱。我们来看看我们如何从主包中拆分另一个模块：

project

```
webpack-demo
|- package.json
|- webpack.config.js
|- /dist
|- /src
  |- index.js
+ |- another-module.js
|- /node_modules

```

another-module.js

```
import _ from 'lodash';

console.log(
  _.join(['Another', 'module', 'loaded!'], ' ')
);

```

webpack.config.js

```
const path = require('path');

module.exports = {
  mode: 'development',
  entry: {
    index: './src/index.js',
+   another: './src/another-module.js'
  },
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};

```

这将产生(yield)以下构建结果：

```
Hash: a948f6cc8219cc2d39a1
Version: webpack 4.7.0
Time: 323ms
            Asset     Size   Chunks             Chunk Names
another.bundle.js  550 KiB  another  [emitted]  another
  index.bundle.js  550 KiB    index  [emitted]  index
Entrypoint index = index.bundle.js
Entrypoint another = another.bundle.js
[./node_modules/webpack/buildin/global.js] (webpack)/buildin/global.js 489 bytes {another} {index} [built]
[./node_modules/webpack/buildin/module.js] (webpack)/buildin/module.js 497 bytes {another} {index} [built]
[./src/another-module.js] 88 bytes {another} [built]
[./src/index.js] 86 bytes {index} [built]
    + 1 hidden module

```

如前所述(As mentioned)，这种方法存在一些缺陷(pitfalls):
- 如果在入口块之间有任何重复的模块，他们将被包含在两个包中。
- 它不够灵活，不能用于动态地(dynamically)将代码与核心应用程序逻辑分离。

这两点中的第一点对于我们的例子来说无疑(definitely)是个问题，因为 lodash 也是在 ./src/index.js 中导入的，因此会在这两个包重复使用。让我们通过使用 SplitChunks 插件删除这个重复。


### 防止重复
SplitChunks允许我们将公共依赖关系提取(extract)到现有的entry块或全新的块中。用它来解决前面例子中的lodash依赖重复的问题：

webpack.config.js

```
  const path = require('path');

  module.exports = {
    mode: 'development',
    entry: {
      index: './src/index.js',
      another: './src/another-module.js'
    },
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
+   optimization: {
+     splitChunks: {
+       chunks: 'all'
+     }
+   }
  };
```

使用SplitChunks后，我们现在应该看到从 index.bundle.js中删除了重复的依赖项。该插件应该注意到我们已经将lodash分离出来，并将其从主包中移除。让我们npm run build来看看它是否工作：

```
Hash: ac2ac6042ebb4f20ee54
Version: webpack 4.7.0
Time: 316ms
                          Asset      Size                 Chunks             Chunk Names
              another.bundle.js  5.95 KiB                another  [emitted]  another
                index.bundle.js  5.89 KiB                  index  [emitted]  index
vendors~another~index.bundle.js   547 KiB  vendors~another~index  [emitted]  vendors~another~index
Entrypoint index = vendors~another~index.bundle.js index.bundle.js
Entrypoint another = vendors~another~index.bundle.js another.bundle.js
[./node_modules/webpack/buildin/global.js] (webpack)/buildin/global.js 489 bytes {vendors~another~index} [built]
[./node_modules/webpack/buildin/module.js] (webpack)/buildin/module.js 497 bytes {vendors~another~index} [built]
[./src/another-module.js] 88 bytes {another} [built]
[./src/index.js] 86 bytes {index} [built]
    + 1 hidden module
```

以下是社区提供的用于拆分代码的其他一些有用的插件和加载器：
- [mini-css-extract-plugin](https://webpack.js.org/plugins/mini-css-extract-plugin/):用于将css从主应用程序中分离出来
- [bundle-loader](https://webpack.js.org/loaders/bundle-loader/): 用于分割代码并延迟加载结果包
- [promise-loader](https://github.com/gaearon/promise-loader):类似于bundle-loader，但用了promises

[CommonsChunkPlugin](https://webpack.js.org/plugins/commons-chunk-plugin/)`还可以用`[explicit vendor chunks](https://webpack.js.org/plugins/commons-chunk-plugin/#explicit-vendor-chunk)`，从核心应用程序代码拆分vendor 模块。`


### 动态导入
当涉及动态代码拆分时，webpack支持两种类似的技术。第一种和推荐的方法是使用符合[ECMAScript提案](https://github.com/tc39/proposal-dynamic-import)的[import()](https://webpack.js.org/api/module-methods/#import-)语法来进行动态导入。传统的(The legacy),特定于webpack的方法是使用 [require.ensure](https://webpack.js.org/api/module-methods/#require-ensure)。让我们尝试使用者两种方法的第一种。


`import（）调用在内部使用`[promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)`，如果您在旧浏览器中使用import（），请记住使用`[es6-promise](https://github.com/stefanpenner/es6-promise)`或`[promise-polyfill](https://github.com/taylorhakes/promise-polyfill)`等polyfill来支持Promise。`


在我们开始之前，让我们从我们的配置中删除额外的 entry和[optimization.splitChunks](https://webpack.js.org/plugins/split-chunks-plugin/)，因为下一个演示(demonstration)不需要它们:

webpack.config.js

```
  const path = require('path');

  module.exports = {
    mode: 'development',
    entry: {
+     index: './src/index.js'
-     index: './src/index.js',
-     another: './src/another-module.js'
    },
    output: {
      filename: '[name].bundle.js',
+     chunkFilename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
-   optimization: {
-     splitChunks: {
-       chunks: 'all'
-     }
-   }
  };
```

请注意使用chunkFilename,它决定了non-entry 块文件的名称，有关chunkFilename的更多信息，请参阅 output documentation。我们还会更新我们的项目以删除现在未使用的文件：

project

```
webpack-demo
|- package.json
|- webpack.config.js
|- /dist
|- /src
  |- index.js
- |- another-module.js
|- /node_modules

```

现在，我们将使用动态导入来分隔(separate)块，而不是静态导入lodash:

src/index.js

```
- import _ from 'lodash';
-
- function component() {
+ function getComponent() {
-   var element = document.createElement('div');
-
-   // Lodash, now imported by this script
-   element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+   return import(/* webpackChunkName: "lodash" */ 'lodash').then(_ => {
+     var element = document.createElement('div');
+     var _ = _.default;
+
+     element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+
+     return element;
+
+   }).catch(error => 'An error occurred while loading the component');
  }

- document.body.appendChild(component());
+ getComponent().then(component => {
+   document.body.appendChild(component);
+ })
```

请注意在评论中使用webpackChunkName。这将导致我们的单独的bundle被命名为 lodash.bundle.js，而不仅仅是 [id] .bundle.js。有关webpackChunkName和其他可用选项的更多信息，请参阅import（）文档。让我们运行webpack查看lodash分离出来的一个单独的包：

```
Hash: a3f7446ffbeb7fb897ff
Version: webpack 4.7.0
Time: 316ms
                   Asset      Size          Chunks             Chunk Names
         index.bundle.js  7.88 KiB           index  [emitted]  index
vendors~lodash.bundle.js   547 KiB  vendors~lodash  [emitted]  vendors~lodash
Entrypoint index = index.bundle.js
[./node_modules/webpack/buildin/global.js] (webpack)/buildin/global.js 489 bytes {vendors~lodash} [built]
[./node_modules/webpack/buildin/module.js] (webpack)/buildin/module.js 497 bytes {vendors~lodash} [built]
[./src/index.js] 394 bytes {index} [built]
    + 1 hidden module
```

由于import（）返回一个promise，它可以用于异步函数。然而，这需要使用Babel和Syntax Dynamic Import Babel Plugin等预处理器。以下介绍了它将如何简化代码：

src/index.js

```
- function getComponent() {
+ async function getComponent() {
-   return import(/* webpackChunkName: "lodash" */ 'lodash').then(_ => {
-     var element = document.createElement('div');
-
-     element.innerHTML = _.join(['Hello', 'webpack'], ' ');
-
-     return element;
-
-   }).catch(error => 'An error occurred while loading the component');
+   var element = document.createElement('div');
+   const _ = await import(/* webpackChunkName: "lodash" */ 'lodash');
+
+   element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+
+   return element;
  }

  getComponent().then(component => {
    document.body.appendChild(component);
  });

```

### 预取/预加载模块
webpack 4.6.0+增加了对预取和预加载的支持。
在声明导入时使用这些内联指​​令允许webpack输出“资源提示”，告诉​​浏览器：
- prefetch: 未来某些导航可能需要的资源
- preload: 在当前导航期间可能需要的资源

简单的prefetch示例可以有一个HomePage组件，该组件可以呈现一个LoginButton组件，然后在点击后按需加载LoginModal组件。

LoginButton.js

```
import(/* webpackPrefetch: true */ 'LoginModal');
```


这将导致<link rel="prefetch" href="login-modal-chunk.js">被附加在页面的头部，这将指示浏览器在空闲时间预取login-modal-chunk.js文件。

`一旦父块已被加载，webpack将添加prefetch提示(hint:暗示，示意)。`

与prefetch相比，Preload(预加载)指令有很多不同之处：
- preload的块开始与父块并行加载。prefetch块在父块完成后开始。
- preload的块具有中等优先级并立即下载。prefetch块在浏览器空闲时间下载。
- 父块应立即请求preload的块。prefetch的块可以在将来随时使用。
- 浏览器支持不同

简单的Preload示例可以是 有一个组件，它总是 依赖于 应该位于单独块中的 大型库。

让图像组件ChartComponent需要巨大的ChartingLibrary。它在渲染时显示一个LoadingIndicator，并立即按需导入ChartingLibrary：

ChartComponent.js

```
import(/* webpackPreload: true */ 'ChartingLibrary');
```

当请求使用ChartComponent的页面时，通过通过<link rel="preload">，charting-library-chunk也被请求了，假设页面块较小并且速度更快，该页面将显示一个LoadingIndicator，直到已经请求的charting-library-chunk完成。这样可以缩短加载时间，因为它只需要请求(往返round-trip)一次而不是两次。特别是在高延迟(high-latency)环境中。

`错误地(incorrectly)使用webpackPreload实际上可能会损害性能(hurt performance)，所以在使用它时要小心`

### Bundle 分析
一旦开始分解代码，分析输出以检查模块结束的位置会很有用。 [official analyze tool](https://github.com/webpack/analyse) 是一个很好的开始。还有其他一些社区支持的选项：
- [webpack-chart](https://alexkuz.github.io/webpack-chart/): webpack统计的交互饼图
- [webpack-visualizer](https://chrisbateman.github.io/webpack-visualizer/): 可视化并分析你的 bundles，看看哪些模块占用了空间，哪些可能是重复的。
- [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer): 插件和CLI实用程序，它将bundle内容表示为方便的交互式可缩放树形图


### 下一步
有关如何在实际应用程序中使用import（）以及如何更有效地分解代码的[缓存](https://webpack.js.org/guides/caching)，请参阅[Lazy Loading](https://webpack.js.org/guides/lazy-loading)。


### 进一步(Further)阅读
- [<link rel=”prefetch/preload”> in webpack](https://medium.com/webpack/link-rel-prefetch-preload-in-webpack-51a52358f84c)
- [Preload, Prefetch And Priorities in Chrome](https://medium.com/webpack/link-rel-prefetch-preload-in-webpack-51a52358f84c)
- [Preloading content with rel="preload"](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content)

