# Grin — Inborn Design Problems

> Structural problems baked into the language model itself, not documentation or
> marketing issues. These can't be fixed without changing what Grin fundamentally
> *is*. Working notes for later resolution.

---

## The through-line

Problems 1, 2, and 5 are the same disease: **the three-way wall has dead zones
wherever two concerns legitimately need to meet.**

- logic × structure → dynamic UI (conditionals, modals, routing)
- data × style → computed visuals (progress width, charts, interpolated color)
- all-three × instance → custom components with props

The web avoids these because its separation is a *convention you can break*, not a
*wall the compiler enforces*. The moment you enforce it absolutely, every natural
intersection becomes a homeless construct.

**Core question to resolve:** is the hard compile-time wall the point, or is the
structure/style/logic *clarity* the point? If it's the wall, the language will
spend its life building costumes to smuggle concerns across it. If it's the
clarity, there's more design freedom than the spec currently allows itself.

---

## 1. The dynamic-structure dead zone (highest priority)

`.grin` forbids logic. `.gl` forbids components. So where does conditional and
dynamic structure live?

```
// Need: show AdminPanel only if user is admin.
// .grin? No logic allowed — can't write `if`.
// .gl?   No components allowed — can't write <AdminPanel>.
```

Lists (`bind: $orgs as org`) are the *only* dynamic structure the spec allows.
Real UIs also need:

- conditional rendering (`if` / `else` / show / hide)
- modals & dialogs that mount and unmount
- routing / view swapping
- forms whose fields depend on a previous answer

Every one is "structure that depends on state," and the strict wall gives it no
home. This is a direct consequence of the founding rule, not a gap to patch.
Likely outcomes, both of which crack the wall:

- a `<If>` / `<Show>` pseudo-component → logic smuggled into `.grin` in a costume
- a `render`-like construct in `.gl` → components smuggled into `.gl`

**This is the problem that decides whether Grin can build a real app at all.**

---

## 2. The data-driven style dead zone

`.grs` can't see data. `.grin` can't hold style. `.gl` can't touch style
properties. So where does a computed visual value go?

```
// progress bar: width should equal $percent
// width: $percent%   ← belongs in style, needs data. Nobody can write it.
```

`:state` selectors handle **discrete** visual states (`:loading`, `:disabled`)
fine. They fail completely for **continuous or computed** values:

- progress width / fill
- charts, sparklines, heatmap cells
- color interpolated from a score
- `transform: translateX($dragX)` during a drag

Pre-declaring every value as a named state is combinatorial nonsense. The split
assumes "style is static, data is dynamic, they never touch." False for any
data-visualization or interactive UI.

---

## 3. Separation of *files* ≠ separation of *concerns*

To understand one button you must read three files:

```
.grin:  <Button id: "follow-btn" [label: "Follow"]>
.grs:   #follow-btn:loading { opacity: 0.6 }
.gl:    followBtn.click { loading = true }
```

The three concerns are as tightly coupled as ever — joined by the string
`follow-btn` across three files. Nothing was decoupled; one cohesive thing was
scattered across three locations connected by stringly-typed magic names.

- Rename the button → edit three files.
- Understand the button → open three files and reassemble it mentally.

The web spent ~15 years *walking back* from this exact model — CSS-in-JS,
styled-components, Tailwind, single-file Vue components, colocated JSX — because
**locality of behavior beats separation of technology**. Things that change
together should live together. Grin enforces, at compile time with no escape, the
precise model the React/Vue/Svelte communities abandoned by choice.

"HTML/CSS/JS separation is the right architecture" is the founding premise, but
the web itself stopped believing it. That's a load-bearing assumption the most
relevant prior art contradicts.

---

## 4. kebab ↔ camel auto-conversion is a silent collision machine

`follow-btn` → `#follow-btn` → `followBtn`. The compiler name-mangles across the
file boundary. Consequences:

- `follow-btn` and `followBtn` collide.
- `user-list` and `userList` collide.
- `userID` vs `user-id` vs `user-i-d` — acronyms, numbers, and consecutive caps
  are edge cases every framework eventually regrets.
- Rename refactoring must be compiler-aware across three files *and* a name
  transform. Plain find-and-replace breaks it.

The linking convention is implicit and lossy. The "type-safe bindings caught at
compile time" promise sits on top of a **stringly-typed, name-mangled linker**.
Those two ideas are in tension.

---

## 5. No user-defined components = it doesn't scale

The spec has built-ins (`Button`, `Card`, `Stack`, ...) and `class` for list-item
templates. There is no way to define a reusable component with parameters. A UI
language without composable custom components + props can't build beyond a demo.

Adding them exposes the question the wall creates:

- A custom `<UserBadge size verified>` has structure, style, AND logic.
- Do its three concerns split across three files *per component*? → file
  explosion (3 files × N components).
- How do **props** cross the wall? Props are data (`.grin`) but they parameterize
  style (`.grs`) and behavior (`.gl`). The wall has no concept of a per-instance
  value shared by all three files.

This is the scaling cliff. Lists alone won't hide it for long.

---

## 6. The wall has no escape hatch

Every rigid system survives on its pressure valve: CSS has inline `style`, HTML
has `<script>`, Rust has `unsafe`. Grin's selling point is that violations are
**compile errors, not lint warnings** — deliberately *no* valve.

UIs are the most irregular domain in software; there's always the one screen that
doesn't fit the model. A lint warning lets you ship with a `// TODO`. A compile
error leaves you stuck until the language itself grows a feature. Rigidity is the
brand, but rigidity with zero give is how languages accumulate hated workarounds
(`<If>` components, dummy state vars to fake dynamic style, etc.). The strictness
that's the headline will also generate the most day-to-day pain.

---

## 7. Error locality across the wall

A typo'd id in `.grin` surfaces as an error in `.grs` and `.gl`. Which file owns
the error? When `followBtn.click` binds to nothing, is that a `.gl` error or a
`.grin` error? Cross-file compile errors are notoriously bad UX (C++ templates,
early TypeScript path errors). The compile-time-safety promise guarantees these
will be frequent, and the three-file split turns every one into a "which file is
actually wrong?" puzzle.

---

## Priority order for fixing

1. **#1 dynamic structure** — existential; blocks real apps.
2. **#2 data-driven style** — existential for any interactive/data UI.
3. **#5 custom components + props** — the scaling cliff.
4. **#3 / #6** — philosophical; decide wall-vs-clarity, which informs 1/2/5.
5. **#4 / #7** — important but addressable once the model above is settled.

---

# Round 2 — after the slots / props / custom-component fixes

The spec was revised: slots for dynamic structure, `[value: $x]` data binding for
continuous visuals, custom components via three-file-by-basename + `$props`, and
exact string matching (no kebab↔camel conversion). Status of the original seven,
then the new problems those fixes introduced.

## Status of the original seven

| # | Problem | Status | Note |
|---|---------|--------|------|
| 1 | Dynamic structure | **Mostly solved** | Slots + `bind:` + `null`-unmount cover routing/modals/conditional mount. See new problems A, B. |
| 2 | Data-driven style | **Partial** | Works for built-ins that render internally (`ProgressBar`). Users still can't author data-driven visuals — dead zone moved, not closed. |
| 3 | Files ≠ concerns | **Unchanged (by choice)** | Wall kept deliberately. |
| 4 | Name mangling | **Solved** | Exact string match + quoted refs (`"follow-btn".click`). Tradeoff: `.gl` refs are now string literals, not identifiers. |
| 5 | Custom components | **Solved** | `$props` is the official per-instance gateway through the wall. 3×N file explosion accepted by choice. |
| 6 | No escape hatch | **Improved** | Slots/props/components are now managed valves. Still no break-glass for truly irregular cases — a bet, not a fix. |
| 7 | Error locality | **Unchanged** | Quoted refs make `.grin` references more visually obvious, but "which file owns the error" is still open. |

**Remaining through-line:** the hard cases all got pushed *inside* components, and
inside a component the three files still can't fully meet. That's why #2-residual
(users can't author data-driven visuals) and new problem A (slots can't
parameterize their children) exist. Same disease, deeper in.

---

## A. Slots can't pass data to what they mount (highest of the new ones)

```
// .gl
activeModal = SettingsDialog   // mounts the component...
                               // ...but for WHICH user's settings?
```

Slot assignment sets a component *type* but provides no mechanism to pass
arguments/props on mount. A dynamically mounted component can only read shared
`app` state — it can never receive parameters. This breaks modals and detail views
that need an argument (edit *this* item, show *this* user). Needs a prop-passing
syntax on slot assignment, or dynamic mounting is limited to parameterless views.

---

## B. Conditional show/hide of a single primitive is heavyweight

Slots are declared the **only** dynamic-structure mechanism. So conditionally
showing one validation `<Text>` requires wrapping it in a `<slot>` and assigning a
component type. Minting a component to show an error string is too much ceremony
and will produce slot-sprawl. Consider whether a plain component bound to a
nullable value can hide itself, short of a full slot.

---

## C. `.grs` now reads prop values — a data-aware crack on the style side

```
UserCard[compact=true] { pad: 8 }
#follow-btn[theme="dark"] { bg: "dark-accent" }
```

This contradicts the architecture table's "`.grs` cannot contain data." It's
reasonable (CSS attribute selectors do the same), but two things need pinning down:

- It only works for **discrete** prop values, not continuous data — consistent
  with B and #2-residual, but the wall is now officially data-aware on the style
  side and the spec should say so.
- `#follow-btn[theme="dark"]` implies a parent component's prop cascades into a
  *descendant* selector. That scoping mechanism (how a child element's style sees
  the parent component's prop) is unspecified.

---

## D. Components-as-values needs an explicit casing rule

```
currentView: LoginPage     // a PascalCase type stored in a camelCase variable
bind: $currentView         // ...then mounted
```

Slots make component types first-class assignable values. The casing rule says
PascalCase = type/function, camelCase = value — storing `LoginPage` in
`currentView` sits in the gap between those. The spec should state explicitly that
component types are assignable/passable values, or the casing rule and the slot
mechanism are in mild tension.

---

## Updated priority order

1. **A** — slots can't parameterize children. Blocks modals/detail views; existential for real apps.
2. **#2-residual** — let users author data-driven visual components, not just built-ins.
3. **B** — lightweight conditional for single primitives (avoid slot-sprawl).
4. **C / D** — specify the prop-in-`.grs` scoping and components-as-values rule.
5. **#3 / #6 / #7** — philosophical / tooling; settled or deferred by choice.

---

# Round 3 — after the slot-props / lightweight-conditional fixes

The spec was revised again: slot prop-passing (`Component(key: value)`), a
lightweight `[bind: $x]` null-hides conditional, discrete-only prop selectors with
descendant cascade, and an explicit components-as-values rule.

## Status of the Round 2 problems

| # | Problem | Status | Note |
|---|---------|--------|------|
| A | Slots can't pass data | **Solved** | `activeModal = UserEditor(user: user, mode: "edit")`. |
| B | Heavyweight conditional | **Solved, but spawned E** | `[bind: $x]` null → no render. Good instinct, overloaded `bind:`. |
| C | `.grs` reads prop values | **Solved** | Discrete-only stated; descendant cascade specified. |
| D | Components-as-values | **Solved** | Explicit: casing governs variable naming, not the value's type. |
| #2-residual | User-authored data-driven visuals | **Still open** | Fixes routed around it; see below. |

---

## E. `bind:` is overloaded four ways and statically undecidable (new)

The lightweight-conditional fix made `bind:` carry four distinct behaviors:

1. `Input [bind: $var]` — two-way data binding
2. `[bind: $source as alias]` — list iteration
3. `[bind: $errorMessage]` — render-if-non-null / show primitive as content
4. `[bind: $currentView]` (slot) — mount a component type

What `[bind: $x]` *does* depends on the **runtime type of `$x`**: `null` hides, a
primitive renders as text, a component type mounts. Two of Grin's own goals break:

- **"machine-parseable, unambiguous"** — you cannot tell what `[bind: $x]` does by
  reading it; you must know the runtime type of `$x`.
- **"Slots are the only mechanism for dynamic structure"** — the conditional's own
  rule says a component-typed bind "replaces the host, slot-like behavior on any
  element." So there are now two dynamic-structure mechanisms and "only" is false.

Smaller spurs off the same fix:

- **`[bind:]` vs `[value:]` overlap.** `<Text [value: $x]>` and `<Text [bind: $x]>`
  both put `$x` into a Text, but only `bind:` hides on null. Users won't reliably
  predict which to use.
- **Instantiation vs call ambiguity.** `UserEditor(user: user)` is syntactically
  identical to a function call, and both are PascalCase. Resolved by symbol kind,
  but the spec should state that a component invoked with args yields a configured
  component value, not a call result.

**Suggested direction:** split the overload. Keep `bind:` for data (two-way +
list); introduce a distinct keyword (`show:` / `mount:`) for conditional mount/show
so behavior is decidable from source alone. That also lets the "slots are the only
mechanism" claim be honestly kept or dropped instead of self-contradicting.

---

## #2-residual (reopened) — users still can't author data-driven visuals

The continuous-value section is unchanged: `[value: $percent]` works only because
`ProgressBar` is a **built-in** that renders `::fill` internally. A user-defined
gauge/sparkline/heatmap still has no path:

- `.grs` can't read continuous data (C confirmed prop selectors are discrete-only).
- No `::fill`-style hook is exposed to custom components.

So the original data × style dead zone persists exactly where it was — built-ins
can map data → style internally, user components cannot. The Round 2/3 fixes routed
*around* this rather than through it. Closing it needs one of: a continuous-value
style hook usable by custom components, a sanctioned data→style expression, or
accepting that data-driven visuals are a built-in-only capability (and saying so).

---

## Updated priority order

1. **E** — de-overload `bind:`; restore static decidability and the "only mechanism" claim.
2. **#2-residual** — decide the user-authored data-driven visual story (hook, expression, or built-in-only by design).
3. Everything from Rounds 1–2 marked solved/by-choice stays settled.
