ewasm specification
===================

Introduction
------------

This document aims to specify an ewasm VM in a way useful to contract writers and VM implementers.
To this end, multiple things are specified:

-   The extra state that a VM needs to have around to successfully respond to calls into the EEI.
-   The EEI (Ethereum Environment Interface), currently specified loosley [here](eth_interface.md).

Notation
--------

### BuiltIn Sorts

The following sorts are assumed to exist *a priori*:

-   `Int`: The integers.
-   `IntN` for any `N`: Fixed-with integers.
-   `String`: Text strings.
-   `ByteString`: Sequences of `Int8`.

### Productions

Productions are defined in an EBNF style.
Terminals are regular expressions inside quotations, eg. `"if"`, and non-terminals are capitalized identifiers (referred to as sorts).
Multiple productions for a sort can be declared together using the alternation separator, (`|`).

```
    Exp ::= Exp "+" Exp
          | Exp "*" Exp
          | "-" Exp
```

Sort-parametric productions allow for specifying data-structures (using curly-braces following the sort names).
For example, the following declares sort-parametric tuples, for any sorts `S1` and `S2`.

```
    Tuple{S1, S2} ::= "(" S1 "," S2 ")"
```

When referring to an operator/production, we'll use a name which replaces the non-terminal positions with underbars (`_`).
For example, the above production `Exp ::= Exp "+" Exp` would be referred to as the operator `_+_`.

### Records

Records productions are defined using curly-braces and a comma-separated list of named fields.
A named field is a field name followed by a colon and the sort of the field.

```
    Nationality ::= "Resistance" | "Empire"

    Person ::= { name : String
               , age  : Int
               , side : Nationality
               }
```

Record fields can be accessed using a dot, for instance to access the `name` field of a `Person` variable `p`, use `p.name`.
A record can be copied with a field updated using the `with` keyword, eg. `p with age = p.age + 1` would increment the `age` field of `p`.

### Maps

Maps are key-value stores used throughout the specification, where the key and value sorts are specified when declaring the production.
Their syntax is given in terms of parametric productions described above.
The syntax of maps is given here:

```
    Map{S1,S2} ::= S1 "|->" S2                      // a single entry
                 | ".Map"                           // empty map
                 | Map{S1, S2} Map{S1, S2}          // map union
                 | Map{S1, S2} "[" S1 "<-" S2 "]"   // map update

    S1 ::= Map{S1,S2} "[" S1 "]"                    // map access
```

Map values can be accessed with the `_[_]` operator, and set with the `_[_<-_]` operator.
For example, if `m` is the following `Map{Int256,String}`: `3 |-> "hello"   5 |-> "goodbye"`, then:

-   `m[3]` is `"hello"`,
-   `m[6]` is undefined,
-   `m[5 <- "hello again"]` is `3 |-> "hello"   5 |-> "hello again"`, and
-   `m[6 <- "hello again"]` is `3 |-> "hello"   5 |-> "hello"   6 |-> "hello again"`.

Execution State
---------------

### Overall

The ewasm VM contains state relevant to the EEI functions, as well as a wasm engine.

**TODO**: `WasmVM` is not defined, but refers in general to the "wasm VM specified by the wasm spec".
          Perhaps this corresponds roughly to the `Store` component of the wasm spec?
          It's not clear, but in fleshing out the document it will become clearer.

```
    EwasmVM ::= { eei    : EEI
                , engine : WasmVM
                }
```

### The EEI

The EEI contains fields relevant to VMs used for executing contracts on the Ethereum blockchain.

**TODO**: In the definition of `EEI.useGas`, the field `gas` is treated like an `i64.const`, but here it's an `Int256`.
          How do we reconcile these?
          In EVM, it's an `Int256` in general.

```
    EEI ::= { statusCode : StatusCode
            , accounts   : Accounts
            , gas        : Int256
            }
```

The `StatusCode` is defined by the [EVM-C API Standard](https://github.com/ethereum/evmc).
See the repository for an explanation of each status code.

```
    StatusCode ::= "EVM_SUCCESS"
                 | "EVM_FAILURE"
                 | "EVM_INVALID_INSTRUCTION"
                 | "EVM_UNDEFINED_INSTRUCTION"
                 | "EVM_OUT_OF_GAS"
                 | "EVM_BAD_JUMP_DESTINATION"
                 | "EVM_STACK_OVERFLOW"
                 | "EVM_STACK_UNDERFLOW"
                 | "EVM_INVALID_MEMORY_ACCESS"
                 | "EVM_REVERT"
                 | "EVM_STATIC_MODE_ERROR"
                 | "EVM_REJECTED"
                 | "EVM_INTERNAL_ERROR"
```

Accounts store the four relevant fields for accounts on the Ethereum network.

```
    AccountId ::= Int160
    Accounts  ::= Map{AccountId, Account}
    Account   ::= { balance : Int256
                  , code    : ByteString
                  , storage : Map{Int256, Int256}
                  , nonce   : Int256
                  }
```

Initialisation of a contract
----------------------------

**TODO**

Execution of the contract entry point
-------------------------------------

**TODO**

Host functions available to the contract
----------------------------------------

The EEI specifies several host functions available to the user.
Conceptually, it is defined as a wasm module which is available while executing ewasm contracts.
The definition of the functions is *opaque*, meaning they are provided in terms of their update to the state and not in terms of wasm code.
In the future, they may become more transparent as suitable wasm code is found to represent them.

### EEI Module

The function signatures are provided here in wast format:

```wast
    (module
        (func (export "EEI.useGas")         (param i64)                 (result))
        (func (export "EEI.call")           (param i64 i32 i32 i32 i32) (result i32))
        (func (export "EEI.returnDataCopy") (param i32 i32 i32)         (result))
    )
```
