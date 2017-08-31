---
layout: project
title: Simple Brainfuck Interpreter
permalink: simple-brainfuck-interpreter/
---

Simple Brainfuck Interpreter is an interpreter for the esolang,
[Brainfuck](https://en.wikipedia.org/wiki/Brainfuck), created by Urban Müller.
<br>

### What is Brainfuck?

Like I already said, Brainfuck is an esoteric programming language, meaning it was
desgined to test the boundaries of computer science and also make programmers bang
their heads on their desks.

The language is noted for its _extreme minimalism_. Brainfuck contains an `instruction pointer`,
a `data pointer` and just `eight commands`. It is assumed that there are infinite data cells of
1 byte each. The breakdown of these eight commands is as follows:

| Command |                             What it does                               |
| :-----: | ---------------------------------------------------------------------- |
|         |                                                                        |
| `>`     | Increments the `data pointer` (to point to the next cell)              |
|         |                                                                        |
| `<`     | Decrements the `data pointer` (to point to previous cell)              |
|         |                                                                        |
| `+`     | Increments the value pointed to by `data pointer`                      |
|         |                                                                        |
| `-`     | Decrements the value pointed to by `data pointer`                      |
|         |                                                                        |
| `.`     | Print the value pointed to by `data pointer`                           |
|         |                                                                        |
| `,`     | Accept 1 byte of input and store it the cell pointer by `data pointer` |
|         |                                                                        |
| `[`     | Beginning of a `while` loop. If a `[` is encountered and the value     |
|         | pointed by the `data pointer` is 0, then the `instruction pointer` is  |
|         | moved forward to the instruction just after the macthing `]`           |
|         |                                                                        |
| `]`     | End of a `while` loop. If a `]` is encountered and the value pointed   |
|         | by the `data pointer` is non-zero, then the `instruction pointer` is   | 
|         | moved backwards to the instruction just after the macthing `[`         |

(_Table courtesy: Wikipedia_) 
<br>

The C/C++ equivalent of these commands are:

|  Command  |         C/C++ equivalent        |
| :-------: | ------------------------------- |
|           |                                 |
|  _Start_  | `char data[INFINITY] = { 0 };`  |
|           | `char* ip = data;`              |
| `>`       | `++ip;`                         |
| `<`       | `--ip;`                         |
| `+`       | `++(*ip);`                      |
| `-`       | `--(*ip);`                      |
| `.`       | `putchar(*ip);`                 |
| `,`       | `*ip = getchar();`              |
| `[`       | `while (*ip) {`                 |
| `]`       | `}`                             |

(_Table courtesy: Wikipedia_)

NOTE : Ideal value for `INFINITY` is `30000` in most brainfuck
interpreters.
<br>

### Hello World The Brainfuck Way

```
Anything not in the command set is ignored
This listing prints 'Hello World!' to the standard output
++++++++++[>+++++++>++++++++++>+++>+<<<<-]>++.>+.++++++
+..+++.>++.<<+++++++++++++++.>.+++.------.--------.>+.>.
```
<br>

### Source, Building & Running
 
You can find the source of Simple Brainfuck Interpreter
[here](https://github.com/TheIllusionistMirage/Simple-Brainfuck-Interpreter).

Build & run:

```
$ g++ *.cpp -o s-bfck-i
$ ./s-bfck-i TowersOfHanoi.bf
```

You can try the two brainfuck sources called `HelloWorld.bf` and
`TowersOfHanoi.bf` included in the repo.
<br>

### Screenshots

[![Simple-Brainfuck-Interpreter]({{site.url}}/resources/images/simple-brainfuck-interpreter-thumb.jpg "An exmaple run of Simple Brainfuck Interpreter running a brainfuck source to solve a nine disk tower of Hanoi problem")]({{site.baseurl}}/images/simple-brainfuck-interpreter-1.jpg)
Example run of the `TowersOfHanoi.bf` code for solving a nine disk towers of hanoi problem.
