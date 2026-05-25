# Discrete Math Through Functional Programming

> **A self-contained, interactive textbook for learning discrete mathematics via Scheme — no installation, no server, no build step. Just open the HTML file in a browser.**

---

## What This Is

This project is a single-file interactive learning environment that teaches discrete mathematics through functional programming in Scheme. Each module combines:

- **Rigorous mathematical exposition** — formal proofs, recurrences, induction arguments
- **Animated visualizations** — Towers of Hanoi solver, recursion trees, N-Queens board
- **Live, in-browser Scheme interpreters** — write and run real Scheme code without leaving the page

The entire application — BiwaScheme R5RS interpreter, jQuery, all CSS, all JavaScript, all content — is bundled into a single `.html` file. Nothing to install. Nothing to configure.

---

## Quick Start

```bash
git clone https://github.com/YOUR_USERNAME/discrete-math-fp.git
cd discrete-math-fp
open discrete_math_modules.html   # macOS
# or: xdg-open discrete_math_modules.html  (Linux)
# or: just double-click the file in your file explorer
```

That's it. The file runs entirely in the browser. No local server needed.

---

## Modules

| Module | Topic | Key Concepts |
|--------|-------|--------------|
| **03** | Induction & Towers of Hanoi | Structural induction, recurrence relations, recursive proof |
| **04** | Balanced Parentheses & Catalan Numbers | Accumulator pattern, invariants, counting arguments |
| **05** | N-Queens & Backtracking | Constraint satisfaction, search trees, `attacks?` / `safe?` |

Each module follows the same pedagogical arc:

1. **The Puzzle** — concrete problem statement, no abstraction yet
2. **You're Not Lost** — acknowledges discomfort, sets expectations
3. **Break It Down** — interactive visualization of the core idea
4. **Find the Pattern** — closed-form or recurrence observation
5. **The Structured Argument** — informal proof sketch
6. **The Formal Proof** — full induction proof with pre/postconditions
7. **The Code** — step-by-step derivation from spec to working Scheme
8. **Full Scheme IDE** — open sandbox pre-loaded with module functions
9. **Exercises** — pencil-and-paper problems
10. **Bonus Challenges** — harder extensions for the curious

---

## The BiScheme Windows

The heart of this project is its **live Scheme execution environment**. Every module contains multiple interactive Scheme windows powered by [BiwaScheme 0.8.3](https://www.biwascheme.org/) — a full R5RS Scheme interpreter compiled to JavaScript.

There are two distinct types of BiScheme window, each serving a different purpose.

---

### Type 1 — Inline IDE Windows

These are compact, self-contained scratch pads embedded directly within the prose of a section. You encounter them mid-lesson, immediately after a concept is introduced.

**Anatomy:**

```
┌─────────────────────────────────────────────┐
│  ● ● ●   SCHEME PLAYGROUND — section name  │  ← header bar
├─────────────────────────────────────────────┤
│  (define balance-helper                     │
│    (lambda (chars count)                    │  ← editable textarea
│      ...))                                  │
├─────────────────────────────────────────────┤
│  BiwaScheme loading…                        │  ← output panel
├─────────────────────────────────────────────┤
│  ▶ Run  [try count=1]  [test balanced?]     │  ← toolbar
└─────────────────────────────────────────────┘
```

**How they work:**

- Each inline window shares the **same BiwaScheme interpreter instance** as all other windows on the page. This means definitions you evaluate in one window persist and are available in others.
- The output panel is capped at `140px` with scroll — it won't disrupt page flow.
- Snippet buttons on the toolbar **replace the editor contents** with a preset code example. This lets the lesson guide you through progressively more complex versions of a function.
- Keyboard shortcuts work: **Tab** inserts 2 spaces, **Enter** auto-indents (with extra indent after an opening `(`), **Ctrl+Enter** / **Cmd+Enter** runs the code.

**Initialization sequence:**

```
page load
  └─ setTimeout(_initSharedInterp, 600ms)
       └─ new BiwaScheme.Interpreter(errorHandler)
            └─ _sharedReady = true
                 └─ flush _sharedQueue (any IDEs that initialized before interpreter was ready)
                      └─ setTimeout(_checkInlineReady, 700ms)
                           └─ _inlineInterpreter = _sharedInterp
                                └─ _inlineReady = true  →  inline IDEs now accept runs
```

All inline windows wait for `_inlineReady` before executing. Runs attempted before the interpreter is ready are queued and replayed automatically.

---

### Type 2 — Module Main IDE (Full Split-Pane IDE)

Each module ends with a full-featured split-pane IDE. This is the primary coding environment for that module.

**Anatomy:**

```
┌──────────────────────────────────────────────────────────┐
│  ● ● ●   Scheme R5RS IDE — Live In-Browser               │
│  Examples → [hanoi 3] [count moves] [count 0–6] ...      │
├──────────────────────────────┬───────────────────────────┤
│  editor.scm                  │  output                   │
│                              │                           │
│  (hanoi 3 'A 'C 'B)          │  BiwaScheme R5RS ready.   │
│                              │  Press ► Run or Ctrl+Enter│
│  (your code here)            │                           │
│                              │  ─── run #1 ───           │
│                              │  ((move 1 from A to C)    │
│                              │   (move 2 from A to B)    │
│                              │   ...)                    │
├──────────────────────────────┴───────────────────────────┤
│  ● ready    Ln 1, Col 1                   runs: 3        │
├──────────────────────────────────────────────────────────┤
│  [(define f …)]  [(if … … …)]  [(cond …)]  [(let …)]    │
└──────────────────────────────────────────────────────────┘
```

**Features:**

- **520px fixed height** — editor pane on the left, output pane on the right (42% width)
- **Status bar** — live color-coded status (`● ready` in green, `● running` in amber, `● error` in red), cursor position (Ln/Col), and a run counter
- **Snippet toolbar** — loads preset code into the editor. Each module has its own set of examples tailored to its content
- **Insert bar** — inserts Scheme template fragments (`(define f …)`, `(lambda …)`, `(cond …)`, etc.) at the cursor position
- **Run counter** — tracks how many times you've executed code in the session
- **Pre-loaded definitions** — when the IDE initializes, it evaluates a `preload` block so module-specific functions are available immediately without you having to define them

**Pre-loaded functions by module:**

| Module | Pre-loaded | What it does |
|--------|-----------|--------------|
| **m3** | `hanoi`, `count-hanoi` | Towers of Hanoi solver; returns a list of move triples |
| **m4** | `balance-helper`, `balanced?` | Accumulator-based parenthesis checker |
| **m5** | `attacks?`, `safe?`, `solve-helper`, `n-queens` | Full N-Queens backtracking solver |

This means you can open the Module 3 IDE and immediately type `(hanoi 3 'A 'C 'B)` without defining anything — the function is already there.

---

### The Shared Interpreter Architecture

All windows — both inline and module IDEs — share **one BiwaScheme interpreter instance**. This is a deliberate design decision with important implications:

**Benefits:**
- Definitions accumulate across windows within a module's session. Define a helper in an inline window, use it in the main IDE.
- No cold-start cost on the second, third, fourth window — only one interpreter ever boots.
- Memory footprint is kept to one instance regardless of how many IDE windows appear on screen.

**Implications for users:**
- Re-defining a function in any window updates it globally for the session. This is intentional — it lets you experiment with variations.
- If you define a function with the same name as a preloaded one (e.g., `hanoi`), your version replaces it. To restore the original, reload the page.
- Errors in one window do not crash other windows — each evaluation is independently try/catch-wrapped.

**Shared interpreter boot sequence (detailed):**

```javascript
// _initSharedInterp fires ~600ms after page load
_sharedInterp = new BiwaScheme.Interpreter(errorHandler);
_sharedReady = true;

// Any module IDE that called createModuleIDE() before the interpreter
// was ready queued itself in _sharedQueue. Those are now flushed:
_sharedQueue.forEach(item => {
  _sharedInterp.evaluate(item.preload, () => {});  // load module functions
  item.callback();                                   // initialize the IDE UI
});
```

---

### Multi-Expression Evaluation

Both window types use a shared `_splitTopLevel(src)` function to parse the editor contents into individual top-level expressions before evaluating them. This means you can write multiple expressions in the editor:

```scheme
(define (square x) (* x x))
(square 5)
(square 12)
```

Each expression is evaluated sequentially. Results are printed as separate output lines. Errors on one expression do not stop subsequent ones from running.

---

### Output Formatting

Output lines are color-coded:

| Class | Color | Meaning |
|-------|-------|---------|
| `out-val` / `.ov` | Green | Return value of an expression |
| `out-err` / `.oe` | Red | Runtime or parse error |
| `out-info` / `.oi` | Dim italic | System messages (loading, ready, cleared) |
| `out-sep` | Dim | Run separator (`─── run #N ───`) |

Values are formatted using `BiwaScheme.to_write(result)`, which gives Scheme-style output (e.g., `'(a b c)` not `[object Object]`).

---

## Project Structure

```
discrete-math-fp/
├── README.md                    ← you are here
├── discrete_math_modules.html   ← the entire application (single file)
└── docs/
    ├── ARCHITECTURE.md          ← BiScheme window internals
    ├── ADDING_MODULES.md        ← how to add a new module
    └── SCHEME_REFERENCE.md      ← quick reference for R5RS Scheme
```

---

## How to Add a New Module

See [`docs/ADDING_MODULES.md`](docs/ADDING_MODULES.md) for the full walkthrough. The short version:

1. Add a nav link in the `<nav class="top-nav">` element
2. Add a `<div class="module-section" id="module-mN">` container
3. Write your sections using the existing section structure
4. Add inline IDE blocks using `<div class="inline-ide" id="mN-ideK">` 
5. Add the module main IDE with `createModuleIDE({...})` at the bottom of the script
6. Provide a `preload` string with the module's key function definitions

---

## Browser Compatibility

Tested and working in:
- Chrome / Chromium 90+
- Firefox 88+
- Safari 14+
- Edge 90+

Requires JavaScript enabled. No network requests at runtime (all assets are inlined, fonts load from Google Fonts on first open).

---

## Dependencies (all bundled)

| Library | Version | License |
|---------|---------|---------|
| [BiwaScheme](https://www.biwascheme.org/) | 0.8.3 | MIT |
| [jQuery](https://jquery.com/) | 3.6.0 | MIT |

Both are inlined directly in the HTML file. No CDN dependency at runtime.

---

## License

Source code: MIT  
Content (prose, proofs, exercises): © the author — all rights reserved unless otherwise stated.

---

## Contributing

Issues and pull requests welcome. If you're adding a module, please follow the pattern established by Modules 03–05: puzzle → proof → code → IDE. The BiScheme window architecture is documented in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).
