# 3. časť: vymazanie produktu


## Vymazanie produktu
Chceme, aby sa po kliknutí na vymazať produkt (v zozname produktov) zobrazil dialóg potvrdzujúci vymazanie , a aby sa následne zobrazila notifikačná správa informujúca o vymazaní predmetného produktu. 

Použijeme pluginy Quasaru [Dialog](https://quasar-framework.org/components/dialog.html) a [Notify](https://quasar-framework.org/components/notify.html). Do súboru `quasar.conf.js` pridajte plugin *Dialog* (*Notify* by tam už mal byť):

```js
  // Quasar plugins
  plugins: [
    'Notify',
    'Dialog'
  ]
```

V komponente `Index` naviažme na tlačidlo *Delete* na udalosť `@click` zavolania metódy `destroy`. Ako parametre odovzdáme *id* a *name*.

```html
<q-btn round icon="delete" @click="destroy(props.row.id, props.row.name, props.row.__index)"/>
```

Zadefinujme metódu `destroy`:
```js
methods: {
  request ({ pagination }) {...},
  destroy (id, name) {
    this.$q.dialog({
      title: 'Delete',
      message: 'Are you sure to delete ' + name + '?',
      color: 'primary',
      ok: true,
      cancel: true
    }).then(() => {
      this.$q.notify({type: 'positive', timeout: 2000, message: 'The product has been deleted.'})
    }).catch(() => {
      // cancel - do nothing?
    })
  }
```
V metóde vyvoláme dialogové okno (s príslušným nastavením a správami). Ide o typický *promise*, teda definujeme, čo sa má vykonať v prípade úspechu (resolve - kliknutie na tlačidlo OK), a čo v prípade odmietnutia (reject - kliknutie na tlačidlo Cancel). V našom prípade sa po kliknutí na tlačidlo *OK* zobrazí na 2s pozitívna notifikácia so správou `The product has been deleted`.

Potrebujeme samozrejme vytvoriť na BE službu, ktorá zrealizuje vymazanie.

Do `routes/web.php` pridajme:
```php
Route::delete('products/{product}', 'ProductController@destroy');
```

V controlleri `ProductController.php` vytvorme metódu destroy:
```php
public function destroy(Product $product)
{
    $product->delete();
    // error handling is up to you!!! ;)
    return response()->json(['status'=>'success','msg' => 'Product deleted successfully']);
}
```

Upravme metódu `destroy` v komponente `Index`:
```js
destroy (id, name, rowIndex) {
      this.$q.dialog({
        title: 'Delete',
        message: 'Are you sure to delete ' + name + '?',
        color: 'primary',
        ok: true,
        cancel: true
      }).then(() => {
        axios
          .delete(`http://127.0.0.1:8000/products/${id}`)
          .then(() => {
            this.serverData[rowIndex].id = 'DELETED'
            this.$q.notify({type: 'positive', timeout: 2000, message: 'The product has been deleted.'})
          })
          .catch(error => {
            this.$q.notify({type: 'negative', timeout: 2000, message: 'An error has been occured.'})
            console.log(error)
          })
      }).catch(() => {
        // cancel - do nothing?
      })
    }
  },
```

Použili sme metódu `delete` (*axios.delete*), voláme endpoint `/products/${id}`. 
Za povšimnutie stojí kód `this.serverData[rowIndex].id = 'DELETED'`. Problém je, že keď na BE vymažeme daný záznam, na FE v *q-table* (`serverData`) zostane. Riešenie je - aktualizovať `serverData` pre aktuálnu stránku tabuľky - a teda pri zmazaní produktu si vypýtame zo servera aj nový zoznam produktov pre danú stránku. V našom riešení iba označíme daný záznam za vymazaný. Uvedené (krajšie) riešenie ponechám na vás.

Potrebujeme spraviť malé zmeny v šablóne komponentu `Index`:
```html
<template>
<div class="q-my-xl">
<q-table
      :data="serverData"
      row-key="name"
      :pagination.sync="serverPagination"
      :loading="loading"
      @request="request"
      :columns="columns"
      title="List of products"
      binary-state-sort
      >
      <q-tr slot="body" slot-scope="props" :props="props">
        <q-td key="id" :props="props">
          <span>{{ props.row.id }}</span>
        </q-td>
        <q-td key="name" :props="props">
          <span>{{ props.row.name }}</span>
        </q-td>
        <q-td class="text-right">
          <div v-if="props.row.id == 'DELETED'">DELETED</div>
          <div v-else>
            <q-btn round icon="edit" class="q-mr-xs" @click="$router.push('/products/' + props.row.id + '/edit')" />
            <q-btn round icon="delete" @click="destroy(props.row.id, props.row.name, props.row.__index)"/>
          </div>
        </q-td>
      </q-tr>
    </q-table>
</div>
</template>
```
V poslednom stĺpci sme pridali podmienku `v-if`, resp. `v-else`.

### Problém zvaný CSRF
O [CSRF](https://developer.mozilla.org/en-US/docs/Glossary/CSRF) sme si hovorili na prednáške.  
Problém je, že keď sa teraz pokúsime nejaký produkt vymazať, Laravel nám vráti 419 (unknown status). Je to preto, že Laravel pri `delete` metóde očakáva `csrf_token`, ktorý nám (pôvodne) poskytol. Je potrebné mu ho pri požiadavke poslať , čím sa "legitimizujeme". Spomeňte si - v Blade šablóne sme pri formulároch museli uviesť `{{ csrf_field() }}`. Pri každom odoslaní predmetného formulára sme teda zároveň posielali Laravelu aj `csrf token`, ktorý nám on sám pri zostavení formulára poskytol - vygeneroval.

Problém je, že v našom prípade rozhranie zostavuje Quasar (Vue.js) - Laravel sa nijako nepričiňuje, inými slovami naša Quasar aplikácia nemá k dispozícii `csrf token` od Laravelu pri jej zostavovaní. Quasar dokonca vytvára server (`quasar dev`). Lenže, ak `csrf token` Laravelu neposkytneme pri požiadavkách na služby, nemôže naša aplikácia zrealizovať `delete` (ani `post`, atď., okrem `get`). 

Jedno z rýchlych riešení je, že pojdeme do súboru `app/Http/Kernel.php`, kde zakomentujeme  middleware`\App\Http\Middleware\VerifyCsrfToken::class` z:
```php
protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            // \Illuminate\Session\Middleware\AuthenticateSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            // \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
            \App\Http\Middleware\Language::class,
        ],
        ...
    ];
``` 
Laravel prestane robiť verifikáciu `csrf tokenu`. Toto môžeme spraviť maximálne v DEV prostredí!!! V produkcii určite nie, nie, nie!!!

Ďalšia možnosť je, že našu Quasar aplikáciu zbuildujeme a tento build bude súčasťou Laravel aplikácie. Zároveň nastavíme Laravel tak, aby nám zostavoval - servíroval skelet pre našu Quasar aplikáciu, ktorého súčasťou bude vygenerovaný `csrf token`. Ukážme si, ako na to...

V našej Quasar aplikácii v priečinku `src/` otvorme súbor `index.template.html`.
Je to HTML skelet našej Quasar aplikácie. Do `<div id="q-app"></div>` sa zostavuje naša Quasar aplikácia - je to element koreňového komponentu. 

Upravme túto šablónu tak, že do nej pridáme `csrf token` - ten nám bude poskytovať (generovať) Laravel, ktorý ho očakáva pri HTTP požiadavkách (post, delete...):
```html
<!DOCTYPE html>
<html>
  <head>
    <title><%= htmlWebpackPlugin.options.productName %></title>

    <meta charset="utf-8">
    <meta name="description" content="<%= htmlWebpackPlugin.options.productDescription %>">
    <meta name="format-detection" content="telephone=no">
    <meta name="msapplication-tap-highlight" content="no">
    <meta name="viewport" content="user-scalable=no, initial-scale=1, maximum-scale=1, minimum-scale=1, width=device-width<% if (htmlWebpackPlugin.options.ctx.mode.cordova) { %>, viewport-fit=cover<% } %>">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <link rel="icon" href="statics/quasar-logo.png" type="image/x-icon">
    <link rel="icon" type="image/png" sizes="32x32" href="statics/icons/favicon-32x32.png">
    <link rel="icon" type="image/png" sizes="16x16" href="statics/icons/favicon-16x16.png">
  </head>
  <body>
    <!-- DO NOT touch the following DIV -->
    <div id="q-app"></div>
  </body>
  <script>
    window.Laravel = <?php echo json_encode([
    'csrfToken' => csrf_token(),
    ]); ?>
  </script>
</html>
```

Chceme, aby nám Laravel - keď zavoláme endpoint `http://127.0.0.1:8000` - vrátil HTML dokument založený na tejto šablóne a vygeneroval do nej `csrf token`. Budeme mať tak k dispozícii skelet našej Quasar aplikácie a zároveň `csrf token`, ktorý môže Quasar aplikácia posielať pri každej požiadavke Laravel službám. 

Vidíme ale, že v šablóne nie sú žiadne skripty s našou Quasar aplikáciou, a teda prehliadač nemá čo zostaviť (akú Quasar aplikáciu zostaví?). Skripty na Quasar aplikáciu sa pridajú do šablony po jej zbuildovaní.

Keď spustíte aplikáciu v *dev* režime `quasar dev` a otvoríte si obsah HTML dokumentu, tak môžeme vidieť niečo takéto:
```html
<!DOCTYPE html>
<html>
  <head>
    <title>Quasar App</title>

    <meta charset="utf-8">
    <meta name="description" content="A Quasar Framework app">
    <meta name="format-detection" content="telephone=no">
    <meta name="msapplication-tap-highlight" content="no">
    <meta name="viewport" content="user-scalable=no, initial-scale=1, maximum-scale=1, minimum-scale=1, width=device-width">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <link rel="icon" href="statics/quasar-logo.png" type="image/x-icon">
    <link rel="icon" type="image/png" sizes="32x32" href="statics/icons/favicon-32x32.png">
    <link rel="icon" type="image/png" sizes="16x16" href="statics/icons/favicon-16x16.png">
  </head>
  <body>
    <!-- DO NOT touch the following DIV -->
    <div id="q-app"></div>
  <script type="text/javascript" src="app.js"></script></body>
  <script>
    window.Laravel = <?php echo json_encode([
    'csrfToken' => csrf_token(),
    ]); ?>
  </script>
```

Súčasťou je už `app.js` - naša Quasar aplikácia, ale tiež pridané CSRF fragmenty. To je to, čo sme chceli. Problém je, že daný dokument nám nezostavil Laravel, a teda stále nemáme vygenerovaný `csrf token`. Musíme teda zabezpečiť, aby nám tento dokument vrátil Laravel, a teda naša Quasar aplikácia bola súčasťou Laravel aplikácie.

Zbuildujme našu Quasar aplikáciu príkazom `quasar build`. V priečinku našej aplikácie by sa nám mál vytvoriť priečinok `dist`.

V Laraveli do `routes/web.php` pridáme:
```php
Route::view('/', 'welcome');
```

V `resources/views/` vytvoríme šablónu `welcome.blade.php`. Do tejto šablóny skopírujeme obsah súboru `dist/spa-mat/index.html`, ktorý je takýto:
```html
<!DOCTYPE html>
<html>
   <head>
      <title>Quasar App</title>
      <meta charset=utf-8>
      <meta name=description content="A Quasar Framework app">
      <meta name=format-detection content="telephone=no">
      <meta name=msapplication-tap-highlight content=no>
      <meta name=viewport content="user-scalable=no,initial-scale=1,maximum-scale=1,minimum-scale=1,width=device-width">
      <meta name=csrf-token content="{{ csrf_token() }}">
      <link rel=icon href=statics/quasar-logo.png type=image/x-icon>
      <link rel=icon type=image/png sizes=32x32 href=statics/icons/favicon-32x32.png>
      <link rel=icon type=image/png sizes=16x16 href=statics/icons/favicon-16x16.png>
      <link as=style href=css/app.7bcf48a1.css rel=preload>
      <link as=script href=js/app.b777c49b.js rel=preload>
      <link as=script href=js/runtime.953d23ae.js rel=preload>
      <link as=script href=js/vendor.092f200f.js rel=preload>
      <link href=css/5fc3a329.358a428a.css rel=prefetch>
      <link href=css/cfa1e154.358a428a.css rel=prefetch>
      <link href=js/2d0deaf3.dc707870.js rel=prefetch>
      <link href=js/2d0e9741.1b162c53.js rel=prefetch>
      <link href=js/2d226763.f902fbeb.js rel=prefetch>
      <link href=js/4b47640d.21cfc388.js rel=prefetch>
      <link href=js/5fc3a329.e024b0df.js rel=prefetch>
      <link href=js/cfa1e154.53ca33cc.js rel=prefetch>
      <link href=css/app.7bcf48a1.css rel=stylesheet>
   </head>
   <body>
      <div id=q-app></div>
      <script type=text/javascript src=js/app.b777c49b.js></script><script type=text/javascript src=js/runtime.953d23ae.js></script><script type=text/javascript src=js/vendor.092f200f.js></script>
   </body>
   <script>window.Laravel = <?php echo json_encode([
      'csrfToken' => csrf_token(),
      ]); ?> </script>
</html>
```

Ako vidíme, je to šablóna skeletu našej Quasar aplikácie, ktorá sa pri builde "rozdrobila" na viacero menších skriptov. Je tam ale všetko, čo treba a aj fragmenty pre `csrf token`. 

Z priečinka `dist/spa-mat/` prekopírujte všetky priečinky (css, fonts, js, statics) do priečinka `public/` v Laravel aplikácii. 

Týmto sme docielili, že keď zavoláme `http://127.0.0.1:8000` Laravel použije šablónu `welcome.blade.php`, zostaví HTML dokument (skelet Quasar aplikácie) s `csrf` tokenom - vygeneruje ho, prehliadač po stiahnutí HTML dokumentu posťahuje všetky zdroje - zostaví Quasar aplikáciu. Máme SPA Quasar aplikáciu využívajúcu služby Laravelu, ktorá už má k dispozícii aj `csrf token`.  Otvorte si obsah HTML dokumentu, ktorý vrátil Laravel - obsahuje token:

```html
<!DOCTYPE html>
<html>
   <head>
      <title>Quasar App</title>
      <meta charset=utf-8>
      <meta name=description content="A Quasar Framework app">
      <meta name=format-detection content="telephone=no">
      <meta name=msapplication-tap-highlight content=no>
      <meta name=viewport content="user-scalable=no,initial-scale=1,maximum-scale=1,minimum-scale=1,width=device-width">
      <meta name=csrf-token content="hsROvb8UjtPRhgKA7g6sf7jkOZT2RA5fWsM2stsF">
      <link rel=icon href=statics/quasar-logo.png type=image/x-icon>
      <link rel=icon type=image/png sizes=32x32 href=statics/icons/favicon-32x32.png>
      <link rel=icon type=image/png sizes=16x16 href=statics/icons/favicon-16x16.png>
      <link as=style href=css/app.7bcf48a1.css rel=preload>
      <link as=script href=js/app.b777c49b.js rel=preload>
      <link as=script href=js/runtime.953d23ae.js rel=preload>
      <link as=script href=js/vendor.092f200f.js rel=preload>
      <link href=css/5fc3a329.358a428a.css rel=prefetch>
      <link href=css/cfa1e154.358a428a.css rel=prefetch>
      <link href=js/2d0deaf3.dc707870.js rel=prefetch>
      <link href=js/2d0e9741.1b162c53.js rel=prefetch>
      <link href=js/2d226763.f902fbeb.js rel=prefetch>
      <link href=js/4b47640d.21cfc388.js rel=prefetch>
      <link href=js/5fc3a329.e024b0df.js rel=prefetch>
      <link href=js/cfa1e154.53ca33cc.js rel=prefetch>
      <link href=css/app.7bcf48a1.css rel=stylesheet>
   </head>
   <body>
      <div id=q-app></div>
      <script type=text/javascript src=js/app.b777c49b.js></script><script type=text/javascript src=js/runtime.953d23ae.js></script><script type=text/javascript src=js/vendor.092f200f.js></script>
   </body>
   <script>window.Laravel = {"csrfToken":"hsROvb8UjtPRhgKA7g6sf7jkOZT2RA5fWsM2stsF"} </script>
</html>
```

Pripomínam, keď ste v DEV prostredí, môžete CSRF DOČASNE v Laraveli vypnúť. Keď máte aplikáciu hotovú, vytvoríte produkčnú verziu Quasar aplikácie (`quasar build`), vložíte ju do Laravel aplikácie a CSRF zapnete. 


Celý proces sa dá aj zautomatizovať a mať Quasar dev verziu aplikácie ako súčasť Laravelu - pogooglite trochu ;) 
BTW: Keď presuniete Quasar aplikáciu k Laravelu, bežia na rovnakej doméne a odpadá problém s CORS. 

Otázkou je, čo v prípade, keď Quasar aplikácia a Laravel služby (backend) nebudú spolunažívať. V tomto prípade vystavíme naše služby doslova ako API. Keď definujeme smerovanie našich služieb v `routes/web.php`, potom je všetkým *smerovaniam* priradená `web` middleware skupina definovaná v `Kernel.php`, kde sa nachádza aj spomínaná verifikácia `CSRF`, a teda smerovania definované vo `web.php` musia prejsť aj touto verifikáciou. V prípade ale, že vystavíme naše služby ako API, máme možnosť definovať smerovanie v `routes/api.php`. Tieto smerovania spadajú pod `api` middleware skupinu. Keď si pozrieme súbor `Kernel.php`, v `api` skupine nie je registrovaná žiadna `CSRF` verifikácia, a teda nie je potrebný `csrf token`.

# KONIEC 3. ČASTI ... TO BE CONTINUED