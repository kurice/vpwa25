# Vytvorenie prvého Vue.js projektu - krok za krokom

##  Prerekvizity
Odporúčam vývojové prostredie (IDE) [Visual Studio Code](https://code.visualstudio.com/) s rozšírením [Volar](https://marketplace.visualstudio.com/items?itemName=johnsoncodehk.volar).
Konečné rozhodnutie s ohľadom na vaše preferencie je samozrejme na vás.

## Vytvorenie prvého Vue.js projektu

**1**. Ak ešte nemáte, [stiahnite a nainštalujte si Node.js](https://nodejs.org/en/download/). ktorý zároveň obsahuje NPM (Node Package Manager). NPM je správca balíčkov (modulov/knižníc) pre JavaScript vo všeobecnosti. Node.js sme si predstavili na prednáške.

**2.** Použitím NPM si vytvoríme Vue.js projekt cez command-line (CLI):
```js
npm init vue@latest 
```

**3.** Po spustení príkazu si inštalátor vypýta názov projektu, pomenujme ho napr. ``MyFirstProject``
Zvyšné možnosti na začiatok ponechajme na hodnote ``No`` a potvrďme. 


**4.** Nastavme sa do priečinku s našim projektom a nainštalujme závislostí.
```js
cd MyFirstProject 
```
Všimnime si, že v priečinku sa nenachádza priečinok ``node_modules``.


Príkazom ``npm install`` nainštalujeme závislosti (externé knižnice/balíčky), ktoré naša aplikácia používa. Zoznam bezprostredných balíčkov nájdeme v súbore ``package.json``. Keď sa po spustení príkazu pozrieme do novo-vytvoreného priečinka ``node_modules`` uvidíme viacero priečinkov, ktoré reprezetujú externé závislosti (knižnice/balíčky). Je ich preto viac ako v našom súbore ``package.json``, lebo každý balíček môže vyžadovať ďalšie závislosti (vo svojom súbore ``package.json``). Príkaz ``npm install`` sa postará o rekurzívne nainštalovanie závislostí. 

Za povšimnutie stojí tiež súbor ``package-lock.json``. Tento súbor sa vytvorí po vykonaní príkazu ``npm install``. V ``package.json`` sú hlavné balíčky, zatiaľ čo ``package-lock.json``obsahuje už celý strom závislostí (je to podrobnejší obraz všetkých závislostí pre náš projekt).  

Súbor ``package-lock.json`` needitujeme ručne. Zmeny sa realizujú cez príkaz ``npm`` (pridávaním/odoberaním závislostí do projektu).

Prosim, vo voľnom čase sa začnite [oboznamovať širšie s NPM](https://css-tricks.com/a-complete-beginners-guide-to-npm/).


**5.** Spustime aplikáciu (na deve).
```js
npm run dev
```
Vykonaním príkazu sa zostaví aplikácia v režime dev a inicializuje Node.js dev server na localhoste. Aplikáciu otvoríme v prehliadači priamo kliknutím na odkaz localhostu v termináli (napr. ``http://localhost:3000/``), alebo zadaním predmetnej adresy do pola s URL adresou vo webvom prehliadači.

Pre informáciu, nasledujúciim príkazom vytvoríme produkčné zostavenie (build) našej aplikácie, ktoré sa uloží do priečinku ``./dist``. 
```js
npm run build
```

## Vytvorenie jednoduchého počítadla kliknutí

Nastavme sa do priečinku ``src``.

Súbor ``index.html`` je HTML dokument (kostra/skelet), do ktorého sa zostavuje/vykresľuje (renderuje) naša Vue.js aplikácia. 

Kľúčový je element
```html
<div id="app"></div>
```
do ktorého sa bude vykresľovať koreňový komponent našej prvej Vue.js aplikácie. Tento element reprezentuje aplikačný kontajner.

Súbor ``main.js``  obsahuje príkaz na vytvorenie inštancie Vue.js aplikácie:
```js
createApp(App).mount('#app')
```

Selektor ``#app`` určuje element (aplikačný kontajner), do ktorého sa bude naša aplikácia vykresľovať (prostredníctvom funkcie ``mount``). V prípade potreby môžeme identifikátor (príp. aj element) kontajnera samozrejme zmeniť. 

Za povšimnutie tiež stojí parameter ``create(App)``. Týmto parametrom určujeme, ktorý komponent je koreňovým komponentom našej aplikácie. V tomto prípade ide o kompoent reprezentovaný súborom ``App.vue`` (pozn., spomeňme si, čo sme si hovorili o komponentovo-riadenom vývoji a špeciálne o [SFC - Single File Components](https://vuejs.org/guide/scaling-up/sfc.html))

Na ukážku premenujme súbor ``App.vue`` na ``RootComponent.vue`` a upravme ``main.js`` súbor:

```js
import RootComponent from './RootComponent.vue'
createApp(RootComponent).mount('#app')
```

Direktívou ``import`` "sprístupňujeme" komponent v aplikačnom kóde. Za direktívou je alias komponentu a následne cesta k súboru s komponentom. Všimnime si, že sme upravili direktívu ``inmport`` tak, aby aj alias, pod ktorým sa budeme odkazovať na premenovaný komponent reflektoval názov komponentu. Pozn., alias a názov súboru komponentu sú dve rozdielne veci a záleži, ako si zvolíme konvenciu pomenovania komponentov v projekte (tzv. naming conventions, [mali by sme byť konzistentní naprieč celým projektom](https://medium.com/swlh/best-practices-for-writing-vue-apps-component-naming-and-organization-6c1593a251a0)).

Ak máme stále spustenú aplikáciu v režime dev cez príkaz ``npm run dev``, tak uloženie súboru vykoná automatické prekreslenie aplikácie. Akákoľvek zmena sa prejavuje v reálnom čase (nie je potrebný "reload" vo webovom prehliadači). 


Vytvorme v priečinku ``src`` súbor ``App.vue``. Pôjde o súbor reprezentujúci koreňový komponent našej jednoduchej aplikácie - Počítadlo kliknutí.

Obsah súboru je takýto:

```js
<script>
import ButtonCounter from './components/ButtonCounter.vue'

export default {
  components: {
    ButtonCounter
  }
}
</script>

<template>
  <div>
    <h1>Počítadlo kliknutí</h1>
    <ButtonCounter></ButtonCounter>
  </div>
</template>
```

Vidíme, že súbor obsahuje triviálnu logiku komponentu (v elemente ``<script>``) a šablónu (v elemente ``<template>``). Šablóna obsahuje HTML element ``<h1>`` a komponent ``<ButtonCounter>``. Tento komponent musíme vytvoriť. 

V logike komponentu je direktíva na importovanie komponentu ``ButtonCounter``. Aby sme mohli importovaný komponent ``ButtonCounter`` použiť v komponente ``App``, musíme ho zaregistrovať použitím Vue vlastnosti  ``components: {}``. Pozn., názov registrovaného komponentu je jeho alias.  

V priečinku ``src/components`` vytvorme súbor ``ButtonCounter.vue``, ktorého obsah je takýto:

```js
<script>
export default {
  data() {
    return {
      count: 0
    }
  }
}
</script>

<template>
  <button @click="count++">Klikol si na mna {{ count }}-krát.</button>
</template>
```

Vidíme, že náš další SFC (Single-File-Component) má logiku a šablónu (štýly zatiaľ neriešime). V logike už máme aj model komponentu, reprezentovaný JavaScript (JS) funkciou ``data()``. Táto funkcia je zároveň objektom (špecifikum jazyka JS) s atribútom ``count`` inicializovanom na hodnotu 0. 

Šablóna obsahuje tlačidlo - HTML element ``<button>`` - s Vue.js direktívou ``v-on:click`` alebo skrátene iba ``@ciick``. Po kliknutí na tlačidlo sa inkrementuje model - atribút ``count``. Hodnota atribútu sa vykresľuje v obsahu elementu ``<button>`` cez tzv. Mustache syntax ``{{ count }}``. Spomeňte si na prednášku, kde sme si vysvetľovali koncept reaktivity - jednosmerné, obojsmerné previazanie (tzv. One-way and Two-way Data Binding). 


## Lokálne vs Globálne zaregistrovanie komponentu
Príkazom ``import`` sme do komponentu ``App`` zahrnuli komponent ``ButtonCounter`` a cez Vue.js vlastnosť ``components: {}`` sme ho zaregistrovali. Ide o lokálne zaregistrovanie komponentu ``ButtonCounter`` v komponente ``App``, aby sme ho mohli použiť. Komponent môžeme zaregistrovať aj na globálnej úrovni (na úrovni celej aplikácie). V takom prípade bude prístupný (môžeme ho použiť v ľubovoľnom komponente) bez nutnosti jeho lokálnej registrácie. 

V súbore ``main.js`` vykonajme takúto zmenu:

```js
import { createApp } from 'vue'
import App from './App.vue'
import ButtonCounter from './components/ButtonCounter.vue'

const app = createApp(App);
app.component("ButtonCounter", ButtonCounter);
app.mount("#app");
```

Na globálnej urovni registrujeme komponent ``ButtonCounter``. Najskôr sa vytvára  inštancia aplikácie ``const app = createApp(App);``. Následne registrujeme komponent ``app.component("ButtonCounter", ButtonCounter);``. Nakoniec pripojíme inštanciu aplikácie k aplikačnému kontajneru na vykresľovanie. 

V komponente ``App`` už nie je potrebná lokálna registrácia komponentu. Upravíme súbor ``App.vue``  takto:

```js
<script>
export default {
}
</script>

<template>
  <h1>Počítadlo kliknutí</h1>
  <ButtonCounter></ButtonCounter>
</template>
```

Pozor, ak si globálne zaregistrujeme komponent, ale nikde ho nepoužijeme, stále bude obsiahnutý vo výslednom zostavení (build) aplikácie. Globálnu registráciu komponentov treba používať uvážlivo. Vo veľkých aplikáciách spôsobuje, že sú vzťahy menej explicitné/zjavné. Sťažuje nájdenie implementácie podriadeného komponentu z nadriadeného komponentu. V prípade veľkého množstva globálnych komponentov môže byť údržba aplikačného kódu stažená/náročná. 


## Props - odovzdanie dát z nadriadeného podriadenému komponentu

Upravme v komponente ``ButtonCounter`` šablónu takto:

```js
<script>
export default {
  data() {
    return {
      count: 0,
      title: 'Klikol si na mňa',
    }
  }
}
</script>

<template>
  <button @click="count++">{{ title }} {{ count }}-krát.</button>
</template>
```

Vidíme, že obsah tlačidla obsahuje atribút ``title``. Hodnotu tohto atribútu ("Klikol si na mňa") nechceme definovať (staticky) v modeli komponentu (objekt ``data()``), ale chceme aby ju odovzdal komponentu nadradený komponent, v našom prípade ``App``. Použijeme na to Vue.js vlastnosť ``props``, v ktorej zadefinujeme atribút ``title``.

Upravme komponent ``ButtonCounter`` takto:

```js
<script>
export default {
  data() {
    return {
      count: 0
    }
  },
  props: {
    title: String
  }
}
</script>

<template>
  <button @click="count++">{{ title }} {{ count }}-krát.</button>
</template>
```

V komponente ``App`` na odovzdanie hodnoty "Klikol si na mňa" komponentu ``ButtonCounter`` použijeme atribút ``title``:

```js
<script>
export default {
  data() {
    return {
    }
  }
}
</script>

<template>
  <h1>Počítadlo kliknutí</h1>
  <ButtonCounter title="Klikol si na mňa"></ButtonCounter>
</template>
```

**Na úvod všetko.**  

