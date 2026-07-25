---
name: frontend-code-style
description: Write, refactor, or review clear production-grade frontend code. Use for React or TypeScript components, XML or JSX component trees, pages, dashboards, design systems, HTML/CSS layouts, UI polish, component boundaries, naming, state organization, effects, responsive behavior, and accessibility.
---

# Frontend Code Style

Write code that follows the product and repository instead of imposing a new
architecture or visual language.

## Inspect first

1. Read repository instructions, nearby components, package dependencies, and
   TypeScript and styling configuration.
2. Reuse the existing component library, state tools, utilities, design tokens,
   and atomic CSS framework.
3. Search for the local pattern before introducing a new abstraction.
4. Preserve public behavior unless the request explicitly changes it.

## Reuse before implementing

After understanding the requirement, search for an existing solution before
writing code. Use this order:

1. Search the repository for an existing helper or implementation.
2. Inspect installed dependencies and their existing usage. Prefer the utility
   library already adopted by the repository, such as `es-toolkit`,
   `lodash-es`, or Lodash.
3. If no local option fits, search maintained community packages and their
   official documentation or source.
4. Write a custom implementation only when the available solutions do not fit,
   and keep the reason evident in the change.

Do not hand-roll large object, collection, path, equality, debounce, throttle,
retry, scheduling, caching, or concurrency utilities before completing this
search.

Evaluate a candidate for semantic fit, edge cases, TypeScript types,
compatibility, maintenance, tree-shaking and bundle cost, security, license, and
the repository's dependency policy. Do not add a dependency for a trivial
one-line operation when direct code is clearer.

## Keep structure intentional

- Name components, props, handlers, and derived values by product intent.
- Export `XxxProps` only for public components; use a local `Props` type for
  private subcomponents.
- Keep files and component trees flat. Split only at stable visual or product
  boundaries, not to make every JSX block a component.
- Derive view state and handlers before JSX. Keep markup easy to scan.
- Prefer direct code over a helper used once. Extract repeated policy or
  behavior, not merely repeated syntax.

## Classify markup components

Apply this classification by default. Classify every component that encapsulates
an XML or JSX tag tree as one of two roles, except for the narrowly defined
trivial-component exception below:

### Layout component

- Plan regions, hierarchy, ordering, placement, sizing, overflow, and responsive
  behavior.
- Compose child regions and responsibility components.
- Keep product logic, data access, effects, and detailed interactions out.
- Name regions by layout purpose, such as `HeaderSection`, `SidebarSection`, and
  `ContentSection`.
- When using React Component Kit `layout`, name the generated layout object
  `Layout`.
- Pair each `Layout.Region` placement wrapper with the feature's corresponding
  responsibility component named `<Feature><Region>`. For a feature named
  `XxxForm`, write:

```tsx
<Layout.Bottom>
  <XxxFormBottom />
</Layout.Bottom>
```

Keep `Layout.Bottom` responsible for placement and sizing; keep
`XxxFormBottom` responsible for the feature content and behavior. Name other
region implementations with the same `<Feature><Region>` pattern.

Do not use compound names such as `XxxForm.Bottom`. Do not put product logic,
data access, effects, or detailed interactions directly in `Layout` or
`Layout.Region`.

### Responsibility component

- Implement one concrete product or interaction responsibility, such as search,
  task actions, a results table, or an account menu.
- Own only the state, data, effects, and handlers required by that
  responsibility.
- In an RCK project, encapsulate the component's semantic elements, base styles,
  and state-dependent styles with module-scope `rck.tag` or `Rck` components.
- Group those local styled primitives in `L`; reserve `Layout` for the RCK
  region layout.
- Express visual state through inferred semantic props such as `selected`,
  `busy`, `tone`, or `status`. Put their class rules in the RCK configuration
  instead of assembling conditional class strings in JSX.
- Avoid arranging sibling page regions or taking over parent-level layout.
- Extract a nested layout component when its internal region planning becomes a
  separate concern.

```tsx
const L = {
  Container: rck.form(
    'flex items-center gap-3',
    { busy: 'pointer-events-none opacity-60' },
  ),
  Submit: rck.button(
    'rounded px-3 py-2',
    { disabled: 'cursor-not-allowed opacity-50' },
    {
      tone: {
        primary: 'bg-primary text-white',
        danger: 'bg-danger text-white',
      },
    },
  ),
};

const XxxFormBottom = ({ busy, tone }: Props) => (
  <L.Container busy={busy}>
    <L.Submit disabled={busy} tone={tone}>Submit</L.Submit>
  </L.Container>
);
```

Keep region placement classes on `Layout.Region`, not on the responsibility
component. Keep the responsibility component's own presentation and visual
states in its `L` primitives.

Do not mix both roles by default. Split the component when it both plans regions
and implements a product responsibility.

### Trivial-component exception

Allow both roles in one component only after confirming that the component is
genuinely trivial. A small left/right row with a middle ellipsis is a typical
example. Require all of these conditions:

- The markup is short, flat, and understandable at a glance.
- The layout is one local arrangement, without named regions, complex
  responsive rules, track planning, or overflow coordination.
- The behavior is limited to rendering and simple event forwarding; it has no
  data loading, effects, context orchestration, or non-trivial state.
- Neither the layout nor the responsibility has independent reuse or testing
  value.
- Keeping them together is clearer than introducing another component boundary.

Reassess whenever the component grows. Split it immediately when any condition
stops being true. Do not apply this exception to an explicit RCK `Layout` and
its named regions; keep those strictly separated from feature components.

Classify at the current abstraction level: a responsibility component may use
small local Flexbox or Grid arrangements without becoming a layout component.

## Build layout and style

- Prefer semantic HTML and the repository's existing primitives.
- For a substantial component, group stable styled primitives in a module-scope
  `L` object with names such as `Container`, `HeaderSection`, and `ActionButton`.
- Prefer RCK boolean and variant configurations over ternaries, template
  strings, arrays, or `clsx` calls for state-dependent styles that RCK can
  represent directly.
- Use Grid for explicit page regions and Flexbox for one-dimensional alignment.
  Define sizing and overflow deliberately; do not add `overflow-auto` blindly.
- Use existing tokens and utility classes. Do not invent colors, spacing names,
  or a second styling system.
- Choose a clear visual direction from the product context, then express it
  consistently through typography, spacing, color, shape, depth, and motion.
- Handle narrow widths, keyboard focus, contrast, touch targets, loading, empty,
  error, and reduced-motion states.

## Organize React behavior

Plan component state before implementing its consumers. For a substantial
component tree, classify and layer it as follows:

| Layer | Ownership | React Context mapping |
| --- | --- | --- |
| External | Public props and callbacks supplied by the parent | `ctx.props` |
| Internal model | Autonomous component state and generated setters | `ctx.model` |
| Internal shared hooks | Shared infrastructure instances/results, especially React refs and TanStack React Query | `ctx.hooks` |

- Create one set of factories per component ownership boundary and define them
  at module scope.
- Compose providers from least dependent to most dependent: external props,
  internal model, then internal hooks.

```tsx
const Xxx = withXxxProps(
  withXxxModel(
    withXxxHooks(RawXxx),
  ),
);
```

- Let `ctx.hooks` consume outer props and model when needed. Never make an outer
  layer depend on an inner layer.
- Keep the hooks map stable and unconditional. Put shared refs and React Query
  hooks there so descendants consume the same resolved instances and results.
- Use `ctx.model` for simple autonomous state.
- Do not put reducer, validation, Effect, or domain-operation Hooks in
  `ctx.hooks`. Call them directly from the single-responsibility component that
  owns the behavior.
- Keep the three consumer hooks distinct so call sites can consume only the
  layer they need.
- Keep ownership component-scoped; do not turn these layers into an
  application-global store by default.
- Make effect dependencies, cleanup, cancellation, and stale-request behavior
  explicit.
- Keep network, persistence, and scheduling side effects outside visual
  primitives.
- Preserve inferred native props and refs; avoid `any` at public boundaries.

### Invoke Auto Hooks exactly once

Treat every Auto Hook as bound to one corresponding component:

- Name it by component and automatic responsibility, for example
  `useTaskAutoSelectFirstAvailable`.
- Call it exactly once, unconditionally, at the top level of that corresponding
  component, before early returns and JSX.
- When contexts wrap the component, call it in the corresponding raw component
  rendered below those providers.
- Never call it from `ctx.hooks`, another Hook, a parent or child component, a
  condition, a loop, a callback, or an event handler.
- Allow several different Auto Hooks in one component when responsibilities are
  separate, but call each one exactly once.

```tsx
const RawTask = () => {
  useTaskAutoSelectFirstAvailable();
  return <TaskView />;
};
```

## Control source file size

Apply these limits to every handwritten frontend implementation file:

- Target at most 100 physical lines after normal formatting.
- Treat 150 physical lines as an absolute maximum. Do not finish a change with
  an implementation file over this limit.
- Start evaluating a split as the file approaches 100 lines. Split along real
  ownership boundaries: layout, feature region, single responsibility, hook,
  service, or reusable type.
- Keep each responsibility component together with its local RCK `L`
  primitives. Do not move styles into an unrelated file only to reduce the line
  count.
- Do not compress formatting, combine unrelated statements, remove useful
  types, or hide logic in dense expressions to satisfy the limit.

Exclude generated, vendored, and snapshot files. If an existing file is already
over 150 lines and the requested edit touches it materially, include a
responsibility-based split in the change.

## Verify

Run the narrowest relevant checks, then expand according to risk:

1. Type-check edited packages and applications.
2. Run focused tests and lint edited files.
3. Build the consuming frontend.
4. Inspect the result at representative viewport widths when visuals changed.
5. Confirm keyboard behavior, focus, async failure paths, and cleanup that types
   cannot prove.
6. Check the physical line count of every edited implementation file.
7. Confirm new custom utilities were necessary and no adopted library already
   provides them.
