---
name: typescript-best-practices
description: "TypeScript coding standards covering strict tsconfig setup, interfaces vs types, type safety (unknown vs any, type guards, exhaustive checks), null handling, async/await patterns, immutability (readonly, as const), utility types, enums, file naming, and barrel file (index.ts) usage. Use whenever writing, reviewing, or refactoring TypeScript code, setting up a tsconfig.json, or making decisions about project/module structure."
---

# TypeScript Best Practices

Standards for writing clean, effective, scalable, and type-safe TypeScript code.

## When to Use

- Writing or reviewing TypeScript code
- Configuring `tsconfig.json` for a new or existing project
- Deciding between `interface` and `type`
- Handling `null`/`undefined`, unions, or exhaustive `switch` statements
- Writing async functions or error handling
- Structuring files, modules, or barrel (`index.ts`) exports

## Project Configuration

Always enable **strict mode** in `tsconfig.json` to maximize type safety and catch hidden bugs at compile time.

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true
  }
}
```

## Interfaces vs. Types

Both define object shapes but have different strengths.

**Interfaces** — use for object shapes that can be extended or implemented.

```ts
interface User {
  id: number;
  name: string;
}

interface AdminUser extends User {
  permissions: string[];
}
```

**Types** — use for unions, tuples, or mapped types where extension isn't needed.

```ts
type Status = "active" | "inactive" | "pending";
type Point = [number, number];
```

## Type Safety & Inference

### Prefer `unknown` over `any`

`any` defeats the purpose of TypeScript. `unknown` forces type-checking before the variable is used.

```ts
// PASS
function processInput(input: unknown) {
  if (typeof input === "string") {
    console.log(input.toUpperCase());
  }
}

// FAIL
function processInput(input: any) {
  console.log(input.toUpperCase()); // no safety check
}
```

### Let TypeScript infer obvious types

```ts
// PASS
const name = "John";

// FAIL: redundant annotation
const name: string = "John";
```

### Be explicit on public APIs and return types

Always declare parameter and return types for functions, to prevent accidental drift and catch errors when behavior changes.

### Use type guards

Narrow types safely with `typeof`, `instanceof`, or custom guard functions.

```ts
function isString(value: unknown): value is string {
  return typeof value === "string";
}
```

## Null, Undefined, and Exhaustive Checks

### Optional chaining and nullish coalescing

```ts
const userName = user?.profile?.name ?? "Guest";
```

### Exhaustive checks with `never`

In `switch` statements over a union type, assign the default case to a `never`-typed variable so TypeScript throws a compile error if a new variant is added later without being handled.

```ts
type Shape = Circle | Square | Triangle;

function getArea(shape: Shape) {
  switch (shape.type) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side * shape.side;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

## Functions and Async Patterns

### Keep functions pure and focused

Avoid "god functions" that handle validation, transformation, and side effects all at once. Split into smaller, predictable, pure functions.

### Async/await error handling

Always wrap async operations in `try/catch`, and use `Promise.all` for parallel operations.

```ts
async function fetchData<T>(url: string): Promise<T> {
  try {
    const response = await fetch(url);

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return (await response.json()) as T;
  } catch (error) {
    console.error("Failed to fetch data:", error);
    throw error;
  }
}
```

## Immutability and Utility Types

### `readonly` properties

Prevent accidental mutation of properties that shouldn't change after initialization.

```ts
interface Product {
  readonly id: number;
  readonly name: string;
  price: number;
}
```

### `as const` assertions

Use on literal types, arrays, or objects to improve inference and create deeply read-only, narrowed types.

```ts
const colors = ["red", "green", "blue"] as const;

const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
} as const;
```

### Built-in utility types

Leverage `Partial`, `Pick`, `Omit`, `Readonly`, etc. to avoid repetitive type declarations.

```ts
type OptionalUser = Partial<User>;
```

## Code Organization

### Enums for meaningful constants

Use `enum` to define a fixed set of named constants, making code expressive and self-documenting.

```ts
enum UserRole {
  Admin = "ADMIN",
  User = "USER",
  Guest = "GUEST",
}
```

> Note: some teams prefer object literals with `as const` over enums to avoid JS compilation quirks, but enums remain a valid standard approach.

### File naming conventions

Group code into logical modules with consistent file naming.

```
// PASS
user.service.ts
user.model.ts
user.controller.ts

// FAIL
UserService.ts
user_service.ts
```

### Barrel files (`index.ts`)

Barrel files re-export modules from a directory, creating a clean public API and simplifying imports — but must be used selectively at scale to avoid circular dependencies.

**Recommended for static/independent code** — DTOs, interfaces & types, constants, entities/models, helpers/utils:

```ts
// user/dto/index.ts
export * from "./create-user.dto";
export * from "./update-user.dto";
export * from "./user-response.dto";
```

**Use with caution for business logic and DI** — services, controllers, modules. Grouping these in an `index.ts` often causes implicit cross-imports; if two services are exported from the same barrel and later need to interact, they can trap each other in a circular dependency loop.

**Rule of thumb:**
1. Isolated/data-centric folders (`dto/`, `interfaces/`) → use a barrel file.
2. Core business logic (`users.service.ts`) → import directly from the specific file for predictable execution order.

```ts
// PASS: direct import prevents circular dependency risk
import { UserService } from "./user.service";

// FAIL: relying on a barrel file for DI components
import { UserService } from "./index";
```
