# react-url-loader

[![NPM version](https://img.shields.io/npm/v/react-url-loader.svg?style=flat)](https://npmjs.com/package/react-url-loader)
[![NPM downloads](https://img.shields.io/npm/dm/react-url-loader.svg?style=flat)](https://npmjs.com/package/react-url-loader)

This is a `file-loader` wrapper for React and Webpack, solve the image relative path issues when HTML and CSS are not in the same directory.

## install

To begin, you'll need to install `react-url-loader`:

```bash
npm install react-url-loader --save-dev
```

`react-url-loader` works like
[`file-loader`](https://github.com/webpack-contrib/file-loader), but solve a file-loader`s relative path issue when image, css and html in diffrence directorys.

## purpose

if your output directory structure is like this:

```text
index.html
images
  1.png
  2.png
styles
  1.css
  2.css
script
  1.js
  2.js
```

1.css load 1.png, the url in `style` file may be write like this:

```css
  ...some css...
  .some-selector {
    background: url(images/1.png);
  }
  ...some css...
```

to support that output directory structure, your webpack config maybe like this:

```js
{
  test: /\.(png|jpe?g|gif|svg)(\?\S*)?$/,
  use: [{ loader: 'file-loader', options: { publicPath: '../', name: 'images/[name].[ext]?[hash]' }]
}
```

then webpack will translate `images/1.png` to `../images/1.png`, this does solve the issues, but wait, if your `react component render function` has img tag that:

```jsx
<div>
  ...some html...
  <img src="image/2.png">
  ...some html...
</div>
```

webpack will also translate `images/2.png` to `../images/2.png`, it leads html load `2.png` failure, so file-loader can't tell the picture from HTML or CSS.
`react-url-loader` just solving this issues!
your webpack config can be like this:

```js
{
  test: /\.(png|jpe?g|gif|svg)(\?\S*)?$/,
  use: [{ loader: 'react-url-loader', options: { publicPath: '', publicStylePath: '../', name: 'images/[name].[ext]?[hash]' }]
}
```

or

```js
{
  test: /\.(png|jpe?g|gif|svg)(\?\S*)?$/,
  use: [{
    loader: 'url-loader',
    options: {
      limit: 2048,
      publicPath: '',
      publicStylePath: '../',
      fallback: 'react-url-loader',
      name: 'images/[name].[ext]?[hash]'
    }
  ]
}
```

`react-url-loader` will choose `publicPath` or `publicStylePath` based on the image from `react component render function` or `style file`.

you can also use `outputPath` or `outputStylePath`:

```js
{
  test: /\.(png|jpe?g|gif|svg)(\?\S*)?$/,
  use: [
    { 
      loader: 'react-url-loader', 
      options: { 
        outputPath: '', 
        outputStylePath: '../', 
        name: 'images/[name].[ext]?[hash]' 
      }
  ]
}
```

or

```js
{
  test: /\.(png|jpe?g|gif|svg)(\?\S*)?$/,
  use: [{
    loader: 'url-loader',
    options: {
      limit: 2048,
      outputPath: '',
      outputStylePath: '../',
      fallback: 'vue-asset-loader',
      name: 'images/[name].[ext]?[hash]'
    }
  ]
}
```

## License

[MIT](./LICENSE)
