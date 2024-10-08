# Quasar: Použitie storu (Vuex) s TypeSriptom
V predošlej časti sme používali v Quasar projekte JavaScript. Ukážme si použitie storu v Quasar projekte s TypeScriptom. Prináša to určité rozdiely.
Vytvorme si nový Quasar projekt, pričom v sprievodcovi povoľme balíčky Vuex a TypeScript (odporúčam aj ESLint, používam preset Standard), component style vyberme "Options API", package manager NPM.

```js
 npm init quasar
```

Vytvorme nový Vuex store s názvom ``showcase`` príkazom:

```js
 quasar new store -f ts showcase   
```

V prečinku ``src/store`` sa vytvoril priečinok ``showcase`` so store modulom. Ak používame v projekte ESlint s presetom "standard", spustením nasledujúceho CLI príkazu fixneme problém s bodkočiarkami v novo-vytvorených súboroch store modulu:

```js
npm run lint -- --fix
```

Upravme ``state`` store modulu (súbor ``showcase/state.ts``) takto:

```js
export interface ExampleStateInterface {
  drawerState: boolean;
}

function state (): ExampleStateInterface {
  return {
    drawerState: true
  }
}

export default state
```

Pridali sme do ``state`` atribút ``drawerState`` typu boolean a zadefinovali rozhranie (``ExampleStateInterface``), ktoré ``state`` implementuje. Pridajme mutáciu ``updateDrawerState`` pre state atribút ``drawerState``  v súbore ``showcase/mutations.ts``:

```js
import { MutationTree } from 'vuex'
import { ExampleStateInterface } from './state'

const mutation: MutationTree<ExampleStateInterface> = {
  updateDrawerState (state: ExampleStateInterface, opened: boolean) {
    state.drawerState = opened
  }
}

export default mutation
```

Ďalej potrebujeme "zaregistrovať" náš nový store modul ``showcase``. V hlavnom súbore aplikačného storu ``/src/store/index.ts`` importujme store module ``showcase`` a rozhranie ``ExampleStateInterface``, ktoré sme zadefinovali v jeho ``state`` (na začiatku súboru, zvyčajne riadok 9):

```js
// ...
import showcase from './showcase'
import { ExampleStateInterface } from './showcase/state'
// import example from './module-example'
// import { ExampleStateInterface } from './module-example/state';
// ...
```

V rovnakom súbore nájdime export rozhrania ``StateInterface`` a pridajme ``showcase: ExampleStateInterface``:

```js
// ...
export interface StateInterface {
  // Define your own store structure, using submodules if needed
  // example: ExampleStateInterface;
  // Declared as unknown to avoid linting issue. Best to strongly type as per the line above.
  example: unknown
  showcase: ExampleStateInterface
}
// ...
```

Ako poslednú vec v predmetnom súbore zaregistrujme v inicializácii aplikačného storu modul ``showcase``:

```js
// ...
export default store(function (/* { ssrContext } */) {
  const Store = createStore<StateInterface>({
    modules: {
      // example
      showcase
    },
// ...
```

Nový store modul ``showcase`` použijeme v komponente ``src/layouts/MainLayout.vue``. V jeho skripte upravme ``defineComponent`` takto:

```js
export default defineComponent({
  name: 'MainLayout',

  components: {
    EssentialLink
  },

  data () {
    return {
      essentialLinks: linksList
    }
  },
  computed: {
    drawerState: {
      get () {
        return this.$store.state.showcase.drawerState
      },
      set (val) {
        this.$store.commit('showcase/updateDrawerState', val)
      }
    }
  }
})
```

Upravme udalosť ``@click`` tlačidla ``q-btn`` v hlavičke ``q-header`` layoutu:
```js
@click="drawerState = !drawerState"
```

a ``v-model`` komponentu ``q-drawer``:
```html
 <q-drawer v-model="drawerState" show-if-above bordered>
```

Hotovo, store nám funguje v Quasare s TypeScriptom.

# Quasar: Validácia vstupných polí

Na validáciu vstupných polí (formulárové polia) je možné použiť knižnicu [Vuelidate](https://vuelidate-next.netlify.app/) (knižnica, ktorú odporúčajú aj tvorcovia Quasaru v dokumentácii). V projekte Quasar s Typescriptom môžu maž začiatočníci problémy s jej použitím. Ukážme si, ako danú knižnicu použiť.

Vychádzajme z [Quasar projektu, ktorý je výsledkom predošlej časti](zdroje/c5-quasar-ts-store.zip).

Nainštalujme si najskôr príslušné balíčky do Quasar projektu:

```js
npm install @vuelidate/core @vuelidate/validators
```

Upravme šablónu komponentu ``src/pages/Index.vue`` takto:

```html
<template>
  <q-page class="row items-center justify-evenly">
    <q-form @submit.stop="onSubmit">
      <q-card class="q-pa-md">
        <q-card-section>
          <div class="text-h4">Hello TypeScript</div>
        </q-card-section>
        <q-card-section>
          <q-field bottom-slots dense>
            <q-input v-model="inputText" />
            <template v-slot:hint>
              Count: {{ count }}
            </template>
          </q-field>
        </q-card-section>
        <q-card-actions>
          <q-btn type="submit">Submit</q-btn>
          <q-btn @click="reset()">Reset</q-btn>
        </q-card-actions>
      </q-card>
    </q-form>
  </q-page>
</template>
```

a skript takto:

```js
<script lang="ts">
import { defineComponent } from 'vue'

interface State {
  inputText: string;
}

export default defineComponent({
  name: 'PageIndex',
  data: () : State => {
    return { inputText: '' }
  },
  methods: {
    reset () {
      this.inputText = ''
    }
  },
  computed: {
    count () {
      return this.inputText.length
    }
  }
})
</script>
```

(Zatiaľ išlo o [príklad z prednášky](/prednasky/zdroje/3p-ts-01-intro.pdf)). Povedzme, že chceme validovať vstupné pole ``inputText`` a to tak, že bude povinné (``required``), minimálna dĺžka reťazca (``minLength``) budú 2 znaky a maximálna dĺžka (``maxLength``) 8 znakov. Ak chceme použiť **Vuelidate knižnicu** v komponete (``Index.vue``), potrebujeme pridať ``import useVuelidate`` v skripte komponentu (``Index.vue``): 

```js
<script lang="ts">
import { defineComponent } from 'vue'
import useVuelidate from '@vuelidate/core'
import {
  minLength,
  maxLength,
  required
} from '@vuelidate/validators'
// ...
</script>
```

Vuelidate ponúka rôzne preddefinované "built-in" validačné pravidlá. Ich úplný zoznam môžeme [nájsť v dokumentácii knižnice](https://vuelidate-next.netlify.app/validators.html). V komponente importujeme tie "built-in" validačné pravidlá, ktoré použijeme, a to ``minLength``, ``maxLength`` a ``required`` (každé validčné pravidlo, ktoré použijeme musíme importovať). 

Vytvorme inštanciu validátora pridaním vlastnosti ``setup`` do definície komponentu ``defineComponent``:

```js
// ...
export default defineComponent({
  name: 'PageIndex',
  setup () {
    return { v$: useVuelidate({ $autoDirty: true }) }
  },
// ...
```

Zadefinujme validáciu (pravidlá) vstupného poľa ``inputText`` pridaním funkcie ``validations``:

```js
// ...
 computed: {
   count () {
     return this.inputText.length
   }
 },
 validations () {
    return {
      inputText: {
        required,
        minLength: minLength(2),
        maxLength: maxLength(8)
      }
    }
  }
```

Ďalej pridajme asynchrónnu funkciu ``onSubmit`` do ``methods``:

```js
  methods: {
    reset () {
      this.inputText = ''
    },
    async onSubmit () {
      const isFormCorrect = await this.v$.$validate()
      if (!isFormCorrect) {
        this.$q.notify({
          color: 'red-4',
          textColor: 'white',
          icon: 'warning',
          message: this.v$.$errors.map(e => e.$message).join()
        })
      }
    }
  },
  ```
V šablóne pridajme do ``<q-form>`` udalosť ``@submit`` (``v-on:submit``) s modifikátorom ``stop``. Hodnotou bude volanie funkcie ``onSubmit``:

```html
<q-form @submit.stop="onSubmit">
```
(pozn., o modifikátoroch sme si hovorili na prednáške o Vue.js. Modifikátor ``stop`` zastaví propagation/prešírenie udalosti. [Spomeňte si na ``bubbling`` vs ``capturing``.](https://javascript.info/bubbling-and-capturing))
  

Do ``<q-field>`` pridajme prop ``error``:

```html
<q-field bottom-slots dense :error="v$.inputText.$error">
  <q-input v-model="inputText"/>
  <template v-slot:hint>
    Count: {{ count }}
  </template>
</q-field>
...
```

Tým, že sme zadefinovali validáciu na vstupnom poli ``inputText`` dochádza reaktívne k revalidácii hodnoty poľa v súľade s definovanými validačnými pravidlami (použili sme 3) pri každej jednej zmene hodnoty poľa, a teda každom zadanom znaku od používateľa. Ak hodnota vo vstupnom poli nevyhovuje niektorému z validačných pravidiel, je nastavená premenná ``v$.inputText.$error`` na hodnotu ``true`` a ``q-field`` komponent zobrazí ikonu s výkričníkom (pozrite dokumentáciu Quasaru, [Vue Components / Form Components / Field (wrapper) a prop error](https://quasar.dev/vue-components/field)).

Keď používateľ klikne na tlačidlo "**Submit**" odošle sa formulár a zavolá sa funkcia ``onSubmit``.
Vo funkcii ``onSubmit`` deklarujeme koštantu ``isFormCorrect``, ktorá nadobudne hodnotu ``false``, ak sa vyskytne validačná chyba. 
Ak sa vyskytne validačná chyba, chceme používateľovi zobraziť notifikáciu s textom chyby. Používame [plugin Quasaru Notify](https://quasar.dev/quasar-plugins/notify). Ak chceme ``Notify plugin`` používať v projekte, musíme ho najskôr v konfiguračnom súbore ``/quasar.conf.js`` zaregistrovať:

```js
 // Quasar plugins
      plugins: ['Notify']
```

Ukázali sme si ako spravit validáciu nad formulárovým poľom. Podobne vieme spraviť validáciu akéhokoľvek poľa. Použiť môžeme niektoré z "built-in" validačných pravidiel, alebo si môžeme vytvoriť [vlastné validačné pravidlo](https://vuelidate-next.netlify.app/custom_validators.html).

[Výsledok si môžete stiahnúť](zdroje/c5-quasar-vuelidate.zip).
