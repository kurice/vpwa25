# Slek Lite - 2. časť: vytvorenie autentifikačného aparátu na klientovi a server API

**Obsah**:
* [Vytvorenie komponentov register/login a zadefinovanie smerovania](#anchor1-components)   
    * [Komponent MainLayout](#anchor11-layout)  
    * [Komponent RegisterPage](#anchor12-register)  
    * [Routes](#anchor13-routes) 
    * [Komponent LoginPage](#anchor14-login) 
    * [Komponent ChannelPage](#anchor15-channel) 
* [Vytvorenie aparátu na autentifikáciu používateľa na klientovi (slek-client)](#anchor2-createc)    
    * [Autentifikačné služby](#anchor21-services)    
        * [AuthManager](#anchor211-authmanager)  
        * [AuthService](#anchor212-authservice)  
    * [Kontrakty](#anchor22-contracts)  
    * [Boot súbory axios a auth](#anchor23-boot)  
        * [Boot súbor axios](#anchor231-bootaxios)  
        * [Boot súbor auth](#anchor232-bootauth)  
   * [Store module auth](#anchor24-store)   
        * [Store module auth: state](#anchor241-state)  
        * [Store module auth: mutácie](#anchor242-mut)  
        * [Store module auth: gettre](#anchor243-get)  
        * [Store module auth: akcie](#anchor244-act)  
    * [Doplnenie konfiguračného súboru](#anchor25-conf)  
* [Vytvorenie aparátu na autentifikáciu používateľa na serveri (slek-server)](#anchor31-creates)  


## <a name="anchor1-components"></a> Vytvorenie komponentov register/login a zadefinovanie smerovania

V Quasar appke (slek-client) vytvorme základné komponenty používateľského rozhrania pre registráciu (register) a prihlasovanie (login) používateľa.

### <a name="anchor11-layout"></a> Komponent MainLayout

V priečinku ``src/layouts`` otvorme súbor (komponent) ``MainLayout.vue`` a upravme jeho obsah takto:

```vue
<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-toolbar-title>
          Slek Lite
        </q-toolbar-title>

        <div>Quasar v{{ $q.version }}</div>
      </q-toolbar>
    </q-header>
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'MainLayout'
})
</script>
```

### <a name="anchor12-register"></a> Komponent RegisterPage

V priečinku ``src/pages`` vytvorme súbor (komponent) ``RegisterPage.vue`` s týmto obsahom:

```vue
<template>
<q-page class="row items-center justify-evenly">
  <q-card square style="width: 400px; padding:50px">
    <q-card-section>
      <div class="text-h6">
        Register
      </div>
    </q-card-section>

    <q-form ref="form" class="q-gutter-md">
      <q-card-section>
        <q-input
          name="email"
          id="email"
          v-model.trim="form.email"
          type="email"
          label="Email"
          autofocus
        />
        <q-input
          id="password"
          name="password"
          v-model="form.password"
          label="Password"
          :type="showPassword ? 'text' : 'password'"
          bottom-slots
        >
          <template v-slot:append>
            <q-icon
              :name="showPassword ? 'visibility' : 'visibility_off'"
              class="cursor-pointer"
              @click="showPassword = !showPassword"
            />
          </template>
        </q-input>
        <q-input
          id="password_confirmation"
          name="password_confirmation"
          v-model="form.passwordConfirmation"
          label="Confirm Password"
          :type="showPassword ? 'text' : 'password'"
          bottom-slots
        >
          <template v-slot:append>
            <q-icon
              :name="showPassword ? 'visibility' : 'visibility_off'"
              class="cursor-pointer"
              @click="showPassword = !showPassword"
            />
          </template>
        </q-input>
      </q-card-section>

      <q-card-actions align="between">
        <q-btn label="Login" size="sm" flat :to="{ name: 'login' }"></q-btn>
        <q-btn
          label="Register"
          color="primary"
          :loading="loading"
          @click="onSubmit"
        />
      </q-card-actions>
    </q-form>
  </q-card>
</q-page>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { RouteLocationRaw } from 'vue-router'

export default defineComponent({
  name: 'RegisterPage',
  data () {
    return {
      form: { email: '', password: '', passwordConfirmation: '' },
      showPassword: false
    }
  },
  computed: {
    redirectTo (): RouteLocationRaw {
      return { name: 'login' }
    },
    loading (): boolean {
      return this.$store.state.auth.status === 'pending'
    }
  },
  methods: {
    onSubmit () {
      this.$store.dispatch('auth/register', this.form).then(() => this.$router.push(this.redirectTo))
    }
  }
})
</script>
```

Komponent obsahuje formulár na registráciu používateľa, konkrétne vstupné polia (email a heslo) a tlačidlo na odoslanie formulára. K logike komponentu sa vrátime kúsok ďalej.

### <a name="anchor13-routes"></a> Routes

Zadefinujme routovanie. V priečinku ``src/router`` otvorme súbor ``routes.ts`` a zadefinujme tieto cesty:

```ts
  {
    path: '/',
    // try redirect to home route
    redirect: () => ({ name: 'home' })
  },
  {
    path: '/auth',
    component: () => import('layouts/MainLayout.vue'),
    children: [
      { path: 'register', name: 'register', meta: { guestOnly: true }, component: () => import('pages/RegisterPage.vue') },
      { path: 'login', name: 'login', meta: { guestOnly: true }, component: () => import('pages/LoginPage.vue') }
    ]
  },
  {
    path: '/channels',
    // channels requires auth
    meta: { requiresAuth: true },
    component: () => import('layouts/MainLayout.vue'),
    children: [
      { path: '', name: 'home', component: () => import('src/pages/ChannelPage.vue') }
    ]
  },
```

Vidíme, že koreňovú cestu ``/`` presmerovávame na ``home`` route. Routa s názvom ``home`` (``name: 'home'``) je definovaná v ceste ``path: '/channels'``.  Atribút ``requiresAuth`` určuje, že cesta ``'/channels'`` vyžaduje autentifikáciu, teda je prístupná iba prihlásenému používateľovi. Preto, ak používateľ nie je prihlásený, budeme ho chcieť presmerovať na cestu ``/auth/login``. Inými slovami, ku kanálom sa vie dostať až prihlásený používateľ.

### <a name="anchor14-login"></a> Komponent LoginPage

V priečinku ``src/pages`` vytvorme súbor (komponent) ``LoginPage.vue`` s týmto obsahom:
```vue
<template>
<q-page class="row items-center justify-evenly">
  <q-card square style="width: 400px; padding:50px">
    <q-card-section>
      <div class="text-h6">
        Login
      </div>
    </q-card-section>

    <q-form ref="form" class="q-gutter-md">
      <q-card-section>
        <q-input
          name="email"
          id="email"
          v-model.trim="credentials.email"
          type="email"
          label="Email"
          autofocus
        />
        <q-input
          id="password"
          name="password"
          v-model="credentials.password"
          label="Password"
          :type="showPassword ? 'text' : 'password'"
          bottom-slots
        >
          <template v-slot:append>
            <q-icon
              :name="showPassword ? 'visibility' : 'visibility_off'"
              class="cursor-pointer"
              @click="showPassword = !showPassword"
            />
          </template>
        </q-input>
        <q-checkbox
          id="rememberMe"
          v-model="credentials.remember"
          label="Remember me"
        />
      </q-card-section>

      <q-card-actions align="between">
        <q-btn label="Create account" size="sm" flat :to="{ name: 'register' }"></q-btn>
        <q-btn
          label="Login"
          color="primary"
          :loading="loading"
          @click="onSubmit"
        />
      </q-card-actions>
    </q-form>
  </q-card>
</q-page>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { RouteLocationRaw } from 'vue-router'

export default defineComponent({
  name: 'LoginPage',
  data () {
    return {
      credentials: { email: '', password: '', remember: false },
      showPassword: false
    }
  },
  computed: {
    redirectTo (): RouteLocationRaw {
      return (this.$route.query.redirect as string) || { name: 'home' }
    },
    loading (): boolean {
      return this.$store.state.auth.status === 'pending'
    }
  },
  methods: {
    onSubmit () {
      this.$store.dispatch('auth/login', this.credentials).then(() => this.$router.push(this.redirectTo))
    }
  }
})
</script>
```

Komponent obsahuje formulár na prihlásenie používateľa, konkrétne vstupné polia (email a heslo) a tlačidlo na odoslanie formulára. K logike komponentu sa vrátime kúsok ďalej.

### <a name="anchor15-channel"></a> Komponent ChannelPage

V priečinku ``src/pages`` vytvorme súbor (komponent) ``ChannelPage.vue`` s týmto obsahom (neskôr ho nahradíme plnohodnotným obsahom):
```vue
<template>
<q-page class="row items-center justify-evenly">
   channel page content coming soon
</q-page>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'ChannelPage'
})
</script>
```

## <a name="anchor2-createc"></a> Vytvorenie aparátu na autentifikáciu používateľa na klientovi (slek-client)

### <a name="anchor21-services"></a> Autentifikačné služby

Vytvoríme dve služby, ktoré budú realizovať klúčové úlohy autentifikácie na klientovi. Ide o služby ``AuthManager`` a ``AuthService``.  

#### <a name="anchor211-authmanager"></a> AuthManager 

Úlohou služby ``AuthManager`` je manažment nad API tokenom, ktorý je klientovi pridelený serverom po úspešnom prihlásení. Manažment zahŕňa uloženie tokenu do local storage na klientovi a obsluhu pri zmene tokena. Túto službu môžeme považovať za "knižnicu as is" a nie je potrebné jej venovať priestor. Ak nepotrebujete, nerobte v nej zmeny. Mala by dobre poslúžiť tak ako je.

Vytvorme v priečinku ``src`` priečinok ``services`` a v ňom súbor ``AuthManager.ts `` s týmto obsahom:

```ts
import { LocalStorage } from 'quasar'

type ChangeListener = (newToken: string | null, oldToken: string | null) => void

// this handles token storing to localstorage and notifying about changes
// also triggereing listeners when storage key is changed from another browser tab
class AuthManager {
  private currentToken: string | null = this.getToken()
  private onChangeListeners: ChangeListener[] = []

  private storageListener = (evt: StorageEvent) => {
    if (evt.key !== this.storageKey) {
      return
    }

    this.notifyListeners()
  }

  constructor (private storageKey: string) {
    window.addEventListener('storage', this.storageListener)
  }

  private notifyListeners (): void {
    const newToken = this.getToken()

    if (this.currentToken === newToken) {
      return
    }

    this.onChangeListeners.forEach((fn) => fn(newToken, this.currentToken))
    this.currentToken = newToken
  }

  // ads listener for changes, returns unsubscribe function
  public onChange (listener: ChangeListener): () => void {
    this.onChangeListeners.push(listener)

    // call new listener with current token if we have one
    if (this.currentToken !== null) {
      window.setTimeout(() => listener(this.currentToken, null), 0)
    }

    return () => {
      const idx = this.onChangeListeners.indexOf(listener)

      if (idx >= 0) {
        this.onChangeListeners.splice(idx, 1)
      }
    }
  }

  // this is just shortcut for adding change listener which calls callback only when token was removed
  public onLogout (listener: () => void): () => void {
    return this.onChange((token) => {
      if (token === null) {
        listener()
      }
    })
  }

  public getToken (): string | null {
    return LocalStorage.getItem(this.storageKey)
  }

  public removeToken (): void {
    LocalStorage.remove(this.storageKey)
    this.notifyListeners()
  }

  // this is just an alias for removing the token from storage
  public logout (): void {
    return this.removeToken()
  }

  public setToken (token: string): void {
    LocalStorage.set(this.storageKey, token)
    this.notifyListeners()
  }
}

export default new AuthManager('auth_token')
```

#### <a name="anchor212-authservice"></a> AuthService

Úlohou služby ``AuthService`` je poskytnúť metódy na komunikáciu s autentifikačným server API cez HTTP protokol. Ide o metódy ``register``, ``login``, ``logout`` a ``me`` (informácie o aktuálne prihlásenom používateľovi). Služba používa kontrakty ``ApiToken``, ``LoginCredentials``, ``RegisterData`` a ``User`` (definície kontraktov nižšie).

Vytvorme v priečinku ``src/services`` súbor ``AuthService.ts`` s týmto obsahom:
```ts
import type { AxiosError, AxiosRequestConfig } from 'axios'
import type { ApiToken, LoginCredentials, RegisterData, User } from 'src/contracts'
import { api } from 'src/boot/axios'

class AuthService {
  async me (dontTriggerLogout = false): Promise<User | null> {
    return api.get(
      'auth/me',
      { dontTriggerLogout } as AxiosRequestConfig
    )
      .then((response) => response.data)
      .catch((error: AxiosError) => {
        if (error.response?.status === 401) {
          return null
        }

        return Promise.reject(error)
      })
  }

  async register (data: RegisterData): Promise<User> {
    const response = await api.post<User>('auth/register', data)
    return response.data
  }

  async login (credentials: LoginCredentials): Promise<ApiToken> {
    const response = await api.post<ApiToken>('auth/login', credentials)
    return response.data
  }

  async logout (): Promise<void> {
    await api.post('auth/logout')
  }
}

export default new AuthService()
```

Metódy služby vytvárajú požiadavky na rovnomenné API routes na serveri. Na vytváranie asynchrónnych HTTP požiadaviek používame knižnicu [Axios](https://github.com/axios/axios).

Napríklad, metóda ``register`` vytvára HTTP požiadavku typu POST (HTTP metóda POST) smerovanú na routu (endpoint) ``auth/register``, resp. ``http://localhost:3333/auth/register``. V požiadavke posielame informácie zo vstupných polí registračného formulára (``RegisterData``). Na serveri požiadavku obslúži rovnomenná metóda ``register`` controllera ``AuthController`` (``AuthController``, je podrobnejšie opísaný neskôr). Ak je registrácia úspešná, odpoveďou je zaregistrovaný používateľ, a teda objekt typu ``User``. 

Metóda ``login`` vytvára HTTP požiadavku typu POST na routu ``auth/login``. V požiadavke posielame informácie zo vstupných polí prihlasovacieho formulára. Na serveri požiadavku obslúži rovnomenná metóda ``login`` controllera ``AuthController``. Ak je prihlásenie úspešné, odpoveďou je ``ApiToken``. Pri ďalších požiadavkách bude klient posielať (v hlavičke požiadavky) pridelený API token a server overí (napr. ``auth`` middleware) jeho platnosť.

V priečinku ``src/services`` vytvorme súbor ``index.ts``, ktorého obsah je takýto:
```ts
export { default as authManager } from './AuthManager'
export { default as authService } from './AuthService'
```

### <a name="anchor22-contracts"></a> Kontrakty

Na klientovi používane viacero kontraktov (rozhraní). Spomenuli sme už napr. kontrakty ``User``,``RegisterData`` a ``ApiToken``. Vytvorme priečinok ``src/contracts`` a v ňom súbor ``Auth.ts`` s týmito kontraktmi:

```ts
export interface ApiToken {
  type: 'bearer'
  token: string
  expires_at?: string
  expires_in?: number
}

export interface RegisterData {
  email: string
  password: string
  passwordConfirmation: string
}

export interface LoginCredentials {
  email: string
  password: string
  remember: boolean
}

export interface User {
  id: number
  email: string
  createdAt: string,
  updatedAt: string
}
```

Vytvorme tiež v priečinku ``src/contracts`` súbor ``index.ts`` s týmto obsahom:
```ts
export * from './Auth'
```

### <a name="anchor23-boot"></a> Boot súbory axios a auth

V komplexnejších Quasar aplikáciach bežne potrebujeme spustiť nejaký kód ešte predtým, ako sa vytvorí koreňová Vue app inštancia. Za týmto účelom za zvyknú používať tzv. [**boot súbory**](https://quasar.dev/quasar-cli-webpack/boot-files). 
Boot súbory vytvárame v priečinku ``src/boot``. 

Môžeme vidieť, že sa v priečinku ``src/boot`` nachádza súbor ``axios.ts``. Boot súbor ``axios.ts`` vytvára inštanciu a sprístupňuje nám Axios API v našej Quasar aplikácii. Boot súbory je potrebné zaregistrovať v konfiguračnom súbore ``quasar.config.js``. 

```ts
boot: [
  'axios'
],
```

#### <a name="anchor231-bootaxios"></a> Boot súbor axios

Upravme súbor ``src/boot/axios.ts`` takto:
```ts
import { boot } from 'quasar/wrappers'
import axios, { AxiosInstance } from 'axios'
import { authManager } from 'src/services'

declare module '@vue/runtime-core' {
  interface ComponentCustomProperties {
    $axios: AxiosInstance
    $api: AxiosInstance
  }
}

// Be careful when using SSR for cross-request state pollution
// due to creating a Singleton instance here;
// If any client changes this (global) instance, it might be a
// good idea to move this instance creation inside of the
// "export default () => {}" function below (which runs individually
// for each client)
const api = axios.create({
  baseURL: process.env.API_URL,
  withCredentials: true,
  headers: {}
})

const DEBUG = process.env.NODE_ENV === 'development'

// add interceptor to add authorization header for api calls
api.interceptors.request.use(
  (config) => {
    const token = authManager.getToken()

    if (token !== null) {
      config.headers.Authorization = `Bearer ${token}`
    }

    if (DEBUG) {
      console.info('-> ', config)
    }

    return config
  },
  (error) => {
    if (DEBUG) {
      console.error('-> ', error)
    }

    return Promise.reject(error)
  }
)

// add interceptor for response to trigger logout
api.interceptors.response.use(
  (response) => {
    if (DEBUG) {
      console.info('<- ', response)
    }

    return response
  },
  (error) => {
    if (DEBUG) {
      console.error('<- ', error.response)
    }

    // server api request returned unathorized response so we trrigger logout
    if (error.response.status === 401 && !error.response.config.dontTriggerLogout) {
      authManager.logout()
    }

    return Promise.reject(error)
  }
)

export default boot(({ app }) => {
  // for use inside Vue files (Options API) through this.$axios and this.$api
  app.config.globalProperties.$axios = axios
  app.config.globalProperties.$api = api
})

export { api }
```

Boot **súbor Axiosu sme upravili** tak, aby klient v hlavičke každej požiadavky posielal pridelený API token (ak má). 

#### <a name="anchor232-bootauth"></a> Boot súbor auth

Boot aparát použijeme aj na prepojenie autentifikačného aparátu so smerovaním (routes) na strane klienta . 

Vytvorme boot súbor ``auth`` cez Qusar CLI:
```console
quasar new boot auth --format ts
```

V priečinku ``src/boot`` pribudne súbor ``auth.ts``. Zaregistrujme ho v konfiguračnom súbore ``quasar.config.js``:
```ts
boot: [
  'axios',
  'auth'
],
```

Otvorme boot súbor ``auth.ts``a upravme jeho obsah takto:
```ts
import { boot } from 'quasar/wrappers'
import { authManager } from 'src/services'
import { RouteLocationNormalized, RouteLocationRaw } from 'vue-router'

declare module 'vue-router' {
  interface RouteMeta {
    requiresAuth?: boolean,
    guestOnly?: boolean
  }
}

const loginRoute = (from: RouteLocationNormalized): RouteLocationRaw => {
  return {
    name: 'login',
    query: { redirect: from.fullPath }
  }
}

// this boot file wires together authentication handling with router
export default boot(({ router, store }) => {
  // if the token was removed from storage, redirect to login
  authManager.onLogout(() => {
    router.push(loginRoute(router.currentRoute.value))
  })

  // add route guard to check auth user
  router.beforeEach(async (to) => {
    const isAuthenticated = await store.dispatch('auth/check')

    // route requires authentication
    if (to.meta.requiresAuth && !isAuthenticated) {
      // check if logged in if not, redirect to login page
      return loginRoute(to)
    }

    // route is only for guests so redirect to home
    if (to.meta.guestOnly && isAuthenticated) {
      return { name: 'home' }
    }
  })
})
```

Vidíme, že registrujeme (``router.beforeEach``), aby pre každú požiadavku (route), ktorá vyžaduje autentifikáciu (``requiresAuth``) bola vykonaná kontrola (na serveri volanie API metódy``me``), či je používateľ riadne prihlásený (má platný API token). Keď sa vrátime k už zadefinovaným routam (v ``src/router/routes.ts``), vidáme, že ``meta`` atribútom ``requiresAuth`` predurčujeme, že predmetná route podlieha kontrole na autentifikáciu. Túto kontrolu realizujeme v podmienke ``if (to.meta.requiresAuth && !isAuthenticated)``.


### <a name="anchor24-store"></a> Store module auth

V boot súbore ``auth`` vidíme, že používame **store**. Okrem toho používajú store aj priamo komponenty používateľského rozhrania, napr. ``RegisterPage`` alebo ``LoginPage``.

V boot súbore ``auth`` používame akciu storu ``store.dispatch('auth/check')``. 
Úlohou akcie ``auth/check`` je overiť použitím služby ``authService.me()`` , či je používateľ prihlásený (``authService`` je inštancia ``src/services/AuthSerice.ts``).

V komponente ``src/Pages/RegisterPage.vue`` používame v metóde ``onSubmit`` akciu ``this.$store.dispatch('auth/register')``. Úlohou akcie ``auth/register`` je zaregistrovať používateľa volaním služby ``authService.register(form)``.

V priečinku ``src/store`` premenujme priečinok ``module-example`` na ``module-auth``. V súbore ``src/store/index.ts`` zaregistrujme store module ``auth``:
```ts
import auth from './module-auth'
import type { AuthStateInterface } from './module-auth/state'
...
export interface StateInterface {
  auth: AuthStateInterface
}
...
const Store = createStore<StateInterface>({
    modules: {
      auth
    },
...
```

#### <a name="anchor241-state"></a> Store module auth: state
 
Upravme obsah súboru ``module-auth/state.ts`` takto:
```ts
import { User } from 'src/contracts'

export interface AuthStateInterface {
  user: User | null,
  status: 'pending' | 'success' | 'error',
  errors: { message: string, field?: string }[]
}

function state (): AuthStateInterface {
  return {
    user: null,
    status: 'pending',
    errors: []
  }
}

export default state
```

``State`` store modulu ``auth`` implementuje rozhranie ``AuthStateInterface``.  State drží informácie o aktuálne prihlásenom používateľovi, status a chybové správy.  Mutáciami meníme ``status``, ktorý môže mať hodnotu "pending", "success", alebo "error".

#### <a name="anchor242-mut"></a> Store module auth: mutácie 

V store module ``auth`` zadefinujme mutácie, a teda upravme súbor ``mutations.ts`` takto:
```ts
import { User } from 'src/contracts'
import { MutationTree } from 'vuex'
import { AuthStateInterface } from './state'

const mutation: MutationTree<AuthStateInterface> = {
  AUTH_START (state) {
    state.status = 'pending'
    state.errors = []
  },
  AUTH_SUCCESS (state, user: User | null) {
    state.status = 'success'
    state.user = user
  },
  AUTH_ERROR (state, errors) {
    state.status = 'error'
    state.errors = errors
  }
}

export default mutation
```

#### <a name="anchor243-get"></a> Store module auth: gettre 

Súbor ``getters.ts`` upravme takto:
```ts
import { GetterTree } from 'vuex'
import { StateInterface } from '../index'
import { AuthStateInterface } from './state'

const getters: GetterTree<AuthStateInterface, StateInterface> = {
  isAuthenticated (context) {
    return context.user !== null
  }
}

export default getters
```

#### <a name="anchor244-act"></a> Store module auth: akcie 

Súbor ``actions.ts`` upravme takto:
```ts
import { ActionTree } from 'vuex'
import { StateInterface } from '../index'
import { AuthStateInterface } from './state'
import { authService, authManager } from 'src/services'
import { LoginCredentials, RegisterData } from 'src/contracts'

const actions: ActionTree<AuthStateInterface, StateInterface> = {
  async check ({ commit }) {
    try {
      commit('AUTH_START')
      const user = await authService.me()
      commit('AUTH_SUCCESS', user)
      return user !== null
    } catch (err) {
      commit('AUTH_ERROR', err)
      throw err
    }
  },
  async register ({ commit }, form: RegisterData) {
    try {
      commit('AUTH_START')
      const user = await authService.register(form)
      commit('AUTH_SUCCESS', null)
      return user
    } catch (err) {
      commit('AUTH_ERROR', err)
      throw err
    }
  },
  async login ({ commit }, credentials: LoginCredentials) {
    try {
      commit('AUTH_START')
      const apiToken = await authService.login(credentials)
      commit('AUTH_SUCCESS', null)
      // save api token to local storage and notify listeners
      authManager.setToken(apiToken.token)
      return apiToken
    } catch (err) {
      commit('AUTH_ERROR', err)
      throw err
    }
  },
  async logout ({ commit }) {
    try {
      commit('AUTH_START')
      await authService.logout()
      commit('AUTH_SUCCESS', null)
      // remove api token and notify listeners
      authManager.removeToken()
    } catch (err) {
      commit('AUTH_ERROR', err)
      throw err
    }
  }
}

export default actions
```

Vidíme, že akcie okrem toho, že volajú služby ``authManager`` a ``authService`` priebežne menia ``state`` store modulu, konkrétne ``status`` prostredníctvom mutácií. Napríklad, akcia ``login`` najskôr zmutuje ``status`` na **pending** (``commit('AUTH_START')``) a následne čaká na vykonanie asynchrónnej metódy ``authService.login(credentials)``. V prípade úspechu (fulfilled) je ``status`` zmutovaný na **success** (``commit('AUTH_SUCCESS', null)``). Ak nastane chyba (vyhodená výnimka), ``status`` je zmutovaný na **error** (``commit('AUTH_ERROR', err)``). 

Zmenu stavu (``status``) používame aj v komponentoch používateľského rozhrania. Napríklad, computed atribút ``loading`` v komponente s registračnym formulárom:
```js
<q-btn
    label="Register"
    color="primary"
    :loading="loading"
    @click="onSubmit"
/>

...

loading (): boolean {
   return this.$store.state.auth.status === 'pending'
}
```
Keď používateľ klikne na tlačidlo "Register", akcia ``auth/register`` zmutuje``status`` na **pending** a computed atribút ``loading`` sa nastaví na hodnotu ``true``. Tlačidlo "Register" sa po vizuálnej stránke prepne do režimu "loading" (zobrazí sa točiaci sa spinner). Ak úspešne prebehne registrácia, zmutuje ``status`` na **success** a computed atribút ``loading`` sa zmení na hodnotu ``false``, čím sa dostane tlačidlo do pôvodného stavu.

### <a name="anchor25-conf"></a> Doplnenie konfiguračného súboru

Upravme konfiguračný súbor Quasaru ``quasar.config.js``. Pridajme atribút ``sourceFiles`` a v ``build`` zmeňme hodnotu atribútu ``vueRouterMode`` a pridajme ``env``:
```js
...
sourceFiles: { store: 'src/store/index.ts' },
build: {
  vueRouterMode: 'history', // available values: 'hash', 'history'
  env: {
    API_URL: process.env.API_URL || (ctx.dev ? 'http://localhost:3333' : 'https://prod.api.com')
  },
...
```

## <a name="anchor31-creates"></a> Vytvorenie aparátu na autentifikáciu používateľa na serveri (slek-server)

Vytvorme ``AuthController``. V Adonis projekte (slek-server) vytvorme v priečinku ``app/Controllers/Http`` súbor ``AuthController.ts`` cez CLI:
```console
node ace make:controller Auth
```

Do ``AuthController`` pridajme tento kód:

```ts
import type { HttpContextContract } from '@ioc:Adonis/Core/HttpContext'
import Channel from 'App/Models/Channel'
import User from 'App/Models/User'
import RegisterUserValidator from 'App/Validators/RegisterUserValidator'

export default class AuthController {
  async register({ request }: HttpContextContract) {
    // if invalid, exception
    const data = await request.validate(RegisterUserValidator)
    const user = await User.create(data)
    // join user to general channel
    const general = await Channel.findByOrFail('name', 'general')
    await user.related('channels').attach([general.id])

    return user
  }

  async login({ auth, request }: HttpContextContract) {
    const email = request.input('email')
    const password = request.input('password')

    return auth.use('api').attempt(email, password)
  }

  async logout({ auth }: HttpContextContract) {
    return auth.use('api').logout()
  }

  async me({ auth }: HttpContextContract) {
    await auth.user!.load('channels')
    return auth.user
  }
}
```
``AuthController`` deklaruje metódy na registráciu používateľa, prihlásenie, odhlásenie, a informácie o aktuálne prihlásenom používateľovi.
V metóde ``register`` validujeme vstupné polia z registračného formulára, vytvárame inštanciu modelu používateľa (z dát z requestu) a prepájame tohto používateľa s pred-vytvoreným kanálom "general" (cez seeder). 

V metóde ``login`` sa overuje používateľ (či je používateľ identifikovaný emailom v DB a či sa heslá zhodujú). Ak je overenie úspešné, vráti sa klientovi API token. Pri každej ďalšej požiadavke bude klient posielať pridelený API token a server overí (napr. ``auth`` middleware) jeho platnosť (token môže byť invalidovaný, alebo po čase exspiruje).  

Vidíme, že v controlleri používame ``RegisterUserValidator``. Vytvorme ho cez CLI:
```console
node ace make:validator RegisterUser
 ```

V priečinku ``app/Validators`` vznikol súbor ``RegisterUserValidator.ts``, ktorého obsah upravme takto:
```ts
import { schema, rules } from '@ioc:Adonis/Core/Validator'
...
  public schema = schema.create({
    email: schema.string({}, [
      rules.email(),
      rules.unique({ table: 'users', column: 'email' })
    ]),
    password: schema.string({}, [
      rules.minLength(8),
      rules.confirmed('passwordConfirmation')
    ])
  })
...
```
Validačná schéma definuje pravidlá pre hodnoty vstupných polí ``email`` a ``password`` získaných z registračného formulára. [Viac informácií o validačných schémach v dokumentácií](https://docs.adonisjs.com/guides/validator/introduction).

Zadefinujme API routes pre autentifikáciu. Do súboru ``start/routes.ts`` pridajme:
```js
Route.group(() => {
  Route.post('register', 'AuthController.register')
  Route.post('login', 'AuthController.login')
  Route.post('logout', 'AuthController.logout').middleware('auth')
  Route.get('me', 'AuthController.me').middleware('auth')
}).prefix('auth')
```
Každú požiadavku cez metódu POST HTTP protokolu smerovanú na ``/auth/register`` obslúži ``AuthController``, konkrétne metóda ``register``. Ostatné podobne. Smerovania na ``/auth/logout`` a ``/auth/me`` majú navyše kontrolu, či má požiadavka oprávnenie, resp. či používateľ, ktorý požiadavku poslal je aktuálne prihlásený. Kontrolu oprávnenia zabezpečuje middleware ``auth``.

Týmto máme vytvorenú ukážku autentifikačného aparátu ako na klientovi, tak aj na serveri. Pre autentifikáciu sme použili HTTP protokol. Nabudúce pokračujeme vytvorením aparátu pre kanály a výmenu správ. Pre túto časť aplikácie nebudeme používať HTTP protokol, ale websockety.

Koniec druhej časti.
 
**[Zdrojový kód po druhej časti - slek-server a slek-client](../slek-part2.zip)**
