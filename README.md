# TetriScript

*A falling‑block, tape‑based esoteric programming language.*

---

## Table of Contents
1. [Introduction](#introduction)
2. [Official Tetrominoes & Colours](#official-tetrominoes--colours)
3. [Execution Model](#execution-model)
4. [Opcode Reference](#opcode-reference)
5. [Source‑File Format](#source-file-format)
6. [Sample Program – “+1 then Print”](#sample-program--1-then-print)
7. [Full “Hello World!” Script](#full-hello-world-script)
8. [Design Tips](#design-tips)
9. [Turing‑Completeness Sketch](#turing-completeness-sketch)
10. [License](#license)

---

## Introduction

TetriScript turns a classic **Tetris®** play‑field into the instruction stream of a tape machine:

* Every **tetromino you drop** is both a game move *and* an instruction waiting to
  happen.  
* Whenever a **row clears**, the colours of its **left‑most three cells** are read
  as an **opcode** that manipulates an infinite byte tape with a single read/write
  head.  
* Because the language exposes all tape‑machine primitives—pointer motion, cell
  mutation, byte I/O, and conditional jumps—it can simulate any Turing
  machine.

Programming, therefore, feels like playing speed‑Tetris while *engineering*
precise row clears in just the right order.

---

## Official Tetrominoes & Colours

| Shape | Nick‑name  | Canonical Colour | Abbrev |
|-------|------------|------------------|--------|
| **I** | Straight   | Cyan             | `C` |
| **J** | Left Gun   | Blue             | `B` |
| **L** | Right Gun  | Orange           | `O` |
| **O** | Square     | Yellow           | `Y` |
| **S** | Left Snake | Green            | `G` |
| **T** | Tee        | Purple           | `P` |
| **Z** | Right Snake| Red              | `R` |

Each tetromino *must* appear in its canonical colour.

---

## Execution Model

| Concept      | Details |
|--------------|---------|
| **Play‑field** | Width `W` (default = 10); unlimited height upward. |
| **Game tick** | 1. Spawn the declared tetromino (shape, colour, rotation, column).<br>2. It descends until it lands.<br>3. Every **full row clears**. |
| **Instruction fetch** | For **each cleared row**, read the three left‑most colours (`X Y Z`). Convert that triplet to an opcode (see below). Multiple simultaneous clears yield sequential opcodes. |
| **Tape** | Bi‑infinite 8‑bit cells, all zero. Pointer starts at cell 0. |
| **Program order** | Exactly the order tetrominoes are listed in the source file. |
| **Halting** | Interpreter stops when all drops are processed *or* a spawn collides with the ceiling. |

---

## Opcode Reference

Triplet order is **left → right** (column 0, 1, 2).

| Triplet | Mnemonic | Effect on Tape |
|---------|----------|----------------|
| `R O G` | **PTR+** | Move head one cell right |
| `R O B` | **PTR‑** | Move head one cell left |
| `R G Y` | **INC**  | Add 1 (mod 256) to current cell |
| `R G C` | **DEC**  | Subtract 1 (mod 256) from current cell |
| `O G B` | **OUT**  | Output current cell as ASCII |
| `O G P` | **IN**   | Read one byte of input into current cell |
| `G B R` | **LOOP START** | If cell = 0, jump forward to matching **LOOP END** |
| `B R G` | **LOOP END**   | If cell ≠ 0, jump back to matching **LOOP START** |
| `Y Y Y` | **NOP**  | Row ignored |
| *other* | **ERROR**| Interpreter aborts |

---

## Source‑File Format

One tetromino drop per line:

`<ColourLetter> <Shape> <Rotation> <Column>   ; optional comment`


* **ColourLetter** ∈ `C B O Y G P R`  
* **Shape** ∈ `I J L O S T Z`  
* **Rotation** ∈ `0 1 2 3`  (clockwise quarter‑turns)  
* **Column** ∈ `0 … W-1`  (location of the **left‑most** block)  

Example:

G S 3 4 ; green S‑piece, rotated 270°, left edge at column 4


Lines beginning with `;` are comments.

---

## Sample Program – “+1 then Print”

Stores `1` in cell 0, then outputs it (ASCII SOH).

```tsc
; Pre‑fill columns 3‑9 so row 0 is 70 % full
C I 0 3
C I 0 4
C I 0 5
C I 0 6
C I 0 7
C I 0 8
C I 0 9

; --- Row 0:  R G Y  →  INC ---
R Z 0 0    ; red Z
G S 0 1    ; green S
Y O 0 2    ; yellow O  → row clears (INC)

; Prepare next row in the same way…
C I 0 3
C I 0 4
C I 0 5
C I 0 6
C I 0 7
C I 0 8
C I 0 9

; --- Row 1:  O G B  →  OUT ---
O L 0 0
G S 0 1
B J 0 2    ; prints ASCII 1
```

## Full “Hello World!” Script

Below is the exact sequence of row‑clears required to print Hello World!\n.
Deliver these clears in order. Any legal arrangement of tetromino drops that accomplishes this sequence is valid.

```tsc
; ─── bootstrap cell0 = 10 ──────────────────────────────────────────────
R G Y   ; INC  ×10

; ─── build loop counter, copy & multiply ───────────────────────────────
G B R   ; LOOP START
R O G   ; PTR+
R G Y   ; INC ×7
R O G   ; PTR+
R G Y   ; INC ×10
R O G   ; PTR+
R G Y   ; INC ×3
R O G   ; PTR+
R G Y   ; INC
R O B   ; PTR- ×4
R G C   ; DEC
B R G   ; LOOP END

; ─── print “H” ─────────────────────────────────────────────────────────
R O G   ; PTR+
R G Y   ; INC ×2
O G B   ; OUT

; ─── print “e” ─────────────────────────────────────────────────────────
R O G
R G Y   ; INC
O G B   ; OUT

; ─── print “l” “l” ─────────────────────────────────────────────────────
R G Y   ; INC ×7
O G B   ; OUT
O G B   ; OUT

; ─── print “o” ─────────────────────────────────────────────────────────
R G Y   ; INC ×3
O G B   ; OUT

; ─── print space ──────────────────────────────────────────────────────
R O G   ; PTR+
R G Y   ; INC ×2
O G B   ; OUT

; ─── print “W” ─────────────────────────────────────────────────────────
R O B   ; PTR- ×2
R G Y   ; INC ×15
O G B   ; OUT

; ─── print “o” ─────────────────────────────────────────────────────────
R O G   ; PTR+
O G B   ; OUT

; ─── print “r” ─────────────────────────────────────────────────────────
R G Y   ; INC ×3
O G B   ; OUT

; ─── print “l” ─────────────────────────────────────────────────────────
R G C   ; DEC ×6
O G B   ; OUT

; ─── print “d” ─────────────────────────────────────────────────────────
R G C   ; DEC ×8
O G B   ; OUT

; ─── print “!” ─────────────────────────────────────────────────────────
R O G   ; PTR+
R G Y   ; INC
O G B   ; OUT

; ─── print newline ────────────────────────────────────────────────────
R O G   ; PTR+
O G B   ; OUT
```

### Reading the script
* **Triplet** (`R G Y`, `O G B`, …) → colours found in columns 0-2 of a cleared row.  
* **Comment** shows the mnemonic and, when helpful, how many times that triplet
  should repeat (`× n`).  
* Ensure each loop start (`G B R`) has a matching loop end (`B R G`) in execution
  order—treat them like coloured parentheses.

---

### Design Tips
* **Tower method** – Build three-column “opcode towers” (e.g., permanent
  `R G Y` in columns 0-2) and use columns 3-9 to pad the rest of each row.  
* **Loop balancing** – Place `LOOP START` / `LOOP END` rows so they clear in
  properly nested order.  
* **Colour hygiene** – Only columns 0-2 are decoded; columns 3-9 can use any
  colour without affecting opcodes.  
* **Debug visually** – Bright colours for pointer moves, muted ones for data
  changes help you spot pointer drift at a glance.

---

### Turing-Completeness Sketch
TetriScript provides:
1. **Pointer motion** (`PTR+`, `PTR-`) for unbounded head movement.  
2. **Cell mutation** (`INC`, `DEC`) for arbitrarily changing tape values.  
3. **Conditional jumps** (`LOOP START`, `LOOP END`) forming potentially
   unbounded loops.  
4. **Byte I/O** (`IN`, `OUT`) for interaction but not required for computation.

These are exactly the primitives needed to emulate a universal Turing machine,
so—given enough rows and time—TetriScript can compute any computable function.
