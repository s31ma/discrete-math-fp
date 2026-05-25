# Scheme R5RS Quick Reference

This file documents the Scheme dialect used in the in-browser IDEs (BiwaScheme 0.8.3, R5RS-compatible). It covers the subset most relevant to the discrete math modules.

---

## Basic Expressions

```scheme
; Arithmetic
(+ 1 2)          ; → 3
(- 10 3)         ; → 7
(* 4 5)          ; → 20
(/ 10 2)         ; → 5
(expt 2 10)      ; → 1024
(modulo 17 5)    ; → 2
(remainder 17 5) ; → 2

; Comparison
(= 3 3)          ; → #t
(< 2 5)          ; → #t
(> 5 2)          ; → #t
(<= 3 3)         ; → #t
(>= 4 3)         ; → #t

; Boolean
(and #t #f)      ; → #f
(or  #f #t)      ; → #t
(not #f)         ; → #t
```

---

## Defining Functions

```scheme
; Short form (syntactic sugar — preferred in this course)
(define (square x)
  (* x x))

; Long form (equivalent)
(define square
  (lambda (x)
    (* x x)))

; Multiple parameters
(define (add a b)
  (+ a b))

; Calling
(square 5)       ; → 25
(add 3 4)        ; → 7
```

---

## Conditionals

```scheme
; if — one condition
(if (= x 0)
    'zero          ; then
    'nonzero)      ; else

; cond — multiple branches (like if-else-if)
(cond
  ((< x 0) 'negative)
  ((= x 0) 'zero)
  ((> x 0) 'positive)
  (else     'unknown))   ; else is the fallthrough
```

---

## Lists

```scheme
; Construction
'(1 2 3)                    ; literal list
(list 1 2 3)                ; same
(cons 1 '(2 3))             ; → (1 2 3)  — prepend
(append '(1 2) '(3 4))      ; → (1 2 3 4) — concatenate

; Access
(car '(1 2 3))              ; → 1  — first element
(cdr '(1 2 3))              ; → (2 3)  — rest of list
(cadr '(1 2 3))             ; → 2  — second element (car of cdr)

; Predicates
(null? '())                 ; → #t  — empty list?
(null? '(1))                ; → #f
(pair? '(1 2))              ; → #t  — non-empty list?
(list? '(1 2 3))            ; → #t

; Length
(length '(a b c))           ; → 3

; Higher-order
(map (lambda (x) (* x x)) '(1 2 3 4))   ; → (1 4 9 16)
(filter odd? '(1 2 3 4 5))              ; → (1 3 5)
(apply + '(1 2 3))                       ; → 6
(apply append '((1 2) (3 4) (5)))        ; → (1 2 3 4 5)
```

---

## Recursion Patterns

### Linear recursion (build result on the way back up)
```scheme
(define (length lst)
  (if (null? lst)
      0
      (+ 1 (length (cdr lst)))))
```

### Tail recursion with accumulator
```scheme
(define (length-iter lst)
  (define (helper lst acc)
    (if (null? lst)
        acc
        (helper (cdr lst) (+ acc 1))))
  (helper lst 0))
```

### Tree recursion (two recursive calls)
```scheme
(define (fib n)
  (if (< n 2)
      n
      (+ (fib (- n 1))
         (fib (- n 2)))))
```

---

## Symbols and Quoting

```scheme
; Symbols are like labels/enums — they evaluate to themselves when quoted
'foo                ; → foo
(eq? 'foo 'foo)    ; → #t
(eq? 'foo 'bar)    ; → #f

; Quote prevents evaluation
'(1 2 3)           ; → (1 2 3) as data, not evaluated
(quote (1 2 3))    ; same

; Common pattern in these modules: lists of symbols
'(op cl op cl)     ; used in the balanced parentheses module
'(A B C)           ; peg names in the Hanoi module
```

---

## Let Bindings

```scheme
; let — bind local variables
(let ((x 5)
      (y 3))
  (+ x y))          ; → 8

; let* — bindings can reference previous ones
(let* ((x 5)
       (y (* x 2)))
  y)                ; → 10

; Named let — local recursion (loop pattern)
(let loop ((i 0) (acc '()))
  (if (= i 5)
      acc
      (loop (+ i 1) (cons i acc))))   ; → (4 3 2 1 0)
```

---

## `iota` — generate a range

BiwaScheme includes `iota` from SRFI-1:

```scheme
(iota 5)          ; → (0 1 2 3 4)
(iota 5 1)        ; → (1 2 3 4 5)   — start at 1
(iota 5 1 2)      ; → (1 3 5 7 9)   — start at 1, step 2
```

Used in Module 5 (`n-queens`) to generate column indices: `(iota n 1)` gives `(1 2 ... n)`.

---

## Common Pitfalls

**`'()` vs `(list)`** — both produce an empty list, but `'()` is idiomatic.

**`eq?` vs `equal?`** — `eq?` tests identity (works for symbols and booleans); `equal?` tests structural equality (works for lists and strings). Use `eq?` for symbols, `equal?` for lists.

**`define` inside a function** — creates a local binding, not a global one:
```scheme
(define (outer x)
  (define (inner y) (+ x y))  ; inner is local to outer
  (inner 10))
```

**Tail position matters** — in `(if test then else)`, both `then` and `else` are in tail position. In `(+ 1 (f x))`, the call to `f` is NOT in tail position. Tail calls don't consume stack.

**`display` vs return value** — `(display x)` prints `x` as a side effect and returns `undef` (which is suppressed in the IDE output). To see a value, just write the expression without `display`: `x` or `(my-function args)`.
