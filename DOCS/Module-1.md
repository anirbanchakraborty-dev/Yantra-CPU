# Module 1: What Is a CPU?

## The Von Neumann Architecture — Fetch / Decode / Execute

Let's start with the most fundamental question: **what actually is a computer?**

Strip away the screen, the keyboard, the apps, the operating system — at the very bottom of everything, a computer is a machine that does exactly one thing in an endless loop:

1. **Read** an instruction from memory
2. **Figure out** what that instruction means
3. **Do** it
4. **Go back to step 1**

That's it. Every program you've ever run — a web browser, a game, a compiler, an AI model — is ultimately just this loop executing billions of times per second. Each instruction is tiny and simple: "add these two numbers," "copy this value," "jump to a different instruction." But execute billions of them per second and you get the complexity we see.

This concept was formalized in 1945 by the mathematician **John von Neumann** (along with Eckert and Mauchly, though history credits von Neumann). Before this, "computers" were essentially hardwired — you physically rewired the machine for each different problem. Von Neumann's insight was radical: **store the instructions in the same memory as the data, and build a general-purpose machine that reads and executes those instructions automatically.**

This is the **stored-program concept**, and it's the foundation of nearly every computer ever built since, including the CPU we're about to design.

---

### The Three Core Components

A von Neumann machine has three essential parts:

**1. Memory** — A big array of numbered slots. Each slot holds a binary value (a fixed number of bits). The "number" of each slot is called its **address**. Memory stores two things: the **instructions** (the program) and the **data** (the values the program operates on). They coexist in the same memory space — this is the defining characteristic of von Neumann architecture.

Think of memory like a giant row of numbered mailboxes:

```text
Address:  0x0000  0x0004  0x0008  0x000C  0x0010  0x0014  ...
Content:  [inst]  [inst]  [inst]  [data]  [data]  [data]  ...
```

Each address points to one location. The CPU can **read** from any address (give me what's in mailbox 7) or **write** to any address (put this value in mailbox 12).

**2. The Central Processing Unit (CPU)** — The brain. This is what we're building. The CPU contains:

- A small amount of **fast internal storage** (registers) to hold the values it's currently working with
- An **arithmetic/logic unit (ALU)** that can do math and logical operations
- A **control unit** that orchestrates everything — it reads the instruction, figures out what to do, and sends signals to all the other parts telling them what to do this cycle
- A special register called the **Program Counter (PC)** that holds the address of the _next_ instruction to execute

**3. Input/Output (I/O)** — How the computer communicates with the outside world: keyboards, displays, disks, network. For Yantra-CPU, we'll largely ignore I/O and focus on the CPU and memory.

---

### The Fetch-Decode-Execute Cycle in Detail

Let me walk through exactly what happens each time the CPU goes around the loop. Suppose memory contains this simple program starting at address 0:

```text
Address 0x0000:  "Load the value from memory address 100 into register R1"
Address 0x0004:  "Load the value from memory address 104 into register R2"
Address 0x0008:  "Add R1 and R2, store result in R3"
Address 0x000C:  "Store R3 back to memory address 108"
```

(These are written in English for now — soon we'll design the actual binary encoding.)

The CPU starts with PC = 0x0000.

**Cycle 1 — FETCH:**
The CPU sends the value in the PC (0x0000) to memory as an address. Memory responds with whatever is stored at that location — the first instruction. The CPU stores this instruction in an internal register (called the **Instruction Register**, or IR). The PC is then incremented to point to the next instruction: PC becomes 0x0004.

**Cycle 1 — DECODE:**
The control unit examines the instruction now sitting in the IR. It determines: "This is a LOAD instruction. The source is memory address 100. The destination is register R1." It sets up internal signals accordingly — tell the memory "I want to read from address 100," tell the register file "get ready to write to R1."

**Cycle 1 — EXECUTE:**
The actual work happens. Memory is read at address 100, the value found there travels into the CPU, and it's written into register R1.

Now the cycle repeats. PC is 0x0004, so the second instruction is fetched, decoded (it's another load), and executed. Then the third (an ADD — the ALU adds R1 and R2, result goes to R3). Then the fourth (a STORE — R3's value is sent to memory address 108).

Each instruction is tiny. But the cycle repeats billions of times per second. A modern CPU runs at 3–5 GHz, meaning 3 to 5 _billion_ cycles per second.

---

### A Critical Limitation of Von Neumann

Notice something about the design: instructions and data share the **same memory** and the **same bus** (the pathway between CPU and memory). This means the CPU can either fetch an instruction OR access data in any given moment — not both simultaneously.

This is called the **von Neumann bottleneck**. The CPU is often waiting for memory, because the single pathway becomes a chokepoint. This limitation directly motivates the alternative we'll discuss next — the Harvard architecture — and it's one of the reasons caches, pipelines, and other tricks were invented.

---

#### Where Yantra-CPU Starts

Right now, Yantra-CPU exists only as a concept: a machine that will execute the fetch-decode-execute cycle. We haven't decided:

- How wide our data and instructions will be (8-bit? 16-bit? 32-bit?)
- How many registers we'll have
- What instructions we'll support
- Whether to use von Neumann or Harvard memory
- Whether to execute one instruction at a time or overlap them (pipelining)

Every single one of those will be a decision we make together, with clear reasoning for each choice.

---

## Harvard vs. Von Neumann Memory Architecture

Now that you understand the von Neumann bottleneck — instructions and data competing for the same memory pathway — let's look at the alternative.

The **Harvard architecture** (named after the Harvard Mark I computer from the 1940s) uses **separate memories for instructions and data**, with separate buses:

```text
Von Neumann:
┌──────┐     single bus      ┌──────────────────┐
│ CPU  │◄───────────────────►│ Memory           │
│      │                     │ (instructions +  │
│      │                     │  data mixed)     │
└──────┘                     └──────────────────┘

Harvard:
┌──────┐   instruction bus   ┌────────────────┐
│      │◄───────────────────►│ Instruction    │
│ CPU  │                     │ Memory (ROM)   │
│      │   data bus          ├────────────────┤
│      │◄───────────────────►│ Data           │
│      │                     │ Memory (RAM)   │
└──────┘                     └────────────────┘
```

**The advantage is huge:** the CPU can fetch the next instruction and read/write data **at the same time**, because they use different buses and different memories. No bottleneck. This is critical for pipelining — later we'll see that in a pipelined CPU, the fetch stage is reading the next instruction while the memory stage is simultaneously loading or storing data. If they shared one memory, they'd collide every cycle.

**The tradeoff:** two separate memories means more complexity, and the program can't easily modify itself (instructions are in a different memory than data). But for a simple CPU like ours, the benefits far outweigh the costs.

**What real-world processors do:** Most modern CPUs use a **modified Harvard architecture** — at the highest level, there's one unified memory (you can load both programs and data from the same disk, same RAM). But inside the CPU, the L1 cache is split into a separate **instruction cache (I-cache)** and **data cache (D-cache)**, giving you the Harvard advantage where it matters most — right next to the CPU.

---
