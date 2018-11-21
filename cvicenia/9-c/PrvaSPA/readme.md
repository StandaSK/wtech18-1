# Mojá prvá Single Page alikácia vo Vue.js s využitím Quasar rámca


## Vytvorenie čistého Quasar projektu

**1**. [Stiahnite a nainštalujte si NPM (Node Package Manager)](https://nodejs.org/en/download/)

**2.** Použitím NPM si nainštalujte Quasar CLI:
```js
npm install -g quasar-cli
```

**3.** Vytvorte si nový Quasar projekt:
```js
quasar init <folder_name>
```
Pri výzve "Check the features needed for your project" označte všetky možnosti, najmä však *Axios*.

Pozn.: nie je potrebné inštalovať Vue.js, je už priamou súčasťou Quasar projektu.

**4.** Quasar projekt spustíte z konzoly ``quasar dev``. Musíte sa nachádzať v priečinku projektu.


## Vytvorenie služby na BE v Laraveli, ktorá vráti zoznam používateľov v JSON formáte

Použijeme vytvorený Jednoduchý manažér úloh (Task Manager), ktorý sme si naprogramovali na predošlých cvičeniach.

**1.** V ``UserController.php`` vytvoríme metódu ``list``
```php
public function list() {
    return User::all()->toJson(JSON_PRETTY_PRINT);
}
```
**2.** V ``routes/web.php`` nastavíme smerovanie:
```php
Route::get('/users/list/', 'UserController@list');
```

## Vytvorenie jednoduchej SPA v Quasar, ktorá vypíše zoznam používateľov

**1.** V ``src/layouts/`` vytvorte súbor ``Main.vue``:
```html
<template>
  <q-layout>
    <q-layout-header>
      <q-toolbar>
        <q-toolbar-title>
          Ahoj
        </q-toolbar-title>
      </q-toolbar>
    </q-layout-header>

    <q-page-container>
      <!-- This is where pages get injected -->
      <router-view />
    </q-page-container>

  </q-layout>
</template>
```

**2.** V ``src/pages`` vytvorte súbor ``Users.vue``:
```html
<template>
  <q-page padding>
    <q-list highlight>
      <q-list-header>Users list</q-list-header>
        <q-item v-for="user in users" v-bind:key="user.name">
          <q-item-main :label="user.name" />
        </q-item>
    </q-list>
  </q-page>
</template>
  
<script>
import axios from 'axios'

export default {
  // name: 'PageName',
  data: function () {
    return {
      users: []
    }
  },
  created () {
    axios.get('http://localhost:8000/users/list')
      .then(response => {
        this.users = response.data
      })
      .catch(e => {
        this.errors.push(e)
      })
  }
}
</script>
  
<style>
</style>
```


**3.** V ``src/router/routes.js`` pridajte:
```js
import Main from 'layouts/Main'
import Users from 'pages/Users'
```
a prepíšte ``const routes=[...]`` na:

```js
const routes = [
  {
    path: '/',
    component: Main,
    children: [
      // users page
      {
        path: '/',
        component: Users
      }
    ]
  }
]
```
## Problém s CORS
Quasar aplikácia ma iný **Origin** ako Laravel BE aplikácia. Na to, aby sme mohli volať službu ``users/list`` potrebujeme povedať našej Laravel aplikácii, aby povolila **cross-origin**. [Viac si môžete prečítať na tejto stránke.](https://enable-cors.org/)  

V Laravel aplikácii v (Task Manageri) nainštalujme túto knižnicu:
 
``composer require barryvdh/laravel-cors``

V Laravel aplikácii v súbore``Http/Kernel.php`` pridajme do   ``protected $middleware = [...];`` nainštalovaný middleware:
  
``\Barryvdh\Cors\HandleCors::class,``