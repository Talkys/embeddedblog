+++
title = "Developing a Chess Engine for the ESP32"
date = 2026-04-12T16:00:00Z
draft = false
+++

Since my last post, I’ve been deep in the weeds developing a chess bot specifically for the ESP32. This project turned out to be far more challenging than I anticipated, but the hurdles only made the process more interesting.

## Understanding the Engine: Bitboards vs. Mailboxes

My first step was researching how modern engines actually function. Generally, chess engines are implemented in one of two ways:

### 1. Bitboards (The Modern Standard)
Bitboards represent the board as a collection of 64-bit integers, where each bit corresponds to a square. This maps perfectly to 64-bit CPU architectures.
*   **Pros:** Extremely fast bitwise logic, supports SIMD operations, and allows for highly optimized move generation using pre-computed masks.
*   **Cons:** High memory overhead and computationally expensive on non-64-bit hardware.

### 2. Mailbox (The Classic Approach)
The Mailbox method uses a simple array to represent the board. A common variant is the **0x88 implementation**, which uses a 10x12 array (or an 8x16 layout) to make "off-board" detection significantly faster through bitmasking.

## Iteration 1: The Bitboard Prototype

I initially built a prototype using a C++ bitboard library by [Disservin](https://github.com/Disservin/chess-library), a Stockfish developer. The library was a single-header file, making it easy to integrate. However, it didn't include search or evaluation logic, which was perfect, as it allowed me to implement my own.

### The Bot Logic Flow
1.  **Initialize the Board:** Set up the state (either once for a persistent bot or per-move for an ephemeral one).
2.  **Generate Moves:** Calculate all legal moves for the current position.
3.  **Evaluate:** Score the board for each potential move.
4.  **Recurse:** Use search algorithms to look several moves ahead.
5.  **Finalize:** Play the best move found before hitting a timeout or depth limit.

To handle communication, I used the **UCI** protocol. While UCI is easiest to implement in languages with strong concurrency, I ran it in synchronous mode to keep things simple for the first version.

## Transitioning to the Mailbox Method

As I moved toward running the code on an MCU, I hit a snag. The ESP32 is a 32-bit processor; handling `uint64_t` bitboards isn't as efficient as it is on a desktop CPU. 

I decided to pivot to a **Mailbox engine**. I studied the works of **Harm Geert Muller**, a legend in the chess programming community, specifically his work on *micro-max*. My goal was to create a modular engine where the search and evaluation logic were separated from the move generation—a modularity lacking in many older, "entangled" engines.

## Engineering the Custom ESP32 Engine

I built the new engine from scratch using a modular C++ approach (though keeping the logic C-style for performance).

### Architecture
*   **Config Module:** Constants and helper functions.
*   **State Module:** Manages the engine's current state.
*   **Board/Search/Eval Modules:** Logic separated into distinct namespaces.
*   **I/O:** Replaced `iostream` with `cstdio` to reduce binary size and ensure compatibility with the ESP32 environment.

---

## Technical Deep Dive

### The Search: Looking Ahead
The engine uses **Iterative Deepening**. It searches 1 move deep, then 2, then 3, until the time limit is reached. At its core is **Negamax with Alpha-Beta Pruning**:
*   **Negamax:** A simplified version of Minimax that assumes a zero-sum game (where one player's gain is exactly the other's loss).
*   **Alpha-Beta Pruning:** This allows the bot to "prune" branches of the move tree that are mathematically guaranteed to be worse than previously analyzed moves, saving massive amounts of compute time.

### The Evaluation: Judging the Position
When the engine reaches the end of a branch, the `Eval` namespace scores the board based on:
*   **Material:** Simple piece counting (e.g., Queen = 9, Knight = 3).
*   **Positioning:** Piece-Square Tables (PST) penalize knights on the rim and reward kings for staying safe behind pawns.
*   **Pawn Structure:** We give bonuses to "Passed Pawns" that have a clear path to promotion.

### Efficiency & Optimizations
*   **Move Ordering:** We use **PV (Principal Variation)** and the **History Heuristic** to check the most promising moves first, which makes Alpha-Beta pruning much more effective.
*   **Quiescence Search:** To avoid "tactical blindness," the bot continues searching during captures/checks even after reaching its depth limit, ensuring it doesn't stop right in the middle of a piece trade.
*   **The "Flip" Trick:** To save memory, we use a single strategic table for White and "flip" it for Black moves.

## Conclusion
While the ESP32 might not be a supercomputer, with the right optimizations, it plays a surprisingly strong game of chess. You can find the full source code and follow the development on my GitHub. I've been able to play against it on depth 5, and on the ESP32 it was computing around 19-20k nodes per second. This was quite impressive for a device this small.

Here's a print of the bot playin on CoolTerm:
![Match](/images/esp_bot_coolterm.png)

And the respective board:
![Board](/images/esp_bot_board.png)

And this is a check-mate I got from the bot (To be fair I'm not an excelent player):
![Checkmate](/images/esp_bot_checkmate.png)

**Source Code:** [Talkys-Chess-Bot on GitHub](https://github.com/Talkys/Talkys-Chess-Bot/tree/main)