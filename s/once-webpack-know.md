### <center>一更新文--假如了解webpack</center>
>  webpack is a static module bundler for modern JavaScript applications.

`复杂的情况下就是枯燥的兼容优化。`  
我对这个东西的了解仅存在于会配置...
**我还知道菜市场上比较多的loader,比如**
- `css-loader`
- `sass-loader， node-sass（编译sass的）`
-  `less-loader`
-  `postcss-loader（自动添加浏览器前缀的）`
-  `url-loader`解依赖 file-loader，当图片小于 limit 值的时候，会将图片转为 base64 编码，大于 limit 值的时候依然是使用 file-loader 进行拷贝
-  `file-loader`解决图片引入问题，并将图片 copy 到指定目录，默认为 dist
-  `img-loader`	压缩图片
-  `ts-loader`
-  `babel-loader @babel/core @babel/preset-env` es5以上版本转换为es5
-  `style-loader` 
-  `cache-loader`缓存一些性能开销比较大的 loader 的处理结果; 缓存地址（node_modules/.cache/cache-loader）
额 好像也就这点...  
**还有些插件**
- `html-webpack-plugin`
- 压缩js的插件， 
- 还有那个清空的插件`clean-webpack-plugin`  
- 哦哦哦 可能还要知道却换环境的一个工具， `cross-env`, 就是我们写在package.json里面的那个, `cross-env NODE_ENV=dev|prod|test|uat`
- `mini-css-extract-plugin`这个就是把css文件通过link的方式引用（原来直接就是style标签）
- `optimize-css-assets-webpack-plugin` 压缩css的
- `webpack-bundle-analyzer` 打包后查看大小的
- `speed-measure-webpack-plugin` 构建费时分析
#### Source Map
> 输入 => 转换器 => 输出，经过这一过程后，线上的代码已经被压缩或者混淆，虽然可能代码体积减小了，减少了网络开销，但是对于开发者调试来说确是无比痛苦的。压缩后的代码调试成了掉头发的难题。
一种反映射关系， 通过特殊的编码来实现，主要吧源代码分为组和段两个概念， 例如段用'；' 组用‘，’来表示，这样合成特殊编码

#### 项目优化
从宏观角度去分析项目进行优化， 
- 项目编写时
- 项目打包时候
  - 代码抽离
  - 利用cdn而非众多依赖增加开销
  - 
- 项目运行时候
  - 常见的首屏加载
  - 路由懒加载
  - 图片预加载、懒加载
  - 缓存
#### 构建速度优化
额... 就是无非按照人家提供的东西多去使用呗，避免我们把我们的习惯强加之勉。  
借鉴大佬之文。 [作者：ITEM](https://juejin.cn/post/7023242274876162084)  
-  resolve 配置（公共路径）
   -  modules 告诉 webpack 解析模块时应该搜索的目录，常见配置如下
        ```js 
        const path = require('path');

        // 路径处理方法
        function resolve(dir){
            return path.join(__dirname, dir);
        }

        const config = {
            //...
            resolve: {
                modules: [resolve('src'), 'node_modules'],
            },
        };
        ///告诉 webpack 优先 src 目录下查找需要解析的文件，会大大节省查找时间
        ```
   - resolveLoader
        > 一般情况下， 你需要下载 各种各样的loader 来进行代码匹配处理规则，但 当我们要使用自己的开发的loader时， 总不能现发布一个npm包吧， 这就是一个连接器。
        这个就是加载本地的loder。
        ```js
        const path = require('path');

        // 路径处理方法
        function resolve(dir){
        return path.join(__dirname, dir);
        }

        const config = {
        //...
        resolveLoader: {
            modules: ['node_modules',resolve('loader')]
        },
        };

        ```
   - externals
      > `externals` 配置选项提供了「从输出的 bundle 中排除依赖」的方法。此功能通常对 library 开发人员来说是最有用的，然而也会有各种各样的应用程序用到它。  

     - 例如，从 CDN 引入 jQuery，而不是把它打包：
        ```js
            // 引入
            <script
                src="https://code.jquery.com/jquery-3.1.0.js"
                integrity="sha256-slogkvB1K3VOkzAI8QITxV3VzpOnkeNVsKvtkYLMjfk="
                crossorigin="anonymous"
            ></script>
            // 配置
            const config = {
                //...
                externals: {
                    jquery: 'jQuery',
                },
            };

            // 使用
            import $ from 'jquery';
            $('.my-element').animate(/* ... */);
        ```
     - 更精准的使用
        > 在配置 loader 的时候，我们需要更精确的去指定 loader 的作用目录或者需要排除的目录，通过使用 include 和 exclude 两个配置项，可以实现这个功能，常见的例如：
        1. include：符合条件的模块进行解析
        2. exclude：排除符合条件的模块，不解析
        3. exclude 优先级更高
        ```js
            const path = require('path');
            // 路径处理方法
            function resolve(dir){
            return path.join(__dirname, dir);
            }

            const config = {
            //...
            module: { 
                noParse: /jquery|lodash/,
                rules: [
                {
                    test: /\.js$/i,
                    include: resolve('src'),
                    exclude: /node_modules/,
                    use: [
                    'babel-loader',
                    ]
                },
                // ...
                ]
            }
            };
        ``` 
     - noParse
       不需要解析依赖的第三方大型类库等，可以通过这个字段进行配置，以提高构建速度；使用 noParse 进行忽略的模块文件中不会解析 import、require 等语法
       ```js
        const config = {
        //...
        module: { 
            noParse: /jquery|lodash/,
            rules:[...]
        }

        };
       ``` 
   - 多进程配置
      > 注意：实际上在小型项目中，开启多进程打包反而会增加时间成本，因为启动进程和进程间通信都会有一定开销。
      - thread-loader
        ```js
            // npm i -D  thread-loader 
            const path = require('path');

            // 路径处理方法
            function resolve(dir){
            return path.join(__dirname, dir);
            }

            const config = {
            //...
            module: { 
                noParse: /jquery|lodash/,
                rules: [
                {
                    test: /\.js$/i,
                    include: resolve('src'),
                    exclude: /node_modules/,
                    use: [
                    {
                        loader: 'thread-loader', // 开启多进程打包
                        options: {
                        worker: 3,
                        }
                    },
                    'babel-loader',
                    ]
                },
                // ...
                ]
            }
            };

        ``` 
      -  happypack
        > 同样为开启多进程打包的工具，webpack5 已弃用。  
   - 利用缓存(利用缓存可以大幅提升重复构建的速度)
     -  babel-loader 开启缓存
        ```v
            babel 在转译 js 过程中时间开销比价大，将 babel-loader 的执行结果缓存起来，重新打包的时候，直接读取缓存
            缓存位置： node_modules/.cache/babel-loader
        ```  

        ```js
        const config = {
        module: { 
            noParse: /jquery|lodash/,
            rules: [
            {
                test: /\.js$/i,
                include: resolve('src'),
                exclude: /node_modules/,
                use: [
                // ...
                {
                    loader: 'babel-loader',
                    options: {
                    cacheDirectory: true // 启用缓存
                    }
                },
                ]
            },
            // ...
            ]
        }
        }
        ```
        那其他的 loader 如何将结果缓存呢？cache-loader 就可以帮我们完成这件事情
     -  cache-loader  
        ` npm i -D cache-loader `  
        ```js
        const config = {
        module: { 
            // ...
            rules: [
            {
                test: /\.(s[ac]|c)ss$/i, //匹配所有的 sass/scss/css 文件
                use: [
                // 'style-loader',
                MiniCssExtractPlugin.loader,
                'cache-loader', // 获取前面 loader 转换的结果
                'css-loader',
                'postcss-loader',
                'sass-loader', 
                ]
            }, 
            // ...
            ]
        }
        }

        ```
     - hard-source-webpack-plugin
      > hard-source-webpack-plugin 为模块提供了中间缓存，重复构建时间大约可以减少 80%，但是在 webpack5 中已经内置了模块缓存，不需要再使用此插件 
     -  cache 持久化缓存
        通过配置 cache 缓存生成的 webpack 模块和 chunk，来改善构建速度。
        ```js
          const config = {
            cache: {
              type: 'filesystem',
            },
          };
        ``` 
     - 
   - 
#### Hash值
| 占位符      | 解释                                                                                   |
| ----------- | -------------------------------------------------------------------------------------- |
| ext         | 文件后缀名                                                                             |
| name        | 文件名                                                                                 |
| path        | 文件相对路径                                                                           |
| folder      | 文件所在文件夹                                                                         |
| hash        | 每次构建生成的唯一 hash 值; 任何一个文件改动，整个项目的构建 hash 值都会改变；         |
| chunkhash   | 根据 chunk 生成 hash 值 ;文件的改动只会影响其所在 chunk 的 hash 值；                   |
| contenthash | 根据文件内容生成hash 值;每个文件都有单独的 hash 值，文件的改动只会影响自身的 hash 值； |


#### Webpack5资源模块
内置了资源处理模块，file-loader 和 url-loader 都可以不用安装
> webpack5 新增资源模块(asset module)，允许使用资源文件（字体，图标等）而无需配置额外的 loader。

资源模块支持以下四个配置：
>- asset/resource 将资源分割为单独的文件，并导出 url，类似之前的 file-loader 的功能.
>- asset/inline 将资源导出为 dataUrl 的形式，类似之前的 url-loader 的小于 limit 参数时功能.
>- asset/source 将资源导出为源码（source code）. 类似的 raw-loader 功能.
>- asset 会根据文件大小来选择使用哪种类型，当文件小于 8 KB（默认） 的时候会使用 asset/inline，否则会使用 asset/resource
```js
// ./src/index.js

const config = {
  // ...
  module: { 
    rules: [
      // ... 
      {
        test: /\.(jpe?g|png|gif)$/i,
        type: 'asset',
        generator: {
          // 输出文件位置以及文件名
          // [ext] 自带 "." 这个与 url-loader 配置不同
          filename: "[name][hash:8][ext]"
        },
        parser: {
          dataUrlCondition: {
            maxSize: 50 * 1024 //超过50kb不转 base64
          }
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/i,
        type: 'asset',
        generator: {
          // 输出文件位置以及文件名
          filename: "[name][hash:8][ext]"
        },
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024 // 超过100kb不转 base64
          }
        }
      },
    ]
  },
  // ...
}

module.exports = (env, argv) => {
  console.log('argv.mode=',argv.mode) // 打印 mode(模式) 值
  // 这里可以通过不同的模式修改 config 配置
  return config;
}

```




### interview Question
- Webpack 配置中用过哪些 Loader ？都有什么作用？
- Webpack 配置中用过哪些 Plugin ？都有什么作用？
- Loader 和 Plugin 有什么区别？
- 如何编写 Loader ? 介绍一下思路？
- 如何编写 Plugin ? 介绍一下思路？
- Webpack optimize 有配置过吗？可以简单说说吗？
- Webpack 层面如何性能优化？
- Webpack 打包流程是怎样的？
- tree-shaking 实现原理是怎样的？
- Webpack 热更新（HMR）是如何实现？
- Webpack 打包中 Babel 插件是如何工作的？
- Webpack 和 Rollup 有什么相同点与不同点？
- Webpack5 更新了哪些新特性？
- webpack 的构建流程是什么
- webpack 打包时hash码是如何生成的
- webpack 离线缓存静态资源如何实现


### 参见
- [作者：ITEM](https://juejin.cn/post/7023242274876162084)