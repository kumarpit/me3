---
title: "Definition of Programming Languages"
publishDate: "27 Nov 2023"
description: "This article summarizes most of the ideas I learned in CPSC 311: Definition of Programming Languages"
tags: ["Racket", "Interpreters"]
---

This article is my attempt of putting together most of the ideas I learned in CPSC 311: Definition of Programming Languages - the most influential course of my undergraduate degree so far. I will attempt to take you through specifying a programming language from the ground up as well as building an interpreter for it in Racket.

First, a little bit about CPSC 311 and why I enjoyed taking it. Working with Racket was amazing - throughout the course, we implemented interpreters for numerous languages, each building upon the other with more complex features. The template driven approach to development [(HtDP)](https://htdp.org), where the structure of our code followed directly from our data type definitions, made it a lot easier to reason about and debug programs. This coupled with the monadic design patterns we used to implement our interpreters lead to really delightful software that I thouroughly enjoyed working with. Ron emphasized how APIs and libraries are really just micro-DSLs in disguise, which allows us to use the learnings from this course in our development endevours.

Alright, let's build a programming language. Meet "Khichdi", a functional language which supports

- Mutable variables (by-reference and by-value)
- First-class (recursive) functions
- Arrays, dictionaries and pairs
- Backstops (more on this later)
- Generalized search
- Exceptions
- Continuations

You can check out the full code at the repository [here](https://github.com/kumarpit/khichdi)

Let's start with the basics and define the surface-level syntax for conditionals, functions, identifiers and arithmetic.

```Racket
;; Khichdi (खिचड़ी)
;; a toy programming language with
;; - Mutable variables (by-reference and by-value)
;; - First-class (recursive) functions
;; - Arrays, dictionaries and pairs
;; - Backstops (more on this later)
;; - Generalized search
;; - Exceptions
;; - Continuations

;; interp. expressions in a language that supports conditionals, functions and arithmetic
;; Its syntax is defined by the following BNF:
<Khichdi> ::=
(ARITHMETIC)
          <num>
        | {+ <Khichdi> <Khichdi>}
        | {- <Khichdi> <Khichdi>}
(IDENTIFIERS)
        | {with {<id> <Khichdi>} <Khichdi>}
        | <id>
(CONDITIONALS)
        | {if0 <Khichdi> <Khichdi> <Khichdi>}
(FUNCTIONS)
        | {<Khichdi> <Khichdi>}
        | {fun {<id>} <Khichdi>}
 where
 {with {x named} body} ≡ { {fun {x} body} named}
```

Writing a parser in Racket is straighforward, but before we get started with parsing we need to define a data type for Khichdi. Our parser will transform the input to a Racket expression of this type.

```Racket
(define-type Khichdi
  [num (n number?)]
  [add (lhs Khichdi?) (rhs Khichdi?)]
  [sub (lhs Khichdi?) (rhs Khichdi?)]
  [id (name rid?)]
  [fun (param rid?) (body Khichdi?)]
  [app (rator Khichdi?) (arg Khichdi?)]
  [if0 (predicate Khichdi?) (consequent Khichdi?) (alternative Khichdi?)])
```

Using this definition, we can write the parser.

```Racket
;; the add case
[`{+ ,sexp1 ,sexp2}
       (add (parse/ralph sexp1)
            (parse/ralph sexp2))]
;; the fun case
[`{fun {,x} ,sexp} #:when (rid? x)
                         (fun x (parse/khichdi sexp))]
;; and so on...
```

The interpreter looks like this.

```Racket
;; Khichdi Env -> Value
;; environment passing interpreter
;; EFFECT: signals an error in case of a runtime error
(define (interp/ralph-env r env)
  (type-case Khichdi r
    [num (n) (numV n)]
    [add (l r) (add-value (interp/khichdi-env l env)
                          (interp/khichdi-env r env))]
    [sub (l r) (sub-value (interp/khichdi-env l env)
                          (interp/khichdi-env r env))]
    [id (x) (lookup-env env x)]
    [fun (x body) (funV x body env)]
    [app (rator rand) (apply-value (interp/khichdi-env rator env)
                                   (interp/khichdi-env rand env))]
    [if0 (p c a)
         (if  (zero-value? (interp/khichdi-env p env))
              (interp/khichdi-env c env)
              (interp/khichdi-env a env))]))
```

Before moving on and adding new features, we should prepare our iterpreter for effects - i.e monadify our code.

@TODO!!!
