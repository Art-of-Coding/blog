One of the biggest features of Nexus Engine will be mods interface. The core of
the engine has been designed to be abstract and extensible, while functionality
is built into _mods_. Many of what can be considered _core functions_ will be
implemented as mods themselves! In this post I dive a little deeper into the
proposed mods architecture. As an example we will be creating a mod which
implements a custom storage provider.

## Anatomy of a Mod

> Much of this work has been based on
> [xxxscreeps](https://github.com/laverdet/xxscreeps)

Mods must conform to a minimal structure. For example, we assume you are using
TypeScript, which must output the generated JavaScript files into a directory
called `dist`. If for whatever reason a file cannot be located in this
directory, the engine will display a warning message and continue without
loading the mod in question.

While this feature is fault-tolerant in the way that a malfunctioning mod
doesn't crash the instance, running without expected mods may still have
side-effects.

### The Manifest

Each mod must have a manifest. The manifest is loaded on engine startup and
provides metadata about the mod. The manifest should be in your mod's `main`
file. Currently the manifest only tells the engine which providers it supports.

In the example below we are creating a manifest and we provide a single
provider, the `storage` provider:

```ts
// Import the manifest type
import type { Manifest } from "@nexus-engine/engine";

// Export the mod's manifest
export const manifest: Manifest = {
  // A mod can provide multiple providers simultaneously
  provides: ["storage"],
};
```

Providers are imported at various stages during the lifecycle of the instance,
and can extend the Engine instance through the exposed mods API.

### An Example Provider

The hypothetical mod in the example above supports a single provider; the
`storage` provider. This provider is called by the engine before data providers
are created, allowing you to add data providers or do other preperatory work
before the floodgates open.

In order to actually implement the provider, simply create a `storage.ts` in
your source directory. We need to import the mods API so we can make use of the
engine's exports. Then we create an anonymous function which must be exported as
`default`. Your mod's provider logic will live in this function, where it has
access to the mods API.

In the following example we import a hypothetical data provider and register it
with the engine:

```ts
import type Engine from "@nexus-engine/engine";
import { MyDataProvider } from "./my-provider.js";

export default async function (engine: typeof Engine) {
  // this is where your logic lives
  engine.registerProvider("data", "mongodb:", (path) => MyDataProvider(path));
}
```

The first argument to `registerProvider` is the disposition. The engine
currently requires data providers for four dispositions:

1. `data`; this is the shard's persistent storage (modeled after the MongoDB
   API)
2. `keyval`; acts as (possibly) distributed key-value store (modeled after the
   Redis API)
3. `stream`; provides stream 'storage' (modeled after Redis Streams)
4. `pubsub`; provides publish-subscribe 'storage' (modeled after the Redis API)

By default the engine uses the `memory:` protocol, which loads providers which
hold all their data in-memory. While this is useful for testing, even a single
deployment would be better off choosing a more robust database and storage
back-end, such as MongoDB and Redis.

Now we have created a mod which registers our data provider. Congratulations!
Now what?

We need to enable the mod in your engine's `.engine.yaml`:

```yaml
mods:
  - my-provider-mod # Add your mod name to the list
```

How do we actually tell the engine to use our new data provider? We need to make
one more modification to our configuration file:

```yaml
storage:
  data:
    # Use the mongodb: protocol to use the mod's data provider
    path: mongodb://localhost
```

On startup the engine will register your data provider and build a new instance
using the path in the configuration file. That's all there is to it!

> Depending on your development environment, you may need to link your mod to
> your engine package. You can use
> [npm link](https://docs.npmjs.com/cli/v8/commands/npm-link) for that, although
> there are alternatives.

## Contribute

Nexus Engine is being developed as an open source project, and contributions are
always welcome! Read our
[contributing page](https://github.com/NexusEngine/nexus/blob/main/CONTRIBUTING.md)
to learn more.
