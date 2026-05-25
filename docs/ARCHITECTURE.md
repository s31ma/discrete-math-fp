# BiScheme Window Architecture

This document is the technical reference for the two types of interactive Scheme windows used in this project. Read this before modifying the IDE code.

---

## Overview

The application uses [BiwaScheme 0.8.3](https://www.biwascheme.org/), a complete R5RS Scheme interpreter written in JavaScript. It is bundled inline in the HTML file — no external script tags.

There are **two window types**:

| Type | Class | Engine function | Count per module |
|------|-------|----------------|-----------------|
| Inline IDE | `.inline-ide` | `runInlineIDE(id)` | 0–4 |
| Module Main IDE | `.ide-wrap` (inside `.widget-container`) | `createModuleIDE(opts)` | 1 |

Both share a **single BiwaScheme interpreter instance** managed by the shared interpreter module.

---

## The Shared Interpreter

### Why one instance?

Creating a `BiwaScheme.Interpreter` is expensive — it initializes the entire standard library. Sharing one instance means:
- Boot happens once (~600ms after page load)
- All subsequent evaluations are instant
- Definitions accumulate across windows (intentional)

### Global state

```javascript
var _sharedInterp = null;   // the BiwaScheme.Interpreter instance
var _sharedReady  = false;  // true once the interpreter has booted
var _sharedQueue  = [];     // [{preload: string, callback: fn}] — module IDEs waiting for boot
```

### Boot sequence

```javascript
function _initSharedInterp() {
  // Guard: BiwaScheme might not be parsed into the JS engine yet
  if (typeof BiwaScheme === 'undefined' || typeof BiwaScheme.Interpreter === 'undefined') {
    setTimeout(_initSharedInterp, 200);
    return;
  }
  try {
    _sharedInterp = new BiwaScheme.Interpreter(function(e) {
      console.error('BiwaScheme error:', e.message || e);
    });
  } catch(e) {
    setTimeout(_initSharedInterp, 300);
    return;
  }
  _sharedReady = true;
  // Flush any module IDEs that were created before the interpreter was ready
  _sharedQueue.forEach(function(item) {
    try { _sharedInterp.evaluate(item.preload, function(){}); } catch(e){}
    item.callback();
  });
  _sharedQueue = [];
}
setTimeout(_initSharedInterp, 600);
```

The 600ms delay gives the rest of the page (jQuery, BiwaScheme library code) time to finish executing before the interpreter is constructed.

---

## Inline IDE Windows

### HTML structure

```html
<div class="inline-ide" id="mN-ideK">
  <div class="ide-header">
    <span>TITLE — description</span>
    <div class="dots">
      <div class="dot r"></div>
      <div class="dot y"></div>
      <div class="dot g"></div>
    </div>
  </div>

  <textarea id="mN-ideK-editor" rows="12" spellcheck="false" autocomplete="off">
    ; initial code here
  </textarea>

  <div class="ide-out" id="mN-ideK-out">
    <span class="oi">BiwaScheme loading…</span>
  </div>

  <div class="ide-bar">
    <button class="btn btn-run" onclick="runInlineIDE('mN-ideK')">▶ Run</button>
    <button class="snippet-btn" onclick="document.getElementById('mN-ideK-editor').value='...'">
      label
    </button>
  </div>
</div>
```

**ID convention:** `mN-ideK` where `N` is the module number and `K` is a counter (1, 2, 3…).  
**Corresponding element IDs:** `mN-ideK-editor` (textarea), `mN-ideK-out` (output div).

### The `runInlineIDE(ideId)` function

Called by the Run button's `onclick`, and by Ctrl+Enter inside the textarea.

```
runInlineIDE('mN-ideK')
  │
  ├─ Find editor (#mN-ideK-editor) and output (#mN-ideK-out)
  │
  ├─ Attach keyboard handlers (once only, guarded by editor._handlersAttached)
  │     Tab     → insert 2 spaces
  │     Enter   → auto-indent (match current line; extra indent after '(')
  │     Ctrl+Enter → call runInlineIDE again
  │
  ├─ If !_inlineReady:
  │     show "Loading interpreter…" in output
  │     queue this run in _inlineQueue (deduplicated by ideId)
  │     return
  │
  ├─ Clear output div
  ├─ Split source into top-level expressions via _splitTopLevel(src)
  └─ Evaluate each expression sequentially:
        on success: append green .ov span with BiwaScheme.to_write(result)
        on error:   append red .oe span with error message
        (undefined / BiwaScheme.undef results are silently skipped)
```

### Inline interpreter readiness

```javascript
var _inlineInterpreter = null;
var _inlineReady = false;
var _inlineQueue = [];

function _checkInlineReady() {
  if (_sharedReady) {
    _inlineInterpreter = _sharedInterp;  // same instance
    _inlineReady = true;
    _inlineQueue.forEach(fn => fn());
    _inlineQueue = [];
  } else {
    setTimeout(_checkInlineReady, 200);
  }
}
setTimeout(_checkInlineReady, 700);  // slightly later than _initSharedInterp
```

`_inlineReady` becomes true approximately 700–800ms after page load (when `_sharedReady` is first observed to be true by the polling loop).

---

## Module Main IDE

### HTML structure

```html
<div class="widget-container">
  <div class="widget-header">
    <span>title</span>
    <div class="dots">...</div>
  </div>

  <div class="ide-wrap">

    <!-- Toolbar: preset code snippets -->
    <div class="ide-toolbar">
      <span class="ide-toolbar-label">Examples →</span>
      <button class="snippet-btn"
              data-target="mN-ide-main-editor"
              data-snippet="(some-expression)">
        label
      </button>
      <!-- Run + Clear buttons -->
      <div style="margin-left:auto; display:flex; gap:8px;">
        <button class="btn btn-run" id="mN-ide-run-btn" title="Run (Ctrl+Enter)">▶ Run</button>
        <button class="btn" id="mN-ide-clear-btn" title="Clear output">Clear</button>
      </div>
    </div>

    <!-- Split pane -->
    <div class="ide-panes">
      <div class="ide-editor-pane">
        <div class="ide-pane-label"><span>editor.scm</span></div>
        <textarea class="ide-editor" id="mN-ide-main-editor"
                  spellcheck="false" autocomplete="off"
                  autocorrect="off" autocapitalize="off"></textarea>
      </div>
      <div class="ide-output-pane">
        <div class="ide-pane-label">
          <span>output</span>
          <span id="mN-ide-run-count" style="font-size:10px;color:var(--border-bright)"></span>
        </div>
        <div class="ide-output" id="mN-ide-main-output">
          <div class="out-line out-info">BiwaScheme R5RS loading…</div>
        </div>
      </div>
    </div>

    <!-- Status bar -->
    <div class="ide-statusbar">
      <span id="mN-ide-status" class="status-busy">● loading</span>
      <span id="mN-ide-cursor-pos">Ln 1, Col 1</span>
    </div>

    <!-- Insert bar: template fragments inserted at cursor -->
    <div class="ide-snippets" style="border-top:1px solid var(--border)">
      <button class="snippet-btn" data-insert="define">(define f …)</button>
      <button class="snippet-btn" data-insert="if">(if … … …)</button>
      <button class="snippet-btn" data-insert="cond">(cond …)</button>
      <button class="snippet-btn" data-insert="let">(let ((x …)) …)</button>
      <button class="snippet-btn" data-insert="lambda">(lambda (x) …)</button>
      <button class="snippet-btn" data-insert="map">(map f lst)</button>
      <button class="snippet-btn" data-insert="append">(append l1 l2)</button>
    </div>

  </div><!-- .ide-wrap -->
</div><!-- .widget-container -->
```

### The `createModuleIDE(opts)` factory

Called once per module, at the bottom of the `<script>` block.

```javascript
createModuleIDE({
  prefix:       'mN',               // used to scope querySelector
  editorId:     'mN-ide-main-editor',
  outputId:     'mN-ide-main-output',
  statusId:     'mN-ide-status',
  cursorId:     'mN-ide-cursor-pos',
  runBtnId:     'mN-ide-run-btn',
  clearBtnId:   'mN-ide-clear-btn',
  runCountId:   'mN-ide-run-count',
  snippetTarget: 'mN-ide-main-editor',  // data-target value on toolbar buttons
  preload: `
; Scheme definitions to evaluate before the IDE becomes interactive.
; These are available immediately when the user opens the editor.
(define (my-function ...) ...)
`
});
```

**`opts.preload`** is the most important field. It is evaluated by the shared interpreter during `initIDE()`. Any functions defined here are immediately available in the editor without the user needing to define them.

### Initialization flow

```
createModuleIDE(opts)
  │
  ├─ Grab all DOM elements by ID
  ├─ Wire up Run button → runCode()
  ├─ Wire up Clear button → clear output
  ├─ Wire up keyboard: Tab, Enter (auto-indent), Ctrl+Enter (run)
  ├─ Wire up data-target snippet buttons (scoped to this module's container)
  ├─ Wire up data-insert buttons (insert template at cursor)
  ├─ Wire up keyup/click on editor → updateCursor()
  └─ Call initIDE()
       │
       ├─ If !_sharedReady:
       │     push {preload, callback: initIDE} onto _sharedQueue
       │     return (will be called again when interpreter boots)
       │
       ├─ interp = _sharedInterp
       ├─ interp.evaluate(opts.preload, () => {})   ← load module functions
       ├─ Clear output, print "BiwaScheme R5RS ready."
       ├─ Set status to "ready" (green)
       └─ Enable Run button
```

### `runCode()` — execution flow

```
runCode()
  ├─ Get src from editor, trim
  ├─ runNum++
  ├─ Disable Run button, set status "running"
  ├─ Append "─── run #N ───" separator to output
  ├─ _splitTopLevel(src) → array of expression strings
  └─ evalNext() loop:
       ├─ Evaluate expressions[idx] via interp.evaluate(expr, callback)
       ├─ On success: if result !== undefined && !== BiwaScheme.undef,
       │              append green out-val div with to_write(result)
       ├─ On error: append red out-err div, set hasError=true
       └─ After last expression:
            re-enable Run button
            set status "ready" or "error"
            update run counter
            scroll output to bottom
```

---

## `_splitTopLevel(src)` — Expression Parser

Inline and module IDEs both use this function to split multi-expression input into individual evaluable units.

```javascript
function _splitTopLevel(src) {
  const exprs = [];
  let depth = 0, start = null, inStr = false, escape = false;

  for (let i = 0; i < src.length; i++) {
    const ch = src[i];

    // String escape handling
    if (escape) { escape = false; continue; }
    if (ch === '\\' && inStr) { escape = true; continue; }
    if (ch === '"') { inStr = !inStr; }
    if (inStr) continue;

    // Line comments
    if (ch === ';') { while (i < src.length && src[i] !== '\n') i++; continue; }

    // Parenthesized expression
    if (ch === '(' || ch === '[') {
      if (depth === 0) start = i;
      depth++;
    } else if (ch === ')' || ch === ']') {
      depth--;
      if (depth === 0 && start !== null) {
        exprs.push(src.slice(start, i+1).trim());
        start = null;
      }
    }

    // Atom (non-paren top-level token, e.g. a number or bare symbol)
    else if (depth === 0 && start === null && ch.trim() !== '') {
      let end = i;
      while (end < src.length && src[end].trim() !== '' && src[end] !== '(' && src[end] !== ')') end++;
      exprs.push(src.slice(i, end).trim());
      i = end - 1;
    }
  }
  return exprs.filter(Boolean);
}
```

This correctly handles:
- Multiple top-level expressions in one editor
- Nested parentheses of any depth
- String literals containing parens or semicolons
- Line comments
- Square bracket aliases for parens
- Bare atoms (symbols, numbers) at top level

---

## CSS Variables Reference

All IDE colors use CSS custom properties defined in the `<style>` block:

```css
:root {
  --surface:       #111;          /* IDE header/toolbar/statusbar background */
  --border:        #222;          /* default border */
  --border-bright: #333;          /* brighter border, run count text */
  --accent:        #c8b26a;       /* editor text, cursor */
  --accent-dim:    #8a7a4a;       /* dimmer accent */
  --text-dim:      #666;          /* output info text */
  --green:         #5a9e6f;       /* success / ready status */
  --red:           #c0564a;       /* error */
  --mono:          'JetBrains Mono', monospace;
}
```

The main editor textarea uses:
- Background: `#080808` (nearly black)
- Text: `var(--accent)` (warm gold)
- Caret: `var(--accent)`

The output panel uses:
- Background: `#060606`
- Text: `var(--text-dim)` (dim)
- Values: `var(--green)` via `.ov` / `.out-val`
- Errors: `var(--red)` via `.oe` / `.out-err`

---

## Known Behaviors and Edge Cases

**Definitions persist across page sections.** Because all windows share one interpreter, `(define x 5)` in Module 3's inline IDE means `x` is defined for Module 5's main IDE too. This is intentional but can confuse students who expect isolation.

**Reloading the page resets everything.** The interpreter instance is in memory only. Nothing is saved to localStorage or any backend.

**Preload runs on every page load, not once.** When `createModuleIDE` fires, it calls `interp.evaluate(preload, ...)`. Since the shared interpreter accumulates state, re-running the preload on top of prior state is safe — `define` simply rebinds names. It does not cause errors.

**`BiwaScheme.undef` vs `undefined`.** BiwaScheme returns its own `undef` sentinel for expressions that have no meaningful return value (like `(define ...)` or `(display ...)`). Both `BiwaScheme.undef` and JavaScript `undefined` are silently filtered from output — only actual values are printed.

**Error recovery.** Each expression in a run is try-catch-wrapped independently. An error in expression 2 of 5 will print the error and continue to evaluate expressions 3, 4, and 5.
