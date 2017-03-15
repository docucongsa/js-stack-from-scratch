# 03 - Express, Nodemon и PM2

Code for this chapter available [here](https://github.com/verekia/js-stack-walkthrough/tree/master/03-express-nodemon-pm2).
Код для этой главы доступен [тут](https://github.com/verekia/js-stack-walkthrough/tree/master/03-express-nodemon-pm2).

In this section we are going to create the server that will render our web app. We will also set up a development mode and a production mode for this server.
В этом разделе мы создадим сервер, который будет рендерить наше веб приложение. Также мы настроим для этого сервера режимы разработки и выпуска (production).

## Express

> 💡 **[Express](http://expressjs.com/)** is by far the most popular web application framework for Node. It provides a very simple and minimal API, and its features can be extended with *middleware*.
Express к тому же наиболее популярный фреймворк для веб приложений под Node. У него очень простой и минимальный API, а его возможности могут быть расширены с помощью *промежуточного ПО* (middleware). 


Let's set up a minimal Express server to serve an HTML page with some CSS.
Давайте настроим минимальный сервер Express, выдающий HTML страницу и немного CSS.  

- Delete everything inside Удалите все внутри `src`

Create the following files and folders:
Создайте следующие файлы и папки:

- Create a `public/css/style.css` file containing:
- Создайте файл `public/css/style.css` содержащий:

```css
body {
  width: 960px;
  margin: auto;
  font-family: sans-serif;
}

h1 {
  color: limegreen;
}
```

- Create an empty `src/client/` folder.
- Создайте пустую папку `src/client/`.

- Create an empty `src/shared/` folder.
- Создайте пустую папку `src/shared/`.

This folder is where we put *isomorphic / universal* JavaScript code – files that are used by both the client and the server. A great use case of shared code is *routes*, as you will see later in this tutorial when we'll make an asynchronous call. Here we simply have some configuration constants as an example for now.
Эта папка - место в которое мы поместим *изоморфный / универсальный* JavaScript код - файлы которые будут использованы как на клиенте так и на сервере. Отличный пример использования общего кода - *маршруты* (routes), как вы увидите дальше в этом руководстве когда мы будем использовать асинхронный вызов. Пока что мы просто разместим тут несколько конфигурационных констант в качестве примера.


- Create a `src/shared/config.js` file, containing:
- Создайте файл `src/shared/config.js`, содержащий:

```js
// @flow

export const WEB_PORT = process.env.PORT || 8000
export const STATIC_PATH = '/static'
export const APP_NAME = 'Hello App'
```

If the Node process used to run your app has a `process.env.PORT` environment variable set (that's the case when you deploy to Heroku for instance), it will use this for the port. If there is none, we default to `8000`.
Если процесс Node, запускающий ваше приложение содержит переменную окружения `process.env.PORT` (например, в случае, если вы публикуете на Heroku), она будет задавать порт. В противном случае, по умолчанию будет использоваться `8000`.

- Create a `src/shared/util.js` file containing:
- Создайте файл `src/shared/util.js`, содержащий:

```js
// @flow

// eslint-disable-next-line import/prefer-default-export
export const isProd = process.env.NODE_ENV === 'production'
```

That's a simple util to test if we are running in production mode or not. The `// eslint-disable-next-line import/prefer-default-export` comment is because we only have one named export here. You can remove it as you add other exports in this file.
Это простая утилита для проверки запущены ли мы в режиме production или нет. Комментарий `// eslint-disable-next-line import/prefer-default-export` нужен изза того, что у нас только один именованный экспорт в это файле. Вы можете его убрать как только добавите сюда другие экспортируемые переменные.

- Выполните `yarn add express compression`

`compression` is an Express middleware to activate Gzip compression on the server.
`compression` - это промежуточное ПО для Express активирующее Gzip сжатие на сервере.

- Создайте файл `src/server/index.js` содержащий:

```js
// @flow

import compression from 'compression'
import express from 'express'

import { APP_NAME, STATIC_PATH, WEB_PORT } from '../shared/config'
import { isProd } from '../shared/util'
import renderApp from './render-app'

const app = express()

app.use(compression())
app.use(STATIC_PATH, express.static('dist'))
app.use(STATIC_PATH, express.static('public'))

app.get('/', (req, res) => {
  res.send(renderApp(APP_NAME))
})

app.listen(WEB_PORT, () => {
  // eslint-disable-next-line no-console
  console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' : '(development)'}.`)
})
```

Nothing fancy here, it's almost Express' Hello World tutorial with a few additional imports. We're using 2 different static file directories here. `dist` for generated files, `public` for declarative ones.
Здесь ничего особенного, это практически 'Hello World' для Express плюс несколько дополнительных импортов. Мы здесь используем две отдельных директории для статических файлов. `dist` для генерируемых и `public` для декларируемых.

- Создайте файл `src/server/render-app.js` содержащий:

```js
// @flow

import { STATIC_PATH } from '../shared/config'

const renderApp = (title: string) =>
`<!doctype html>
<html>
  <head>
    <title>${title}</title>
    <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
  </head>
  <body>
    <h1>${title}</h1>
  </body>
</html>
`

export default renderApp
```

You know how you typically have *templating engines* on the back-end? Well these are pretty much obsolete now that JavaScript supports template strings. Here we create a function that takes a `title` as a parameter and injects it in both the `title` and `h1` tags of the page, returning the complete HTML string. We also use a `STATIC_PATH` constant as the base path for all our static assets.
Возможно, вам привычно использовать *шаблонизаторы* при работе с back-end? Что ж, теперь их можно считать довольно устаревшимы с тех пор, как JavaScript стал поддерживать шаблонные строки. Здесь мы создали функцию, которая принимает в качестве параметра `title` и вставляет это значение в тэги `title` и `h1`, возвращая строку с полноценной HTML страницей. Мы также используем константу `STATIC_PATH` для задания базового пути для всех наших статических элементов.

### HTML template strings syntax highlighting in Atom (optional)
### Подсветка HTML синтаксиса в шаблонных строках для Atom (не обязательное)

It might be possible to get syntax highlighting working for HTML code inside template strings depending on your editor. In Atom, if you prefix your template string with an `html` tag (or any tag that *ends* with `html`, like `ilovehtml`), it will automatically highlight the content of that string. I sometimes use the `html` tag of the `common-tags` library to take advantage of this:
В зависимости от используемого текстового редактора, возможна подсветка синтаксиса HTML кода внутри шаблонных строк. В Атоме, если вы добавите префикс `html` к шаблонной строке (или любой другой префикс, заканчивающийся на `html`, как `ilovehtml`), то содержимое этой строки автоматически будет подсвечиваться. Я иногда использую `html` тэг из библиотеки `common-tags`, чтобы воспользоваться данной возможностью:

```js
import { html } from `common-tags`

const template = html`
<div>Wow, colors!</div>
`
```

I did not include this trick in the boilerplate of this tutorial, since it seems to only work in Atom, and it's less than ideal. Some of you Atom users might find it useful though.
Я не стал включать этот трюк в boilerplate этого руководства, поскольку это, похоже работает только в Атоме, и это не идеальный подход. Однако для тех, кто использует Атом, это может быть полезным.

Anyway, back to business!
В любом случае вернемся к нашему проекту.

- In `package.json` change your `start` script like so: `"start": "babel-node src/server",`
- В `package.json` измените скрипт `start` таким образом: `"start": "babel-node src/server",`


🏁 Run `yarn start`, and hit `localhost:8000` in your browser. If everything works as expected you should see a blank page with "Hello App" written both on the tab title and as a green heading on the page.
🏁 Запустите `yarn start` и откройте `localhost:8000` в браузере. Если все заработало как и ожидалось, то вы увидете пустую страницу с надписями "Hello App" в названит вкладки и на зеленом заголовке страницы.

**Note**: Some processes – typically processes that wait for things to happen, like a server for instance – will prevent you from entering commands in your terminal until they're done. To interrupt such processes and get your prompt back, press **Ctrl+C**. You can alternatively open a new terminal tab if you want to keep them running while being able to enter commands. You can also make these processes run in the background but that's out of the scope of this tutorial.
**Примечание**: Некоторые процессы (обычно процессы, ожидающие своего завершения, как, например, сервер) не позволяют вам вводить команды в терминале пока они не завершатся. Чтобы прервать подобный процесс и получить обратно приглашение к вводу, нажмите **Ctrl+C**. Как вариант, вы можете открыть еще одну вкладку с терминалом, если хотите, чтобы процесс работал, пока вы вводите команды. Вы также можете запустить эти процессы в фоне, но это вне рамок данного руководства.

## Nodemon

> 💡 **[Nodemon](https://nodemon.io/)** is a utility to automatically restart your Node server when file changes happen in the directory.
> 💡 **[Nodemon](https://nodemon.io/)** - утилита для автоматического перезапуска сервера Node при изменении файлов в директории.

We are going to use Nodemon whenever we are in **development** mode.
Мы будем использовать Nodemon в режиме **разработки**

- Запустите `yarn add --dev nodemon`

- Измените `scripts` так, чтобы:

```json
"start": "yarn dev:start",
"dev:start": "nodemon --ignore lib --exec babel-node src/server",
```

`start` is now just a pointer to an other task, `dev:start`. That gives us a layer of abstraction to tweak what the default task is.
Теперь `start` лишь указатель на другую задачу. Это дает нам уровень абстракции, позволяющий настраивать какая задача будет выполняться по умолчанию.

In `dev:start`, the `--ignore lib` flag is to *not* restart the server when changes happen in the `lib` directory. You don't have this directory yet, but we're going to generate it in the next section of this chapter, so it will soon make sense. Nodemon typically runs the `node` binary. In our case, since we're using Babel, we can tell Nodemon to use the `babel-node` binary instead. This way it will understand all the ES6/Flow code.
В `dev:start`, мы устанавливаем флаг `--ignore lib` для того, чтобы *не* перезапускать сервер, когда изменения происходят в директории `lib`. У вас пока еще нет этой директории, но мы создадим ее в следующем разделе этой главы. Так что скоро это пригодится. Обычно Nodemon используют для запуска двоичных файлов `node`. В нашем случае, поскольку мы используем Babel, мы, вместо этого, указали Nodemon запускать `babel-node`. Таким образом, мы сделали доступным весь наш ES6/Flow код.

🏁 Run `yarn start` and open `localhost:8000`. Go ahead and change the `APP_NAME` constant in `src/shared/config.js`, which should trigger a restart of your server in the terminal. Refresh the page to see the updated title. Note that this automatic restart of the server is different from *Hot Module Replacement*, which is when components on the page update in real-time. Here we still need a manual refresh, but at least we don't need to kill the process and restart it manually to see changes. Hot Module Replacement will be introduced in the next chapter.
🏁 Запустите `yarn start` и откройте `localhost:8000`. Двигаемся дальше и изменим константу `APP_NAME` в `src/shared/config.js`, что должно вызвать перезапуск сервера в терминале. Обновите страницу, чтобы увидеть измененный заголовок. Заметьте, что этот автоматический рестарт сервера отличается от *Hot Module Replacement*, при котором компоненты обновляются на странице в реальном времени. Здесь нам по прежнему требуется ручное обновление, но по крайней мере не нужно убивать процесс и вручную перезапускать сервер, чтобы увидеть изменения. Hot Module Replacement будет представлен в следующей главе.

## PM2

> 💡 **[PM2](http://pm2.keymetrics.io/)** is a Process Manager for Node. It keeps your processes alive in production, and offers tons of features to manage them and monitor them.
> 💡 **[PM2](http://pm2.keymetrics.io/)** - это менеджер процессов для Node, обеспечивающий жизнеспособность вашего приложения и предлагающий тонны возможностей по управлению и мониторингу.

We are going to use PM2 whenever we are in **production** mode.
Мы будем использовать PM2 в режиме **production**


- Выполните `yarn add --dev pm2`

In production, you want your server to be as performant as possible. `babel-node` triggers the whole Babel transpilation process for your files at each execution, which is not something you want in production. We need Babel to do all this work beforehand, and have our server serve plain old pre-compiled ES5 files.
В production вы хотите, чтобы сервер был настолько производительным, насколько это возможно. `babel-node` начинает процесс транспиляции всех файлов при каждом перезапуске, чего вы бы хотели избежать в production. Нам нужно, чтобы Babel выполнил всю эту работу заранее, и сервер выдавал обычные старые предкомпилированные ES5 файлы.

One of the main features of Babel is to take a folder of ES6 code (usually named `src`) and transpile it into a folder of ES5 code (usually named `lib`).
Одной из основных возможностей Babel является способность взять папку с ES6 кодом (обычно `src`) и транспилировать его в папку с ES5 кодом (обычно `lib`). 

This `lib` folder being auto-generated, it's a good practice to clean it up before a new build, since it may contain unwanted old files. A neat simple package to delete files with cross platform support is `rimraf`.
Поскольку папка `lib` автогенерируется, хорошей практикой будет очищать ее перед каждым новым построением, поскольку она может содержать нежелательные старые файлы. `rimraf` – простой лаконичный пакет, для удаления файлов с кроссплатформенной поддержкой.

- Запустите `yarn add --dev rimraf`

Давайте добавим следующую задачу `prod:build` в `package.json`:

```json
"prod:build": "rimraf lib && babel src -d lib --ignore .test.js",
```

- Run `yarn prod:build`, and it should generate a `lib` folder containing the transpiled code, except for files ending in `.test.js` (note that `.test.jsx` files are also ignored by this parameter).
- Запустите `yarn prod:build`. Это должно сгенерировать папку `lib`, содержащую транспилированный код, за исключением файлов, заканчивающихся на `.test.js` (заметьте, что файлы `.test.jsx` также игнорируются с помощью этого параметра).

- Добавьте `/lib/` в `.gitignore`

One last thing: We are going to pass a `NODE_ENV` environment variable to our PM2 binary. With Unix, you would do this by running `NODE_ENV=production pm2`, but Windows uses a different syntax. We're going to use a small package called `cross-env` to make this syntax work on Windows as well.
Последняя вещь: Мы собираемся передать переменную окружения `NODE_ENV` в исполняемый файл PM2. В Unix, вы бы сделали это через `NODE_ENV=production pm2`, но Windows использует другой синтаксис. Мы воспользуемся небольшим пакетом `cross-env`, чтобы заставить этот синтаксис работать также и для Windows.

- Запустите `yarn add --dev cross-env`

Обновим `package.json` так:

```json
"scripts": {
  "start": "yarn dev:start",
  "dev:start": "nodemon --ignore lib --exec babel-node src/server",
  "prod:build": "rimraf lib && babel src -d lib --ignore .test.js",
  "prod:start": "cross-env NODE_ENV=production pm2 start lib/server && pm2 logs",
  "prod:stop": "pm2 delete all",
  "test": "eslint src && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test"
},
```

🏁 Run `yarn prod:build`, then run `yarn prod:start`. PM2 should show an active process. Go to `http://localhost:8000/` in your browser and you should see your app. Your terminal should show the logs, which should be "Server running on port 8000 (production).". Note that with PM2, your processes are run in the background. If you press Ctrl+C, it will kill the `pm2 logs` command, which was the last command our our `prod:start` chain, but the server should still render the page. If you want to stop the server, run `yarn prod:stop`
🏁 Запустите `yarn prod:build`, а затем `yarn prod:start`. PM2 должен показать активный процесс. Зайдите на  `http://localhost:8000/`  в браузере и вы должны увидеть наше приложение. Терминал должен показать логи: "Server running on port 8000 (production).". Заметьте, что PM2 запускает процессы в фоне. Если вы нажмете Ctrl+C, это прервет команду `pm2 logs`, которая была последней в цепочке после `prod:start`, но сам сервер по прежнему должен генерировать страницы. Если вам нужно остановить сервер, наберите `yarn prod:stop`.

Now that we have a `prod:build` task, it would be neat to make sure it works fine before pushing code to the repository. Since it is probably unnecessary to run it for every commit, I suggest adding it to the `prepush` task:
Теперь, когда у нас есть задача `prod:build`, было бы здорово проверять все ли работает хорошо перед тем как закачивать код в репозиторий. Поскольку, возможно не требуется запускать его перед каждым коммитом, я предлагаю добавить это в задачу `prepush`:

```json
"prepush": "yarn test && yarn prod:build"
```

🏁 Run `yarn prepush` or just push your files to trigger the process.
🏁 Запустите `yarn prepush` или просто начните загружать файлы (push), чтобы запустить этот процесс.

**Note**: We don't have any test here, so Jest will complain a bit. Ignore it for now.
**Примечание**: У нас пока нет никаких тестов, так что Jest пожалуется на это. Пока что проигнорируйте это.

Next section: [04 - Webpack, React, HMR](04-webpack-react-hmr.md#readme)

Back to the [previous section](02-babel-es6-eslint-flow-jest-husky.md#readme) or the [table of contents](https://github.com/verekia/js-stack-from-scratch#table-of-contents).