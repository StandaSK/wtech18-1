# 4. časť: vytvorenie a editovanie produktu

## Vytvorenie produktu
V komponente `pages/products/Create.vue` vytvoríme metódu `createProduct`.
Element `<script>` v komponente bude obsahovať:

```js
<script>
import axios from 'axios'

export default {
  data () {
    return {
      productName: '',
      productDescription: ''
    }
  },
  methods: {
    createProduct () {
      axios
        .post(`http://127.0.0.1:8000/products`, this.productData)
        .then(response => {
          this.$q.notify({type: 'positive', timeout: 2000, message: 'The product has been created.'})
          this.$router.push({path: '/products/' + response.data.id + '/edit'})
        })
        .catch(error => {
          this.$q.notify({type: 'negative', timeout: 2000, message: 'An error has been occured.'})
          console.log(error)
        })
    }
  },
  computed: {
    productData: function () {
      return { name: this.productName, description: this.productDescription }
    }
  }
}
</script>
``` 

Vidíme, že cez XHR (axios) HTTP metódu `post` voláme endpoint `/products`, ktorému odosielamé údaje produktu - `productData`, čo je `computed property`, ktorá zostavuje objekt z `productName` a `productDescription`. Po úspešnom vytvorení projektu sme "presmerovaní" na `/products/{ID PROJEKTU}/edit`

V šablóne nastavíme *v-model* vstupným poliam `name` a `description`, čím (reaktívne) prepojíme model (dáta) so šablónou (vstupnými poliami). Pri akejkoľvek zmene niektorého zo vstupných polí je automaticky aktualizovaný objekt `productData`:

```html
<template>
<div class="q-my-xl">
    <q-card>
        <q-card-title>Create new product</q-card-title>
        <q-card-main>
            <q-field :count="250">
                <q-input float-label="Name" v-model="productName" max-length="250" />
            </q-field>
            <q-field :count="5000">
                <q-input
                    type="textarea"
                    float-label="Description"
                    v-model="productDescription"
                    :max-height="100"
                    rows="7"
                />
            </q-field>
            <q-field helper="Supported format: JPG, max. file size: 300KiB" class="q-mt-lg">
                <q-uploader float-label="Images" multiple extensions=".jpg" auto-expand/>
            </q-field>
        </q-card-main>
        <q-card-actions class="q-mt-md">
            <div class="row justify-end full-width docs-btn">
                <q-btn label="Cancel" flat to="/products/index"/>
                <q-btn label="Create" color="primary" @click="createProduct"/>
            </div>
        </q-card-actions>
    </q-card>
</div>
</template>
```

Na tlačidlo `Create` naviažeme udalosť *click* s volaním metódy `createProduct`. 

Na backende v `ProductController` vytvoríme službu `store`:
```php
public function store(Request $request)
{
    // validations and error handling is up to you!!! ;)
    /*
    $request->validate([
        'name' => 'required|min:3',
        'description' => 'required',
    ]);
    */
      
    $product = Product::create(['name' => $request->name, 'description' => $request->description]);
    return response()->json(['id' => $product->id]);
}
```
 
Samozrejme nezabudneme na smerovanie:
```php
Route::post('products/', 'ProductController@store');
```

### Ajax loading bar
[Quasar ajax bar](https://quasar-framework.org/components/ajax-bar.html) je komponent, ktorý indikuje načítavanie - niečo podobné ako používa napr. YouTube (keď si dáte prehrať nejaké video, tak sa zobrazí pod address barom prehliadača červená čiara, ktorá sa rozširuje a indikuje tým načítavanie). 

Integrovanie tohto komponentu do našej Quasar aplikácie je veľmi jednoduché. Do `/quasar.conf.js` pridajme `QAjaxBar`. V komponente (layoute) `Main.vue` pod `<router-view></router-view>` pridajme:

```html
<router-view></router-view>
<q-ajax-bar />
```

Ajax bar sa spustí automaticky pri každom AJAXovom volaní a zastaví sa (dobehne) po prijatí odpovede. 
 
 
## Editovanie produktu
Logika komponentu `pages/products/Edit.vue` bude obsahovať dve AJAXové volania. Jedným volaním potrebujeme načítať do vstupných polí predmetný produkt a druhé volanie zrealizuje na požiadanie aktualizáciu produktu.
Element `<script>` v komponente bude obsahovať:

```js
<script>
import axios from 'axios'

export default {
  data () {
    return {
      productName: '',
      productDescription: ''
    }
  },
  methods: {
    updateProduct () {
      axios
        .put(`http://127.0.0.1:8000/products/` + this.$route.params.id, this.productData)
        .then(response => {
          this.$q.notify({type: 'positive', timeout: 2000, message: 'The product has been updated.'})
        })
        .catch(error => {
          this.$q.notify({type: 'negative', timeout: 2000, message: 'An error has been occured.'})
          console.log(error)
        })
    }
  },
  mounted () {
    axios
      .get(`http://127.0.0.1:8000/products/` + this.$route.params.id + '/edit')
      .then(response => {
        this.productName = response.data.name
        this.productDescription = response.data.description
      })
      .catch(error => {
        this.$q.notify({type: 'negative', timeout: 2000, message: 'Loading product: an error has been occured.'})
        console.log(error)
      })
  },
  computed: {
    productData: function () {
      return { name: this.productName, description: this.productDescription }
    }
  }
}
</script>
```

Spomeňte si z prednášky na životný cyklus Vue.js komponentu, alebo si [pozrite dokumentáciu](https://vuejs.org/v2/api/#Options-Lifecycle-Hooks). Fáza `mounted` je vhodná na načítanie a nastavenie dát v komponente. Tu teda vytvoríme AJAXové volanie na endpoint  `/products/{ID PRODUCT}/edit`, ktorý nám vráti informácie o predmetnom produkte. Týmito informáciami naplníme náš model komponentu - objekt `productData`. 

Vytvoríme metódu `updateProduct`, v ktorej voláme HTTP metódou `put` endpoint `/products/{ID PRODUCT}` a odosielame informácie o produkte. Ošetrovanie možných chýb ponechávam na vás. 

V šablóne komponentu nastavíme vstupným poliam `v-model` a na tlačidlo `Update` naviažeme udalosť `click` s volaním metódy `updateProduct`:
```html
<template>
<div class="q-my-xl">
    <q-card>
        <q-card-title>Edit {{ productName }}</q-card-title>
        <q-card-main>
            <q-field :count="250">
                <q-input float-label="Name" v-model="productName" max-length="250" />
            </q-field>
            <q-field :count="5000">
                <q-input
                    type="textarea"
                    float-label="Description"
                    v-model="productDescription"
                    :max-height="100"
                    rows="7"
                />
            </q-field>
        </q-card-main>
        <q-card-actions class="q-mt-md">
            <div class="row justify-end full-width docs-btn">
                <q-btn label="Cancel" flat to="/products/index"/>
                <q-btn label="Update" color="primary" @click="updateProduct"/>
            </div>
        </q-card-actions>
    </q-card>
</div>
</template>
``` 

Na backende v `ProductController` vytvoríme potrebné služby:

```php
public function edit(Product $product)
{
    return response()->json($product);
}

public function update(Request $request, Product $product)
{
    // validations and error handling is up to you!!! ;)
    /*
    $request->validate([
        'name' => 'required|min:3',
        'description' => 'required',
    ]);  
    */
         
    $product->name = $request->name;
    $product->description = $request->description;
    $product->save();
}
```

Nastavíme smerovanie:
```php
Route::get('products/{product}/edit', 'ProductController@edit');
Route::put('products/{product}', 'ProductController@update');
```

Nahrávanie obrázkov k produktom ponechám na vás. Malý hint, v komponente [`q-uploader`](https://quasar-framework.org/components/uploader.html) skryte *upload button* (atribút `hide-upload-button`) a `upload()` volajte v metóde `createProduct` (resp. `updateProduct`). Vytvorte si BE službu na nahrávanie obrázkov. Upravte metódu `createProduct` tak, že najskôr vytvoríte produkt (služba `store`), nahráte fotografie, a `id`, ktoré vráti služba `store` použijete na prepojenie nahratých fotografií s produktom v DB.

# KONIEC 