# 04 - Webpack, React и Hot Module Replacement

Код для этой главы доступен [здесь](https://github.com/verekia/js-stack-walkthrough/tree/master/04-webpack-react-hmr).

## Webpack

> 💡 **[Webpack](https://webpack.js.org/)** - *сборщик модулей*. Он берет все возможные исходные файлы, обрабатывает их и собирает в один (обычно) JavaScript файл, называемый сборкой, и это будет единственный файл исполняемый на клиенте.

Давайте создадим какой-нибудь простой *hello world* и соберем его с помощью Webpack.

- В `src/shared/config.js` добавьте следующие константы:

```js
export const WDS_PORT = 7000

export const APP_CONTAINER_CLASS = 'js-app'
export const APP_CONTAINER_SELECTOR = `.${APP_CONTAINER_CLASS}`
```

- Создайте файл `src/client/index.js`, содержащий:

```js
import 'babel-polyfill'

import { APP_CONTAINER_SELECTOR } from '../shared/config'

document.querySelector(APP_CONTAINER_SELECTOR).innerHTML = '<h1>Hello Webpack!</h1>'
```

Если вы хотите использовать новейшие возможности ES в клиентском коде, такие как `Promise`, то вам нужно включить [Babel Polyfill](https://babeljs.io/docs/usage/polyfill/) до какого-либо другого кода в сборке.

- Запустите `yarn add babel-polyfill`

Если вы запустите ESLint на этом файле, он будет жаловаться, что `document` undefined.

- Добавьте раздел `env` в `.eslintrc.json`, чтобы позволить использование `window` и `document`:

```json
"env": {
  "browser": true,
  "jest": true
}
```

Хорошо, теперь нам нужно собрать это клиентское ES6 приложение в ES5 сборку.

- Создайте файл `webpack.config.babel.js` содержащий:

```js
// @flow

import path from 'path'

import { WDS_PORT } from './src/shared/config'
import { isProd } from './src/shared/util'

export default {
  entry: [
    './src/client',
  ],
  output: {
    filename: 'js/bundle.js',
    path: path.resolve(__dirname, 'dist'),
    publicPath: isProd ? '/static/' : `http://localhost:${WDS_PORT}/dist/`,
  },
  module: {
    rules: [
      { test: /\.(js|jsx)$/, use: 'babel-loader', exclude: /node_modules/ },
    ],
  },
  devtool: isProd ? false : 'source-map',
  resolve: {
    extensions: ['.js', '.jsx'],
  },
  devServer: {
    port: WDS_PORT,
  },
}
```

Этот файл используется для описания того, как должна быть устроена наша сборка: `entry` - стартовая точка нашего приложения, `output.filename` - имя генерируемой сборки, `output.path` и `output.publicPath` описывают путь до папки со сборкой и URL. Мы поместим сборку в папку  `dist`, которая будет содержать автоматически генерируемые вещи (в отличие от обитающих в `public` декларативных CSS, которые мы создавали до этого). В `module.rules` мы сообщаем Webpack к каким типам файлов применять какие обработчики. Здесь мы говорим, что хотим пропускать все `.js` и `.jsx` (для реакта) файлы через нечто, называемое `babel-loader`, за исключением того, что находится в `node_modules`. Мы также хотим *разрешать* (`resolve`) эти два расширения при `import` модулей (т.е. эти расширения можно будет опускать при импорте - прим. пер.)

**Примечание**: Расширение `.babel.js` сообщает Webpack применять трансформации Babel к данному конфигурационному файлу. 

`babel-loader` - это плагин для Webpack, транспилирующий код, так же как мы это делали с начала этого руководства. Единственное на данный момент отличие, что этот код исполняется в браузере а не на сервере.

- Запустите `yarn add --dev webpack webpack-dev-server babel-core babel-loader`

`babel-core` is a peer-dependency of `babel-loader`, so we installed it as well.
Мы установили также `babel-core`, поскольку это peer-dependency (требуемая зависимость) для `babel-loader`.

- Добавьте `/dist/` в `.gitignore`

### Обновление задач

В режиме разработки мы будем использовать `webpack-dev-server` чтобы пользоваться преимуществами Hot Module Reloading (позже в этой главе), а в продакшене мы просто используем `webpack`, чтобы сгенерировать сборку. В обоих случаях, флаг `--progress` будет полезен для вывода дополнительной информации когда Webpack компилирует файлы. В продакшене мы также передаем в `webpack` флаг `-p` для минификации кода и переменную `NODE_ENV` установленную в `production`.

Давайте обновим наши `scripts` чтобы реализовать это, а также улучшим некоторые другие задачи:

```json
"scripts": {
  "start": "yarn dev:start",
  "dev:start": "nodemon -e js,jsx --ignore lib --ignore dist --exec babel-node src/server",
  "dev:wds": "webpack-dev-server --progress",
  "prod:build": "rimraf lib dist && babel src -d lib --ignore .test.js && cross-env NODE_ENV=production webpack -p --progress",
  "prod:start": "cross-env NODE_ENV=production pm2 start lib/server && pm2 logs",
  "prod:stop": "pm2 delete server",
  "lint": "eslint src webpack.config.babel.js --ext .js,.jsx",
  "test": "yarn lint && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test && yarn prod:build"
},
```

В `dev:start` мы явно указываем расширения для наблюдения: `.js` и `.jsx`, и добавляем `dist` в игнорируемые директории.

Мы создали отдельную задачу `lint` и добавили `webpack.config.babel.js` в список проверяемых файлов.

- Затем давайте создадим контейнер для нашего приложения в `src/server/render-app.js` и включим его в генерируемую сборку:

```js
// @flow

import { APP_CONTAINER_CLASS, STATIC_PATH, WDS_PORT } from '../shared/config'
import { isProd } from '../shared/util'

const renderApp = (title: string) =>
`<!doctype html>
<html>
  <head>
    <title>${title}</title>
    <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
  </head>
  <body>
    <div class="${APP_CONTAINER_CLASS}"></div>
    <script src="${isProd ? STATIC_PATH : `http://localhost:${WDS_PORT}/dist`}/js/bundle.js"></script>
  </body>
</html>
`

export default renderApp
```

Depending on the environment we're in, we'll include either the Webpack Dev Server bundle, or the production bundle. Note that the path to Webpack Dev Server's bundle is *virtual*, `dist/js/bundle.js` is not actually read from your hard drive in development mode. It's also necessary to give Webpack Dev Server a different port than your main web port.

- Finally, in `src/server/index.js`, tweak your `console.log` message like so:

```js
console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
  '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
```

That will give other developers a hint about what to do if they try to just run `yarn start` without Webpack Dev Server.

Alright that was a lot of changes, let's see if everything works as expected:

🏁 Run `yarn start` in a terminal. Open an other terminal tab or window, and run `yarn dev:wds` in it. Once Webpack Dev Server is done generating the bundle and its sourcemaps (which should both be ~600kB files) and both processes hang in your terminals, open `http://localhost:8000/` and you should see "Hello Webpack!". Open your Chrome console, and under the Source tab, check which files are included. You should only see `static/css/style.css` under `localhost:8000/`, and have all your ES6 source files under `webpack://./src`. That means sourcemaps are working. In your editor, in `src/client/index.js`, try changing `Hello Webpack!` into any other string. As you save the file, Webpack Dev Server in your terminal should generate a new bundle and the Chrome tab should reload automatically.

- Kill the previous processes in your terminals with Ctrl+C, then run `yarn prod:build`, and then `yarn prod:start`. Open `http://localhost:8000/` and you should still see "Hello Webpack!". In the Source tab of the Chrome console, you should this time find `static/js/bundle.js` under `localhost:8000/`, but no `webpack://` sources. Click on `bundle.js` to make sure it is minified. Run `yarn prod:stop`.

Good job, I know this was quite dense. You deserve a break! The next section is easier.

**Note**: I would recommend to have at least 3 terminals open, one for your Express server, one for the Webpack Dev Server, and one for Git, tests, and general commands like installing packages with `yarn`. Ideally, you should split your terminal screen in multiple panes to see them all.

## React

> 💡 **[React](https://facebook.github.io/react/)** is a library for building user interfaces by Facebook. It uses the **[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html)** syntax to represent HTML elements and components while leveraging the power of JavaScript.

In this section we are going to render some text using React and JSX.

First, let's install React and ReactDOM:

- Run `yarn add react react-dom`

Rename your `src/client/index.js` file into `src/client/index.jsx` and write some React code in it:

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'

import App from './app'
import { APP_CONTAINER_SELECTOR } from '../shared/config'

ReactDOM.render(<App />, document.querySelector(APP_CONTAINER_SELECTOR))
```

- Create a `src/client/app.jsx` file containing:

```js
// @flow

import React from 'react'

const App = () => <h1>Hello React!</h1>

export default App
```

Since we use the JSX syntax here, we have to tell Babel that it needs to transform it with the `babel-preset-react` preset. And while we're at it, we're also going to add a Babel plugin called `flow-react-proptypes` which automatically generates PropTypes from Flow annotations for your React components.

- Run `yarn add --dev babel-preset-react babel-plugin-flow-react-proptypes` and edit your `.babelrc` file like so:

```json
{
  "presets": [
    "env",
    "flow",
    "react"
  ],
  "plugins": [
    "flow-react-proptypes"
  ]
}
```

🏁 Run `yarn start` and `yarn dev:wds` and hit `http://localhost:8000`. You should see "Hello React!".

Now try changing the text in `src/client/app.jsx` to something else. Webpack Dev Server should reload the page automatically, which is pretty neat, but we are going to make it even better.

## Hot Module Replacement

> 💡 **[Hot Module Replacement](https://webpack.js.org/concepts/hot-module-replacement/)** (*HMR*) is a powerful Webpack feature to replace a module on the fly without reloading the entire page.

To make HMR work with React, we are going to need to tweak a few things.

- Run `yarn add react-hot-loader@next`

- Edit your `webpack.config.babel.js` like so:

```js
import webpack from 'webpack'
// [...]
entry: [
  'react-hot-loader/patch',
  './src/client',
],
// [...]
devServer: {
  port: WDS_PORT,
  hot: true,
},
plugins: [
  new webpack.optimize.OccurrenceOrderPlugin(),
  new webpack.HotModuleReplacementPlugin(),
  new webpack.NamedModulesPlugin(),
  new webpack.NoEmitOnErrorsPlugin(),
],
```

- Edit your `src/client/index.jsx` file:

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'
import { AppContainer } from 'react-hot-loader'

import App from './app'
import { APP_CONTAINER_SELECTOR } from '../shared/config'

const rootEl = document.querySelector(APP_CONTAINER_SELECTOR)

const wrapApp = AppComponent =>
  <AppContainer>
    <AppComponent />
  </AppContainer>

ReactDOM.render(wrapApp(App), rootEl)

if (module.hot) {
  // flow-disable-next-line
  module.hot.accept('./app', () => {
    // eslint-disable-next-line global-require
    const NextApp = require('./app').default
    ReactDOM.render(wrapApp(NextApp), rootEl)
  })
}
```

We need to make our `App` a child of `react-hot-loader`'s `AppContainer`, and we need to `require` the next version of our `App` when hot-reloading. To make this  process clean and DRY, we create a little `wrapApp` function that we use in both places it needs to render `App`. Feel free to move the `eslint-disable global-require` to the top of the file to make this more readable.

🏁 Restart your `yarn dev:wds` process if it was still running. Open `localhost:8000`. In the Console tab, you should see some logs about HMR. Go ahead and change something in `src/client/app.jsx` and your changes should be reflected in your browser after a few seconds, without any full-page reload!

Next section: [05 - Redux, Immutable, Fetch](05-redux-immutable-fetch.md#readme)

Back to the [previous section](03-express-nodemon-pm2.md#readme) or the [table of contents](https://github.com/verekia/js-stack-from-scratch#table-of-contents).