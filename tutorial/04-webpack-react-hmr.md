# 04 - Webpack, React, e Hot Module Replacement

Il codice per questo capitolo si trova [qua](https://github.com/verekia/js-stack-walkthrough/tree/master/04-webpack-react-hmr).

## Webpack

> 💡 **[Webpack](https://webpack.js.org/)** è un *module bundler*. Si occupa di prendere un insieme di vari file sorgenti, processarli, e assemblarli in un unico (solitamente) file JavaScript chiamato bundle, che è l'unico file che il client eseguirà.

Creiamo un semplice *hello world* e facciamone il bundle con Webpack.

- In `src/shared/config.js`, aggiungi queste costanti:

```js
export const WDS_PORT = 7000

export const APP_CONTAINER_CLASS = 'js-app'
export const APP_CONTAINER_SELECTOR = `.${APP_CONTAINER_CLASS}`
```

- Crea il file `src/client/index.js` contenente:

```js
import 'babel-polyfill'

import { APP_CONTAINER_SELECTOR } from '../shared/config'

document.querySelector(APP_CONTAINER_SELECTOR).innerHTML = '<h1>Hello Webpack!</h1>'
```

Se vuoi utilizzare alcune delle funzionalità ES più recenti nel codice del client, ad esempio le `Promise`, devi includere il [Polyfill di Babel](https://babeljs.io/docs/usage/polyfill/) prima di qualunque altra cosa all'interno del bundle.

- Esegui `yarn add babel-polyfill`

Se lanci ESLint su questo file, segnalerà che `document` è undefined.

- Aggiungi il campo `env` nel file `.eslintrc.json` per permettere l'utilizzo di `window` e `document`:

```json
"env": {
  "browser": true,
  "jest": true
}
```

Bene, adesso dobbiamo fare il bundle di questa applicazione client ES6 in un bundle ES5.

- Crea il file `webpack.config.babel.js` contenente:

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
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist/js'),
    publicPath: `http://localhost:${WDS_PORT}/dist/js/`,
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

Questo file serve a descrivere in che modo assemblare il nostro bundle: `entry` è il punto di accesso della nostra app, `output.filename` è il nome del file di bundle da generare, `output.path` e `output.publicPath` descrivono la cartella di destinazione e l'URL. Mettiamo il bundle nella cartella `dist`, che conterrà i file generati automaticamente (a differenza del CSS che abbiamo creato in precedenza e si trova in `public`). `module.rules` è dove spieghi a Webpack di applicare alcune operazioni a certi tipi di file. Qua diciamo che vogliamo che tutti i file `.js` e `.jsx` (per React) ad esclusione di quelli in `node_modules` passino attraverso un modulo chiamato `babel-loader`. Vogliamo inoltre che queste due estensioni vengano utilizzate per la risoluzione dei moduli (`resolve`). Infine definiamo una porta per il Webpack Dev Server.

**Nota**: L'estensione `.babel.js` è una funzionalità di Webpack che permette di applicare le trasformazioni di Babel a questo file di configurazione.

`babel-loader` è un plugin per Webpack che transpila il codice esattamente come abbiamo fatto dall'inizio di questo tutorial. L'unica differenza è che adesso il codice verrà eseguito all'interno del browser invece che nel server.

- Esegui `yarn add --dev webpack webpack-dev-server babel-core babel-loader`

`babel-core` è una dipendenza di `babel-loader`, quindi dobbiamo installarlo.

- Aggiungi `/dist/` al file `.gitignore`

### Update dei Task

In modalità di sviluppo utilizzeremo `webpack-dev-server` per sfruttare le funzionalità di Hot Module Reloading (più avanti in questo capitolo), mentre in produzione utilizzeremo semplicemente `webpack` per generare i bundle. In entrambe i casi, il parametro `--progress` viene utilizzato per mostrare informazioni aggiuntive quando Webpack sta compilando i file. In produzione passeremo a `webpack` anche il parametro `-p` per minificare il codice, e la variabile `NODE_ENV` impostata a `production`.

Aggiorniamo gli `scripts` per implementare il tutto, e migliorare anche altri task:

```json
"scripts": {
  "start": "yarn dev:start",
  "dev:start": "nodemon -e js,jsx --ignore lib --ignore dist --exec babel-node src/server",
  "dev:wds": "webpack-dev-server --progress",
  "prod:build": "rimraf lib dist && babel src -d lib --ignore .test.js && cross-env NODE_ENV=production webpack -p --progress",
  "prod:start": "cross-env NODE_ENV=production pm2 start lib/server && pm2 logs",
  "prod:stop": "pm2 delete all",
  "lint": "eslint src webpack.config.babel.js --ext .js,.jsx",
  "test": "yarn lint && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test && yarn prod:build"
},
```

In `dev:start` dichiariamo esplicitamente le estensioni da monitorare, `.js` e `.jsx`, e aggiungiamo `dist` alle cartelle da ignorare.

Creiamo un task `lint` separato e aggiungiamo `webpack.config.babel.js` ai file su cui effettuare il lint.

- Adesso creiamo il contenitore per la nostra app in `src/server/render-app.js`, ed includiamo il bundle che verrà generato:

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

In base all'ambiente in cui ci troviamo, includeremo o il bundle del Webpack Dev Server, oppure quello di produzione. Nota che il percorso per il bundle del Webpack Dev Server è *virtuale*, `dist/js/bundle.js` non viene di fatto letto dall'hard disk in modalità sviluppo. È inoltre necessario configurare il Webpack Dev Server in modo da utilizzare una porta differente da quella del tuo server web principale.

- Infine, in `src/server/index.js`, modifica il messaggio della `console.log` in questo modo:

```js
console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
  '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
```

Questo suggerirà agli altri sviluppatori cosa fare nel caso eseguissero `yarn start` senza il Webpack Dev Server.

OK, abbiamo modificato molte cose, vediamo se tutto continua a funzionare come dovrebbe:

🏁 Lancia `yarn start` in un terminale. Apri un altro terminale, ed esegui `yarn dev:wds`. Quando il Webpack Dev Server ha terminato di generare il bundle ed i sourcemap (che dovrebbero essere entrambi file di ~600kB) ed entrambe i processi sono fermi nei rispettivi terminal, apri `http://localhost:8000/` e dovresti vedere "Hello Webpack!". Apri la console di Chrome e nella scheda Sources controlla quali file sono inclusi. Dovresti vedere semplicemente `static/css/style.css` sotto `localhost:8000/`, e tutti i file ES6 sotto `webpack://./src`. Questo significa che i sourcemap stanno funzionando. Nell'editor, in `src/client/index.js`, prova a cambiare `Hello Webpack!` in qualcos'altro. Dopo aver salvato il file, Webpack Dev Server nel terminale dovrebbe generare un nuovo bundle e la scheda di Chrome dovrebbe aggiornarsi automaticamente.

- Killa i processi precedenti nei terminali premento Ctrl+C, poi esegui `yarn prod:build`, e successivamente `yarn prod:start`. Apri `http://localhost:8000/` e dovresti vedere "Hello Webpack!". Nella scheda Sources della console di Chrome, questa volta dovresti trovare `static/js/bundle.js` sotto a `localhost:8000/`, ma nessun sorgente sotto `webpack://`. Clicca su `bundle.js` e assicurati che sia minificato. Esegui `yarn prod:stop`.

Ottimo lavoro, lo so che è stato un po' intenso. Ti meriti una pausa! La prossima sezione è più semplice.

**Nota**: Suggerirei di avere almeno 3 terminali aperti, uno per il server Express, uno per il Webpack Dev Server, e uno per Git, i test, e comandi generali come installare pacchetti con `yarn`. Idealmente, dovresti dividere lo schermo dei terminali in più pannelli in modo da poterli vedere tutti contemporaneamente.

## React

> 💡 **[React](https://facebook.github.io/react/)** è una libreria per costruire interfacce utente sviluppata da Facebook. Utilizza la sintassi  **[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html)** per rappresentare elementi HTML e componenti sfruttando la potenza di JavaScript.

In questa sezione faremo il rendering di un po' di testo con React e JSX.

Prima di tutto installiamo React e ReactDOM:

- Esegui `yarn add react react-dom`

Rinomina `src/client/index.js` in `src/client/index.jsx` e scrivici del codice per React:

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'

import App from './app'
import { APP_CONTAINER_SELECTOR } from '../shared/config'

ReactDOM.render(<App />, document.querySelector(APP_CONTAINER_SELECTOR))
```

- Crea il file `src/client/app.jsx` contenente:

```js
// @flow

import React from 'react'

const App = () => <h1>Hello React!</h1>

export default App
```

Siccome stiamo usando la sintassi JSX, dobbiamo dire a Babel che deve trasformare anche questo.

- Esegui `yarn add --dev babel-preset-react` e aggiungi `react` al file `.babelrc` in questo modo:

```json
{
  "presets": [
    "env",
    "flow",
    "react"
  ]
}
```

🏁 Lancia `yarn start` e `yarn dev:wds` e apri `http://localhost:8000`. Dovresti vedere "Hello React!".

Adesso prova a cambiare il testo in `src/client/app.jsx` in qualcos'altro. Webpack Dev Server dovrebbe ricaricare la pagina automaticamente, che è bello, ma lo renderemo ancora migliore.

## Hot Module Replacement

> 💡 **[Hot Module Replacement](https://webpack.js.org/concepts/hot-module-replacement/)** (*HMR*) è una funzionalità di Webpack per rimpiazzare un modulo al volo senza dover ricaricare tutta la pagina.

Per fare in modo che HMR funzioni con React, dovremo modificare alcune cose.

- Esegui `yarn add react-hot-loader@next`

- Modifica `webpack.config.babel.js` così:

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

- Modifica `src/client/index.jsx`:

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

Dobbiamo rendere `App` un figlio dell'`AppContainer` di `react-hot-loader`, e dobbiamo fare una `require` della versione successiva di `App` quando facciamo l'hot-reloading. Per rendere questo processo pulito ed efficiente, creiamo una funzione `wrapApp` che utilizziamo in entrambe i punti in cui occorre fare il render di `App`. Sentiti libero di spostare `eslint-disable global-require` in cima al file per renderlo più leggibile.

🏁 Riavvia il processo `yarn dev:wds` se era ancora in esecuzione. Apri `localhost:8000`. Nella scheda Console, dovresti vedere alcuni log relativi a HMR. Prosegui a modificare qualcosa in `src/client/app.jsx` e le modifiche dovrebbero comparire nel browser dopo qualche secondo, senza dover ricaricare l'intera pagina!

Prossimo capitolo: [05 - Redux, Immutable, Fetch](05-redux-immutable-fetch.md#readme)

Torna al [capitolo precedente](03-express-nodemon-pm2.md#readme) o all'[indice dei contenuti](https://github.com/fbertone/guida-javascript-moderno#indice-dei-contenuti).
