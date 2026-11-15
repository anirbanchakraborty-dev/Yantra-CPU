<div align="center">

<a href="DOCS/Module-0.md"><picture><source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/anirbanchakraborty-dev/anirbanchakraborty-dev/output/repo-yantra-cpu-banner-dark.svg"><img alt="Yantra-CPU, a 32-bit CPU designed from first principles. Beside the description, the binary number 10110011 from Module 0 is weighed bit by bit, its partial sums climb to 179, and a row of dots shows how many of the planned modules are written." src="https://raw.githubusercontent.com/anirbanchakraborty-dev/anirbanchakraborty-dev/output/repo-yantra-cpu-banner-light.svg" width="100%"></picture></a>

[Module 0](DOCS/Module-0.md) · [Module 1](DOCS/Module-1.md) · [Roadmap](#roadmap) · [Planned design](#planned-design) · [Licence](#licence)

</div>

Yantra-CPU is a 32-bit processor designed from the ground up, starting at number systems and working toward a machine that runs real programs. It is an educational, design-first project: every architectural decision is made deliberately, written down with the reasoning behind it, and documented beside the implementation. The aim is not only to end with a working CPU but to understand why each piece takes the shape it does.

The work is in its early design phase. Nothing is built in hardware yet; the instruction set, the datapath and the microarchitecture are being worked out one module at a time in [`DOCS/`](DOCS/), and each module leans on the ones before it.

```mermaid
flowchart LR
  m0["The fundamentals"] --> m1["What is a CPU?"]
  m1 --> isa["The instruction set"]
  isa --> alu["The ALU"]
  alu --> rf["The register file"]
  rf --> cu["The control unit"]
  cu --> mem["The memory system"]
  mem --> pipe["Pipelining"]
  classDef written fill:#2e7d5b,stroke:#2e7d5b,color:#ffffff
  classDef planned fill:none,stroke:#8a8f98,stroke-dasharray:4 3
  class m0,m1 written
  class isa,alu,rf,cu,mem,pipe planned
```

## Roadmap

The modules run in the order below, which is also the order the diagram above follows. A filled box there is a module already written.

| Topic | Document | Status |
| --- | --- | --- |
| The fundamentals | [Module 0](DOCS/Module-0.md) | Written |
| What is a CPU? | [Module 1](DOCS/Module-1.md) | Written |
| The instruction set | not yet started | Planned |
| The ALU | not yet started | Planned |
| The register file | not yet started | Planned |
| The control unit | not yet started | Planned |
| The memory system | not yet started | Planned |
| Pipelining | not yet started | Planned |

## What the written modules cover

**Module 0, The Fundamentals,** starts from the decimal system everyone already uses and shows that binary is the same positional idea with two symbols instead of ten. Its worked example reads `10110011` as $1 \cdot 128 + 0 \cdot 64 + 1 \cdot 32 + 1 \cdot 16 + 0 \cdot 8 + 0 \cdot 4 + 1 \cdot 2 + 1 \cdot 1 = 179$. From there it covers converting between decimal and binary, hexadecimal as a shorthand, the difference between unsigned and signed numbers, sign-magnitude and why it falls short, two's complement and the hardware argument for it, sign extension, overflow, and a brief look at fixed-point numbers.

**Module 1, What Is a CPU?,** introduces the von Neumann machine: one memory holding both the program and its data, a processor that repeats the fetch, decode and execute cycle, and the single shared pathway between them that becomes the von Neumann bottleneck. It then sets the Harvard architecture beside it, with separate memories and buses for instructions and data, so that an instruction can be fetched while data is read or written in the same moment.

## Planned design

These are the design's planned highlights. The modules still to come work each one out in detail.

- A 32-bit word, with signed values in two's complement.
- A custom instruction set, designed in the next module.
- An ALU in which addition and subtraction share one adder, since subtracting $b$ is adding its two's complement $\bar{b} + 1$.
- Immediates that can be sign-extended or zero-extended to the full 32-bit word.
- Separate instruction and data memories in the Harvard style.
- A datapath laid out from the start so that it can be pipelined later.

## Licence

Released under the MIT Licence. See [`LICENSE`](LICENSE).

## Author

Dr. Anirban Chakraborty ([@anirbanchakraborty-dev](https://github.com/anirbanchakraborty-dev)), [dranirbanchakraborty.com](https://dranirbanchakraborty.com)
