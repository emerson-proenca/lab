# Introduction to a new type: `interval`

`interval(start: int = 0, stop: int, step: int = 1, exponent: int = 0)`

The only accepted input type is `int` and the output type is `interval` or `int`. If `interval ⊕ interval` the output is `interval`, and if `interval ⊕ int` or `int ⊕ interval` the output is `int`.

> Interval arithmetic is only preserved when the compiler can statically infer safe bounds. Otherwise it degrades to `int`.

The `interval` type receives `int` as input and outputs an `int` at runtime. For the following definition of each parameter, I'll use C as an example due to its extensive use:

```rust
fn main() {
    let one: i32 = 1;
    let two: interval(2) = 2;
    let three: i32 = one + two;
    println!("{}", three);
}
```

> In the following examples I'll omit the `fn main() {}` wrapper to avoid repetition.

The reason an operation like this is possible is because interval is a disappear type. It does not exist beyond the IDE and compiler; at runtime this simply becomes an `int`.

## Type definition

`interval(start: int = 0, stop: int, step: int = 1, exponent: int = 0)`

- `interval()` is the name of the type.
- `start: int = 0` defines the range of possible values that this interval can operate. Default is 0, to resemble unsigned integer, for array indexing and ergonomics.
- `stop: int` is required to define the type, and both obey the following: `start <= x <= stop`.
- `step: int = 1` defaults to one and obeys the following:
  - `step > 0`: `start <= x <= stop`
  - `step < 0`: `stop <= x <= start`
  - `step = 0`: `stop == x == start` (If `stop != x != start` it won't compile. When it compiles, it's a zero sized type.)
- `exponent: int = 0` defaults to zero. This parameter exists to allow decimal operations to happen without floating point. Its use is `return value * (10 ** exponent);`

## Examples

**Stops values out of the interval range:**

```rust
let digits: interval(9) = 5;

digits = 10; 
digits = -1;
// Both won't compile due to the value not being inside the interval.
```

**Specifying the `stop` parameter in the interval:**

```rust
let months: interval(start = 1, stop = 12) = 12;

months = 13; 
months = -1;
// Both won't compile due to the value not being inside the interval.
```

**Specifying the `step` parameter in the interval:**

```rust
let odd: interval(start = 1, stop = 9, step = 2) = 5;

even = 4; 
even = 6;
// Both won't compile due to the value not being allowed per the step.
```

**Specifying the `exponent` parameter in the interval:**

```rust
let money: interval(start = -100_000, stop = 100_000, exponent = -2) = 100;
// Represents 1.00 because it exponents down (money ^-2)

money = 123456; // 1234.56
money = 6789; // 67.89
```

**Example with all the parameters:**

```rust
let sensor_delta: interval(
    start = -1_500_000,
    stop = 1_500_000,
    step = 50,
    exponent = -3
);

// sensor_delta is stored as signed 2 byte integer
// because step reduces the amount of possible values to 60,000.
```

## On the explanation of integer of 64-bits

- The input type will always be valid as long as the value passed can be stored by the language biggest integer type.
- Interval values are semantically equivalent to int, but may be stored using a narrower machine representation when provably safe.

```rust
let equals_to_uint_8: interval(255) = 100;
// Stored as uint_8, same as u8

let equals_to_u8: interval(start = 256, stop = 512) = 100;
// Stored as uint_8 or u8, same as above

let equals_to_uint_8: interval(start = 100_000, stop = 150_000) = 100_100;
// Stored as an 8-bit unsigned integer plus a constant bias known at compile time

let equals_to_u8: interval(start = 1_000_000_000, stop = 1_000_000_255) = 1_000_000_100;
// The compiler can optimize and do +1,000,000,000 regardless of size

let equals_to_int_8: interval(start = -150_000, stop = -100_000) = 123_456;
// Same thing but for signed integer of 8 bits

let equals_to_i8: interval(start = -100_000, stop = -150_000, step = -1) = -123_456;
// Same thing but for signed integer of 8 bits
```
