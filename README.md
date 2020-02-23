# vue3-webpack
> vue3 的 webpack 实验项目

## 运行

```dash
npm install

npm run dev # http://localhost:8090
```

## 打包

```dash
npm run build
```


## 文件目录
```
├── build
|  ├── webpack.base.js
|  ├── webpack.dev.js
|  └── webpack.prod.js
├── package.json
├── postcss.config.js
├── public
|  └── index.html
├── README.md
├── src
|  ├── App.vue
|  ├── components
|  |  └── MyButton.vue
|  └── main.js
└── yarn.lock
```

## 依赖安装
> 自我习惯使用`yarn`, 可以通过 `npm install -g yarn` 来安装

### 1. 初始化项目
```dash
mkdir vue3Test
cd vue3Test
yarn init -y
```

### 2. 安装依赖

##### a. 安装 vue3
```dash
yarn add vue@next
```

##### b. 安装webpack相关
```dash
yarn add webpack webpack-cli webpack-dev-server webpack-merge --dev
```

##### c. 安装webpack插件
```dash
yarn add html-webpack-plugin clean-webpack-plugin terser-webpack-plugin optimize-css-assets-webpack-plugin --dev
```

 - `html-webpack-plugin` 处理静态文件
 - `clean-webpack-plugin` 打包时自动删除之前打包文件
 - `terser-webpack-plugin` JS压缩插件
 - `optimize-css-assets-webpack-plugin` CSS压缩插件

##### d. 安装webpack加载器
```dash
yarn add file-loader image-webpack-loader style-loader css-loader sass-loader sass fibers vue-loader @vue/compiler-sfc postcss-loader autoprefixer cssnano --dev
```

 - `file-loader`, `image-webpack-loader` 图片文件加载
 - `style-loader`, `css-loader`, `sass-loader`, `sass`, `fibers` CSS加载(此处使用sass预处理)
- `vue-loader`, `@vue/compiler-sfc` Vue加载
- `postcss-loader`, `autoprefixer`, `cssnano` postcss加载


## 配置postcss
在 `postcss.config.js`中配置
`autoprefixer`用于给css加前缀,  `cssnano`用于压缩优化css

```js
module.exports = {
  plugins: [
    require('autoprefixer'),
    require('cssnano')({
      preset: 'default'
    })
  ]
}
```

## 配置webpack
从目录可以看到, 将webpack文件分为三个, `webpack.base.js`用于通用配置, `webpack.dev.js`用于配置开发模式, `webpack.prod.js`用于配置生产模式, 开发模式和生产模式的配置文件通过之前安装的`webpack-merge`来将通用配置合并进来

### 1.通用配置
> webpack.base.js

通用配置, 主要是对基本的图片文件和CSS的加载配置

```js
const webpack = require('webpack')
const path = require('path')
const sass = require('sass')
const Fiber = require('fibers')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { VueLoaderPlugin } = require('vue-loader')

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist')
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'
    }),
    new VueLoaderPlugin(),
    new webpack.HashedModuleIdsPlugin()
  ],
  module: {
    rules: [
      {
        test: /\.vue$/,
        use: 'vue-loader'
      }, {
        test: /\.(png|svg|jpg|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              limit: 5000,
              name: 'imgs/[hash].[ext]'
            }
          }
        ]
      }, {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }, {
        test: /\.(scss|sass)$/,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'sass-loader',
            options: {
              implementation: sass,
              sassOptions: {
                fiber: Fiber,
              }
            }
          },
          'postcss-loader'
        ]
      }
    ]
  }
}
```

### 2. 开发模式配置
> webpack.dev.js

开发模式主要是对`webpack-dev-server`进行配置, 开发服务的端口, 代理配置等

```js
const path = require('path')
const merge = require('webpack-merge')
const common = require('./webpack.base.js')

module.exports = merge(common, {
  devtool: 'inline-source-map',
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
    compress: true,
    port: 8090,
    hot: true
  },
  output: {
    filename: 'js/[name].[hash].js',
    path: path.resolve(__dirname, '../dist')
  },
  module: {},
  mode: 'development'
})
```

### 3. 生产模式
> webpack.prod.js

生产模式, 主要是对JS, CSS进行压缩, 对文件进行分片

```js
const path = require('path')
const merge = require('webpack-merge')
const common = require('./webpack.base')
const {
  CleanWebpackPlugin
} = require('clean-webpack-plugin')
const TerserPlugin = require('terser-webpack-plugin')
const OptimizeCssAssertsPlugin = require('optimize-css-assets-webpack-plugin')

module.exports = merge(common, {
  mode: 'production',
  output: {
    filename: 'js/[name].[contenthash].js',
    path: path.resolve(__dirname, '../dist')
  },
  plugins: [
    new CleanWebpackPlugin()
  ],
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          name: 'vendor',
          test: /[\\/]node_modules[\\/]/,
          priority: 10,
          chunks: 'initial'
        }
      }
    },
    minimizer: [
      new OptimizeCssAssertsPlugin({}),
      new TerserPlugin({
        test: /\.js(\?.*)?$/i,
        cache: true,
        parallel: true,
        sourceMap: false,
        terserOptions: {
          wranings: false,
          output: {
            comments: false
          }
        }
      })
    ],
  },
  module: {
    rules: [{
      test: /\.(gif|png|jpe?g|svg)$/i,
      use: [
        'file-loader',
        {
          loader: 'image-webpack-loader',
          options: {
            mozjpeg: {
              progressive: true,
              quality: 65
            },
            // optipng.enabled: false will disable optipng
            optipng: {
              enabled: false,
            },
            pngquant: {
              quality: '65-90',
              speed: 4
            },
            gifsicle: {
              interlaced: false,
            },
            // the webp option will enable WEBP
            webp: {
              quality: 75
            }
          }
        },
      ],
    }]
  }
})
```

## 代码测试
[https://klren0312.github.io/vue3-webpack/](https://klren0312.github.io/vue3-webpack/)

App.vue
```html
<template>
  <div class="container">
    hello {{state.message}}
    <div>数字: {{count}}</div>
    <div>两倍数字: {{doubleCount}}</div>
    <my-button name="增加" @get="increment"></my-button>
  </div>
</template>

<script>
import { computed, ref, reactive } from 'vue'
import MyButton from './components/MyButton.vue'
export default {
  components: {
    MyButton
  },
  setup () {
    const count = ref(0)

    const state = reactive({
      message: 'Vue3'
    })

    const doubleCount = computed(() => count.value * 2)

    const increment = e => {
      console.log(e)
      count.value += 10
      state.message = 'Vue3. Good!'
    }

    return {
      count,
      state,
      doubleCount,
      increment
    }
  }
}
</script>
<style lang="scss">
body {
  font-size: 16px;
}
.container {
  text-align: center;
  line-height: 2;
}
</style>
```

## 参考资料
[https://github.com/vuejs/vue-next](https://github.com/vuejs/vue-next)
[https://webpack.js.org/concepts/](https://webpack.js.org/concepts/)


