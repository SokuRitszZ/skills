---
name: sokutils-frontend-code
description: Build, refactor, or review concise type-safe React and TypeScript code with Sokutils. Use when reducing component and class boilerplate, props drilling, repeated subtree state, imperative dialog callbacks, duplicate caches, or utility and concurrency code with @sokutils/react-component-kit, @sokutils/react-context, @sokutils/react-promisify, @sokutils/cache, or @sokutils/pure.
---

# Sokutils Frontend Code

Use the smallest Sokutils abstraction that removes repeated ceremony without
hiding product behavior or weakening inferred types.

## Resolve the actual API

1. Check the installed version and existing usage.
2. Read documentation in this order:
   - the corresponding `packages/<name>` directory in a Sokutils workspace;
   - `README.md`, `skill.md`, `AGENTS.md`, declarations, and metadata in the
     resolved npm package directory;
   - the matching GitHub package directory or npm page.
3. Prefer a release tag matching the installed version. If documentation and
   installed code differ, follow installed exports, declarations, and behavior.

Read [references/packages.md](references/packages.md) for package links, API
selection, compact examples, or migration guidance.

## Select one responsibility

| Need | Prefer |
| --- | --- |
| Typed intrinsic element or reusable class variants | React Component Kit `rck` |
| Style an existing component | React Component Kit `Rck` |
| Named CSS Grid regions | React Component Kit `layout` |
| Share public props within one subtree | React Context `ctx.props` |
| Share component-scoped model state | React Context `ctx.model` |
| Share results from several hooks | React Context `ctx.hooks` |
| Call a mounted dialog or drawer as a Promise | React Promisify |
| Cache a function result | Cache |
| Reuse generic functions, paths, locks, or schedulers | Pure |

Keep one-off expressions inline when they are already clearer.

## Apply safely

- Define generated components, context factories, promisify registrations, and
  cached functions at module scope.
- Name abstractions by product intent.
- Mount providers outside consumers and order composed providers by dependency.
- Keep JSX declarative; do not hide network or persistence work in visual
  primitives.
- Preserve semantic elements, native props, accessibility attributes, refs, and
  consumer class overrides.
- Give every Promisify close path an explicit resolve or reject.
- Give every cache a deterministic key and explicit lifetime or invalidation
  policy.
- Use locks and schedulers only for a real ordering or concurrency requirement;
  release resources in `finally`.
- Reject a reduction that couples unrelated features, creates factories during
  render, introduces `any`, or discards loading, error, rejection, or empty
  states.

## Verify

1. Remove superseded imports and duplicated implementations.
2. Type-check every edited library and consumer.
3. Run focused tests for stateful utilities and lint edited files.
4. Build the consuming application.
5. For publishable packages, build and run `npm pack --dry-run`.
6. Confirm refs and classes reach the element, hooks sit below providers,
   Promises always settle, and cache behavior matches its policy.

