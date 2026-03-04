# Type-Safe API Clients Without Code Generation

Using TypeScript's type system to infer request and response shapes directly from a schema object, no codegen step needed.

## The problem

Every API client faces the same tension: you want type safety, but you don't want to maintain types separately from your API definition. The usual solution is code generation — tools like OpenAPI Generator or GraphQL Code Generator that read a schema and spit out TypeScript types.

But codegen has friction. You need a build step, generated files in your repo, and a process to keep them in sync. What if we could skip all that?

## The idea: schema as the source of truth

Instead of generating types from a schema, what if the schema itself *was* the type?

```typescript
const api = {
  'GET /users': {
    response: {} as { id: string; name: string; email: string }[],
  },
  'POST /users': {
    body: {} as { name: string; email: string },
    response: {} as { id: string; name: string; email: string },
  },
  'GET /users/:id': {
    params: {} as { id: string },
    response: {} as { id: string; name: string; email: string },
  },
} as const;
```

The `as const` assertion preserves the literal types. Now we can write a generic client function that infers everything:

```typescript
type ApiSchema = typeof api;
type Endpoint = keyof ApiSchema;

type RequestConfig<E extends Endpoint> = 
  ApiSchema[E] extends { body: infer B } ? { body: B } : {} &
  ApiSchema[E] extends { params: infer P } ? { params: P } : {};

type ResponseType<E extends Endpoint> = 
  ApiSchema[E] extends { response: infer R } ? R : never;

async function request<E extends Endpoint>(
  endpoint: E,
  config: RequestConfig<E>
): Promise<ResponseType<E>> {
  // implementation...
}
```

## Does it scale?

For small-to-medium APIs (under ~50 endpoints), this approach works beautifully. You get full type inference, IDE autocompletion, and compile-time checking without any build tools.

For larger APIs, you start hitting TypeScript's type recursion limits and the schema object becomes unwieldy. At that point, codegen starts to make more sense.

## Tradeoffs

| Approach | Pros | Cons |
|----------|------|------|
| **Schema-as-type** | No build step, instant feedback | Scales poorly, complex conditional types |
| **Code generation** | Handles any size, industry standard | Build step required, generated files |
| **Manual types** | Simple, no tooling | Drift risk, maintenance burden |

## When to use this

- Internal tools and small APIs
- Prototyping where iteration speed matters
- Projects where adding a codegen step feels like overkill

The key insight is that TypeScript's type system is powerful enough to be its own schema language. You just have to meet it halfway.
