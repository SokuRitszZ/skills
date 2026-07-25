# Sokutils package reference

## Contents

- [Documentation](#documentation)
- [React Component Kit](#react-component-kit)
- [React Context](#react-context)
- [React Promisify](#react-promisify)
- [Cache](#cache)
- [Pure](#pure)
- [Migration](#migration)

## Documentation

Resolve the installed package root instead of assuming a pnpm, npm, or Yarn
storage layout. Published packages may contain their own `README.md`, `skill.md`,
`AGENTS.md`, or type declarations.

| Package | Workspace | GitHub | npm |
| --- | --- | --- | --- |
| React Component Kit | `packages/react-component-kit` | [docs](https://github.com/SokuRitszZ/sokutils/tree/main/packages/react-component-kit) | [package](https://www.npmjs.com/package/@sokutils/react-component-kit) |
| React Context | `packages/react-context` | [docs](https://github.com/SokuRitszZ/sokutils/tree/main/packages/react-context) | [package](https://www.npmjs.com/package/@sokutils/react-context) |
| React Promisify | `packages/react-promisify` | [docs](https://github.com/SokuRitszZ/sokutils/tree/main/packages/react-promisify) | [package](https://www.npmjs.com/package/@sokutils/react-promisify) |
| Cache | `packages/cache` | [docs](https://github.com/SokuRitszZ/sokutils/tree/main/packages/cache) | [package](https://www.npmjs.com/package/@sokutils/cache) |
| Pure | `packages/pure` | [docs](https://github.com/SokuRitszZ/sokutils/tree/main/packages/pure) | [package](https://www.npmjs.com/package/@sokutils/pure) |

## React Component Kit

Use `rck.tag` for native elements, `Rck` for components that apply `className`,
and `layout` for named grid regions.

```tsx
const Layout = layout(`
content
bottom
`);

const L = {
  Container: rck.main('size-full overflow-auto'),
  Button: rck.button(
    'rounded px-3 py-2',
    { disabled: 'cursor-not-allowed opacity-50' },
    { tone: { primary: 'bg-primary text-white' } },
  ),
};

const Page = () => (
  <Layout>
    <Layout.Content>
      <XxxFormContent />
    </Layout.Content>
    <Layout.Bottom>
      <XxxFormBottom />
    </Layout.Bottom>
  </Layout>
);
```

Define generated components at module scope. Let native props, children, events,
and refs infer. Apply configuration in its documented priority order and verify
where consumer `className` lands. Do not wrap a component that discards
`className` or the required ref. Strictly name an RCK layout object `Layout`;
limit its region components to layout concerns, and put feature content and
behavior in the corresponding `XxxRegion` component. Never use `Xxx.Region`.

For a single-responsibility component, group its RCK primitives in `L`. Put
static classes in string arguments and model state-dependent styles with RCK
configuration:

```tsx
const L = {
  Item: rck.button(
    'rounded px-3 py-2',
    { selected: 'bg-blue-500 text-white' },
    {
      checked: [
        ['bg-gray-200'],
        ['bg-blue-500'],
      ],
    },
    {
      tone: {
        primary: 'text-blue-700',
        danger: 'text-red-700',
      },
    },
  ),
};
```

A string or string array activates for a truthy prop. A two-entry nested array
maps false then true. A record defines an enum variant. Split independent style
concerns into separate arguments. Configuration merges left to right; consumer
`className` merges last and has highest priority.

## React Context

Inventory the component's state and dependencies before creating contexts:

| Layer | Contents | Factory |
| --- | --- | --- |
| External | Public props and callbacks from the parent | `ctx.props` |
| Internal model | Autonomous component state and setters | `ctx.model` |
| Internal shared hooks | Shared React ref and TanStack React Query instances/results | `ctx.hooks` |

Use the factory matching that ownership:

```tsx
const [withPanelProps, usePanelProps] = ctx.props<PanelProps>();
const [withPanelModel, usePanelModel] = ctx.model<PanelModel>({
  selectedId: undefined,
});
const [withPanelHooks, usePanelHooks] = ctx.hooks({
  container: () => useRef<HTMLDivElement>(null),
  query: usePanelQuery,
});

const Panel = withPanelProps(
  withPanelModel(
    withPanelHooks(RawPanel),
  ),
);
```

Create separate factories for separate subtrees. Initialize optional model keys
explicitly. Keep the hooks map stable and unconditional. The hooks layer may
consume props and model because their providers are outside it; never reverse
that dependency. Reserve `ctx.hooks` for Hook instances or results that
descendants must share. Keep reducer, validation, Effect, and domain-operation
Hooks local to the single-responsibility component that owns them.

## React Promisify

Use it for dialogs, drawers, and pickers that need a typed imperative entry
point:

```tsx
const [PickerRegister, usePickerTools, pickItem] =
  promisify.component<Output, Input, Config, Error>(Picker);
```

Mount the Register before calling the function. Treat input as optional before
the first call. Resolve or reject every close path. Confirm whether the installed
version queues or permits concurrent calls before relying on that behavior.

## Cache

Prefer `CacheBuild` for incremental configuration and `CacheCore` when all
options are already known:

```ts
const getUser = CacheBuild()
  .Function((id: string) => fetchUser(id))
  .KeyGenerator(id => id)
  .Strategy(CachePresetStrategyLRU(100))
  .Build();
```

Choose `Once`, `Timeout`, `ExpireAt`, or `LRU` deliberately. Match Storage
schemas to Strategy context. Check the installed documentation for rejected
Promise handling and whether Storage saves are awaited.

## Pure

Search Pure before writing generic function-control, result, object-path,
locking, or scheduling helpers. Read the relevant source guide before using
mutexes, semaphores, priority locks, or schedulers. Do not import Cache from
Pure; use `@sokutils/cache`.

## Migration

```tsx
divx({}, '...')            → rck.div('...')
divy('section', {}, '...') → rck.section('...')
divz(Component, {}, '...') → Rck(Component, '...')

@sokutils/react ctx        → @sokutils/react-context
@sokutils/react promisify  → @sokutils/react-promisify
@sokutils/pure Cache       → @sokutils/cache
```

Delete an old import only after confirming no remaining symbol uses it.
