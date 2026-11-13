# Hello World

## A History of "Hello, World"

### Origins in BCPL and B (1967–1972)

The earliest known "hello, world" program appeared in Brian Kernighan's 1972 *A Tutorial Introduction to the Language B*. It was used to show how a B program could print a few characters to the terminal. Kernighan has also noted that a similar example, printing "hello" and "world" in pieces, was used in his work around BCPL, the language designed by Martin Richards in 1967 that influenced B.

### Popularization through C (1978)

The phrase became famous when it opened *The C Programming Language* (1978) by Brian Kernighan and Dennis Ritchie, commonly called "K&R". Its first example was:

```c
#include <stdio.h>

main()
{
    printf("hello, world\n");
}
```

Because K&R was the standard text for learning C, and C spread with Unix, the exercise became the conventional first program for newcomers to a language.

### Why it works

A Hello World program is small, but it confirms that the toolchain is working: the source can be written, compiled or interpreted, run, and its output seen. It also shows a language's minimal syntax and program structure at a glance.

### Spread across languages and platforms

From the 1980s onward, nearly every language, framework and platform documented its own Hello World:

- **Scripting languages** such as Perl, Python and Ruby reduced it to a single line, e.g. `print("Hello, World!")` in Python.
- **Object-oriented languages** such as C++ and Java (1995) showed more ceremony, with classes and `main` methods, making Hello World a quick way to compare language verbosity.
- **The web** brought JavaScript's `console.log` and `alert`, and HTML pages containing just the text "Hello, World!".
- **Graphical, mobile and embedded platforms** adopted variants: a window or label with the text on GUI toolkits and mobile apps, and a blinking LED on microcontrollers ("physical computing's Hello World").
- **Modern ecosystems** such as Go, Rust and cloud/serverless platforms continue the tradition, and many tutorials use it to verify installation and deployment.

### Variations and culture

The exact text varies: "hello, world" in K&R, "Hello, World!" in many later texts, and localized versions in other languages. The program has inspired collections of examples in hundreds of languages, including deliberately obscure ones, and "Hello, World" is a common title or reference in books, talks, and software projects.

### Legacy

More than fifty years after its first appearance, Hello World remains the standard first step in learning to program: a minimal, friendly check that a new tool works and that you are ready to build something more.
