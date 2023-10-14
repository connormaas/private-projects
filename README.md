# Private Projects

## Overview

This file consists of comprehensive descriptions for my non-public projects, accompanied by the links to said projects. For quick access to the private project repositories, I have provided the following links:

- Proxy: https://github.com/connormaas/proxy-2022
- Malloc: https://github.com/connormaas/malloc
- Shell: https://github.com/connormaas/tsh-shel
- Cache Simulation: https://github.com/connormaas/cache-siml
- CPS Mergesort: https://github.com/connormaas/mergesort-cps
- AI Checkers: https://github.com/connormaas/ai-checkers
- Lightsout Solver: https://github.com/connormaas/lightsout
- c0vm: https://github.com/connormaas/c0vm
- Exp Language: https://github.com/connormaas/exp-language

However, please note that these links will lead to 404 error unless I have granted you access to them. Please reach out to me at connor@maasclinic.com and I would be happy to provide applicable access.

## [Proxy](https://github.com/connormaas/proxy-2022)

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

## [Malloc](https://github.com/connormaas/malloc)

#### Overview
This is an implementation of the C memory allocation functions: `malloc`, `calloc`, `realloc`, and `free`, designed to have the same behavior and performance as the corresponding C Standard Library functions.

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

## [Shell](https://github.com/connormaas/tsh-shell)

#### Overview
A Unix command-line shell that interprets user inputs, capable of executing concurrent built-in and user-driven commands. This shell supports error handling, signal handling, and IO redirection.

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

## [Cache Simulation](https://github.com/connormaas/cache-sim)

#### Overview

This program simulates a L1 [cache](https://en.wikipedia.org/wiki/Cache_(computing)). It integrates a least recently used ([lru](https://www.educative.io/implement-least-recently-used-cache#)) eviction policy. An L1 cache is generally structured as follows:

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

The program begins by parsing the command line for specific arguments, including `s` (the number of bits required to represent the set index), `b` (the number of bits required to represent the block offset), and `E` (the number of lines per set). It then parses the specified trace file, which contains load and save operations with specific addresses. Additionally, the number of bytes required for each operation is included in said file. The program returns the number of: 
- Hits: The number of times the cache successfully finds and retrieves the requested data.
- Misses: The number of times the cache fails to find the requested data, necessitating a fetch from a higher-level memory.
- Evictions: The process of removing a cache line from the cache to make space for new data, typically when the cache is full.
- Dirty Bytes: Bytes in the cache that have been modified but not yet written back to the main memory or next level of cache.
- Dirty Byte Evictions: The process of evicting cache lines that contain dirty bytes, which requires writing them back to the main memory before eviction.

## [CPS Mergesort](https://github.com/connormaas/mergesort-cps)

#### Overview
This is a parallel implementation of mergesort written in [continuation-passing style](https://en.wikipedia.org/wiki/Continuation-passing_style) utilizing the functional programming langauge, SML. The sequential complexity of this solution is the standard O(nlogn) for mergesort. However, the parallel complexity is O(n). We come to this conclusion using the [brick method]():

In normal merge sort, we have:
Split : O(n)
Merge: O(n)
Other operations: O(1)

Therefore, our recurrence is:

T(n) = 2T(n/2) + O(n)

However, the parallelization of our calls to sort allow our recurrence to become:

T(n) = T(n/2) + O(n)

We can conclude that this recurrence is *root dominated*. That is, if the root has cost $C(r)$, level $i$ will have total cost at most $(\frac{1}{2})^i C(r)$. This is because the cost of the children of every node on a level decrease by at least a factor of 2 to the next level. The total cost is therefore upper bounded by 

$\sum_i \left(\frac{1}{2}\right)^i C(r)$

This is a decaying geometric sequence and therefore is upper bounded by $\frac{1}{2 - 1} C(r)$ = C(r). Therefore, the parallel complexity of this program is O(n)!

## [AI Checkers](https://github.com/connormaas/ai-checkers)

#### Overview
This program is a functional AI capable of effectively playing checkers, using the alpha-beta algorithm (alpha-beta pruning). This algorithm considers future moves by the opponent for each possible move that it could make on that turn. Although this algorithm is relatively simple, its functional implementation becomes more challenging.

#### Background
Inspired by [JVM](https://en.wikipedia.org/wiki/Java_virtual_machine). More information on the alpha-beta algorithm can be found [here](https://www.chessprogramming.org/Alpha-Beta).

## [LightsOut Solver](https://github.com/connormaas/lightsout-fast)

#### Overview
Lightsout is an electronic board game, consisting of a 5x5 board where the main goal is to turn off all the lights. However, when a light is turned off, the neighboring lights (left, right, up, down) are turned on. A more formal and in-depth description of the game can be found [here](https://en.wikipedia.org/wiki/Lights_Out_(game)). This program, written in C, produces a solution to any solvable board using the minimum number of moves. The solution is outputted step-by-step, including the coordinates of each required move, as well as a depiction of the board using “#” for lights that are on and “0” for lights that are off. Additionally, a hash table library was implemented, which stores previous board solutions, preventing repetitive computation.

## [c0vm](https://github.com/connormaas/c0vm)

#### Overview
c0vm is a virtual machine for C0, a C-like language developed at CMU. It's a stack-based machine that handles arithmetic operations and other instructions. The call stack consists of frames, each containing local variables, a local operand stack, and a return address. Operands are popped from an operand stack and their result is pushed back onto the stack. Functions defined in a source file are kept in a “function pool” and library functions are kept in a “native pool”.

## [Exp Language](https://github.com/connormaas/exp-language)

#### Overview
Exp is a simple programming language in C, capable of effectively performing mathematical calculations based on the standard order of operations using the left-associative principle. The operators are listed from lowest to highest precedence as follows (items with equivalent precedence are grouped together): `||`, `&&`, `(<, >, ==, !=)`, `<<`, `>>`, `(+, -)`, `(*, /, %)`, `**`. The description of each operator can be found in the image file within this repository. The language first implemented stack, queue, dictionary, and libraries to assist with parsing an entry for computation. The function “parse” takes in a queue of strings (numbers and operators) and a dictionary of precedences for each operator. The stack is used to store the current value and operators that have been popped from the queue as applicable. Once the stack is empty, the result is returned (assuming that the ordered combination of operators and values are valid for a complete operation).
