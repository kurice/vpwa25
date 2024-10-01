# Vue.js - Dynamické props, previazanie, v-html direktíva, štýly

## Dynamické vs statické props

 Na predošlom cvičení sme si ukázali odovzdanie dát z nadriadeného komponentu ``App`` podriadenému komponentu ``ButtonCounter``. Zadefinovali sme atribút  ``title`` v špeciálnej Vue.js vlastnosti  ``props``. V nadriadenom komponente sme použili ako hodnotu atribútu reťazec (string) "Klikol si na mňa":

```html
   <ButtonCounter title="Klikol si na mňa"></ButtonCounter>
```

Ak odovzdávame reťazec, resp, statický text, potom hovoríme o statickej "prop". Ak chceme Vue.js povedať, že miesto reťazca má pracovať s hodnotou ako JavaScript výraz (JavaScript expression), potom musíme použiť direktívu ``v-bind``. 

V nadriadenom komponente zadefinujme v modeli atribút ``titleDynamicProp``. Použitím direktívy ``v-bind`` povieme Vue.js, že ``titleDynamicProp`` je JavaScript expression, a teda dynamická prop (premenná).

```js
<script>
export default {
  data() {
    return {
      titleDynamicProp: 'Klikol si na mňa'
    }
  }
}
</script>

<template>
  <h1>Počítadlo kliknutí</h1>
  <ButtonCounter v-bind:title="titleDynamicProp"></ButtonCounter>
</template>
```

Direktívu ``v-bind`` nie je potrebné explicitne písať. Môžeme používať skrátený zápis:
```html
<template>
  <h1>Počítadlo kliknutí</h1>
  <ButtonCounter :title="titleDynamicProp"></ButtonCounter>
</template>
```

## Jednosmerné vs obojsmerné previazanie

Ako sme si spomínali na prednáške, v reaktívnych rámcoch sú spravidla modely (JavaScript objekty, premenné) napojené na DOM (Document Object Model) jednosmerným previazaním (angl. one-way binding), alebo obojsmerným previazaním (angl. two-way binding). Zjednodušene povedané, pri jednosmernom previazaní je JavaScript premenná napojená na DOM. Pri obojsmernom previazaní sú dáta napojené z DOMu späť na JavaScript premennú.

Najlepšie to bude ilustrovať príklad. Pridajme do komponentu ButtonCounter HTML element input:

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
  <input type="number" v-model="count"/>
</template>
```

Veďla tlačidla "Klikol si na mňa..." nám pribudlo vstupné pole, v ktorom je hodnota 0. Pokiaľ hodnotu zmeníme vo vstupnom poli (napr. na 10), reaktívne sa zmení aj štítok (angl. label) tlačidla ("Klikol si na mňa 10-krát"). Keď klikneme na tlačidlo, atribút ``count`` sa inkrementuje a zmena sa reaktívne prejaví vo vstupnom poli. Toto funguje pomocou direktívy ``v-model``, ktorá zabezpečuje obojsmerné previazanie.

Ak direktívu zmeníme na ``v-bind``, potom môžeme vidieť, ako sa správa jednosmerné previazanie:

```html
<template>
  <button @click="count++">{{ title }} {{ count }}-krát.</button>
  <input type="number" v-bind:value="count"/>
</template>
```

 Zmenou hodnoty vo vstupnom poli sa nemení atribút ``count``, a preto sa štítok tlačidla nemení (týmto smerom nie je aktívne previazanie). Kliknutím na tlačidlo sa inkrementuje atribút ``count`` čím sa zmení štítok tlačidla a aj hodnota vo vstupnom poli. 

Direktívu ``v-bind`` používame na **jednosmerné previazanie**. Direktívu ``v-model`` na **obojsmerné previazanie**, a to spravidla s formulárovými poliami (prepínače, zaškrtávacie polia, textové polia, ...). 

## Mustache syntax vs v-html direktíva

Ukázali sme si, že atribúty z modelu vieme začleniť do obsahu elementov v šablónach cez tzv. mustache syntax  ``{{ }}``:

```html
<button @click="count++">{{ title }} {{ count }}-krát.</button>
```

Cez mustache syntax sú však dáta interpretované ako "plain-text". To znamená, že ak by hodnota obsahovala HTML elementy, tieto nie sú interpetované ako HTML elementy ale ako obyčajné reťazce. Upravme atribút ``titleDynamicProp`` v komponente ``App`` takto:

```js
<script>
export default {
  data() {
    return {
      titleDynamicProp: '<strong>Klikol si na mňa</strong>'
    }
  }
}
</script>
```

Vidíme, že štítok tlačidla obsahuje neželané HTML elementy. Použitím direktívy ``v-html`` zabezpečíme, aby bola hodnota interpetovaná ako HTML kód. Šablónu komponentu ``ButtonCounter`` upravíme takto:

```html
<template>
  <button @click="count++" v-html="title + ' ' + count"></button>
  <input type="number" v-bind:value="count"/>
</template>
```

## Naviazanie štýlov (css tried)

Pridajme do komponentu ``ButtonCounter`` štýly (element ``style``), ktoré definujú selektor na triedu ``.red``. Ďalej pridajme do modelu atribút ``isActive`` typu ``Boolean``:

```js
<script>
export default {
  data() {
    return {
      count: 0
    }
  },
  props: {
    title: String,
    isActive: Boolean
  }
}
</script>

<template>
  <button :class="{ red : isActive }" @click="count++" v-html="title + ' ' + count"></button>
  <input type="number" v-bind:value="count"/>
</template>

<style scoped>
.red { 
  color: red
}
</style>
```

Direktívu ``v-bind`` vieme elegantným spôsobom použiť na aplikáciu "určitých štýlov za určitých podmienok". To nám umožňuje vysokú flexibilitu a dynamiku pri dizajnovaní používateľských rozhraní. Výrazom ``:class="{ red : isActive }`` hovoríme, že CSS trieda ``.red`` bude naviazaná na element ``<button />``, ak má atribút ``isActive`` hodnotu ``true``. Inými slovami, text tlačidla bude červený, ak cez nadriadený komponent (``App``) odovzdáme hodnotu ``true`` cez dynamickú prop ``isActive`` komponentu ``ButtonCounter``. Upravme šablónu komponentu ``App`` takto:

```html
<template>
  <h1>Počítadlo kliknutí</h1>
  <ButtonCounter :isActive="true" v-bind:title="titleDynamicProp"></ButtonCounter>
</template>
```
