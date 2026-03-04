# Rewriting Our CLI Tool in Go: What Changed, What Didn't

A six-month retrospective on migrating a Node.js dev tool to Go. Startup time, binary distribution, and the things we didn't anticipate.

## Context

We had a CLI tool written in Node.js that our team used daily. It handled scaffolding, local dev servers, and deployment. It worked fine — until we started onboarding people who didn't have Node installed.

The promise of Go was simple: **compile to a single binary, distribute anywhere.**

## What got better

### Startup time

This was the most dramatic improvement. Our Node CLI took about 400ms just to start (requiring modules, parsing config). The Go version starts in under 10ms.

For a tool you run dozens of times a day, this matters more than you'd think.

```
# Node version
$ time mycli --version
real    0m0.412s

# Go version
$ time mycli --version
real    0m0.008s
```

### Distribution

No more "install Node 18+, then run `npm install -g`." Just download a binary. We set up GoReleaser for cross-compilation and now publish binaries for Linux, macOS, and Windows on every tag.

### Error handling

Go's explicit error handling forced us to think about every failure mode. The Node version had a lot of uncaught promise rejections and vague error messages. The Go version is much better about telling you *what went wrong and why.*

## What got worse

### Development speed

TypeScript with hot reload is genuinely faster for iteration than Go's compile-test cycle. We used to be able to prototype a new command in an hour. In Go, it takes a morning.

### String manipulation

Go's standard library is powerful but verbose for string operations. Things that were one-liners in JavaScript became 5-10 line functions in Go.

```go
// JavaScript: path.split('/').filter(Boolean).map(s => s.toLowerCase())
// Go:
parts := strings.Split(path, "/")
result := make([]string, 0, len(parts))
for _, p := range parts {
    if p != "" {
        result = append(result, strings.ToLower(p))
    }
}
```

### JSON handling

Dynamic JSON is easy in JavaScript and painful in Go. We have config files with varying shapes, and defining structs for every variation was tedious.

## What didn't change

- **The core logic.** The actual business logic ported almost 1:1. Algorithms don't care about language.
- **Bug count.** We didn't see significantly fewer bugs. Different bugs, yes. Fewer, no.
- **User satisfaction.** Users cared about speed and easy installation. They got both. They didn't care what language it was written in.

## Would I do it again?

Yes, but I'd be more intentional about timing. We rewrote during a period of active feature development, which meant maintaining two versions for three months. In retrospect, we should have frozen features, done the rewrite, then resumed development.

The Go version is unambiguously better for our use case (a CLI tool distributed to non-developers). But the rewrite cost us about three engineer-months, and during that time we shipped zero new features.

**The lesson: rewrites are investments. Make sure the return is worth the cost.**
