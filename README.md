# Grin

> Kotlin is to Java what Grin is to C# — the 4th generation.
> HTML/CSS is the correct architecture. Desktop deserves it too.

---

## Why

### XAML is not a language for humans

If you've written CSS, you will never, ever build a desktop app with XAML again.

XAML takes the worst of XML — angle brackets swallowing your screen, attributes nested inside attributes inside attributes — and calls it a UI framework. Styling is trapped in `<Style>` blocks. Layout is hidden in `Grid.RowDefinitions` boilerplate. Every styling change requires a recompile. It's a format designed for Visual Studio's property grid, not for developers who actually ship products.

The web figured this out twenty years ago. Structure goes in one file. Style goes in another. Logic goes in a third. It's not a "best practice" — it's basic hygiene. And yet the .NET ecosystem still forces developers to choose between this and native performance.

### Electron proved the architecture was right

Electron is slow. Electron is bloated. Electron eats RAM for breakfast. And yet it won.

Not because of JavaScript. Not because of npm. But because **HTML/CSS/JS — the three-way separation of structure, style, and logic — is the only architecture that makes sense at scale.** Developers accepted the performance tax of an entire browser engine just to get access to this separation.

The industry didn't need convincing that HTML/CSS is the right model. The industry needed someone to deliver that model **without the browser**.

### The missing 4th generation

Every major 3rd-generation language has a 4th-generation successor.

| 3rd gen | 4th gen |
|---------|---------|
| Java | Kotlin |
| C | Rust |
| JavaScript | TypeScript |
| **C#** | **???** |

C# stands alone — the only major language without a modern successor. F# doesn't count. Nobody uses it.

**Grin fills that gap.**

> `.grin` / `.grs` — native desktop structure and style, the way the web does it  
> `.gl` — the 4th-generation successor to C#  
> The architecture of Electron, the performance of native.  
> You never have to drag a browser into your app again.

### How `.gl` thinks

**Not for C# veterans.** Python didn't win by persuading Java developers. It won by being the obvious choice for the next generation. Grin takes the same bet: target the developers who will build desktop apps in 2030, not the ones defending XAML in 2025.

Brace syntax. Not Rust-level strict, but concise and unambiguous — because in the AI era, both humans and machines are paying by the token. Every character must carry its weight. Every line must declare its context without ceremony.

- **Kotlin was the template**: Java interop was non-negotiable. C# interop is non-negotiable for Grin.
- **NuGet is the ecosystem**: Every existing .NET assembly, every NuGet package — callable from `.gl` day one.
- **Casing is semantics**: `variableName` is a value. `TypeName` is a type. First-letter uppercase on a variable is a compile error. No linter required.
- **Phase 1: transpile to C#** → compile via Roslyn → run on .NET. Same path Kotlin took with JVM.
- **Phase 2: LLVM native** — only after the language matures. Kotlin/Native took years. So will this.

No Python interop (yet). We want it. We're not stupid enough to promise it.

---

## Example

```
// structure (.grin file)
window "My App" size:800*600 {
    layout grid[rows: auto 1fr auto, cols: 200px 1fr] {
        
        label "Name:" [row:0, col:0, align:center]
        
        input bind:Name [row:0, col:1]
        
        button "Confirm" [row:2, col:1, align:right] {
            click => submit()
        }
    }
}

// style (.grs file — completely separate)
style {
    .input {
        padding: 8 12
        border: 1 solid #ccc
        border-radius: 4
    }
    
    button {
        padding: 8 16
        background: #4a90d9
        color: white
    }
}
```

---

## File Extensions

| Extension | Role | Web analogy |
|-----------|------|-------------|
| `.grin` (4) | Structure | `.html` |
| `.grs` (3) | Style | `.css` |
| `.gl` (2) | Logic | `.js` |

4·3·2 — deliberate pattern.

---

## Core Principles

| Principle | Description |
|-----------|-------------|
| **Real Separation** | Structure, style, and logic each live in their own files |
| **First-Class Layout** | `grid`, `flex`, `stack` are language keywords, not hidden in XML attributes |
| **Type-Safe Bindings** | Broken bindings caught at compile time, not runtime |
| **Native Performance** | Target is LLVM native compilation |

---

## Inspiration

- **HTML/CSS**: The gold standard of structure/style separation — desktop deserves this
- **Electron**: Proved the architecture was right, but paid for it with a browser engine
- **Kotlin**: Java's 4th generation. What Grin intends to be for C#
- **Rust**: The philosophy of a type system that trusts no one

