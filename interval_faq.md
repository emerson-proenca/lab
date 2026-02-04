# Range Type Proposal: Questions & Answers

## 1. Conceptual & Theoretical Issues

**Q1: Is this actually a new type, or syntactic sugar over refinement / subrange types?**  
What *formal property* does `range(start, stop, step, scale)` provide that cannot be encoded in existing dependent or refinement type systems?

> It is a new type, because while similar to Integer, it has different semantic meaning and obey different properties. Semantics: `id: int = 1` reads like "Id is an Integer of value 1", while `id: range(1_000) = 1` reads like "Id is in range 1_000 of value 1". Both are similar in the sense of being implemented as a Fixed type integer, but they have different bounds, one is the machine the other is the developer or user.

**Q2: Is `range()` really a primitive, or a family of parameterized types?**  
If parameterized, how does this differ from const generics or dependent typing?

> It's a primitive because an unsigned integer of size 2 maps from 0 to 65_535. While range() can map any interval which requires more than 256 states and less than or equal to 65_536 states.

**Q3: What is the mathematical structure of a `range`?**  
Is it:
- an integer modulo class?
- a finite lattice?
- a bounded affine space?

Without this, algebraic reasoning is unclear.

> It's based on Set Theory. It has no order, all elements are unique and it represents a range of possible value/states.

**Q4: What invariants are guaranteed by the type system vs runtime checks?**  
Are all violations statically rejected, or do some degrade to runtime errors?

> Yes all violations are statically rejected. It's on the developer to guarantee that the value is inside the range, if the developer doesn't, the compilation fails.

**Q5: Is `step` semantically necessary, or an optimization hint?**  
If it affects type safety, how? If not, why is it in the type?

> It's a type safety as to stop values not allowed to be injected into the variable `february_days: range(28) = 31` this doesn't compile because 31 isn't in the range of possible values. Step can also lead to optimizations such as bit packing but most importantly `even: range(stop = 510, step = 2)` can be stored in 1 byte instead of 2 bytes. Due to step effectively jumping all odd numbers.

---

## 2. Arithmetic Semantics & Closure

**Q6: Is the type closed under addition, subtraction, multiplication, division, and modulo?**  
For each operator, define:
- result type
- overflow behavior
- rejection conditions

> I have no idea.

**Q7: What happens when arithmetic produces values not aligned to `step`?**  
Example:
```text
range(0, 10, step = 2) + range(0, 10, step = 2)
```
Does the step remain 2? Become 4? Collapse to step 1?

> The step remains at 2. As all possible operations with 2 values can be destructured to those same previous values, and those values were created with that step.

**Q8: How is division defined?**  
Is this integer division, rational division, or forbidden unless exact?

> It's Integer division, if a division would cause the integer to enter decimal place beyond the scale it loses precision.

**Q9: Does multiplication explode the domain too quickly to be useful?**  
If yes, how often will arithmetic force widening to a generic integer?

> It does explode the domain, but it's still useful as the only operations that matter are done with `start` and `stop` values.

**Q10: Is arithmetic associative and commutative at the type level?**  
If not, does evaluation order affect type inference?

> I have no idea.

---

## 3. Scale & Numeric Precision

**Q11: Is `scale` part of the type identity?**  
Are `range(0,100,1,scale=2)` and `range(0,1,0.01)` the same type?

> Good point, it made me change the approach, now it's called `precision` and it only accepts integers, works similar to SQL DECIMAL(scale, precision). Your example would become: "Are `range(0,100,1,precision = 2)` and `range(0,1,0.01)` the same type?" No because precision is only int, and it maps to how many decimal places the developer wants.

**Q12: How do mixed-scale operations work?**  
What is the rule for:
```text
range(scale=2) + range(scale=3)
```

> Current mixed precision operations work by using the highest `precision`. Let AP be the precision of range A, and Let BP be the precision of range B. max(AP, BP) whichever returns is the precision to be used.

**Q13: Is scale inferred from literals, or must it be explicit?**  
If inferred, is inference stable under refactoring?

> Yes precision is inferred. I have no idea what the next question means.

**Q14: What prevents accidental scale explosion (LCM growth)?**

> The fact it will only check the start and stop values. It can still happen.

**Q15: How do you avoid recreating floating-point complexity under a new name?**

> There's no floating point, so it can't be complex. range only accepts int as input.

---

## 4. Type Inference & Ergonomics

**Q16: Can this type be inferred, or must it always be explicit?**

> It can be inferred and is inferred.

**Q17: How verbose does real-world code become?**  
Have you tested this on non-trivial programs?

> This is an ideal model and hasn't seen production use, we are building a package for it.

**Q18: What is the failure mode when inference fails?**  
Cryptic type errors kill adoption.

> The most generic error is `ValueNotInRange` which means that the Value can't ever be in that range and so, was rejected and range returned an error. More specific errors are: `OutOfRange` the value isn't in between start and stop. `OutOfStep` the value is in bound, but not allowed per the step e.g.: `range(0, 10, 2) = 5` doesn't compile because 5 is out of the step. Needs better wording.

**Q19: Does this interact sanely with generics / templates?**

> Yes, it interacts the same way Integers can.

**Q20: Can higher-order functions preserve range information, or is it erased?**

> Yes, higher-order functions can preserve range information. Same way as Integers can.

---

## 5. Interoperability & Compatibility

**Q21: What exactly does "interchangeable with int" mean?**  
Implicit coercion? Subtyping? Structural equivalence?

> It means it's possible to cast an `int` to a `range` and a cast a `range` to an `int`. They can't be treated as the same value or type. A 'naive' add operation would result in type error: `TypeError: unsupported operand type(s) for +: 'int' and 'range'`

**Q22: If `range()` defaults to int32, why is that not just `int32`?**

> range doesn't default to int32. It CAN represent int32 as `range(-2_147_483_647, 2_147_483_646)`, but it's not an integer.

**Q23: How does this interact with FFI and system ABIs?**  
Are range types layout-compatible with C integers?

> Yes, because they are just a range of possible values, they can be represented inside an INT.

**Q24: Can this be serialized/deserialized without metadata loss?**

> Yes? I have no idea.

**Q25: How do databases, network protocols, and file formats see this type?**

> It should be converted to an Integer.

---

## 6. Representation & Optimization Claims

**Q26: Is minimal bitpacking guaranteed or optional?**  
If optional, your safety claims weaken.

> It should be optional, as most languages do not benefit from trading raw speed for bit packing.

**Q27: How do you represent negative ranges efficiently?**

> The same way with an Int, by using a signed bit.

**Q28: What is the cost of bounds checks at runtime, if any?**

> I believe it's constant.

**Q29: Does bitpacking inhibit vectorization or CPU arithmetic instructions?**

> Yes it does, it shouldn't be done for most programming languages.

**Q30: How does this interact with atomic operations and concurrency?**

> The same way an Integer does.

---

## 7. Compiler & Tooling Feasibility

**Q31: How complex is the compiler implementation compared to current range analysis?**

> More complex than Integer for sure, I'm not sure what are the current range analysis.

**Q32: Can existing compilers realistically adopt this without a rewrite?**

> LLVM already does something similar to this with `@i8` and `@u16`, it'd map to that.

**Q33: What happens when range information is lost (e.g., through function calls)?**

> If the value is lost, it's still "safe" but if one of the inputs: `start`, `stop`, `step`, `scale` is lost then it should error.

**Q34: Is this a local or whole-program analysis problem?**

> It's the same as Integer, which I believe is local.

**Q35: How does this interact with separate compilation and libraries?**

> I have no idea.

---

## 8. Safety Guarantees & Soundness

**Q36: Is the system sound?**  
Can a program type-check and still overflow or violate bounds?

> No?

**Q37: Are there escape hatches?**  
If yes, do they undermine the core safety claim?

> I have no idea.

**Q38: Is this preventing bugs or merely moving them to type annotations?**

>> I have no idea.

**Q39: How does this compare to existing "checked arithmetic" in practice?**

> I have no idea.

---

## 9. Language Design & Adoption

**Q40: Why hasn't this already won historically?**  
Pascal and Ada tried parts of this—why did it not generalize?

> I have no idea.

**Q41: What is the migration path from existing integer-heavy codebases?**

> Slow adoption of a more safe and readable type, especially when the maximum and minimum values are known to be in a certain range, but the values aren't.

**Q42: Does this favor safety languages and penalize performance languages?**

> Yes at Compile time, at Runtime both should result in the same performance, due to Exhaustiveness Checking at compile time.

**Q43: Would most developers actually use this, or just default to `range()`?**

> `range()` would actually be an IDE error as it's lacking `stop` input.

---

## 10. Evaluation & Evidence

**Q44: Where is the empirical evidence?**
- Code size
- Performance
- Bug reduction

> This is a concept paper, a proposal not the final feature, I want to publish this to get feedback and then build it.

**Q45: What benchmarks demonstrate real-world benefit?**

> This is a concept paper, a proposal not the final feature, I want to publish this to get feedback and then build it.

**Q46: What is the minimal example where this clearly beats existing approaches?**

> "We need to represent years from 1900 to 2100"  
> `year: int = 2005` and then the developer has to check and guarantee that it always stays in the 1900 to 2100 interval.  
> `year: range(1900, 2100) = 2005` the type guarantees it can't be 1800 or 2200, it's also stored in 1 byte instead of 2 bytes.

**Q47: What is the worst-case example where this becomes unusable?**

> "Receiving surprising dynamic data"  
> Data that is inherently unbounded and has no 'end' is difficult to represent, as would either require requesting more memory which goes against the idea of range() unless range adds the idea of INFINITE which also creates many problems. This `user_id: range(0, 1_000_000) = get_next_id()` would be easier by just using integer `user_id: int = get_next_id()`.

---

## 11. Scope & Claims

**Q48: Is this a language feature, a library, or a type-system extension?**  
The paper must pick one.

> A language feature, a primitive.

**Q49: What problems does this *not* solve?**

> It creates problems and does not solve:
> - Different languages would have different syntax for the same type
> - Developers would need to learn a new way of thinking about numbers
> - Creates a new primitive that somehow is dependent on the int primitive for passing values?

**Q50: What would cause you to reject your own paper?**

> If I realized that this is not a primitive and is better represented as an unbounded Integer.
