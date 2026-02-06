# Introduction to a new type: `interval`

```rust
interval(start: isize = 0, stop: isize, step: isize = 1, exponent: isize = 0)
```

The only accepted input type is `isize` and the output type is `interval` or `isize`. If `interval ⊕ interval` the output is `interval`, and if `interval ⊕ isize` or `isize ⊕ interval` the output is `isize`.

> Interval arithmetic is only preserved when the compiler can statically infer safe bounds. Otherwise it degrades to `isize`.

## Backus-Naur Form BNF

```bnf
<interval-type> ::= "interval" "(" <parameters> ")"

<parameters> ::= <stop-only>
               | <named-parameters>

<stop-only> ::= <isize>

<named-parameters> ::= <parameter-list>

<parameter-list> ::= <parameter>
                   | <parameter> "," <parameter-list>

<parameter> ::= <start-param>
              | <stop-param>
              | <step-param>
              | <exponent-param>

<start-param> ::= "start" "=" <isize>

<stop-param> ::= "stop" "=" <isize>

<step-param> ::= "step" "=" <isize>

<exponent-param> ::= "exponent" "=" <isize>

<isize> ::= <integer>

<integer> ::= <optional-sign> <digits>
            | <optional-sign> <digits-with-underscores>

<optional-sign> ::= "+" | "-" | ε

<digits> ::= <digit> | <digit> <digits>

<digits-with-underscores> ::= <digit-or-underscore> | <digit-or-underscore> <digits-with-underscores>

<digit-or-underscore> ::= <digit> | "_"

<digit> ::= "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
```

The `interval` type receives `isize` as input and outputs an `isize` at runtime. For the following definition of each parameter, I'll use Rust as an example, due to my familiarity with it:

```rust
fn main() {
    let one: i32 = 1;
    let two: interval(2) = 2;
    let three: i32 = one + two;
    println!("{}", three);
}
```

> In the following examples I'll omit the `fn main() {}` wrapper to avoid repetition.

The reason an operation like this is possible is because interval is a disappear type. It does not exist beyond the IDE and compiler. At runtime this simply becomes an `isize`.

## Type definition

`interval(start: isize = 0, stop: isize, step: isize = 1, exponent: isize = 0)`

- `interval()` is the name of the type.
- `start: isize = 0` defines the range of possible values that this interval can operate. Default is 0, to resemble unsigned integer, for array indexing and ergonomics.
- `stop: isize` is required to define the type, and both obey the following: `start <= x <= stop`.
- `step: isize = 1` defaults to one and obeys the following:
  - `step > 0`: `start <= x <= stop`
  - `step < 0`: `stop <= x <= start`
  - `step = 0`: `stop == x == start` (If `stop != x != start` it won't compile. When it compiles, it's a zero sized type.)
- `exponent: isize = 0` defaults to zero. This parameter exists to allow decimal operations to happen without floating point. Its use is `return value * (10 ** exponent);`

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
- Interval values are semantically equivalent to `isize`, but may be stored using a narrower machine representation when provably safe.

```rust
let equals_to_uint_8: interval(255) = 100;
// Stored as uint_8, same as u8

let equals_to_u8: interval(start = 256, stop = 512) = 100;
// Stored as uint_8 or u8, same as above

let equals_to_uint_16: interval(start = 100_000, stop = 150_000) = 100_100;
// Stored as an 16-bit unsigned integer plus a constant bias known at compile time

let equals_to_u8: interval(start = 1_000_000_000, stop = 1_000_000_255) = 1_000_000_100;
// The compiler can optimize and do +1,000,000,000 regardless of size

let equals_to_int_16: interval(start = -150_000, stop = -100_000) = 123_456;
// Same thing but for signed integer of 16 bits

let equals_to_i16: interval(start = -100_000, stop = -150_000) = -123_456;
// Same thing but for signed integer of 16 bits
```

---


# Interval Type Operations


## Addition (`+`)

**Interval + Interval → Interval**
The compiler sums the minimums and maximums respectively. The step is the Greatest Common Divisor (GCD) of the two steps.

```rust
let even: interval(start = 0, stop = 10, step = 2) = 4; // [0, 10]
let odd: interval(start = 1, stop = 9, step = 2) = 5; // [1, 9]

let sum: interval(start = 1, stop = 19, step = 2) = even + odd;
// start: 0 + 1 = 1
// stop: 10 + 9 = 19
// step: gcd(2, 2) = 2 
```

### Subtraction (`-`)

**Interval - Interval → Interval**
To find the minimum possible result, subtract the second interval's maximum from the first's minimum.

```rust
let a: interval(10) = 5; // [0, 10]
let b: interval(5) = 2;  // [0, 5]

let diff: interval(start = -5, stop = 10) = a - b;
// start: 0 - 5 = -5
// stop: 10 - 0 = 10
// step: gcd(2, 2) = 2
```

### Multiplication (`*`)

**Interval * Interval → Interval**
The compiler evaluates all four boundary products  to determine the new range.

```rust
let a: interval(start = -2, stop = 2) = 1;
let b: interval(start = 5, stop = 10) = 6;

let prod: interval(start = -20, stop = 20) = a * b;
// Products: {-10, -20, 10, 20}
// start: -20
// stop: 20
// step: gcd()
```

### Division (`/`)

**Interval / Interval → Interval (Conditional)**
The compiler **must** prove that the divisor interval does not contain `0`. If , the code fails to compile.

```rust
let numerator: interval(100) = 50;
let divisor: interval(start = 2, stop = 10, step = 2) = 2;

// Safe: Divisor range [2, 10] does not contain 0.
// start: 0 / 10 = 0
// stop: 100 / 2 = 50
let result: interval(50) = numerator / divisor;

let unsafe_divisor: interval(10) = 5; 
let error = numerator / unsafe_divisor; 
// ^ ERROR: potential division by zero. Divisor range [0, 10] includes 0.

```

### Exponent Interaction

When multiplying or dividing with exponents, the exponent values are added or subtracted, respectively.

```rust
let price: interval(stop = 1000, exponent = -2) = 500; // 5.00
let qty: interval(10) = 2;

// Exponent: -2 + 0 = -2
let total: interval(stop = 10000, exponent = -2) = price * qty; // 10.00

```


---










Okay so generate the correct examples following this block:

---

### Addition (`+`)

**Interval + Interval → Interval (when bounds are statically safe)**
```rust
let even interval(start = 0, stop = 8, step = 2) = 4;    // {0, 2, 4, 6, 8}
let odd interval(start = 1, stop = 9, step = 2) = 5;       // {1, 3, 5, 7, 9}

let addition = a + b;    //  interval(start = 1, stop = 17, step = 2) can be infered
// start = min(even.start + odd.start, even.stop + odd.stop) = 1
// stop = max(even.start + odd.start, even.stop + odd.stop) = 17
// step = gcd(even.start, odd.stop) = 2
```






### At Compile Time: **Step 1: Validation** 
```
Compiler checks: (-1_234_550 - (-1_500_000)) % 50 == 0 → 265_450 % 50 == 0 → ✓ Valid 
```

**Step 2: Encoding**
```
stored_index = (value - start) / step = (-1_234_550 - (-1_500_000)) / 50 = 265_450 / 50 = 5_309 Binary: 5_309 = 0b0001010010111101 (fits in 16 bits ✓)
``` 
### At Runtime: **When you need the value back:** 
``` actual_value = start + (stored_index × step) = -1_500_000 + (5_309 × 50) = -1_500_000 + 265_450 = -1_234_550 ✓






