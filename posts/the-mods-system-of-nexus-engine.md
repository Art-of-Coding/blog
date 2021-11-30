One of the biggest features of Nexus Engine is its mods interface. The core of
the engine has been designed to be abstract and extensible, while functionality
is build into _mods_. Many of what can be considered _core functions_ are
implemented as mods themselves!

## Anatomy of a Mod

> Much of this work has been based on
> [xxxscreeps](https://github.com/laverdet/xxscreeps)

Mods must have a minimal structure in order to function. For example, we assume
you are using TypeScript, so we expect to find some files in your package's
`dist` directory. This post assumes some familiarity with TypeScript and
packages in general. If you are not familiar with TypeScript yet, take a look at
their
[documentation](https://www.typescriptlang.org/docs/handbook/2/basic-types.html).

### The Manifest

The need for a manifest is another requirement. The manifest should be exported
from your package's main file, the one defined in your `package.json`'s `main`
key. The manifest tells the engine which providers it supports. In the example
below, we have the manifest for a mod which provides a `storage` provider:

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

### Example Provider

The example above shows the manifest for a mod which provides a single provider,
`storage`. All storage providers are imported before data providers are set up,
allowing you to add data providers and do other work before the floodgates open.

To create the storage provider, create a `storage.ts` in your project source
directory.

> The engine expects to find this file in the mod's `dist` directory. See
> earlier in this post for a description of why.

```ts
import type Engine from "@nexus-engine/engine";

export default async function (engine: typeof Engine) {
  // your mod logic here
}
```

The Engine exports the entire mods API, which can then be used by your mod's
logic.

Now that that mods interface is functional, it's time to focus on the engine's
internals. This will involve a lot of work to orchestrate, but I am confident in
its outcome.

## Contribute

Nexus Engine is being developed as an open source project, and contributions are
always welcome! Read our
[contributing page](https://github.com/NexusEngine/nexus/blob/main/CONTRIBUTING.md)
to learn more.
