# Grin

> A declarative language for building desktop UIs.
> Born out of anger at XAML.

---

## Why

XAML is not a language meant to be written by humans.  
HTML/CSS/JS proved that separation of concerns is possible, but they're trapped inside the browser.  
Electron devours memory. Native frameworks lack expressiveness.

**Grin says:**
> Structure, style, and logic must be *actually* separated.  
> Layout is a first-class citizen of the language.  
> The type system must guarantee data binding correctness.  
> Native performance is non-negotiable.

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

## Core Principles

| Principle | Description |
|-----------|-------------|
| **Real Separation** | Structure, style, and logic each live in their own files |
| **First-Class Layout** | `grid`, `flex`, `stack` are language keywords, not hidden in XML attributes |
| **Type-Safe Bindings** | Broken bindings caught at compile time, not runtime |
| **Native Performance** | Target is LLVM native compilation |

---

## File Extensions

| Extension | Role | Web analogy |
|-----------|------|-------------|
| `.grin` (4) | Structure | `.html` |
| `.grs` (3) | Style | `.css` |
| `.gl` (2) | Logic | `.js` |

4·3·2 — deliberate pattern.

---

---

## Inspiration

- **XAML**: Being built to replace it
- **HTML/CSS**: The gold standard of separation of concerns
- **Flutter/Dart**: Modern approach to cross-platform UI languages
- **Rust**: The philosophy of a type system that trusts no one
