# Grin

> C#의 4세대. Kotlin이 Java에 한 일을, Grin은 C#에 한다.
> HTML/CSS는 올바른 구조다. 데스크톱에도 그게 필요했다.

---

## Why

### Electron이 증명한 것

Electron이 느리고 무거운데도 데스크톱 앱 생태계를 점령한 이유는 하나뿐이다:

**HTML/CSS/JS — 구조·스타일·로직의 3단 분리가 올바른 아키텍처였기 때문.**

사람들은 성능을 깎아먹으면서까지 브라우저를 통째로 끌고 왔다. 정답을 알지만 정답을 네이티브로 가져올 방법이 없었으니까.

### C#의 빈자리

3세대 메이저 언어들은 모두 4세대 후계를 얻었다.

| 3세대 | 4세대 |
|--------|--------|
| Java | Kotlin |
| C | Rust |
| JavaScript | TypeScript |
| **C#** | **???** |

C#만 유일하게 4세대가 없다. F#은 아무도 안 쓴다.

**Grin says:**
> `.grin` / `.grs` — 네이티브 데스크톱을 위한 HTML/CSS  
> `.gl` — C#을 계승하는 4세대 언어  
> 구조는 Electron의 정답에서 가져오고, 성능은 네이티브로.  
> 이제 더 이상 브라우저를 끌고 오지 않아도 된다.

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

- **HTML/CSS**: 구조와 스타일의 분리 — 데스크톱에도 그게 필요했다
- **Electron**: 올바른 아키텍처의 승리를 증명했지만, 너무 무거웠다
- **Kotlin**: Java의 4세대. Grin이 C#에 대해 해내야 할 일의 선례
- **Rust**: 4세대 언어의 타입 시스템 철학 — 아무도 믿지 않는다
