# Impostare un progetto Vue+Vite

usando le dipendenze che conosciamo

## Inizializziamo il progetto

lancio i seguenti comandi:

- npm create vite@latest .
- seleziono su richiesta:

  - vue
  - javascript

- npm install
<!-- installa npm -->
- npm run dev
<!-- ci fa partire il server virtuale -->

## installiamo sass per leggere l'scss

lancio i comando

- npm add -D sass
- note:
  - ricorda di mettere l'attributo lang='scss' nel campo style degli elementi
  - crea un file .scss per le istruzioni scss e importalo in ->main.js<-
  - extra: controlla che sia l'ultimo file di style in modo da sovrascrivere le direttive, per esempio, di bootstrap

## installiamo bootstrap

lancio il comando

- npm i bootstrap
- inserisco bootstrap nel file ->main.js<- (fai attenzione al percorso)

```javascript
// importo il css di bootstrap
import "bootstrap/dist/css/bootstrap.min.css";
// importo le regole javascript di bootstap
import * as bootstrap from "bootstrap";
```

## installo font awesome

lancio i comand

- npm i --save @fortawesome/fontawesome-svg-core
- i seguenti comandi sono per le librerie di icone gratuite:
  - npm i --save @fortawesome/free-solid-svg-icons
  - npm i --save @fortawesome/free-regular-svg-icons
  - npm i --save @fortawesome/free-brands-svg-icons
- nell'ambiente vue 3.x
  - npm i --save @fortawesome/vue-fontawesome@latest-3

una volta installato quanto ci serve di Fontawesome vado ad implementarlo nel file ->main.js<-

```javascript
/* import the fontawesome core */
import { library } from "@fortawesome/fontawesome-svg-core";

/* import font awesome icon component */
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
```

importo icone specifiche:

questa parte importa una icona specifica. io scelgo di importare la busta della mail (faEnvelope) e l'aereoplano di carta (faPaperPlane)

- questa operazione è da fare per ogni icona di FontAwesome che includo nel mio progetto
- notare che il nome dell'oggetto del'icona è sempre:
  - fa (di fontawesome) + Nome (maiuscolo, senza spazi ma con lettera maiuscola a ogni nuova parola.. es: paper-line = PaperLine)

```javascript
/* import specific icons */
import { faEnvelope } from "@fortawesome/free-solid-svg-icons";
import { faPaperPlane } from "@fortawesome/free-solid-svg-icons";
```

ogni icona va aggiunta alla libreria
nel caso in esame aggiungo sempre faEnvelope e faPaperPlane come segue:

```javascript
/* add icons to the library */
library.add(faEnvelope, faPaperPlane);
```

per usare font awerome devo agganciarmi a un componente al quale aggiungerò tut-ti gli elementi che mi serviranno

```javascript
/* attacco la riga font-awesome-icon al lancio dell'app  */
const app = createApp(App);
app.component("font-awesome-icon", FontAwesomeIcon);
app.component("AppLoading", AppLoading);
app.mount("#app");
```

## installo axios

lancio il comando

- npm install axios

è possibile importare axios solo nei componenti che lo usano aggiungendo queste righe all'interno del tag

```html
<script>
   // importo axios
   import axios from "axios";

  [...il resto del codice...]
</script>
```

## intallo uno store

- creiamo una cartella 'data' nel nostro progetto
- creiamo al suo interno un un file store.js - all'interno del file store.js lo valorizzo così:

  ```javascript
  import { reactive } from "vue";
  // qui dentro mettimao tutti i data globali
  export const store = reactive({
    apiUri: "valore di apiUri",
    ecc: " valore di ecc",
    ecc2: " valore di ecc2",
    ecc3: " valore di ecc3",
  });
  ```

- importo lo store in tutti i componenti che lo usano

```html
<script>
  // importare store
  // bisogna importarlo con le grafe perchè ha bisogno del destructuring
  import { store } from "./data/store";
</script>
```

- inserisco 'store' nei data dei componenti che lo usano:

```javascript
	  data() {
	    return {
	      store,
	    };
```

- nota bene: potrai chiamare le variabili dento store con la dot notation:
  - store.apiUri;
  - store.ecc;

## inserisco il Router: (meglio leggere le slide 93-vue-Router)

lancio il comando

- npm install vue-router@4
  inserisco il tag router-view nella App.vue, qui verranno visualizzate le rotte gestite dal router

```html
<template>
  <router-view></router-view>
</template>
```

- suggerimento: mettere le viste dei nostri componenti "pagina" dentor a una cartella 'pages'

- di seguito un esempio di router.js

```javascript
import { createRouter, createWebHistory } from "vue-router";
// vedi sotto
import AppHome from "./pages/AppHome.vue";
import PostList from "./pages/PostList.vue";
const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      // in path troviamo l'url che vedremo anche nel browser
      path: "/",
      //   in name troviamo il nome della rotta. è il valore che verrà chiamato dai bottoni che implementano il tag <router-link :to="{name: 'home'}"> </router-link>
      name: "home",
      //quale componente apparirà a questa rotta? lo abbiamo importato nel file router.js? vedi sopra
      component: AppHome,
    },
    {
      // in questo caso passo il parametro id
      path: "/blog/:id",
      name: "post",
      component: PostList,
    },
  ],
});
export { router };
```

- importo il file router.js nel main.js - che lo userà al mounth dell'applicazione
  -come sempre attenzione ai percorsi degli import

  ```javascript
  import { router } from "./router";
  createApp(App).use(router).mount("#app");
  ```

- per navigare uso il tag

```html
<router-link :to="{ name: 'home' }"> portami alla home </router-link>
```

- per passare un parametro uso questa dicitura

```html
<router-link
  :to="{ name: 'post', params: { id: post_id } }"
  class="btn btn-primary"
>
  Leggi articolo
</router-link>
```

- per leggere il valore di parametro passato all'apertura del componente posso usare la seguente dicitura

```javascript
// leggendo al contrario: recupera l'id passato ai params della $route
this.$route.params.id;
```

## elementi che conosco per inizializzare un componente:

sarebbe più comodi crearsi uno snippet

```javascript
export default {
  components: {},

  data() {
    return {
      store,
    };
  },

  computed: {},

  methods: {},

  //created(): {},

  props: {},

  emits: [],
};
```
