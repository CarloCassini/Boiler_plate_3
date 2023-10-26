php artisan make:request StoreProjectRequest

mettere true sul return dell' authorize

scrivere le regole sul metodo Rules()
!! mi raccomando di inserire tutti i campi del fillable (hai aggiunto il fillable al model?)

vanno gestite le regole come un array
esempio:

```php
    public function rules()
    {
        return [
            'name' => ['required', 'string', 'max:50'],
            'git_url' => ['url'],
            'description' => ['string']
        ];
    }
```

inserire tra le regole anche i campi che regole non ne hanno. mettere array vuoto

<!-- xxx -->

```php
'ciccio' => [],
```

<!-- xxx -->

-   per i messaggi personalizzati bisogna creare un altro public function messages() --come rules sopra--

```php
    public function messages()
    {
        return [
            'name.required' => 'il nome è obbligatiorio',
            'name.string' => 'il nome deve essere un testo',
            'name.max' => 'il nome deve essere max di 50 caratteri',

            'git_url.url' => 'inserisci un URL',

            'description.string' => 'la descrizione deve essere di tipo testo',
        ];
    }
```

spostarsi sul controller

## per store

inserire il nome del validator tra parametri della funzione store

```php
  public function store(StoreProjectRequest $request)
```

mettere validated invece di "all"

```php
 $data = $request->validated();
```

---

per visualizzare la lapide degli errori

```html
@if ($errors->any())
<div class="alert alert-warning">
    <h5>correggi i seguenti errori</h5>
    <ul>
        @foreach ($errors->all() as $error)
        <li>{{ $error}}</li>
        @endforeach
    </ul>
</div>
@endif
```

---

come risposta della chiamata allo STORE
per fare apparire gli errori sotto i campi

```html
<input
    type="text"
    class="form-control 
      
    <!--*** se eistono errori sul name metti la classe is-invalid (fa diventare rosso il bordo dell'input) -->
     
      @error('name')
          is-invalid
      @enderror"
    id="name"
    name="name"
    value="{{old('name')}}"
/>

<!--*** se eistono errori sul name crea un div con invalid-feedback che mostra il primo messaggio di errore del campo -->

@error('name')
<div class="invalid-feedback">{{ $message}}</div>
@enderror
```

---

per prendere il valore vecchio ci vuole ,
{{old}} nella value
