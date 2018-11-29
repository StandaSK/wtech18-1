# 2. časť: zoznam produktov - backend v Laraveli a jeho prepojenie s komponentom Index

## Backend v Laraveli

Pre *produkt* vytvoríme model, cotroller a migračný súbor:
``php artisan make:model Product -a``

V migračnom súbore ``*_create_roles_table.php`` doplníme v metóde ``up()``
*name* a *description* a testovacie dáta:

```php
public function up() {
    Schema::create('products', function (Blueprint $table) {
        $table->increments('id');
        $table->string('name');
        $table->text('description');
        $table->timestamps();
    });
  
    DB::table('products')->insert([
        'name' => 'Product 1',
        'description' => 'Description of product 1'
    ]);
    
    DB::table('products')->insert([
        'name' => 'Product 2',
        'description' => 'Description of product 2'
    ]);
  
    DB::table('products')->insert([
        'name' => 'Product 3',
        'description' => 'Description of product 3'
    ]);
 
    DB::table('products')->insert([
        'name' => 'Product 4',
        'description' => 'Description of product 4'
    ]);
 
    DB::table('products')->insert([
        'name' => 'Product 5',
        'description' => 'Description of product 5'
    ]);
  
    DB::table('products')->insert([
        'name' => 'Product 6',
        'description' => 'Description of product 6'
    ]);
  
    DB::table('products')->insert([
        'name' => 'Product 7',
        'description' => 'Description of product 7'
    ]);
  
    DB::table('products')->insert([
        'name' => 'Product 8',
        'description' => 'Description of product 8'
    ]);
  
    DB::table('products')->insert([
        'name' => 'Product 9',
        'description' => 'Description of product 9'
    ]);
  
    DB::table('products')->insert([
        'name' => 'Product 10',
        'description' => 'Description of product 10'
    ]);   
}
```

**Laravel poskytuje aparát na pohodlné vytváranie testovacích dát, čiže nemusíte vytvárať testovacie dáta takto "neohrabane", odporúčam pozrieť si [Laravel Database: Seeding](https://laravel.com/docs/5.7/seeding)**

Spustíme migráciu:
``php artisan migrate``


Ďalej, nastavíme smerovanie, v súbore ``routes/web.php`` pridáme:

```php
Route::get('products/list', 'ProductController@list');
```

V ``ProductController`` vytvorme metódu ``list``:

```php
public function list() {
    return Product::all()->toJson(JSON_PRETTY_PRINT);
}
```

Keď zavoláme službu (endpoint) ``/products/list`` mali by sme získať našich 10 produktov v JSON formáte.


## Načítanie zoznamu produktov a vytvorenie stránkovania

Ideme upraviť náš komponent *Index* (súbor `/src/pages/products/Index.vue`)
Načou úlohou je nahradiť náš dočasný statický súbor s produktami dynamickým načítaním dát z DB.

Pôvodný obsah elementu ``<script>`` nahradíme týmto kódom:

```js
<script>
import axios from 'axios'

export default {
  data () {
    return {
      columns: [
        { name: 'id', label: 'ID', field: 'id', sortable: false, align: 'left' },
        { name: 'name', label: 'Name', field: 'name', sortable: true, align: 'left' },
        { name: 'actions', label: 'Actions', sortable: false, align: 'right' }
      ],
      selected: [],
      loading: false,
      serverPagination: {
        page: 1,
        rowsNumber: 10, // the number of total rows in DB
        rowsPerPage: 5,
        sortBy: 'name',
        descending: true
      },
      serverData: []
    }
  },
  methods: {
    request ({ pagination }) {
      // QTable to "loading" state
      this.loading = true
      // fetch data
      axios
        .get(`http://127.0.0.1:8000/products/list/${pagination.page}?rowsPerPage=${pagination.rowsPerPage}&sortBy=${pagination.sortBy}&descending=${pagination.descending}`)
        .then(({ data }) => {
          // updating pagination to reflect in the UI
          this.serverPagination = pagination
  
          // we also set (or update) rowsNumber
          this.serverPagination.rowsNumber = data.rowsNumber
  
          // then we update the rows with the fetched ones
          this.serverData = data.rows
  
          // finally we tell QTable to exit the "loading" state
          this.loading = false
        })
        .catch(error => {
          // there's an error... do SOMETHING
          console.log(error)
  
          // we tell QTable to exit the "loading" state
          this.loading = false
        })
    }
  },
  mounted () {
    // once mounted, we need to trigger the initial server data fetch
    this.request({
      pagination: this.serverPagination,
      filter: this.filter
    })
  }
}
```

Čo sme pridali:
* *loading* - ak je nastaveny na ``true``, prepne vizuálny stav ``q-table`` do stavu loading 
* *serverPagination* - (inicializačné) nastavenie stránkovania v ``q-table``, číslo stránky; celkový počet záznamov v DB; počet záznamov na stránku; názov stĺpca, podľa ktorého usporadúvame; a príznak true/false - či ide o DESC, resp. ASC
* *serverData* - údaje načítane zo servera - riadky s produktami

Ďalej sme pridali metódu ``request``. V nej prepneme *q-table* do stavu *loading*, pomocou knižnice *axios* načítavame asynchrónne (XHR) dáta zo servera/z našej služby *list* (metóda get). Ako vidíme v požiadavke sa nachádzajú aj query parametre, a to poradie aktuálnej stránky (od 1), počet riadkov na stránku, triedenie podľa stĺpca, resp. ako triedime. V prípade, že úspešne príjmeme dáta zo servera, priradíme ich do ``serverData`` a vypneme ``loading``. 


## Zmeny v službe /products/list

Keďže sme použili komponent *q-table*, musíme trochu prispôsobiť našu službu. 

Upravíme smerovanie (pridali sme page):
```php
Route::get('products/list/{page}', 'ProductController@list');
```

V controlleri použijeme fasádu *DB*:
```php
use Illuminate\Support\Facades\DB;
```

Naša metóda ``list`` bude vyzerať takto:
```php
public function list($page) {  
    // get rowsPerPage from query parameters (after ?), if not set => 5
    $rowsPerPage = request('rowsPerPage', 5);
  	  
    // get sortBy from query parameters (after ?), if not set => name
    $sortBy = request('sortBy', 'name');
  	  
    // get descending from query parameters (after ?), if not set => false 
    $descendingBool = request('descending', 'false');
    // convert descending true|false -> desc|asc
    $descending = $descendingBool === 'true' ? 'desc' : 'asc';
    
    // pagination
    // IFF rowsPerPage == 0, load ALL rows
    if ($rowsPerPage == 0) {
        // load all products from DB
        $products = DB::table('products')
            ->orderBy($sortBy, $descending)
            ->get();
    } else {
        $offset = ($page - 1) * $rowsPerPage;
		  
        // load products from DB
        $products = DB::table('products')
            ->orderBy($sortBy, $descending)
            ->offset($offset)
            ->limit($rowsPerPage)
            ->get();
    }
  
    // total number of rows -> for quasar data table pagination
    $rowsNumber = DB::table('products')->count();
    	
    return response()->json(['rows' => $products, 'rowsNumber' => $rowsNumber]);
}
```

V ``$page`` si posielame aktuálne čislo stránky, ďalej cez *helper* ``request`` načítavame parametre z url za ``?`` (i.e. podľa čoho usporadúvame a ako). Následne načítame dáta z DB a vrátime ich ako JSON. V *q-table* je [Rows per page: All] reprezentované hodnotou 0 (je to uvedené aj v dokumentácii) - v takom prípade vynecháme stránkovanie. Booleovskú hodnotu *descending* transformujeme na DESC, resp. ASC.

V tomto momente by malo fungovať načítanie produktov do *q-table* s plnohodnotným stránkovaním a usporadúvaním podľa *name*.


# KONIEC 2. ČASTI ... TO BE CONTINUED



