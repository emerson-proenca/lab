# Integer Problems:

## Ambiguous Intent
A standard int doesn't tell you what the data actually represents. Is it a day of the week? A percentage? An array index? The compiler has no idea, and neither will you when you come back to this code next time.

```c
int days_in_the_year = 36500;   // compiler: "Seems fine to me!"
int percentage = -123;          // also fine, apparently
int array_index = -5;           // why not?
```

## Too Many Types
We've got `int`, `short`, `long`, `uint8_t`, `long long`, `long int`, `int4`, `int2`, and the list goes on. These types are optimized for the machine, not for humans. Quick, what's the maximum value of int2?

```c
uint8_t small_number = 300;     // whoops, overflow
int16_t mystery_size;           // Is this -32768 to 32767? I always forget
unsigned long long big_thing;   // What can this even store?
```

## Inconsistency
An `int` in Python is completely different from an `int` in C which makes sense, okay. But here's a good one an `int` in C can be different from an `int` in C! depending on whether you're compiling for 32-bit or 64-bit!

```c
int x = 2147483647;
x = x + 1;  

// On some systems: -2147483648
// On others: undefined behavior
```

## Silent Overflow
Integers overflow silently, wrapping around like a clock or just crashing your program. They don't know that your domain logic says "this should never exceed 100" because the type itself has no concept of domain constraints.

```c
unsigned char age = 27;
age = age + 228;  // age is now 0, congrats on de-aging!

int balance = 2147483647;
balance = balance + 1;  // undefined behavior, might be negative now
```

## The "Power of Two" Trap
Hardware loves powers of two, but your domain logic doesn't and your brain certainly hates it. You need to store values 0 to 100? Too bad, pick 8 bits (wastes 155 values) or do bitpacking manually.

```c
uint8_t percentage;     // 0-255, but you only need 0-100
uint8_t day_of_month;   // 0-255, but you only need 1-31
uint8_t hour;           // 0-255, but you only need 0-23

// Wasted: 155 + 224 + 232 = 611 invalid states per struct
```

## Loss of Precision (Scaling)
Try representing money or measurements with integers and you'll be manually tracking decimal places in your head. Forget to scale correctly once and the financial system sends someone a bill for $1,234.00 instead of $12.34.

```c
int price_in_cents = 1234;  // is this $12.99 or $1299.00?
int quantity = 100;       

int total = price_in_cents * quantity;  // oops, should have been (price_in_cents * quantity) / 100
```

## Undefined Step Logic
Integers assume every value between min and max is valid. Want only even numbers? Multiples of 5? Values divisible by 3? You'll be writing validation logic everywhere!

```c
int prime_number = 37;  // Is 37 prime? If it compiles it's fine!
int odd_only = 2;   // 2 is odd by the way
```

## Invalid State Space
When your array has 10 elements, why does your index type allow 4 billion possible values? You're doing bounds checking at runtime to catch what the type system should have prevented at compile time.

```c
int arr[10];
int index = 2147483647;  // completely valid according to the type!
arr[index];              
```
