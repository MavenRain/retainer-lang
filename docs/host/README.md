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
| K2 | U256 and Addr core types | Done |
| K3 to K5 | See the retainer-lang brief | Planned |


## U256 and Addr (slice K2)

The core has two more types next to `Nat` and `Flag`:

- `U256` is a 256-bit unsigned number. The evaluator keeps it in 4 limbs of 64 bits.
- `Addr` is a 160-bit address. The only operation on two addresses is the equality test `addrEq`.

`Nat` stays a 64-bit number for time and counts.

Literal forms:

- `1000u` is a U256 in decimal. `0xffu` is a U256 in hex, with 1 to 64 hex digits. A value above 2^256 - 1 is a parse refusal (`LEX_U256_RANGE`).
- `0x` and exactly 40 hex digits, with no suffix, is an Addr. Another digit count is a parse refusal (`LEX_HEX`).
- The printer writes a U256 as decimal digits and `u`, and an Addr as `0x` and 40 lowercase hex digits. The parser reads each printed form again.

Prelude names:

| Name | Type | Result |
|------|------|--------|
| `u256Add` | `U256 -> U256 -> U256` | The sum. An overflow past 2^256 - 1 traps. |
| `u256Sub` | `U256 -> U256 -> U256` | The difference. The result stops at 0. |
| `u256Mul` | `U256 -> U256 -> U256` | The product. An overflow traps. |
| `u256Div` | `U256 -> U256 -> U256` | The quotient, rounded down. Division by 0 traps. |
| `u256Le` | `U256 -> U256 -> Flag` | `flagYes` when the first is less than or equal to the second. |
| `u256Eq` | `U256 -> U256 -> Flag` | `flagYes` when the two are equal. |
| `u256Min` | `U256 -> U256 -> U256` | The smaller of the two. |
| `toU256` | `Nat -> U256` | The same number as a U256. This is the only conversion. |
| `addrEq` | `Addr -> Addr -> Flag` | `flagYes` when the two addresses are equal. |

The helpers are in `src/front/u256.c`. They use no `__int128` and no shift by 64 or more. `examples/words.lang` has the laws and the eval cases. The EVM output for these types is planned for slice K4: `langc build` still stops with `langc: PLANNED: ...`.

## Commands

- `make check`: the tcc build with `-Wall -Werror`, the clang syntax pass, `test/gate.sh` and `test/asm.sh`. `test/asm.sh` needs geth `evm` on PATH.
- `build/langc check FILE` and `build/langc eval FILE`: as in tcc-wasm.
- `build/langc build FILE` stops with `langc: PLANNED: ...` until slice K4.
- `build/asmtool`: the assembler self-test. `build/asmtool runtime|creation|abi` writes the test program or its ABI line.

The tool versions and the test path for K4 are in `docs/CAPABILITY.md`.
