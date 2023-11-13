# CREAZIONE API

**_CONSIGLIO_**  
ricorda che se stai utilizzando _`laravel`_ e _`vue`_ in contemporanea, è facile confondersi, utilizza lo snippet per cambiare il colore della barra

**ES.**

```JSON
  "Set Laravel Theme": {
    "prefix": "!theme-laravel",
    "body": [
      "{",
      "  \"workbench.colorCustomizations\": {",
      "    \"titleBar.activeBackground\": \"#ac3535\",",
      "    \"titleBar.activeForeground\": \"#fff\",",
      "  }",
      "}"
    ],
    "description": "Log output to console"
  },
```

## _`PARTE LARAVEL`_

## Creazione Controller API

Creiamo il controller che gestirà le rotte delle _API_

```
php artisan make:controller Api\PostController --api
```

**_N.B._**

`APi\postController` indica il _namespace_ e quindi **Cartella\NomeController**  
`--api` indica il resource controller dell'api (escludendo automaticamente i form _create, edit_)

## Creazione Rotta

Apriamo il file `api.php` che si trova nella cartella `routes`, importiamo il _Controller_ appena creato e creiamo la rotta dandogli la risposta in _Json_

```php
// api.php

use App\Http\Controllers\Api\PostController;    // Importazione controller

Route::apiResource("/posts", PostController::class)->only("index","show");   // Collegamento delle Rotta in automatico
```

**_N.B._**

`->only("index","show")` indica al controller che deve contare solo _index_, _show_ come rotte

# Creazione Api

Sul controller precedentemente creato andremo ad inserire una variabile che conterra l'oggetto che dovrà diventare una api
e lo ritorniamo sottoforma di _Json_

```php
// PostController.php

public function index()
{
  $projects = Project::select('id', 'name', 'slug')->with('tags:id,color,label', 'category:id,color,label')->paginate(10);
  return response()->json($posts);
}
```

**_N.B._**

Se non hai bisogno del controller e quindi non lo crei, puoi dare le direttive direttamente nelle route

```php
// api.php

Route::get("/posts", function(){
    $posts=Post::paginate(10);
    return response()-json([$posts]);
});
```

## _`PARTE VUE`_

**_RIEPILOGO_**

Avvia un altro progetto ma con `vite` e `vue`, installa tutte le dipendence e tecnologie, (_sass_, _bootstrap_, _axios_ ecc)

**ES.**

```
npm create vite@latest
npm i
npm run dev
npm i -D sass-loader sass
npm i bootstrap@5.3.2
npm i axios
```

## Collegamento API

Creiamo un metodo con `axio`e diamoli come _url_ la rotta delle _API_ e richiamiamolo anche alla _creazione_ della pagina

```JavaScript
//  App.vue

export default {
  data() {
    return {
      title: "APP",
      projects: [],
      api: {
        baseUrl: "http://127.0.0.1:8000/api/",
      },
    };
  },

  methods: {
    fetchProjects(uri = this.api.baseUrl + "project") {
      axios.get(uri).then((responce) => {
        this.projects = responce.data.data;
      });
    },
  },

  created() {
    this.fetchProjects();
  },
}
```

##### `POWERED BY FRANCESCO`
