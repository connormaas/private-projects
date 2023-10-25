# Private Projects

## Overview

This file consists of comprehensive descriptions for my non-public projects, accompanied by the links to said projects. For quick access to the private project repositories, I have provided the following links:

- Malloc: https://github.com/connormaas/malloc
- Proxy: https://github.com/connormaas/proxy-2022
- Shell: https://github.com/connormaas/tsh-shell
- Cache Simulation: https://github.com/connormaas/cache-sim
- AI Checkers: https://github.com/connormaas/ai-checkers
- CPS Mergesort: https://github.com/connormaas/mergesort-cps
- Lightsout Solver: https://github.com/connormaas/lightsout
- c0vm: https://github.com/connormaas/c0vm
- Exp Language: https://github.com/connormaas/exp-language

However, please note that these links will lead to a 404 error unless I have granted you access to them. Please reach out to me at cmaas@andrew.cmu.edu and I would be happy to provide applicable access.

## Malloc

#### Private Repository

The full reporistory can be found [here](https://github.com/connormaas/malloc). To request access, email cmaas@andrew.cmu.edu.

#### Overview
In this project, I implement the C memory allocation functions: `malloc`, `calloc`, `realloc`, and `free`, designed to have the same behavior and performance as their corresponding C Standard Library functions.

#### Implementation
- The heap is implemented with blocks that contain headers and payloads. The payloads are zero-length arrays, where the last 8 bytes are used to represent the footer (only in freed blocks). Headers and footers of each block are identical and contain both the size of the payload, as well as information on the allocation status of previous blocks.
- Mini-blocks: mini-blocks are used for small allocations where bytes would be wasted in the payload. They are only 16 bytes (as opposed to regular free blocks which are a minimum of 32 bytes). The first 8 bytes are for the header and the additional 8 bytes are for the payload. When mini-blocks are free, the payload is used as a "next" pointer, allowing for the creation of a singly linked list between all mini-blocks.
- Performance is optimized by implementing segregated free lists, which aggregate blocks of similar size for quicker access.
- Utilization is optimized through several strategies:
  1. During allocation, the free blocks being allocated are split into smaller blocks.
  2. During freeing, blocks are coalesced with their neighbors into a larger blocks if either or both of their neighbors are free.
  3. Allocated blocks do not contain or require footers (the bytes of the footer are used as additional payload bytes).
  4. There is a separately linked list containing free mini-blocks, which significantly reduces wasted bytes in small allocations.
  5. The use of a [best-fit](https://www.geeksforgeeks.org/best-fit-allocation-in-operating-system) policy for allocating free blocks slightly improves the utilization.

## Proxy

#### Private Repository

The full reporistory can be found [here](https://github.com/connormaas/proxy-2022). To request access, email cmaas@andrew.cmu.edu.

#### Introduction
A web proxy serves as a "middle-man" between a client and server. To the client, it acts as a server, interpreting requests and providing responses. While to the server, it acts as a client, generating requests and reading responses.

#### High-Level Overview
1. Open a socket which listens for a client.
2. Use threads for each new client.
3. Read requests from client.
4. Search the cache (shared response cachen between all threads). If present, write this response to the client and skip steps 5-7.
5. Write the client's request to the server.
6. Read the server's response and cache it.
7. Write the response back to the client.

#### Details
- Capable of reading and writing both binary and text data.
- Handles requests concurrently from different clients.
- Employs proper error checking and will not exit the program when errors arise.
- Uses a cache (shared with all threads) which stores server responses using client requests as keys. The proxy uses mutexes to prevent data races and other data sharing issues.

## Shell

#### Private Repository

The full reporistory can be found [here](https://github.com/connormaas/tsh-shell). To request access, email cmaas@andrew.cmu.edu.

#### Overview
In this project, I implement a Unix command-line shell that interprets user inputs, capable of executing concurrent built-in and user-driven commands. This shell supports error handling, signal handling, and IO redirection.

#### Creating Processes
- A user may simply enter an executable that they wish to run, and if the file is reachable, the process will immediately begin taking place.
- If the `&` operator is added to the end of the line, the process will run in the background and the user may continuing employing the shell as normal.
- If the `&` operator is not added, the process will run in the foreground. This can be terminated or paused using `ctrl-c` or `ctrl-z` respectively.

#### Details
- Each job being executed has a process ID (PID) and a job ID (JID), which is unique to that process. This is used for identification and further manipulation of processes.
- Each child process is given its own process group, so when signals are forwarded to that process group, it will not interfere with the parent process. Specifically, this is necessary because otherwise these signals would be sent back to the original shell (parent), as well as every process created by it.
- When a background process begins, its JID and PID as well as the command line which invoked the execution are displayed.
- The user typing `ctrl-c` will terminate the execution of the foreground group.
- The user typing `ctrl-z` will stop the execution of the foreground group.
- Children are properly [reaped](http://www.microhowto.info/howto/reap_zombie_processes_using_a_sigchld_handler.html) using a signal handler. The signal handler is able to deal with multiple terminated children despite receiving just one signal.
- During a foreground process, background processes that terminate do not become [zombies](http://www.microhowto.info/howto/reap_zombie_processes_using_a_sigchld_handler.html), as signals are handled concurrently with process execution.

#### Built-in Commands
- `quit`: terminates the shell.
- `jobs`: lists all background jobs.
- `bg job`: resumes job by sending it a SIGCONT signal, and then runs it in the background. The job argument can be either a PID or JID.
- `fg job`: command resumes job by sending it a SIGCONT signal, and then runs it in the foreground. The job argument can be either a PID or JID.

#### IO Redirection
- The operators `<` (input) and `>` (output) proceed the name of the file to which the input or output will be redirected.
- The default input and output are stdout and stdin respectively.
- Users may redirect output to any valid file which they have permission to write to.
- If a request is made to write to a file that doesn't exist, a new file will be created and written to.
- Output can also be redirected from built-in commands, such as the `jobs` command, which lists the background jobs as described in the previous section.
- Users may redirect input from any valid file which they have permission to read from. The file itself must exist or the process will be terminated.

#### Error Handling
- Error handling has been carefully constructed to ensure no input can break the shell.

## Cache Simulation

#### Private Repository

The full reporistory can be found [here](https://github.com/connormaas/cache-sim). To request access, email cmaas@andrew.cmu.edu.

#### Overview
This program simulates a L1 [cache](https://en.wikipedia.org/wiki/Cache_(computing)), which integrates two policies: 
1. [write-back](https://www.geeksforgeeks.org/write-through-and-write-back-in-cache) data synchronization
2. least recently used ([LRU](https://www.educative.io/implement-least-recently-used-cache#)) eviction

#### Illustration

```
v = valid bit
d = dirty bit

+----------------------------------------------------------------------------------+

|                                   Cache Set 0                                    |

| +----------------------------+               +----------------------------+      |

| |            Line 0          |               |            Line 1          | ...  |

| | v | d | tag |    offset    |               | v | d | tag |    offset    |      |

| +----------------------------+               +----------------------------+      |

+----------------------------------------------------------------------------------+

+----------------------------------------------------------------------------------+
|                                   Cache Set 1                                    |
| +----------------------------+               +----------------------------+      |
| |            Line 0          |               |            Line 1          | ...  |
| | v | d | tag |    offset    |               | v | d | tag |    offset    |      |
| +----------------------------+               +----------------------------+      |
+----------------------------------------------------------------------------------+
                                        .                          
                                        .                                 
                                        .                                   
+----------------------------------------------------------------------------------+
|                                   Cache Set _                                    |
| +----------------------------+               +----------------------------+      |
| |            Line 0          |               |            Line 1          | ...  |
| | v | d | tag |    offset    |               | v | d | tag |    offset    |      |
| +----------------------------+               +----------------------------+      |
+----------------------------------------------------------------------------------+
```

#### Details
The program begins by parsing the command line for specific arguments, including `s` (the number of bits required to represent the set index), `b` (the number of bits required to represent the block offset), and `E` (the number of lines per set). It then parses a specified trace file, which contains load and save operations as well as corresponding memory addresses. The number of bytes required for each operation is also included in said trace file. Assuming th trace file is well-formatted, the program returns the number of: 
- `Hits`: The number of times the cache successfully finds and retrieves the requested data.
- `Misses`: The number of times the cache fails to find the requested data, necessitating a fetch from a main memory.
- `Evictions`: The process of removing a cache line from the cache to make space for new data, taking place when the cache is full.
- `Dirty Bytes`: Bytes in the cache that have been modified but not yet written back to the main memory or next level of cache.
- `Dirty Byte Evictions`: The process of evicting cache lines that contain dirty bytes, which requires writing them back to the main memory before eviction.

## Parallel Mergesort

#### Private Repository

The full reporistory can be found [here](https://github.com/connormaas/mergesort-cps). To request access, email cmaas@andrew.cmu.edu.

#### Overview
In this project, we implement parallel mergesort in [continuation-passing style](https://en.wikipedia.org/wiki/Continuation-passing_style) utilizing the functional programming langauge, SML. The sequential complexity of this solution is the same as standard mergesort: O(nlogn). However, the parallel complexity is $O(n)$.

In normal mergesort, we have: 
- `split` : $O(n)$
- `merge`: $O(n)$
- Other operations: $O(1)$

Since we make two recursive calls to `sort` on each half aftering splitting, our recurrence is:

$T(n) = 2T(\frac{n}{2}) + O(n)$

In this implementation, however, the parallel calls to `sort` reduce our recurrence to:

$T(n) = T(\frac{n}{2}) + O(n)$

We can conclude that this recurrence is *root dominated*. That is, if the root has cost $C(r)$, level $i$ will have total cost at most $(\frac{1}{2})^i C(r)$. This is because the cost of the children of every node on a level decrease by at least a factor of 2 to the next level. The total cost is therefore upper bounded by 

$\sum_i \left(\frac{1}{2}\right)^i C(r)$

This is a decaying geometric sequence, so our upper bounded becomes $\frac{1}{2 - 1} C(r)$ = C(r). Therefore, the parallel complexity of this program is $O(n)$.

## AI Checkers

#### Private Repository

The full reporistory can be found [here](https://github.com/connormaas/ai-checkers). To request access, email cmaas@andrew.cmu.edu.

#### Overview
This program hosts an AI Checkers player effective at winning games versus real, human opponents. Implemented in the functional programming language SML, it uses [alpha-beta pruning](https://www.chessprogramming.org/Alpha-Beta) to make decisions. In essence, this algorithm selects its next move by exploring all potential future scenarios, but smartly disregards options that are clearly unfavorable or overly speculative. While this algorithm is conceptually simple, its functional implementation becomes much more challenging.

## LightsOut Solver

#### Private Repository

The full reporistory can be found [here](https://github.com/connormaas/lightsout-fast). To request access, email cmaas@andrew.cmu.edu.

#### Overview

[Lightsout](https://en.wikipedia.org/wiki/Lights_Out_(game)) is an electronic board game, consisting of a 5x5 board where the main goal is to "turn off all the lights." However, when a light is flipped, the neighboring lights (left, right, up, down) are also flipped. This program, written in C, produces a solution to any solvable board using the minimum number of moves. The program outputs a step-by-step solution to any solvable board . Additionally, I implemented a hash table library to stores previous board solutions, preventing any repetitive computation.

## c0vm

#### Private Repository

The full reporistory can be found [here](https://github.com/connormaas/c0vm). To request access, email cmaas@andrew.cmu.edu.

#### Overview
The C0 Virtual Machine (C0VM) is a stack-based virtual machine designed to execute [C0](https://c0.cs.cmu.edu/docs/c0-reference.pdf) bytecode. It draws inspiration from the Java Virtual Machine ([JVM](https://www.geeksforgeeks.org/jvm-works-jvm-architecture)).

#### Key Features:
- Bytecode Execution: C0VM reads and executes compiled C0 code represented in a bytecode format. It processes one instruction at a time and executes it based on the operational semantics defined for each instruction.
- Value Representation: Primitive types are represented as 32-bit signed integers, while reference types are represented as void pointers. These are encapsulated under a special type called `c0_value`, which can hold either a primitive or a reference type.
- Runtime Data Management: The runtime environment manages an operand stack, bytecode, a program counter, local variables, and a call stack to ensure the correct execution flow of the program. 
- Instruction Set: The instruction set comprises various instructions covering stack manipulation, arithmetic operations, local variables handling, control flow management, function calls and returns, memory allocation, and load/store operations. 
- Error Handling: C0VM is equipped to detect and handle various runtime errors, issuing error messages for anomalies such as null dereferencing, array index out of bounds, and division by zero. 
- Heap Management: Manages a C0 runtime heap to satisfy C0 allocation requests, with memory allocation and deallocation handled through C runtime heap functions, ensuring memory safety.

#### Structure:
- Operand Stack: The operand stack is a Last In First Out ([LIFO](https://www.geeksforgeeks.org/lifo-last-in-first-out-approach-in-programming)) data structure where C0 values are pushed and popped during the execution of instructions.
- Bytecode: The bytecode is an array of bytes where each byte represents an instruction or part of an instruction for the current C0 function.
- Program Counter: The program counter holds the address of the currently executing instruction, advancing to the next instruction after the current instruction is executed.
- Local Variables: Local variables are stored in a `c0_value` array with a designated size for each function, facilitating the storage and retrieval of function-local data.
- Call Stack: The call stack comprises frames, each containing local variables, a local operand stack, and a return address, which facilitates function calls and returns.
- Constant Pools: The constant pools house numerical and string constants which can be referenced by the bytecode during execution.
- Function Pools: Function pools store C0 functions and library functions in separate pools, facilitating function call resolution.
- Heap: The heap contains C0 strings, arrays, and cells, with memory allocated directly using C runtime heap functions, ensuring efficient memory management.

## Exp Language

#### Private Repository

The full reporistory can be found [here](https://github.com/connormaas/exp-language). To request access, email cmaas@andrew.cmu.edu.

#### Overview
Exp is a simple programming language implemented in C, which evaluates mathematical expressions based on a user-defined order of operations and left-associative principle. Valid operators include:

- `**`: power
- `*`, `/`, `%`: multiply, divide, modulo
- `+`, `-`: add, subtract
- `<<`, `>>`, arithmetic left-shift, arithmetic right-shift
- `<`, `>`, `==`, `!=`: less than, greater than, equal to, not equal to
- `||`, `&&`: or, and

#### Implementation
The function `parse` takes in a queue of strings (numbers and operators) and a dictionary of precedences for each operator. A stack is used to store the current value and operators that have been popped from the queue. Once the stack is empty, the result is returned. The handling of invalid inputs is accounted for in all cases.
