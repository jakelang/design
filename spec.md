# ewasm specification

## Introduction

This document aims to specify an ewasm VM in a way useful to contract writers and VM implementers.
To this end, multiple things are specified:

-   The extra state that a VM needs to have around to successfully respond to calls into the EEI.
-   The EEI (Ethereum Environment Interface), currently specified loosley [here](eth_interface.md).

## Notation

Productions are defined in an EBNF style.
Terminals are regular expressions inside quotations, eg. `"if"`, and non-terminals are capitalized identifiers (referred to as sorts).
Multiple productions for a sort can be declared together using the alternation separator, (`|`).

```
    Exp ::= Exp "+" Exp
          | Exp "*" Exp
          | "-" Exp
```

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

## Execution State

## Initialisation of a contract

## Execution of the contract entry point

## Host functions available to the contract

