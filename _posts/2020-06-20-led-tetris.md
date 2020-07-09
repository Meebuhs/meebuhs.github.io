---
layout: post
title: LED Tetris
description: A self-playing tetris AI displayed on an array of LED matrices.
category: completed
permalink: /projects/led-tetris
urls:
    source: https://github.com/meebuhs/led-tetris
images:
    cover: /assets/images/led-tetris/cover.jpg
    top: /assets/images/led-tetris/demo.gif
mathjax: true
tags: [python, ai, heuristic, raspberry-pi, led-matrix]
---

This project started as a uni project for my intro to computer systems course in 2016. We were given the skeleton for a simple four button game of tetris written in C and were tasked with completing the code to have the game playable on an ATmega324A microcontroller, four push buttons, two 8x8 led matrices and a 2-digit seven segment display.

A couple years later, in 2018, I came across the 32x32 P6 SMD3528 LED matrices used in this project and thought it could be interesting to extend my old tetris project to a larger scale. I went about wiring up the panels, getting pcbs manufactured for the Raspberry Pi IO and
rewriting the C code in python, utilising [rpi-rgb-led-matrix](https://github.com/hzeller/rpi-rgb-led-matrix) to drive the displays. Soon I had tetris running on a Raspberry Pi with an array of six matrices, accepting user input over SSH.

It was immediately obvious that the game in that form was simply too big to be fun. It takes three times as long to drop a tetromino and ten times as long to complete a line, resulting in a game that was thirty times less enjoyable.

The next idea was to write an AI which could autoplay a much faster paced version of the game, which is exactly what I did.

## Implementation

There are two threads running concurrently, one which controls the display while the other performs the heuristic calculations.

### Display thread

The display data is stored as an array of RGB tuples, each representing a pixel on the matrix. A single pixel at position `(x, y)` can be set by changing the value of `board_display[BOARD_WIDTH * y + x]`. These are set as the position of theh tetrominoes change and the array is then converted to an image and pushed to the matrices to be displayed.

### Heuristic thread

When a new tetromino is added to the display, it is also added to a queue to be processed by the heuristic thread.

The tetromino is added to a `game` which simply defines where on screen it drops and the `BOARD_WIDTH / NUM_GAMES` columns within which it can be played. For each column in range and for each possible rotation of the given tetromino, the tetromino is dropped into place, any complete lines are cleared and the heuristic calculates the score for the resulting board.

The heuristic considers 6 metrics.

#### Complete Lines

<a href="/assets/images/led-tetris/complete-lines.png">
    <img src="/assets/images/blank.png" alt="Tetris heuristic complete lines" data-echo="/assets/images/led-tetris/complete-lines.png" width="400px" />
</a>

This is the most obvious metric and is simply the number of lines which would be completed by a given move. 
As this is the main goal of tetris, the heuristic tries to maximise this value.

$$ m_1 = \sum^n{r} \; \forall \; r = (1 << \text{board width}) - 1 $$

#### Created Empty Spaces

<a href="/assets/images/led-tetris/created-empty-spaces.png">
    <img src="/assets/images/blank.png" alt="Tetris heuristic created empty spaces" data-echo="/assets/images/led-tetris/created-empty-spaces.png" width="400px" />
</a>

An empty space is a void in the board which prevents a line from being cleared. This metric counts empty spaces which occur within, or one row below, the bounding box of a tetromino, which also have an occupied space above them. By minimising this value, the heuristic discourages moves which create new empty spaces.

$$ 
m_2 = \sum^n{pos_{x, y}} \text{ where for tetromino } t\\
pos_{x, y} \; \in \; \{pos_{x, y} \; | \; t_x \leq x \leq t_x + t_{width}, \; t_y \leq y \leq t_y + t_{height}, \; pos_{x, y} = 0\} \text{ and } \\
\exists \; pos_{x, z} \; | \; z < y, \; pos_{x, z} = 1 
$$

#### Nearby Empty Spaces

<a href="/assets/images/led-tetris/nearby-empty-spaces.png">
    <img src="/assets/images/blank.png" alt="Tetris heuristic nearby empty spaces" data-echo="/assets/images/led-tetris/nearby-empty-spaces.png" width="400px" />
</a>

This metric is similar to created empty spaces, but instead discourages tetrominoes from being placed in the same column as an empty space. The further an empty space is buried in the board, the harder it becomes to clear that line, and as such the heuristic minimises this value.

The emphasis placed on empty spaces by this heuristic comes from the difference in dimensions of the game. At 96x64 (compared to the standard 10x24) a missed space is much more punishing, requiring significantly more pieces to clear and providing proportionately fewer lines to do so.

$$ m_3 = \sum^n{pos_{x, y}} \text{ for } pos_{x, y} \; \in \; \{pos_{x, y} \; | \; x \in [t_x, t_x + t_{width}], \; pos_{x, y} = 0\} \text{ for tetromino } t $$

#### Average Column Height and Height Variation

<a href="/assets/images/led-tetris/line-height.png">
    <img src="/assets/images/blank.png" alt="Tetris heuristic line height and height variation" data-echo="/assets/images/led-tetris/line-height.png" width="400px" />
</a>

The game of tetris ends when a new tetromino can no longer be placed without colliding with the board. The heuristic discourages this by minimising the average height of the board and by promoting an even distribution of tetrominoes across all columns. The variation is measured by summing the difference in height of each pair of adjacent columns.

$$ m_4 = \frac{1}{n}\sum^n h_n \text{ where } h_n = \text{height of column } n $$

$$ m_5 = \sum_{n=1}^{n} |h_n - h_{n-1}| $$

#### Distance
The distance is the number of columns a tetromino has to be moved to reach the given position. The heuristic provides a small bonus if this value is <= 1. This is particularly for aesthetic reasons near the start of the game. Without this metric and on an empty board, the heuristic will result in an equivalent score for a given rotation in any position. This results in the pieces all rushing to the left side as the left most column is processed first, and as such, is stored as the highest score.

$$ 
m_6 = 
\begin{cases} 
    1,& \text{if} |x - t_x| \leq 1\\
    0,& \text{if} |x - t_x| > 1 
\end{cases}
$$

The score is then calculated as the sum of these metrics `m` multiplied by their factor `f`, which can be set in `Constants.py`.

$$ score = \sum^{n} f_n m_n $$

### Possible Improvements

- More advanced movement. Currently all movement is applied as soon as the heuristic resolves and can only place tetrominoes as they collide from directly above.
This could be improved by recognising that pieces can be moved horizontally into place once past an overhang. Even more advanced movement such as T or S/Z spins could also be added.
- More advanced application of the heuristic by looking ahead a few moves in advance.
- More thoughtful selection of heuristic factors via a more interesting process such as a genetic algorithm, reinforcement learning or particle swarm optimisation.

See the source code on [github](https://github.com/meebuhs/led-tetris)

## Hardware

<a href="/assets/images/led-tetris/hardware.png">
    <img src="/assets/images/blank.png" alt="Tetris led matrix raspberry pi setup" data-echo="/assets/images/led-tetris/hardware.png" />
</a>

This build uses 6 32x32 P6 SMD3528 LED matrices hooked up to a raspberry pi and an external power supply. 

The wiring is outlined very well by the [rpi-rgb-led-matrix documentation](https://github.com/hzeller/rpi-rgb-led-matrix/blob/master/wiring.md) so I won't repeat it here.

Note: If using the [adapter pcb](https://github.com/hzeller/rpi-rgb-led-matrix/tree/master/adapter) as shown in the image above, it is important to populate the chains from the top down. 
{:.notice}

