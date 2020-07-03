I'm Grant Glouser, and these are the little projects I've put on GitHub.

## [GIS Projects](/gis)

Examples of GIS work I've done.

## [InciWeb Viewer](https://github.com/gglouser/InciWebViewer)

This is a simple Android app for viewing the National Incidents RSS feed from
[InciWeb](https://inciweb.nwcg.gov/), which tracks wildfire incidents in the US.

(Kotlin)

## [Cut and Project Tiling](https://github.com/gglouser/cut-and-project-tiling)

[This interactive applet](https://gglouser.github.io/cut-and-project-tiling) draws tilings like the [Penrose tiling](https://en.wikipedia.org/wiki/Penrose_tiling) using a generalized multigrid method or the equivalent cut-and-project method. [Example gallery.](https://gglouser.github.io/cut-and-project-tiling/docs/gallery.html)

![example screenshot](/assets/images/tiling_screenshot.png)

(Javascript, Rust wasm)

## Advent of Code Visualizations

I love [Advent of Code](http://adventofcode.com)!
My solution repos for all years are available.

In 2018, I created simple interactive visualizations for a couple of the problems ([source code](https://github.com/gglouser/advent2018/tree/master/visualizations)). Both of these days had large input data with a recursive structure, and I wanted to see how they looked when drawn like an [L-system](https://en.wikipedia.org/wiki/L-system).

- Day 5: [Polymer Wreath](https://gglouser.github.io/advent2018/visualizations/day05.html) - *[task description](https://adventofcode.com/2018/day/5)*

  ![example polymer](https://i.imgur.com/IiZH05N.png)

- Day 8: [License Trees](https://gglouser.github.io/advent2018/visualizations/day08.html) - *[task description](https://adventofcode.com/2018/day/8)*

  ![example tree](https://i.imgur.com/Io6Xp0S.png)

Solution repos:

- [2015](https://github.com/gglouser/advent2015) - Haskell
- [2016](https://github.com/gglouser/advent2016) - PureScript
- [2017](https://github.com/gglouser/advent2017) - Rust
- [2018](https://github.com/gglouser/advent2018) - Rust

## [brainfree](https://github.com/gglouser/brainfree)

A [brainf*ck](https://en.wikipedia.org/wiki/Brainfuck) interpreter in Haskell. It represents the program using a free monad and offers multiple "back ends" to evaluate it. It also applies some optimizations to the bf program (arithmetic contraction, clear/copy/multiplication loops, and operation offsets).
