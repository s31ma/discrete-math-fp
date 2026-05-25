# Adding a New Module

This guide walks through adding a Module 06 (or any new module) to the project. All edits happen inside the single `discrete_math_modules.html` file.

---

## Step 1 — Add a Nav Link

Find the `<nav class="top-nav">` element near the top of the `<body>`. Add your module:

```html
<nav class="top-nav">
  <span class="nav-label">Discrete Math</span>
  <a href="#" class="active" onclick="showModule('m3',this);return false;" id="nav-m3">03 — Induction &amp; Towers of Hanoi</a>
  <a href="#" onclick="showModule('m4',this);return false;" id="nav-m4">04 — Balanced Parentheses &amp; Catalan Numbers</a>
  <a href="#" onclick="showModule('m5',this);return false;" id="nav-m5">05 — N-Queens &amp; Backtracking</a>
  <!-- ADD THIS: -->
  <a href="#" onclick="showModule('m6',this);return false;" id="nav-m6">06 — Your Topic Title</a>
</nav>
```

---

## Step 2 — Add the Module Container

Find the last `</div><!-- end module-mN -->` block. After it, add:

```html
<div class="module-section" id="module-m6">
<div class="page-wrap">

  <div class="breadcrumb">
    Discrete Math Through FP <span>›</span> Module 06 <span>›</span> Your Topic Title
  </div>
  <div class="module-label">Module 06</div>
  <h1>Your Topic Title —<br><em>Subtitle or Tagline</em></h1>

  <div class="toc">
    <div class="toc-title">Contents</div>
    <a href="#m6-puzzle" class="toc-link">01 — The Puzzle</a>
    <a href="#m6-proof" class="toc-link">02 — The Proof</a>
    <a href="#m6-code" class="toc-link">03 — The Code</a>
    <a href="#m6-ide-main" class="toc-link">04 — Full Scheme IDE</a>
  </div>

  <!-- === SECTIONS GO HERE === -->

</div><!-- .page-wrap -->
</div><!-- #module-m6 -->
```

---

## Step 3 — Write Your Sections

Each section follows this template:

```html
<div class="section" id="m6-puzzle">
  <h2><span class="section-num">01 —</span> The Puzzle</h2>
  <p>Your prose here.</p>

  <!-- Optional callout -->
  <div class="callout hint">
    <span class="label">💡 Hint</span>
    <p>Helpful note for students.</p>
  </div>

  <!-- Optional inline IDE (see Step 4) -->
</div>
```

**Callout types:** `hint` (blue-tinted), `warn` (amber-tinted). Use `hint` for tips, `warn` for common mistakes or important caveats.

**Code blocks:** Use `<pre>` with `<span class="kw">`, `<span class="fn">`, `<span class="sym">`, `<span class="num">`, `<span class="str">`, `<span class="cmt">` for syntax highlighting. These are styled by the existing CSS.

---

## Step 4 — Add Inline IDE Windows

Place these inside sections, after prose that introduces a concept.

```html
<div class="inline-ide" id="m6-ide1">
  <div class="ide-header">
    <span>SCHEME PLAYGROUND — section title</span>
    <div class="dots">
      <div class="dot r"></div>
      <div class="dot y"></div>
      <div class="dot g"></div>
    </div>
  </div>

  <textarea id="m6-ide1-editor" rows="10" spellcheck="false" autocomplete="off">
; Your initial code here
(define my-function
  (lambda (x)
    x))

(my-function 42)
  </textarea>

  <div class="ide-out" id="m6-ide1-out">
    <span class="oi">BiwaScheme loading…</span>
  </div>

  <div class="ide-bar">
    <button class="btn btn-run" onclick="runInlineIDE('m6-ide1')">▶ Run</button>
    <button class="snippet-btn"
            onclick="document.getElementById('m6-ide1-editor').value='...'">
      preset label
    </button>
  </div>
</div>
```

**ID convention:** `m6-ide1`, `m6-ide2`, `m6-ide3` — sequential per module.  
**Textarea rows:** 8–16 is typical. The textarea is resizable by the user.  
**Snippet buttons:** Each `onclick` replaces the textarea value with a preset string. Escape single quotes as `\'` inside the JS string. Use `\n` for newlines.

---

## Step 5 — Add the Module Main IDE

After your last section, add the full split-pane IDE. Copy this template and adjust the IDs and content:

```html
<div class="section" id="m6-ide-main">
  <h2><span class="section-num">04 —</span> Full Scheme IDE</h2>
  <p>Use the editor below to experiment with the functions defined in this module.</p>

  <div class="widget-container">
    <div class="widget-header">
      <span>Scheme R5RS IDE — Live In-Browser</span>
      <div class="dots">
        <div class="dot r"></div>
        <div class="dot y"></div>
        <div class="dot g"></div>
      </div>
    </div>

    <div class="ide-wrap">

      <div class="ide-toolbar">
        <span class="ide-toolbar-label">Examples →</span>
        <button class="snippet-btn"
                data-target="m6-ide-main-editor"
                data-snippet="(my-function 42)">
          example 1
        </button>
        <button class="snippet-btn"
                data-target="m6-ide-main-editor"
                data-snippet="; Write your Scheme code here&#10;">
          new file
        </button>
        <div style="margin-left:auto; display:flex; gap:8px;">
          <button class="btn btn-run" id="m6-ide-run-btn" title="Run (Ctrl+Enter)">▶ Run</button>
          <button class="btn" id="m6-ide-clear-btn" title="Clear output">Clear</button>
        </div>
      </div>

      <div class="ide-panes">
        <div class="ide-editor-pane">
          <div class="ide-pane-label"><span>editor.scm</span></div>
          <textarea class="ide-editor" id="m6-ide-main-editor"
                    spellcheck="false" autocomplete="off"
                    autocorrect="off" autocapitalize="off">
(my-function 42)
          </textarea>
        </div>
        <div class="ide-output-pane">
          <div class="ide-pane-label">
            <span>output</span>
            <span id="m6-ide-run-count" style="font-size:10px;color:var(--border-bright)"></span>
          </div>
          <div class="ide-output" id="m6-ide-main-output">
            <div class="out-line out-info">BiwaScheme R5RS loading…</div>
          </div>
        </div>
      </div>

      <div class="ide-statusbar">
        <span id="m6-ide-status" class="status-busy">● loading</span>
        <span id="m6-ide-cursor-pos">Ln 1, Col 1</span>
      </div>

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
</div><!-- #m6-ide-main -->
```

---

## Step 6 — Register the Module IDE in JavaScript

Near the **very bottom** of the `<script>` block (after the existing `createModuleIDE({...})` calls), add:

```javascript
// ═══ MODULE 6 JS ═══
createModuleIDE({
  prefix:      'm6',
  editorId:    'm6-ide-main-editor',
  outputId:    'm6-ide-main-output',
  statusId:    'm6-ide-status',
  cursorId:    'm6-ide-cursor-pos',
  runBtnId:    'm6-ide-run-btn',
  clearBtnId:  'm6-ide-clear-btn',
  runCountId:  'm6-ide-run-count',
  preload: `
(define (my-function x)
  x)

; Add all functions students should have available immediately.
; These are evaluated when the IDE boots — students can call them
; without defining them first.
`
});
```

**The `preload` field is critical.** It defines the "pre-loaded functions" for your module — whatever the student should be able to call immediately in the editor without having to write the definitions themselves.

---

## Step 7 — Add Module Navigation Logic

Find the `showModule` function in the script. It should already handle any module ID generically:

```javascript
function showModule(id, el) {
  document.querySelectorAll('.module-section').forEach(s => s.classList.remove('active'));
  document.querySelectorAll('.top-nav a').forEach(a => a.classList.remove('active'));
  const section = document.getElementById('module-' + id);
  if (section) section.classList.add('active');
  if (el) el.classList.add('active');
}
```

If this function already exists and works generically, no change is needed.

---

## Checklist

Before committing, verify:

- [ ] Nav link added with correct `id="nav-m6"` and `onclick="showModule('m6',this)"`
- [ ] Module container has `id="module-m6"` and `class="module-section"` (not `active` unless it should be the default)
- [ ] All inline IDE element IDs follow `m6-ideK`, `m6-ideK-editor`, `m6-ideK-out` convention
- [ ] Module main IDE element IDs follow `m6-ide-main-*` convention
- [ ] `createModuleIDE` called with all required fields
- [ ] `preload` string contains all functions students will need
- [ ] Tested in browser: module loads, inline IDEs run, main IDE runs, preloaded functions work
