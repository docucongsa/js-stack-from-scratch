# 05 - Redux, Immutable, and Fetch

Код для этой части статьи доступен [здесь](https://github.com/verekia/js-stack-walkthrough/tree/master/05-redux-immutable-fetch).

В этой части мы соединим React и Redux в процессе написания простого приложения. Приложение будет содержать сообщение и кнопку. Сообщение будет меняться, когда пользователь кликнет на кнопку.

Перед тем как мы начнем, здесь будет быстрое введение в ImmutableJS, которое не связано с React и Redux, но будет использованно в этой главе.

## ImmutableJS

> 💡 **[ImmutableJS](https://facebook.github.io/immutable-js/)** (или просто Immutable) библиотека разработанная компанией Facebook для манипулирования неизменяемыми коллекциями, например списки и итерируемые объекты. Любое изменение неизменяемого объекта возвращает новый объект без изменения первоначального объекта.

Например, вместо следующих действий:

```js
const obj = { a: 1 }
obj.a = 2 // изменение `obj`
```

Мы должны делать так:

```js
const obj = Immutable.Map({ a: 1 })
obj.set('a', 2) // возвращает новый объект без изменения `obj`
```

Этот подход следует парадигме **функциональное программирование**, который отлично работает с Redux.

Когда мы создаем неизменяемую коллекцию, есть очень удобный метод `Immutable.fromJS()`, который берет обычный JS объект или массив и возвращает неизменияемую версию:

```js
const immutablePerson = Immutable.fromJS({
  name: 'Stan',
  friends: ['Kyle', 'Cartman', 'Kenny'],
})

console.log(immutablePerson)

/*
 *  Map {
 *    "name": "Stan",
 *    "friends": List [ "Kyle", "Cartman", "Kenny" ]
 *  }
 */
```

- Запустите в консоли `yarn add immutable@4.0.0-rc.2`

## Redux

> 💡 **[Redux](http://redux.js.org/)** библиотека для управления состоянием вашего приложения. Она создает *store (хранилище)*, который является единственным источником истины состояния вашего приложения в любой момент времени.

Начнем с простой части, объявим наши Redux actions (действия):

- Запустите в консоли `yarn add redux redux-actions`

- Создайте файл `src/client/action/hello.js` содержащий:

```js
// @flow

import { createAction } from 'redux-actions'

export const SAY_HELLO = 'SAY_HELLO'

export const sayHello = createAction(SAY_HELLO)
```

Этот файл предоставляет нам *action (действие)*, `SAY_HELLO`, это *action creator (создатель действия)*, `sayHello`, это-функция. Мы используем [`redux-actions`](https://github.com/acdlite/redux-actions) для уменьшения шаблонов связанных с Redux actions (действия). `redux-actions` реализуют [Flux Standard Action](https://github.com/acdlite/flux-standard-action) (действие согласно архитектуре флакс) модель, который создает *action creators (создатель действия)* возвращает объект с ключами `type` и `payload`.

- Создадим файл `src/client/reducer/hello.js` содержащий следующее:

```js
// @flow

import Immutable from 'immutable'
import type { fromJS as Immut } from 'immutable'

import { SAY_HELLO } from '../action/hello'

const initialState = Immutable.fromJS({
  message: 'Initial reducer message',
})

const helloReducer = (state: Immut = initialState, action: { type: string, payload: any }) => {
  switch (action.type) {
    case SAY_HELLO:
      return state.set('message', action.payload)
    default:
      return state
  }
}

export default helloReducer
```
В этом файле мы проинициализировали состояние для нашего редьюсера при помощи неизменяемых данных, содержащих одно свойство `message`, со значением `Initial reducer message`. `helloReducer` обрабатывает `SAY_HELLO` экшен просто устанавливая новое `message (сообщение)` при помощи ключа action.payload. Flow проводит деструктуризацию `action` в `type` и `payload`. `payload` может быть `any (любого)` типа. Это сначала пугает, но потом становится довольно понятным. Для типизации `state`, мы используем `import type (импорт типа)` Flow инструкция для получения типа `fromJS`. Мы переименовали его в `Immut` для ясности, потому что `state: fromJS` выглядит довольно запутанным. `import type` линия будет удалена из исполняемых файлов, как и любая другая Flow ноттация. Обратите внимание на `Immutable.fromJS()` и `set()` которые вы видели ранее.

## React-Redux

> 💡 **[react-redux](https://github.com/reactjs/react-redux)** *connects (соединяет)* Redux store с React компонентами. Благодаря `react-redux`, когда the Redux store изменяется, React компоненты получают автоматические обновления. Они также могут Redux actions (действия).

- Запустите `yarn add react-redux`

В этой секции мы будем создавать *Components (Компоненты)* и *Containers (Контейнеры)*.

**Components (Компоненты)** это *глупые* React компоненты, они ничего не знают о Redux state. **Containers (Контейнеры)** это *умные* которые знаю от состоянии и что мы собираемся *connect (подключить)* к нашим глупым компонентам.

- Создайте файл `src/client/component/button.jsx` содержащий:

```js
// @flow

import React from 'react'

type Props = {
  label: string,
  handleClick: Function,
}

const Button = ({ label, handleClick }: Props) =>
  <button onClick={handleClick}>{label}</button>

export default Button
```

**Примечание**: Здесь вы можете увидеть случай использования Flow *псевдоним типа*. Мы определяем тип `Props`, проводим деструктуризацию `props` и проверяем типы `props` согласно `Props`.--

- Создайте файл `src/client/component/message.jsx` содержащий:

```js
// @flow

import React from 'react'

type Props = {
  message: string,
}

const Message = ({ message }: Props) =>
  <p>{message}</p>

export default Message
```

Здесь пример *глупых* компонентов. У них мало логики, и просто показывают все, что их просят показать через React **props**. Главное отличие между `button.jsx` и `message.jsx` это `Button` содержит ссылку на action dispatcher (диспечтер действий) в этих props, в то же время `Message` просто содержит некоторые данные для отображения.

Повторим, *components (компонент)* ничего не знает о Redux **actions (действия)** или **state (состояние)** в нашем приложении, поэтому мы собираемся создать умный **containers (контейнер)**, который будет предоставлять соответствующих action dispatchers (диспетчеров действий) для этих 2х глупых компонента.

- Создайте файл `src/client/container/hello-button.js` содержащий:

```js
// @flow

import { connect } from 'react-redux'

import { sayHello } from '../action/hello'
import Button from '../component/button'

const mapStateToProps = () => ({
  label: 'Say hello',
})

const mapDispatchToProps = dispatch => ({
  handleClick: () => { dispatch(sayHello('Hello!')) },
})

export default connect(mapStateToProps, mapDispatchToProps)(Button)
```

Этот контейнер подключает `Button` компонент к `sayHello` action (действие) и Redux `dispatch (отправка)` метод.

- Создайте файл `src/client/container/message.js` содержащий:

```js
// @flow

import { connect } from 'react-redux'

import Message from '../component/message'

const mapStateToProps = state => ({
  message: state.hello.get('message'),
})

export default connect(mapStateToProps)(Message)
```

Этот контейнер присоединяет Redux state приложения с `Message` компонент. Когда state (состояние) изменится, `Message` будет автоматически перерендерен с правильными prop (получаемые данные от родительского компонента) `message`. Эти соединения выполняются благодаря фунции `connect` из пакета `react-redux`. 

- Обновите ваш файл `src/client/app.jsx` согласно следующему примеру:

```js
// @flow

import React from 'react'
import HelloButton from './container/hello-button'
import Message from './container/message'
import { APP_NAME } from '../shared/config'

const App = () =>
  <div>
    <h1>{APP_NAME}</h1>
    <Message />
    <HelloButton />
  </div>

export default App
```

Мы все еще не имеем инициализированного Redux store и еще не поместили 2 контейнера в любоое место в нашем приложении:

- Обновите ваш файл `src/client/index.jsx` согласно следующему примеру:

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'
import { AppContainer } from 'react-hot-loader'
import { Provider } from 'react-redux'
import { createStore, combineReducers } from 'redux'

import App from './app'
import helloReducer from './reducer/hello'
import { APP_CONTAINER_SELECTOR } from '../shared/config'
import { isProd } from '../shared/util'

const store = createStore(combineReducers({ hello: helloReducer }),
  // eslint-disable-next-line no-underscore-dangle
  isProd ? undefined : window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__())

const rootEl = document.querySelector(APP_CONTAINER_SELECTOR)

const wrapApp = (AppComponent, reduxStore) =>
  <Provider store={reduxStore}>
    <AppContainer>
      <AppComponent />
    </AppContainer>
  </Provider>

ReactDOM.render(wrapApp(App, store), rootEl)

if (module.hot) {
  // flow-disable-next-line
  module.hot.accept('./app', () => {
    // eslint-disable-next-line global-require
    const NextApp = require('./app').default
    ReactDOM.render(wrapApp(NextApp, store), rootEl)
  })
}
```

Давайте воспользуемся моментом для детального рассмотрения. Первое, мы создаем *store* благодаря `createStore`. Stores (хранилища) создаются проходя через соответсвующие reducers (редьюсеры). Здесь у нас есть только один редьюсер, но ради будущей масштабируемости, мы используем `combineReducers` для группировки всех наших редьюсеров вместе. Последний магический параметр `createStore` используется для доступа к Redux в браузере [Devtools](https://github.com/zalmoxisus/redux-devtools-extension), что невероятно полезно при отладке. Поскольку ESLint будет жаловаться на подчеркивания `__REDUX_DEVTOOLS_EXTENSION__`, мы отлключаем это ESLint правило. Далее, мы удобно оборачиваем наше приложение внутрь `react-redux`'s `Provider` компонент благодаря нашей `wrapApp` функци, и передаем наш store (хранилище) ему.

🏁 Ты можешь запустить `yarn start` и `yarn dev:wds` и перейди `http://localhost:8000`. Ты должен увидеть "Initial reducer message" и кнопку. Когда вы кликните на кнопку, сообщение должно измениться на "Hello!". Если вы установили Redux Devtools в вашем браузете, вы должны увидеть измениние state приложения, после того как вы кликните на кнопку.

Подзравляем, мы наконец-то сделали приложение, которое делает что-то! Ладно, это не *супер* впечатляющией фронтенд, но мы ве знаем, что под капотом скрывается крутой стек.

## Расширяем наше приложение асинхронными вызовами

Тепер мы собираемся добавить вторую кнопку в наше приложение, которая будет посылать AJAX запрос для получения сообщения с сервера. Для демонстрации этого вызова также будем отправлять некоторые данные, например захардкоженый номер `1234`.

### Сервер точка доступа

- Создайте файл `src/shared/routes.js` содержащий:

```js
// @flow

// eslint-disable-next-line import/prefer-default-export
export const helloEndpointRoute = (num: ?number) => `/ajax/hello/${num || ':num'}`
```

Эта фунция маленький помощник для воспроизведения следующего:

```js
helloEndpointRoute()     // -> '/ajax/hello/:num' (for Express)
helloEndpointRoute(1234) // -> '/ajax/hello/1234' (for the actual call)
```

Давайте быстро создадим настоящий тест, чтобы убедиться, что эта штука хорошо работает.

- Создайте файл `src/shared/routes.test.js` содержащий:

```js
import { helloEndpointRoute } from './routes'

test('helloEndpointRoute', () => {
  expect(helloEndpointRoute()).toBe('/ajax/hello/:num')
  expect(helloEndpointRoute(123)).toBe('/ajax/hello/123')
})
```

- Запустите `yarn test` и убедитесь, что тесты проходят.

- В `src/server/index.js`, добавьте следующее:

```js
import { helloEndpointRoute } from '../shared/routes'

// [under app.get('/')...]

app.get(helloEndpointRoute(), (req, res) => {
  res.json({ serverMessage: `Hello from the server! (received ${req.params.num})` })
})
```

### Новые контейнеры

- Создайте файл `src/client/container/hello-async-button.js` содержащий следющее:

```js
// @flow

import { connect } from 'react-redux'

import { sayHelloAsync } from '../action/hello'
import Button from '../component/button'

const mapStateToProps = () => ({
  label: 'Say hello asynchronously and send 1234',
})

const mapDispatchToProps = dispatch => ({
  handleClick: () => { dispatch(sayHelloAsync(1234)) },
})

export default connect(mapStateToProps, mapDispatchToProps)(Button)
```

In order to demonstrate how you would pass a parameter to your asynchronous call and to keep things simple, I am hard-coding a `1234` value here. This value would typically come from a form field filled by the user.

- Create a `src/client/container/message-async.js` file containing:

```js
// @flow

import { connect } from 'react-redux'

import MessageAsync from '../component/message'

const mapStateToProps = state => ({
  message: state.hello.get('messageAsync'),
})

export default connect(mapStateToProps)(MessageAsync)
```

You can see that in this container, we are referring to a `messageAsync` property, which we're going to add to our reducer soon.

What we need now is to create the `sayHelloAsync` action.

### Fetch

> 💡 **[Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)** is a standardized JavaScript function to make asynchronous calls inspired by jQuery's AJAX methods.

We are going to use `fetch` to make calls to the server from the client. `fetch` is not supported by all browsers yet, so we are going to need a polyfill. `isomorphic-fetch` is a polyfill that makes it work cross-browsers and in Node too!

- Run `yarn add isomorphic-fetch`

Since we're using `eslint-plugin-compat`, we need to indicate that we are using a polyfill for `fetch` to not get warnings from using it.

- Add the following to your `.eslintrc.json` file:

```json
"settings": {
  "polyfills": ["fetch"]
},
```

### 3 asynchronous actions

`sayHelloAsync` is not going to be a regular action. Asynchronous actions are usually split into 3 actions, which trigger 3 different states: a *request* action (or "loading"), a *success* action, and a *failure* action.

- Edit `src/client/action/hello.js` like so:

```js
// @flow

import 'isomorphic-fetch'

import { createAction } from 'redux-actions'
import { helloEndpointRoute } from '../../shared/routes'

export const SAY_HELLO = 'SAY_HELLO'
export const SAY_HELLO_ASYNC_REQUEST = 'SAY_HELLO_ASYNC_REQUEST'
export const SAY_HELLO_ASYNC_SUCCESS = 'SAY_HELLO_ASYNC_SUCCESS'
export const SAY_HELLO_ASYNC_FAILURE = 'SAY_HELLO_ASYNC_FAILURE'

export const sayHello = createAction(SAY_HELLO)
export const sayHelloAsyncRequest = createAction(SAY_HELLO_ASYNC_REQUEST)
export const sayHelloAsyncSuccess = createAction(SAY_HELLO_ASYNC_SUCCESS)
export const sayHelloAsyncFailure = createAction(SAY_HELLO_ASYNC_FAILURE)

export const sayHelloAsync = (num: number) => (dispatch: Function) => {
  dispatch(sayHelloAsyncRequest())
  return fetch(helloEndpointRoute(num), { method: 'GET' })
    .then((res) => {
      if (!res.ok) throw Error(res.statusText)
      return res.json()
    })
    .then((data) => {
      if (!data.serverMessage) throw Error('No message received')
      dispatch(sayHelloAsyncSuccess(data.serverMessage))
    })
    .catch(() => {
      dispatch(sayHelloAsyncFailure())
    })
}
```

Instead of returning an action, `sayHelloAsync` returns a function which launches the `fetch` call. `fetch` returns a `Promise`, which we use to *dispatch* different actions depending on the current state of our asynchronous call.

### 3 asynchronous action handlers

Let's handle these different actions in `src/client/reducer/hello.js`:

```js
// @flow

import Immutable from 'immutable'
import type { fromJS as Immut } from 'immutable'

import {
  SAY_HELLO,
  SAY_HELLO_ASYNC_REQUEST,
  SAY_HELLO_ASYNC_SUCCESS,
  SAY_HELLO_ASYNC_FAILURE,
} from '../action/hello'

const initialState = Immutable.fromJS({
  message: 'Initial reducer message',
  messageAsync: 'Initial reducer message for async call',
})

const helloReducer = (state: Immut = initialState, action: { type: string, payload: any }) => {
  switch (action.type) {
    case SAY_HELLO:
      return state.set('message', action.payload)
    case SAY_HELLO_ASYNC_REQUEST:
      return state.set('messageAsync', 'Loading...')
    case SAY_HELLO_ASYNC_SUCCESS:
      return state.set('messageAsync', action.payload)
    case SAY_HELLO_ASYNC_FAILURE:
      return state.set('messageAsync', 'No message received, please check your connection')
    default:
      return state
  }
}

export default helloReducer
```

We added a new field to our store, `messageAsync`, and we update it with different messages depending on the action we receive. During `SAY_HELLO_ASYNC_REQUEST`, we show `Loading...`. `SAY_HELLO_ASYNC_SUCCESS` updates `messageAsync` similarly to how `SAY_HELLO` updates `message`. `SAY_HELLO_ASYNC_FAILURE` gives an error message.

### Redux-thunk

In `src/client/action/hello.js`, we made `sayHelloAsync`, an action creator that returns a function. This is actually not a feature that is natively supported by Redux. In order to perform these async actions, we need to extend Redux's functionality with the `redux-thunk` *middleware*.

- Run `yarn add redux-thunk`

- Update your `src/client/index.jsx` file like so:

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'
import { AppContainer } from 'react-hot-loader'
import { Provider } from 'react-redux'
import { createStore, combineReducers, applyMiddleware, compose } from 'redux'
import thunkMiddleware from 'redux-thunk'

import App from './app'
import helloReducer from './reducer/hello'
import { APP_CONTAINER_SELECTOR } from '../shared/config'
import { isProd } from '../shared/util'

// eslint-disable-next-line no-underscore-dangle
const composeEnhancers = (isProd ? null : window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__) || compose

const store = createStore(combineReducers({ hello: helloReducer }),
  composeEnhancers(applyMiddleware(thunkMiddleware)))

const rootEl = document.querySelector(APP_CONTAINER_SELECTOR)

const wrapApp = (AppComponent, reduxStore) =>
  <Provider store={reduxStore}>
    <AppContainer>
      <AppComponent />
    </AppContainer>
  </Provider>

ReactDOM.render(wrapApp(App, store), rootEl)

if (module.hot) {
  // flow-disable-next-line
  module.hot.accept('./app', () => {
    // eslint-disable-next-line global-require
    const NextApp = require('./app').default
    ReactDOM.render(wrapApp(NextApp, store), rootEl)
  })
}
```

Here we pass `redux-thunk` to Redux's `applyMiddleware` function. In order for the Redux Devtools to keep working, we also need to use Redux's `compose` function. Don't worry too much about this part, just remember that we enhance Redux with `redux-thunk`.

- Update `src/client/app.jsx` like so:

```js
// @flow

import React from 'react'
import HelloButton from './container/hello-button'
import HelloAsyncButton from './container/hello-async-button'
import Message from './container/message'
import MessageAsync from './container/message-async'
import { APP_NAME } from '../shared/config'

const App = () =>
  <div>
    <h1>{APP_NAME}</h1>
    <Message />
    <HelloButton />
    <MessageAsync />
    <HelloAsyncButton />
  </div>

export default App
```

🏁 Run `yarn start` and `yarn dev:wds` and you should now be able to click the "Say hello asynchronously and send 1234" button and retrieve a corresponding message from the server! Since you're working locally, the call is instantaneous, but if you open the Redux Devtools, you will notice that each click triggers both `SAY_HELLO_ASYNC_REQUEST` and `SAY_HELLO_ASYNC_SUCCESS`, making the message go through the intermediate `Loading...` state as expected.

You can congratulate yourself, that was an intense section! Let's wrap it up with some testing.

## Testing

In this section, we are going to test our actions and reducer. Let's start with the actions.

In order to isolate the logic that is specific to `action/hello.js` we are going to need to *mock* things that don't concern it, and also mock that AJAX `fetch` request which should not trigger an actual AJAX in our tests.

- Run `yarn add --dev redux-mock-store fetch-mock`

- Create a `src/client/action/hello.test.js` file containing:

```js
import fetchMock from 'fetch-mock'
import configureMockStore from 'redux-mock-store'
import thunkMiddleware from 'redux-thunk'

import {
  sayHelloAsync,
  sayHelloAsyncRequest,
  sayHelloAsyncSuccess,
  sayHelloAsyncFailure,
} from './hello'

import { helloEndpointRoute } from '../../shared/routes'

const mockStore = configureMockStore([thunkMiddleware])

afterEach(() => {
  fetchMock.restore()
})

test('sayHelloAsync success', () => {
  fetchMock.get(helloEndpointRoute(666), { serverMessage: 'Async hello success' })
  const store = mockStore()
  return store.dispatch(sayHelloAsync(666))
    .then(() => {
      expect(store.getActions()).toEqual([
        sayHelloAsyncRequest(),
        sayHelloAsyncSuccess('Async hello success'),
      ])
    })
})

test('sayHelloAsync 404', () => {
  fetchMock.get(helloEndpointRoute(666), 404)
  const store = mockStore()
  return store.dispatch(sayHelloAsync(666))
    .then(() => {
      expect(store.getActions()).toEqual([
        sayHelloAsyncRequest(),
        sayHelloAsyncFailure(),
      ])
    })
})

test('sayHelloAsync data error', () => {
  fetchMock.get(helloEndpointRoute(666), {})
  const store = mockStore()
  return store.dispatch(sayHelloAsync(666))
    .then(() => {
      expect(store.getActions()).toEqual([
        sayHelloAsyncRequest(),
        sayHelloAsyncFailure(),
      ])
    })
})
```

Alright, Let's look at what's happening here. First we mock the Redux store using `const mockStore = configureMockStore([thunkMiddleware])`. By doing this we can dispatch actions without them triggering any reducer logic. For each test, we mock `fetch` using `fetchMock.get()` and make it return whatever we want. What we actually test using `expect()` is which series of actions have been dispatched by the store, thanks to the `store.getActions()` function from `redux-mock-store`. After each test we restore the normal behavior of `fetch` with `fetchMock.restore()`.

Let's now test our reducer, which is much easier.

- Create a `src/client/reducer/hello.test.js` file containing:

```js
import {
  sayHello,
  sayHelloAsyncRequest,
  sayHelloAsyncSuccess,
  sayHelloAsyncFailure,
} from '../action/hello'

import helloReducer from './hello'

let helloState

beforeEach(() => {
  helloState = helloReducer(undefined, {})
})

test('handle default', () => {
  expect(helloState.get('message')).toBe('Initial reducer message')
  expect(helloState.get('messageAsync')).toBe('Initial reducer message for async call')
})

test('handle SAY_HELLO', () => {
  helloState = helloReducer(helloState, sayHello('Test'))
  expect(helloState.get('message')).toBe('Test')
})

test('handle SAY_HELLO_ASYNC_REQUEST', () => {
  helloState = helloReducer(helloState, sayHelloAsyncRequest())
  expect(helloState.get('messageAsync')).toBe('Loading...')
})

test('handle SAY_HELLO_ASYNC_SUCCESS', () => {
  helloState = helloReducer(helloState, sayHelloAsyncSuccess('Test async'))
  expect(helloState.get('messageAsync')).toBe('Test async')
})

test('handle SAY_HELLO_ASYNC_FAILURE', () => {
  helloState = helloReducer(helloState, sayHelloAsyncFailure())
  expect(helloState.get('messageAsync')).toBe('No message received, please check your connection')
})
```

Before each test, we initialize `helloState` with the default result of our reducer (the `default` case of our `switch` statement in the reducer, which returns `initialState`). The tests are then very explicit, we just make sure the reducer updates `message` and `messageAsync` correctly depending on which action it received.

🏁 Run `yarn test`. It should be all green.

Next section: [06 - React Router, Server-Side Rendering, Helmet](06-react-router-ssr-helmet.md#readme)

Back to the [previous section](04-webpack-react-hmr.md#readme) or the [table of contents](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
