# Interval Solutions

## Clear Intent Through Type Signature
An interval type explicitly declares the domain of valid values right in the type itself. The compiler knows exactly what the data represents, and so will you when you revisit the code.

```rust
let days_in_year: interval(365) = 365;      // compiler knows valid range [0, 365]
let percentage: interval(100) = 75;         // clearly [0, 100]
let array_index: interval(9) = 5;           // valid indices for 10-element array

let invalid_days: interval(365) = 36500;    // won't compile
let invalid_percent: interval(100) = -123;  // won't compile
let invalid_index: interval(9) = -5;        // won't compile
```

## One Type, Infinite Configurations
Instead of memorizing dozens of integer variants optimized for machine layout, you have one type that adapts to your domain. The compiler figures out the optimal storage.

```rust
let small: interval(255) = 200;                             // stored as u8
let medium: interval(65_535) = 50_000;                      // stored as u16
let offset: interval(start = 1_000, stop = 1_255) = 1_100;  // stored as u8 with bias

// No need to remember int2, int4, uint8_t,...
// Just describe your domain: the compiler optimizes storage automatically
```

## Portable Semantics
Interval semantics remain consistent across platforms. The interval describes the logical domain, not the machine representation. No more platform-dependent behavior.

```rust
let x: interval(start = 0, stop = 2_147_483_647) = 2_147_483_647;
let y: interval(start = 0, stop = 2_147_483_648) = x + 1;

// Always results in interval(start = 0, stop = 4_294_967_295) 4 bytes unsigned
// Behavior is consistent regardless of 32-bit or 64-bit compilation
```

## Overflow Prevention at Compile Time
The compiler tracks the range of possible values through every operation. If an overflow is possible, the resulting interval type reflects the expanded range—or the code won't compile if it violates a type constraint.

```rust
let age: interval(start = 0, stop = 150) = 27;
let years_passed: interval(228) = 228;

// Result type is interval(start = 0, stop = 378)
let future_age = age + years_passed;  // Compiles, type reflects reality

let constrained_age: interval(150) = future_age;  // Won't compile: 378 > 150

let balance: interval(start = 0, stop = 2_147_483_647) = 2_147_483_647;
let increment: interval(1) = 1;

// Result is interval(start = 0, stop = 2_147_483_648)
let new_balance = balance + increment;  // Type system tracks the overflow
```

## Optimal Storage for Any Domain
Hardware loves powers of two, but neither your domain or your brain has to suffer. Define exactly the range you need, and the compiler optimizes storage with biasing and offset.

```rust
let percentage: interval(100) = 75;                                   // 1 byte simply
let big_number: interval(start = 100_000, stop = 150_000) = 123_456;  // Stored in 2 bytes not 4
let hour: interval(start = 0, stop = 512, step = 2) = 334;            // Stored in 1 bytes not 2
```

## Built-in Decimal Precision
The exponent parameter handles decimal places natively. No manual scaling, no decimal confusion. The type system knows you're working with fractional values.

```rust
let price: interval(start = 0, stop = 100_000, exponent = -2) = 12_34;  // $12.34
let quantity: interval(100) = 100;

let subtotal: interval(start = 0, stop = 10_000_000, exponent = -2) = price * quantity;  // $1234.00

// No manual scaling, no forgetting to divide by 100
// The type system tracks decimal places through all operations
```

## Constrained Step Logic
The step parameter encodes domain constraints like "only even numbers" or "multiples of 5" directly in the type. Invalid values simply won't compile.

```rust
let even_only: interval(start = 0, stop = 100, step = 2) = 42;  // valid
let invalid_even: interval(start = 0, stop = 100, step = 2) = 43;  // won't compile

let nickels: interval(start = 0, stop = 100, step = 5) = 50;  // valid: 50 cents
let invalid_amount: interval(start = 0, stop = 100, step = 5) = 23;  // won't compile

let odd_primes: interval(start = 3, stop = 97, step = 2) = 37;  // it was a prime all along!
let not_odd: interval(start = 3, stop = 97, step = 2) = 38;  // won't compile
```

## Minimal Valid State Space
The type exactly matches the valid domain. An array of 10 elements gets an index type with exactly 10 possible values (0-9). No bounds checking needed at runtime because invalid indices can't exist.

```rust
let arr: [i32; 10] = [0; 10];
let index: interval(9) = 5;  // only 0-9 are valid, matches array size exactly

let value = arr[index];  // no runtime bounds check needed!

let invalid_index: interval(9) = 2_147_483_647;  // won't compile
let out_of_bounds: interval(9) = 10;  // won't compile
```
