# Fee Schedule

Fees are charged in three different ways:
- running opcodes
- expanding memory
- calls to EEI methods

## Opcodes

All fees for opcodes are currently 1 gas. This needs to be updated before finalising this specification.

### Registers
|Opcode     |Price |
|-----------|------|
|get_local  | 1    |
|set_local  | 1    |
|tee_local  | 1    |
|get_global | 1    |
|set_global | 1    |

### Memory
|Opcode        |Price |
|--------------|------|
|i32.load8_s   | 1    |
|i32.load8_u   | 1    |
|i32.load16_s  | 1    |
|i32.load16_u  | 1    |
|i32.load      | 1    |
|i64.load8_s   | 1    |
|i64.load8_u   | 1    |
|i64.load16_s  | 1    |
|i64.load16_u  | 1    |
|i64.load32_s  | 1    |
|i64.load32_u  | 1    |
|i64.load      | 1    |
|grow_memory   | 1    |
|current_memory| 1    |

### Flow Control
|Opcode     |Price |
|-----------|------|
|nop        | 1    |
|block      | 1    |
|loop       | 1    |
|if         | 1    |
|br         | 1    |
|br_if      | 1    |
|br_table   | 1    |
|return     | 1    |

### Calls
|Opcode       |Price |
|-------------|------|
|call         | 1    |
|call_indirect| 1    |
|call_import  | 1    |

### Constants
|Opcode     |Price |
|-----------|------|
|i32.const  | 1    |
|i64.const  | 1    |


### 32-bit Integer operators
|Opcode     |Price |
|-----------|------|
|i32.add    |1     |
|i32.sub    |1     |
|i32.mul    |1     |
|i32.div_s  |1     |
|i32.div_u  |1     |
|i32.rem_s  |1     |
|i32.rem_u  |1     |
|i32.and    |1     |
|i32.or     |1     |
|i32.xor    |1     |
|i32.shl    |1     |
|i32.shr_u  |1     |
|i32.shr_s  |1     |
|i32.rotl   |1     |
|i32.rotr   |1     |
|i32.eq     |1     |
|i32.ne     |1     |
|i32.lt_s   |1     |
|i32.le_s   |1     |
|i32.lt_u   |1     |
|i32.le_u   |1     |
|i32.gt_sa  |1     |
|i32.ge_s   |1     |
|i32.gt_u   |1     |
|i32.ge_u   |1     |
|i32.clz    |1     |
|i32.ctz    |1     |
|i32.popcnt |1     |
|i32.eqz    |1     |

### 64-bit Integer operators
|Opcode     |Price |
|-----------|------|
|i64.add    |1     |
|i64.sub    |1     |
|i64.mul    |1     |
|i64.div_s  |1     |
|i64.div_u  |1     |
|i64.rem_s  |1     |
|i64.rem_u  |1     |
|i64.and    |1     |
|i64.or     |1     |
|i64.xor    |1     |
|i64.shl    |1     |
|i64.shr_u  |1     |
|i64.shr_s  |1     |
|i64.rotl   |1     |
|i64.rotr   |1     |
|i64.eq     |1     |
|i64.ne     |1     |
|i64.lt_s   |1     |
|i64.le_s   |1     |
|i64.lt_u   |1     |
|i64.le_u   |1     |
|i64.gt_sa  |1     |
|i64.ge_s   |1     |
|i64.gt_u   |1     |
|i64.ge_u   |1     |
|i64.clz    |1     |
|i64.ctz    |1     |
|i64.popcnt |1     |
|i64.eqz    |1     |

### Datatype conversions, truncations, reinterpretations, promotions, and demotions
|Opcode          |Price |
|----------------|------|
|i32.wrap/i64    |1     |
|i64.extend_s/i32|1     |
|i64.extend_u/i32|1     |

### Type-parametric operators.
|Opcode     |Price |
|-----------|------|
|drop       |1     |
|select     |1     |


### Other
|Opcode     |Price |
|-----------|------|
|unreachable|1     |


## Expanding memory

Memory can be expanded in pages, where a page corresponds to 65536 bytes of space.

The EVM1 formula for extending memory is `words * 3 + words ^ 2 / 512` where `word` corresponds to 32 bytes.

From this we can calculate that a 65536 byte page should cost 14336 gas.


## Calls to the EEI

Calls to the EEI are charged the same price as their equivalent EVM1 opcode.

| Method               | EVM1 Opcode  |
|----------------------|--------------|
| getAddress           | ADDRESS      |
| getBalance           | BALANCE      |
| getBlockHash         | BLOCKHASH    |
| call                 | CALL         |
| callDataCopy         | CALLDATACOPY |
| getCallDataSize      | CALLDATASIZE |
| callCode             | CALLCODE     |
| callDelegate         | DELEGATECALL |
| storageStore         | SSTORE       |
| storageLoad          | SLOAD        |
| getCaller            | CALLER       |
| getCallValue         | CALLVALUE    |
| codeCopy             | CODECOPY     |
| getCodeSize          | CODESIZE     |
| getBlockCoinbase     | COINBASE     |
| create               | CREATE       |
| getBlockDifficulty   | DIFFICULTY   |
| externalCodeCopy     | EXTCODECOPY  |
| getExternalCodeSize  | EXTCODESIZE  |
| getGasLeft           | GAS          |
| getBlockGasLimit     | GASLIMIT     |
| getTxGasPrice        | GASPRICE     |
| log                  | LOG*n*       |
| getBlockNumber       | NUMBER       |
| getTxOrigin          | ORIGIN       |
| return               | RETURN       |
| selfDestruct         | SELFDESTRUCT |
| getBlockTimestamp    | TIMESTAMP    |
