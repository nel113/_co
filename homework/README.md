# The Hack Computer System: Technical Specification & Implementation Guide

### All copied from [Havivha](https://github.com/havivha/Nand2Tetris) without modification

---

**System Overview:** The Hack computer is a 16-bit, Von Neumann architecture machine constructed entirely from `Nand` gates. It runs code written in the high-level object-oriented language "Jack," which is compiled down to a stack-based Virtual Machine, then to Hack Assembly, and finally to binary machine code.

---

## **Layer 1: The Hardware Platform**

*The goal is to build a predictable, state-based machine from chaotic electrons.*

### [**1. Elementary Logic (The Atomic Foundation)**](https://github.com/nel113/_co/tree/0f2c58f10606407390f20972b98ec8d5a00c8e12/homework/1)

We begin with the axiom that `Nand(a, b)` produces `0` only if both `a` and `b` are `1`. From this, we derive the standard boolean set.

* **Multiplexers (Mux):** Crucial for routing data. `Mux(a, b, sel)` acts as a switch. If `sel=0`, output `a`; else output `b`.
* *Implementation:* .


* **Demultiplexers (DMux):** The reverse of Mux; sends a single input to one of two destinations.

### **2. Combinational Arithmetic (The ALU)**

The Arithmetic Logic Unit (ALU) is the computational brain. It is stateless—inputs flow through immediately to outputs.

* **The Hack ALU Design:** It takes two 16-bit inputs () and computes a 16-bit output based on 6 control bits:
* `zx`: Zero the x input.
* `nx`: Negate (bitwise Not) the x input.
* `zy`: Zero the y input.
* `ny`: Negate the y input.
* `f`: Function select (1 for Add, 0 for And).
* `no`: Negate the output.


* **Status Flags:** The ALU also outputs two 1-bit flags: `zr` (if output == 0) and `ng` (if output < 0), used later for conditional jumping.

### **3. Sequential Logic (Memory)**

To create "Time," we introduce the Clock.

* **The DFF (Data Flip-Flop):** .
* **Registers:** A Register is simply a DFF with a "Load" bit. If `load=1`, the DFF takes a new value; if `load=0`, it recycles its old output back into its input (remembering the state).
* **RAM:** An array of registers. We use `DMux` logic to route the "load" signal to the correct register address and `Mux` logic to route the correct register's output back to the bus.

### **4. The Hack CPU & Architecture**

The CPU integrates the ALU, Registers, and Program Counter.

* **Registers:**
* **D-Register:** Stores data (Data Register).
* **A-Register:** Stores data *or* addresses (Address Register).


* **The Instruction Cycle:**
1. **Fetch:** The ROM sends the instruction at `ROM[PC]` to the CPU.
2. **Decode:** The CPU looks at the instruction bits.
3. **Execute:** The ALU performs the calc, Memory reads/writes, and the PC updates (increment or jump).



### **5. Instruction Set Architecture (ISA)**

The interface between hardware and software. Hack has two instructions:

**A-Instruction (Addressing):** `@value`

* **Binary:** `0vvv vvvv vvvv vvvv`
* **Action:** Loads the 15-bit value `v` into the **A-Register**.

**C-Instruction (Computation):** `dest = comp ; jump`

* **Binary:** `111a cccc ccdd djjj`
* **Bits Breakdown:**
* `a`: Determines if the computation uses `A` register or Memory input `M` (RAM[A]).
* `cccccc`: Six bits controlling the ALU (matches the ALU design in Project 2).
* `ddd`: Destination bits (Store result in A? D? M? or combinations).
* `jjj`: Jump bits (Jump if result is <0? =0? >0? etc.).



---

## **Layer 2: The Virtualization Layer (Projects 6–8)**

*Bridging the gap between specific hardware constraints and abstract software needs.*

### **6. The Assembler**

Converts symbolic text (`@loop`, `D=D+1`) into binary strings.

* **Symbol Table Handling:**
1. **Predefined:** Initialize table with `R0..R15`, `SCREEN`, `KBD`.
2. **Pass 1 (Labels):** Scan for `(LABEL)`. Record the line number of the *next* instruction in the table. do not output code.
3. **Pass 2 (Variables):** Scan for instructions. If `@symbol` is found:
* If in table, replace with address.
* If not, assign next available RAM address (starting at 16) and add to table.





### **7. The Virtual Machine (Stack Arithmetic)**

The VM abstracts the CPU. Instead of registers, we use a **Stack**.

* **Push/Pop:** The VM translates `push segment index` into Hack Assembly.
* *Example:* `push constant 5` becomes:
```asm
@5
D=A
@SP
A=M
M=D   // *SP = 5
@SP
M=M+1 // SP++

```




* **Memory Segments:** The VM maps virtual segments (`local`, `argument`, `this`, `that`) to physical RAM addresses using pointers stored in `R1`–`R4`.

### **8. VM Control Flow (Function Calls)**

This enables recursion and modularity.

* **The Stack Frame:** When a function is called, the VM pushes a "state package" onto the stack so it can return later:
1. Return Address (ROM location).
2. Saved LCL (Local segment pointer).
3. Saved ARG (Argument segment pointer).
4. Saved THIS/THAT.


* **Bootstrapping:** The specific assembly code `Sys.init` that sets up the stack pointer (`SP=256`) and calls the main OS function.

---

## **Layer 3: The High-Level Software (Projects 9–12)**

*The user-facing application layer.*

### **9. Jack Language**

A Java-like, object-oriented language. It supports classes, methods, constructors, arrays, and static variables, but uses a simplified syntax to make compilation manageable.

### **10. Syntax Analysis (The Front-End)**

* **Tokenizer:** Breaks `while (x < 0)` into `<keyword>while`, `<symbol>(`, `<identifier>x`...
* **Parser:** Creates an XML structure based on the Jack Grammar.
* *Challenge:* Handling "lookahead". Knowing that `term` is an array entry `a[i]` vs just a variable `a` requires peeking at the next token.



### **11. Code Generation (The Back-End)**

Traverses the parse tree and emits VM code.

* **Expression Handling:** Converts infix math () to postfix stack logic (`push 3`, `push 5`, `add`).
* **Object Handling:**
* **Methods:** The first argument of any method call is implicitly `this` (pointer to the object instance).
* **Constructors:** Must call `Memory.alloc(size)` to find space on the heap for the new object.



### **12. The Operating System**

A library of 8 classes written in Jack, compiled, and linked with user programs.

* **Memory.jack:** Implements a "Free List" algorithm. The heap is a linked list of available memory segments. `alloc` finds a block of size , splits it, and returns the pointer.
* **Math.jack:** Since the Hack ALU only adds, `Math.multiply` is implemented using bit-shifting (checking the bits of the multiplier one by one).
* **Screen.jack:** Manages the 8K RAM map starting at 16384. To draw a pixel, it calculates the specific bit in a specific word of RAM and flips it.
