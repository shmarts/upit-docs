Bun treats TypeScript as a first-class citizen.

<Callout>
**Note** — To add type declarations for Bun APIs like the `Bun` global, follow the instructions at [Intro > TypeScript](https://bun.sh/docs/typescript). This page describes how the Bun runtime runs TypeScript code.
</Callout>

## Running `.ts` files

Bun can directly execute `.ts` and `.tsx` files just like vanilla JavaScript, with no extra configuration. If you import a `.ts` or `.tsx` file (or an `npm` module that exports these files), Bun internally transpiles it into JavaScript then executes the file.


## Path mapping

When resolving modules, Bun's runtime respects path mappings defined in [`compilerOptions.paths`](https://www.typescriptlang.org/tsconfig#paths) in your `tsconfig.json`. No other runtime does this.

Consider the following `tsconfig.json`.

```json
{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "data": ["./data.ts"]
    }
  }
}
```

Bun will use `baseUrl` to resolve module paths.

```ts
// resolves to ./src/components/Button.tsx
import { Button } from "components/Button.tsx";
```

Bun will also correctly resolve imports from `"data"`.


```ts#index.ts
import { foo } from "data";
console.log(foo); // => "Hello world!"
```

## Experimental Decorators

Bun supports the pre-TypeScript 5.0 experimental decorators syntax.

```ts#hello.ts
// Simple logging decorator
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with:`, args);
    return originalMethod.apply(this, args);
  };
}

class Example {
  @log
  greet(name: string) {
    return `Hello ${name}!`;
  }
}

// Usage
const example = new Example();
example.greet("world"); // Logs: "Calling greet with: ['world']"
```

To enable it, add `"experimentalDecorators": true` to your `tsconfig.json`:

```jsonc#tsconfig.json
{
  "compilerOptions": {
    // ... rest of your config
    "experimentalDecorators": true,
  },
}
```

We generally don't recommend using this in new codebases, but plenty of existing codebases have come to rely on it.

### emitDecoratorMetadata

Bun supports `emitDecoratorMetadata` in your `tsconfig.json`. This enables emitting design-time type metadata for decorated declarations in source files.

```ts#emit-decorator-metadata.ts
import "reflect-metadata";

class User {
  id: number;
  name: string;
}

function Injectable(target: Function) {
  // Get metadata about constructor parameters
  const params = Reflect.getMetadata("design:paramtypes", target);
  console.log("Dependencies:", params); // [User]
}

@Injectable
class UserService {
  constructor(private user: User) {}
}

// Creates new UserService instance with dependencies
const container = new UserService(new User());
```

To enable it, add `"emitDecoratorMetadata": true` to your `tsconfig.json`:

```jsonc#tsconfig.json
{
  "compilerOptions": {
    // ... rest of your config
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
  },
}
```