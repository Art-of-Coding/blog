One of the biggest features of Nexus Engine will be mods interface. The core of
the engine has been designed to be abstract and extensible, while functionality
is built into _mods_. Many of what can be considered _core functions_ will be
implemented as mods themselves! In this post I dive a little deeper into the
proposed mods architecture.

## Anatomy of a Mod

> Much of this work has been based on
> [xxxscreeps](https://github.com/laverdet/xxscreeps).

Enabled mods are loaded on engine startup, and the various providers they export
are imported at various stages during the engine's lifecycle.

> ### TypeScript: enable global types
>
> If you're using TypeScript, you can enable the engine's global types by adding
> the engine to the `types` array in your `tsconfig.json`:
>
> ```json
> {
>   "types": [
>     "@nexus-engine/engine"
>   ]
> }
> ```

### The Manifest

Each mod should export a `manifest` property in its `main` module. The manifest
provides metadata about what providers your mod supports. `Providers` are
imported at the various stages of the engine lifecycle.

This is an example of a manifest which exports a `game` provider:

```ts
export const manifest: Manifest = {
  provides: ["game"],
};
```

> The engine assumes you are using TypeScript and will look for the `game`
> provider at `/dist/game.js`. If your provider is located elsewhere, you can
> provide a `path` for the provider:
>
> ```ts
> export const manifest: Manifest = {
>   provides: ["game"],
>   paths: {
>     // Note the forward slash as the first character
>     game: "/another-dir/game.js",
>   },
> };
> ```

### Implementing a Game Object

Almost anything can be a game object. Game objects are persisted to the database
and provide the main functionality of your game. Users can call specific methods
on these objects (named `intents`), which are then processed according to your
own game logic.

As an example we will be building a `Stock` game object, which represents a
stock that has a particular price. We will implement the `buy` intent, which
allows the player to buy a number of stocks.

Each game object is represented by a shape - the shape of the underlying data
structure. Each shape should extend the `BaseShape` type, which adds an `_id`
property with the given type. Each game object must have an `_id`.

In our case the shape looks like this:

```ts
export interface StockShape extends BaseShape<string> {
  price: number;
}
```

Next, let's take a look at the actual implementation of our game object.

```ts
export class Stock extends GameObject<StockShape> {
  constructor(data: StockShape) {
    super(null, "stocks", data);
  }

  get price() {
    return this["#data"].price;
  }

  async buy(context: IntentContext, amount: number) {
    await chainIntentChecks(
      () => checkUser(context),
      () => checkBalance(context.userId, amount * this.price),
      () => Game.executeIntent(this, context, amount),
    );
  }
}
```

The intent method receives `IntentContext` as its first argument. The arguments
the method was called with follow. In this case, there is one argument, the
amount of stocks to buy.

We use the global `chainIntentChecks` to chain multiple checks together. In
`checkUser`, we will check that the intent was called by an actual player and
not the system or one of its mods. In `checkBalance`, we check to see that the
user has enough money to buy the number of stocks they want.

The methods can be implemented as follows:

```ts
async function checkUser(context: IntentContext) {
  if (!context.userId) {
    throw new Error("This intent can only be executed by a user!");
  }
}

async function checkBalance(userId: string, amount: number) {
  if (Memory.get() < amount) {
    throw new Error("Not enough money");
  }
}
```

Finally, `Game.executeIntent()` is called. This will actually execute the intent
logic. But where does the logic live? You can define it by calling
`Game.registerIntentProcessor()`:

```ts
Game.registerIntentProcessor(
  Stock,
  "buy",
  (stock, context, amount: number) => {
    await Promise.all([
      Memory.decrBy(`balances:${context.userId}`, amount * stock.price),
      Memory.incrBy(`stocks:${context.userId}:${stock.id}`, amount),
    ]);
  },
);
```

You have just implemented a game object with an intent!

The mods interface is currently very fluid and subject to quick change.

## Contribute

Nexus Engine is being developed as an open source project, and contributions are
always welcome! Read our
[contributing page](https://github.com/NexusEngine/nexus/blob/main/CONTRIBUTING.md)
to learn more.
