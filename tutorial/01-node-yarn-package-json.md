# 01 - Node, Yarn, e `package.json`

Il codice per questo capitolo è disponibile [qua](https://github.com/verekia/js-stack-walkthrough/tree/master/01-node-yarn-package-json).

In questa sezione configureremo Node, Yarn, un file `package.json` di base, e proveremo un package.

## Node

> 💡 **[Node.js](https://nodejs.org/)** è un ambiente di runtime per JavaScript. Viene utilizzato principalmente per lo sviluppo di Back-End, ma anche come ambiente di scripting in generale. Nel contesto dello sviluppo di Front-End, può essere utilizzato per l'esecuzione di tutta una serie di task, come il linting, esecuzione di test, e manipolazione dei file.

Utilizzeremo Node praticamente per tutto il tutorial, quindi ne avrai bisogno. Vai alla pagina di [download](https://nodejs.org/en/download/current/) per l'installazione in **macOS** o **Windows**, o alla pagina di [installazione tramite package manager](https://nodejs.org/en/download/package-manager/) per le distribuzioni Linux.

Ad esempio, in **Ubuntu / Debian**, eseguiresti questi comandi per installare  Node:

```sh
curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
sudo apt-get install -y nodejs
```

Ti serve una versione di Node > 6.5.0.

## NVM

Se Node è già installato, o se vuoi maggiore flessibilità, installa NVM ([Node Version Manager](https://github.com/creationix/nvm)), e fai in modo che NVM installi e configuri l'ultima versione di Node per te.

## NPM

NPM è il package manager di default per Node. Viene automaticamente installato insieme a Node. I package manager sono utilizzati per installare e gestire i pacchetti (moduli di codice scritti da te o da altri). Utilizzeremo molti pacchetti in questo tutorial, ma ci serviremo di Yarn, un package manager differente.

## Yarn

> 💡 **[Yarn](https://yarnpkg.com/)** è un package manager per Node.js che è molto più veloce di NPM, ha il supporto offline, e gestisce le dipendenza in modo più [predicibile](https://yarnpkg.com/en/docs/yarn-lock).

Da quando è stato [rilasciato](https://code.facebook.com/posts/1840075619545360) ad ottobre 2016, è stato adottato molto rapidamente e potrebbe diventare presto il package manager di riferimento per la comunità JavaScript. Se preferisci rimanere con NPM puoi semplicemente sostituire tutti i comandi `yarn add` e `yarn add --dev` con `npm install --save` e `npm install --save-dev`.

Installa Yarn seguendo le [istruzioni](https://yarnpkg.com/en/docs/install) per il tuo OS. Se utilizzi macOS o Unix, ti consiglio di usare lo script **Installation Script** che trovi nella scheda *Alternatives* per [evitare](https://github.com/yarnpkg/yarn/issues/1505) di essere dipendente da altri package manager:

```sh
curl -o- -L https://yarnpkg.com/install.sh | bash
```

## `package.json`

> 💡 **[package.json](https://yarnpkg.com/en/docs/package-json)** è il file che serve a descrivere e configurare i progetti JavaScript. Contiene delle informazioni generali (il nome del progetto, la versione, i contributor, la licenza, etc), opzioni di configurazione dei tool che usi, e anche una sezione per l'esecuzione di *task*.

- Crea una cartella di progetto ed entraci da console (`cd`).
- Esegui `yarn init` e rispondi alle domande (`yarn init -y` per saltare tutte le domande), per generare automaticamente il file `package.json`.

Ecco il contenuto del file `package.json` che useremo per questo tutorial:

```json
{
  "name": "your-project",
  "version": "1.0.0",
  "license": "MIT"
}
```

## Hello World

- Crea un file `index.js` contenente `console.log('Hello world')`

🏁 Esegui `node .` in questa cartella (`index.js` è il file che Node cerca di default per l'esecuzione). Dovrebbe scrivere "Hello world".

**Nota**: Vedi quella bandierina 🏁 ? La inserirò ogni volta che raggiungi un **checkpoint**. A volte faremo molte modifiche di seguito, e il codice potrebbe non funzionare fino al raggiungimento del checkpoint successivo.

## `start` script

Scrivere `node .` per eseguire il nostro programma è un po' troppo di basso livello. Utilizzeremo uno script NPM/Yarn per far partire l'esecuzione del codice. Questo ci permetterà di avere un buon livello di astrazione e poter sempre eseguire `yarn start`, anche quando il nostro programma diventerà più complesso.

- All'interno di `package.json`, aggiungi un oggetto `scripts` in questo modo:

```json
{
  "name": "your-project",
  "version": "1.0.0",
  "license": "MIT",
  "scripts": {
    "start": "node ."
  }
}
```

`start` è il nome che diamo al *task* che farà partire il nostro programma. Creeremo molti task differenti in questo oggetto `scripts` nel corso di questo tutorial. `start` tipicamente è il nome che viene dato al task di default. Altri nomi di task standard sono `stop` e `test`.

`package.json` deve essere un file JSON valido, il che significa che non può avere delle virgole finali. Devi quindi fare attenzione quando modifichi a mano il file `package.json`.

🏁 Lancia `yarn start`. Dovrebbe venire scritto `Hello world`.

## Git e `.gitignore`

- Inizializza un repository Git con `git init`

- Crea un file `.gitignore` ed inserisci al suo interno quanto segue:

```gitignore
.DS_Store
/*.log
```

I file `.DS_Store` sono dei file di macOS che vengono generati automaticamente e non dovresti mai avere inclusi nel tuo repository.

`npm-debug.log` e `yarn-error.log` sono dei file che vengono creati quando il package manager riscontra un errore, non li vogliamo includere nel repository.

## Installare ed utilizzare un package

In questa sezione vedremo come installare ed utilizzare un package. Un "package" è semplicemente del codice scritto da altri, che puoi riutilizzare nel tuo codice. Può essere qualunque cosa. Adesso ad esempio utilizzeremo un package che può aiutarti ad utilizzare i codici dei colori.

- Installa il package chiamato `color` attraverso il comando `yarn add color`

Apri `package.json` per vedere come Yarn ha automaticamente aggiunto `color` nelle `dependencies`.

Una cartella `node_modules` è stata creata per contenere il package.

- Aggiungi `node_modules/` al file `.gitignore`

Noterai anche che un file `yarn.lock` è stato generato da Yarn. Dovresti includere questo file nel repository, perchè farà in modo da assicurare che ogni persona del tuo team utilizzi le stesse versioni dei package. Se stai utilizzando NPM invece di Yarn, l'equivalente di questo file si chiama *shrinkwrap*.

- Scrivi quanto segue nel file `index.js`:

```js
const color = require('color')

const redHexa = color({ r: 255, g: 0, b: 0 }).hex()

console.log(redHexa)
```

🏁 Esegui `yarn start`. Dovrebbe scrivere `#FF0000`.

Congratulazioni, hai installato ed utilizzato un package!

`color` è stato utilizzato in questa sezione unicamente per farti vedere come puoi utilizzare un semplice package. Non lo utilizzeremo più in seguito, puoi quindi disintallarlo:

- Esegui `yarn remove color`

## Due tipi di dipendenze

Ci sono due tipi di dipendenze di package, `"dependencies"` e `"devDependencies"`:

**Dependencies** sono librerie che sono necessarie per il funzionamento dell'applicazione (React, Redux, Lodash, jQuery, etc). Le installi con il comando `yarn add [package]`.

**Dev Dependencies** sono librerie utilizzate durante lo sviluppo o per fare il build dell'applicazione (Webpack, SASS, linter, framework di testing, etc). Queste vanno installate utilizzando il comando `yarn add --dev [package]`.

Prossimo capitolo: [02 - Babel, ES6, ESLint, Flow, Jest, Husky](02-babel-es6-eslint-flow-jest-husky.md#readme)

Torna all'[indice dei contenuti](https://github.com/fbertone/guida-javascript-moderno#indice-dei-contenuti).
