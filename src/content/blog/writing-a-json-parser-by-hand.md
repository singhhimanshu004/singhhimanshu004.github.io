---
title: "From Text to Tree — Writing a JSON Parser by Hand"
description: "A two-stage JSON parser in Python: a lexer that catches malformed numbers and surrogate pairs, and an LL(1) recursive-descent parser whose call stack mirrors the grammar. Plus the edge cases that separate a toy from a real parser."
pubDate: 2026-06-08
tags:
  - "Parsing"
  - "Compilers"
  - "Python"
  - "Software Engineering"
draft: false
---

Everyone uses `JSON.parse` or `json.loads`. Almost no one has written one. That's a shame, because a JSON parser is the single best on-ramp to how *every* compiler and interpreter works — and it's small enough to finish in a weekend.

So I built one from scratch in Python, no regex grammar, no parser generator, as the second tool in my [Coding Challenges Deep Dive](https://github.com/singhhimanshu004/coding-challenges-deepdive). The goal wasn't to beat the standard library. It was to understand the two-stage pipeline that turns a flat string into a structured tree, and to handle the edge cases that quietly separate a toy from something you'd trust.

## Two stages, one clean seam

```
"raw text"  ──▶  Lexer  ──▶  [tokens]  ──▶  Parser  ──▶  Python value
```

The **lexer** (tokenizer) walks the input one character at a time and emits a flat list of tokens — `LBRACE`, `STRING`, `NUMBER`, `COMMA`, and so on. It validates everything that can be checked *locally*: a number's shape, a complete string escape, a correctly spelled keyword. It knows nothing about grammar.

The **parser** consumes that token list and decides whether the tokens are arranged *legally* — whether that comma belongs where it sits — and builds the actual `dict`, `list`, `str`, `int`, `float`, `bool`, or `None`.

Keeping those concerns apart is the entire trick. The lexer never worries about nesting; the parser never worries about how a `\uXXXX` escape decodes. Each stage does one job, and both become easy to test in isolation.

## The lexer: where the gotchas live

People assume the parser is the hard part. For JSON, most of the genuinely fiddly correctness lives in the lexer — specifically numbers and strings.

JSON numbers look simple and aren't. The spec's grammar forbids things that *feel* legal:

```
number = [ '-' ] int [ frac ] [ exp ]
int    = '0' | digit1-9 *digit
```

That grammar rejects `01` (leading zero), a bare `-`, `.5` (no integer part), and `1.` (no fraction digits). My scanner mirrors the grammar rule-for-rule rather than reaching for a regex, so each rejection maps to an obvious line of code:

```python
# Integer part: a lone '0', or a non-zero digit followed by more.
if self._peek() == "0":
    self._advance()
    if self._peek().isdigit():
        raise self._error("Leading zeros are not allowed in numbers")
elif self._peek().isdigit():
    while self._peek().isdigit():
        self._advance()
else:
    raise self._error("Invalid number: expected a digit")
```

One detail I care about: integers stay integers. An integral JSON number becomes a Python `int` (arbitrary precision — so a 30-digit id keeps every digit), and only `.`/`e` shapes become `float`. Parse `10000000000000000001` and you get the exact value back, not a lossy float.

Strings hide the other monster: `\uXXXX` escapes and UTF-16 **surrogate pairs**. A character outside the Basic Multilingual Plane — an emoji, say — is encoded in JSON as *two* `\u` escapes, a high surrogate followed by a low one, that must be recombined into a single code point:

```python
if 0xD800 <= code <= 0xDBFF:          # high surrogate
    if self._peek() == "\\" and self._peek(1) == "u":
        self._advance(); self._advance()
        low = self._read_hex4()
        if not (0xDC00 <= low <= 0xDFFF):
            raise self._error("Invalid low surrogate in \\u escape")
        combined = 0x10000 + ((code - 0xD800) << 10) + (low - 0xDC00)
        return chr(combined)
    raise self._error("Unpaired high surrogate in \\u escape")
```

Skip this and `"\uD83D\uDE00"` either crashes or produces garbage instead of 😀. It's the kind of case a toy parser ignores and a real one must not.

## The parser: the grammar *is* the code

JSON's grammar is recursive — a value can be an array whose elements are values, which can themselves be arrays — and that recursion is the whole reason **recursive descent** is such a natural fit. You write one function per grammar rule and let the call stack track *where am I nested*:

```
value   = object | array | STRING | NUMBER | true | false | null
object  = '{' [ member (',' member)* ] '}'
member  = STRING ':' value
array   = '[' [ value (',' value)* ] ']'
```

Each rule becomes a `_parse_*` method. `_parse_value` peeks at the next token and dispatches:

```python
def _parse_value(self):
    ttype = self._peek().type
    if ttype is TokenType.LBRACE:
        return self._parse_object()
    if ttype is TokenType.LBRACKET:
        return self._parse_array()
    if ttype in (TokenType.STRING, TokenType.NUMBER):
        return self._advance().value
    if ttype in (TokenType.TRUE, TokenType.FALSE, TokenType.NULL):
        return self._advance().value
    raise JSONParseError(f"Expected a value but found {self._describe(self._peek())}",
                         self._peek().line, self._peek().column)
```

No backtracking is needed because JSON is **LL(1)**: a single token of lookahead always tells you, unambiguously, which rule applies. That one property is why the parser is a clean dispatch instead of a tangle of "try this, undo, try that." The same shape — peek, dispatch, recurse — is the foundation you'd reuse to build a `jq`, a calculator, or a Lisp interpreter.

## The edge cases that earn trust

A parser that accepts valid JSON is half a parser. The half that matters is *rejecting* invalid JSON precisely:

- **Trailing commas.** `{"a":1,}` and `[1,]` are invalid JSON, even though almost every hand-written object tempts you to allow them. After consuming a comma, the parser explicitly checks that a real member or element follows.
- **Trailing junk.** `parse` runs the value, then asserts the next token is `EOF`. So `{"a":1} extra` fails instead of silently returning `{"a": 1}` and ignoring the rest.
- **Empty input.** A document of only `EOF` is invalid — `json.loads("")` raises, and so does this.
- **Duplicate keys.** RFC 8259 permits either behavior. The default matches the standard library (last value wins), but a flag flips it to raise instead — a small nod to the fact that "permitted" and "safe" aren't the same.

Every error carries an exact `line` and `column`, tracked as the lexer advances, so a failure says *where* it happened, not just *that* it happened.

## Testing against an oracle

The best part of cloning a standard tool: you have a reference implementation to test against. The most valuable tests don't assert hand-written expected values — they assert that **my parser agrees with `json.loads`**, on both acceptance and rejection:

```
test_valid_matches_stdlib
test_invalid_agrees_with_stdlib
test_large_integers_keep_precision
test_surrogate_pair_escape
test_error_reports_line_and_column
test_malformed_numbers_rejected
```

`test_invalid_agrees_with_stdlib` is the one that found my real bugs. It's easy to accept valid input; matching the standard library on what to *reject* is where a parser is actually forced to be correct. Differential testing against an oracle is the highest-leverage thing you can do when you reimplement something that already exists.

## Why this is worth your weekend

A JSON parser is the smallest honest version of the lexer → parser → tree pipeline that powers every language tool you use. Build one and tokenization, single-token lookahead, recursive descent, and precise error reporting stop being words in a compilers course and become things you've actually done. Then the next interpreter — a calculator, a query language, a toy Lisp — is just the same skeleton with a bigger grammar.

Full source, tests, and a teaching README are here: [json-parser](https://github.com/singhhimanshu004/coding-challenges-deepdive/tree/master/phase-01-foundations/json-parser).

**Further reading:** RFC 8259 (the JSON spec — short and worth reading in full); *Crafting Interpreters* by Robert Nystrom for where recursive descent goes next.
