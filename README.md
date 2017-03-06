# Guida alla creazione di un progetto Javascript Moderno

[![Build Status](https://travis-ci.org/verekia/js-stack-from-scratch.svg?branch=master)](https://travis-ci.org/verekia/js-stack-from-scratch)
[![Release](https://img.shields.io/github/release/verekia/js-stack-from-scratch.svg?style=flat-square)](https://github.com/verekia/js-stack-from-scratch/releases)
[![Dependencies](https://img.shields.io/david/verekia/js-stack-boilerplate.svg?style=flat-square)](https://david-dm.org/verekia/js-stack-boilerplate)
[![Dev Dependencies](https://img.shields.io/david/dev/verekia/js-stack-boilerplate.svg?style=flat-square)](https://david-dm.org/verekia/js-stack-boilerplate?type=dev)
[![Gitter](https://img.shields.io/gitter/room/js-stack-from-scratch/Lobby.svg?style=flat-square)](https://gitter.im/js-stack-from-scratch/)

[![React](/img/react-padded-90.png)](https://facebook.github.io/react/)
[![Redux](/img/redux-padded-90.png)](http://redux.js.org/)
[![React Router](/img/react-router-padded-90.png)](https://github.com/ReactTraining/react-router)
[![Flow](/img/flow-padded-90.png)](https://flowtype.org/)
[![ESLint](/img/eslint-padded-90.png)](http://eslint.org/)
[![Jest](/img/jest-padded-90.png)](https://facebook.github.io/jest/)
[![Yarn](/img/yarn-padded-90.png)](https://yarnpkg.com/)
[![Webpack](/img/webpack-padded-90.png)](https://webpack.github.io/)
[![Bootstrap](/img/bootstrap-padded-90.png)](http://getbootstrap.com/)

Benvenuto a questa guida: **Guida alla creazione di un progetto Javascript Moderno**.

> 🎉 **Questa è la seconda versione della guida, grosse modifiche sono state fatte rispetto alla prima. Controlla il [Change Log](/CHANGELOG.md)!**

Questa è una guida essenziale per la costruzione di uno stack Javascript utilizzando gli strumenti più attuali. Richiede qualche conoscienza generica di programmazione ed alcune basi di Javascript. **È focalizzata sull'utilizzo in sequenza di alcuni tool** e ti presenta **l'esempio più semplice possibile** per ogni tool. Puoi vedere questa guida come *un modo per scrivere il tuo boilerplate personale partendo da zero*. Siccome l'obiettivo di questa guida è l'utilizzo combinato di una serie di tool, non ti spiegherò nel dettaglio come funziona ciascun tool da solo. Fai riferimento alla loro documentazione o cerca altri tutorial se ti interessa approfondire maggiormente.

Chiaramente non avrai bisogno di utilizzare tutto lo stack se devi solo creare una semplice pagina web con poche interazioni in JS (una combinazione di Browserify/Webpack + Babel + jQuery è sufficiente per poter scrivere codice ES6 in file separati), ma se vuoi creare una web app scalabile, ed hai bisogno di un aiuto per configurare il tutto, questo tutorial farà proprio al tuo caso.

Una buona parte dello stack descritto in questa guida utilizza React. Se sei un principiante e vuoi solo imparare React, il progetto  [create-react-app](https://github.com/facebookincubator/create-react-app) ti permetterà di avere un ambiente React funzionante molto rapidamente, utilizzando una configurazione già pronta. Consiglierei questa strada a chi ad esempio entra in un team che utilizza già React e deve mettersi al passo utilizzando un ambiente semplice per fare delle prove. In questa guida non ti fornirò una configurazione già pronta perchè voglio che tu comprenda tutto il procedimento che c'è dietro.

Esempi di codice sono inclusi in ogni capitolo, puoi avviarli utilizzando i comandi `yarn && yarn start`. Ti consiglio comunque di riscrivere tutto per conto tuo seguendo le **istruzioni passo a passo**.

Il codice completo è disponibile in questo repository: [JS-Stack-Boilerplate](https://github.com/verekia/js-stack-boilerplate) e nelle [release](https://github.com/verekia/js-stack-from-scratch/releases). C'è anche una [demo live](https://js-stack.herokuapp.com/)

Funziona su Linux, macOS, e Windows.

## Indice dei contenuti

[01 - Node, Yarn, `package.json`](/tutorial/01-node-yarn-package-json.md#readme)

[02 - Babel, ES6, ESLint, Flow, Jest, Husky](/tutorial/02-babel-es6-eslint-flow-jest-husky.md#readme)

[03 - Express, Nodemon, PM2](/tutorial/03-express-nodemon-pm2.md#readme)

[04 - Webpack, React, HMR](/tutorial/04-webpack-react-hmr.md#readme)

[05 - Redux, Immutable, Fetch](/tutorial/05-redux-immutable-fetch.md#readme)

[06 - React Router, Server-Side Rendering, Helmet](/tutorial/06-react-router-ssr-helmet.md#readme)

[07 - Socket.IO](/tutorial/07-socket-io.md#readme)

[08 - Bootstrap, JSS](/tutorial/08-bootstrap-jss.md#readme)

[09 - Travis, Coveralls, Heroku](/tutorial/09-travis-coveralls-heroku.md#readme)

## Prossimamente

Impostare il tuo editor (Atom inizialmente), MongoDB, Progressive Web App.

## Traduzioni

Se vuoi aggiungere la tua traduzione, leggi le [raccomandazioni](/how-to-translate.md) per iniziare!

### V2

Presto nuove traduzioni ;)

Controlla le [traduzioni in corso](https://github.com/verekia/js-stack-from-scratch/issues/147).

### V1

- [中文](https://github.com/pd4d10/js-stack-from-scratch) by [@pd4d10](http://github.com/pd4d10)
- [Italiano](https://github.com/fbertone/js-stack-from-scratch) by [Fabrizio Bertone](https://github.com/fbertone)
- [日本語](https://github.com/takahashim/js-stack-from-scratch) by [@takahashim](https://github.com/takahashim)
- [Русский](https://github.com/UsulPro/js-stack-from-scratch) by [React Theming](https://github.com/sm-react/react-theming)
- [ไทย](https://github.com/MicroBenz/js-stack-from-scratch) by [MicroBenz](https://github.com/MicroBenz)

## Credits

Versione originale creata da [@verekia](https://twitter.com/verekia) – [verekia.com](http://verekia.com/).
Traduzione di [Fabrizio Bertone](https://github.com/fbertone) – [fbertone.it](http://fbertone.it/).

Licenza: MIT
