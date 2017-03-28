# 07 - Socket.IO

Il codice di questo capitolo è disponibile [qua](https://github.com/verekia/js-stack-walkthrough/tree/master/07-socket-io).

> 💡 **[Socket.IO](https://github.com/socketio/socket.io)** è una libreria per utilizzare in modo semplice i Websocket. Fornisce delle comode API e supporta delle soluzioni alternative per i browser che non implementano i nativamente Websocket.

In questo capitolo configureremo un semplice scambio di messaggi tra il client ed il server. Per non aggiungere ulteriori pagine e component – che sarebbero scollegati dalle funzionalità che ci interessano – implementeremo questo scambio di messaggi direttamente nella console del browser. Niente UI per questo capitolo.

- Esegui `yarn add socket.io socket.io-client`

## Server-side

- Modifica `src/server/index.js` in questo modo:

```js
// @flow

import compression from 'compression'
import express from 'express'
import { Server } from 'http'
import socketIO from 'socket.io'

import routing from './routing'
import { WEB_PORT, STATIC_PATH } from '../shared/config'
import { isProd } from '../shared/util'
import setUpSocket from './socket'

const app = express()
// flow-disable-next-line
const http = Server(app)
const io = socketIO(http)
setUpSocket(io)

app.use(compression())
app.use(STATIC_PATH, express.static('dist'))
app.use(STATIC_PATH, express.static('public'))

routing(app)

http.listen(WEB_PORT, () => {
  // eslint-disable-next-line no-console
  console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
    '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
})
```

Nota che per permettere a Socket.IO di funzionare, devi usare `Server` di `http` in modalità `listen` per le richieste in arrivo, e non `app` di Express. Per fortuna non cambia molto nel codice. Tutti i dettagli dei Websocket sono configurati in un file esterno, richiamato da `setUpSocket`.

- Aggiungi le seguenti costanti in `src/shared/config.js`:

```js
export const IO_CONNECT = 'connect'
export const IO_DISCONNECT = 'disconnect'
export const IO_CLIENT_HELLO = 'IO_CLIENT_HELLO'
export const IO_CLIENT_JOIN_ROOM = 'IO_CLIENT_JOIN_ROOM'
export const IO_SERVER_HELLO = 'IO_SERVER_HELLO'
```

Questi sono i *tipi di messaggi* che il client ed il server si scambieranno. Ti consiglio di usare come prefisso `IO_CLIENT` o `IO_SERVER` per rendere chiaro *chi* sta inviando il messaggio. Altrimenti le cose possono farsi parecchio confuse quando hai molti tipi di messaggio.

Come puoi vedere, abbiamo un `IO_CLIENT_JOIN_ROOM`, perchè come dimostrazione faremo in modo che i client si colleghino ad una room (come una chatroom). Le room sono utili per inviare i messaggi in broadcast ad una serie di utenti.

- Crea `src/server/socket.js` contenente:

```js
// @flow

import {
  IO_CONNECT,
  IO_DISCONNECT,
  IO_CLIENT_JOIN_ROOM,
  IO_CLIENT_HELLO,
  IO_SERVER_HELLO,
} from '../shared/config'

/* eslint-disable no-console */
const setUpSocket = (io: Object) => {
  io.on(IO_CONNECT, (socket) => {
    console.log('[socket.io] A client connected.')

    socket.on(IO_CLIENT_JOIN_ROOM, (room) => {
      socket.join(room)
      console.log(`[socket.io] A client joined room ${room}.`)

      io.emit(IO_SERVER_HELLO, 'Hello everyone!')
      io.to(room).emit(IO_SERVER_HELLO, `Hello clients of room ${room}!`)
      socket.emit(IO_SERVER_HELLO, 'Hello you!')
    })

    socket.on(IO_CLIENT_HELLO, (clientMessage) => {
      console.log(`[socket.io] Client: ${clientMessage}`)
    })

    socket.on(IO_DISCONNECT, () => {
      console.log('[socket.io] A client disconnected.')
    })
  })
}
/* eslint-enable no-console */

export default setUpSocket
```

Okay, in questo file abbiamo implementato *come il server dovrebbe reagire quando dei client si collegano o inviano dei messaggi*:

- Quando il client si connette, lo logghiamo nella console del server, ed utilizziamo l'oggetto `socket`, attraverso il quale possiamo comunicare con il client.
- Quando un client invia `IO_CLIENT_JOIN_ROOM`, lo facciamo accedere alla `room` che desidera. Quando si è collegato ad una room, inviamo 3 messaggi: 1 un messaggio ad ogni utente, 1 messaggio agli utenti della room, 1 messaggio solo a quel client.
- Quando il client invia `IO_CLIENT_HELLO`, logghiamo il messaggio nella console del server.
- Quando il client si disconnette, logghiamo anche questo evento.

## Client-side

Il lato client sarà molto simile.

- Modifica `src/client/index.jsx` così:

```js
// [...]
import setUpSocket from './socket'

// [at the very end of the file]
setUpSocket(store)
```

Come puoi vedere, passiamo lo store di Redux a `setUpSocket`. In questo modo quando un messaggio Websocket che arriva dal server deve modificare lo stato Redux del client, possiamo fare il `dispatch` delle azioni. Tuttavia in questo esempio non eseguiremo il `dispatch` di niente.

- Crea `src/client/socket.js` contenente:

```js
// @flow

import socketIOClient from 'socket.io-client'

import {
  IO_CONNECT,
  IO_DISCONNECT,
  IO_CLIENT_HELLO,
  IO_CLIENT_JOIN_ROOM,
  IO_SERVER_HELLO,
  } from '../shared/config'

const socket = socketIOClient(window.location.host)

/* eslint-disable no-console */
// eslint-disable-next-line no-unused-vars
const setUpSocket = (store: Object) => {
  socket.on(IO_CONNECT, () => {
    console.log('[socket.io] Connected.')
    socket.emit(IO_CLIENT_JOIN_ROOM, 'hello-1234')
    socket.emit(IO_CLIENT_HELLO, 'Hello!')
  })

  socket.on(IO_SERVER_HELLO, (serverMessage) => {
    console.log(`[socket.io] Server: ${serverMessage}`)
  })

  socket.on(IO_DISCONNECT, () => {
    console.log('[socket.io] Disconnected.')
  })
}
/* eslint-enable no-console */

export default setUpSocket
```

Quello che succede qua non dovrebbe sorprenderti se hai capido quello che abbiamo fatto nel server:

- Appena il client si collega, lo logghiamo nella console del browser ed entriamo in `hello-1234` tramite un messaggio `IO_CLIENT_JOIN_ROOM`.
- Successivamente inviamo `Hello!` tramite un messaggio `IO_CLIENT_HELLO`.
- Se il server ci invia un messaggio `IO_SERVER_HELLO`, lo logghiamo nella console del browser.
- Logghiamo anche ogni eventuale disconnessione.

🏁 Esegui `yarn start` e `yarn dev:wds`, poi apri `http://localhost:8000`. Apri poi la console del browser, e controlla la console del server Express. Dovresti vedere la comunicazione tramite Websocket tra il client ed il server.

Prossimo capitolo: [08 - Bootstrap, JSS](08-bootstrap-jss.md#readme)

Torna al [capitolo precedente](06-react-router-ssr-helmet.md#readme)  o all'[indice dei contenuti](https://github.com/fbertone/guida-javascript-moderno#indice-dei-contenuti).
