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

> 🚧 PLACEHOLDER — not yet designed

---

## Common Properties

> 🚧 PLACEHOLDER — not yet designed

---

## Style (separate file: `.grs`)

> 🚧 PLACEHOLDER — not yet designed

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

