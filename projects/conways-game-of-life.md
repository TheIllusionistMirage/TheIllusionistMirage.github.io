---
layout: project
title: Conway's Game of Life
permalink: conways-game-of-life/
---

Game of Life is a zero-player infinite cellular automation 2D game created by mathematician John Conway.
<br>

### Screenshots

[![Conways-Game-of-Life-1]({{site.url}}/resources/images/conways-game-of-life-1-thumb.jpg)]({{site.baseurl}}/images/conways-game-of-life-1.jpg)

[![Conways-Game-of-Life-1]({{site.url}}/resources/images/conways-game-of-life-2-thumb.jpg)]({{site.baseurl}}/images/conways-game-of-life-2.jpg)
<br>

### Rules

Since Game of Life is a zero-player game, no player input of any kind is involved. The game world of Game
of Life consists of an infinite 2D grid of squares, called ***cells***. Each of these cells may be ***populated***
or ***unpopulated*** (or, ***dead*** or ***alive***). And during every ***generation***, whether or not a given cell will
be populated or unpopulated will be determined by the particular cell's neighbours as follows:

* A neighbour of the `(i,j)-th` cell are the eight cells adjacent to it horizontally, vertically and diagonally,
  i.e., the cells `(i-1,j-1), (i,j-1), (i+1,j-1), (i-1,j), (i+1,j), (i-1,j+1), (i,j+1), (i+1,j+1)`.
* An unpopulated cell becomes populated if it has ***exactly*** **three** populated neighbours.
* A populated cell with **two** or **three** populated neighbours lives on to the next generation.
* A populated cell with **one** or **zero** populated neighours dies in this generation.
* A populated cell with **four** or more populated neighbours dies in this generation.

The initial value of the world grid is called the ***seed*** of the game. The above rules are applied infinitely.
You can consult the [wikipedia entry](https://en.wikipedia.org/wiki/Conway's_Game_of_Life) for Conway's Game of
Life to get a deeper insight about popular seeds and other trivia.

***NOTE:***

This implementation of Game of Life implements ***world warping***, which may or may not be done in other implementations.
(world warping means the top and bottom rows of the world grid are adjacent, and the left and right columns are
adjacent).
<br>

### Source, Building & Running

You can find the source [here](https://github.com/TheIllusionistMirage/Game-Of-Life).

You can build two versions of Game of Life - one a CLI version and one GUI version.
By default the CMake build script supplied with the code builds the CLI version.
To build the GUI version, just set the preprocessor `RENDERER_SFML`.

```
$ git clone https://github.com/TheIllusionistMirage/Game-Of-Life.git
$ cd Game-Of-Life 
$ mkdir build && cd build
$ cmake ..
$ make
```

Then invoke either the CLI version or GUI version.
