# EVM Transcompiler

Transcompiles EVM1 bytecode into ewasm bytecode. It takes EVM bytecode as an input and returns an ewasm compatible bytecode,
which can be executed the same way regular ewasm contracts can.

When executing it, one must make sure however that host functions dealing with the bytecode in the state
(such as `codeCopy` and `externalCodeCopy`) must always refer to the untransformed EVM1 bytecode.

The transcompiler is implemented as a contract or a precompiled contract at a specific address.
