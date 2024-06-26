# @livingdocs/fastify-webpack-hmr

> :warning: This project is meant to be used in development environment only.  

This is a fork of https://github.com/lependu/fastify-webpack-hmr/ with support for [Webpack v5](https://github.com/lependu/fastify-webpack-hmr/pull/100).
Under the hood it sets up [webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware) and [webpack-hot-middleware](https://github.com/webpack-contrib/webpack-hot-middleware).

## Install
```
$ npm i --save-dev @livingdocs/fastify-webpack-hmr
```

## Usage

For a more detailed exampe please check out the `example` directory.  
The plugin accepts a configuration object, with a following properties:

### compiler
`{object}` `optional`

If you pass a custom `webpack compiler` instance to the plugin, it will pass that to the middlewares.  
*Note:* if you pass a `compiler` option the plugin omits the `config` option.
```js
const fastify = require('fastify')()
const HMR = require('@livingdocs/fastify-webpack-hmr')
const webpack = require('webpack')
const webpackConfig = require('path/to/your/webpack/config')

const compiler = webpack(webpackConfig)

fastify.register(HMR, { compiler })

fastify.listen(3000)
```

### config
`{string|object}` `optional`

If you pass this option instead of a `compiler`, the plugin tries to set up the webpack compiler and will pass that compiler instance to the middlewares. For the detailed configuration options please check the [`webpack documentation`](https://webpack.js.org/configuration/).  

If config is a `string` it has to be a path to a valid webpack configuration file.
```js
const fastify = require('fastify')()
const HMR = require('@livingdocs/fastify-webpack-hmr')
const { join } = require('path')

const config = join(__dirname, 'path.to.your.webpack.config')

fastify.register(HMR, { config })

fastify.listen(3000)
  ```
  
Or you can directly pass a valid webpack configuration `object`.
```js
const fastify = require('fastify')()
const HMR = require('@livingdocs/fastify-webpack-hmr')
const { join } = require('path')
const hotConf = 'webpack-hot-middleware/client?path=/__webpack_hmr'

const config = {
  mode: 'development', // Prevents webpack warning
  // Add the webpack-hot-middleware to the entry point array.
  entry: [join(__dirname, 'path.to.your.client.file'), hotConf],
  output: {
    publicPath: '/assets',
    filename: 'main.js'
  }
}

fastify.register(HMR, { config })

fastify.listen(3000)
```

### webpackDev
`{object}` `optional`

Additional configuration options which will be passed to [`webpack-dev-middleware`](https://github.com/webpack/webpack-dev-middleware#options).

### webpackHot
`{boolean|object}` `optional`

You can `disable` webpack-hot-middleware if you set this option `false`.
If it is an `object` it will be passed to [`webpack-hot-middleware`](https://github.com/webpack-contrib/webpack-hot-middleware#config).

## Multi compiler mode
In multi compiler mode you must pass the `webpackDev.publicPath` option to the plugin.

> *Tip:* Don't forget to set name parameter when you register `webpack-hot-middleware` in entry array. It makes sure that bundles don't process each other's updates.

```js
const fastify = require('fastify')()
const HMR = require('@livingdocs/fastify-webpack-hmr')
const { join } = require('path')
const hotConf = 'webpack-hot-middleware/client?path=/__webpack_hmr'

const config = [
  {
    name: 'mobile',
    mode: 'development',
    entry: [
      join(__dirname, 'example', 'mobile.js'),
      `${hotConf}&name=mobile`
    ],
    stats: false,
    output: { filename: 'mobile.js', publicPath: '/assets' }
  },
  {
    name: 'desktop',
    mode: 'development',
    entry: [
      join(__dirname, 'example', 'desktop.js'),
      `${hotConf}&name=desktop`
    ],
    stats: false,
    output: { filename: 'desktop.js', publicPath: '/assets' }
  }
]

const webpackDev = { publicPath: '/assets' }

fastify.register(HMR, { config, webpackDev })

fastify.listen(3000)
```

## Reference
This plugin decorates the `fastify` instance with `webpack` object. The object has the following properties:
- `compiler` The [`webpack compiler` instance](https://webpack.js.org/api/node/).
- `dev` The [`webpack-dev-middleware` instance](https://github.com/webpack/webpack-dev-middleware#api).
- `hot` The [`webpack-hot-middleware` instance](https://github.com/webpack-contrib/webpack-hot-middleware).

## License
Licensed under [MIT](./LICENSE).
