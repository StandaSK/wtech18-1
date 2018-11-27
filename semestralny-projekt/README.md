# Semestrálny projekt - eshop

## Zadanie
Vytvorte webovú aplikáciu - eshop, ktorá komplexne rieši nižšie definované prípady použitia vo vami zvolenej doméne (napr. elektro, oblečenie, obuv, nábytok). Presný rozsah a konkretizáciu prípadov použitia si dohodnete s Vašim cvičiacim na cvičení.


## Termíny odovzdania
* **Odovzdanie 1. fázy projektu: 2. týždeň - 29.9. do 23:59 v AIS, 4 body,** vytvorenie skíc jednotlivých stránok pre rozlíšenie podľa výberu (ideálne mobile first)
* **Odovzdanie 2. fázy projektu: 5. týždeň - 20.10. do 23:59 v AIS, 12 bodov (10 + 2 body),** vytvorenie responzívnych šablón (10 bodov); návrh dátového modelu a implementovaný relačný model (2 body) 
* **Kontrolný bod: 9. týždeň, binárne hodnotenie 0/4 body**  implementácia klientskej časti eshopu so zostavením na serveri (server-side rendering) s využitím PHP rámca (odporúčaný Laravel) - so všetkými funkciami podľa požiadaviek 
* **Odovzdanie 3. fázy projektu: 11. týždeň - 8.12. do 23:59 v AIS, 20 bodov** 
  * implementácia klientskej časti eshopu so zostavením na serveri (server-side rendering) s využitím PHP frameworku (odporúčaný Laravel); 
  * implementácia administrátorského panela/admin zóny so zostavením na klientovi (Single Page Application, client-side rendering) s využitím Vue rámca (odporúčaný Quasar) 
  * finálna dokumentácia


## Termíny prezentovania
V čase cvičení študent predvedie na svojom počítači svoje riešenie (fázy projektu), a to:
* **Prezentovanie 1. fázy projektu: 3. týždeň - 2. a 3.10.**
* **Prezentovanie 2. fázy projektu: 6. týždeň - 23. a 24.10.**
* **Prezentovanie finálneho projektu: 12. týždeň - 4. a 5.12.**


## Aplikácia - eshop
V poslednom kontrolnom termíne **aplikácia musí pozostávať aspoň z týchto častí**:
### Klientská časť - zostavenie na strane servera
* **hlavná/promo stránka** 
* **zobrazenie prehľadu všetkých produktov** z vybratej kategórie používateľom, s možnosťou základného filtrovania, rozumne stránkovaných (ak je to potrebné)
* **stránka s detailom o produkte** s možnosťou vloženia produktu do nákupného košíka
* **nákupný košík** s možnosťou vytvorenia objednávky
    * pozostávajúci aspoň z troch krokov, a to: sumarizácia daných položiek v košíku, výber dopravy a platby, dodacie údaje
* **prihlásenie a registrácia používateľa**

### Administrátorská časť - zostavenie na strane klienta (Single Page Application)
* **administrátorské rozhranie**
  * prihlásenie používateľa
  * pridanie/upravenie/vymazanie produktu

**Aplikácia musí realizovať tieto prípady použitia:**

**Klientská časť**
* zobrazenie prehľadu všetkých produktov z vybratej kategórie používateľom s možnosťou základného filtrovania, rozumne stránkovaných (ak je to potrebné)
* zobrazenie konkrétneho produktu - detail produktu
    * pridanie produktu do košíka
* zobrazenie nákupného košíka
    * zmena množstva pre daný produkt
    * odobratie produktu
    * výber dopravy
    * výber platby
    * zadanie dodacích údajov
    * dokončenie objednávky
* registrácia používateľa/zákazníka
* prihlásenie používateľa/zákazníka
* odhlásenie zákazníka

**Administrátorská časť**
* prihlásenie administrátora do administrátorského rozhrania eshopu
* odhlásenie administrátora z administrátorského rozhrania
* vytvorenie nového produktu administrátorom cez administrátorské rozhranie
* upravenie/vymazanie existujúceho produktu administrátorom cez administrátorské rozhranie

## Dátový model
V druhom kontrolnom termíne sa odovzdáva JPG (JPEG) obrázok logického dátového modelu reprezentovaného UML class diagramom a implementovaný model v SQL databáze - schéma SQL DDL, ktorým sa vytvára (odporúčaný PosgreSQL).

Vo štvrtom kontrolnom termíne sa odovzdáva kompletná databáza, a teda dáta aj schéma. Diagram logického a fyzického dátového modelu bude súčasťou dokumentácie.

## Spôsob odovzdávania
Výstupy všetkých kontrolných bodov sa odovzdávajú do AISu. Databáza sa odovzdáva kompletná (dáta aj schéma, resp. DDL).

Odovzdávajú sa všetky zdrojové kódy aplikácie, okrem samotných rámcov a knižníc z manažéra balíkov (composer, npm). V prípade, že študent modifikoval používanú knižnicu, je potrebné pribaliť aj zmenené knižnicu a uviesť zmenu s odôvodnením v dokumentácii.

Odovzdáva sa ZIP alebo RAR archív.


## Oneskorenie odovzdania
V kontrolnom termíne sa môže odovzdanie oneskoriť maximálne o 3 dni.

Za každý deň oneskoreného odovzdania je študentovi odobratých 25% bodov z pôvodného maxima (deň po termíne študent získa 3/4 bodov, dva dni po termíne 1/2, atď.) 

 Neskoršie odovzdanie nie je možné. Neodovzdanie niektorej časti projektu znamená nesplnenie podmienok absolvovania predmetu.
 
 
## Kontrolná fáza progresu implementácie
V kontrolnej fáze - v 9. týždni - sa očakáva implementovaná klientská časť aplikácie. Fáza je hodnotená 4 bodmi, a to binárne. Študent letmo predvedie cvičiacemu funkčnosť klientskej aplikácie s ohľadom na požadované prípady použitia. Ak aplikácia umožňuje realizovať všetky prípady použitia, študent získa 4 body.


## Implementačné prostredie
Odporúčané technológie:
* Klientská časť: PHP - Laravel rámec
* Aministrátorská časť: Vue.js - Quasar rámec
* PostgreSQL relačný databázový systém

Použitie iných technológií, napr. iný PHP rámec alebo relačný databázový systém je podmienené súhlasom cvičiaceho (jazyk PHP je povinný).


## Dokumentácia
Dokumentácia musí obsahovať minimálne tieto časti:
* zadanie
* vytvorené skice rozhrania 
* diagram logického a fyzického dátového modelu spolu s ich opisom
* opis návrhu / návrhové rozhodnutia
* prog. prostredie
* opis implementácie jednotlivých prípadov použitia, snímky obrazoviek (angl. screenshot, snapshot) konečného  rozhrania


## Spôsob hodnotenia
### 2. fáza
- šablóny vytvorené pre všetky požadované prípady použitia - 5b
- správne použitie HTML 5 elementov - 2b
- responzívny dizajn - 2b
- formátovanie a logická štruktúra zdrojového kódu,
konzistencia/jednotná konvencia pri názvosloví identifikátorov - 1b
- databázový model - 2b

**Študent musí vedieť vysvetliť ktorúkoľvek časť (kód) svojho riešenia.**