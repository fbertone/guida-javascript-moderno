# 09 - Travis, Coveralls, ed Heroku

Il codice di questo capitolo è disponibile nel branch `master` del repository [JS-Stack-Boilerplate](https://github.com/verekia/js-stack-boilerplate).

In questo capitolo integreremo l'app con servizi di terze parti. Questi servizi offrono piani gratuiti e a pagamento. È un po' controverso utilizzare servizi di questo tipo in un tutorial che fa leva unicamente su tool opensource sviluppati dalla comunità, per questo motivo verranno mantenuti 2 branch separati del repository [JS-Stack-Boilerplate](https://github.com/verekia/js-stack-boilerplate), `master` e `master-no-services`.

## Travis

> 💡 **[Travis CI](https://travis-ci.org/)** è una piattaforma popolare di continuous integration, gratuita per progetti open source.

Se il tuo progetto è disponibile apertamente su Github, l'integrazione con Travis è molto semplice. Innanzitutto autenticati su Travis utilizzando il tuo account Github, e aggiungi il tuo repository.

- Poi crea il file `.travis.yml` contenente:

```yaml
language: node_js
node_js: node
script: yarn test && yarn prod:build
```

Travis si accorgerà automaticamente che utilizzi Yarn perchè è presente il file `yarn.lock`. Ogni volta che fai un push del codice sul tuo repository in Github, eseguirà `yarn test && yarn prod:build`. Se niente va storto, dovresti vedere una build verde.

## Coveralls

> 💡 **[Coveralls](https://coveralls.io)** è un servizio che mantiene una cronologia e delle statistiche sulla copertura dei test.

Se il tuo progetto è open-source su Github e compatibile con i servizi di Continuous Integration supportati da Coveralls, l'unica cosa che devi fare è inviare in pipe il file di copertura generato da Jest all'eseguibile di `coveralls`.

- Esegui `yarn add --dev coveralls`

- Modifica la sezione `script` di `.travis.yml` in questo modo:

```yaml
script: yarn test && yarn prod:build && cat ./coverage/lcov.info | ./node_modules/coveralls/bin/coveralls.js
```

Adesso, ogni volta che Travis farà una build, sottometterà automaticamente le informazioni di copertura dei test di Jest a Coveralls.

## Badges

È tutto verde su Travis e Coveralls? Ottimo, mostralo al mondo con dei badge scintillanti!

Puoi usare direttamente il codice fornito da Travis o Coveralls, oppure utilizzare [shields.io](http://shields.io/) per rendere omogenei o personalizzare i badge. Usiamo shields.io.

- Crea o modifica il tuo `README.md` in questo modo:

```md
[![Build Status](https://img.shields.io/travis/GITHUB-USERNAME/GITHUB-REPO.svg?style=flat-square)](https://travis-ci.org/GITHUB-USERNAME/GITHUB-REPO)
[![Coverage Status](https://img.shields.io/coveralls/GITHUB-USERNAME/GITHUB-REPO.svg?style=flat-square)](https://coveralls.io/github/GITHUB-USERNAME/GITHUB-REPO?branch=master)
```

Ovviamente, sostituisci `GITHUB-USERNAME/GITHUB-REPO` con il tuo username di Github ed il nome del repository.

## Heroku

> 💡 **[Heroku](https://www.heroku.com/)** è una [PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service) su cui puoi effettuare il deploy. Si occuperà dei dettagli dell'infrastruttura, in modo che tu possa focalizzarti sullo sviluppo dell'app senza preoccuparti di quello che succede al di sotto.

Questo tutorial non è sponsorizzato in alcun modo da Heroku, ma essendo Heroku una piattaforma che sa il fatto suo, ti spegherò come puoi farci il deploy dell'app. Questo è il tipo di affetto gratuito che ricevi quando costruisci un ottimo prodotto.

**Nota**: Una sezione su AWS potrebbe essere aggiunta in questo capitolo successivamente, ma una cosa alla volta.

### Web setup

- Se non l'hai ancora fatto, installa la [CLI di Heroku](https://devcenter.heroku.com/articles/getting-started-with-nodejs) ed effettua il log in.

- Vai nella tua [Dashboard di Heroku](https://dashboard.heroku.com/) e crea 2 app, una chiamata `your-project` e un'altra chiamata ad esempio `your-project-staging`.

Lasceremo che Heroku si occupi di effettuare il transpiling del nostro codice ES6/Flow tramite Babel, e generi dei bundle per i client tramite Webpack. Ma siccome quelle sono delle  `devDependencies`, Yarn non le installerà in un ambiente di produzione come Heroku. Cambiamo questo comportamente utilizzando la variabile d'ambiente `NPM_CONFIG_PRODUCTION`.

- In entrambe le app, sotto Settings > Config Variables, aggiunge `NPM_CONFIG_PRODUCTION` impostata a `false`.

- Crea una Pipeline, e permetti ad Heroku l'accesso al tuo Github.

- Aggiungi le 2 app alla pipeline, imposta quella di staging in modo che venga effettuato un auto-deploy in seguito ai cambiamenti in `master`, e abilita Review Apps.

Ok, prepariamo il nostro progetto per il deploy su Heroku.

### Esecuzione in modalità produzione in locale

- Crea il file `.env` contenente:

```.env
NODE_ENV='production'
PORT='8000'
```

È all'interno di questo file che dovresti inserire le variabili locali e segrete. Non inserirle nei commit in un repository pubblico se il tuo progetto è privato.

- Aggiungi `/.env` al tuo `.gitignore`

- Crea il file `Procfile` contenente:

```Procfile
web: node lib/server
```

Qua è dove specifichiamo il punto d'ingresso del nostro server.

Non useremo più PM2, useremo al suo posto `heroku local` per lanciare in modalità produzione in locale.

- Esegui `yarn remove pm2`

- Modilica lo script `prod:start` in `package.json`:

```json
"prod:start": "heroku local",
```

- Rimuovi `prod:stop` da `package.json`. Non ci servirà più siccome `heroku local` è un processo che rimane appeso e possiamo killare con Ctrl+C, a differenza di `pm2 start`.

🏁 Esegui `yarn prod:build` e `yarn prod:start`. Dovrebbe far partire il server e visualizzare i log.

### Deploy in produzione

- Aggiungi la seguente linea a `scripts` in `package.json`:

```json
"heroku-postbuild": "yarn prod:build",
```

`heroku-postbuild` è un task che verrà eseguito ogni volta che effettuerai il deploy di un'app su Heroku.

Probabilmente vorrai anche specificare una versione precisa di Node o Yarn da utilizzare su Heroku.

- Aggiungi quanto segue a `package.json`:

```json
"engines": {
  "node": "7.x",
  "yarn": "0.20.3"
},
```

- Crea `app.json` contenente:

```json
{
  "env": {
    "NPM_CONFIG_PRODUCTION": "false"
  }
}
```

Questo è da utilizzare in Review App.

Adesso dovresti essere pronto per effettuare i deploy tramite la Pipeline di Heroku.

🏁 Crea un nuovo branch di git, fai delle modifiche e apri Github Pull Request per istanziare una Review App. Verifica le tue modifiche in Review App URL e se ti sembra tutto corretto, fai il merge della tua Pull Request sul `master` in Github. Dopo alcuni minuti, la tua app di staging dovrebbe essere stata inviata in deploy automaticamente. Verifica le modifiche nello staging app URL, e se ti sembra che vada ancora tutto bene, promuovi lo staging in produzione.

Hai finito! Congratulazioni se hai terminato questo tutorial partendo da zero.

Ti meriti questa medaglia emoji: 🏅

Torna al [capitolo precedente](08-bootstrap-jss.md#readme) o all'[indice dei contenuti](https://github.com/fbertone/guida-javascript-moderno#indice-dei-contenuti).
