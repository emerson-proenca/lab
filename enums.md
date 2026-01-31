# ENUM-Based Type System: A Programming Language Concept

## Core Philosophy

**What if there were no primitive types? Only constrained value sets.**

Every value in this language is a member of an explicitly defined enumeration of valid values. Instead of choosing data types based on their implementation (int32, float64, char), you declare the exact range or set of values you need, and the compiler optimizes storage automatically.

## Motivation

Traditional programming forces you to think in terms of containers:
- "I need an int" (gives you -2 billion to +2 billion, even if you only need 0-100_000)
- "I need a char" (gives you 256 values, even if you only need 4)
- "I need a float" (gives you approximate decimals with unpredictable precision)

This language inverts that model:
- **You declare what values you need**
- **The compiler figures out optimal storage**

## Basic Syntax

### Numeric Ranges

```ts
// Integer range
id: ENUM(range(0, 999_999)) = 0
// Stores any value from 0 to 999,999

// Decimal range (precision inferred from bounds)
money: ENUM(range(-999_999.99, +999_999.99)) = 0
// Stores values with 2 decimal places

temperature: ENUM(range(-273.15, 1_000_000.0)) = 0
// Stores values with 2 decimal places (max precision from bounds)
```

### Explicit Value Sets

```ts
// Boolean (2 values → 1 bit)
flag: ENUM(true, false) = true

// DNA nucleotide (4 values → 2 bits!)
nucleotide: ENUM('A', 'C', 'G', 'T') = 'A'

// Days of week (7 values → 3 bits)
day: ENUM('Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun') = 'Mon'

// Cardinal directions (4 values → 2 bits)
direction: ENUM('N', 'E', 'S', 'W') = 'N'
```

### Character Ranges

```ts
// Single character from range
grade: ENUM(range('A', 'F')) = 'A'

// String with character constraints (needs additional syntax)
username: ENUM(range('a','z'), length(3, 20)) = "user"
```

## Default Numeric Type

If no constraints specified:

```
num: ENUM() 
```

Defaults to:
- **Range**: -9,999,999,999.999999 to +9,999,999,999.999999
- **Precision**: 6 decimal places
- **Storage**: 128 bits (16 bytes)
- **Implementation**: Scaled integer (exact decimal arithmetic)

This handles 99% of everyday use cases without floating-point approximation errors.

## Key Advantages

### 1. Self-Documenting Code

```ts
year: ENUM(range(1900, 2100))
```

Immediately clear: this is a year between 1900 and 2100.

Compare to:
```c
uint16_t year;  // Could be anything from 0 to 65535
```

### 2. Automatic Optimization

The compiler calculates minimum bits needed:

```ts
// Traditional approach
char nucleotide;  // 8 bits, wastes 6

// ENUM approach  
nucleotide: ENUM('A', 'C', 'G', 'T')  // 2 bits, optimal!

// DNA sequence of 1000 base pairs
dna: ARRAY(ENUM('A','C','G','T'), 1000)
// 2000 bits instead of 8000 bits! 4x smaller
```

**Storage calculation**: For N possible values, use `ceil(log2(N))` bits

| Values | Bits Needed | Traditional (bytes) | Savings |
|--------|-------------|---------------------|---------|
| 2      | 1 bit       | 1 byte              | 87.5%   |
| 4      | 2 bits      | 1 byte              | 75%     |
| 7      | 3 bits      | 1 byte              | 62.5%   |
| 256    | 8 bits      | 1 byte              | 0%      |

### 3. Exact Decimal Arithmetic

Using scaled integers internally:

```ts
price: ENUM(range(0.00, 999.99))
// Internally: int16 storing cents (0 to 99999)
// No floating-point errors!

a = 0.1
b = 0.2  
print(a + b)  // Exactly 0.3, not 0.30000000000000004
```

### 4. Built-in Validation

```ts
age: ENUM(range(0, 150)) = 25
age = 200  // Compile-time or runtime error!
```

The type system **IS** the validation layer.

### 5. Intuitive Arithmetic

```ts
id: ENUM(range(0, 999_999)) = 1234
money: ENUM(range(-999_999.99, +999_999.99)) = 567.89

result = id + money  
// 1234 + 567.89 = 1801.89
// Result type inferred: ENUM(range(...), precision=2)
```

## Compound Types (Structures)

ENUMs can be nested to create record types:

```ts
User: ENUM(
  id: ENUM(range(0, 999_999)),
  username: ENUM(range('a','z'), length(3, 20)),
  age: ENUM(range(0, 150)),
  is_active: ENUM(true, false)
)

user: User = {
  id: 12345,
  username: "alice",
  age: 28,
  is_active: true
}
```

## Collections

### Maps/Hashmaps

```ts
scores: MAP(
  key: ENUM(range('a','z'), length(1, 10)),
  value: ENUM(range(0, 100))
)
```

### Arrays

```ts
grades: ARRAY(ENUM('A', 'B', 'C', 'D', 'F'), length(10))
// 10 grades, each using 3 bits = 30 bits total
```

## Relationship to Traditional Types

Every traditional type is just a predefined ENUM:

```ts
// These are equivalent:
int32    ≡ ENUM(range(-2_147_483_648, 2_147_483_647))
uint8    ≡ ENUM(range(0, 255))
bool     ≡ ENUM(true, false)
char     ≡ ENUM(range(0, 127))  // ASCII
byte     ≡ ENUM(range(0, 255))
```

The language just makes the constraint explicit and generalizes it.

## Real-World Use Cases

### Bioinformatics
```ts
// DNA sequence: 4× memory savings
genome: ARRAY(ENUM('A', 'C', 'G', 'T'), length(3_000_000_000))
```

### Financial Systems
```ts
// Exact decimal math, no rounding errors
balance: ENUM(range(-999_999_999.99, +999_999_999.99))
```

### Embedded Systems
```ts
// Minimal bit usage for sensors
sensor_state: ENUM('IDLE', 'ACTIVE', 'ERROR', 'CALIBRATING')
// 2 bits instead of 8 or 32
```

### Game Development
```ts
// Compact state machines
player_state: ENUM('IDLE', 'WALKING', 'RUNNING', 'JUMPING', 'FALLING')
// 3 bits per entity
```

## Implementation Details

### Storage Strategy

1. **Calculate minimum bits**: `ceil(log2(number_of_values))`
2. **Decide on packing**:
   - Bit-pack for space efficiency (arrays, structs)
   - Byte-align for speed (frequently accessed variables)
3. **For decimal ranges**: Use scaled integers (multiply by 10^precision)

### Arithmetic Type Inference

```ts
a: ENUM(range(0, 100))      // max 100
b: ENUM(range(0, 50))       // max 50
c = a + b                   // inferred: ENUM(range(0, 150))
```

Result range = operation applied to input ranges.

### Precision Handling

For decimal operations, result precision = max(operand precisions):

```ts
x: ENUM(range(0, 100.00))    // 2 decimals
y: ENUM(range(0, 10.5))      // 1 decimal  
z = x + y                    // 2 decimals
```

## Design Philosophy

1. **Declarative over Imperative**: Say *what* you need, not *how* to store it
2. **Intent over Implementation**: Types express domain constraints, not memory layout
3. **Optimize by Default**: Compiler does the hard work of finding optimal representation
4. **Fail Fast**: Invalid values are caught at compile-time or assignment-time
5. **No Surprises**: Exact decimal math, explicit precision, clear bounds

## Open Questions

- How to handle true floating-point (scientific notation, very large exponents)?
  - Perhaps: `ENUM(float, range(1e-10, 1e10))` for approximate math
- Function signatures and return type inference?
- Interaction with existing type systems (FFI, interop)?
- Performance vs space tradeoffs (when to bit-pack vs align)?
- Syntax for complex string patterns (regex-like constraints)?

## Inspiration

This concept draws from:
- **Ada's range types**: Explicit numeric bounds
- **SQL's NUMERIC/DECIMAL**: Exact decimal with precision
- **TypeScript's literal types**: Explicit value sets
- **Design by Contract**: Types as constraints
- **Algebraic data types**: Composition of constrained types

## Conclusion

By recognizing that **all types are just constrained value sets**, we can build a language that:
- Is more expressive (declare exact intent)
- Is more efficient (optimal storage automatically)
- Is more correct (validation built into types)
- Is more intuitive (think in domain terms, not bytes)

The fundamental insight: **Types should describe your problem, not your computer's memory.**

---

*This is a conceptual design for discussion and exploration. Implementation details would need significant refinement.*
