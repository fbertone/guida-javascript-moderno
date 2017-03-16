# 05 - Redux, Immutable, e Fetch

Il codice di questo cap[itolo è disponibile] [QUA](https://github.com/verekia/js-stack-walkthrough/tree/master/05-redux-immutable-fetch).

In questo capitolo inseriremo React e Redux per fare un'app molto semplice. L'app consisterà in un messaggio ed un bottone. Il messaggio cambia quando l'utente preme il bottone.

Prima di cominciare facciamo una piccola introduzione a ImmutableJS, che è completamente scorrelato da React e Redux, ma verrà utilizzato in questo capitolo.

## ImmutableJS

> 💡 **[ImmutableJS](https://facebook.github.io/immutable-js/)** (o semplicemente Immutable) è una libreria sviluppata da Facebook per manipolare collection immutabili, come list e map. Ogni modifica fatta ad un oggetto immutabile ritorna una nuova copia del nuovo oggetto modificato senza alterare l'oggetto originale.

Ad esempio, invece di fare:

```js
const obj = { a: 1 }
obj.a = 2 // Mutates `obj`
```

Faresti:

```js
const obj = Immutable.Map({ a: 1 })
obj.set('a', 2) // Returns a new object without mutating `obj`
```

Questo approccio segue il paradigma della **programmazione funzionale**, che funziona molto bene con Redux.

Quando creiamo delle collection immutabili, un metodo molto utile è `Immutable.fromJS()`, che prende un normale oggetto JS o un array e restituisce una sua versione immutabile:

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

- Esegui `yarn add immutable@4.0.0-rc.2`

## Redux

> 💡 **[Redux](http://redux.js.org/)** iè una libreria per la gestione del lifecycle della tua applicazione. Crea uno *store*, che è la single source of truth dello stato della tua app per un qualsiasi momento.

Iniziamo con la parte semplice, dichiarando le nostre azioni Redux:

- Esegui `yarn add redux redux-actions`

- Crea il file `src/client/action/hello.js` contenente:

```js
// @flow

import { createAction } from 'redux-actions'

export const SAY_HELLO = 'SAY_HELLO'

export const sayHello = createAction(SAY_HELLO)
```

Questo file espone una *action*, `SAY_HELLO`, ed il suo *action creator*, `sayHello`, che è una funzione. Usiamo [`redux-actions`](https://github.com/acdlite/redux-actions) per ridurre il codice ripetitivo legato alle action di Redux. `redux-actions` implementa il modello [Flux Standard Action](https://github.com/acdlite/flux-standard-action), che fa in modo che gli *action creator* ritornino oggetti contenenti gli attributi `type` e `payload`.

- Crea il file `src/client/reducer/hello.js` contenente:

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

In questo file inizializziamo lo stato del nostro reducer con una Immutable Map contenente una proprietà, `message`, impostata e `Initial reducer message`. `helloReducer` gestisce le azioni `SAY_HELLO` semplicemente impostando il nuovo `message` con il payload dell'azione. L'annotazione di Flow per `action` lo suddivide in `type` e `payload`. Il `payload` può essere di qualsiasi (`any`) tipo. Può sembrare strano se non hai mai visto nulla del genere, ma rimane abbastanza comprensibile. Per il tipo dello `state`, usiamo l'istruzione `import type` di Flow per prendere il tipo che ritorna `fromJS`. Lo rinominiamo in `Immut` per chiarezza, perchè `state: fromJS` confonderebbe un po' le idee. La riga `import type` verrà rimossa come qualsiasi annotazione di Flow. Nota l'utilizzo di `Immutable.fromJS()` e `set()` come visto prima.

## React-Redux

> 💡 **[react-redux](https://github.com/reactjs/react-redux)** *connette* uno store di Redux con component di React. Con `react-redux`, quando lo store di Redux cambia, i component di React vengono aggiornati automaticamente. Possono anche lanciare azioni di Redux.

- Esegui `yarn add react-redux`

In questa sezione creeremo dei *Component* e dei *Container*.

I **Component** sono dei componenti di React *dumb*, nel senso che non conoscono nulla dello stato di Redux. I **Container** sono componenti *smart* che conoscono lo stato e saranno collegati ai nostri component dumb.

- Crea il file `src/client/component/button.jsx` contenente:

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

**Nota**: Puoi notare qua un caso di *type alias* di Flow. Definiamo il tipo di `Props` prima di annotare il tipo di `props` del componente.

- Crea il file `src/client/component/message.jsx` contenente:

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

Questi sono esempi di component *dumb*. Sono senza logica, e mostrano qualunque cosa gli venga detto di mostrare tramite le **props** di React. La differenza principale tra `button.jsx` e `message.jsx` è che `Button` contiene un riferimento ad un action dispatcher tra le sue props, mentre `Message` contiene semplicemente delle informazioni da visualizzare.

Nuovamente, i *component* non conoscono nulla delle **action** di Redux o dello **stato** della nostra app, per questo motivo creeremo dei  **container** smart che forniranno gli action dispatchers ed i dati appropriati agli altri 2 component dumb.

- Crea il file `src/client/container/hello-button.js` contenente:

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

Questo container collega il component `Button` con l'action `sayHello` ed il metodo `dispatch` di Redux.

- Crea il file `src/client/container/message.js` contenente:

```js
// @flow

import { connect } from 'react-redux'

import Message from '../component/message'

const mapStateToProps = state => ({
  message: state.hello.get('message'),
})

export default connect(mapStateToProps)(Message)
```

Questo container collega lo stato Redux dell'app con il component `Message`. Quando lo stato cambia, `Message` si aggiornerà automaticamente per visualizzare la prop `message` corretta. Queste connesisoni sono effettuate tramite la funzione `connect` di `react-redux`.

- Aggiorna il file `src/client/app.jsx` in questo modo:

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

Non abbiamo ancora inizializzato lo store di Redux e non abbiamo ancora inserito i due container da nessuna parte all'interno della nostra app:

- Modifica `src/client/index.jsx` in questo modo:

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

Rivediamo brevemente cosa abbiamo fatto. Innanzitutto creiamo uno *store* con `createStore`. Gli store sono creati passando i reducer. Qua abbiamo un solo reducer, ma per mantanere la scalabilità, usiamo `combineReducers` per raggruppare assieme tutti i reducer. L'ultimo strano parametro di `createStore` è qualcosa per collegare Redux ai [Devtools](https://github.com/zalmoxisus/redux-devtools-extension) del browser, che sono estremamente utili durante il debugging. Siccome ESLint avrà da ridire sull'underscores in `__REDUX_DEVTOOLS_EXTENSION__`, disabilitiamo questa regola di ESLint. Successivamente rinchiudiamo tutta l'app nel component `Provider` di `react-redux` grazie alla funzione `wrapApp`, passandogli successivamente lo store.

🏁 Adesso puoi eseguire `yarn start` e `yarn dev:wds` e aprire `http://localhost:8000`. Dovresti vedere "Initial reducer message" ed un bottone. Quando clicki il bottone, Il messaggio dovrebbe cambiare in "Hello!". Se hai installato i Redux Devtools nel browser, dovresti vedere i cambiamenti dello stato dell'app quando clicki sul bottone.

Congratulazioni, finalmente abbiamo fatto un'app che fa qualcosa! Okay, per il momento non fa molto effetto vista da fuori, ma sappiamo che internamente è supportata da uno stack molto potente.

## Estendiamo la nostra app con una chiamata asincrona

Adesso aggiungeremo un secondo bottone alla nostra app, che farà partire una chiamata AJAX per recuperare un messaggio dal server. Per dimostrarne il funzionamento questa chiamata ritornerà anche dei dati: il numero `1234`.

### L'endpoint del server

- Crea il file `src/shared/routes.js` contenente:

```js
// @flow

// eslint-disable-next-line import/prefer-default-export
export const helloEndpointRoute = (num: ?number) => `/ajax/hello/${num || ':num'}`
```

Questa funzione serve a produrre quanto segue:

```js
helloEndpointRoute()     // -> '/ajax/hello/:num' (for Express)
helloEndpointRoute(1234) // -> '/ajax/hello/1234' (for the actual call)
```

Creiamo velocemente un test per verificare che funzioni correttamente.

- Crea il file `src/shared/routes.test.js` contenente:

```js
import { helloEndpointRoute } from './routes'

test('helloEndpointRoute', () => {
  expect(helloEndpointRoute()).toBe('/ajax/hello/:num')
  expect(helloEndpointRoute(123)).toBe('/ajax/hello/123')
})
```

- Esegui `yarn test` e dovrebbe passare con successo.

- In `src/server/index.js`, aggiungi:

```js
import { helloEndpointRoute } from '../shared/routes'

// [under app.get('/')...]

app.get(helloEndpointRoute(), (req, res) => {
  res.json({ serverMessage: `Hello from the server! (received ${req.params.num})` })
})
```

### Nuovi container

- Crea il file `src/client/container/hello-async-button.js` contenente:

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

Per dimostrare come puoi passare un parametro alla chiamata asincrona e mantenere le cose semplici, sto utilizzando il valore fisso `1234`. Questo valore tipicamente proverrebbe dal campo di un form compilato dall'utente.

- Crea il file`src/client/container/message-async.js` contenente:

```js
// @flow

import { connect } from 'react-redux'

import MessageAsync from '../component/message'

const mapStateToProps = state => ({
  message: state.hello.get('messageAsync'),
})

export default connect(mapStateToProps)(MessageAsync)
```

Puoi vedere che in questo container, facciamo riferimento ad una proprietà `messageAsync`, che aggiungeremo presto al nostro reducer.

Quello che dobbiamo fare adesso è creare l'azione `sayHelloAsync`.

### Fetch

> 💡 **[Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)** è una funzione JavaScript standardizzata per eseguire chiamate asincrone ispirata dai metodi AJAX di jQuery.

Useremo `fetch` per fare chiamate al server dal client. `fetch` non è per il momento ancora supportata da tutti i browser, avremo quindi nisogno di un polyfill. `isomorphic-fetch` è un polyfill che ne permette il funzionamento cross-browser ed anche in Node!

- Esegui `yarn add isomorphic-fetch`

Siccome stiamo usando `eslint-plugin-compat`, dobbiamo indicare che stiamo usando un polyfill per `fetch`, per non ricevere dei warning a causa del suo utilizzo.

- Aggiungi al file `.eslintrc.json`:

```json
"settings": {
  "polyfills": ["fetch"]
},
```

### 3 action asincrone

`sayHelloAsync` non sarà un'action normale. Le action asinctone sono solitamente suddivise in 3 action, che invocano 3 stati differenti: un'action *request* (o "loading"), un'action *success*, ed un'action *failure*.

- Modifica `src/client/action/hello.js` in questo modo:

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

Invece di ritornare un'action, `sayHelloAsync` ritorna una funzione che lancia la chiamata `fetch`. `fetch` ritorna una `Promise`, della quale richiamiamo action differenti in base allo stato attuale della chiamata asincrona.

### 3 handler di action asincrone

Gestiamo queste action differenti nel file `src/client/reducer/hello.js`:

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

Abbiamo aggiunto un nuovo campo allo store, `messageAsync`, e lo aggiorniamo con messaggi differenti a seconda dell'action che riceviamo. Durante `SAY_HELLO_ASYNC_REQUEST`, visualizziamo `Loading...`. `SAY_HELLO_ASYNC_SUCCESS` aggiorna `messageAsync` similmente per visualizzare il `message` aggiornato di `SAY_HELLO`. `SAY_HELLO_ASYNC_FAILURE` mostra un messaggio di errore.

### Redux-thunk

In `src/client/action/hello.js`, abbiamo inserito `sayHelloAsync`, un generatore di action che ritorna una funzione. Questa non è una funzionalità nativamente supportata da Redux. Per poter effettuare queste action asincrone, dovviamo estendere le funzionalità di Reduc con il *middleware* `redux-thunk`.

- Lancia `yarn add redux-thunk`

- Aggiorna il file `src/client/index.jsx`:

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

Qua passiamo `redux-thunk` alla funzione `applyMiddleware` di Redux. Per fare in modo che i Devtools di Redux continuino a funzionare, dobbiamo anche usare la funzione `compose` di React. Non preoccuparti troppo di questa parte, dicordati solo che abbiamo aumentato le funzionalità di Redux con `redux-thunk`.

- Aggiorna `src/client/app.jsx` così:

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

🏁 Esegui `yarn start` e `yarn dev:wds` e dovresti riuscire a cliccare il bottone "Say hello asynchronously and send 1234" e ricevere un messaggio corrispondente dal server! Siccome stai lavorando in locale, la chiamata è istantanea, ma se apri i Redux Devtools, noterai che ogni cllick ha fatto partire `SAY_HELLO_ASYNC_REQUEST` e `SAY_HELLO_ASYNC_SUCCESS`, facendo passare il messaggio dallo stato `Loading...` come previsto.

Congratulazioni, questa è stata una sezione intensa! Rivediamo il tutto con qualche test.

## Testing

In questa sezione testeremo i nostri reducer e action. Iniziamo dalle action.

Per isolare la logica specifica di `action/hello.js`, avremo bisogno di simulare (mock) le cose che non sono correlate, compresa la `fetch` AJAX che non farà partire una vera richiesta AJAX nei nostri test.

- Esegui `yarn add --dev redux-mock-store fetch-mock`

- Crea il file `src/client/action/hello.test.js` contenente:

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

Bene, diamo un'occhiata a cosa sta succedendo. Prima facciamo un mock dello store Redux tramite `const mockStore = configureMockStore([thunkMiddleware])`. In questo modo possiamo lanciare delle action senza coinvolgere delle logiche di reducer. Per ogni test, simuliamo la `fetch` tramite `fetchMock.get()` facendogli ritornare quello che vogliamo. Quello che testiamo con `expect()` è la sequenza di azioni che vengono lanciate dallo store, grazie alla funzione `store.getActions()` di `redux-mock-store`. Dopo ogni  test ripristiniamo il normale funzionamento della `fetch` con `fetchMock.restore()`.

Testiamo adesso il nostro reducer, che è molto più semplice.

- Crea il file `src/client/reducer/hello.test.js` contenente:

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

Prima di ogni test, inizializziamo `helloState` con il risultato di default del nostro reducer (il case `default` del blocco `switch` nel reducer, che ritorna `initialState`). I test sono molto espliciti, verifichiamo semplicemente che il reducer aggiorni correttamente `message` e `messageAsync` a seconda dell'action ricevuta.

🏁 Esegui `yarn test`. Dovrebbe essere tutto verde.

Prossima sezione: [06 - React Router, Server-Side Rendering, Helmet](06-react-router-ssr-helmet.md#readme)

Torna alla [sezione precedente](04-webpack-react-hmr.md#readme) o all'[indice dei contenuti](https://github.com/fbertone/guida-javascript-moderno#indice-dei-contenuti).
