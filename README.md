# JavaScript Stack from Scratch

[![Build Status](https://travis-ci.org/verekia/js-stack-from-scratch.svg?branch=master)](https://travis-ci.org/verekia/js-stack-from-scratch) [![Join the chat at https://gitter.im/js-stack-from-scratch/Lobby](https://badges.gitter.im/js-stack-from-scratch/Lobby.svg)](https://gitter.im/js-stack-from-scratch/Lobby)

[![React](/img/react-padded-90.png)](https://facebook.github.io/react/)
[![Redux](/img/redux-padded-90.png)](http://redux.js.org/)
[![React Router](/img/react-router-padded-90.png)](https://github.com/ReactTraining/react-router)
[![Flow](/img/flow-padded-90.png)](https://flowtype.org/)
[![ESLint](/img/eslint-padded-90.png)](http://eslint.org/)
[![Jest](/img/jest-padded-90.png)](https://facebook.github.io/jest/)
[![Yarn](/img/yarn-padded-90.png)](https://yarnpkg.com/)
[![Webpack](/img/webpack-padded-90.png)](https://webpack.github.io/)
[![Bootstrap](/img/bootstrap-padded-90.png)](http://getbootstrap.com/)

>Это русскоязычная версия руководства Джонатана Верекии ([@verekia](https://twitter.com/verekia)). Оригинальное руководство расположено [здесь](https://github.com/verekia/js-stack-from-scratch). **Начата [работа](https://github.com/UsulPro/js-stack-from-scratch/issues/8) по переводу второй части**. Первая версия находится [тут](https://github.com/UsulPro/js-stack-from-scratch-v1-rus)

Добро пожаловать в мое современное руководство по стеку технологий JavaScript: **Стек технологий JavaScript с нуля**.

> 🎉 **Это вторая версия руководства. По сравнению с предыдущм релизом 2016г произведены значительные изменения. См. [Change Log](/CHANGELOG.md)!**


Это практико-ориентированное пособие по применению JavaScript технологий. Вам потребуются общие знания по программированию и основы JavaScript. Это пособие **нацелено на интеграцию необходимых инструментов** и предоставляет **максимально простые примеры** для каждого инструмента. Вы можете рассматривать данный документ, как *возможность создать свой собственный шаблонный проект с нуля*. Поскольку целью этого руководства является сборка различных инструментов, я не буду вдаваться в детали по каждому из них. Если вы хотите получить по ним более глубокие знания, изучайте их документацию или другие руководства.

Конечно, вам не нужны все эти технологии, если вы делаете простую веб страницу с парой JS функций (комбинации Browserify / Webpack + Babel + jQuery достаточно, чтобы написать ES6 код в нескольких файлах), но если вы собираетесь создать масштабируемое веб приложение, и вам нужно все правильно настроить, то это руководство вам отлично подходит.

В большой части технологий, описываемых здесь, используется React. Если вы только начинаете использовать React и просто хотите изучить его, то [create-react-app](https://github.com/facebookincubator/create-react-app) поможет вам и кратко ознакомит с инфраструктурой React на основе предустановленной конфигурации. Я бы, например, порекомендовал такой подход для тех, кому нужно влиться в команду, использующую React, и на чем-то потренироваться, чтобы подтянуть свои знания. В этом руководстве мы не будем пользоваться предустановленными конфигурациями, поскольку я хочу, чтобы вы полностью понимали все, что происходит "под капотом".

В каждой части руководства имеются примеры кода, и вы можете запускать их через `yarn && yarn start`. Однако я рекомендую писать все с нуля самостоятельно, следуя **пошаговым инструкциям**.

Итоговый код данного руководства доступен в отдельном репозитории: [JS-Stack-Boilerplate repository](https://github.com/verekia/js-stack-boilerplate). Он работает под Linux, macOS, и Windows.

## Содержание

[01 - Node, Yarn, `package.json`](/tutorial/01-node-yarn-package-json_ru.md)

[02 - Babel, ES6, ESLint, Flow, Jest, Husky](/tutorial/02-babel-es6-eslint-flow-jest-husky.md)

[03 - Express, Nodemon, PM2](/tutorial/03-express-nodemon-pm2_ru.md)

[04 - Webpack, React, HMR](/tutorial/04-webpack-react-hmr.md)

[05 - Redux, Immutable, Fetch](/tutorial/05-redux-immutable-fetch_ru.md)

[06 - React Router, Server-Side Rendering, Helmet](/tutorial/06-react-router-ssr-helmet_ru.md)

[07 - Socket.IO](/tutorial/07-socket-io.md)

[08 - Bootstrap, JSS](/tutorial/08-bootstrap-jss.md)

[09 - Travis, Coveralls, Heroku](/tutorial/09-travis-coveralls-heroku.md)

## Далее планируется

Настройка вашего редактора (Atom и другие), MongoDB, Прогрессивное веб приложение (Progressive Web App).

## Переводы на другие языки

Если вы хотите добавить перевод на другой язык, пожалуйста читайте [рекомендации по переводу](/how-to-translate.md) чтобы начать!

### Версия 2

- [Русский _в процессе превода_](https://github.com/UsulPro/js-stack-from-scratch) by [React Theming](https://github.com/sm-react/react-theming)

### Версия 1

- [中文](https://github.com/pd4d10/js-stack-from-scratch) by [@pd4d10](http://github.com/pd4d10)
- [Italiano](https://github.com/fbertone/js-stack-from-scratch) by [Fabrizio Bertone](https://github.com/fbertone)
- [日本語](https://github.com/takahashim/js-stack-from-scratch) by [@takahashim](https://github.com/takahashim)
- [Русский](https://github.com/UsulPro/js-stack-from-scratch-v1-rus) by [React Theming](https://github.com/sm-react/react-theming)
- [ไทย](https://github.com/MicroBenz/js-stack-from-scratch) by [MicroBenz](https://github.com/MicroBenz)

## Сведения

Создано [@verekia](https://twitter.com/verekia) – [verekia.com](http://verekia.com/).

Переведено [@usulpro](https://github.com/UsulPro) - [react-theming](https://github.com/sm-react/react-theming)

Лицензия: MIT
