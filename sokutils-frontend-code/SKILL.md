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

## Reuse libraries before custom code

Search before implementing:

1. Search the repository for an existing implementation.
2. Search the installed Sokutils packages and the repository's adopted utility
   library, especially `es-toolkit`, `lodash-es`, or Lodash.
3. If needed, check maintained community packages and their official
   documentation or source.
4. Write custom utility code only when no suitable option exists.

Prefer Sokutils for its documented responsibilities and the repository's
existing general-purpose utility library for common object and collection
operations. Before adding a dependency, evaluate its types, compatibility,
maintenance, bundle and tree-shaking cost, security, license, and repository
policy. Do not add a package when a direct one-line expression is clearer.

## Select one responsibility

| Need | Prefer |
| --- | --- |
| Typed intrinsic element or reusable class variants | React Component Kit `rck` |
| Style an existing component | React Component Kit `Rck` |
| Named CSS Grid regions | React Component Kit `layout` |
| Share public props within one subtree | React Context `ctx.props` |
| Share component-scoped model state | React Context `ctx.model` |
| Share one ref or query result across descendants | React Context `ctx.hooks` |
| Call a mounted dialog or drawer as a Promise | React Promisify |
| Cache a function result | Cache |
| Reuse generic functions, paths, locks, or schedulers | Pure |

Keep one-off expressions inline when they are already clearer.

## Enforce component roles

Treat the layout/responsibility split as the default rule:

- Classify every XML or JSX component as either a layout component or a
  single-responsibility component.
- Name every RCK `layout` result `Layout`.
- Keep `Layout` and `Layout.Region` limited to region hierarchy, placement,
  sizing, overflow, and responsive behavior.
- Name each region implementation `<Feature><Region>` and render it inside the
  matching wrapper:

```tsx
<Layout.Bottom>
  <XxxFormBottom />
</Layout.Bottom>
```

- Never use `XxxForm.Bottom`.
- Move feature state, data, effects, handlers, content, and interactions into
  `XxxFormBottom` or another single-responsibility component.

Allow both roles only for a genuinely trivial component, such as one short,
flat left/right row with a middle ellipsis. Confirm that it has no named regions,
complex responsive or overflow rules, data loading, effects, context
orchestration, non-trivial state, or independently reusable layout. Split it as
soon as it grows beyond those limits.

Do not apply this exception to an explicit RCK `Layout`: keep every
`Layout.Region` limited to layout and render the matching `<Feature><Region>`
component inside it.

## Encapsulate responsibility styles

For every single-responsibility component:

- Define its styled primitives at module scope and group them in `L`.
- Use `rck.tag` for semantic intrinsic elements and `Rck` for existing
  components that correctly apply `className`.
- Put base styles in string arguments.
- Put truthy boolean, false/true boolean, and enum variant styles in RCK
  configuration arguments so RCK infers the visual-state props.
- Pass semantic state props such as `selected`, `busy`, `tone`, or `status` in
  JSX instead of constructing conditional class strings there.
- Keep `Layout.Region` placement styles out of `L`; keep the responsibility
  component's own presentation and state-dependent styles in `L`.

```tsx
const L = {
  Item: rck.button(
    'rounded px-3 py-2',
    { selected: 'bg-primary text-white' },
    { tone: { normal: 'text-default', danger: 'text-danger' } },
  ),
};

const XxxFormBottom = ({ selected, tone }: Props) => (
  <L.Item selected={selected} tone={tone}>Submit</L.Item>
);
```

## Plan component state layers

Plan the complete component state before creating contexts:

| Layer | Put here | Use |
| --- | --- | --- |
| External | Parent-supplied props and callbacks | `ctx.props` |
| Internal model | Autonomous state and setters | `ctx.model` |
| Internal shared hooks | Shared React refs and TanStack React Query results | `ctx.hooks` |

Create the factories at module scope and compose them in dependency order:

```tsx
const Xxx = withXxxProps(
  withXxxModel(
    withXxxHooks(RawXxx),
  ),
);
```

The hooks layer may consume props and model because it is inside both providers.
Never make an outer layer consume an inner context. Keep factories scoped to one
component tree and keep each consumer hook separate unless a combined Hook is
clearly useful. Do not put reducer, validation, Effect, or domain-operation
Hooks in `ctx.hooks`; call them directly in their owning
single-responsibility component.

## Invoke Auto Hooks exactly once

Bind each Auto Hook to one corresponding component. Name it
`use<Component>Auto<Responsibility>` and call it exactly once, unconditionally,
at the top level of that component before early returns and JSX. If contexts
wrap the component, call it in the corresponding raw component below the
providers.

Never call an Auto Hook from `ctx.hooks`, another Hook, a parent or child,
conditional code, a loop, a callback, or an event handler. A component may call
several different Auto Hooks, but each must appear exactly once.

## Control source file size

- Keep every handwritten frontend implementation file at or below 100 physical
  lines when practical.
- Never leave an implementation file above the hard limit of 150 physical
  lines.
- Split near 100 lines along layout, feature-region, responsibility, hook,
  service, or type ownership.
- Keep a responsibility component and its local RCK `L` primitives together;
  do not separate styles merely to game the limit.
- Never compress or obscure code to reduce the measured line count.

Exclude generated, vendored, and snapshot files.

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
7. Check that edited implementation files target 100 lines and never exceed
   150.
8. Confirm that every new custom utility was necessary after searching local,
   Sokutils, adopted utility-library, and community options.
