# 1. časť: vytvorenie základných šablón pre ADMIN rozhranie eshopu

Vytvoríme si príklad rozhranie v Quasare pre eshop. Bude pozostávať zo zoznamu produktov, a tiež rozhrania na pridanie, editovanie a vymazanie produktov. 
Produkt bude na ilustráciu pozostávať iba z mena, opisu a fotografií. Admin rohranie bude obsahovať aj stránku na prihlásenie administrátora.

## Vytvorenie čistého Quasar projektu

**1**. Ak ešte nemáte, [stiahnite a nainštalujte si NPM (Node Package Manager)](https://nodejs.org/en/download/)

**2.** Použitím NPM si nainštalujte Vue CLI a Quasar CLI:
```js
npm install -g vue-cli
npm install -g quasar-cli
```

**3.** Vytvorte si nový Quasar projekt:
```js
quasar init <folder_name>
```
Pri výzve "Check the features needed for your project" označte všetky možnosti, najmä však *Axios* a *vuex*.


**4.** Quasar projekt spustíte z konzoly ``quasar dev``. Musíte sa nachádzať v priečinku projektu.



## Vytvorenie Layoutu 


**1.** V priečinku ``src/layouts`` vytvorme súbor s názvom ``Main.vue`` /ide o komponenent layoutu/
 
**2.** Zadefinujeme šablónu ``<template>``. Použujeme prevažne Quasar komponenty ako ``<q-layout>``, ktorý ďalej môže obsahovať ``<q-layout-header>``, ``<q-layout-drawer>``, atď. [Viac informácií o komponente ``<q-layout>`` ako aj ostatných komponentoch Quasaru nájdete v dokumentácii na oficiálnej stránke](https://quasar-framework.org/components/layout.html). 

Nezabudnite, že všetky Quasarovské komponenty, ktoré používate je potrebné zadefinovať v súbore ``/quasar.conf``.

Môj súbor ``quasar.conf`` po 1. časti tutoriálu obsahuje tieto komponenty:

```js
framework: {
      components: [
        'QLayout',
        'QLayoutHeader',
        'QLayoutFooter',
        'QLayoutDrawer',
        'QPageContainer',
        'QPage',
        'QToolbar',
        'QToolbarTitle',
        'QBtn',
        'QIcon',
        'QList',
        'QListHeader',
        'QItem',
        'QItemMain',
        'QItemSide',
        'QCard',
        'QCardTitle',
        'QCardMain',
        'QCardMedia',
        'QCardSeparator',
        'QCardActions',
        'QField',
        'QInput',
        'QUploader',
        'QTable',
        'QTh',
        'QTr',
        'QTd',
        'QTableColumns'
      ],
``` 

Komponent layoutu ``Main`` vyzerá takto:

```html
<template>
<q-layout>

  <q-layout-header>
    <q-toolbar>
        <q-btn flat round dense icon="menu" @click="left = !left" class="q-mr-md"/>
        <q-toolbar-title>MyESHOP - ADMIN<span slot="subtitle">Some subtitle</span></q-toolbar-title>
    </q-toolbar>
  </q-layout-header>

  <q-layout-drawer v-model="left" side="left">
    <main-layout-drawer></main-layout-drawer>
  </q-layout-drawer>

  <q-page-container>
    <div class="row full-width justify-center">
      <router-view class="c-container" />
    </div>
  </q-page-container>

  <q-layout-footer>
    <div class="text-center q-pa-md">Copyright (C) 2018, Homer Simpson</div>
  </q-layout-footer>
</q-layout>
</template>
```

Obsahuje teda hlavičku, bočný panel, kontajner, do ktorého sa vykresluje hlavný obsah a pätičku. [Používam grid systém quasaru](https://quasar-framework.org/components/flex-css.html), napr. trieda ``.row``. Tiež budem často používať rôzne [helpre](https://quasar-framework.org/components/other-helper-classes.html), [helpre na pozicovanie](https://quasar-framework.org/components/positioning.html), [na uľahčenie práce s priestorom](https://quasar-framework.org/components/spacing.html), [s farbami](https://quasar-framework.org/components/color-palette.html), [s písmom](https://quasar-framework.org/components/typography.html), atď., pozrite dokumentáciu.

Použili sme napr. ``q-pa-md``, čo vytvorí *padding* zo všetkých strán o veľkosti *md*. Podobne ``q-mr-md`` vytvorí *margin* z pravej strany o veľkosti *md*. Quasarovské komponenty začínajú prefixom *q-*, nie však všetky helpre, napr. `full-width, text-center, text-weight-*`.

V súbore ``src/css/app.styl`` som si zadefinoval globálnu css triedu ``.c-container``:
```css
.c-container {
    width: 95%;
    max-width: 1150px;
}
```

**3.** Vytvoríme nový komponent ``<main-layout-drawer>``, ktorý bude v ``<q-layout-drawer>``. V tomto komponente budeme mať postranné menu.

Vytvorte v priečinku ``src/components/`` priečinok ``layouts`` a následne priečinok ``Main``. Vytvorte súbor s názvom ``Drawer.vue``, ktorého obsah bude takýto:

```html
<template>
<div class="bg-grey-1 full-height q-px-md q-py-lg">
    <div class="q-subheading q-mb-sm">Products</div>
    <q-list link class="no-border">
        <q-item class="q-body-1" to="/products/index">
            <q-item-main>List</q-item-main>
        </q-item>
        <q-item class="q-body-1" to="/products/create">
            <q-item-main>Create</q-item-main>
        </q-item>
    </q-list>
</div>
</template>
```

Používame komponent [``q-list``](https://quasar-framework.org/components/lists-and-list-items.html), ktorý v tomto prípade reprezentuje menu. Zatiaľ máme iba jednu kategóriu - *Products*, ktorá obsahuje položky *List* a *Create*. V prvom prípade sme po kliknutí presmerovaní na */products/index*, teda stránku so zoznamom produktov, v druhom prípade ide o stránku na vytvorenie nového produktu. 

Všimnite si, že používame ďalšie helpre, napr. ``bg-grey-1`` (background color grey stupeň 1), ``q-subheading`` (štýl písma), `no-border` (v ``q-list`` sme vypli predvolený rámik).

Uvedené smerovanie a stránky musíme zadefinovať. 

**4.** Predtým však ešte v layoute *Main* pod element `<template>` vložte element `<script>`. Na tomto mieste sa definuje prevažne správanie komponentu. Zatiaľ použijeme `import` na sprístupnenie komponentu *Drawer* v komponente (layoute) *Main*. 
```js
<script>
import MainLayoutDrawer from 'components/layouts/Main/Drawer.vue'
  
export default {
  components: {
    MainLayoutDrawer
  }
}
</script>
``` 
Bez tohto importu by nemobolo možné komponent *Drawer* použiť.

**5.** Vytvorme (zatiaľ) prázdne súbory s podstránkami, a teda v priečinku ``src/pages/`` vytvorte priečinok ``products`` a v ňom tieto súbory:
* *Create.vue*
* *Edit.vue*
* *Index.vue*
* *Show.vue*


## Vytvorenie smerovania (routes)

V súbore ``src/router/routes.js`` zadefinujeme takéto smerovanie:

```js
const routes = [
  {
    path: '/',
    redirect: () => {
      return { path: '/products/index' }
    }
  },
  {
    path: '/products',
    component: () => import('layouts/Main'),
    children: [
      {
        path: 'index',
        component: () => import('pages/products/Index')
      },
      {
        path: 'create',
        component: () => import('pages/products/Create')
      }
    ]
  },
  {
    path: '/products/:id',
    component: () => import('layouts/Main'),
    children: [
      {
        path: '',
        component: () => import('pages/products/Show')
      },
      {
        path: 'edit',
        component: () => import('pages/products/Edit')
      }
    ]
  }
]

// Always leave this as last one
if (process.env.MODE !== 'ssr') {
  routes.push({
    path: '*',
    component: () => import('pages/Error404.vue')
  })
}

export default routes

```

Prvý záznam nám definuje - ak smerujeme na endpoint `/`, tak sme presmerovaní na `/products/index`.
 
Druhý záznam - ak smerujeme na `/products`, potom použi layout `layouts/Main` a v závislosti od zvyšku cesty použi predmetnú stránku. Ak teda smerujeme na endpoint `/products/index` použi v  layoute *Main* stránku ``pages/products/Index``. Analogicky *Create*.
 
 Keď sa opäť lepšie pozrieme na náš layout *Main*, tak sa v ňom nachádza tento komponent:
 ```html
   <q-page-container>
     <div class="row full-width justify-center">
       <router-view class="c-container" />
     </div>
   </q-page-container>
 ```
 
 "Placeholder" ``<router-view>`` je špeciálny, do neho je zavedená predmetná stránka - v prípade smerovania na `/products/index` je teda zavedená stránka ``pages/products/Index``.
 
 Keď si pozornejšie prezriete ďalšie definície smerovania, môžete si všimnúť cestu ``/products/:id``. Ide o parameter, ku ktorému sa dá prístúpiť (napr. ``{{ $route.params.id }}``, viac neskôr).  Stránka `pages/products/Show` je zavedená do layoutu *Main* - napr. v prípade smerovania na `/products/24`, podobne  stránka `pages/products/Edit`, keď smerujeme na `/products/24/edit`.
 

## Vuex - store, resp. state management

Na prednáške sme si vysvetlili princíp `storu`.  
Keď si opäť pozrieme podrobnejšie náš layout *Main*, môžeme si všimnúť, že je v ňom tlačidlo, ktorého úlohou je zobrazovať a skrývať bočný panel s menu (má nastavený ``v-model="left"``):

```html
  <q-layout-header>
    <q-toolbar>
        <q-btn flat round dense icon="menu" @click="left = !left" class="q-mr-md"/>
        <q-toolbar-title>MyESHOP - ADMIN<span slot="subtitle">Some subtitle</span></q-toolbar-title>
    </q-toolbar>
  </q-layout-header>
 ```
 Na tlačidlo je najviazaná udalosť `v-on:click`, ktorá nastavuje premennú *left* na negáciu jej aktuálnej hodnoty. Túto premennú zadefinujeme v našom komponente (layoute) *Main*. 
 
```js
<script>
import MainLayoutDrawer from 'components/layouts/Main/Drawer.vue'

export default {
  components: {
    MainLayoutDrawer
  },
  computed: {
    left: {
      get () { return this.$store.state.moduleUI.layout.drawerState },
      set (val) { this.$store.commit('moduleUI/updateDrawerState', val) }
    }
  }
}
</script>
``` 
Vidíme, že používame vlastnosť `computed` a hodnota, resp. doslova stav premennej `left` je zdielaná v *store*. K stavu premennej pristupujeme cez *getter*, resp. zmenu realizujeme cez  *mutáciu*.

Vytvorme teda `store`, v ktorom je držaný stav premennej *left*. V priečinku `/src/store/` vytvorte priečinok `moduleUI`. Skopírujte súbory z priečinka `module-example` do `moduleUI`. 
V súbore `/src/store/index.js` vložte:

```js
import moduleUI from './moduleUI'
```

a `moduleUI` pridajte do:
```js
const Store = new Vuex.Store({
    modules: {
      moduleUI
    }
  })
```

Do súboru `moduleUI/state.js` vložte:
```js
export default {
  layout: {
    drawerState: true
  }
}
```

Do súboru `moduleUI/mutations.js` vložte:
```js
export const updateDrawerState = (state, opened) => {
  state.layout.drawerState = opened
}

```

To je všetko - stav bočného panelu (zobrazený/skrytý) je odteraz zdielaný a prostredníctvom *storu* k nemu môžu pristupovať ďalšie komponenty - podobne ako komponent (layout) *Main*.

Komponent *Drawer* má cez `v-model` naviazanú premennú `left`. V závislosti od aktuálnej hodnoty (true/false) sa teda skryje/zobrazí. Hodnotu meníme kliknutím na tlačidlo v hlavičke layoutu. Kliknutie vyvolá zmenu v *store*. Vďaka reaktivite je pomocou *gettera* každá zmena v premennej 'left' bezprostredne odzrkadlená. 

Je to ukážka vzájomnej "komunikácie" medzi komponentami prostredníctvom *storu*, bez toho, aby komponent *Main* volal akúkoľvek akciu komponentu *Drawer*. Zdielané dáta sú extrahované do *storu*. Komponenty tak nie sú na sebe závislé (nie sú nijako previazané).


## Vytvorenie podstránok

### Index

Vytvorme komponent (podstránku) `Index`. Do súboru `/src/pages/products/Index.vue` vložte:

```html
<template>
<div class="q-my-xl">
    <q-table
      :data="tableData"
      :columns="columns"
      row-key="id"
      title="List of products"
      >
      <q-tr slot="body" slot-scope="props" :props="props">
        <q-td key="id" :props="props">
          <span>{{ props.row.id }}</span>
        </q-td>
        <q-td key="name" :props="props">
          <span>{{ props.row.name }}</span>
        </q-td>
        <q-td class="text-right">
            <q-btn round icon="edit" class="q-mr-xs" @click="$router.push('/products/' + props.row.id + '/edit')" />
            <q-btn round icon="delete" />
        </q-td>
      </q-tr>
    </q-table>
</div>
</template>
 
<script>
import tableData from 'assets/table-data'
export default {
  data () {
    return {
      tableData,
      columns: [
        { name: 'id', label: 'ID', field: 'id', sortable: false, align: 'left' },
        { name: 'name', label: 'Name', field: 'name', sortable: true, align: 'left' },
        { name: 'actions', label: 'Actions', sortable: false, align: 'right' }
      ],
      selected: []
    }
  }
}
</script>
 ```
 
 Použili sme komponent [`<q-table>`](https://quasar-framework.org/components/datatable.html). Zadefinovali sme stĺpce, riadky, pričom jeden zo stĺpcov obsahuje tlačidlá - akcie - editovať a vymazať. 
 
 Vytvorili sme si tiež v `/assets/` testovací súbor `table-data.js`, ktorý obsahuje:
 ```json
 export default [
   {
     id: 1,
     name: 'Televízor'
   },
   {
     id: 2,
     name: 'Mikrovlnná rúra'
   }
 ]
 ```
 
 Všimnite si, že v akcii (tlačidle) *edit* využívame `props.row.id` na určenie smerovania endpointu pre predmetný produkt.
 
 
 ### Create
 
 Do súboru `/src/pages/products/Create.vue` vložte:
 
 ```html
 <template>
 <div class="q-my-xl">
     <q-card>
         <q-card-title>Create new product</q-card-title>
         <q-card-main>
             <q-field :count="250">
                 <q-input float-label="Name" max-length="250" />
             </q-field>
             <q-field :count="5000">
                 <q-input
                     type="textarea"
                     float-label="Description"
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
                 <q-btn label="Create" color="primary" />
             </div>
         </q-card-actions>
     </q-card>
 </div>
 </template>
  
 <style lang="stylus">
 .docs-btn .q-btn
     padding 15px 20px
 </style>
 ```
 
 Na nahrávanie obrázkov použijeme komponent [`<q-uploader>`](https://quasar-framework.org/components/uploader.html). Použijeme tiež komponent [`<q-card>`](https://quasar-framework.org/components/card.html). 
 
 
  ### Edit
  
  Do súboru `/src/pages/products/Edit.vue` vložte:
  
  ```html
  <template>
  <div class="q-my-xl">
      <q-card>
          <q-card-title>Create new product</q-card-title>
          <q-card-main>
              <q-field :count="250">
                  <q-input float-label="Name" max-length="250" />
              </q-field>
              <q-field :count="5000">
                  <q-input
                      type="textarea"
                      float-label="Description"
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
                  <q-btn label="Create" color="primary" />
              </div>
          </q-card-actions>
      </q-card>
  </div>
  </template>
   
  <style lang="stylus">
  .docs-btn .q-btn
      padding 15px 20px
  </style>
  ```
  
 # KONIEC 1. časti