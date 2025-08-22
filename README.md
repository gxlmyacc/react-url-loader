# react-url-loader

[![npm version](https://badge.fury.io/js/react-url-loader.svg)](https://badge.fury.io/js/react-url-loader)
[![license](https://img.shields.io/npm/l/react-url-loader.svg)](https://github.com/gxlmyacc/react-url-loader/blob/master/LICENSE)
[![npm monthly downloads](https://img.shields.io/npm/dm/react-url-loader.svg)](https://npmjs.com/package/react-url-loader)

[English](https://github.com/gxlmyacc/react-url-loader/blob/master/README.md) | [中文](https://github.com/gxlmyacc/react-url-loader/blob/master/README_CN.md)

A file-loader for React that solves image relative path issues when HTML and CSS are not in the same directory.

## 🚀 Features

- **Smart Path Resolution**: Automatically handles different output paths for styles and other assets
- **Webpack Compatible**: Works with webpack 3.x, 4.x, and 5.x
- **Flexible Configuration**: Supports custom output paths for different file types
- **Easy Integration**: Drop-in replacement for file-loader with enhanced functionality

## 🎯 Purpose

This loader solves a common problem in webpack-based projects where HTML and CSS files are output to different directories, causing broken image references.

### The Problem

When your output directory structure looks like this:

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

**Scenario 1: CSS referencing images**
```css
/* styles/main.css */
.header {
  background: url(images/logo.png); /* This works fine */
}
```

**Scenario 2: React component referencing images**
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

### The Issue

With standard `file-loader`, you might configure it like this:
```javascript
{
  test: /\.(png|jpe?g|gif|svg)$/i,
  use: [{
    loader: 'file-loader',
    options: {
      publicPath: '../',  // This fixes CSS paths
      name: 'images/[name].[ext]'
    }
  }]
}
```

**Problem**: The `../` prefix that fixes CSS paths (`../images/logo.png`) breaks React component paths, resulting in `../images/hero.jpg` which resolves to the wrong location (JavaScript relative paths are based on HTML location, not the JavaScript file's relative position).

### The Solution

`react-url-loader` automatically detects the context and applies the correct path:

- **In CSS files**: Uses `publicStylePath` (e.g., `../`)
- **In React components**: Uses `publicPath` (e.g., `''`)

```javascript
{
  test: /\.(png|jpe?g|gif|svg)$/i,
  use: [{
    loader: 'react-url-loader',
    options: {
      publicPath: '',           // For React components
      publicStylePath: '../',   // For CSS files
      name: 'images/[name].[ext]'
    }
  }]
}
```

**Result**:
- CSS: `url(../images/logo.png)` ✅ Works
- React: `src="images/hero.jpg"` ✅ Works

## 📦 Installation

```bash
npm install --save-dev react-url-loader
```

or

```bash
yarn add --dev react-url-loader
```

## 🔧 Usage

### Basic Configuration

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

### Advanced Configuration with Style Paths

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
              // Separate paths for style files
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

## ⚙️ Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `outputPath` | `string\|function` | - | Output path for assets |
| `publicPath` | `string\|function` | - | Public path for assets |
| `outputStylePath` | `string\|function` | - | Output path for assets when imported in style files |
| `publicStylePath` | `string\|function` | - | Public path for assets when imported in style files |
| `name` | `string` | `[hash].[ext]` | Name template for output files |

### Additional Options

All other options from [file-loader](https://github.com/webpack-contrib/file-loader) are supported and will be passed through.

## 🔍 How It Works

The loader detects when assets are imported in style files (`.scss`, `.sass`, `.less`, `.styl`, `.stylus`, `.css`) and automatically applies different path configurations:

1. **Style Files**: Uses `outputStylePath` and `publicStylePath` options
2. **Other Files**: Uses standard `outputPath` and `publicPath` options

This solves the common issue where CSS and HTML files are output to different directories, causing broken image references.

## 📁 Example Project Structure

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

## 🎯 Use Cases

- **React Applications**: Perfect for React projects with complex asset structures
- **Component Libraries**: When components have their own assets and styles
- **Multi-page Applications**: When different pages output to different directories
- **Style-heavy Projects**: Projects with extensive use of SCSS/SASS/LESS

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Built on top of [file-loader](https://github.com/webpack-contrib/file-loader)
- Inspired by the need to solve asset path issues in React applications

## 📞 Support

If you encounter any issues or have questions:

- [GitHub Issues](https://github.com/gxlmyacc/react-url-loader/issues)
- [GitHub Repository](https://github.com/gxlmyacc/react-url-loader)

---

Made with ❤️ by [gxlmyacc](https://github.com/gxlmyacc)
