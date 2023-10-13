# Private Projects

## Overview

This file consists of comprehensive descriptions for my non-public projects, accompanied by the links to said projects. For quick access to the private project repositories, I have provided the following links:

- Cache Simulation: https://github.com/connormaas/cache-sim
- Malloc: https://github.com/connormaas/malloc
- Proxy: https://github.com/connormaas/proxy-2022
- Shell (tsh): https://github.com/connormaas/tsh-shell
- Mergesort CPS (functional): https://github.com/connormaas/mergesort-cps
- Checkers AI (functional): https://github.com/connormaas/ai-checkers
- c0vm: https://github.com/connormaas/c0vm
- Exp (language): https://github.com/connormaas/exp-language
- Lights Out Solver: https://github.com/connormaas/lightsout

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
- Uses a cache (shared with all threads) which stores server responses using client requests as "keys". The proxy uses mutexes to prevent data races and other data sharing issues. **Particularly, see the information on incrementing and decrementing reference count.** See `cache.h` for more details.
- Utilizes a robust I/O package defined in `csapp.h` for reading and writing data.

## [Malloc](https://github.com/connormaas/malloc)

#### Overview
This is an implementation of the C memory allocation functions: `malloc`, `calloc`, `realloc`, and `free`, designed to have the same behavior as the C Standard Library functions.

#### Implementation
- The heap is implemented with blocks that contain headers and payloads. The payloads are zero-length arrays, where the last 8 bytes are used to represent the footer (only in freed blocks). Headers and footers of each block are identical in free blocks and contain both the size of the payload, as well as information on the allocation status of previous blocks.
- Throughput is optimized by implementing segregated free lists, which categorize blocks of similar size for quicker access.
- Utilization is optimized through several strategies:
  1. During allocation, the free blocks being allocated are split into smaller blocks.
  2. During freeing, blocks are attempted to be coalesced with their neighbors into a larger single block if either or both of their neighbors are free.
  3. Allocated blocks do not contain or require footers (the bytes of the footer are used as additional payload bytes).
  4. There is a separately linked list containing free mini-blocks, which significantly reduces wasted bytes in small allocations.
  5. The use of a better-fit search algorithm for allocating free blocks slightly improves the utilization (see the `better_fit_search` function spec for more details).
- Mini-blocks: mini-blocks are used for small allocations where bytes would be wasted in the payload. Mini blocks are only 16 bytes (as opposed to regular free blocks which are a minimum of 32 bytes). Mini blocks use the first 8 bytes for the header and the additional 8 bytes of payload space for allocation or (when free) for a next pointer, allowing for the creation of a singly linked list.

## [Shell](https://github.com/connormaas/tsh-shell)

#### Overview
A fully functional command-line shell that constantly interprets the user's inputs, running both built-in and user-driven commands (depending on the user's requests). This shell supports several built-in commands, process execution, and IO redirection.

#### Creating Processes
- A user may simply enter an executable that they wish to run, and if the file is reachable, the process will immediately begin taking place.
- If the '&' operator is added to the end of the line with a space in between, the process will run in the background and the user may still work with the shell, as well as run more executables.
- If the '&' operator is not added, the process will run in the foreground. This can be terminated or stopped using ctrl-c or ctrl-z respectively (mentioned in the next section).

#### Details
- Each job being executed has a process ID (PID) and a job ID (JID), which is unique to that process. This is used for identification and further manipulation of processes.
- Each child process is given its own process group, so when signals are forwarded to that process group, it will not interfere with the parent process. Specifically, this is necessary because otherwise these signals would be sent back to the original shell (parent), as well as every process created by it.
- When a background process begins, its JID and PID as well as the command line which invoked the execution are nicely displayed.
- The user typing 'ctrl-c' will terminate the execution of the foreground group.
- The user typing 'ctrl-z' will stop the execution of the foreground group.
- Children are properly reaped using a signal handler. The signal handler is able to deal with multiple terminated children despite receiving just one signal.
- During a foreground process, background processes that terminate do not become zombies, as signals are constantly handled concurrently with process execution.

#### Built-in Commands
- `quit`: terminates the shell.
- `jobs`: lists all background jobs.
- `bg job`: resumes job by sending it a SIGCONT signal, and then runs it in the background. The job argument can be either a PID or JID.
- `fg job`: command resumes job by sending it a SIGCONT signal, and then runs it in the foreground. The job argument can be either a PID or JID.

#### IO Redirection
- The operators '<' (input) and '>' (output) proceed the name of the file to which the input or output will be redirected.
- The default input and output are standard input and output respectively.
- Users may redirect output to any valid file which they have permission to write to.
- If a request is made to write to a file that doesn't exist, a new file will be created with the same name and that file will be written to instead.
- Output can also be redirected from built-in commands, such as the `jobs` command, which lists the background jobs as described in the previous section.
- Users may redirect input to any valid file which they have permission to read from. The file itself must exist or the process will be terminated.

#### Error Handling
- Error handling has been carefully constructed to ensure that invalid or extraneous inputs will not be able to break the shell.
- Please see the function descriptions in this file for more details on error handling specifics.

## [Cache Simulation](https://github.com/connormaas/cache-sim)

#### Background
[Cache Computing](https://en.wikipedia.org/wiki/Cache_(computing))

This program simulates a cache by first parsing the command line for specific arguments, including `s` (the number of bits for the set index), `b` (the number of bits for the block offset), `E` (the number of lines per set), `h` (help), and `v` (an optional debugging tool). It then parses a trace file, which contains load and save data with specific addresses (to read and write), as well as ask the number of bytes. The program prints the number of Hits, Misses, Evictions, Dirty Bytes, and Dirty Byte Evictions.

## [Mergesort (CPS)](https://github.com/connormaas/mergesort-cps)

#### Overview
This is a version of merge sort written in continuation-passing style using the language SMLNJ.

## [AI Checkers](https://github.com/connormaas/ai-checkers)

#### Overview
This program is a functional AI capable of effectively playing checkers, using the alpha-beta algorithm (alpha-beta pruning). This algorithm considers future moves by the opponent for each possible move that it could make on that turn. Although this algorithm is relatively simple, its functional implementation becomes more challenging.

#### Background
Inspired by [JVM](https://en.wikipedia.org/wiki/Java_virtual_machine). More information on the alpha-beta algorithm can be found [here](https://www.chessprogramming.org/Alpha-Beta).

## [LightsOut Solver (fast)](https://github.com/connormaas/lightsout-fast)

#### Overview
Lightsout is an electronic board game, consisting of a 5x5 board where the main goal is to turn off all the lights. However, when a light is turned off, the neighboring lights (left, right, up, down) are turned on. A more formal and in-depth description of the game can be found [here](https://en.wikipedia.org/wiki/Lights_Out_(game)). This program, written in C, produces a solution to any solvable board using the minimum number of moves. The solution is outputted step-by-step, including the coordinates of each required move, as well as a depiction of the board using “#” for lights that are on and “0” for lights that are off. Additionally, a hash table library was implemented, which stores previous board solutions, preventing repetitive computation.

## [c0vm](https://github.com/connormaas/c0vm)

#### Overview
c0vm is a virtual machine for C0, a C-like language developed at CMU. It's a stack-based machine that handles arithmetic operations and other instructions. The call stack consists of frames, each containing local variables, a local operand stack, and a return address. Operands are popped from an operand stack and their result is pushed back onto the stack. Functions defined in a source file are kept in a “function pool” and library functions are kept in a “native pool”.

## [Exp Language](https://github.com/connormaas/exp-language)

#### Overview
Exp is a simple programming language in C, capable of effectively performing mathematical calculations based on the standard order of operations using the left-associative principle. The operators are listed from lowest to highest precedence as follows (items with equivalent precedence are grouped together): `||`, `&&`, `(<, >, ==, !=)`, `<<`, `>>`, `(+, -)`, `(*, /, %)`, `**`. The description of each operator can be found in the image file within this repository. The language first implemented stack, queue, dictionary, and libraries to assist with parsing an entry for computation. The function “parse” takes in a queue of strings (numbers and operators) and a dictionary of precedences for each operator. The stack is used to store the current value and operators that have been popped from the queue as applicable. Once the stack is empty, the result is returned (assuming that the ordered combination of operators and values are valid for a complete operation).
