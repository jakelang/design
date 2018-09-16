# Metering

Given a set of operations and a corresponding set of costs for each operation we can deterministically run computation for any number of cost units by summing up the costs on the execution of each operation. We call the cost units here "gas" and it stands as a estimation for computational time. 

## Metering in WASM

The following has been implemented [here](https://github.com/ewasm/wasm-metering)

## Metering by Branch
To meter Webassembly we first define all the operation that can cause branches. 

`const branching_ops = new Set(['end', 'br', 'br_table', 'br_if', 'if', 'else', 'return', 'loop'])`

We also define a map that contains each opcode and its associated cost. We will refer to this as the cost table. The Default cost table is defined [here](https://github.com/ewasm/design/blob/master/determining_wasm_gas_costs.md)

```
cost_table = {
  'op': cost
}
```

Lastly we need a metering statement, we will uses
```
i64.const <cost>
call $meter
```

And a metering function `$meter`. The meter function has a signature of `(type (func (param i64)))`. Internally this function should keep a running sum and if that sum grows larger than a given threshold, end the program's execution. The metering function can be imbedded in the binary itself or can use wasm's import to define it externally. 

Then given an array of opcodes we iterate the array and divided into segments that start with one of the `branching_ops`

```javascript
const code = [...opcodes]
const segments = []
let current_segment = []

for (let op in code) {
  current_segment.push(op)
  if (branching_ops.has(op)) {
    segments.push(current_segment)
    current_segment = []
  }
}
```

Then for each segment we calculate the sum of the operations. At the beginning for each segment we then append a metering statement. 

```javascript
metered_segments = segments.map(segment => {
  let cost_of_segment = segment.reduce((sum, op) => {
    return sum + cost_table[op]
  }, 0)
  
  return getMeterStatement(cost).concat(segment)
})
```

Lastly we concatenate all the metered segments together
```javascript
metered_code = metered_segments.reduce(a, b => {
  return a.concat(b)
},[])
```

## Special metering: memory

Metering memory makes use of the separate memory system of WebAssembly:
- the module parameter `memory`
- the two instructions:
  - `grow_memory`
  - `current_memory`

Memory size is expressed in pages, where a page is 65536 bytes.

The module parameter specifies the initial page count and an optional maximum page count the module cannot exceed. The currently available page count can be queried via `current_memory` and new pages can be allocated via `grow_memory`. Read more about memory in the [the WebAssembly design](https://github.com/WebAssembly/design/blob/master/Modules.md#linear-memory-section).

Gas needs to be charged for the initial allocated pages as well as any increase of pages via `grow_memory`.

### Initial memory allocation

The cost of pre-allocated memory should be counted before instantiating the module.

### Increasing memory

Any calls to `grow_memory` needs to be prepended with a call for metering.

## Examples

The following examples assume we have a cost table that defines all operations to have the cost of 1

### Basic
```
(module
  (fun
    i64.const 1 ;; +1
    drop        ;; +1
    end         ;; +1
  )
)
```
This code can be transformed to
```
(module
  (type (func))
  (type (func (param i64)))
  
  (import "ethereum" "useGas" (func $meter (type 1)))
  (func (type 0)
   	i64.const 5 ;; 3 +  2 the cost of metering  
	call $meter
    i64.const 1 
    drop       
    end       
  )
    
```
### Conditionals


```
(module
  (func $fac (param i64) (result i64)
    (if i64
      (i64.lt_s (get_local 0) (i64.const 1))
      (then (i64.const 1))
      (else
        (i64.mul
          (get_local 0)
          (call $fac
            (i64.sub
              (get_local 0)
              (i64.const 1)))))))
  (export "fac" (func $fac)))
```
This code can be transformed to
```
(module
  (type $type0 (func (param i32) (result i32)))
  (type $type1 (func (param i32)))
  (import $meter "metering" "usegas"  (param i32))
  (export "fac" $func1)
  (func $func1 (param $var0 i32) (result i32)
    i32.const 211
    call $meter
    get_local $var0
    i32.const 1
    i32.lt_s
    if i32
      i32.const 180
      call $import0
      i32.const 1
    else
      i32.const 420
      call $meter
      get_local $var0
      get_local $var0
      i32.const 1
      i32.sub
      call $func1
      i32.mul
    end
  )
)
```

## Future Work

More efficient metering algorithms can be defined. For example if we can prove that an `end` is impossible to jump to it doesn't need to be segmented. The tradeoff here is the complexity for implementing these algorithms.
