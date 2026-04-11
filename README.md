# Yantra-CPU

A 32-bit CPU designed and built from the ground up — from number systems and transistors to a working processor capable of executing real programs.

Yantra-CPU is an educational, design-first project: every architectural decision is made deliberately, with clear reasoning, and documented alongside the implementation. The goal is not just to produce a working CPU, but to understand *why* each piece exists the way it does.

## Project Status

Early design phase. The ISA, datapath, and microarchitecture are being worked out module by module in [DOCS/](DOCS/).

## Documentation

The project is structured as a series of modules that build up the CPU incrementally:

- [Module 0 — The Fundamentals](DOCS/Module-0.md): Number systems, binary, two's complement, sign extension, and overflow.
- [Module 1 — What Is a CPU?](DOCS/Module-1.md): Von Neumann architecture, the fetch-decode-execute cycle, and Harvard vs. Von Neumann.

More modules will follow as the design progresses through the ISA, ALU, register file, control unit, memory system, and pipelining.

## Planned Highlights

- 32-bit word size, two's complement arithmetic
- Custom instruction set architecture
- ALU with add/subtract sharing a single adder (via two's complement)
- Sign-extended and zero-extended immediates
- Harvard-style instruction and data memory
- Designed for eventual pipelining

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

## Author

Anirban Chakraborty ([@anirbanchakraborty-dev](https://github.com/anirbanchakraborty-dev))
