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

let equals_to_uint_8: interval(start = 100_000, stop = 150_000) = 100_100;
// Stored as an 8-bit unsigned integer plus a constant bias known at compile time

let equals_to_u8: interval(start = 1_000_000_000, stop = 1_000_000_255) = 1_000_000_100;
// The compiler can optimize and do +1,000,000,000 regardless of size

let equals_to_int_8: interval(start = -150_000, stop = -100_000) = 123_456;
// Same thing but for signed integer of 8 bits

let equals_to_i8: interval(start = -100_000, stop = -150_000, step = -1) = -123_456;
// Same thing but for signed integer of 8 bits
```

---


# Interval Type Operations


## Arithmetic Operations

### Addition (`+`)

**Interval + Interval → Interval (when bounds are statically safe)**
```rust
let a: interval(10) = 5;        // [0, 10]
let b: interval(20) = 10;       // [0, 20]
let c: interval(30) = a + b;    // [0, 30], interval(30) can be infered
```

**Interval + isize → isize (runtime check or degrades)**
```rust
let a: interval(10) = 5;
let b: isize = 3;
let c: isize = a + b;    // Result: 8 (type degrades to isize)
```

**Step-aware addition:**
```rust
let even: interval(start = 0, stop = 20, step = 2) = 4;
let odd: interval(start = 1, stop = 19, step = 2) = 5;
let sum: interval(start = 1, stop = 39, step = 1) = even + odd;    // step becomes gcd(2,2)=1 or just 1
```

**Exponent-aware addition:**
```rust
let money1: interval(start = 0, stop = 100_00, exponent = -2) = 10_50;             // $10.50
let money2: interval(start = 0, stop = 100_00, exponent = -2) = 5_25;              // $5.25
let total: interval(start = 0, stop = 200_00, exponent = -2) = money1 + money2;    // $15.75
```

### Subtraction (`-`)

**Interval - Interval → Interval**
```rust
let a: interval(start = 0, stop = 100) = 50;
let b: interval(start = 0, stop = 30) = 20;
let c: interval(start = -30, stop = 100) = a - b;    // Result: 30
```

**Watch for unsigned underflow:**
```rust
let a: interval(10) = 5;
let b: interval(10) = 8;
let c = a - b;  // Won't compile: result could be -3, outside [0, 10]
```

### Multiplication (`*`)

**Interval * Interval → Interval**
```rust
let a: interval(10) = 5;
let b: interval(20) = 10;
let c: interval(200) = a * b;    // max = 10 * 20 = 200
```

**Exponent handling (addition):**
```rust
let price: interval(start = 0, stop = 100_00, exponent = -2) = 1050;    // $10.50
let quantity: interval(100) = 5;
let total: interval(start = 0, stop = 10_000_00, exponent = -2) = price * quantity;  
// exponent = -2 + 0 = -2, so $52.50
```

**Step interaction:**
```rust
let a: interval(start = 0, stop = 10, step = 2) = 4;
let b: interval(5) = 3;
let c = a * b;    // step becomes 2*gcd(3,all_values) or degrades
```

### Division (`/`)

**Interval / Interval → Interval (when divisor excludes 0)**
```rust
let a: interval(100) = 50;
let b: interval(start = 1, stop = 10) = 5;    // Doesn't include 0
let c: interval(100) = a / b;                 // Safe, result: 10
```

**Division by zero protection:**
```rust
let a: interval(100) = 50;
let b: interval(10) = 5;    // Includes 0
let c = a / b;              // Won't compile: divisor range includes 0
```

**Exponent handling (subtraction):**
```rust
let total: interval(start = 0, stop = 1_000_00, exponent = -2) = 52_50;  // $52.50
let quantity: interval(start = 1, stop = 100) = 5;
let price: interval(start = 0, stop = 1_000_00, exponent = -2) = total / quantity;  
// exponent = -2 - 0 = -2, so $10.50
```

### Modulo (`%`)

**Interval % Interval → Interval**
```rust
let a: interval(100) = 47;
let b: interval(start = 1, stop = 10) = 5;
let c: interval(start = 0, stop = 9) = a % b;    // Result: 2, range [0, b.max-1]
```

### Negation (`-` unary)

**-Interval → Interval**
```rust
let a: interval(start = 5, stop = 10) = 7;
let b: interval(start = -10, stop = -5) = -a;    // Result: -7
```

## Comparison Operations

All comparison operations return `bool`.

### Equality (`==`, `!=`)

```rust
let a: interval(10) = 5;
let b: interval(20) = 5;
let equal: bool = a == b;    // true
```

### Ordering (`<`, `<=`, `>`, `>=`)

```rust
let a: interval(10) = 5;
let b: interval(20) = 10;
let less: bool = a < b;  // true
```

**Compile-time optimization:**
```rust
let a: interval(start = 100, stop = 200) = 150;
let b: interval(start = 0, stop = 50) = 25;
let always_greater: bool = a > b;    // Compiler knows this is always true
```

## Bitwise Operations

### AND (`&`), OR (`|`), XOR (`^`)

**Interval ⊕ Interval → Interval (conservative bounds)**
```rust
let a: interval(15) = 12;       // 0b1100
let b: interval(7) = 5;         // 0b0101
let c: interval(15) = a & b;    // 0b0100 = 4, but type allows [0, min(15,7)]
```

### Shift (`<<`, `>>`)

**Left shift increases max exponentially:**
```rust
let a: interval(10) = 5;
let shift: interval(3) = 2;
let c: interval(40) = a << shift;    // max = 10 << 3 = 80, but declared as 40
```

**Right shift decreases max:**
```rust
let a: interval(100) = 80;
let shift: interval(3) = 2;
let c: interval(25) = a >> shift;    // max = 100 >> 0 = 100, min = 0 >> 3 = 0
```

### NOT (`~`)

**Bitwise complement:**
```rust
let a: interval(255) = 5;    // Stored as u8
let b = ~a;                  // Returns isize, not interval (unbounded result)
```

## Compound Assignment Operations

All arithmetic and bitwise operations have compound forms:

```rust
let mut a: interval(100) = 50;
a += 10;    // a is now 60
a -= 5;     // a is now 55
a *= 2;     // a is now 110 — Won't compile! Exceeds interval(100)
```

## Range/Iteration Operations

### Range construction (`..`, `..=`)

```rust
for i in interval(10) {    // Implicitly 0..=10
    println!("{}", i);
}

let range: interval(start = 5, stop = 15) = ...;
for i in range {    // Iterates 5..=15 with step awareness
    println!("{}", i);
}
```

## Type Conversion Operations

### Widening (always safe)

```rust
let narrow: interval(10) = 5;
let wide: interval(100) = narrow;    // Safe widening
let as_isize: isize = narrow;        // Degradation to isize
```

### Narrowing (compile-time checked)

```rust
let wide: interval(100) = 5;
let narrow: interval(10) = wide;            // Safe if value is known at compile time

let runtime_wide: interval(100) = get_value();
let narrow: interval(10) = runtime_wide;    // Won't compile unless proven safe
```

### Explicit casting

```rust
let a: interval(1000) = 500;
let b: interval(100) = a.saturate();     // Clamps to 100
let c: interval(100) = a.wrap();         // Wraps: 500 % 101 = 95
let d: interval(100) = a.try_into()?;    // Runtime check, returns Result
```

## Boundary Query Operations

```rust
let x: interval(start = 10, stop = 100, step = 5);

x.min()         // Returns 10
x.max()         // Returns 100
x.step()        // Returns 5
x.exponent()    // Returns 0
x.count()       // Returns (100-10)/5 + 1 = 19
```

## Summary Table

| Operation                   | interval ⊕ interval | interval ⊕ isize | Preserves interval  |
| --------------------------- | ------------------- | ---------------- | ------------------- |
| `+` `-` `*` `/` `%`         | ✓ (with bound calc) | → isize          | When bounds safe    |
| `==` `!=` `<` `>` `<=` `>=` | → bool              | → bool           | N/A                 |
| `&` `\|` `^` `<<` `>>`      | ✓ (conservative)    | → isize          | When bounds safe    |
| `~`                         | → isize             | N/A              | No                  |
| Compound (`+=`, etc.)       | ✓ (with checks)     | ✓ (with checks)  | Must stay in bounds |

---



