# ewasm specification

## Introduction

This document aims to specify an ewasm VM in a way useful to contract writers and VM implementers.
To this end, multiple things are specified:

-   The extra state that a VM needs to have around to successfully respond to calls into the EEI.
-   The EEI (Ethereum Environment Interface), currently specified loosley [here](eth_interface.md).

## Notation

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

## Execution State

## Initialisation of a contract

## Execution of the contract entry point

## Host functions available to the contract

