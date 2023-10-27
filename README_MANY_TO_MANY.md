# MANY TO MANY

## Migrations

Realizziamo una relazione fra l'entità tag (molte per post) e l'entità post (molti per tag).

### Prima migration (posts)

possiamo creare il file di migrazione con il comando

```
php artisan make:migration create_posts_table
```

nel quale andremo poi a specificare i campi, gli indici della tabella

```php
// xxxx_xx_xx_xxxxxx_create_posts_table

/**
 * Run the migrations.
 *
 * @return void
 */
public function up()
{
  Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title', 100);
    // altre colonne ...
    $table->timestamps();
  });
}
```

### Seconda migration (tags)

creeremo poi la migration per la tabella tags

```
php artisan make:migration create_tags_table
```

al cui interno aggiungeremo i campi

```php
// xxxx_xx_xx_xxxxxx_create_tags_table

/**
 * Run the migrations.
 *
 * @return void
 */
public function up()
{
  Schema::create('tags', function (Blueprint $table) {
    $table->id();
    $table->string('label', 20);
    // altre colonne ...
    $table->timestamps();
  });
}
```

### Terza migration (Tabella ponte, FKs & vincoli)

Infine abbiamo bisogno di la tabella ponte o pivot tra le due entità, aggiungere le foreign keys e realizzare i vincoli fra le altre due tabelle. Per farlo abbiamo bisogno di una terza migration.

NB: i nomi nelle tabelle nella file della migration sono al singolare ed in ordine alfabetico.

```
php artisan make:migration create_post_tag_table
```

Nella tabella pivot le FKs non possono essere null, altrimenti il DB sarebbe incoerente e le relazioni si romperebbero. In questo caso usiamo la cancellazione "a cascata".

```php
// xxxx_xx_xx_xxxxxx_create_post_tag_table

/**
 * Run the migrations.
 *
 * @return void
 */
public function up()
{
  Schema::create('post_tag', function (Blueprint $table) {
    $table->id();

    $table->foreignId('post_id')
      ->constrained()
      ->cascadeOnDelete();

    $table->foreignId('tag_id')
      ->constrained()
      ->cascadeOnDelete();
  });
}
```

## Models

Nei modelli aggiungiamo la relazione così che l'ORM possa mapparli correttamente.

### Primo model (tag)

Dal momento che è una relazione "molti a molti" non esiste entità "forte".
Iniziamo arbitrariamente dal modello `Tag`.

```php
// tag

class Tag extends Model {

  // ...

  public function posts() {
    return $this->belongsToMany(Post::class);
  }
}
```

### Secondo model (post)

Faremo lo stesso "al rovescio" per il modello Post

```php
// Post

class Post extends Model {

  // ...

  public function tags() {
    return $this->belongsToMany(Tag::class);
  }
}
```

Ora abbiamo accesso alla sintassi del tipo `$post->tag` oppure `$tag->posts`

## Seeders

Partiamo arbitrariamente col seeders per i tags.

#### Primo seeder (tags)

Creazione del seeder

```
php artisan make:seeder TagSeeder
```

In questo caso usiamo un array di categorie predefinite, ma possono essere generare anche con `Faker`.
Non aggiungeremo le FKs (che sono sulla tabella ponte).

```php
// tagSeeder

/**
* Run the database seeds.
*
* @return void
*/
public function run(Faker $faker)
{
  $labels = ["HTML", "CSS", "SQL", "JavaScript", "PHP", "GIT", "Blade"];

  foreach($labels as $label) {
    $tag = new Tag();
    $tag->label = $label;
    // ...
    $tag->save();
  }
}
```

Nel caso si usi Faker va sempre importato con:

```php
use Faker\Generator as Faker;
```

Possiamo quindi aggiungere il TagSeeder nel metodo `run` del file `DatabaseSeeder`

```php
// DatabaseSeeder

/**
 * Seed the application's database.
 *
 * @return void
 */
public function run()
{
  $this->call([
    TagSeeder::class,
    // ...
  ]);
}
```

#### Secondo seeder (posts)

Creiamo un seeder anche per i post

```
php artisan make:seeder TagSeeder
```

Non aggiungeremo le FKs (che sono sulla tabella ponte).

```php
// PostSeeder
/**
 * Run the database seeds.
 *
 * @return void
 */
public function run(Faker $faker)
{
  for($i = 0; $i < 40; $i++) {
      $post = new Post;
      $post->title = $faker->catchPhrase();
      // ...
      $post->save();
  }
}
```

Possiamo quindi aggiungere il PostSeeder nel metodo `run` del file `DatabaseSeeder`

```php
// DatabaseSeeder

/**
 * Seed the application's database.
 *
 * @return void
 */
public function run()
{
  $this->call([
    TagSeeder::class,
    PostSeeder::class,
    // ...
  ]);
}
```

#### Terzo seeder (tabella ponte post_tag)

Creiamo un seeder anche per la tabella ponte.

1. Prendiamo tutti i posts.
2. Prendiamo tutti i tags come array di id.
3. Per ognuno dei post aggiungiamo da 0 a 3 tags

```php
// PostTagSeeder

/**
 * Run the database seeds.
 *
 * @return void
 */
public function run(Faker $faker)
{
  $posts = Post::all();                       // object Post
  $tags = Tag::all()->pluck('id')->toArray(); // array  [1, 2, ... n]

  foreach($posts as $post) {
    $post
      ->tags()
      ->attach($faker->randomElements($tags, random_int(0, 3)));
  }
}
```

Possiamo quindi aggiungere il PostTagSeeder nel metodo `run` del file `DatabaseSeeder`

```php
// DatabaseSeeder

/**
 * Seed the application's database.
 *
 * @return void
 */
public function run()
{
  $this->call([
    TagSeeder::class,
    PostSeeder::class,
    PostTagSeeder::class,
    // ...
  ]);
}
```

## Controller + Views

Le CRUD per entrambe le entità possono essere realizzate seguendo la guida per le CRUD. Dobbiamo decidere su quale entità gestire la relazione. Arbitrariamente (perché sembra più comodo) la gestiremo dal controller e dalle viste della risorsa `posts`.

### Lettura: index

Nel controller non c'è bisogno di apportare modifiche. E' opportuno però visualizzare il nome della categoria nella lista

```blade
// views/posts/index.blade.php

<table class="table">
    <thead>
        <tr>
            ...
            <th scope="col">Tags</th>
            ...
        </tr>
    </thead>
    <tbody>
        @forelse($posts as $post)
        <tr>
            ...
            <td>
            @forelse($post->tags as $tag)
              {{ $tag->label }} @unless($loop->last) , @else . @endunless
            @empty
              -
            @endforelse
            </td>
            ...
        </tr>
        @empty
        <tr>
            <td colspan="n">Nessun risultato</td>
        </tr>
        @endforelse
    </tbody>
</table>
```

### Lettura: show

Nel controller non c'è bisogno di apportare modifiche. E' opportuno però visualizzare i tags associati nel dettaglio del post

```blade
  <strong>Tags:</strong>
  @forelse ($post->tags as $tag)
    {{ $tag->label }} @unless($loop->last) , @else . @endunless
  @empty
    Nessun tag associato
  @endforelse
```

### Creazione: create

Nel controller dobbiamo prendere tutti i possibili tags da passare alla vista

```php
public function create()
{
  $post = new Post;
  $tags = Tag::orderBy('label')->get();
  return view('admin.posts.form', compact('post', 'tags'));
}
```

e nel form dovremo stampare i checkbox.

l'attributo `name="tags[]"` con le quadre alla fine permette di inviare i valori di tutte le checkbox selezionate come array.

la riga `@if (in_array($tag->id, old('tags', []))) checked @endif` stampa l'attributo `checked` con le seguenti priorità:

1. valori precedentemente inviati dal form (caso form inviato e validazione fallita)
2. nessuno (caso form non inviato)

```blade
<label class="form-label">Tags</label>

<div class="form-check @error('tags') is-invalid @enderror p-0">
  @foreach ($tags as $tag)
    <input
      type="checkbox"
      id="tag-{{ $tag->id }}"
      value="{{ $tag->id }}"
      name="tags[]"
      class="form-check-control"
      @if (in_array($tag->id, old('tags', $post_tags ?? []))) checked @endif
    >
    <label for="tag-{{ $tag->id }}">
      {{ $tag->label }}
    </label>
    <br>
  @endforeach
</div>

@error('tags')
  <div class="invalid-feedback">
    {{ $message }}
  </div>
@enderror
```

### Creazione: store

Nel file PostStoreRequest dovremo validare la richiesta controllando che gli id ricevuti esistano nella tabella tags.

```php
public function rules()
  {
    return [
      // ...
      'tags' => ['nullable', 'exists:tags,id'],
    ];
  }

  public function messages()
  {
    return [
      // ...
      'tags.exists' => 'I tag inseriti non sono validi',
    ];
  }
```

Nel controller, metodo `store()`, bisogna usare `attach()` per salvare le relazioni nella tabella ponte.

```php
public function store(StorePostRequest $request) {
  $data = $request->validated();

  $post = new Post;
  $post->fill($data);
  $post->save();

  // ...

  if(Arr::exists($data, "tags")) $post->tags()->attach($data["tags"]);
}
```

### Modifica: edit

Nel controller dobbiamo selezionare

1. tutti i tag esistenti
2. tutti associati al post

ed inviarli alla view per la corretta visualizzazione delle checkbox

```php
/**
 * Show the form for editing the specified resource.
 *
 * @param  \App\Models\Post  $post
 * @return \Illuminate\Http\Response
 */
public function edit(Post $post)
{
    $tags = Tag::orderBy('label')->get();
    $post_tags = $post->tags->pluck('id')->toArray();
    return view('admin.posts.form', compact('post', 'tags', 'post_tags'));
}
```

nel form la riga `@if (in_array($tag->id, old('tags', $post_tags))) checked @endif` stampa l'attributo checked con le seguenti priorità:

1. valori precedentemente inviati dal form (caso form inviato e validazione fallita)
2. valori contenuti dall'istanza (caso form non inviato)

### Modifica: update

Nel file UpdateStoreRequest dovremo validare la richiesta con le stesse regole aggiunte nel file UpdatePostRequest.

Se la chiave tags è stata ricevuta Invece dell'attach va usato il `sync()`, Altrimenti vuol dire che nessun tag è stato selezionato e si usa il `detach()`

```php
/**
  * Update the specified resource in storage.
  *
  * @param  \Illuminate\Http\Request  $request
  * @param  \App\Models\Post  $post
  * @return \Illuminate\Http\Response
  */
public function update(UpdatePostRequest $request, Post $post)
{
  $data = $request->validated();

  $post->update($data);

  if(Arr::exists($data, "tags"))
    $post->tags()->sync($data["tags"]);
  else
    $post->tags()->detach();
}
```

### Cancellazione: destroy

Se è stato settato "on delete cascade" questo passaggio è opzionale.
Facciamo il `detach()` di tutte le relazioni prima dell'eliminazione del post

```php
/**
 * Remove the specified resource from storage.
 *
 * @param  \App\Models\Post  $post
 * @return \Illuminate\Http\Response
 */
public function destroy(Post $post)
{
  $post->tags()->detach();
  $post->delete();
}
```
