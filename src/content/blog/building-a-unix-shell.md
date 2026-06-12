---
title: "The Pipe That Hangs Forever — Building a Unix Shell From Scratch"
description: "What `cmd1 | cmd2 > file` actually does under the hood: tokenizing with quotes, variable expansion, fork/exec, and the file-descriptor bookkeeping that decides whether your pipeline streams cleanly or deadlocks forever."
pubDate: 2026-06-13
tags:
  - "Systems Programming"
  - "Operating Systems"
  - "Go"
  - "Unix"
heroImage: "/blog/shell-pipeline.svg"
heroAlt: "A three-stage pipeline — cat file, grep err, wc -l — connected by two kernel pipes, with a warning box explaining that if the parent keeps a write end open, the reader never sees EOF and the pipeline hangs."
heroCaption: "A pipeline is several processes running at once, wired together by kernel pipes. The whole game is closing the right file descriptors at the right time."
draft: false
---

Type `cat access.log | grep ERROR | wc -l` and three programs spring to life *simultaneously*, data flows between them through the kernel, and a single number comes back. No program in that chain knows the others exist. How?

I built a working Unix shell — `gosh` — in Go to answer that, as the Phase 3 capstone of my [Coding Challenges Deep Dive](https://github.com/singhhimanshu004/coding-challenges-deepdive). It tokenizes with proper quoting, expands `$VARIABLES`, runs builtins like `cd`, handles `&&`/`||`/`;`, redirects with `>`/`>>`/`<`/`2>`, and wires up arbitrary-length pipelines. The shell is three stages — **lex → parse → execute** — and each one taught me something I'd taken for granted for years.

## Stage 1: tokenizing is not `split()`

The first instinct is `line.split(" ")`. It dies instantly on `echo "hello world"`, which must stay a *single* argument. A real tokenizer scans byte by byte and tracks quote state, emitting operator tokens (`|`, `;`, `&&`, `>`, `2>`, …) only when it sees them **outside** quotes.

The subtlety that surprised me is that a single "word" can be made of differently-quoted pieces. Consider `'$HOME'/"$USER"` — one argument, but the first chunk must stay literal and the second must expand. So a word isn't a string; it's a list of *parts*, each tagged with whether expansion applies:

```go
type wordPart struct {
    text   string
    expand bool  // false for single-quoted or backslash-escaped chunks
}

type Word struct {
    parts []wordPart
}
```

That little `expand` flag is the entire reason `'$HOME'` prints literally while `"$HOME"` and `$HOME` print your home directory. Get the model wrong here and no amount of patching downstream will make quoting behave.

## Stage 2: a grammar, even for a shell

Shell syntax has real precedence, and the cleanest way to honor it is a small grammar with one parser method per level:

```
list     := andOr ( ';' andOr )*
andOr    := pipeline ( ('&&' | '||') pipeline )*
pipeline := command ( '|' command )*
command  := word+ redirect*
```

`;` binds loosest, then `&&`/`||`, then `|`, then a command and its redirections. The parser turns the flat token list into this tree, and the executor walks it top-down — which means the execution code reads exactly like the grammar.

## Stage 3: where the AST becomes processes

This is the part I actually wanted to understand. Two details carry the whole lesson.

**First: builtins must run in the shell's own process.** `cd` changes the *shell's* working directory. If you fork a child to run it, the child's directory changes and then the child exits — your shell hasn't moved. So a lone command checks the builtin table first and runs it in-process; only external commands get a child:

```go
if bfn, ok := builtins[args[0]]; ok {
    return bfn(sh, args, in, out, errw)   // in-process: cd can mutate the shell
}
ec := exec.Command(args[0], args[1:]...)  // external: fork/exec a child
ec.Stdin, ec.Stdout, ec.Stderr = in, out, errw
```

This is why `cd` is *always* a shell builtin and never a program in `/bin`. It physically cannot work as an external command.

**Second: a pipeline is concurrent, and that's where the deadlock lives.** For `a | b | c`, you create two pipes up front and start all three stages at once. Stage `i` reads from pipe `i-1` and writes to pipe `i`:

```go
pipes := make([]pipePair, n-1)
for i := 0; i < n-1; i++ {
    r, w, _ := os.Pipe()
    pipes[i] = pipePair{r, w}
}
// stage i: stdin = pipes[i-1].r, stdout = pipes[i].w
```

Every stage runs concurrently, so data streams through the kernel's pipe buffers as it's produced. That's why `yes | head` terminates — `head` reads ten lines, exits, and the broken pipe kills `yes`, even though `yes` would otherwise run forever.

But here's the trap that cost real debugging time. When the parent forks a child, the child *inherits a copy* of the pipe's file descriptors. A reader only sees end-of-file when **every** write end is closed. If the parent shell keeps its own copy of a pipe's write end open, the downstream reader waits for EOF that never comes — and the entire pipeline hangs forever:

```go
// Critical: close the parent's pipe copies NOW so readers can reach EOF.
closeAll(parentCloses)
wg.Wait()
```

That two-line comment is the most important thing in the whole program. A pipeline that "hangs for no reason" is almost always a file descriptor someone forgot to close. Until you build one, this is invisible folklore; after you build one, you never forget that fds are *reference-counted resources* and the kernel won't signal EOF until the count hits zero.

## Variable expansion: small but full of edge cases

Expansion happens after parsing, before execution, and only on parts the lexer tagged `expand=true`. The scanner handles `$NAME`, `${NAME}`, `$?` (last exit status), and `$$` (the shell's PID):

```go
case nx == '?':
    b.WriteString(strconv.Itoa(sh.lastStatus)); i += 2
case nx == '{':
    // scan to the closing brace so ${VAR} can abut other text
case isNameStart(nx):
    // scan a [A-Za-z_][A-Za-z0-9_]* name
```

`${VAR}` exists for exactly one reason: so you can write `${VAR}suffix` without the shell swallowing `suffix` into the variable name. Tiny feature, but it's why every config-file templating system on earth uses braces.

## Why writing a shell rewires how you see the terminal

After this project, `ps aux | grep ssh | awk '{print $2}'` stopped being a magic incantation and became a mental picture: three `fork`/`exec`ed processes, two kernel pipes, file descriptors dup'd into place, the parent dutifully closing its copies so EOF can propagate, and a `wait` collecting exit codes left to right. The exit status of the whole thing is the *last* stage's status — which is why a failing `grep` in the middle of a pipeline often goes unnoticed (and why `set -o pipefail` exists).

You use the shell every day. Building one turns a black box into a system you can reason about.

Full source — `lexer.go`, `parser.go`, `expand.go`, `executor.go`, `builtins.go`, `repl.go` — with tests for each stage: [shell](https://github.com/singhhimanshu004/coding-challenges-deepdive/tree/master/phase-03-advanced-cli/shell).

**Further reading:** the classic *Advanced Programming in the Unix Environment* (Stevens & Rago) chapters on process control and pipes; the POSIX shell command language spec for the real grammar; and `man 2 pipe` / `man 2 dup2` for the system calls underneath it all.
