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
- `id:` — unique identifier. Exposed to `.grs` as `#id` and to `.gl` as `camelCaseId`.
- `class:` — reusable identifier for list templates. Exposed to `.grs` as `.class` and to `.gl` as `className`.
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

Lists use `<template>` with `class:` and `$.` scope variables:

```
<Stack id: "user-list" [bind: $users]>
    <template>
        <Card class: "user-card">
            <Image [src: $.avatar]>
            <Text [value: $.name]>
            <Button class: "follow-btn" [label: "Follow"]>
        </Card>
    </template>
</Stack>
```

- `$.avatar`, `$.name` — scoped to the current template item.
- `class:` not `id:` — templates produce multiple instances, classes are reusable.
- The binding source (`$users`) is declared once on the parent container.

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
// Static: bind by id
followBtn.click {
    account.follow(user)
    loading = true
}

// List: bind by class, receives the item
userCard.doubleClick(item) {
    navigate.toProfile(item.id)
}

// Lifecycle
init {
    account.loadData()
}
```

- `id.event` — for unique components (camelCase conversion: `follow-btn` → `followBtn`).
- `class.event(item)` — for list template instances. `item` is the current list element.
- `.gl` has zero knowledge of the UI tree, layout, or styling.

### State variables

State variables declared in `.gl` are automatically exposed as `:state` selectors in `.grs`.

```
UserProfile {
    user: $user
    loading: false
    saved: false

    saveBtn.click {
        loading = true
        await api.save(user)
        loading = false
        saved = true
    }
}
```

When `loading = true`, `.grs` rules for `#saveBtn:loading` apply automatically. `.gl` does not touch style properties — it only sets state.

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

