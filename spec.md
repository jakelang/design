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

Sort-parametric productions allow for specifying data-structures.
For example, the following declares sort-parametric tuples, for any sorts `S1` and `S2`.

```
    Tuple{S1, S2} ::= "(" S1 "," S2 ")"
```

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

Map productions are key-value stores used throughout the specification, where the key and value sorts are specified when declaring the production.
The syntax of maps is given here:

```
    Map{S1,S2} ::= S1 "|->" S2                      // a single entry
                 | ".Map"                           // empty map
                 | Map{S1, S2} Map{S1, S2}          // map union
                 | Map{S1, S2} "[" S1 "<-" S2 "]"   // map update

    S1{S1,S2} ::= Map{S1,S2} "[" S1 "]"             // map access
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

```
    EEI ::= { statusCode : StatusCode
            , accounts   : Accounts
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

**TODO**

