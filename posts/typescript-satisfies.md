# TIL: TypeScript's `satisfies` Operator and When to Use It

A small but genuinely useful addition in TS 4.9 that lets you validate a value against a type without widening it.

## The problem

Consider this common pattern:

```typescript
type Theme = {
  colors: Record<string, string>;
  spacing: Record<string, number>;
};

const theme: Theme = {
  colors: {
    primary: '#b06a3a',
    background: '#f8f6f1',
  },
  spacing: {
    sm: 4,
    md: 8,
    lg: 16,
  },
};
```

This works, but there's a catch. Because we annotated `theme` as `Theme`, TypeScript forgets the *specific* keys:

```typescript
theme.colors.primary    // ✅ works, but type is `string`
theme.colors.oops       // ✅ also works — no error! Type is `string`
theme.spacing.sm        // ✅ works, but type is `number`
```

We've lost the literal key information. TypeScript knows `colors` is `Record<string, string>`, so any key is valid.

## Enter `satisfies`

The `satisfies` operator checks that a value conforms to a type, but preserves the *inferred* type:

```typescript
const theme = {
  colors: {
    primary: '#b06a3a',
    background: '#f8f6f1',
  },
  spacing: {
    sm: 4,
    md: 8,
    lg: 16,
  },
} satisfies Theme;
```

Now:

```typescript
theme.colors.primary    // ✅ type is string
theme.colors.oops       // ❌ Error! Property 'oops' does not exist
theme.spacing.sm        // ✅ type is number
theme.spacing.xl        // ❌ Error! Property 'xl' does not exist
```

We get the best of both worlds: type checking *and* precise inference.

## When to use it

- **Configuration objects** where you want to validate the shape but keep literal keys
- **Route maps** where keys are specific path strings
- **Event handlers** where you want to ensure all events are handled
- **Any `Record<>` type** where you want key-level type safety

## When NOT to use it

- When you're already using `as const` — they serve different purposes
- When you genuinely want the wider type (e.g., the object will have dynamic keys at runtime)
- In type-only contexts — `satisfies` is a value-level operator

## The mental model

Think of it this way:
- **Type annotation (`:`)** = "treat this value AS this type" (widens)
- **`satisfies`** = "check this value AGAINST this type" (preserves)

It's a small distinction, but once you internalize it, you'll find uses for it everywhere.
