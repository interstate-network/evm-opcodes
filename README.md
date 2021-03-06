# EVM Opcodes
An updated version of the EVM reference page at [ethervm.io](https://ethervm.io/).
Also drawn from the [Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf), the [Jello Paper](https://jellopaper.org/evm.html), and the [geth](https://github.com/ethereum/go-ethereum) implementation.
This is intended to be an accessible reference, but it is not particularly rigorous.
If you want to be certain of correctness and aware of every edge case, I would suggest using the Jello Paper or a client implementation.

| Hex | Name        | Gas | Stack In            | Stack Out             | Mem / Storage Out     | Notes |
| :---: | :---      | :---: | :---              | :---                  | :---                  | :--- |
|   |               |   | top, bottom      | top, bottom           ||
00  | STOP          | 0 |                       |                       || halt execution
01  | ADD           | 3 | a, b                  | a + b	                || (u)int256 addition modulo 2\*\*256
02  | MUL           | 5 | a, b                  | a \* b                || (u)int256 multiplication modulo 2\*\*256
03  | SUB           | 3 | a, b                  | a - b	                || (u)int256 addition modulo 2\*\*256
04  | DIV           | 5 | a, b                  | a // b                || uint256 division
05  | SDIV          | 5 | a, b                  | a // b                || int256 division
06  | MOD           | 5 | a, b                  | a % b                 || uint256 modulus
07  | SMOD          | 5 | a, b                  | a % b                 || int256 modulus
08  | ADDMOD        | 8 | a, b, N               | (a + b) % N           || (u)int256 addition modulo N
09  | MULMOD        | 8 | a, b, N               | (a * b) % N           || (u)int256 multiplication modulo N
0A  | EXP           |[A3](#a3-exp)| a, b                  | a \*\* b              || uint256 exponentiation modulo 2\*\*256
0B  | SIGNEXTEND    | 5 | b, x                  | SIGNEXTEND(x, b)      || sign extend `x` from `(b + 1) * 8` bits to 256 bits.
0C-0F| *invalid*
10  | LT            | 3 | a, b                  | a < b                 || uint256 less-than
11  | GT            | 3 | a, b                  | a > b                 || uint256 greater-than
12  | SLT           | 3 | a, b                  | a < b                 || int256 less-than
13  | SGT           | 3 | a, b                  | a > b                 || int256 greater-than
14  | EQ            | 3 | a, b                  | a == b                || (u)int256 equality
15  | ISZERO        | 3 | a                     | a == 0                || (u)int256 iszero
16  | AND           | 3 | a, b                  | a && b                || bitwise AND
17  | OR            | 3 | a, b                  | a \|\| b              || bitwise OR
18  | XOR           | 3 | a, b                  | a ^ b                 || bitwise XOR
19  | NOT           | 3 | a                     | ~a                    || bitwise NOT
1A  | BYTE          | 3 | i, x                  | (x >> (248 - i * 8)) && 0xFF || `i`th byte of (u)int256 `x`, from the left
1B  | SHL           | 3 | shift, val            | val << shift          || shift left
1C  | SHR           | 3 | shift, val            | val >> shift          || logical shift right
1D  | SAR           | 3 | shift, val            | val >> shift          || arithmetic shift right
1E-1F| *invalid*
20  | SHA3          |[A4](#a4-sha3)| ost, len           | keccak256(mem[ost:ost+len]) || keccak256
21-2F| *invalid*
30  | ADDRESS       | 2 |                       | address(this)         || address of executing contract
31  | BALANCE       |700| addr                  | addr.balance          || balance, in wei
32  | ORIGIN        | 2 |                       | tx.origin             || address that originated the tx
33  | CALLER        | 2 |                       | msg.sender            || address of msg sender
34  | CALLVALUE     | 2 |                       | msg.value             || msg value, in wei
35  | CALLDATALOAD  | 3 | idx                   | msg.data[idx:idx+32]  || read word from msg data at index `idx`
36  | CALLDATASIZE  | 2 |                       | len(msg.data)         || length of msg data, in bytes
37  | CALLDATACOPY  |[A5](#a5-copy-operations)| dstOst, ost, len      |                       | mem[dstOst:dstOst+len= msg.data[ost:ost+len | copy msg data
38  | CODESIZE      | 2 |                       | len(this.code)        || length of executing contract's code, in bytes
39  | CODECOPY      |[A5](#a5-copy-operations)| dstOst, ost, len      |                       | mem[dstOst:dstOst+len] = this.code[ost:ost+len] | copy executing contract's bytecode
3A  | GASPRICE      | 2 |                       | tx.gasprice           || gas price of tx, in wei per unit gas
3B  | EXTCODESIZE   |700| addr                  | len(addr.code)        || size of code at addr, in bytes
3C  | EXTCODECOPY   |[A5](#a5-copy-operations)|addr, dstOst, ost, len |                       | mem[dstOst:dstOst+len] = addr.code[ost:ost+len] | copy code from `addr`
3D  |RETURNDATASIZE | 2 |                       | size                  || size of returned data from last external call, in bytes
3E  |RETURNDATACOPY |[A5](#a5-copy-operations)| dstOst, ost, len      |                       | mem[dstOst:dstOst+len] = returndata[ost:ost+len] | copy returned data from last external call
3F  | EXTCODEHASH   |700| addr                  | hash                  || hash = addr.exists ? keccak256(addr.code) : 0
40  | BLOCKHASH     |20 | blockNum              |block.blockHash(blockNum)||
41  | COINBASE      | 2 |                       | block.coinbase        || address of miner of current block
42  | TIMESTAMP     | 2 |                       | block.timestamp       || timestamp of current block
43  | NUMBER        | 2 |                       | block.number          || number of current block
44  | DIFFICULTY    | 2 |                       | block.difficulty      || difficulty of current block
45  | GASLIMIT      | 2 |                       | block.gaslimit        || gas limit of current block
46  | CHAINID       | 2 |                       | chain\_id             || push current [chain id](https://eips.ethereum.org/EIPS/eip-155) onto stack
47  | SELFBALANCE   | 5 |                       | address(this).balance || balance of executing contract, in wei
48-4F| *invalid*
50  | POP           | 2 | \_anon                |                       || remove item from top of stack and discard it
51  | MLOAD         |3[\*](#a2-memory-expansion)| ost                   | mem[ost:ost+32]       || read word from memory at offset `ost`
52  | MSTORE        |3[\*](#a2-memory-expansion)| ost, val              |                       | mem[ost:ost+32] = val | write a word to memory
53  | MSTORE8       |3[\*](#a2-memory-expansion)| ost, val              |                       | mem[ost] = val && 0xFF | write a single byte to memory
54  | SLOAD         |800| key                   | storage[key]          || read word from storage
55  | SSTORE        |[A6](#a6-sstore)   | key, val              |                       | storage[key] = val | write word to storage
56  | JUMP          | 8 | dst                   |                       || `$pc = dst`
57  | JUMPI         |10 | dst, condition        |                       || `$pc = condition ? dst : $pc + 1`
58  | PC            | 2 |                       | $pc                   || program counter
59  | MSIZE         | 2 |                       | len[mem]              || size of memory in current execution context, in bytes
5A  | GAS           | 2 |                       | gasRemaining          ||
5B  | JUMPDEST      | 1 |                       |                       || mark valid jump destination
5C-5F| *invalid*
60  | PUSH1         | 3 |                       | uint8                 || push 1-byte value onto stack
61  | PUSH2         | 3 |                       | uint16                || push 2-byte value onto stack
62  | PUSH3         | 3 |                       | uint24                || push 3-byte value onto stack
63  | PUSH4         | 3 |                       | uint32                || push 4-byte value onto stack
64  | PUSH5         | 3 |                       | uint40                || push 5-byte value onto stack
65  | PUSH6         | 3 |                       | uint48                || push 6-byte value onto stack
66  | PUSH7         | 3 |                       | uint56                || push 7-byte value onto stack
67  | PUSH8         | 3 |                       | uint64                || push 8-byte value onto stack
68  | PUSH9         | 3 |                       | uint72                || push 9-byte value onto stack
69  | PUSH10        | 3 |                       | uint80                || push 10-byte value onto stack
6A  | PUSH11        | 3 |                       | uint88                || push 11-byte value onto stack
6B  | PUSH12        | 3 |                       | uint96                || push 12-byte value onto stack
6C  | PUSH13        | 3 |                       | uint104               || push 13-byte value onto stack
6D  | PUSH14        | 3 |                       | uint112               || push 14-byte value onto stack
6E  | PUSH15        | 3 |                       | uint120               || push 15-byte value onto stack
6F  | PUSH16        | 3 |                       | uint128               || push 16-byte value onto stack
70  | PUSH17        | 3 |                       | uint136               || push 17-byte value onto stack
71  | PUSH18        | 3 |                       | uint144               || push 18-byte value onto stack
72  | PUSH19        | 3 |                       | uint152               || push 19-byte value onto stack
73  | PUSH20        | 3 |                       | uint160               || push 20-byte value onto stack
74  | PUSH21        | 3 |                       | uint168               || push 21-byte value onto stack
75  | PUSH22        | 3 |                       | uint176               || push 22-byte value onto stack
76  | PUSH23        | 3 |                       | uint184               || push 23-byte value onto stack
77  | PUSH24        | 3 |                       | uint192               || push 24-byte value onto stack
78  | PUSH25        | 3 |                       | uint200               || push 25-byte value onto stack
79  | PUSH26        | 3 |                       | uint208               || push 26-byte value onto stack
7A  | PUSH27        | 3 |                       | uint216               || push 27-byte value onto stack
7B  | PUSH28        | 3 |                       | uint224               || push 28-byte value onto stack
7C  | PUSH29        | 3 |                       | uint232               || push 29-byte value onto stack
7D  | PUSH30        | 3 |                       | uint240               || push 30-byte value onto stack
7E  | PUSH31        | 3 |                       | uint248               || push 31-byte value onto stack
7F  | PUSH32        | 3 |                       | uint256               || push 32-byte value onto stack
80  | DUP1          | 3 | val                   | val, val              || clone 1st value on stack
81  | DUP2          | 3 | \_, val               | val, \_, val          || clone 2nd value on stack
82  | DUP3          | 3 | \_, \_, val           | val, \_, \_, val      || clone 3rd value on stack
83  | DUP4          | 3 | \_, \_, \_, val       | val, \_, \_, \_, val  || clone 4th value on stack
84  | DUP5          | 3 | ..., val              | val, ..., val         || clone 5th value on stack
85  | DUP6          | 3 | ..., val              | val, ..., val         || clone 6th value on stack
86  | DUP7          | 3 | ..., val              | val, ..., val         || clone 7th value on stack
87  | DUP8          | 3 | ..., val              | val, ..., val         || clone 8th value on stack
88  | DUP9          | 3 | ..., val              | val, ..., val         || clone 9th value on stack
89  | DUP10         | 3 | ..., val              | val, ..., val         || clone 10th value on stack
8A  | DUP11         | 3 | ..., val              | val, ..., val         || clone 11th value on stack
8B  | DUP12         | 3 | ..., val              | val, ..., val         || clone 12th value on stack
8C  | DUP13         | 3 | ..., val              | val, ..., val         || clone 13th value on stack
8D  | DUP14         | 3 | ..., val              | val, ..., val         || clone 14th value on stack
8E  | DUP15         | 3 | ..., val              | val, ..., val         || clone 15th value on stack
8F  | DUP16         | 3 | ..., val              | val, ..., val         || clone 16th value on stack
90  | SWAP1         | 3 | a, b                  | b, a                  ||
91  | SWAP2         | 3 | a, \_, b              | b, \_, a              ||
92  | SWAP3         | 3 | a, \_, \_, b          | b, \_, \_, a          ||
93  | SWAP4         | 3 | a, \_, \_, \_, b      | b, \_, \_, \_, a      ||
94  | SWAP5         | 3 | a, ..., b             | b, ..., a             ||
95  | SWAP6         | 3 | a, ..., b             | b, ..., a             ||
96  | SWAP7         | 3 | a, ..., b             | b, ..., a             ||
97  | SWAP8         | 3 | a, ..., b             | b, ..., a             ||
98  | SWAP9         | 3 | a, ..., b             | b, ..., a             ||
99  | SWAP10        | 3 | a, ..., b             | b, ..., a             ||
9A  | SWAP11        | 3 | a, ..., b             | b, ..., a             ||
9B  | SWAP12        | 3 | a, ..., b             | b, ..., a             ||
9C  | SWAP13        | 3 | a, ..., b             | b, ..., a             ||
9D  | SWAP14        | 3 | a, ..., b             | b, ..., a             ||
9E  | SWAP15        | 3 | a, ..., b             | b, ..., a             ||
9F  | SWAP16        | 3 | a, ..., b             | b, ..., a             ||
A0  | LOG0          |[A7](#a7-log-operations)| ost, len              |                       || LOG0(memory[ost:ost+len])
A1  | LOG1          |[A7](#a7-log-operations)| ost, len, topic0      |                       || LOG1(memory[ost:ost+len], topic0)
A2  | LOG2          |[A7](#a7-log-operations)| ost, len, topic0, topic1 |                    || LOG1(memory[ost:ost+len], topic0, topic1)
A3  | LOG3          |[A7](#a7-log-operations)| ost, len, topic0, topic1, topic2 |            || LOG1(memory[ost:ost+len], topic0, topic1, topic2)
A4  | LOG4          |[A7](#a7-log-operations)| ost, len, topic0, topic1, topic2, topic3 |    || LOG1(memory[ost:ost+len],&#160;topic0,&#160;topic1,&#160;topic2,&#160;topic3)
A5-EF| *invalid*
F0  | CREATE        |32000[\*](#a2-memory-expansion)| val, ost, len                                     | addr          || addr = keccak256(rlp([address(this), this.nonce]))
F1  | CALL          |[AB](#ab-call-operations)| gas,&#160;addr,&#160;val,&#160;argOst,&#160;argLen,&#160;retOst,&#160;retLen | success        | mem[retOst:retOst+retLen] = returndata |
F2  | CALLCODE      |[AB](#ab-call-operations)| gas, addr, val, argOst, argLen, retOst, retLen    | success       | mem[retOst:retOst+retLen]&#160;=&#160;returndata | same&#160;as&#160;DELEGATECALL,&#160;but&#160;does&#160;not&#160;propagate&#160;original&#160;msg.sender&#160;and&#160;msg.value
F3  | RETURN        |0[\*](#a2-memory-expansion)| ost, len                                          |               || return mem[ost:ost+len]
F4  | DELEGATECALL  |[AB](#ab-call-operations)| gas, addr, argOst, argLen, retOst, retLen         | success       | mem[retOst:retOst+retLen] = returndata |
F5  | CREATE2       |[A9](#a9-create2)| val, ost, len, salt                               | addr          || addr = keccak256(0xff ++ address(this) ++ salt ++ keccak256(mem[ost:ost+len]))[12:]
F6-F9| *invalid*
FA  | STATICCALL    |[AB](#ab-call-operations)| gas, addr, argOst, argLen, retOst, retLen         | success       | mem[retOst:retOst+retLen] = returndata |
FB-FC| *invalid*
FD  | REVERT        |0[\*](#a2-memory-expansion)| ost, len                                          |               || revert(mem[ost:ost+len])
FE  | INVALID       |[AF](#af-invalid)  |                                                   |               || designated invalid opcode - [EIP-141](https://eips.ethereum.org/EIPS/eip-141)
FF  | SELFDESTRUCT  |[AA](#aa-selfdestruct)| addr                                              |               || destroy contract and sends all funds to `addr`
  
  
  
# Appendix - Dynamic Gas Costs

## A1: Intrinsic Gas
Intrinsic gas is the amount of gas paid prior to execution of a transaction.
That is, the gas paid by the initiator of a transaction, which will always be an externally-owned account, before any state updates are made or any code is executed.

Gas Calculation:
- `gas_cost = 21000`: base cost
- **If** `msg.to == null` (contract creation tx):
    - `gas_cost += 32000`
- `gas_cost += 4 * bytes_zero`: gas added to base cost for every zero byte of memory data
- `gas_cost += 16 * bytes_nonzero`: gas added to base cost for every nonzero byte of memory data

## A2: Memory Expansion
An additional gas cost is paid by any operation that expands the memory that is in use.
This memory expansion cost is dependent on the existing memory size and is `0` if the op does not reference a memory address higher than the existing highest referenced memory address.
A reference is any read, write, or other usage of memory (such as in a `CALL`).

Terms:
- `new_mem_size`: the highest referenced memory address after the operation in question (in bytes)
- `new_mem_size_words = (new_mem_size + 31) // 32`: number of (32-byte) words required for memory after the operation in question
- <code><em>C<sub>mem</sub>(machine_state)</em></code>: the memory cost function for a given machine state

Gas Calculation:
- <code>gas\_cost = <em>C<sub>mem</sub>(new_state)</em> - <em>C<sub>mem</sub>(old_state)</em></code>
- <code>gas\_cost = (new\_mem\_size\_words ^ 2 / 512) + (3 * new\_mem\_size\_words) - <em>C<sub>mem</sub>(old_state)</em></code>

Useful Notes:
- The following ops incur a memory expansion cost in addition to their otherwise static gas cost: `RETURN`, `REVERT`, `MLOAD`, `MSTORE`, `MSTORE8`, `CREATE`.
- Referencing a zero length range does not require memory to be extended to the beginning of the range.
- The memory cost function is linear up to 724 bytes of memory used, at which point additional memory costs substantially more.

## A3: EXP
Terms:
- `byte_len_exponent`: the number of bytes in the exponent (exponent is `b` in the stack representation above)

Gas Calculation:
- `gas_cost = 10 + 50 * byte_len_exponent`

## A4: SHA3
Terms:
- `data_size`: size of the message to hash in bytes (`len` in the stack representation above)
- `data_size_words = (data_size + 31) // 32`: number of (32-byte) words in the message to hash
- `mem_expansion_cost`: the cost of any memory expansion required (see [A2](#a2-memory-expansion))

Gas Calculation:
- `gas_cost = 30 + 6 * data_size_words + mem_expansion_cost`

## A5: \*COPY Operations
The following applies for the operations `CALLDATACOPY`, `CODECOPY`, `EXTCODECOPY`, and `RETURNDATACOPY`.
Note that `EXTCODECOPY` has a different `static_cost` than the other ops and also takes its `len` parameter from a different index on the stack.

Terms:
- `static_cost`: the constant base cost for the operation
    - `static_cost = 3` for `CALLDATACOPY`, `CODECOPY`, `RETURNDATACOPY`
    - `static_cost = 700` for `EXTCODECOPY`
- `data_size`: size of the data to copy in bytes (`len` in the stack representation above)
- `data_size_words = (data_size + 31) // 32`: number of (32-byte) words in the data to copy
- `mem_expansion_cost`: the cost of any memory expansion required (see [A2](#a2-memory-expansion))

Gas Calculation:
- `gas_cost = static_cost + 3 * data_size_words + mem_expansion_cost`


## A6: SSTORE
This gets messy.
See [EIP-2200](https://eips.ethereum.org/EIPS/eip-2200), implemented in Istanbul hardfork.
The short version is the cost of an `SSTORE` op is dependent on:
1. Zero vs. nonzero values - storing nonzero values is more costly than storing zero
1. The current value of the slot vs. the value to store - changing the value of a slot is more costly than not changing it
2. "Dirty" vs. "clean" slot - changing a slot that has not yet been changed within the current execution context is more costly than changing a slot that has already been changed

Terms:
- `og_val`: the value of the storage slot if the current transaction is reverted
- `current_val`: the value of the storage slot immediately *before* the `sstore` op in question
- `new_val`: the value of the storage slot immediately *after* the `sstore` op in question

Gas Calculation:
- **If** `gas_left <= 2300`:
    - `throw OUT_OF_GAS` (can not `sstore` with < 2300 gas for backwards compatibility)
- `gas_cost = 0`
- `gas_refund = 0`
- **If** `new_val == current_val` (no-op):
    - `gas_cost += 800`
- **If** `new_val != current_val`:
    - **If** `current_val == og_val` ("clean slot", not yet updated in current execution context):
        - **If** `og_val == 0` (slot started zero, currently still zero, now being changed to nonzero):
            - `gas_cost += 20000`
        - **Else** (slot started nonzero, currently still same nonzero value, now being changed):
            - `gas_cost += 5000` and..
            - **If** `new_val == 0` (the value to store is 0):
                - `gas_refund += 15000`
    - **If**: `current_val != og_val` ("dirty slot", already updated in current execution context):
        - `gas_cost += 800` and..
       - **If** `og_val != 0` (execution context started with a nonzero value in slot):
            - **If** `current_val == 0` (slot started nonzero, currently zero, now being changed to nonzero):
                - `gas_refund -= 15000`
            - **If** `new_value == 0` (slot started nonzero, currently still nonzero, now being changed to zero):
                - `gas_refund += 15000`
        - **If** `new_val == og_val` (slot is reset to the value it started with):
            - **If** `og_val == 0` (slot started zero, currently nonzero, now being reset to zero):
                - `gas_refund += 19200`
            - **Else** (slot started nonzero, currently different nonzero value, now reset to orig. nonzero value):
                - `gas_refund += 4200`

## A7: LOG\* Operations
Note that for `LOG*` ops gas is paid per byte of data (not per word).

Terms:
- `num_topics`: the \* of the LOG\* op. e.g. LOG0 has `num_topics = 0`, LOG4 has `num_topics = 4`
- `data_size`: size of the data to log in bytes (`len` in the stack representation above).
- `mem_expansion_cost`: the cost of any memory expansion required (see [A2](#a2-memory-expansion))

Gas Calculation:
- `gas_cost = 375 + 375 * num_topics + 8 * data_size + mem_expansion_cost`


## A9: CREATE2
Note that `CREATE2` incurs an additional dynamic cost over `CREATE` because of the need to hash the init code.
Also bear in mind that there is a cost incurred for storing code in addition to the costs presented here for `CREATE` and `CREATE2`.

Terms:
- `data_size`: size of the init code in bytes (`len` in the stack representation above)
- `data_size_words = (data_size + 31) // 32`: number of (32-byte) words in the init code
- `mem_expansion_cost`: the cost of any memory expansion required (see [A2](#a2-memory-expansion))

Gas Calculation:
- `gas_cost = 32000 + 6 * data_size_words + mem_expansion_cost`


## AA: SELFDESTRUCT
Note that the gas cost of a `SELFDESTRUCT` op is dependent on whether or not the op results in a new account being added to the state trie.
If a nonzero amount of eth is sent to an address that was previously empty, an additional cost is incurred.
"Empty", in this case is defined according to [EIP-161](https://eips.ethereum.org/EIPS/eip-161) (`balance == nonce == code == 0x`).

Terms:
- `target_addr`: the recipient of the self-destructing contract's funds (`addr` in the stack representation above)
- `context_addr`: the address of the current execution context (e.g. what `ADDRESS` would put on the stack)

Gas Calculation:
- `gas_cost = 5000`: base cost
- `gas_refund = 24000`: base refund
- **If** `balance(context_addr) > 0 && is_empty(target_addr)` (sending funds to a previously empty address):
    - `gas_cost += 25000`


## AB: CALL Operations
Gas costs for `CALL`, `CALLCODE`, `DELEGATECALL`, and `STATICCALL` ops.
Note that a big piece of the gas calculation for these operations is determining the gas to send along with the call.
There's a good chance that you are primarily interested in the `base_cost` and can ignore this additional calculation.
If you're not that lucky, see the `gas_sent_with_call` [section](#ac-gas-to-send-with-call-operations).
Similar to selfdestruct, `CALL` incurs an additional cost if it forces an account to be added to the state trie by sending a nonzero amount of eth to an address that was previously empty.
"Empty", in this case is defined according to [EIP-161](https://eips.ethereum.org/EIPS/eip-161) (`balance == nonce == code == 0x`).

Terms:
- `call_value`: the value sent with the call (`val` in the stack representation above)
- `target_addr`: the recipient of the call (`addr` in the stack representation above)
- `mem_expansion_cost`: the cost of any memory expansion required (see [A2](#a2-memory-expansion))
- `gas_sent_with_call`: the gas ultimately sent with the call

### AB-1: CALL

Gas Calculation:
- `base_gas = mem_expansion_cost`
- **If** `call_value > 0` (sending value with call):
    - `base_gas += 9000`
    - **If** `is_empty(target_addr)` (forcing a new account to be created in the state trie):
        - `base_gas += 25000`

Calculate the `gas_sent_with_call` [below](#ac-gas-to-send-with-call-operations).

And the final cost of the operation:
- `gas_cost = base_gas + gas_sent_with_call`

### AB-2: CALLCODE

Gas Calculation:
- `base_gas = mem_expansion_cost`
- **If** `call_value > 0` (sending value with call):
    - `base_gas += 9000`

Calculate the `gas_sent_with_call` [below](#ac-gas-to-send-with-call-operations).

And the final cost of the operation:
- `gas_cost = base_gas + gas_sent_with_call`

### AB-3: DELEGATECALL

Gas Calculation:
- `base_gas = mem_expansion_cost`

Calculate the `gas_sent_with_call` [below](#ac-gas-to-send-with-call-operations).

And the final cost of the operation:
- `gas_cost = base_gas + gas_sent_with_call`

### AB-4: STATICCALL

Gas Calculation:
- `base_gas = mem_expansion_cost`

Calculate the `gas_sent_with_call` [below](#ac-gas-to-send-with-call-operations).

And the final cost of the operation:
- `gas_cost = base_gas + gas_sent_with_call`


## AC: Gas to Send with CALL Operations
In addition to the base cost of executing the operation, `CALL`, `CALLCODE`, `DELEGATECALL`, and `STATICCALL` also need to determine how much gas to send along with the call.
Calculating this is fairly involved.
Much of the complexity comes from a backward-compatible change made in [EIP-150](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-150.md).
Here's an attempt to explain what's going on:

Basically, EIP-150 increased the `base_cost` of the `CALL` op, but most contracts in use at the time were sending `available_gas - original_base_gas` with every call.
So, when `base_gas` increased, these contracts were suddenly trying to send more gas than they had left (`requested_gas > remaining_gas`).
To avoid breaking these contracts, if the `requested_gas` is more than `remaining_gas`, we send `all_but_one_64th` of `remaining_gas` instead of trying to send `requested_gas`, which would result in an `OUT_OF_GAS_ERROR`.

Terms:
- `base_gas`: the cost of the operation before taking into account the gas that should be sent along with the call.
See [AB](#ab-call-operations) for this calculation.
- `available_gas`: the gas remaining in the current execution context immediately before the execution of the op
- `remaining_gas`: the gas remaining after deducting `base_cost` of the op but before deducting `gas_sent_with_call`
- `requested_gas`: the gas requested to be sent with the call (`gas` in the stack representation above)
- `all_but_one_64th`: All but floor(1/64) of the remaining gas
- `gas_sent_with_call`: the gas ultimately sent with the call

Gas Calculation:
- `remaining_gas = available_gas - base_gas`
- `all_but_one_64th = remaining_gas - (remaining_gas // 64)`
- `gas_sent_with_call = min(requested_gas, all_but_one_64th)`

Also note that any portion of `gas_sent_with_call` that is not used by the recipient of the call is refunded to the caller after the call returns.


## AF: INVALID
On execution of any invalid operation, whether the designated `INVALID` op or simply an undefined op, all remaining gas is consumed and the state is reverted to the point immediately prior to the beginning of the current execution context.
