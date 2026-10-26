# Chunk 1: Set Theory Notation

> Personal notes, not a solution set. See the [README](README.md) before reading.

First chunk of RareSkills' ZK-SNARKs from First Principles. Four lessons: sets, subsets, `∈`, set builder notation.

My first reaction was "this is discrete maths from uni." And it is. But this time I kept asking a different question: where does this already show up in ZK?

Turns out, almost everywhere.

---

## 1. Sets

A set is a collection of things in curly braces. `{1, 2, 3}`. The things inside are elements.

Three rules:

- **It has to be well-defined.** For anything you pick up, there's a clear yes/no answer to "is this in the set?" "Whole numbers below 5" works. "Hard math topics" doesn't, because who decides what's hard?
- **Order doesn't matter.** `{1, 2, 3}` and `{3, 1, 2}` are the same set.
- **Duplicates don't count.** `{2, 4, 4, 6}` is just `{2, 4, 6}`.

Plus the empty set, `∅` or `{}`, and size, written `|A|`. So `|{5, 9, 10}| = 3`.

### Where this shows up in ZK

**Well-defined is basically what a circuit is.** A circuit takes a witness and answers one question: do these values satisfy every constraint, yes or no? That defines a set, the set of valid witnesses.

An under-constrained circuit is a set that isn't defined the way you meant. The constraints let in values you never intended, and a malicious prover can generate a valid proof with one of them. This is the most common class of ZK circuit bug, and in set language, the bug is a sloppy membership rule.

**Order doesn't matter, but hashes care about order.** `{a, b} = {b, a}`, but `hash(a, b) ≠ hash(b, a)`. Hash inputs are not sets, they're ordered tuples (more on those in section 4). If the client code and the circuit disagree on argument order, the values never match and the proof fails.

### Mistakes I made here

- Asked for `|∅|`, I answered "the empty set." Wrong question answered. `∅` is a set, `|∅|` is a number about that set: **0**. Later in the course there's `p` vs `|𝔽_p|`, a polynomial vs its degree, and so on. Object vs fact-about-object keeps coming back.
- `|{a, {a}}|` is **2**. The outer set has two elements: `a`, and the set `{a}`. You only count the top level. A bag with a pen and a pencil case in it holds two things, no matter what's in the pencil case.

---

## 2. Subsets

`A ⊆ B` means **every element of A is also in B**.

```
{1, 2} ⊆ {1, 2, 3}     yes
{1, 4} ⊆ {1, 2, 3}     no, 4 isn't in there
```

- Every set is a subset of itself. `⊆` works like `≤`.
- **Proper subset**, `A ⊊ B`: A ⊆ B and B has something extra. Works like `<`. (Some books write this `⊂`. Others use `⊂` to mean `⊆`. Check which one you're reading.)
- `∅` is a subset of everything. To break `∅ ⊆ B` you'd have to find something in `∅` that isn't in `B`. There's nothing in `∅` to find. So it holds automatically.
- `A = B` exactly when `A ⊆ B` **and** `B ⊆ A`.

The one I tripped on: `⊆` only goes one direction. Knowing everything in A is in B says nothing about whether B has extras. I kept jumping straight to "equal." Equality needs both directions.

### Where this shows up in ZK

**Soundness and completeness are two subset statements.** This was the biggest click of the chunk for me.

Take two sets:

- **Intended**: inputs where the statement is actually true
- **Accepted**: inputs that satisfy the circuit's constraints

| Statement | Meaning | Name |
|---|---|---|
| Intended ⊆ Accepted | every honest input can produce a proof | completeness |
| Accepted ⊆ Intended | nothing false can produce a proof | soundness |
| Both | Intended = Accepted | a correct circuit |

- **Under-constrained circuit**: Intended ⊊ Accepted. Extra stuff gets in. Security bug.
- **Over-constrained circuit**: Accepted ⊊ Intended. Honest users get rejected. Broken product.

Same `A = B ⟺ A ⊆ B and B ⊆ A` rule from above, just with real consequences.

### Counting subsets

A set with `n` elements has `2ⁿ` subsets. For each element you make one decision: in or out. Each new element doubles the count.

I struggled with this one for a while. What finally made it click was packing a bag from `{mango, banana, orange}`. Go to each fruit and decide: take it or leave it.

```
1 fruit   → 2 bags
2 fruits  → 4 bags
3 fruits  → 8 bags
```

Every bag you already had now comes in two versions, with the new fruit and without. Draw it as a tree and each path to the bottom is one subset.

The same tree shows up in Merkle proofs. To climb from a leaf to the root, the leaf's index is turned into bits, and at each level one bit decides: left or right. A tree with 16 levels makes 16 binary decisions, so it has `2¹⁶` possible leaves. Same doubling.

---

## 3. `∈` notation

`x ∈ A` means "x is an element of A." `x ∉ A` means it isn't.

The difference from `⊆`:

| Symbol | Left side | Right side | Asks |
|---|---|---|---|
| `∈` | one element | a set | is this thing in there? |
| `⊆` | a set | a set | is everything here also there? |

```
3 ∈ {1, 2, 3}       true
{3} ⊆ {1, 2, 3}     true
3 ⊆ {1, 2, 3}       doesn't make sense, 3 isn't a set
{2} ∈ {1, 2, 3}     false, {2} isn't one of the elements
```

The last one is a valid statement that just happens to be false. Sets can be elements. `{a} ∈ {a, {a}}` is true.

The standard number sets:

| Symbol | Name | Contains |
|---|---|---|
| ℕ | natural numbers | 1, 2, 3, ... (no negatives, no decimals) |
| ℤ | integers | ..., −2, −1, 0, 1, 2, ... (no decimals) |
| ℝ | real numbers | everything above, plus 0.5, π, 7.25, ... |

`ℕ ⊆ ℤ ⊆ ℝ`. Decimals only show up at ℝ.

### Where this shows up in ZK

**A Merkle membership proof is an `∈` statement.** "This leaf ∈ the set committed to by this root." The ZK part is proving it without revealing which element you are. The verifier learns that `∈` holds, and nothing about where.

**`x: Field` in Noir means `x ∈ 𝔽_p`.** Not ℤ, not ℝ. A specific finite set.

So "which set is this value from?" is a real debugging question. A JavaScript float like `0.7234` is in ℝ, but `0.7234 ∉ 𝔽_p`. Feeding it into code that expects a field element gives garbage.

### Mistakes I made here

- Put a negative number in ℕ. No. Natural numbers are counting numbers. `−4 ∈ ℤ`, `−4 ∉ ℕ`.
- Picked the wrong number set for a condition and ended up with an empty set without noticing. The set you pick decides everything.

---

## 4. Set builder notation

Describing a set with a rule instead of a list.

```
{x ∈ ℤ | x > 3}      or      {x : x > 3 and x ∈ ℤ}
```

"All x in ℤ such that x > 3." `|` and `:` both mean "such that." Commas mean "and."

An element has to pass **every** condition, including where it comes from:

```
{x ∈ ℕ | x ≤ 5}               = {1, 2, 3, 4, 5}     −1 ≤ 5, but −1 ∉ ℕ, so it's out
{x ∈ ℤ | −2 < x < 2}          = {−1, 0, 1}          strict < leaves out −2 and 2
{x ∈ ℤ | 2 ≤ x ≤ 8, x even}   = {2, 4, 6, 8}
{x ∈ ℤ | 3 < x < 4}           = ∅                   no integer fits
{x ∈ ℝ | 3 < x < 4}           3.1, 3.5, π, ...      same condition, different set
```

The last two lines have the exact same condition. Only the "where it comes from" part changed, and one is empty while the other is infinite.

### Where this shows up in ZK

**A circuit is set builder notation.** Honestly, this is the cleanest mapping in the whole chunk.

```
{ w ∈ 𝔽_p^n  :  constraint₁(w), constraint₂(w), ... }
```

- `𝔽_p^n` is the "where it comes from" part. The witness is made of field elements.
- Each `assert` in Noir is one condition after the colon.
- The commas are "and." Every constraint has to hold.

So the set of valid witnesses is literally a set-builder set. Writing a circuit is writing the conditions.

### Roots

`{x : f(x) = 0}` is the set of inputs where `f` **outputs** 0. I misread `f(x) = 0` as "x = 0" at first. It's about the output, not the input.

| f(x) | Inputs giving 0 |
|---|---|
| x − 5 | {5} |
| x² − 4 | {−2, 2} |
| x² + 1 | ∅ over ℝ, since x² + 1 is always at least 1 |

Why this matters: constraints are usually written as "something = 0." Take this one:

```
x · (x − 1) = 0
```

Its roots are `{0, 1}`. That's how a circuit forces a value to be a bit. Types like `u1` in Noir are the friendly version of this constraint.

And if a constraint system's set of roots is empty, like `x² + 1` above, then no valid witness exists and no proof can ever be generated.

Vanishing polynomials (Chunk 21) are built on exactly this idea, so this one is worth holding on to.

### Sets of tuples

Elements can be ordered pairs:

```
{(x, y) : y = 2x, x ∈ ℤ, y ∈ ℤ}
```

`(1, 2)` and `(−3, −6)` are in. `(2, 1)` is out, because 1 ≠ 2·2. `(0.5, 1)` is out, because 0.5 isn't an integer.

`(1, 2)` in and `(2, 1)` out is the whole point: tuples are ordered. That's the hash argument order point from section 1. Hash inputs are tuples, not sets.

---

## What stuck

- Read symbols the way you'd trace code. When I was confused, going back to the literal definition worked every time. Intuition didn't.
- "Which set is this value from?" is a debugging question, not just a math one.
- Soundness and completeness are just `⊆` in two directions.
- A circuit defines a set. Constraints are the membership rule.
