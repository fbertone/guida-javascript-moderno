# 08 - Bootstrap e JSS

Il codice di questo capitolo è disponibile nel branch [`master-no-services`](https://github.com/verekia/js-stack-boilerplate/tree/master-no-services) del repository [JS-Stack-Boilerplate](https://github.com/verekia/js-stack-boilerplate).

Ok! È arrivata l'ora di fare qualche ritocco estetico alla nostra app bruttina. Utilizzeremo Twitter Bootstrap per dargli qualche stile di base. Successivamente tramite una libreria CSS-in-JS aggiungeremo qualche stile personalizzato.

## Twitter Bootstrap

> 💡 **[Twitter Bootstrap](http://getbootstrap.com/)** è una libreria di componenti per UI.

Ci sono 2 possibilità per integrare Bootstrap in un'app React. Entrambe hanno dei pro e dei contro:

- Utilizzando la versione ufficiale, **che usa jQuery e Tether** per il comportamento dei suoi componenti.
- Utilizzando una libreria di terze parti ch re-implementa tutti i componenti di Bootstrap in React, come [React-Bootstrap](https://react-bootstrap.github.io/) oppure [Reactstrap](https://reactstrap.github.io/).

Le librerie terze forniscono dei component React molto comodi che riducono notevolmente il codice inutile rispetto ai componenti HTML originali, e si integrano in modo ottimale con il tuo codebase React. Detto ciò, devo dire che io sono abbastanza restio ad utilizzarle, perchè saranno sempre *indietro* rispetto alle versioni ufficiali (a volte potenzialmente molto indietro). Inoltre non funzionano con i temi Bootstrap che implementano le loro funzionalità JS. Questo è uno svantaggio abbastanza pesante considerando che uno dei maggiori punti di fornza di Bootstrap è la sua grande comunità di designer che costruiscono degli ottimi temi.

Per questa ragione, utilizzerò il compromesso di integrare la versione ufficiale, insieme a jQuery e Tether. Una delle preoccupazioni di questo approccio riguarda ovviamente la dimensione dei file risultanti. Per tua informazione, il bundle occupa circa 200KB (Gzip) con jQuery, Tether, ed i JS di Bootstrap inclusi. Penso che sia ragionevole, ma se per te è troppo, ti conviene considerare le altre opzioni per integrare Bootstrap, o addirittura non utilizzare proprio Bootstrap.

### I CSS di Bootstrap

- Cancella `public/css/style.css`

- Esegui `yarn add bootstrap@4.0.0-alpha.6`

- Copia `bootstrap.min.css` e `bootstrap.min.css.map` da `node_modules/bootstrap/dist/css` nella tua cartella `public/css`.

- Modifica `src/server/render-app.jsx` in questo modo:

```html
<link rel="stylesheet" href="${STATIC_PATH}/css/bootstrap.min.css">
```

### I JS di Bootstrap con jQuery e Tether

Adesso che abbiamo gli stili di Bootstrap caricati nella nostra pagina, abbiamo bisogno dei JavaScript per il comportamento dei componenti.

- Esegui `yarn add jquery tether`

- Modifica `src/client/index.jsx` in questo modo:

```js
import $ from 'jquery'
import Tether from 'tether'

// [right after all your imports]

window.jQuery = $
window.Tether = Tether
require('bootstrap')
```

Questo caricherà il codice Javascript di Bootstrap.

### I componenti di Bootstrap

Bene, è arrivato il momento di fare il copia-incolla di un sacco di file.

- Modifica `src/shared/component/page/hello-async.jsx` in questo modo:

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import MessageAsync from '../../container/message-async'
import HelloAsyncButton from '../../container/hello-async-button'

const title = 'Async Hello Page'

const HelloAsyncPage = () =>
  <div className="container mt-4">
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello asynchronously' },
        { property: 'og:title', content: title },
      ]}
    />
    <div className="row">
      <div className="col-12">
        <h1>{title}</h1>
        <MessageAsync />
        <HelloAsyncButton />
      </div>
    </div>
  </div>

export default HelloAsyncPage
```

- Edit `src/shared/component/page/hello.jsx` like so:

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import Message from '../../container/message'
import HelloButton from '../../container/hello-button'

const title = 'Hello Page'

const HelloPage = () =>
  <div className="container mt-4">
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello' },
        { property: 'og:title', content: title },
      ]}
    />
    <div className="row">
      <div className="col-12">
        <h1>{title}</h1>
        <Message />
        <HelloButton />
      </div>
    </div>
  </div>

export default HelloPage
```

- Modifica `src/shared/component/page/home.jsx` in questo modo:

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import ModalExample from '../modal-example'
import { APP_NAME } from '../../config'

const HomePage = () =>
  <div>
    <Helmet
      meta={[
        { name: 'description', content: 'Hello App is an app to say hello' },
        { property: 'og:title', content: APP_NAME },
      ]}
    />
    <div className="jumbotron">
      <div className="container">
        <h1 className="display-3 mb-4">{APP_NAME}</h1>
      </div>
    </div>
    <div className="container">
      <div className="row">
        <div className="col-md-4 mb-4">
          <h3 className="mb-3">Bootstrap</h3>
          <p>
            <button type="button" role="button" data-toggle="modal" data-target=".js-modal-example" className="btn btn-primary">Open Modal</button>
          </p>
        </div>
        <div className="col-md-4 mb-4">
          <h3 className="mb-3">JSS (soon)</h3>
        </div>
        <div className="col-md-4 mb-4">
          <h3 className="mb-3">Websockets</h3>
          <p>Open your browser console.</p>
        </div>
      </div>
    </div>
    <ModalExample />
  </div>

export default HomePage
```

- Modifica `src/shared/component/page/not-found.jsx` in questo modo:

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'
import { Link } from 'react-router-dom'
import { HOME_PAGE_ROUTE } from '../../routes'

const title = 'Page Not Found!'

const NotFoundPage = () =>
  <div className="container mt-4">
    <Helmet title={title} />
    <div className="row">
      <div className="col-12">
        <h1>{title}</h1>
        <div><Link to={HOME_PAGE_ROUTE}>Go to the homepage</Link>.</div>
      </div>
    </div>
  </div>

export default NotFoundPage
```

- Modifica `src/shared/component/button.jsx` in questo modo:

```js
// [...]
<button
  onClick={handleClick}
  className="btn btn-primary"
  type="button"
  role="button"
>{label}</button>
// [...]
```

- Crea il file `src/shared/component/footer.jsx` contenente:

```js
// @flow

import React from 'react'
import { APP_NAME } from '../config'

const Footer = () =>
  <div className="container mt-5">
    <hr />
    <footer>
      <p>© {APP_NAME} 2017</p>
    </footer>
  </div>

export default Footer
```

- Crea il file `src/shared/component/modal-example.jsx` contenente:

```js
// @flow

import React from 'react'

const ModalExample = () =>
  <div className="js-modal-example modal fade">
    <div className="modal-dialog">
      <div className="modal-content">
        <div className="modal-header">
          <h5 className="modal-title">Modal title</h5>
          <button type="button" className="close" data-dismiss="modal">×</button>
        </div>
        <div className="modal-body">
          This is a Bootstrap modal. It uses jQuery.
        </div>
        <div className="modal-footer">
          <button type="button" role="button" className="btn btn-primary" data-dismiss="modal">Close</button>
        </div>
      </div>
    </div>
  </div>

export default ModalExample
```

- Modifica `src/shared/app.jsx` in questo modo:

```js
const App = () =>
  <div style={{ paddingTop: 54 }}>
```

Questo è un esempio di *stile React inline*.

Questo verrà convertito in: `<div style="padding-top:54px;">` nel DOM. Abbiamo bisogno di questo stile per spingere il contenuto sotto la barra di navigazione. Gli [stili React inline](https://speakerdeck.com/vjeux/react-css-in-js) sono un ottimo modo per isolare gli stili dei tuoi component dal namspace CSS globale, ma ha un costo: non puoi utilizzare alcune funzionalità native del CSS come `:hover`, le Media Queries, le animazioni, o `font-face`. Questa è [una delle ragioni](https://github.com/cssinjs/jss/blob/master/docs/benefits.md#compared-to-inline-styles) per cui integreremo la libreria CSS-in-JS, JSS, più avanti in questo capitolo.

- Modifica `src/shared/component/nav.jsx` in questo modo:

```js
// @flow

import $ from 'jquery'
import React from 'react'
import { Link, NavLink } from 'react-router-dom'
import { APP_NAME } from '../config'
import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
  NOT_FOUND_DEMO_PAGE_ROUTE,
} from '../routes'

const handleNavLinkClick = () => {
  $('body').scrollTop(0)
  $('.js-navbar-collapse').collapse('hide')
}

const Nav = () =>
  <nav className="navbar navbar-toggleable-md navbar-inverse fixed-top bg-inverse">
    <button className="navbar-toggler navbar-toggler-right" type="button" role="button" data-toggle="collapse" data-target=".js-navbar-collapse">
      <span className="navbar-toggler-icon" />
    </button>
    <Link to={HOME_PAGE_ROUTE} className="navbar-brand">{APP_NAME}</Link>
    <div className="js-navbar-collapse collapse navbar-collapse">
      <ul className="navbar-nav mr-auto">
        {[
          { route: HOME_PAGE_ROUTE, label: 'Home' },
          { route: HELLO_PAGE_ROUTE, label: 'Say Hello' },
          { route: HELLO_ASYNC_PAGE_ROUTE, label: 'Say Hello Asynchronously' },
          { route: NOT_FOUND_DEMO_PAGE_ROUTE, label: '404 Demo' },
        ].map(link => (
          <li className="nav-item" key={link.route}>
            <NavLink to={link.route} className="nav-link" activeStyle={{ color: 'white' }} exact onClick={handleNavLinkClick}>{link.label}</NavLink>
          </li>
        ))}
      </ul>
    </div>
  </nav>

export default Nav
```

Qua abbiamo qualcosa di nuovo, `handleNavLinkClick`. Un problema che ho riscontrato utilizzando la `navbar` di Bootstrap's in una SPA è che cliccando su un link via mobile non richiude il menu, e non ritorna in cima alla pagina. Questa è una buona opportunità per fare un esempio di come puoi integrare del codice jQuery / specifico per Bootstrap nella tua app:

```js
import $ from 'jquery'
// [...]

const handleNavLinkClick = () => {
  $('body').scrollTop(0)
  $('.js-navbar-collapse').collapse('hide')
}

<NavLink /* [...] */ onClick={handleNavLinkClick}>
```

**Note**: ho rimosso gli attributi di accessibilità (come gli attributi `aria`) per rendere il codice più leggibile *inel contesto di questo tutorial*. **Dovresti assolutamente reintegrarli**. Fai riferimento alla documentazione di Bootstrap ed agli esempi di codice per impararne il funzionamento.

🏁 La tua app dovrebbe adesso essere completamente stilizzata tramite Bootstrap.

## Lo stato attuale di CSS

Nel 2016, il tipico stack JavaScript si è stabilizzato. Le librerie ed i tool utilizzati in questo tutorial sono di fatto *gli standard più avanzati* (*cough – anche se questo potrebbe diventare superato da qui ad un anno – cough*). Si, è uno stack complesso da configurare, ma la maggiorparte degli sviluppatori front-end concordano che React-Redux-Webpack sia la strada da percorrere. Riguardo al CSS, ho alcune brutte notizie. Non c'è niente di definitivo, nessuno stack standard.

SASS, BEM, SMACSS, SUIT, Bass CSS, React Inline Styles, LESS, Styled Components, CSSX, JSS, Radium, Web Components, CSS Modules, OOCSS, Tachyons, Stylus, Atomic CSS, PostCSS, Aphrodite, React Native for Web, e molti altri che non ho citato sono tutti approcci differenti o tool per fare lo stesso lavoro. Tutti lo fanno bene, e questo è il problema, non esiste un vincitore univoco, è tutto molto confuso.

Gli sviluppatori React "alla moda", tendono a favorire gli stili React inline, CSS-in-JS, o approcci CSS modulari, siccome si integrano molto bene con React e risolvono molti [problemi](https://speakerdeck.com/vjeux/react-css-in-js) che vengono riscontrati con i CSS regolari.

I moduli CSS Modules funzionano bene, ma non sfruttano il potere di JavaScript e tutte le sue funzionalità riguardo CSS. Forniscono solo l'incapsulamento, che va bene, ma gli stili React inline e CSS-in-JS portano l'utilizzo degli stili ad un altro livello secondo me. Il mio suggerimento personale è di utilizzare gli stili React inline per gli stili comuni (devi usarli anche per React Native), e usare una libreria CSS-in-JS per cose come `:hover` e le media queries.

Esistono [numerose librerie CSS-in-JS](https://github.com/MicheleBertoli/css-in-js). JSS è una di quelle più complete e [performanti](https://github.com/cssinjs/jss/blob/master/docs/performance.md).

## JSS

> 💡 **[JSS](http://cssinjs.org/)** è una libreria di tipo CSS-in-JS per scrivere i tuoi stili in JavaScript ed iniettarli nella tua app.

Adesso che abbiamo qualche template di base con Bootstrap, scriviamo qualche CSS personalizzato. Ho affermato in precedenza che gli stili React inline non supportano `:hover` e le media queries, vediamo un semplice esempio utilizzando JSS. JSS può essere utilizzato tramite `react-jss`, una libreria comoda da utilizzare con i component di React.

- Esegui `yarn add react-jss`

Aggiungi quanto segue al file `.flowconfig` perchè attualmente c'è un [problema](https://github.com/cssinjs/jss/issues/411) con Flow e JSS:

```flowconfig
[ignore]
.*/node_modules/jss/.*
```

### Server-side

JSS può effettuare il render degli stili lato server per il rendering iniziale.

- Aggiungi le seguenti costanti in `src/shared/config.js`:

```js
export const JSS_SSR_CLASS = 'jss-ssr'
export const JSS_SSR_SELECTOR = `.${JSS_SSR_CLASS}`
```

- Modifica `src/server/render-app.jsx` in questo modo:

```js
import { SheetsRegistry, SheetsRegistryProvider } from 'react-jss'
// [...]
import { APP_CONTAINER_CLASS, JSS_SSR_CLASS, STATIC_PATH, WDS_PORT } from '../shared/config'
// [...]
const renderApp = (location: string, plainPartialState: ?Object, routerContext: ?Object = {}) => {
  const store = initStore(plainPartialState)
  const sheets = new SheetsRegistry()
  const appHtml = ReactDOMServer.renderToString(
    <Provider store={store}>
      <StaticRouter location={location} context={routerContext}>
        <SheetsRegistryProvider registry={sheets}>
          <App />
        </SheetsRegistryProvider>
      </StaticRouter>
    </Provider>)
  // [...]
      <link rel="stylesheet" href="${STATIC_PATH}/css/bootstrap.min.css">
      <style class="${JSS_SSR_CLASS}">${sheets.toString()}</style>
  // [...]
```

## Client-side

La prima cosa che il client dovrebbe fare dopo il rendering dell'app lato client, è di rimuovere gli stili JSS generati dal server.

- Aggiungi quanto segue a `src/client/index.jsx` dopo le chiamate `ReactDOM.render` (ad esempio prima di `setUpSocket(store)`):

```js
import { APP_CONTAINER_SELECTOR, JSS_SSR_SELECTOR } from '../shared/config'
// [...]

const jssServerSide = document.querySelector(JSS_SSR_SELECTOR)
// flow-disable-next-line
jssServerSide.parentNode.removeChild(jssServerSide)

setUpSocket(store)
```

Modifica `src/shared/component/page/home.jsx` in questo modo:

```js
import injectSheet from 'react-jss'
// [...]
const styles = {
  hoverMe: {
    '&:hover': {
      color: 'red',
    },
  },
  '@media (max-width: 800px)': {
    resizeMe: {
      color: 'red',
    },
  },
  specialButton: {
    composes: ['btn', 'btn-primary'],
    backgroundColor: 'limegreen',
  },
}

const HomePage = ({ classes }: { classes: Object }) =>
  // [...]
  <div className="col-md-4 mb-4">
    <h3 className="mb-3">JSS</h3>
    <p className={classes.hoverMe}>Hover me.</p>
    <p className={classes.resizeMe}>Resize the window.</p>
    <button className={classes.specialButton}>Composition</button>
  </div>
  // [...]

export default injectSheet(styles)(HomePage)
```

A differenza degli stili React inline, JSS utilizza le classi. Devi passare gli stili a `injectSheet` e le classi CSS finiscono nelle props del tuo component.

🏁 Esegui `yarn start` e `yarn dev:wds`. Apri la homepage. Visualizza il sorgente della pagina (non nell'inspector) per vedere che gli stili JSS sono presenti nel DOM nel render iniziale all'interno dell'elemento `<style class="jss-ssr">` (solo nella Home page). Dovrebbero essere scomparsi nell'inspector, sostituiti da `<style type="text/css" data-jss data-meta="HomePage">`.

**Nota**: In modalità produzione, il `data-meta` è offuscato. Ottimo!

Se sposti il puntatore sopra all'etichetta "Hover me", dovrebbe diventare rossa. Se ridimensioni la finestra del browser per essere più stretta di 800px, l'etichetta "Resize your window" diventa rossa. Il bottone verde estende le classi CSS di Bootstrap usando la proprietà `composes` di JSS.

Prossimo capitolo: [09 - Travis, Coveralls, Heroku](09-travis-coveralls-heroku.md#readme)

Torna al [capitolo precedente](07-socket-io.md#readme) o all'[indice dei contenuti](https://github.com/fbertone/guida-javascript-moderno#indice-dei-contenuti).
