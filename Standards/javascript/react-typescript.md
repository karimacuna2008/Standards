# React + TypeScript

Applies to any React frontend, regardless of Code author (Prompts or Direct).
Complements `prompt-driven-development/architecture.md`, which owns the general
thin-orchestrator and design-token rules; this file states how they land in
React/TypeScript specifically.

## Folder layout

```
src/
  main.tsx            providers and mount only
  App.tsx             route tree only
  index.css           @theme — the ONLY file allowed to contain color literals
  env.ts              environment variables, parsed and validated at boot
  types/              one file per domain, derived from the schemas
  api/                one file per domain + http.ts, config.ts, schemas/
  lib/                own reusable code with NO React import
  components/ui/      design-system primitives
  components/async/   loading and error boundaries shared by every module
  forms/              the shared form hook and its helpers
  modules/<Module>/   one folder per product module
```

- Name the product folder `modules/` or `features/` — pick one per project and
  never mix. Match whatever the project's own documentation calls them.
- `lib/` means strictly "own reusable code that does not import React".
  Enforce with a lint rule, not with a grep somebody has to remember to run.
- Put UI-but-shared code (tour overlays, boundaries, form hooks) in its own
  top-level folder, never in `lib/`.
- Keep the `api/` surface in one language. Do not mix English and the project's
  spoken language in file names.

## File size and composition

- One component per file.
- No `.tsx` over ~250 lines. Enforce as a review gate with
  `find src -name '*.tsx' | xargs wc -l | sort -rn | head`.
- Exactly one file per module carries a leading underscore: the module's thin
  canvas (`_<Module>.tsx`), state and wiring only. See `architecture.md`.

State the line threshold as a number in the project's verification criteria. An
adjective like "thin" is not checkable; a number is. Files reach four digits
because nothing detects "too big" before it has already happened.

## Design tokens — Tailwind 4

Define tokens in three layers in `index.css`:

1. **Raw scale** (`--color-brand-*`) — the only place in the project with color
   literals. Use `@theme`.
2. **Semantic aliases** (`--color-surface`, `--color-text`, `--color-accent`) —
   what components actually write. Use **`@theme inline`**.
3. **Non-color scales** (type scale, spacing, radius, shadow, breakpoints) —
   use `@theme`.

Rules:

- Components use semantic tokens only. Never a literal, never the raw scale,
  never a built-in palette name.
- Use `@theme inline` for any token whose value references another variable that
  gets redefined in a narrower scope (a theme scope, a preview scope, dark
  mode). Plain `@theme` emits the variable into `:root`, so it resolves against
  `:root` and silently keeps the wrong value. It does not throw; it just paints
  wrong.
- Never assume a framework's built-in palette matches a hex you were given.
  Tailwind 4 re-authored its default palette in OKLCH: its named colors are not
  the v3 hex values. Write the brand scale explicitly.
- Never build a class name at runtime. `` `bg-${color}-50` `` is purged at build
  and the element loses its background with no error. Use a static
  `Record<K, string>` map.
- Per-instance dynamic color goes through a CSS variable written into a scoping
  element's `style`, consumed as `bg-[var(--x)]` — that form is literal in the
  source, so it survives purging.

## Data layer

One file per domain under `api/`. Every domain file has the same shape.

- Expose queries with TanStack Query's **`queryOptions()`**, not a separate file
  of bare keys. It co-locates key, function and options in one typed object and
  flows types to `useQuery`, `prefetchQuery`, `getQueryData` and `setQueryData`.
- Put cross-cutting request concerns (base URL, auth header, tenant header,
  JSON handling, error construction) in `http.ts` once. Never in a module.
  A per-module header is a header somebody eventually forgets, and if it selects
  the tenant, forgetting it is a data leak.
- Define an `ApiError` with `status`, and optional `code`, `body` and
  `fieldErrors`. Without it the UI cannot tell 401 from 403 from 422 from 5xx
  and every failure collapses to "something went wrong".
- Validate responses at the boundary with a schema (Zod or equivalent), and
  derive the TypeScript types from the schemas with `z.infer`. Hand-writing the
  type next to the schema creates two truths that drift silently.
- Map field naming explicitly, field by field, in the schema. Do not auto-convert
  `snake_case` to `camelCase`: an automatic rule gets `buho_order_id` right and
  `whatsapp_phone_id` -> `phoneId` wrong, and the failure is a silent
  `undefined`. Explicit mapping is checkable against the API's own docs.
- Parse and validate environment variables once at boot. A missing base URL must
  fail loudly at startup, not produce `undefined/some/path` and a mystery 404.

### Mocking a backend that does not exist yet

- Keep the mock/real seam behind one predicate per domain, defaulting to mock,
  so it is impossible to accidentally call an endpoint that was never built.
- Make that switch table a mirror of the project's technical-debt file. Flipping
  one flag is what "the endpoint shipped" looks like.
- Mocks must add latency. Instant mocks mean no loading state is ever exercised,
  and all of them appear broken the day the real backend lands.
- Mocks must be able to **fail on purpose**, not only be slow. Otherwise the
  error path is unreachable and untested.
- Mocks must return through the same post-processing as the real branch, and
  construct the same error type. That keeps the option of moving to a
  network-level mock (MSW) later without touching a single module.

## Loading and error contract

Ship this in the foundation, before any product module exists:

- A query client with explicit defaults: no retry on 4xx, backoff on 5xx, a
  declared `staleTime`, a declared `throwOnError` policy.
- A global `onError` on both the query cache and the mutation cache.
- One route-level error boundary.
- One reusable `<QueryBoundary>` / async-state component.

Without this, every module invents its own `if (isPending) return <p>Loading…</p>`
and its own error rendering, and they will not match. This is the same
duplication problem as an oversized component, one altitude up. Generated code
produces exactly this pattern when it is not given the contract.

## Forms

Any app whose screens are "load config -> edit -> save" needs one decided form
pattern in the foundation. Without it every batch invents its own dirty
tracking, its own unsaved-changes guard and its own validation wiring.

Provide one shared hook that owns the whole cycle: query -> form with
`defaultValues` from the query -> dirty state -> mutation -> invalidation ->
navigation guard -> server validation errors mapped to per-field errors.

Map the backend's validation-error shape to per-field errors. It is under an
hour of work in the foundation and it is the difference between "something went
wrong" and the offending field highlighted, in every module.

## TypeScript configuration

Always on:

`strict`, `noUnusedLocals`, `noUnusedParameters`, `noUncheckedIndexedAccess`,
`verbatimModuleSyntax`, `erasableSyntaxOnly`, `resolveJsonModule`,
`isolatedModules`, `skipLibCheck`.

- `noUncheckedIndexedAccess` is the flag that catches blind `Record<string, T>`
  indexing. It generates the most friction with generated code. Keep it: the
  friction is the point.
- Split into `tsconfig.app.json` and `tsconfig.node.json` so build config files
  are not type-checked with DOM libs.
- Point the path alias at `./src`, never at the project root. Aliasing the root
  lets imports reach `node_modules` and config files.

## Gates

Define a single `verify` script that runs type-check, lint and tests. That is
what a batch must pass before review.

- Encode project-specific hard rules as lint rules, not as greps in a checklist.
  A rule fails in the editor the moment the offending line is written; a grep
  fails days later if somebody remembers to run it. Anything stated as "never do
  X" in the project's architecture doc belongs in `no-restricted-syntax` or
  `no-restricted-imports`.
- Always include the hooks plugin. Its exhaustive-deps rule catches a whole
  class of generated-code bugs.
- Write greps to match both quote styles and all literal forms. A pattern that
  only matches single quotes or only 6-digit hex gives false confidence.

## Testing scope

Do not require component tests to start. Require this instead:

- **Pure functions in `lib/` get tests.** Layout math, formatters, mappers,
  reducers.
- UI is verified by the project's manual checklist.

Ship the runner and two or three example tests in the foundation, so later
modules inherit a runner instead of arguing about one. The highest-value test
target is always the pure numeric code that several components depend on — that
is where duplicated copies drift and produce bugs nobody traces back.

## Shared math and magic numbers

When several components depend on the same computation, give it one folder that
owns it, and export the constants it uses. Do not hardcode the constants inside
the formula.

Duplicated formulas are not the bug. Duplicated formulas that **drift** are the
bug. Exporting the constants is what makes drift impossible.

Create that folder with its contract written down (a README naming what it owns)
even before its implementation exists, so the first consumer has something to
cite instead of inventing its own copy.

## React 19

- No `forwardRef`. `ref` is a normal prop.
- `useRef` requires an argument.
- Prefer the compiler over manual memoization. Do not scatter `useMemo` and
  `useCallback` by default; add them only when a measurement asks for it.
- Never write a render-time side effect into a mutable module singleton. With
  StrictMode double-invocation and concurrent rendering, a route transition can
  leave the singleton pointing at the previous value while requests for the new
  one are in flight. If that value selects a tenant, it is a data leak. Pass it
  as a parameter and put it in the query key.

## Dependency policy

- Pin the major of anything whose API you are porting from existing reference
  code. A package's `latest` may be a rewrite.
- Check `dist-tags` before choosing a version. A `legacy` tag usually means the
  ecosystem's examples are still on the older major.
- Check whether a `@types/*` package is a deprecated stub before adding it.
  Modern packages ship their own types.
- Prefer the single consolidated package when a library offers one.
- Verify peer dependencies before pinning a build plugin: plugin majors often
  hard-require a specific bundler major.
- Never carry over a dependency that came from a scaffolding tool and is not
  used by the app.

### Choosing between a newer option and an older one

When the code author is an external generator, weigh **corpus size**, not
release date. A library the generator has seen thousands of times produces
working code; a library that became popular after its training produces
invented APIs that look plausible and fail at runtime.

Measure instead of guessing: ask the generator to **write code** with the
library and look at what API it emits. Asking "do you know X" gets a yes.

When you deliberately choose the older option, record it in the project's
technical-debt file with the reason and a revisit condition, so it does not get
re-litigated every time somebody notices something newer exists.

## Prompt template header (Code author = Prompts)

Start every prompt with a fixed block. The generator is stale in specific,
measurable places, and those places are not visible in its own summary.

The block states:

- exact versions of the stack;
- forbidden APIs, named individually — superseded library APIs, deprecated
  import paths, built-in palette names, runtime-built class names, and any
  framework idiom the current major replaced;
- required dialects, where a library's recent major changed syntax;
- the project's own contracts — which shared component handles loading and
  error, which hook handles forms, the canvas rule, the file size limit.
