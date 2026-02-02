# Cantor Language — Memory Management Model

> **Scope**: This document formalizes *only* the memory-management semantics of the Cantor language. It intentionally excludes syntax, control flow, and general semantics, except where required to define memory safety.

---

## 1. Foundational Assumptions

Cantor is defined by the following global constraints:

1. **Finiteness**: All computational domains are finite.
2. **Closed World**: All memory domains are fully known at compile time.
3. **No Atoms**: There are no primitive scalar values; all values are elements of enums or tuples thereof.
4. **No Dynamic Allocation**: No memory region may be created, destroyed, resized, or reallocated at runtime.
5. **No Addresses as Values**: Memory addresses are not first-class values and cannot be observed or manipulated by programs.

These assumptions are not implementation details; they are semantic axioms.

---

## 2. Memory as a Finite Enum

### 2.1 Global Memory Domain

The entire addressable memory of a Cantor program is modeled as a finite enum:

```
MemoryAddress = Enum(0 .. M-1)
```

Where `M` is a compile-time constant determined by the compiler and target platform.

* `MemoryAddress` is **not** directly accessible to user programs.
* No operation may construct, compute, or manipulate values of `MemoryAddress`.

This enum exists solely as an internal semantic device for defining storage.

---

## 3. Storage Locations

### 3.1 Definition

A **storage location** is a compiler-defined association:

```
Location : MemoryAddress → ValueDomain
```

Where `ValueDomain` is a finite enum or tuple of enums.

### 3.2 Properties

* Every storage location exists for the *entire lifetime* of the program.
* Storage locations are allocated **once**, at compile time.
* Storage locations are never deallocated.

There is no concept of "free", "invalid", or "uninitialized" memory.

---

## 4. Variables and Binding

### 4.1 Variables as Names, Not Storage

A variable in Cantor is a **name bound to a storage location**.

* Variables do not own memory.
* Variables cannot be rebound to different storage locations.
* Variable scope affects *visibility only*, not lifetime.

### 4.2 Immutability of Binding

Once a variable is bound to a storage location:

* The binding is immutable.
* Only the *value* stored at the location may change, subject to domain constraints.

---

## 5. Values and Mutation

### 5.1 Value Domains

Each storage location has a fixed value domain:

```
ValueDomain = Enum(v₁, v₂, …, vₙ)
```

or a finite tuple of such enums.

### 5.2 Mutation Rule

A mutation operation is valid **iff**:

```
new_value ∈ ValueDomain
```

Otherwise, the program is rejected at compile time.

There is no runtime checking for value validity.

---

## 6. Absence of Dynamic Memory

Cantor explicitly forbids:

* Heap allocation
* Stack allocation
* Memory resizing
* Variable-sized collections
* Runtime creation of storage locations

As a consequence:

* All memory usage is statically bounded.
* All lifetimes are identical (entire program).

---

## 7. Aliasing

### 7.1 Definition

Aliasing occurs when multiple variable names refer to the same storage location.

### 7.2 Semantics

Aliasing is:

* Explicit
* Deterministic
* Fully known at compile time

There is no implicit aliasing.

### 7.3 Safety Implication

Aliasing does **not** introduce memory unsafety because:

* No storage location can become invalid.
* No storage location can be freed.
* All writes are domain-checked at compile time.

Aliasing may affect *program meaning*, but not memory safety.

---

## 8. Projections and Tuples

### 8.1 Tuple Storage

Tuples are stored as fixed collections of storage locations, one per component.

* Tuple arity is fixed at compile time.
* Each component has its own finite value domain.

### 8.2 Projection

Projection operations produce *views*, not copies.

* Projections do not expose memory addresses.
* Projections cannot escape their domain constraints.

---

## 9. Absence of Temporal Errors

Because:

* Storage locations are never deallocated
* Bindings never change
* All values are always valid

The following classes of errors are unrepresentable:

* Use-after-free
* Dangling reference
* Double free
* Lifetime violation

Time has no effect on memory validity.

---

## 10. Formal Memory Safety Guarantee

A Cantor program is **memory safe by construction**.

### Theorem (Memory Safety)

Given a valid Cantor program `P`:

1. Every memory access refers to a valid storage location.
2. Every value stored in memory belongs to its declared domain.
3. No operation can access memory outside the statically defined memory universe.

Therefore:

> **No execution of `P` can exhibit memory-unsafe behavior.**

This property holds independently of program logic, control flow, or input values.

---

## 11. Consequence

Memory safety in Cantor is not enforced via:

* Runtime checks
* Garbage collection
* Ownership systems
* Borrow checking

It emerges directly from:

* Finiteness
* Closed-world semantics
* Absence of addresses
* Absence of dynamic allocation

Memory management is thus a *compile-time concern only*.
