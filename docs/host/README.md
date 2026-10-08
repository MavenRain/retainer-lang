# tcc-evm-contract

The TinyCC host kit of retainer-lang. It compiles the contract language to EVM bytecode.

## Origin

- The front end, `src/main.c`, `src/ir.h`, `src/target.h`, `gen/`, `domain/`, `examples/` and `test/gate.sh` are copies of `hosts/tcc-wasm`. The SHA-256 of each source file is in `lang-template-work/tcc-evm-contract-snapshot.txt`, with a list of the changes.
- `src/asm.c` and `src/asm.h` come from the escrowc assembler (escrow-lang `src/evm.c`). The labels are numbers, not a fixed enum.
- `src/keccak.c` and `src/keccak.h` are copies of the escrowc files. Only the names are changed.
- `src/evm.c` is the EVM target. It writes the ABI selectors. The bytecode output comes in slice K4.

## Slices

| Slice | Content | Status |
|-------|---------|--------|
| K1 | Kit skeleton, assembler, keccak, ABI selectors, tool probe | Done |
| K2 to K5 | See the retainer-lang brief | Planned |

## Commands

- `make check`: the tcc build with `-Wall -Werror`, the clang syntax pass, `test/gate.sh` and `test/asm.sh`. `test/asm.sh` needs geth `evm` on PATH.
- `build/langc check FILE` and `build/langc eval FILE`: as in tcc-wasm.
- `build/langc build FILE` stops with `langc: PLANNED: ...` until slice K4.
- `build/asmtool`: the assembler self-test. `build/asmtool runtime|creation|abi` writes the test program or its ABI line.

The tool versions and the test path for K4 are in `docs/CAPABILITY.md`.
