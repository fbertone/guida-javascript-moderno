# 02 - Babel, ES6, ESLint, Flow, Jest, e Husky

Il codice di questo capitolo è disponibile [qua](https://github.com/verekia/js-stack-walkthrough/tree/master/02-babel-es6-eslint-flow-jest-husky).

Andremo ad utilizzare un po' di sintassi ES6, che rappresenta un notevole miglioramento rispetto alla "vecchia" sintassi ES5. Tutti i browser e gli ambienti di esecuzione JS conoscono bene la sintassi ES5, ma non ancora quella ES6. Esiste un tool chiamato Babel che ci viene in soccorso!

## Babel

> 💡 **[Babel](https://babeljs.io/)** è un compilatore che trasforma il codice ES6 (e altre cose come la sintassi JSX di React) in codice ES5. È molto modulare e può essere utilizzato in molti [ambienti](https://babeljs.io/docs/setup/). Si tratta sicuramente del compilatore ES5 preferito dalla comunità React.

- Sposta il file `index.js` in una nuova cartella `src`. Qua è dove inserirai il codice ES6. Rimuovi il vecchio codice che fa riferimento a `color` in `index.js`, e sostituiscilo con:

```js
const str = 'ES6'
console.log(`Hello ${str}`)
```

Stiamo utilizzando una stringa *template* (modello), che è una funzionalità di ES6 che ci permette di iniettare variabili direttamente dentro la stringa senza doverle concatenare, utilizzando il segnaposto `${}`. Nota che le stringhe template sono create utilizzando i **backquotes** (apici inversi).

- Esegui `yarn add --dev babel-cli` per installare l'interfaccia da linea di comando (CLI) per Babel.

La CLI di Babel è composta da [due eseguibili](https://babeljs.io/docs/usage/cli/): `babel`, che compila file ES6 in nuovi file ES5, e `babel-node`, che puoi utilizzare per rimpiazzare la chiamata all'eseguibile `node` ed eseguire file ES6 al volo. `babel-node` è molto utile per lo sviluppo ma è pesante e non adatto all'utilizzo in produzione. In questo capitolo useremo `babel-node` per configurare l'ambiente di sviluppo, e nel prossimo utilizzeremo `babel` per la conversione in file ES5 per la produzione.

- In `package.json`, nello script `start`, sostituisci `node .` con `babel-node src` (`index.js` è il file di default che Node cerca, per questo motivo possiamo omettere di scrivere `index.js`).

Se adesso provi ad eseguire `yarn start`, dovrebbe scrivere l'output corretto, ma Babel di fatto non sta facendo niente. Questo perchè non gli abbiamo dato nessuna indicazione su quali trasformazioni vogliamo farli applicare. L'unico motivo per cui l'output è corretto è perchè Node interpreta nativamente il codice ES6 senza l'aiuto di Babel. Tuttavia in alcuni browser o in vecchie versioni di Node non avremmo successo!

- Esegui `yarn add --dev babel-preset-env` per installare un preset di Babel chiamato `env`, che contiene le configurazioni per le funzionalità ECMAScript più recenti supportate da Babel.

- Crea un file `.babelrc` nella cartella principale del progetto, si tratta di un file JSON per la configurazione di Babel. Inserisci quanto segue al suo interno per permettere a Babel di usare il preset `env`:

```json
{
  "presets": [
    "env"
  ]
}
```

🏁 `yarn start` dovrebbe ancora funzionare, ma stavolta sta facendo veramente qualcosa. Non possiamo veramente sapere se sta funzionando siccome stiamo utilizzando `babel-node` per interpretare il codice ES6 al volo. Avrai presto una prova del fatto che il codice ES6 viene veramente convertito nonappena raggiungerai la sezione sulla [sintassi dei moduli ES6](#la-sintassi-dei-moduli-es6) di questo capitolo.

## ES6

> 💡 **[ES6](http://es6-features.org/)**: L'avanzamento più significativo del linguaggio JavaScript. Ci sono troppe funzionalità di ES6 per elencarle qua, ma del codice ES6 tipicamente usa le classi tramite `class`, `const` e `let`, stringhe template, e arrow function (funzioni freccia) (`(text) => { console.log(text) }`).

### Creazione di una classe ES6

- Crea un nuovo file, `src/dog.js`, contenente la seguente classe ES6:

```js
class Dog {
  constructor(name) {
    this.name = name
  }

  bark() {
    return `Wah wah, I am ${this.name}`
  }
}

module.exports = Dog
```

Non dovrebbe sembrarti estraneo se hai già utilizzato la programmazione ad oggetti in un qualunque linguaggio. In Javascript è una funzionalità introdotta abbastanza recentemente. La classe viene esposta verso l'esterno utilizzando l'assegnazione `module.exports`.

In `src/index.js`, inserisci:

```js
const Dog = require('./dog')

const toby = new Dog('Toby')

console.log(toby.bark())
```

Come puoi vedere, a differenza del package `color` che abbiamo utilizzato in precedenza, quando includiamo uno dei nostri file utilizziamo `./` nella `require()`.

🏁 Lancia `yarn start` e dovrebbe scrivere: "Wah wah, I am Toby".

### La sintassi dei moduli ES6

Sostituiamo semplicemente `const Dog = require('./dog')` con `import Dog from './dog'`, che è la nuova sintassi dei moduli ES6 (a differenza dei moduli "CommonJS"). Attualmente non è ancora nativamente supportata da NodeJS, questa è la prova che Babel staprocessando correttamente i file ES6.

In `dog.js`, sostituiamo `module.exports = Dog` con `export default Dog`

🏁 `yarn start` dovrebbe ancora scrivere "Wah wah, I am Toby".

## ESLint

> 💡 **[ESLint](http://eslint.org)** è il linter di preferenza per il codice ES6. Un linter ti suggerisce come formattare il codice, che forza ad utilizzare uno stile consistente, condiviso con le altre persone del tuo team. È anche un buon modo per imparare JavaScript commettendo degli errori che ESLint ci farà notare.

ESLint funziona tramite delle *regole*, e ce ne sono [molte](http://eslint.org/docs/rules/). Invece di configurare da soli le regole che vogliamo utilizzare per il nostro codice, utilizzeremo la configurazione creata da Airbnb. Questa configurazione utilizza alcuni plugin, dovremo quindi installare anche quelli.

Controlla le [istruzioni](https://www.npmjs.com/package/eslint-config-airbnb) di Airbnb più recenti per installare correttamente il package di configurazione e tutte le sue dipendenze. Al 2017-02-03, dicono di utilizzare il seguente comando nel terminale:

```sh
npm info eslint-config-airbnb@latest peerDependencies --json | command sed 's/[\{\},]//g ; s/: /@/g' | xargs yarn add --dev eslint-config-airbnb@latest
```

Dovrebbe installare tutto quello di cui hai bisogno ed aggiungere automaticamente `eslint-config-airbnb`, `eslint-plugin-import`, `eslint-plugin-jsx-a11y`, e `eslint-plugin-react` al file `package.json`.

**Nota**: Ho sostituito `npm install` con `yarn add` in questo comando. Inoltre, questo comando non funziona in Windows, controlla quindi il file `package.json` in questo repository e installa a mano tutte le dipendenze relative a ESLint, usando il comando `yarn add --dev packagename@^#.#.#` dove `#.#.#` sono i numeri di versione che trovi in `package.json` per ogni package.

- Crea un file `.eslintrc.json` nella cartella principale del tuo progetto, come abbiamo fatto in precedenza per Babel, e inserisci quanto segue:

```json
{
  "extends": "airbnb"
}
```

Creeremo uno script di NPM/Yarn per eseguire ESLint. Installiamo il package `eslint` per poter utilizzare la CLI di `eslint`:

- Esegui `yarn add --dev eslint`

Aggiorna la sezione `scripts` del tuo `package.json` includendo un nuovo task `test`:

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src"
},
```

In questo modo diciamo a ESLint che vogliamo verificare tutti i file JavaScript contenuti nella cartella `src`.

Utilizzeremo il task standard `test` per eseguire una catena di tutti i comandi che serviranno a controllare il nostro codice, che si tratti di linting, type checking, o unit testing.

- Esegui `yarn test`, e dovresti vedere tutta una serie di errori di ';' mancanti, ed un warning per l'utilizzo di `console.log()` in `index.js`. Aggiungi `/* eslint-disable no-console */` in cima al file `index.js` per consentire l'utilizzo di `console` all'interno di questo file.

**Nota**: Se sei in Windows, assicurati di configurare il tuo editor e Git in modo da usare i terminatori di linea di tipo Unix LF e non Windows CRLF. Se il tuo progetto è utilizzato unicamente in ambienti Windows, puoi invece aggiungere `"linebreak-style": [2, "windows"]` alle regole di ESLint nell'array `rules` (vedi l'esempio in basso) per forzare l'utilizzo di CRLF.

### Punti e virgola

OK, questo probabilmente è uno dei dibattiti più accesi della comunità JavaScript, affrontiamolo per un minuto. JavaScript ha questa capacità di inserire automaticamente i punti e virgola, questo ti permette di decidere se scriverli o no. Si tratta puramente di preferenze personali e non c'è una scelta giusta o sbagliata. Se preferisci la sintassi nello stile di Python, Ruby, o Scala, vorrai probabilmente ometterli. Se sei abituato alla sintassi di Java, C#, o PHP, probabilmente preferirai scriverli.

La maggiorparte delle persone usano JavaScript con i punti e virgola, per via dell'abitudine. Anche io facevo parte di questo gruppo, fino a quando ho deciso di provare ad ometterli dopo aver visto alcuni esempi nella documentazione di Redux. All'inizio sembrava un po' strano, semplicemente perchè non ero abituato. Dopo un solo giorno a scrivere codice in questo modo non potevo più tornare indietro. I punti e virgola sembrano prolissi e non necessari. Secondo me un codice senza punti e virgola risulta più semplice da leggere e più veloce da scrivere.

Ti raccomando di leggere la [documentazione di ESLint sui punti e virgola](http://eslint.org/docs/rules/semi). Come spiegato in questa pagina, se decidi di omettere i punti e virgola, ci sono comunque alcuni rari casi in cui sono ancora necessari. ESLint può proteggerti da questi casi utilizzando la regola `no-unexpected-multiline`. Configuriamo ESLint per funzionare con questa sintassi senza punti e virgola, modificando il file `.eslintrc.json`:

```json
{
  "extends": "airbnb",
  "rules": {
    "semi": [2, "never"],
    "no-unexpected-multiline": 2
  }
}
```

🏁 Esegui `yarn test`, e dovrebbe funzionare correttamente. Prova ad aggiungere da qualche parte un punto e virgola non necessario e verifica che le regole funzionino correttamente.

Sono consapevole che alcuni di voi vorranno continuare ad utilizzare i punti e virgola, il che andrà in conflitto con il codice disponibile in questo tutorial. Se stai usando questa guida semplicemente per imparare, sono sicuro che protrai seguirla così com'è, e tornerai ad utilizzare i punti e virgola nei tuoi progetti reali. Se vuoi utilizzare il codice fornito nel tutorial come un boilerplate avrai tuttavia bisogno di affettuare un po' di modifiche, il che dovrebbe essere abbastanza veloce impostando ESLint per l'utilizzo con i punti e virgola e seguendo le sue notifiche. Mi scuso se ti troverai in questa situazione.

### ESLint nel tuo editor

Questo capitolo ti spiega come utilizzare ESLint nel terminale, che è ottimo per riconoscere gli errori durante la compilazione / prima di effettuare il push, ma probabilmente vorrai integrarlo anche nel tuo IDE per avere un feedback immediato. NON usare il linting nativo del tuo IDE per il codice ES6. Configuralo in modo che l'eseguibile da utilizzare per il linting sia quello presente nella cartella `node_modules`. In questo modo può utilizzare tutte le configurazioni del tuo progetto, il preset Airbnb, etc. In caso contrario avrai invece semplicemente un linting ES6 generico.

## Flow

> 💡 **[Flow](https://flowtype.org/)**: Uno static type checker rilasciato da Facebook. Riscontra inconsistenze nei tipi delle veriabili utilizzate. Ad esempio ti segnalerà un errore se cerchi di utilizzare una stringa dove dovresti in realtà utilizzare un numero.

Per il momento, il nostro codice JavaScript è un codice ES6 valido. Flow può analizzare il JavaScript per darci qualche suggerimento, ma per utilizzarlo al massimo, dobbiamo aggiungere delle annotazioni nel nostro codice, che lo farà diventare non-standard. Dobbiamo insegnare a Babel ed ESLint cosa sono quelle annotazioni, per evitare che inizino a lamentarsi quando verificano il codice.

- Esegui `yarn add --dev flow-bin babel-preset-flow babel-eslint eslint-plugin-flowtype`

`flow-bin` è l'eseguibile di Flow utilizzato in `scripts` , `babel-preset-flow` è il preset di Babel per comprendere le annotazioni di Flow, `babel-eslint` permette ad ESLint di *fare affidamento al parser di Babel* al posto del suo, e `eslint-plugin-flowtype` è un plugin di ESLint per le annotazioni di Flow. Phew.

- Aggiorna il file `.babelrc` in questo modo:

```json
{
  "presets": [
    "env",
    "flow"
  ]
}
```

- Aggiorna anche `.eslintrc.json`:

```json
{
  "extends": [
    "airbnb",
    "plugin:flowtype/recommended"
  ],
  "plugins": [
    "flowtype"
  ],
  "rules": {
    "semi": [2, "never"],
    "no-unexpected-multiline": 2
  }
}
```

**Nota**: `plugin:flowtype/recommended` contiene le istruzioni perchè ESLint utilizzi il parser di Babel. Se vuoi essere più esplicito, sentiti libero di aggiungere `"parser": "babel-eslint"` in `.eslintrc.json`.

Lo so che c'è molta carne sul fuoco, prenditi un minuto di tempo per ripensare a tutto il processo. Sono ancora stupefatto che ESLint possa utilizzare il parser di Babel'per comprendere le annotazioni di Flow. Questi 2 tool sono veramente incredibili per essere così modulari.

- Concatena `flow` al task `test`:

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src && flow"
},
```

- Crea un file `.flowconfig` alla base del tuo progetto con questo contenuto:

```flowconfig
[options]
suppress_comment= \\(.\\|\n\\)*\\flow-disable-next-line
```

Questa è una piccola utility che abbiamo impostato per far si che Flow ignori qualsiasi warning riscontrato nella linea successiva. Lo utilizzeresti così, similmente a `eslint-disable`:

```js
// flow-disable-next-line
something.flow(doesnt.like).for.instance()
```

OK, dovremmo essere pronti per la configurazione.

- Aggiungi annotazioni di Flow in `src/dog.js` così:

```js
// @flow

class Dog {
  name: string

  constructor(name: string) {
    this.name = name
  }

  bark() {
    return `Wah wah, I am ${this.name}`
  }
}

export default Dog
```

Il commento `// @flow` dice a Flow che vogliamo che questo file venga controllato. Per il resto, le annotazioni Flow sono tipicamente composte da un duepunti dopo un parametro o il nome di una funzione. Controlla la [documentazione](https://flowtype.org/docs/quick-reference.html) per maggiori dettagli.

- Aggiungi `// @flow` anche in cima al file `index.js`.

`yarn test` adesso dovrebbe fare sia il lint che il type-check del codice.

Ci sono 2 cose che dovresti provare:

- In `dog.js`, sostituisci `constructor(name: string)` con `constructor(name: number)`, e lancia `yarn test`. Dovresti ottenere un errore di **Flow** perchè i due tipi sono incompatibili. Questo significa che Flow funziona correttamente.

- Adesso sostituisci `constructor(name: string)` con `constructor(name:string)`, ed esegui `yarn test`. Dovresti ottenere un errore **ESLint** perchè le annotazioni Flow devono avere uno spazio dopo il duepunti. Questo dimostra che il plugin di Flow per ESLint funziona correttamente.

🏁 Se hai ottenuto i 2 errori, sei a posto con Flow ed ESLint! Ricordati di rimettere lo spazio nell'annotazione di Flow.

### Flow nel tuo editor

Proprio come ESLint, dovresti spendere un po' di tempo a configurare il tuo editor / IDE per ricevere un feedback immediato quando Flow rileva problemi nel codice.

## Jest

> 💡 **[Jest](https://facebook.github.io/jest/)**: una libreria JavaScript di testing rilasciata da Facebook. È molto semplice da configurare e ti fornisce direttamente tutte le funzionalità di cui avresti bisogno in una libreria di testing. Può anche testare i component di React.

- Esegui `yarn add --dev jest babel-jest` per installare Jest ed il package per farlo funzionare con Babel.

- Aggiungi quanto segue nel file `.eslintrc.json` alla radice dellóggetto, per poter utilizzare le funzioni di Jest senza doverle importare in ogni file di test:

```json
"env": {
  "jest": true
}
```

- Crea un file `src/dog.test.js` contenente:

```js
import Dog from './dog'

test('Dog.bark', () => {
  const testDog = new Dog('Test')
  expect(testDog.bark()).toBe('Wah wah, I am Test')
})
```

- Aggiungi `jest` allo script `test`:

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src && flow && jest --coverage"
},
```

Il parametro `--coverage` fa generare a Jest automaticamente le informazioni di coverage per i tuoi test. Questo è utile per verificare in queli parti del codice mancano dei test. Inserisce queste informazioni nella cartella `coverage`.

- Aggiungi `/coverage/` al tuo `.gitignore`

🏁 Esegui `yarn test`. Dopo il linting ed il type checking, dovrebbe eseguire i test di Jest e visualizzare una tabella di coverage. Dovrebbe essere tutto verde!

## Git Hooks con Husky

> 💡 **[Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)**: Sono degli script che vengono eseguite quando avvengono certe azioni quali commit o push.

Okay, adesso abbiamo questo task `test` che ci dice se il nostro codice sembra corretto o no. Imposteremo le Git Hooks per eseguire automaticamente questo task prima di ogni `git commit` e `git push`, che ci eviterà di inviare del codice non corretto al repository se non passa il task di `test`.

[Husky](https://github.com/typicode/husky) è un package che rende molto semplice la configurazione delle Git Hooks.

- Esegui `yarn add --dev husky`

Tutto quello che dobbiamo fare è aggiungere due nuovi task in `scripts`, `precommit` e `prepush`:

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test"
},
```

🏁 Adesso se provi a fare un commit o push del codice, dovrebbe partire in automatico il task `test`.

**Nota**: Se stai facendo un push subito dopo un commit, puoi usare `git push --no-verify` per evitare di eseguire tutti i test un'altra volta.

Prossimo capitolo: [03 - Express, Nodemon, PM2](03-express-nodemon-pm2.md#readme)

Torna al [capitolo precedente](01-node-yarn-package-json.md#readme) o all'[indice dei contenuti](https://github.com/fbertone/guida-javascript-moderno#indice-dei-contenuti).
