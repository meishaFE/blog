---
title: webpack系列之处理css
date: 2019-06-02 22:03:43
tags:
  - Webpack
---

## 基本使用

项目中通常是在 js 中引入 css 的，需要配置相应的 loader 才能处理响应的样式文件，并且提取 css 成单独的文件

<!-- more -->

- style-loader 在页面中创建 style 标签并且将 css 创建到 html 中
- css-loader 让 js 可以 import css
- 需要注意 loader 的先后顺序

```javascript
module: {
  rules: [
    {
      test: /\.css$/,
      use: [
        {
          loader: 'style-loader'
        },
        {
          loader: 'css-loader'
        }
      ]
    }
  ];
}
```

## style-loader

**useable 可手动将 css 插入到页面中**

```javascript
// webpack 配置
loader: 'style-loader/useable'

// 使用
import style from './css/styles.css';

style.use(); // 会将css动态插入到页面中
style.unuse(); // 会将这个css从页面删除
```

**options：**

- `insertAt` 插入位置(header,body)
- `insertInto` 插入到某个 dom
- `singleton` 是否只使用一个 style 标签

```javascript
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          {
            loader: 'style-loader',
            // 配置
            options: {
              insertAt: 'header',
              insertInto: '#app',
              singleton: true
            }
          },
          {
            loader: 'css-loader'
          }
        ]
      }
    ]
  }
};
```

## css-loader

**options：**

- `alias` 解析的别名
- `importLoader` css-loader 后面是否还有其他的 loader
- `minimize` 是否压缩
- `modules` 是否启用 css 模块化

## less-loader

加一个 rule 匹配 less 文件，注意：使用 less-loader 需要同时安装 less 这个包

```javascript
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.less$/,
        use: [
          {
            loader: 'style-loader'
          },
          {
            loader: 'css-loader'
          },
          {
            loader: 'less-loader'
          }
        ]
      }
    ]
  }
};
```

## 提取 css 到单独的文件

使用 plugin `extract-text-webpack-plugin`

配置：

```javascript
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.less$/,
        use: ExtractTextPlugin.extract({
          // fallback,不提取时的操作
          fallback: {loader: 'style-loader'},

          // 处理css的loader
          use: [
            {loader: 'css-loader'},
            {loader: 'less-loader'}
          ]
        })
      }
    ]
  },

  plugins: [
    new ExtractTextPlugin({
      filename: 'style.min.css'
    })
  ]
};
```
