# Grin Syntax Signature (draft)

> This is a living document. It will change constantly during implementation.
> **Logic/runtime behavior is not specified yet — still in the philosophy phase.**

---

## Comments

```
// single-line comment
/* multi-line
   comment */
```

---

## Window (root element)

> 🚧 PLACEHOLDER — not yet designed

---

## Layout

> 🚧 PLACEHOLDER — not yet designed

---

## Components

Components use **bracket semantics** — the bracket shape declares the nature of the content:

| Bracket | Role | Example |
|---------|------|---------|
| `< >` `</>` | **Structure** — what component | `<Card>` `</Card>` |
| `{ }` | **Expression** — visual properties, layout | `{layout: row, gap: 16}` |
| `[ ]` | **Data** — values, bindings | `[value: $user.name]` |
| `( )` | **Action** — event handlers | `(click: account.follow)` |

```
<Card> {layout: row, gap: 16, pad: 20, bg: "surface-mid", radius: 12}
    
    <Image> [src: $user.avatar, alt: "Profile"]
        {size: 48, ratio: "1:1"}
    
    <Stack> {layout: col, gap: 4, flex: 1}
        <Text> [value: $user.name]
            {size: 18, weight: "bold", color: "main"}
        <Text> [value: $user.bio]
            {size: 14, color: "muted"}
    </Stack>
    
    <Button> [label: "Follow"]
        (click: account.follow)
        {variant: "primary", width: "auto"}

</Card>
```

### Built-in components

| Component | Data `[]` | Action `()` |
|-----------|-----------|-------------|
| `Window` | `[title]` | — |
| `Label` / `Text` | `[value]` | — |
| `Input` | `[bind: $var, placeholder]` | `(change:)` |
| `Button` | `[label]` | `(click:)` |
| `Image` | `[src, alt]` | — |
| `List` | `[bind: $collection]` | — |
| `Stack` | — | — |
| `Grid` | — | — |
| `Card` | — | — |

### Layout containers (`{ }`)

```
{layout: row | col | grid, gap: N, pad: N, flex: N}
```

---

## Style (`.grs` — overrides expression defaults)

`.grin` provides default `{ }` values. `.grs` overrides them with standard CSS-like selectors:

```
Card {
    bg: "surface-dark"
    radius: 8
}

Button.primary {
    bg: "#4a90d9"
    color: white
    pad: 8 16
}

Text.muted {
    color: "#888"
}
```

- Select by component type: `Card { }`
- Select by variant: `Button.primary { }`
- Select by component + variant: `Card.highlight { }`
- `.grs` always wins over `.grin` defaults.

---

## Logic (`.gl` — the 4th-generation language)

> Brace syntax. Concise, unambiguous. Not Rust-level strict.
> Every character must earn its place — humans and AI both pay by the token.

**Design constraints:**
- Full .NET interop — any C# assembly, any NuGet package, callable from `.gl`
- Transpile to C# (Phase 1), LLVM native later (Phase 2) — Kotlin's proven path
- Syntax must be human-readable first, machine-parseable second — but never at war with each other

**Casing convention (enforced at compile time):**
- `variableName` — camelCase. Always a value.
- `TypeName` / `FunctionName` — PascalCase. Always a type or function.
- **First-letter uppercase on a variable is a compile error.** The compiler will treat it as a type reference.
- No linter needed. Casing *is* semantics. Python proved it works. Go proved it works. Now it graduates from convention to law.

**Placeholder sketch (subject to change):**
```
logic Counter {
    state count: int = 0

    fun increment() {
        count = count + 1
    }

    fun reset() {
        count = 0
    }
}
```

---

## Implementation Status

- [x] Lexer (tokenization)
- [ ] Parser (AST generation)
- [ ] C# transpiler (Phase 1 — .NET interop via Roslyn)
- [ ] LLVM native compiler (Phase 2 — after language maturity)

