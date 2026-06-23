# Grin Syntax Signature (draft)

> This is a living document. It will change constantly during implementation.

---

## Architecture

Grin enforces 3-way separation **at compile time**. The file extension is the boundary.

| File | Role | Can contain | Cannot contain |
|------|------|-------------|----------------|
| `.grin` | Structure | `<Components>`, `[data]`, ids, classes | Style props, event handlers, logic |
| `.grs` | Style | `{ properties }`, `#id`, `.class`, `:state` | Components, logic |
| `.gl` | Logic | State, functions, event binding | UI components, style properties |

**You cannot write `{ color: red }` in `.grin`. You cannot write `<Button>` in `.gl`. Violations are compile errors, not lint warnings.**

---

## Comments

```
// single-line comment
/* multi-line
   comment */
```

---

## Bracket Semantics

The bracket shape declares the nature of the content:

| Bracket | Role | Used in |
|---------|------|---------|
| `< >` `</>` | **Structure** — what component | `.grin` only |
| `[ ]` | **Data** — values, bindings | `.grin` only |
| `{ }` | **Style** — visual properties | `.grs` only |
| `{ }` | **Logic** — state, functions, blocks | `.gl` only |

Same `{ }` characters, different semantic context based on the file extension they appear in.

---

## `.grin` — Structure

`.grin` declares **what exists and where**. No style. No events. No logic.

### Window (root element)

```
<Window id: "main" [title: "My App"]>
    ...
</Window>
```

### Layout

```
<Grid id: "root-layout" [cols: 200px 1fr, rows: auto 1fr auto]>
    ...
</Grid>

<Stack id: "sidebar" [layout: col, gap: 8]>
    ...
</Stack>
```

### Components

```
<Card id: "user-card">
    <Image id: "avatar" [src: $user.avatar]>
    <Text id: "username" [value: $user.name]>
    <Button id: "follow-btn" [label: "Follow"]>
</Card>
```

**Rules:**
- `id:` — unique identifier. Referenced in `.grs` as `#id` and in `.gl` as `"id"` (exact string match).
- `class:` — reusable identifier for list templates. Referenced in `.grs` as `.class` and in `.gl` as `"class"`.
- No kebab↔camel auto-conversion. The name is the name. Compiler enforces exact match across all three files.
- `[ ]` — data binding only. `$variable` for reactive values.
- No `{ }` blocks. No event handlers.

### Built-in components

| Component | Data `[]` |
|-----------|-----------|
| `Window` | `[title]` |
| `Text` | `[value]` |
| `Input` | `[bind: $var, placeholder]` |
| `Button` | `[label]` |
| `Image` | `[src, alt]` |
| `Grid` | `[cols, rows]` |
| `Stack` | `[layout: row\|col, gap]` |
| `Card` | — |

### List templates

Lists use `<template>` with `bind: $source as alias` for scoped access:

```
<Stack id: "org-list" [bind: $orgs as org]>
    <template>
        <Text [value: $org.name]>

        <Stack id: "user-list" [bind: $org.members as user]>
            <template>
                <Text [value: $user.name]>

                <Stack id: "badge-list" [bind: $user.badges as badge]>
                    <template>
                        <Image [src: $badge.icon]>
                        <Text [value: $user.name]>   // parent scope
                        <Text [value: $org.name]>    // grandparent scope
                    </template>
                </Stack>
            </template>
        </Stack>
    </template>
</Stack>
```

**Scoping rules:**
- `$alias.field` — direct reference. Works at any nesting depth.
- **Duplicate alias in nested scope is a compile error.** Use a different name.
- All `as` aliases in the current scope chain are automatically loaded into `.gl` event context.
- No `$1`, `$2`, `$parent` — only named aliases.

### Component naming (class)

```
<Button class: "follow-btn primary large-size" [label: "Follow"]>
```

| Token position | Role | `.grs` selector | `.gl` binding |
|----------------|------|-----------------|---------------|
| 1st | Binding name | `.follow-btn` | `"follow-btn".click` |
| 2nd+ | Style variant | `.primary`, `.large-size` | Can be used to disambiguate |

**Variant disambiguation:** when the same binding name appears multiple times in a template:

```
<template>
    <Button class: "btn primary" [label: "Confirm"]>
    <Button class: "btn secondary" [label: "Cancel"]>
</template>
```

```
// .gl
"btn".primary.click { ... }
"btn".secondary.click { ... }
```

### Event context

All `as` aliases in the scope chain are automatically loaded into the event parameter:

```
// .grin: <Stack [bind: $orgs as org]> ... <Stack [bind: $org.members as user]>

// .gl
"badge-image".click(event) {
    badge = event.badge   // clicked item
    user  = event.user    // parent scope (as user)
    org   = event.org     // grandparent scope (as org)
}
```

No manual chain traversal. No parameter explosion. `as` names become the event context.

---

### Slots & dynamic structure

Slots declare places in the `.grin` tree that `.gl` fills at runtime. This is how conditional rendering, modals, and routing work without `.grin` knowing about logic.

```
// .grin — slot: declares "a component may be here"
<Window id: "app">
    <slot id: "sidebar">
    <slot id: "body" [bind: $currentView]>
    <slot id: "modal" [bind: $activeModal]>
</Window>
```

```
// .gl — fills slots, controls what is mounted
App {
    currentView: LoginPage
    activeModal: null

    onLoginSuccess(user) {
        currentView = HomePage
    }

    openSettings() {
        activeModal = SettingsDialog
    }

    closeModal() {
        activeModal = null   // unmount
    }
}
```

**Rules:**
- `bind:` on a slot → `.gl` controls which component type fills it.
- `null` → nothing mounted (conditional hide, modal close).
- Component type assigned → component mounts with its own `.grin`/`.grs`/`.gl` active.
- Static slots (no `bind:`) → parent `.grin` fills directly, used for layout composition.
- Slots are the **only** mechanism for dynamic structure. No `<If>`, no `render()`, no logic in `.grin`.

### Continuous value binding

Boolean states use `:state` selectors in `.grs`. Continuous values (progress, colors, transforms) use `[ ]` data bindings in `.grin`:

```
// .gl — sets the value, knows nothing about how it's displayed
FileDownload {
    percent: 0

    onProgress(bytes, total) {
        percent = bytes / total * 100
    }
}

// .grin — binds the value to a component that knows how to render it
<ProgressBar id: "download-bar" [value: $percent, max: 100]>

// .grs — static shell only, component handles the fill internally
#download-bar {
    height: 6, radius: 3
    bg: "surface"
}
#download-bar::fill {
    bg: "accent"
    transition: width 300ms ease
}
```

| Value type | Mechanism | Where |
|------------|-----------|-------|
| Boolean (`loading`, `disabled`) | `:state` selector | `.grs` |
| Continuous (`percent`, `color`) | `[ ]` data binding | `.grin` |
| Static (`radius`, `font-size`) | `{ }` property | `.grs` |

### Custom components

Custom components are defined by convention: three files sharing a base name.

```
// UserCard.grin
<Card id: "user-card">
    <Image id: "avatar" [src: $props.user.avatar]>
    <Text id: "name" [value: $props.user.name]>
    <Button id: "follow-btn" [label: "Follow"]>
</Card>

// UserCard.grs
UserCard[compact=true] {
    pad: 8
    font-size: 14
}
#follow-btn[theme="dark"] {
    bg: "dark-accent"
}

// UserCard.gl
UserCard {
    user: $props.user

    init {
        loadProfile(user)
    }
}
```

**Props rules:**
- Parent passes props via `[key: value]` on the component.
- `$props` is read-only inside the component's `.grin` and `.gl`.
- `.grs` selects by prop values with `ComponentName[prop=value]` syntax.
- Props are the only way a parent communicates with a child component.
- No prop type declaration — casing convention handles it (camelCase = value, PascalCase = type).

**Usage:**
```
// .grin
<UserCard [user: $currentUser, theme: "dark", compact: true]>
```

---

## `.grs` — Style

`.grs` defines **how it looks**. Select by id, class, component type, or state.

### Selectors

```
// By id
#user-card {
    layout: row, gap: 16, pad: 20
    bg: "surface", radius: 12
}

// By class (list items)
.user-card {
    layout: row, gap: 12, pad: 16
    bg: "surface", radius: 8
}

// By component type
Button {
    variant: "primary"
    transition: 200ms
}

// By component type + class
Button.primary {
    bg: "#4a90d9"
    color: white
}
```

### State selectors

State variables declared in `.gl` become selectors in `.grs`. `.grs` defines the visual consequence — `.gl` owns when the state changes.

```
#follow-btn:loading {
    opacity: 0.6
    cursor: "wait"
}

#follow-btn:disabled {
    bg: "muted"
    color: "disabled"
}

#user-card:selected {
    border: 2 solid "accent"
}
```

- State names (`loading`, `disabled`, `selected`) are defined in `.gl`.
- `.grs` does not declare states — it only reacts to them.
- State selectors apply automatically when the corresponding variable changes.

### Override priority

1. `.grs` component type — lowest
2. `.grs` class selector
3. `.grs` id selector
4. `.grs` state selector — highest

(`.grin` has no style blocks at all, so there is nothing to override.)

---

## `.gl` — Logic

`.gl` is the 4th-generation successor to C#. **Brace syntax. Concise, unambiguous. Not Rust-level strict.**

### Casing convention (enforced at compile time)

- `variableName` — camelCase. Always a value.
- `TypeName` / `FunctionName` — PascalCase. Always a type or function.
- **First-letter uppercase on a variable is a compile error.** The compiler will treat it as a type reference.
- No linter needed. Casing *is* semantics. Python proved it works. Go proved it works.

### Event binding

`.gl` binds to `.grin` components by id or class. `.grin` does not know about these bindings.

```
// Static: bind by id (exact string match — no case conversion)
"follow-btn".click {
    account.follow(user)
    loading = true
}

// List: bind by class, receives the item
"user-card".doubleClick(item) {
    navigate.toProfile(item.id)
}

// Lifecycle
init {
    account.loadData()
}
```

- `"id".event` — for unique components (exact string match, no name conversion).
- `"class".event(item)` — for list template instances. `item` is the current list element.
- `.gl` has zero knowledge of the UI tree, layout, or styling.

### State variables

State variables declared in `.gl` are automatically exposed as `:state` selectors in `.grs`.

```
UserProfile {
    user: $user
    loading: false
    saved: false

    "save-btn".click {
        loading = true
        await api.save(user)
        loading = false
        saved = true
    }
}
```

When `loading = true`, `.grs` rules for `#save-btn:loading` apply automatically. `.gl` does not touch style properties — it only sets state.

### Cross-component communication

Components never talk to each other directly. They communicate through **shared state**:

```
// app namespace — shared between all components
App {
    notifications: []

    notify(message) {
        notifications.add({ text: message, ts: now() })
    }
}

// Component A — sets state, knows nothing about who reads it
UserCard {
    user: $user

    "follow-btn".click {
        await account.follow(user)
        app.notify(user.name + "님 팔로우 시작")
    }
}

// Component B — reads state, knows nothing about who sets it
NotificationCenter {
    list: app.notifications

    item.dismiss(notif) {
        app.notifications.remove(notif)
    }
}
```

- Components share state through a common namespace (`app`, or any user-defined scope).
- The sender writes to a shared variable. The receiver reads from it.
- Neither component imports, references, or depends on the other.
- `.grin` places both on screen. The only shared surface between components is state.

### Design constraints

- Full .NET interop — any C# assembly, any NuGet package, callable from `.gl`.
- Transpile to C# (Phase 1), LLVM native later (Phase 2) — Kotlin's proven path.
- Human-readable first, machine-parseable second — never at war with each other.

---

## Implementation Status

- [x] Lexer (tokenization)
- [ ] Parser (AST generation)
- [ ] C# transpiler (Phase 1 — .NET interop via Roslyn)
- [ ] LLVM native compiler (Phase 2 — after language maturity)

