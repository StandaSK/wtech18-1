# Cvičenie 9 - JavaScript

## Príklady na precvičenie

### Príklad 1
Vyskúšajte si delegovanie udalosti prebublaním, ktoré ilustruje tento príklad:

```html 
<ul id="parent-list">
	<li id="item-1">Item 1</li>
	<li id="item-2">Item 2</li>
	<ul id="child-list">
	    <li id="item-2-1">Item 2-1</li>
	    <li id="item-2-2">Item 2-2</li>
	    <li id="item-2-3">Item 2-3</li>
	</ul>
	<li id="item-3">Item 3</li>
</ul>
```

```js
document.querySelector("#parent-list").addEventListener("click", function(e) {
    if(e.target && e.target.nodeName == "LI") {
        console.log("List item ", e.target.id, " was clicked!");
    }  
});
```

### Príklad 2
Pripomeňte si [Fetch API z prednášky](/prednasky/zdroje/10-wt-js-ajax-fetch-promises-storage-moduly-webpack-vuejs.pdf), [vrátane ukážky](/prednasky/zdroje/priklady-ajax-fetch.zip). 

V Jednoduchom manažéri úloh prerobte vymazanie úlohy tak, že použijete Fetch API. V zozname úloh po kliknutí na tlačidlo [Vymazať] sa vymaže daná úloha - vykoná sa `TaskController@destroy`. 

V prípade úspešnej akcie, vymazanie záznamu sa prejaví aj priamo v DOM, a teda dynamicky odoberiete element reprezentujúci danú úlohu (použite DOM API, metódu `removeChild`). 


### Príklad 3
Vyskúšajte si príklad s jednoduchou indexovanou DB (IndexedDB). 

Z prednášky si pripomeňte obsluhu `onblocked` a `onversionchange`. V novej karte na rovnakej doméne  (same origin) nasimulujte vytvorenie novej verzie databázy a ošetrite predmetnú obsluhu - v pôvodnej karte sa uzatvorí spojenie, v novej karte sa zmení schéma pôvodnej databázy (napr., pridajte tím, v ktorom daný jazdec F1 pôsobí).

```js
var indexedDB = window.indexedDB || window.mozIndexedDB || window.webkitIndexedDB || window.msIndexedDB || window.shimIndexedDB;
  
// open (or create) the database
var open = indexedDB.open("MyDatabase", 1);
  
// create the database schema
open.onupgradeneeded = function() {
    var db = open.result;
    var store = db.createObjectStore("MyObjectStore", {keyPath: "id"});
    var index = store.createIndex("NameIndex", ["name.last", "name.first"]);
};
 
open.onsuccess = function() {
    // a new transaction
    var db = open.result;
    var tx = db.transaction("MyObjectStore", "readwrite");
    var store = tx.objectStore("MyObjectStore");
    var index = store.index("NameIndex");
  
    // add some data
    store.put({id: 12345, name: {first: "Fernando", last: "Alonso"}, age: 36});
    store.put({id: 67890, name: {first: "Lewis", last: "Hamilton"}, age: 33});
         
    // query the data
    var getFernando = store.get(12345);
    // via index
    var getLewis = index.get(["Alonso", "Fernando"]);
  
    getFernando.onsuccess = function() {
        console.log(getJohn.result.name.first);  // "Fernando"
    };
  
    getLewis.onsuccess = function() {
        console.log(getBob.result.name.first);   // "Lewis"
    };
  
    // close the db when the transaction is done
    tx.oncomplete = function() {
        db.close();
    };
}
```

### Príklad 4
Prerobte tieto ES6 moduly do modulov vo formáte AMD (zavádzač RequireJS), a tiež CommonJS (zavádzač SystemJS)

```js
//------ lib.js ------
export const sqrt = Math.sqrt;
export function square(x) {
    return x * x;
}
export function diag(x, y) {
    return sqrt(square(x) + square(y));
}
  
//------ main.js ------
import { square, diag } from 'lib';
console.log(square(11)); // 121
console.log(diag(4, 3)); // 5
```


### Príklad 5
Vytvorte svoju prvú jednoduchú Vue.js aplikáciu - [počítadlo kliknutí na tlačídlo z prednášky](/prednasky/zdroje/9-wt-js-ajax-fetch-promises-storage-moduly-webpack-vuejs-uvod.pdf)

