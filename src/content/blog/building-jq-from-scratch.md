---
title: "A Filter Is a Function That Streams — Building jq From Scratch"
description: "jq looks like a magic query syntax until you build it. The key insight: every filter is a function from one input value to a stream of output values. Here's the lexer, the precedence-climbing parser, and the tree-walking evaluator that makes `.users[] | .name` work."
pubDate: 2026-06-13
tags:
  - "Interpreters"
  - "Parsing"
  - "Go"
  - "JSON"
heroImage: "/blog/jq-stream.svg"
heroAlt: "A jq pipeline: one JSON object enters .users[], which emits a stream of three objects, each fed through .name to produce three string outputs."
heroCaption: "Every jq filter maps one input value to a stream of outputs. The pipe runs the right side once per left output and flattens the result."
draft: false
---

`jq '.users[] | .name'` reads like line noise until you understand the one idea the whole language is built on: **a jq filter is a function from a single input value to a *stream* of output values.** Not one value — a stream. Identity (`.`) emits one. Iteration (`.[]`) emits many. `select(...)` might emit zero. Once that clicks, every confusing piece of jq syntax becomes obvious, and you can build the whole thing.

So I did, in Go, as part of my [Coding Challenges Deep Dive](https://github.com/singhhimanshu004/coding-challenges-deepdive). It's a real interpreter — **lexer → parser → evaluator** — supporting field access, iteration, pipes, comma, array construction, arithmetic and comparison, `select`, `map`, `length`, and friends. It's the most "language-y" project in the set, and it demystified how every interpreter you've ever used is structured.

## The whole language in one type signature

Here's the function that *is* jq:

```go
func eval(node Node, input any) ([]any, error)
```

One AST node, one input value, **a slice of output values**. That `[]any` return — the stream — is the entire conceptual model. Real jq is lazily streaming; I eagerly collect into slices for readability, a trade-off I note in the README. Everything else is just implementing this signature for each node type.

Identity is the base case — one input, one output:

```go
case Identity:
    return []any{input}, nil
```

Iteration is where the stream stops being a singleton:

```go
func evalIterate(input any) ([]any, error) {
    switch v := input.(type) {
    case []any:
        return append([]any{}, v...), nil       // every array element
    case *Object:
        out := make([]any, 0, v.Len())
        for _, k := range v.Keys() {
            val, _ := v.Get(k)
            out = append(out, val)               // every object value
        }
        return out, nil
    default:
        return nil, fmt.Errorf("cannot iterate over %s", typeName(input))
    }
}
```

## The pipe is the most elegant six lines in the project

`A | B` means: run `A`, then run `B` on *each* of A's outputs, and concatenate. That's it. That's the pipe:

```go
func evalPipe(n Pipe, input any) ([]any, error) {
    lefts, err := eval(n.Left, input)
    if err != nil { return nil, err }
    var out []any
    for _, v := range lefts {
        rights, err := eval(n.Right, v)          // run the right on each left output
        if err != nil { return nil, err }
        out = append(out, rights...)             // flatten
    }
    return out, nil
}
```

This is exactly the `flatMap` / `SelectMany` you've used in other languages, and it's why `.users[] | .name` works: `.users[]` produces a stream of user objects, and `.name` runs once per object, yielding a stream of names. The pipe is *monadic bind* for the list monad — but you don't need that vocabulary to see that it's just a nested loop with a flatten.

## Parsing: precedence without a parser generator

jq has real operator precedence — `|` binds looser than `,`, which binds looser than `==`, then `+`/`-`, then `*`/`/`, and path suffixes like `.foo` and `[0]` bind tightest. The clean way to encode that, with no parser-generator dependency, is **recursive descent with one method per precedence level**, lowest first:

```
pipe     := comma ( '|' comma )*
comma    := compare ( ',' compare )*
compare  := additive ( ('=='|'!='|'<'|'>'|'<='|'>=') additive )?
additive := multiply ( ('+'|'-') multiply )*
multiply := postfix  ( ('*'|'/') postfix )*
postfix  := primary ( '.'IDENT | '['expr?']' | '?' )*
primary  := '.' IDENT? | '[' pipe? ']' | '(' pipe ')' | NUMBER | STRING | IDENT '(' args ')'
```

Each grammar rule is one function that calls the next-tighter rule and loops on its own operators. The *structure of the grammar is the structure of the code* — lower precedence sits higher in the call stack, so by the time you reach `primary` you're parsing the tightest-binding atoms. This is the same shape as Python's own grammar, and once you've written one you can parse almost any expression language.

## The evaluator is a type switch, not a visitor pattern

In a Java or textbook design you'd put an `Eval()` method on every node class (the Visitor pattern). Go's idiomatic move is a single `eval` function with a **type switch** over the concrete node types:

```go
switch n := node.(type) {
case Identity:       return []any{input}, nil
case Field:          return evalField(n, input)
case Iterate:        return evalIterate(input)
case Pipe:           return evalPipe(n, input)
case Comma:          // concatenate left stream + right stream
case ArrayConstruct: return evalArrayConstruct(n, input)
case Call:           return evalCall(n, input)   // select, map, length, ...
case Try:            // run Expr, swallow errors into an empty stream
}
```

Keeping all evaluation in one place made the streaming model *visible*: every branch returns a `[]any`, so you can see at a glance how each construct contributes to the stream. `Comma` returns left ++ right. `Try` turns an error into an empty stream — which is the mechanism behind jq's `?` operator and the whole "optional" path syntax.

## Why a query language is the perfect interpreter starter

A full programming language is a year-long project. jq's filter language is small enough to finish in a weekend but contains *every* concept you need: a lexer, a precedence-aware parser, an AST, a tree-walking evaluator, builtin functions, and an error model. And because the evaluation model — value in, stream out — is so uniform, the code stays legible the whole way through. Building it changed how I read jq one-liners: I now see `.users[] | select(.age > 30) | .name` as a clear pipeline of stream transformations, not a cryptic spell.

Full source — `lexer.go`, `parser.go`, `eval.go`, `jsonval.go`, `output.go` — with a substantial test suite: [jq](https://github.com/singhhimanshu004/coding-challenges-deepdive/tree/master/phase-03-advanced-cli/jq).

**Further reading:** Bob Nystrom's *Crafting Interpreters* (the best book on tree-walking interpreters, free online); the official jq manual to see how far the real language goes; and my [JSON parser](/blog/writing-a-json-parser-by-hand/) post for the lexer/parser stage in isolation.
