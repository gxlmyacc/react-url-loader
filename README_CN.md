# react-url-loader

[![npm version](https://badge.fury.io/js/react-url-loader.svg)](https://badge.fury.io/js/react-url-loader)
[![license](https://img.shields.io/npm/l/react-url-loader.svg)](https://github.com/gxlmyacc/react-url-loader/blob/master/LICENSE)

[English](https://github.com/gxlmyacc/react-url-loader/blob/master/README.md) | [中文](https://github.com/gxlmyacc/react-url-loader/blob/master/README_CN.md)

一个用于React的文件加载器，解决HTML和CSS不在同一目录时图片相对路径的问题。

## 🚀 特性

- **智能路径解析**: 自动处理样式文件和其他资源的不同输出路径
- **Webpack兼容**: 支持webpack 3.x、4.x和5.x版本
- **灵活配置**: 支持为不同文件类型自定义输出路径
- **易于集成**: 作为file-loader的增强替代品，可直接替换使用

## 🎯 用途

这个加载器解决了基于webpack的项目中一个常见问题：当HTML和CSS文件输出到不同目录时，图片引用路径失效的问题。

### 问题描述

当你的输出目录结构如下时：

```
dist/
├── index.html
├── images/
│   ├── logo.png
│   └── hero.jpg
├── styles/
│   ├── main.css
│   └── components.css
└── scripts/
    ├── app.js
    └── vendor.js
```

**场景1：CSS引用图片**
```css
/* styles/main.css */
.header {
  background: url(images/logo.png); /* 这样写没问题 */
}
```

**场景2：React组件引用图片**
```jsx
// components/Header.jsx
import React from 'react';

function Header() {
  return (
    <div className="header">
      <img src={require('../../images/hero.jpg')} alt="Hero" />
    </div>
  );
}
```

### 问题所在

使用标准的`file-loader`，你可能会这样配置：
```javascript
{
  test: /\.(png|jpe?g|gif|svg)$/i,
  use: [{
    loader: 'file-loader',
    options: {
      publicPath: '../',  // 这修复了CSS路径
      name: 'images/[name].[ext]'
    }
  }]
}
```

**问题**：`../`前缀虽然修复了CSS路径（`../images/logo.png`），但破坏了React组件的路径，导致`../images/hero.jpg`解析到错误的位置（js中的相对路径是基于html所在位置寻址的，而不是js文件的相对位置）。

### 解决方案

`react-url-loader`自动检测上下文并应用正确的路径：

- **在CSS文件中**：使用`publicStylePath`（例如：`../`）
- **在React组件中**：使用`publicPath`（例如：`''`）

```javascript
{
  test: /\.(png|jpe?g|gif|svg)$/i,
  use: [{
    loader: 'react-url-loader',
    options: {
      publicPath: '',           // 用于React组件
      publicStylePath: '../',   // 用于CSS文件
      name: 'images/[name].[ext]'
    }
  }]
}
```

**结果**：
- CSS: `url(../images/logo.png)` ✅ 正常工作
- React: `src="images/hero.jpg"` ✅ 正常工作

## 📦 安装

```bash
npm install --save-dev react-url-loader
```

或者

```bash
yarn add --dev react-url-loader
```

## 🔧 使用方法

### 基础配置

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif|svg)$/i,
        use: [
          {
            loader: 'react-url-loader',
            options: {
              name: '[name].[hash].[ext]',
              outputPath: 'images/',
              publicPath: '/assets/images/'
            }
          }
        ]
      }
    ]
  }
};
```

### 带样式路径的高级配置

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif|svg)$/i,
        use: [
          {
            loader: 'react-url-loader',
            options: {
              name: '[name].[hash].[ext]',
              outputPath: 'images/',
              publicPath: '/assets/images/',
              // 为样式文件设置单独的路径
              outputStylePath: 'styles/images/',
              publicStylePath: '/assets/styles/images/'
            }
          }
        ]
      }
    ]
  }
};
```

## ⚙️ 配置选项

| 选项 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `outputPath` | `string\|function` | - | 资源的输出路径 |
| `publicPath` | `string\|function` | - | 资源的公共路径 |
| `outputStylePath` | `string\|function` | - | 在样式文件中导入时资源的输出路径 |
| `publicStylePath` | `string\|function` | - | 在样式文件中导入时资源的公共路径 |
| `name` | `string` | `[hash].[ext]` | 输出文件的命名模板 |

### 其他选项

支持[file-loader](https://github.com/webpack-contrib/file-loader)的所有其他选项，这些选项会被直接传递。

## 🔍 工作原理

该加载器能够检测资源是在样式文件（`.scss`、`.sass`、`.less`、`.styl`、`.stylus`、`.css`）中导入的，并自动应用不同的路径配置：

1. **样式文件**: 使用 `outputStylePath` 和 `publicStylePath` 选项
2. **其他文件**: 使用标准的 `outputPath` 和 `publicPath` 选项

这解决了CSS和HTML文件输出到不同目录时图片引用失效的常见问题。

## 📁 项目结构示例

```
src/
├── components/
│   ├── Header/
│   │   ├── Header.jsx
│   │   ├── Header.scss
│   │   └── logo.png
│   └── Footer/
│       ├── Footer.jsx
│       ├── Footer.scss
│       └── footer-bg.jpg
├── styles/
│   ├── main.scss
│   └── variables.scss
└── assets/
    └── images/
        └── hero.jpg
```

## 🎯 使用场景

- **React应用**: 特别适合具有复杂资源结构的React项目
- **组件库**: 当组件拥有自己的资源和样式时
- **多页面应用**: 当不同页面输出到不同目录时
- **样式密集型项目**: 大量使用SCSS/SASS/LESS的项目

## 🤝 贡献

欢迎贡献代码！请随时提交Pull Request。

1. Fork 该仓库
2. 创建你的特性分支 (`git checkout -b feature/amazing-feature`)
3. 提交你的更改 (`git commit -m 'Add some amazing feature'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 打开一个Pull Request

## 📄 许可证

本项目采用MIT许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 🙏 致谢

- 基于 [file-loader](https://github.com/webpack-contrib/file-loader) 构建
- 灵感来源于解决React应用中资源路径问题的需求

## 📞 支持

如果你遇到任何问题或有疑问：

- [GitHub Issues](https://github.com/gxlmyacc/react-url-loader/issues)
- [GitHub 仓库](https://github.com/gxlmyacc/react-url-loader)

---

由 [gxlmyacc](https://github.com/gxlmyacc) 用 ❤️ 制作
