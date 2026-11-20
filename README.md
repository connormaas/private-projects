# Private Projects

Overviews of projects I keep private. The repos are available on request at connor@connormaas.com.

| Project | Language | Repo |
| --- | --- | --- |
| [Malloc](#malloc) | C | [malloc](https://github.com/connormaas/malloc) |
| [Proxy](#proxy) | C | [proxy-2022](https://github.com/connormaas/proxy-2022) |
| [Shell](#shell) | C | [tsh-shell](https://github.com/connormaas/tsh-shell) |
| [Cache Simulator](#cache-simulator) | C | [cache-sim](https://github.com/connormaas/cache-sim) |
| [c0vm](#c0vm) | C | [c0vm](https://github.com/connormaas/c0vm) |
| [Lights Out Solver](#lights-out-solver) | C | [lightsout-fast](https://github.com/connormaas/lightsout-fast) |
| [Exp Language](#exp-language) | C0 | [exp-language](https://github.com/connormaas/exp-language) |
| [Parallel Mergesort](#parallel-mergesort) | SML | [mergesort-cps](https://github.com/connormaas/mergesort-cps) |
| [AI Checkers](#ai-checkers) | SML | [ai-checkers](https://github.com/connormaas/ai-checkers) |

## Malloc

`malloc`, `calloc`, `realloc`, and `free` from scratch, with the same interface as the C standard library, tuned for throughput and memory utilization.

- **Blocks:** each block is a header plus a payload. The header packs the size with three flags: allocated, previous block allocated, and previous block mini.
- **Footers only when free:** free blocks end with a copy of the header so neighbors can coalesce in O(1). Allocated blocks drop the footer and use those 8 bytes as payload.
- **Segregated free lists:** 12 size classes, so a fit is a few hops away.
- **Mini blocks:** 16 byte blocks for tiny requests (vs. the usual 32 byte minimum), kept in their own singly linked list.
- **Bounded best fit:** take the first block that fits, then check the next few for a tighter one.
- **Splitting and coalescing** keep fragmentation low.

## Proxy

A concurrent, caching HTTP proxy. To the client it's the server, and to the server it's the client.

1. Accept a client and give it its own thread.
2. Read the request and check the shared cache. On a hit, respond immediately.
3. Otherwise, forward the request, cache the response, and send it back.

Handles binary and text data, accepts HTTP/1.0 and 1.1, and recovers from errors without exiting. The cache is LRU, reference counted, and mutex guarded, so threads never free an entry another is still reading.

## Shell

A Unix shell with job control, signal handling, and I/O redirection.

- **Jobs:** run anything in the foreground, or in the background with `&`. Every job gets a PID and JID.
- **Signals:** `ctrl-c` and `ctrl-z` go to the foreground job's process group, never the shell. A `SIGCHLD` handler reaps every finished child, even when several arrive as one signal, so nothing lingers as a zombie.
- **Builtins:** `quit`, `jobs`, `bg`, and `fg`, where `bg` and `fg` take a PID or JID.
- **Redirection:** `<` and `>` work for programs and builtins, so `jobs > file` works too. Output files are created as needed.

## Cache Simulator

Simulates an L1 cache with write back and LRU eviction. Takes the set bits `s`, block bits `b`, and lines per set `E`, replays a trace of loads and stores, and reports hits, misses, evictions, dirty bytes, and dirty bytes evicted.

Each set is a doubly linked list of lines, so a hit moves its line to the front in O(1) and the LRU victim is always the tail.

## c0vm

A stack based virtual machine for [C0](https://c0.cs.cmu.edu/docs/c0-reference.pdf) bytecode, modeled on the JVM.

- **Execution:** operand stack, program counter, locals, and a call stack of frames.
- **Instructions:** arithmetic, stack ops, control flow, function calls (including native library functions), allocation, and loads and stores.
- **Memory:** strings, arrays, and cells on the C heap.
- **Safety:** catches null dereferences, out of bounds access, and division by zero.

## Lights Out Solver

[Lights Out](https://en.wikipedia.org/wiki/Lights_Out_(game)) is a grid puzzle where pressing a light flips it and its neighbors, and the goal is to turn every light off. This solver finds the fewest moves for any solvable board using breadth first search, with boards stored as bit vectors and a hash table so no board is explored twice.

## Exp Language

An expression evaluator with user defined operator precedence and left associativity. It parses a stream of tokens with a stack of values and pending operators.

Supports `**`, `*`, `/`, `%`, `+`, `-`, `<<`, `>>`, `<`, `>`, `==`, `!=`, `&&`, and `||`.

## Parallel Mergesort

Mergesort in [continuation passing style](https://en.wikipedia.org/wiki/Continuation-passing_style). `split` and `merge` are $O(n)$ and the two recursive sorts run in parallel.

- **Work:** $W(n) = 2W(\frac{n}{2}) + O(n) = O(n \log n)$
- **Span:** $S(n) = S(\frac{n}{2}) + O(n)$. Level $i$ costs at most $(\frac{1}{2})^i C$, so the total is at most $\sum_{i \ge 0} (\frac{1}{2})^i C = 2C$, which is $O(n)$.

## AI Checkers

A checkers AI that picks moves with [alpha-beta pruning](https://www.chessprogramming.org/Alpha-Beta): it searches a fixed depth ahead and skips branches that can't change the best move, so it looks deeper in the same time.
