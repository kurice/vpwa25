# Quasar: i18n - internacionalizácia, resp. viacjazyčná aplikácia

Užitočnou knižnicou, v prípade potreby viacjazyčnej aplikácie, je [VueI18n](https://kazupon.github.io/vue-i18n/). Ukážme si ako spraviť podporu slovenčiny a angličtiny v aplikácii. Pri vytváraní Quasar aplikácie vyberme (o.i.) v sprievodcovi **Vue-i18n**.

Vychádzame z komponentov z [poslednej ukážky](zdroje/c5-quasar-vuelidate.zip). V komponente ``src/layouts/MainLayout.vue`` deklarujme pole ``localeOptions``:

```js
// ...
data () {
  return {
    essentialLinks: linksList,
    localeOptions: [
      { value: 'en-US' as string, label: 'English' as string },
      { value: 'sk-SK' as string, label: 'Slovak' as string }
    ]
  }
},
// ...
```
Ide o pole reprezentujúce nami podporované jazyky v aplikácii. Názov jazyka je v atribúte ``label`` a kód jazyka je v atribúte ``value`` ([iso language codes](https://www.andiamo.co.uk/resources/iso-language-codes/)). Hodnota atribútu ``value`` bude **použitá aj pre názov priečinka, v ktorom budú definované prekladové kľúče pre daný jazyk**.

Vytvorme prepínač (klasický "dropdown"), cez ktorý budeme meniť jazyk aplikácie. Použijeme komponent Quasaru ``q-select``. Nahraďme v hlavičke layoutu ``q-header`` element ``div``:
```html
<div>Quasar v{{ $q.version }}</div>
```
komponentom ``q-select``:
```html
<q-select
    v-model="locale"
    :options="localeOptions"
    label="Select language..."
    dense
    borderless
    emit-value
    map-options
    options-dense
    class="q-ml-md"
    color="blue"
    label-color="white"
    style="min-width:150px"
/>
```
Nastavme prop ``options`` na pole ``localeOptions``. 

Pridajme v skripte import ``vue-i18n``:
```js
<script lang="ts">
import { useI18n } from 'vue-i18n'
import EssentialLink from 'components/EssentialLink.vue'
//...
 ```
 a vytvorme vue-i18n inštanciu ``locale``:
```js
// ...
  components: {
    EssentialLink
  },
  setup () {
    const { locale } = useI18n({ useScope: 'global' })
    return { locale }
  },
  data () {
      // ...
  }
//...
```
Túto inštanciu použijeme ako ``v-model`` v prepínači jazykov.  

Všetky reťazce, ktoré chceme mať viacjazyčné musíme nahradiť kľúčmi. Pre kľúče zadefinujeme prekladové reťazce v jednotlivých jazykoch. Použitie kľúčov je cez funkciu ``$t()``. Na ukážku, v komponente ``src/pages/Index.vue`` pridajme funkciu ``$t()`` takto:

```html
<q-card-section>
    <div class="text-h4">{{ $t('message.hello') }} TypeScript</div>
</q-card-section>
```

Vidíme, že parametrom funkcie ``$t()`` je kľúč ``message.hello``. Pre tento kľúč zadefinujme zvlášť prekladový reťazec pre angličtinu a zvlášť pre slovenčinu. V priečinku ``src/i18n`` je priečinok ``en-US`` (jedna z ``value`` v ``localeOptions``). V ňom je súbor ``index.ts``. Obsah tohto súboru nahraďme takto:

```js
export default {
  message: {
    hello: 'Hello'
  }
}
```
Zadefinovali sme pre angličtinu kľúč ``message.hello``, ktorý sa v používateľskom rozhraní nahradí reťazcom "Hello". Podobne zadefinujme kľúč ``message.hello`` pre slovenčinu. V priečinku ``src/i18n`` vytvorme priečinok ``sk-SK`` a v ňom súbor ``index.ts`` s týmto obsahom:

```js
export default {
  message: {
    hello: 'Ahoj'
  }
}
```

Na koniec ešte pridáme do súboru ``src/i18n/index.ts`` nový riadok, ktorý nám zaregistruje novo-vytvorený priečinok pre slovenčinu. Celý súbor by mal následne vyzerať takto:

```js
import enUS from './en-US';
import skSK from './sk-SK';

export default {
    'en-US': enUS,
    'sk-SK': skSK
}
```

Hotovo. Po prepnutí jazyka sa zmení reťazec "Hello" na "Ahoj" a opačne. Možnosti použitia knižnice vue-i18n sú rôzne. Niekto napr. preferuje definovať prekladové kľúče a ich hodnoty pre podporované jazyky [priamo v komponentoch](https://kazupon.github.io/vue-i18n/guide/component.html#component-based-localization). Knižnica tiež poskytuje podporu pre prácu s množným číslom ([pluralization](https://kazupon.github.io/vue-i18n/guide/pluralization.html#pluralization)) a ďalšie vychytávky. Odporúčam spraviť si širší prehľad možností z dokumentácie.

[Riešenie na stiahnutie.](zdroje/c6-quasar-vuei18n.zip)