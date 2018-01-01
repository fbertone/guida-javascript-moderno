# 06 - React Router, Server-Side Rendering, ed Helmet

Il codice per questo capitolo è disponibile [qua](https://github.com/verekia/js-stack-walkthrough/tree/master/06-react-router-ssr-helmet).

In questo capitolo creeremo delle pagine differenti per la nostra app e renderemo possibile la loro navigazione.

## React Router

> 💡 **[React Router](https://reacttraining.com/react-router/)** è una libreria per la navigazione fra le pagine dell'app React. Può essere utilizzara sia nel client che nel server.

- Esegui `yarn add react-router react-router-dom`

Lato client, dobbiamo prima di tutto inserire la nostra app nel component `BrowserRouter`.

- Aggiorna `src/client/index.jsx` in questo modo:

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

## Pagine

La nostra app sarà composta da 4 pagine:

- Una Home page.
- Una pagina Hello con un bottone ed un messaggio per le azioni sincrone.
- Una pagina Hello Async con un bottone ed un messaggio per le azioni asincrone.
- Una pagina 404 "Not Found".

- Crea `src/client/component/page/home.jsx` contenente:

```js
// @flow

import React from 'react'

const HomePage = () => <p>Home</p>

export default HomePage
```

- Crea `src/client/component/page/hello.jsx` contenente:

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

- Crea `src/client/component/page/hello-async.jsx` contenente:

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

- Crea `src/client/component/page/not-found.jsx` contenente:

```js
// @flow

import React from 'react'

const NotFoundPage = () => <p>Page not found</p>

export default NotFoundPage
```

## Navigazione

Aggiungiamo alcune routes nel file di configurazione condiviso.

- Modifica `src/shared/routes.js` in questo modo:

```js
// @flow

export const HOME_PAGE_ROUTE = '/'
export const HELLO_PAGE_ROUTE = '/hello'
export const HELLO_ASYNC_PAGE_ROUTE = '/hello-async'
export const NOT_FOUND_DEMO_PAGE_ROUTE = '/404'

export const helloEndpointRoute = (num: ?number) => `/ajax/hello/${num || ':num'}`
```

La route `/404` verrà utilizzata in un link solo per far vedere cosa succede cliccando su un link non funzionante.

- Crea `src/client/component/nav.jsx` contenente:

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

Qua creiamo semplicemente alcuni `NavLink` che utilizziamo per dichiarare delle route.

- Infine, modifica `src/client/app.jsx` in questo modo:

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

🏁 Esegui `yarn start` e `yarn dev:wds`. Apri `http://localhost:8000`, e clicca sui link per navigare fra le pagine. YDovresti vedere l'URL che cambia dinamicamente. Prova ad usare il pulsante "indietro" del browser per verificare che la cronologia funzioni correttamente.

Adesso, immaginiamo che sei andato su `http://localhost:8000/hello`. Premi il pulsante aggiorna. Adesso ottieni un errore 404, perchè il nostro server Express risponde solo a `/`. Mentre navigavi tra le pagine, lo stavi di fatto facendo solo lato client. Aggiungiamo il server-side rendering per ottenere il comportamento voluto.

## Server-Side Rendering

> 💡 **Server-Side Rendering** significa fare il rendering dell'app al caricamento iniziale della pagina invece di effettuarlo via JavaScript all'interno del browser.

Il SSR è essenziale per il SEO e fornisce un'user experience migliore mostrando subito la pagina richiesta.

La prima cosa che faremo è migrare la maggiorparte del codice client verso la parte condivisa / isomorfica / universale pdel nostro codebase, siccome anche il server si occuperà di fare il render della nostra App React.

### La grande migrazione a `shared`

- Sposta tutti i file presenti in `client` verso `shared`, tranne `src/client/index.jsx`.

Dobbiamo mettere a posto tutta una serie di imports:

- In `src/client/index.jsx`, sostituisci le 3 occorrenze di `'./app'` con `'../shared/app'`, e `'./reducer/hello'` con `'../shared/reducer/hello'`

- In `src/shared/app.jsx`, sostituisci `'../shared/routes'` con `'./routes'` e `'../shared/config'` con `'./config'`

- In `src/shared/component/nav.jsx`, sostituisci `'../../shared/routes'` con `'../routes'`

### Modifiche al Server

- Crea `src/server/routing.js` contenente:

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

In questo file è dove ci occupiamo delle richieste e delle risposte. Le chiamate alla business logic sono demandate ad un modulo `controller` esterno.

**Nota**: Troverai molti esempi di React Router che utilizzano `*` come route sul server, lasciando tutto l'handling del routing a React Router. Siccome tutte le richieste vanno verso la stessa funzione, risulta non conveniente implementare delle pagine utilizzando lo stile MVC. Invece di fare così, noi stiamo dichiarando esplicitamente le routes e le risposte dedicate, per poter facilmente richiedere i dati al database e passarli alla pagina richiesta.

- Crea `src/server/controller.js` contenente:

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

Questo è il nostro controller. Tipicamente implementerà la business logic e le chiamate al database, ma nel nostro caso abbiamo semplicemente inserito in hard-code alcuni risultati. Questi risultati vengono inviati al modulo `routing` per poter inizializzare il nostro store Redux lato server.

- Crea `src/server/init-store.js` contenente:

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

L'unica cosa che facciamo qua, a parte chiamare `createStore` e applicare middleware, è unificare l'oggetto JS che abbiamo ricevuto dal `controller` in uno stato Redux di default contenente oggetti di Immutable.

- Modifica `src/server/index.js` in questo modo:

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

Niente di particolare qua, chiamiamo semplicemente `routing(app)` invece di implementare il routing in questo file.

- Rinomina `src/server/render-app.js` come `src/server/render-app.jsx` e modificalo in questo modo:

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

`ReactDOMServer.renderToString` è dove avviene la magia. React valuterà la `shared` `App`, e ritornerà una stringa di elementi HTML. `Provider` funziona come nel client, ma sul server, inseriamo la nostra app dentro a `StaticRouter` invece di `BrowserRouter`. Per passare lo store Redux dal server al client, lo passiamo a `window.__PRELOADED_STATE__` che è semplicemente il nome di una variabile arbitraria.

**Nota**: Gli oggetti Immutable implementano il metodo `toJSON()` quindi puoi usare `JSON.stringify` per convertirli in stringhe JSON.

- Modifica `src/client/index.jsx` per utilizzare lo stato precaricato:

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

Qua forniamo allo store del client il `preloadedState` che è stato ricevuto dal server.

🏁 Adesso puoi eseguire `yarn start` e `yarn dev:wds` e navigare tra le pagine. Ricaricando la pagina su `/hello`, `/hello-async`, e `/404` (o qualsiasi altro URI), dovrebbe adesso funzionare correttamente. Nota come `message` e `messageAsync` variano a seconda se sei andato sulla pagina via client o se è arrivata direttamente dal server.

### React Helmet

> 💡 **[React Helmet](https://github.com/nfl/react-helmet)**: Una libreria per iniettare contenuto nell'`head` di un'app React, sia sul client che sul server.

Ti ho volutamente fatto scrivere `FIX ME` per evidenziare il fatto che anche se stiamo facendo rendering lato server, non riempiamo correttamente il tag `title` (o qualunque tag all'interno dell'`head`, a seconda della pagina in cui ti trovi).

- Esegui `yarn add react-helmet`

- Modifica `src/server/render-app.jsx` in questo modo:

```js
import Helmet from 'react-helmet'
// [...]
const renderApp = (/* [...] */) => {
  // [...]
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

React Helmet usa la funionalità `rewind` di [react-side-effect](https://github.com/gaearon/react-side-effect) per estrarre alcuni dati dal rendering dell'app, che presto conterrà alcuni component di tipo `<Helmet />`. In questi component `<Helmet />` è dove impostiamo il `title` ed altri dettagli dell'`head` per ogni pagina. Nota che `Helmet.rewind()` *deve* venire dopo `ReactDOMServer.renderToString()`.

- Modifica `src/shared/app.jsx` in questo modo:

```js
import Helmet from 'react-helmet'
// [...]
const App = () =>
  <div>
    <Helmet titleTemplate={`%s | ${APP_NAME}`} defaultTitle={APP_NAME} />
    <Nav />
    // [...]
```

- Modifica `src/shared/component/page/home.jsx` in questo modo:

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

- Modifica `src/shared/component/page/hello.jsx` in questo modo:

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

- Modifica `src/shared/component/page/hello-async.jsx` in questo modo:

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

- Modifica `src/shared/component/page/not-found.jsx` in questo modo:

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

Il component `<Helmet>` non esegue il render di niente, semplicemente inietta delle informazioni nell'`head` del documento ed espone gli stessi dati al server.

🏁 Esegui `yarn start` e `yarn dev:wds` e naviga tra le pagine. Il titolo della scheda dovrebbe cambiare quando navighi, e dovrebbe rimanere costante quando ricarichi la pagina. guarda il sorgente della pagina per vedere come React Helmet imposta i tag `title` e `meta` anche nel rendering effettuato sul server.

Prossimo capitolo: [07 - Socket.IO](07-socket-io.md#readme)

Torna al [capitolo precedente](05-redux-immutable-fetch.md#readme)  o all'[indice dei contenuti](https://github.com/fbertone/guida-javascript-moderno#indice-dei-contenuti).
