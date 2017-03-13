# 03 - Express, Nodemon, e PM2

Il codice per questo capitolo è disponibile [qua](https://github.com/verekia/js-stack-walkthrough/tree/master/03-express-nodemon-pm2).

In questa sezione creeremo il server che si occuperà del render della nostra web app. Configureremo inoltre per il server una modalità di sviluppo ed una di produzione.

## Express

> 💡 **[Express](http://expressjs.com/)** è di gran lunga il framework web più famoso per Node. Dispone di un'API molto semplice e minimale, e le sue funzionalità possono essere estese tramite *middleware*.

Configuriamo un server Express per servire una pagina HTML page con del CSS.

- Cancella tutto in `src`

Crea i seguenti file e cartelle:

- Crea un file `public/css/style.css` con questo contenuto:

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

- Crea una cartella vuota `src/client/`.

- Crea una cartella vuota `src/shared/`.

Questa cartella è dove inseriremo il codice Javascript *isomorfico / universale* – file che verranno utilizzati sia dal client che dal server. Un buon esempio di codice condiviso sono le *routes*, come vedrai più avanti quando faremo delle chiamate asincrone. Per il momento impostiamo solo alcune constanti di configurazione come esempio.

- Crea un file `src/shared/config.js` contenente:

```js
// @flow

export const WEB_PORT = process.env.PORT || 8000
export const STATIC_PATH = '/static'
export const APP_NAME = 'Hello App'
```

Se il processo di Node utilizzato per eseguire la tua applicazione ha una variabile d'ambiente `process.env.PORT` impostata (ad esempio quando fai il deploy su Heroku), utilizzarà il valore di questa variabile come porta. Se non è configurata, utilizziamo come default la `8000`.

- Crea il file `src/shared/util.js` contenente:

```js
// @flow

// eslint-disable-next-line import/prefer-default-export
export const isProd = process.env.NODE_ENV === 'production'
```

Questo è un modo semplice per verificare se siamo in produzione o no. Il commento `// eslint-disable-next-line import/prefer-default-export` ci serve perchè abbiamo un solo export in questo caso. Potrai rimuoverlo quando aggiungerai nuove export in questo file.

- Esegui `yarn add express compression`

`compression` è un middleware di Express per attivare la compressione Gzip sul server.

- Crea il file `src/server/index.js` contenente:

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

Niente di particolare qua, è quasi il tutorial "Hello World" di Express con alcune import aggiuntive. Stiamo utilizzando 2 directory differenti per i file statici. `dist` per i file generati, `public` per quelli fissi.

- Crea il file `src/server/render-app.js` contenente:

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

Hai presente quando si utilizzavano i *template* sul back-end? Beh, adesso sono diventati abbastanza obsoleti con l'introduzione delle stringhe template in Javascript. Abbiamo una funzione che prende `title` come parametro e lo inietta nei tag `title` e `h1` della pagina, ritornando l'HTML completo. Usiamo anche la costante `STATIC_PATH` come percorso di base per tutte le risorse statiche.

### Sintax highlighting delle stringhe template HTML in Atom (opzionale)

Potrebbe essere possibile attivare la syntax highlighting per il codice HTML presente all'interno delle stringhe template in base al tuo editor. In Atom, se inserisci in tag `html` come prefisso in una stringa template (o un qualsiasi tag che *termina* con `html`, come `ilovehtml`), verrà automaticamente attivato l'highlight del contenuto di quella stringa. A volte uso il tag `html` della libreria `common-tags` per utilizzare questa funzione:

```js
import { html } from `common-tags`

const template = html`
<div>Wow, colors!</div>
`
```

Non ho inserito questa tecnica nel boilerplate di questo tutorial siccome funziona solo in Atom, e non è proprio l'ideale. Alcuni utenti di Atom potrebbero comunque trovarlo utile.

Comunque, torniamo al lavoro!

- In `package.json` modifica il script `start` in questo modo: `"start": "babel-node src/server",`

🏁 Esegui `yarn start`, e apri `localhost:8000` nel browser. Se tutto funziona correttamente dovresti vedere una pagina bianca con scritto "Hello App" sia come titolo della scheda del browser che in verde come titolo della pagina.

**Nota**: Alcuni processi – tipicamente processi che attendono l'avvenimento di eventi, com un'istanza di un server – ti impediscono di inserire comandi nel terminale fino a quando non hanno terminato l'esecusione. Per chiuderli e tornare alla linea di comando, premi **Ctrl+C**. Alternativamente puoi aprire un altro terminale, in questo modo potrai lanciare comandi mentre l'altro processo è ancora in esecuzione. Puoi anche mandare i processi in background, ma questo è fuori dal contesto del tutorial.

## Nodemon

> 💡 **[Nodemon](https://nodemon.io/)** è un'utility che permette di riavviare automaricamente il server Node quando vengono modificati dei file all'interno della cartella.

Useremo Nodemon quando siamo in modalità **sviluppo**.

- Lancia `yarn add --dev nodemon`

- Modifica `scripts` in questo modo:

```json
"start": "yarn dev:start",
"dev:start": "nodemon --ignore lib --exec babel-node src/server",
```

`start` è adesso semplicemente un puntatore ad un altro task, `dev:start`. Questo ci permette di avere un livello di astrazione rispetto al task di default da eseguire con il comando start.

In `dev:start`, il parametro `--ignore lib` serve a *non* riavviare il server quando avvengono delle modifiche all'interno della cartella `lib`. Per il momento questa cartella non esiste ancora, ma la creeremo nella prossima sezione di questo capitolo, avrà quindi presto significato. Nodemon tipicamente utilizza l'eseguibile `node`. Nel nostro caso, siccome utilizziamo Babel, possiamo dire a Nodemon di utilizzare invece il comando `babel-node`. In questo modo continuerà ad interpretare tutto il codice ES6/Flow.

🏁 Lancia `yarn start` e apri `localhost:8000`. Modifica la costante `APP_NAME` in `src/shared/config.js`, che dovrebbe far riavviare il processo del server. Aggiorna la pagina per vedere il nuovo titolo. Nota che questo riavvio automatico del server è differente dalla funzionalità di *Hot Module Replacement* (sostituzione dei moduli a caldo), che è quando i componenti della pagina vengono aggiornati in tempo reale. In questo caso dobbiamo continuare ad aggiornare la pagina a mano, ma almeno non dobbiamo preoccuparci di terminare il server e rilanciarlo. L'Hot Module Replacement sarà introdotto del prossimo capitolo.

## PM2

> 💡 **[PM2](http://pm2.keymetrics.io/)** è un Process Manager (gestore di processi) per Node. Mantiene i tuoi processi attivi in produzione, e offre molte funzionalità per gestirli e monitorarli.

Useremo PM2 in modalità **produzione**.

- Esegui `yarn add --dev pm2`

In produzione, vuoi che il server sia il più performante possibile. `babel-node` lancia l'intero processo di transpilazione di Babel transpilation ad ogni esecuzione, il che non è qualcosa desiderabile in produzione. Abbiamo bisogno che Babel faccia tutto il lavoro in anticipo, e il nostro server utilizzi direttamente i file codificati in ES5.

Una delle funzionalità principali di Babel è di prendere una intera cartella di codice ES6 (tipicamente chiamata `src`) e transpilarla in una cartella contenente codice ES5 (tipicamente chiamata `lib`).

Essendo questa cartella `lib` generata automaticamente, è una buona pratica ripulirla automaticamente prima di una nuova compilazione, siccome potrebbe contenere dei vecchi file non più necessari. Un semplice package per cancellare file con supporto multipiattaforma è `rimraf`.

- Esegui `yarn add --dev rimraf`

Aggiungiamo il seguente task `prod:build` all'interno del `package.json`:

```json
"prod:build": "rimraf lib && babel src -d lib --ignore .test.js",
```

- Esegui `yarn prod:build`, dovrebbe generare una cartella `lib` contenente il codice transpilato, esclusi i file che terminano per `.test.js` (nota che anche i file `.test.jsx` vengono ignorati quando si utilizza questo parametro).

- Aggiungi `/lib/` al tuo `.gitignore`

Un'ultima cosa: passeremo una variabile d'ambiente `NODE_ENV` all'eseguibile di PM2. In Unix, faresti questo utilizzando la sintassi `NODE_ENV=production pm2`, ma in Windows è differente. Utilizzeremo un piccolo package chiamato `cross-env` per fare in modo che questa sintassi funzioni anche in Windows.

- Lancia `yarn add --dev cross-env`

Aggiorniamo `package.json` in questo modo:

```json
"scripts": {
  "start": "yarn dev:start",
  "dev:start": "nodemon --ignore lib --exec babel-node src/server",
  "prod:build": "rimraf lib && babel src -d lib --ignore .test.js",
  "prod:start": "cross-env NODE_ENV=production pm2 start lib/server && pm2 logs",
  "prod:stop": "pm2 delete server",
  "test": "eslint src && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test"
},
```

🏁 Lancia `yarn prod:build`, poi esegui `yarn prod:start`. PM2 dovrebbe mostrare un processo attivo. Vai su `http://localhost:8000/` nel tuo browser e dovresti vedere la tua app. Il terminale dovrebbe mostrare il log, che dovrebbe essere "Server running on port 8000 (production).". Nota che con PM2, i tuoi processi vengono eseguiti in background. Se premi Ctrl+C, terminerai il comando `pm2 logs`, che era il nostro ultimo comando nella catena di `prod:start`, ma il server dovrebbe continuare a funzionare. Se vuoi chiudere il server, esegui `yarn prod:stop`

Adesso che abbiamo un task `prod:build`, sarebbe ottimale fare in modo che venga verificato il corretto funzionamento del codice prima di eseguire la push nel repository. Siccome è probabilmente inutile eseguire questa verifica ad ogni commit, suggerisco di aggiungerla nel task `prepush`:

```json
"prepush": "yarn test && yarn prod:build"
```

🏁 Esegui `yarn prepush` o semplicemente lancia la push per far partire il processo di test.

**Nota**: Non abbiamo nessun test qua, quindi Jest si lamenterà un po'. Ignoralo per il momento.

Prossimo capitolo: [04 - Webpack, React, HMR](04-webpack-react-hmr.md#readme)

Torna al [capitolo precedente](02-babel-es6-eslint-flow-jest-husky.md#readme)  o all'[indice dei contenuti](https://github.com/fbertone/guida-javascript-moderno#indice-dei-contenuti).
