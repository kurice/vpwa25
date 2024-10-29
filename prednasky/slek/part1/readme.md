# Slek Lite - 1. časť: vytvorenie projektov, nainštalovanie prerekvizít a vytvorenie databázových modelov

**Obsah**:

- [Vytvorenie nových projektov - klient (Quasar app) a server api (AdonisJS)](#anchor1-create)
  - [Klient](#anchor11-client)
  - [Server](#anchor12-server)
- [Inštalácia a konfigurácia prerekvizít slek-server](#anchor2-install)
- [Databázové modely](#anchor3-db)

## <a name="anchor1-create"></a> Vytvorenie nových projektov - klient (Quasar app) a server api (AdonisJS)

### <a name="anchor11-client"> Klient

Vytvorme si nový Quasar projekt CLI príkazmi:

```console
npm i -g @quasar/cli
npm init quasar
```

Vyberme možnosť **App with Quasar CLI**, priečinok projektu nastavme napríklad na "slek-client" a ďalej vyberme v sprievodcovi inštaláciou tieto možnosti:

- Quasar V2
- TypeScript
- Quasar App CLI with Webpack
- Options API
- Sass
- Eslint, State management (Vuex), Axios
- Eslint preset: Standard
- Install project dependencies? : "Yes, use npm"

Vyskúšajme, či nový projekt funguje:

```console
cd slek-client
quasar dev
```

Ak zbehne build bez zásadných chýb a demo aplikácia sa otvorí v prehliadači, máme kostru projektu - klientskej časti - pripravenú.

### <a name="anchor12-server"> Server

Vytvorme nový Adonis projekt CLI príkazom:

```console
npm init adonis-ts-app@latest slek-server
```

Vyberme projektovú štruktúru **api**. V sprievodcovi inštaláciou potvrďme pre eslint a prettier hodnoty "yes".

Vyskúšajme, spustiť server:

```console
cd slek-server
node ace serve --watch
```

Ak zbehne build bez zásadných chýb a demo sa otvorí v prehliadači, máme kostru projektu - serverovej časti - pripravenú.

## <a name="anchor2-install"> Inštalácia a konfigurácia prerekvizít slek-server

Do Adonis projektu doinštalujme balíček "Lucid". Ide o oficiálny SQL ORM (objektovo-relačný mapovač )pre Adonis. Budeme používať [SQLite databázu](https://www.sqlite.org/index.html), preto vyberme **driver SQLite**:

```console
npm i @adonisjs/lucid@18.4.0
node ace configure @adonisjs/lucid
```

Nainštalujme samotný SQLite DBMS:

```console
npm i sqlite3
```

Vytvorme v priečinku `tmp/` súbor `db.sqlite3` pre našu SQLite databázu:

```console
cd tmp && touch db.sqlite3
```

Nainštalujme [@adonisjs/auth](https://docs.adonisjs.com/guides/auth/introduction) balíček na autentifikáciu používateľov:

```console
npm i @adonisjs/auth npm i @adonisjs/auth@8.2.3
node ace configure @adonisjs/auth
```

V sprievodcovi konfiguráciou auth balička vyberme tieto možnosti:

- **Lucid** Data Models
- Select which guard you need for authentication: **api**
- Enter model name to be used for authentication: **User**
  - pozn.: nezabúdajme používať správne konvencie, preto model s veľkým "U"ser
- Create migration for the users table? **y**
- Select the provider for storing API tokens: **Database**
- Create migration for the api_tokens table?: **y**

Nainštalujme balíček [phc-argon2](https://www.npmjs.com/package/phc-argon2) na hashovanie hesiel používateľov v databáze:

```console
npm i phc-argon2
```

Ďalej nainštalujme podporu pre websockety do nášho Adonis projektu. Vedľa HTTP protokolu budeme používat aj websockety. Ďakujem kolegovi Ľubovi Jeszemu za implementáciu tejto podpory - balíčka `adonis-socket.io`. Bez tohto balíčka by sme mali o niečo staženú prácu.

```console
npm i @ruby184/adonis-socket.io && node ace configure @ruby184/adonis-socket.io
```

Povoľme [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).
V súbore `config/cors.ts` nájdime kľúč **enabled** a nastavme ho na hodnotu **true**:

```ts
...
enabled: true,
...
```

Komunikácia cez HTTP protokol aj cez websockety bude podliehať autentifikácii.
V súbore `start/kernel.ts` zaregistrujme pomenovaný middleware `auth` pre HTTP komunikáciu:

```ts
Server.middleware.registerNamed({
  auth: () => import("App/Middleware/Auth"),
});
```

V súbore `start/wsKernel.ts` zaregistrujme globálny `Auth` middleware pre socketovú komunikáciu:

```ts
Ws.middleware.register([() => import("App/Middleware/Auth")]);
```

Do súboru `app/Middleware/Auth.ts` pridajme funkciu `wsHandle`, ktorá bude zabezpečovať, že websocketové požiadavky prejdu kontrolou na autentifikovaného používateľa (podobne ako funkcia `handle` pre HTTP požiadavky):

```ts
import type { WsContextContract } from "@ioc:Ruby184/Socket.IO/WsContext";
```

```ts
 /**
   * Handle ws namespace connection
   */
  public async wsHandle(
    { auth }: WsContextContract,
    next: () => Promise<void>,
    customGuards: (keyof GuardsList)[]
  ) {
    /**
     * Uses the user defined guards or the default guard mentioned in
     * the config file
     */
    const guards = customGuards.length ? customGuards : [auth.name]
    await this.authenticate(auth, guards)
    await next()
  }
```

## <a name="anchor3-db"> Databázové modely

Vytvorme modely `Channel` a `Message`. Medzi týmito modelmi bude vzťah **1:M** (kanál môže obsahovať viacero správ, správa je spojená s kanálom). Cez CLI príkazy vytvorme migračné súbory a modely ako také:

```console
node ace make:migration channels
node ace make:model Channel
node ace make:migration messages
node ace make:model Message
```

V priečinku `app/models` nájdeme vytvorené modely `Channel` a `Message` a v priečinku `database/migrations` zodpovedajúce migračné súbory.

V modeli `Channel` pridajme stĺpec (atribút) **name** a zadefinujme vzťah **hasMany** na model `Message`. Správa je prepojená s kanálom cez cudzí kľúč (foreign key) **channelId**:

```ts
import { DateTime } from "luxon";
import { BaseModel, column, HasMany, hasMany } from "@ioc:Adonis/Lucid/Orm";
import Message from "App/Models/Message";

export default class Channel extends BaseModel {
  @column({ isPrimary: true })
  public id: number;

  @column()
  public name: string;

  @column.dateTime({ autoCreate: true })
  public createdAt: DateTime;

  @column.dateTime({ autoCreate: true, autoUpdate: true })
  public updatedAt: DateTime;

  @hasMany(() => Message, {
    foreignKey: "channelId",
  })
  public messages: HasMany<typeof Message>;
}
```

V modeli `Message` pridajme stĺpce **createdBy**, **channelId** a **content**. Zadefinujme tiež vzťahy `belongsTo`, teda cudzie kľúče pre `createdBy` (autor správy) a `channelId` (kanál, s ktorým je správa asociovaná):

```ts
import { DateTime } from "luxon";
import { BaseModel, BelongsTo, belongsTo, column } from "@ioc:Adonis/Lucid/Orm";
import User from "App/Models/User";
import Channel from "App/Models/Channel";

export default class Message extends BaseModel {
  @column({ isPrimary: true })
  public id: number;

  @column()
  public createdBy: number;

  @column()
  public channelId: number;

  @column()
  public content: string;

  @column.dateTime({ autoCreate: true })
  public createdAt: DateTime;

  @column.dateTime({ autoCreate: true, autoUpdate: true })
  public updatedAt: DateTime;

  @belongsTo(() => User, {
    foreignKey: "createdBy",
  })
  public author: BelongsTo<typeof User>;

  @belongsTo(() => Channel, {
    foreignKey: "channelId",
  })
  public channel: BelongsTo<typeof Channel>;
}
```

Do migračného súboru pre `Channel` (`database/migrations/..._channels.ts`) pridajme v metóde `up` stĺpec **name**:

```ts
import BaseSchema from "@ioc:Adonis/Lucid/Schema";

export default class Channels extends BaseSchema {
  protected tableName = "channels";

  public async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.increments("id").primary();
      table.string("name").notNullable().unique();
      /**
       * Uses timestamptz for PostgreSQL and DATETIME2 for MSSQL
       */
      table.timestamp("created_at", { useTz: true });
      table.timestamp("updated_at", { useTz: true });
    });
  }

  public async down() {
    this.schema.dropTable(this.tableName);
  }
}
```

Podobne do migračného súboru pre `Message` (`database/migrations/..._messages.ts`) pridajme v metóde `up` stĺpce **created_by**, **channel_id** a **content**:

```js
table
  .integer("created_by")
  .unsigned()
  .references("id")
  .inTable("users")
  .onDelete("CASCADE");
table
  .integer("channel_id")
  .unsigned()
  .references("id")
  .inTable("channels")
  .onDelete("CASCADE");
table.text("content");
```

Medzi modelmi `User` a `Channel` bude vzťah **M:N**. Potrebujeme vytvoriť väzobnú tabuľku `channel_users` (angl. pivot table):

```console
node ace make:migration channel_user
```

Poznámka: Adonis CLI príkaz vytvorí "channel_users", aj keď je na vstupe "channel_user".

Do novovytvoreného migračného súboru (`database/migrations/..._channel_users.ts`) pridajme v metóde `up` cudzie kľúče na používateľa a kanál:

```ts
table
  .integer("user_id")
  .unsigned()
  .notNullable()
  .references("id")
  .inTable("users")
  .onDelete("CASCADE");
table
  .integer("channel_id")
  .unsigned()
  .notNullable()
  .references("id")
  .inTable("channels")
  .onDelete("CASCADE");
table.unique(["user_id", "channel_id"]);
```

Zadefinujme v modeli `User` vzťah **hasMany** pre model `Message`:

```ts
import {
  column,
  beforeSave,
  BaseModel,
  hasMany,
  HasMany,
  manyToMany,
  ManyToMany,
} from "@ioc:Adonis/Lucid/Orm";
import Channel from "App/Models/Channel";
import Message from "App/Models/Message";
```

```ts
@hasMany(() => Message, {
  foreignKey: 'createdBy',
})
public sentMessages: HasMany<typeof Message>
```

a vzťah **manyToMany** pre model `Channel`:

```ts
@manyToMany(() => Channel, {
  pivotTable: 'channel_users',
  pivotForeignKey: 'user_id',
  pivotRelatedForeignKey: 'channel_id',
  pivotTimestamps: true,
})
public channels: ManyToMany<typeof Channel>
```

Použijeme [seeder](https://v5-docs.adonisjs.com/guides/database/seeders) na vytvorenie záznamu (inštanciu modelu Channel) v tabuľke channels.

Nový seeder vytvorme týmto CLI príkazom:

```console
node ace make:seeder Channel
```

Otvorme v priečinku `database/seeders` súbor `Channel.ts` a do metódy **run** doplňme:

```ts
import BaseSeeder from "@ioc:Adonis/Lucid/Seeder";
import Channel from "App/Models/Channel";

export default class ChannelSeeder extends BaseSeeder {
  public async run() {
    const uniqueKey = "name";

    await Channel.updateOrCreateMany(uniqueKey, [
      {
        name: "general",
      },
    ]);
  }
}
```

Seeder nám vytvorí kanál s názvom "general".

**Na záver tejto časti spustime migráciu spolu so seederom:**

```console
node ace migration:refresh --seed
```

Ak nemáte, odporúčam nainštalovať si prehliadač pre SQLite databázu, napríklad [DB Browser for SQLite](https://sqlitebrowser.org/dl/). Skontrolujte štruktúru a obsah databázy, či zodpovedá tomu, čo sme krok-za-krokom vytvorili.

Koniec prvej časti.

**[Zdrojový kód po prvej časti - slek-server](../slek-part1.zip)**
