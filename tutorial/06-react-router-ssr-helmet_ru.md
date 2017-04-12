# 06 - React Router, Server-Side Rendering, and Helmet

Код для этой главы доступен [здесь](https://github.com/verekia/js-stack-walkthrough/tree/master/06-react-router-ssr-helmet).

В этой части мы собираемся создать разные страницы для нашего приложения и возможность перемещаться между ними.

## React Router

> 💡 **[React Router](https://reacttraining.com/react-router/)** это библиотека для навигации между страницами в вашем React приложении, его можно использовать как на клиенте так и на сервере.

React Router получил большое обновление в версии 4, которое все еще находится в бете. Поскольку я хочу, чтобы этот учебник соотвестветствовал требованиям завтрашнего дня, мы будем использовать v4.

- Запустите `yarn add react-router@next react-router-dom@next`

На клиетнской стороне, нам вначале необходимо обернуть наше приложение внутрь `BrowserRouter` компонента.

- Обновите ваш `src/client/index.jsx` согласно следующему:

```js
// [...]
import { BrowserRouter } from 'react-router-dom'
// [...]
const wrapApp = (AppComponent, reduxStore) =>
  <Provider store={reduxStore}>
    <BrowserRouter>
      <AppContainer>
        <AppComponent />
      </AppContainer>
    </BrowserRouter>
  </Provider>
```

## Pages

В нашем приложении будет 4 страницы:

- Домашняя страница.
- Hello page показывающая кнопку и сообщение для синхронного action (действия).
- Hello Async page показывающая кнопку и сообщение для асинхронного action (действия).
- 404 "Не найдена" страница.

- Создайте `src/client/component/page/home.jsx` файл содержащий:

```js
// @flow

import React from 'react'

const HomePage = () => <p>Home</p>

export default HomePage
```

- Создайте `src/client/component/page/hello.jsx` файл содержащий:

```js
// @flow

import React from 'react'

import HelloButton from '../../container/hello-button'
import Message from '../../container/message'

const HelloPage = () =>
  <div>
    <Message />
    <HelloButton />
  </div>

export default HelloPage

```

- Создайте `src/client/component/page/hello-async.jsx` файл содержащий:

```js
// @flow

import React from 'react'

import HelloAsyncButton from '../../container/hello-async-button'
import MessageAsync from '../../container/message-async'

const HelloAsyncPage = () =>
  <div>
    <MessageAsync />
    <HelloAsyncButton />
  </div>

export default HelloAsyncPage
```

- Создайте `src/client/component/page/not-found.jsx` файл содержащий:

```js
// @flow

import React from 'react'

const NotFoundPage = () => <p>Page not found</p>

export default NotFoundPage
```

## Навигация

Давайте добавим некоторые маршруты (routes) в общий конфиг файл.

- Отредактируйте ваш `src/shared/routes.js` согласно следующему:

```js
// @flow

export const HOME_PAGE_ROUTE = '/'
export const HELLO_PAGE_ROUTE = '/hello'
export const HELLO_ASYNC_PAGE_ROUTE = '/hello-async'
export const NOT_FOUND_DEMO_PAGE_ROUTE = '/404'

export const helloEndpointRoute = (num: ?number) => `/ajax/hello/${num || ':num'}`
```

`/404` route (маршрут) просто используем в навигационной ссылке ради демострации того, что случилось, когда вы кликнули на нерабочую ссылку.

- Создайте `src/client/component/nav.jsx` файл содержащий:

```js
// @flow

import React from 'react'
import { NavLink } from 'react-router-dom'
import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
  NOT_FOUND_DEMO_PAGE_ROUTE,
} from '../../shared/routes'

const Nav = () =>
  <nav>
    <ul>
      {[
        { route: HOME_PAGE_ROUTE, label: 'Home' },
        { route: HELLO_PAGE_ROUTE, label: 'Say Hello' },
        { route: HELLO_ASYNC_PAGE_ROUTE, label: 'Say Hello Asynchronously' },
        { route: NOT_FOUND_DEMO_PAGE_ROUTE, label: '404 Demo' },
      ].map(link => (
        <li key={link.route}>
          <NavLink to={link.route} activeStyle={{ color: 'limegreen' }} exact>{link.label}</NavLink>
        </li>
      ))}
    </ul>
  </nav>

export default Nav
```

Сдесь мы просто создаем кучу `NavLink`ов, чтобы использовать в ранее объявленных марштутах.

- Итого, отредактируем `src/client/app.jsx` согласно следующему:

```js
// @flow

import React from 'react'
import { Switch } from 'react-router'
import { Route } from 'react-router-dom'
import { APP_NAME } from '../shared/config'
import Nav from './component/nav'
import HomePage from './component/page/home'
import HelloPage from './component/page/hello'
import HelloAsyncPage from './component/page/hello-async'
import NotFoundPage from './component/page/not-found'
import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
} from '../shared/routes'

const App = () =>
  <div>
    <h1>{APP_NAME}</h1>
    <Nav />
    <Switch>
      <Route exact path={HOME_PAGE_ROUTE} render={() => <HomePage />} />
      <Route path={HELLO_PAGE_ROUTE} render={() => <HelloPage />} />
      <Route path={HELLO_ASYNC_PAGE_ROUTE} render={() => <HelloAsyncPage />} />
      <Route component={NotFoundPage} />
    </Switch>
  </div>

export default App
```

🏁 Запустите `yarn start` и `yarn dev:wds`. Откройте `http://localhost:8000`, и кликните на ссылку навигации между нашими разнымы страницами. Вы должны видеть как динамически изменяется URL. Перейдите между различными страницами и используйте кнопку назад в вашем браузере, чтобы увидеть, что в истории браузера все работает как ожидалось.

Сейчас, допустим вы переходите на страницу `http://localhost:8000/hello`. Нажали кнопку обновить. Вы сейчас получите 404, потому что наш экспресс сервер отвечает только по урлу `/`. Как же вы перемещались по страницам? На самом деле вы это происходило на стороне клиента. Давайте добавим серверный рендеринг в наш проект, чтобы получить ожидаемое поведение.

## Рендеринг на стороне сервера

> 💡 **Рендеринг на стороне сервера** средство для рендеринга вашего приложения на начальном этапе загрузки страницы, не полагаясь на JavaScript для рендера на клиентской стороне.

Рендеринг на стороне сервера необходим для SEO и обеспечивает лучший пользовательский интерфейс, показывающий приложение сразу. 

Первое, что мы собираемся сделать здесь, это переписать большинство нашего клиентского кода к общей / изоморфной / универсальной части нашего кода, так как теперь наше React приложение также будет рендерится на сервере.

### Большая миграция к `общему`

- Переместите все файлы из `client` в `shared`, за исключением `src/client/index.jsx`.

Мы должны настроить целую кучу импортов:

- В `src/client/index.jsx`, замените 3 импорта с `'./app'` на `'../shared/app'`, и `'./reducer/hello'` на `'../shared/reducer/hello'`

- В `src/shared/app.jsx`, замените `'../shared/routes'` на `'./routes'` и `'../shared/config'` на `'./config'`

- В `src/shared/component/nav.jsx`, замените `'../../shared/routes'` на `'../routes'`

### Серверные изменения

- Создайте `src/server/routing.js` файл содержащий:

```js
// @flow

import {
  homePage,
  helloPage,
  helloAsyncPage,
  helloEndpoint,
} from './controller'

import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
  helloEndpointRoute,
} from '../shared/routes'

import renderApp from './render-app'

export default (app: Object) => {
  app.get(HOME_PAGE_ROUTE, (req, res) => {
    res.send(renderApp(req.url, homePage()))
  })

  app.get(HELLO_PAGE_ROUTE, (req, res) => {
    res.send(renderApp(req.url, helloPage()))
  })

  app.get(HELLO_ASYNC_PAGE_ROUTE, (req, res) => {
    res.send(renderApp(req.url, helloAsyncPage()))
  })

  app.get(helloEndpointRoute(), (req, res) => {
    res.json(helloEndpoint(req.params.num))
  })

  app.get('/500', () => {
    throw Error('Fake Internal Server Error')
  })

  app.get('*', (req, res) => {
    res.status(404).send(renderApp(req.url))
  })

  // eslint-disable-next-line no-unused-vars
  app.use((err, req, res, next) => {
    // eslint-disable-next-line no-console
    console.error(err.stack)
    res.status(500).send('Something went wrong!')
  })
}
```

В этом файле где мы имеем дело с запросами и ответами. Вызов бизнес-логики происходит через различные внешние `контроллер` модули.

**Заметка**: Вы можете найти много примеров React Router примеров использующих `*` как маршрут на сервере, полагаясь на обработку маршрутизации React Router. Поскольку все запросы проходят через одну и ту же функцию, это делает его неудобным реализацию страниц в MVC-стиле. Вместо этого, мы здесь явно прописываем маршруты и их четкие ответы, чтобы иметь возможность выборки данных из базы данных и с легкостью вставить их в страницу.

- Создайте `src/server/controller.js` файл содержащий:

```js
// @flow

export const homePage = () => null

export const helloPage = () => ({
  hello: { message: 'Server-side preloaded message' },
})

export const helloAsyncPage = () => ({
  hello: { messageAsync: 'Server-side preloaded message for async page' },
})

export const helloEndpoint = (num: number) => ({
  serverMessage: `Hello from the server! (received ${num})`,
})
```

Здесь в нашем контроллере. Тут обычно выполняется бизнес логика и запросы к базе данных, но в нашем случае мы просто захардкодаем некоторые результаты. Эти результаты возвращаются назад в модуль `маршрутизации` для использования при инициализации нашего Redux store (хранилища) на стороне сервера.

- Создайте `src/server/init-store.js` файл содержащий:

```js
// @flow

import Immutable from 'immutable'
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunkMiddleware from 'redux-thunk'

import helloReducer from '../shared/reducer/hello'

const initStore = (plainPartialState: ?Object) => {
  const preloadedState = plainPartialState ? {} : undefined

  if (plainPartialState && plainPartialState.hello) {
    // flow-disable-next-line
    preloadedState.hello = helloReducer(undefined, {})
      .merge(Immutable.fromJS(plainPartialState.hello))
  }

  return createStore(combineReducers({ hello: helloReducer }),
    preloadedState, applyMiddleware(thunkMiddleware))
}

export default initStore
```

Единственная вещь, которую мы здесь делаем, кроме вызова `createStore (создать хранилище)` и применения middleware(промежуточной функции), это объедининие обычного JS объекта, который мы получаем из `контроллера` в стандартное Redux state (хранилище) содержащее Immutable (неизменяемые) объекты.

- Отредактируйте `src/server/index.js` согласно следующему:

```js
// @flow

import compression from 'compression'
import express from 'express'

import routing from './routing'
import { WEB_PORT, STATIC_PATH } from '../shared/config'
import { isProd } from '../shared/util'

const app = express()

app.use(compression())
app.use(STATIC_PATH, express.static('dist'))
app.use(STATIC_PATH, express.static('public'))

routing(app)

app.listen(WEB_PORT, () => {
  // eslint-disable-next-line no-console
  console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
    '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
})
```

Ничего особенного здесь, мы просто вызываем `routing(app)` вместо реализации маршрутизации в этом файле.

- Переименуйте `src/server/render-app.js` в `src/server/render-app.jsx` и отредактируйте согласно следующему:

```js
// @flow

import React from 'react'
import ReactDOMServer from 'react-dom/server'
import { Provider } from 'react-redux'
import { StaticRouter } from 'react-router'

import initStore from './init-store'
import App from './../shared/app'
import { APP_CONTAINER_CLASS, STATIC_PATH, WDS_PORT } from '../shared/config'
import { isProd } from '../shared/util'

const renderApp = (location: string, plainPartialState: ?Object, routerContext: ?Object = {}) => {
  const store = initStore(plainPartialState)
  const appHtml = ReactDOMServer.renderToString(
    <Provider store={store}>
      <StaticRouter location={location} context={routerContext}>
        <App />
      </StaticRouter>
    </Provider>)

  return (
    `<!doctype html>
    <html>
      <head>
        <title>FIX ME</title>
        <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
      </head>
      <body>
        <div class="${APP_CONTAINER_CLASS}">${appHtml}</div>
        <script>
          window.__PRELOADED_STATE__ = ${JSON.stringify(store.getState())}
        </script>
        <script src="${isProd ? STATIC_PATH : `http://localhost:${WDS_PORT}/dist`}/js/bundle.js"></script>
      </body>
    </html>`
  )
}

export default renderApp
```

`ReactDOMServer.renderToString` здесь происходит магия. React будет сравнивать наше содержимое `shared` `App`, и возвращать простую строку HTML элементов. `Provider` работает аналогично клиентскому, но на сервере, мы оборачиваем наше приложение внутрь `StaticRouter` вместо `BrowserRouter`. Для перехода the Redux store (хранилища) с сервера на клиент, мы передаем `window.__PRELOADED_STATE__` которая является произвольным именем переменной.

**Заметка**: Неизменяемые объекты реализуют `toJSON()` метод, аналогичный, который вы могли использовать `JSON.stringify` преобразует код к простой JSON строке.

- Отредактируйте `src/client/index.jsx` для использования предзагрузочного состояния:

```js
import Immutable from 'immutable'
// [...]

/* eslint-disable no-underscore-dangle */
const composeEnhancers = (isProd ? null : window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__) || compose
const preloadedState = window.__PRELOADED_STATE__
/* eslint-enable no-underscore-dangle */

const store = createStore(combineReducers(
  { hello: helloReducer }),
  { hello: Immutable.fromJS(preloadedState.hello) },
  composeEnhancers(applyMiddleware(thunkMiddleware)))
```

Здесь объединяется наш клиентское store (хранилище) с `preloadedState (состояние до загрузки)`, который мы получили с сервера.

🏁 Вы можете запустить `yarn start` и `yarn dev:wds` и перейти между страницами. Обновите страницу на `/hello`, `/hello-async`, и `/404` (или любой другой URI), сейчас должно работать корректно. Обратите внимание на  `message` и `messageAsync` варьируется, в зависимости от того, перешли вы на эту страницу с клиента или это рендер на стороне сервера.

### React Helmet (Реакт Шлем)

> 💡 **[React Helmet](https://github.com/nfl/react-helmet)**: Библиотека, которая вставляет контент в `head` React приложения, работает на клиенте и на сервере.

Я специально заставил тебя написать `FIX ME` в названии, чтобы подчеркнуть тот факт, что даже если мы делаем обработку на стороне сервера, в данный момент мы не заполняем тег `title` (или какие-либо теги в `head`, которые могуть варьироваться в зависимости от страницы).

- Запустите `yarn add react-helmet`

- Отредактируйте `src/server/render-app.jsx` согласно следующему:

```js
import Helmet from 'react-helmet'
// [...]
const renderApp = (/* [...] */) => {

  const appHtml = ReactDOMServer.renderToString(/* [...] */)
  const head = Helmet.rewind()

  return (
    `<!doctype html>
    <html>
      <head>
        ${head.title}
        ${head.meta}
        <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
      </head>
    [...]
    `
  )
}
```

React Helmet использует [react-side-effect](https://github.com/gaearon/react-side-effect) `rewind` для получения данных для рендеринга нашего приложения, которое будет содержать несоколько `<Helmet />` компонентов. Эти `<Helmet />` компоненты в которые мы установим `title` и другие `head` детали для каждой страницы.

- Отредактируйте `src/shared/app.jsx` согласно следующему:

```js
import Helmet from 'react-helmet'
// [...]
const App = () =>
  <div>
    <Helmet titleTemplate={`%s | ${APP_NAME}`} defaultTitle={APP_NAME} />
    <Nav />
    // [...]
```

- Отредактируйте `src/shared/component/page/home.jsx` согласно следующему:

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import { APP_NAME } from '../../config'

const HomePage = () =>
  <div>
    <Helmet
      meta={[
        { name: 'description', content: 'Hello App is an app to say hello' },
        { property: 'og:title', content: APP_NAME },
      ]}
    />
    <h1>{APP_NAME}</h1>
  </div>

export default HomePage

```

- Отредактируйте `src/shared/component/page/hello.jsx` согласно следующему:

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import HelloButton from '../../container/hello-button'
import Message from '../../container/message'

const title = 'Hello Page'

const HelloPage = () =>
  <div>
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello' },
        { property: 'og:title', content: title },
      ]}
    />
    <h1>{title}</h1>
    <Message />
    <HelloButton />
  </div>

export default HelloPage
```

- Отредактируйте `src/shared/component/page/hello-async.jsx` согласно следующему:

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import HelloAsyncButton from '../../container/hello-async-button'
import MessageAsync from '../../container/message-async'

const title = 'Async Hello Page'

const HelloAsyncPage = () =>
  <div>
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello asynchronously' },
        { property: 'og:title', content: title },
      ]}
    />
    <h1>{title}</h1>
    <MessageAsync />
    <HelloAsyncButton />
  </div>

export default HelloAsyncPage

```

- Отредактируйте `src/shared/component/page/not-found.jsx` согласно следующему:

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

const title = 'Page Not Found'

const NotFoundPage = () =>
  <div>
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello' },
        { property: 'og:title', content: title },
      ]}
    />
    <h1>{title}</h1>
  </div>

export default NotFoundPage
```

Этот компонент `<Helmet>` на самом деле не рендерит ничего, он просто вставляет содержимое в `head` нашего документа и предоставляет те же данные на сервере.

🏁 Запустите `yarn start` и `yarn dev:wds` и понавигируйте по страницам. Заголовок вашей страницы должен изменяться при навигации, и должен оставаться неизменным при обновлении страницы. Посмотрите исходный код страницы, чтобы понять как React Helmet устанавливает `title` и `meta` теги, даже при рендере на сервере.

Следующая секция: [07 - Socket.IO](07-socket-io_ru.md#readme)

Назад [предыдущая секция](05-redux-immutable-fetch_ru.md#readme) или [содержание](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
