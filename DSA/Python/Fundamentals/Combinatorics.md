# Combinatorics

## Core Counting Rules

Counting rules are the foundation. Permutations and combinations are built from these rules.

### Product Rule

Use the product rule when a process has multiple stages, each stage has a certain number of choices, and one choice must be made at every stage to produce a complete outcome.

Use the product rule for "this and then that" choices.

Example:

```text
If a password has:

- 3 choices for the first character
- 4 choices for the second character
- 2 choices for the third character

total passwords = 3 * 4 * 2 = 24
```

### Sum Rule

Use the sum rule when there are multiple separate alternatives, each alternative has multiple choices, and each complete outcome comes from choice of exactly one of the alternatives.

Use the sum rule for "either/or" choices.

If:

- Option A has `a` choices
- Option B has `b` choices

Then the total number of choices is:

```text
a + b
```

Example:

```text
If you can choose either:

- 5 tea options
- 3 coffee options

total drink choices = 5 + 3 = 8
```

### Subtraction Rule

Use the subtraction rule when it is easier to count all possible outcomes and subtract the invalid ones.

This is also called the complement idea.

```text
valid outcomes = total outcomes - invalid outcomes
```

Example:

```text
How many 3-digit strings from `000` to `999` contain at least one `7`?

total 3-digit strings = 10 * 10 * 10 = 1000
without 7 in each position = 9 * 9 * 9 = 729
with at least one 7 = 1000 - 729 = 271
```

### Inclusion-Exclusion

Use inclusion-exclusion when two counted groups overlap.

For two sets:

```text
count(A or B) = count(A) + count(B) - count(A and B)
```

The overlap is subtracted once because it was counted twice.

Example:

In numbers from `1` to `30`:

- Multiples of `2`: `30 // 2 - 0 // 2 = 15`
- Multiples of `3`: `30 // 3 - 0 // 3 = 10`
- Multiples of both `2` and `3`, meaning multiples of `6`: `30 // 6 - 0 // 6 = 5`

```text
multiples of 2 or 3 from 1 to 30 = 15 + 10 - 5 = 20
```

**Example: Counting Multiples**

To count multiples of `k` in an inclusive range `[low, high]`:

```text
count = high // k - (low - 1) // k
```

Why this works:

- `high // k` counts multiples of `k` from `1` to `high`.
- `(low - 1) // k` counts multiples of `k` before the range starts.
- Subtracting them leaves only the multiples inside `[low, high]`.

## Factorial

For a positive integer `n`, `n!` (read "`n` factorial") is a compact way to write the product made by multiplying `n` with the factorial of the next smaller integer.

```text
n! = n * (n - 1)!
   = n * (n - 1) * (n - 2)!
   .
   .
   .
   = n * (n - 1) * (n - 2) * ... * 3 * 2!
   = n * (n - 1) * (n - 2) * ... * 3 * 2 * 1!
   = n * (n - 1) * (n - 2) * ... * 3 * 2 * 1 * 0!
   = n * (n - 1) * (n - 2) * ... * 3 * 2 * 1 * 0!
   = n * (n - 1) * (n - 2) * ... * 3 * 2 * 1 * 1
   = n * (n - 1) * (n - 2) * ... * 3 * 2 * 1

```

Since factorial is built recursively, each factorial is the current number multiplied by the factorial of the next smaller number.

```text
n! = n * (n - 1)!
```

`0! = 1` is the base value chosen to make the rules of factorial consistent.

It makes counting like this valid:

```text
0! = 1 (base value pre set)
1! = 1 * 0! = 1 * 1 = 1
2! = 2 * 1! = 2 * 1 * 0! = 2 * 1 * 1 = 2
3! = 3 * 2! = 3 * 2 * 1! = 3 * 2 * 1 * 0! = 3 * 2 * 1 * 1 = 6
```

Another reason `0! = 1` is chosen as the base case is that it stops the recursive definition before it crosses `0` into negative numbers.

If the recursion tried to continue past `0!` and used a negative factorial value as the base case, it would introduce `0` as a multiplier:

```text
2! = 2 * 1 * 0 * (-1)!
```

That would make the whole product `0`, so trying to extend the same recursive rule into negative integers would break the factorial values that already work for nonnegative integers.

## Permutations

A permutation is an ordered arrangement.

Example:

```text
ABC and BAC are different permutations.
```

### Permutation for `n` Distinct Items

Calculating the number of permutations possible when filling `n` ordered positions from `n` distinct items, without repeating any choice.

This can alternatively be thought of as calculating number of possible arrangements for `n` distinct items.

#### Formula Derivation

```text
first position:  n choices
second position: n - 1 choices   (one item already used)
third position:  n - 2 choices   (two items already used)
...
last position:   1 choice        (only one item left)
```

By the product rule:

```text
total = n * (n - 1) * (n - 2) * ... * 2 * 1 
total = n!
```

#### Example

How many ways can `A`, `B`, and `C` be arranged?

```text
3! = 3 * 2 * 1 = 6
```

```text
ABC
ACB
BAC
BCA
CAB
CBA
```

### Permutation for `n` Items With Duplicates

Calculating the number of permutations possible when filling `n` ordered positions from `n` items, where some items are identical.

This can alternatively be thought of as first arranging all `n` items as if they were distinct, then adjusting for the extra counts created by swapping identical copies.

#### Formula Derivation

The `n!` count assumes every item is distinct.

If some items are identical, `n!` overcounts because swapping identical copies does not create a new arrangement.

First pretend every copy is distinct.

If one repeated group has size `a`, then the copies inside that group can be internally rearranged in:

```text
a!
```

ways that do not create new actual arrangements.

So each actual arrangement is counted `a!` extra times for that repeated group.

If there are multiple repeated groups with sizes `a`, `b`, `c`, and so on, each group creates its own internal factorial overcount.

Divide by all of those overcounts:

```text
total = n! / (a! * b! * c! * ...)
```

- `n` is the total number of items
- `a`, `b`, `c`, etc. are the sizes of repeated groups

#### Example

How many distinct arrangements can be made from `A`, `A`, and `B`?

```text
A A B
```

Temporarily label the repeated `A`s:

```text
A1 A2 B
```

If all three items were distinct, there would be:

```text
3! = 6
```

labeled arrangements:

```text
A1 A2 B
A2 A1 B
A1 B A2
A2 B A1
B A1 A2
B A2 A1
```

But after removing the labels, pairs collapse into the same actual arrangement:

```text
A1 A2 B -> A A B
A2 A1 B -> A A B

A1 B A2 -> A B A
A2 B A1 -> A B A

B A1 A2 -> B A A
B A2 A1 -> B A A
```

The two `A`s can be arranged among themselves in `2!` ways, but those internal arrangements do not matter because the copies are identical.

So each actual arrangement was counted `2!` times.

Divide by the overcount:

```text
actual arrangements = 3! / 2!
actual arrangements = 6 / 2
actual arrangements = 3
```

The actual arrangements are:

```text
AAB
ABA
BAA
```

### Permutations for `k` position with `n` Distinct options(No Repititions)

Calculating the number of permutations possible when filling `k` ordered positions from `n` distinct options, without repeating any choice.

This is written as:

```text
nPk or P(n, k)
```

#### Formula Derivation

For each position, one fewer item is available because previous choices cannot be reused.

```text
1st position: n choices
2nd position: n - 1 choices
3rd position: n - 2 choices
...
kth position: n - (k - 1) = n - k + 1 choices
```

So by the product rule:

```text
nPk = n * (n - 1) * (n - 2) * ... * (n - k + 1)
```

Use factorials to write this product compactly.

```text
n! = n * (n - 1) * (n - 2) * ... * (n - k + 1) * (n - k) * ... * 2 * 1

(n - k)! = (n - k) * (n - k - 1) * ... * 2 * 1

n! / (n - k)! = n * (n - 1) * (n - 2) * ... * (n - k + 1)
```

The denominator `(n-k)!` removes the unused tail of `n!` to make it equivalent to the expression derived using product rule logic.

So:

```text
nPk = n! / (n - k)!
```

#### Example

How many 2-letter ordered arrangements can be made from `A`, `B`, `C`, `D`?

```text
n = 4
k = 2

4P2 = 4! / (4 - 2)!
4P2 = 4! / 2!
4P2 = 24 / 2
4P2 = 12
```

```text
AB AC AD
BA BC BD
CA CB CD
DA DB DC
```

### Permutations for `k` Positions From `n` Distinct Items (With Repetition)

Calculating the number of permutations possible when filling `k` ordered positions from `n` distinct options, with repetition of choice allowed.

Which can alternatively be thought of as making `k` independent ordered choices, where each position has the same `n` distinct options.

#### Formula Derivation

Since repetition is allowed, choosing an item for one position does not remove it from later positions.

```text
first position:  n choices
second position: n choices
third position:  n choices
...
kth position:    n choices
```

So by the product rule:

```text
n * n * n * ... * n
```

This is `n` multiplied by itself `k` times, so:

```text
n^k
```

#### Example

How many 3-letter ordered arrangements can be made from `A`, `B`, `C`, `D` if letters can repeat?

```text
n = 4
k = 3

n^k = 4^3
n^k = 64
```

Examples of valid arrangements:

```text
AAA
AAB
ABA
BAA
DDD
```

## Combinations

A combination is an unordered selection.

Example:

```text
AB and BA are the same combination.
```

Because order does not matter, the main question is not how many arrangements we can make, but how many distinct selections we can form.

### Combinations for `k` Items From `n` Distinct Items (No Repetition)

This counts how many ways we can select `k` items from `n` distinct items without repeating any selection.

Which can alternatively be thought of as first choosing and arranging `k` items from `n`, then removing the extra counts created by ordering the chosen items.

This is written as:

```text
nCk or C(n, k)
```

#### Formula Derivation

Start with the permutation formula for choosing `k` items from `n` distinct items without repetition.

```text
nPk = n! / (n - k)!
```

This counts ordered arrangements.

But combinations do not care about order.

For any selected group of `k` distinct items, those same `k` items can be arranged internally in:

```text
k!
```

different orders.

Those `k!` orders all represent the same combination(choice of `k` elements).

So each actual combination is counted `k!` extra times by the permutation formula.

Divide by the overcount:

```text
nCk = nPk / k!
```

Substitute the permutation formula:

```text
nCk = (n! / (n - k)!) / k!
nCk = n! / ((n - k)! * k!)
```

#### Example

How many 2-item unordered selections can be made from `A`, `B`, `C`, `D`?

```text
n = 4
k = 2

4C2 = 4! / (2! * (4 - 2)!)
4C2 = 4! / (2! * 2!)
4C2 = 24 / (2 * 2)
4C2 = 6
```

The combinations are:

```text
AB
AC
AD
BC
BD
CD
```

Notice that `AB` and `BA` are not listed separately because they represent the same combination, meaning the same unordered selection.

#### Binomial Coefficient

A **monomial** is an algebraic expression with exactly one term, such as `x`, `y`, `x^2y`, or `3x`.

A **binomial** is an algebraic expression with exactly two terms joined by addition or subtraction, such as `x + y`, `x^2 + y`, `2x + y`, or `a - b`.

A **binomial expansion** is the expanded form of a binomial raised to a power, such as `(a + b)^n`.

The value `nCk` is also called a **binomial coefficient** because it counts how many times a particular monomial appears during a binomial expansion.

To see why, start with the general multiplication.

For `(x + y)^n`, the exponent `n` means there are `n` copies of `(x + y)` being multiplied:

```text
(x + y)^n = (x + y)(x + y)(x + y) ... (x + y)
```

Each full copy of `(x + y)` is one factor:

```text
factor 1   factor 2   factor 3         factor n
(x + y)   (x + y)   (x + y)   ...   (x + y)
```

The distributive property for multiplication works here by taking one term from each factor and multiplying those chosen terms together. The final form of a monomial in the resulting expression depends on which term is chosen from each factor.

In this context, saying a factor **contributes** `x` or **contributes** `y` means that `x` or `y` is the term chosen from that factor.

If exactly `k` of the `n` factors contribute `y`, then the remaining `n - k` factors contribute `x`.

That creates this monomial:

```text
x^(n-k)y^k
```

So the coefficient of `x^(n-k)y^k` depends on how many ways there are to choose `k` factors (that contribute `y`) from `n` total factors, which is `nCk`.

Therefore, the coefficient of `x^(n-k)y^k` in `(x + y)^n` is `nCk`.

Written as a general expansion:

```text
(x + y)^n = (nC0)x^n + (nC1)x^(n-1)y + ... + (nCk)x^(n-k)y^k + ... + (nCn)y^n
```

Now apply that idea to `(x + y)^4`:

```text
(x + y)^4 = (x + y)(x + y)(x + y)(x + y)
```

For the `x^3y` term, `n = 4` and `k = 1`, so exactly 1 of the 4 factors must contribute `y`, and the other 3 factors must contribute `x`.

So the counting question is:

```text
How many ways are there to choose 1 of the 4 factors to contribute y?
```

That question is answered by `4C1` because it asks for the number of ways to choose 1 factor, from 4 total factors, to contribute `y`.

```text
4C1 = 4
```

There are 4 possible choices:

```text
y x x x -> yx^3 = x^3y
x y x x -> xyx^2 = x^3y
x x y x -> x^2yx = x^3y
x x x y -> x^3y
```

These are different choices of which factor contributes `y`, but they all simplify to the same monomial, `x^3y`. Therefore, the coefficient of `x^3y` is `4`.

For `(x + y)^4`, `n = 4`, so `k` can be `0`, `1`, `2`, `3`, or `4`. That is why the expansion uses `4C0` through `4C4`:

- `4C0` means choose 0 factors to contribute `y`, so all 4 factors contribute `x`, giving `x^4`.
- `4C1` means choose 1 factor to contribute `y`, giving `x^3y`.
- `4C2` means choose 2 factors to contribute `y`, giving `x^2y^2`.
- `4C3` means choose 3 factors to contribute `y`, giving `xy^3`.
- `4C4` means choose 4 factors to contribute `y`, giving `y^4`.

Example:

```text
(x + y)^4 = (x + y)(x + y)(x + y)(x + y)
          = (4C0)x^4 + (4C1)x^3y + (4C2)x^2y^2 + (4C3)xy^3 + (4C4)y^4
          = x^4 + 4x^3y + 6x^2y^2 + 4xy^3 + y^4
```

#### Subsets

A subset is an unordered selection of elements from a set with no repititions. Since a set has distinct elements.

So a subset of size `k` is basically choosing `k` elements from a set of `n` distinct elements with no repitition .i.e. `nCk`.

```text
number of size-k subsets = nCk = n! / (k! * (n - k)!)
```

The **power set** of a set $S$, denoted by $\mathcal{P}(S)$, is the set containing **all possible subsets** of $S$, including the empty set $\emptyset$ and $S$ itself.

If $S$ contains $n$ distinct elements, then the total number of subsets in the power set is

$$
|\mathcal{P}(S)|
= nC0 + nC1 + nC2 + \cdots + nCn
= \sum_{k=0}^{n} nCk
= 2^n
$$

So, a set with $n$ elements has exactly $2^n$ subsets.

That sum equals `2^n`. Here is why, two ways:

**Way 1: the binomial theorem (`x = y = 1`).**

The expansion just derived holds for any `x` and `y`:

```text
(x + y)^n = (nC0)x^n + (nC1)x^(n-1)y + ... + (nCn)y^n
```

Set `x = 1` and `y = 1`. Every `x^(n-k)y^k` becomes `1`, so each term is just its coefficient:

```text
(1 + 1)^n = nC0 + nC1 + ... + nCn
```

The left side is `2^n`, so:

```text
nC0 + nC1 + ... + nCn = 2^n
```

**Way 2: decide each element independently (product rule).**

Build a subset by walking the `n` elements and, for each one, deciding *include* or *exclude*:

```text
element 1: 2 choices (include or exclude)
element 2: 2 choices
...
element n: 2 choices
```

Each distinct include/exclude pattern is a distinct subset, so by the product rule:

```text
total subsets = 2 * 2 * ... * 2   (n times) = 2^n
```

##### Example

All subsets of `{A, B, C}`:

```text
size 0:  {}                       3C0 = 1
size 1:  {A}   {B}   {C}          3C1 = 3
size 2:  {A,B} {A,C} {B,C}        3C2 = 3
size 3:  {A,B,C}                  3C3 = 1

total = 1 + 3 + 3 + 1 = 8 = 2^3
```

The same 8 subsets, listed as the in/out decisions from way 2:

```text
A B C
0 0 0  -> {}
1 0 0  -> {A}
0 1 0  -> {B}
0 0 1  -> {C}
1 1 0  -> {A,B}
1 0 1  -> {A,C}
0 1 1  -> {B,C}
1 1 1  -> {A,B,C}
```

Each element contributes one binary digit, `n` digits give `2^n` patterns, and each pattern is one subset.

### Combinations for `k` Items From `n` Types (With Repetition)

Calculating the number of combinations possible when choosing `k` items from `n` distinct types, where repetition is allowed and order does not matter.

Which can alternatively be thought of as deciding how many times each type is chosen.

Example:

```text
AAB, ABA, and BAA are the same combination.
```

They all mean:

```text
2 copies of A
1 copy of B
```

#### Formula Derivation

With repetition allowed and order ignored, the question becomes:
"How many times is each type selected?"

For example, choosing 3 items from the types `A`, `B`, `C`, and `D` can be represented as counts:

```text
A count | B count | C count | D count
```

If the selection is:

```text
A A C
```

then the counts are:

```text
A count = 2
B count = 0
C count = 1
D count = 0
```

This can be represented using stars and bars:

```text
**||*|
```

Meaning:

```text
2 stars before the first bar  -> 2 A's
0 stars before the second bar -> 0 B's
1 star before the third bar   -> 1 C
0 stars after the third bar   -> 0 D's
```

To choose `k` total items from `n` types:

- Use `k` stars to represent the chosen items.
- Use `n - 1` bars to separate the `n` types.

So there are `k + (n - 1)` total symbols to choose from.

The purpose of thinking about the original problem as a stars-and-bars layout is to turn a count-based selection problem into a position-selection problem, so that the combination formula can be applied.

Choosing where the `k` stars go in the stars-bar layout determines the which selection is made.

So:

```text
total = C(n + k - 1, k) = (n + k - 1)! / ((n - 1)! * k!)
```

Equivalently, choosing where the `n - 1` bars go gives:

```text
total = (n + k - 1)C(n - 1)
```

Both formulas count the same thing: the number of unordered selections of k items from n types when repetition is allowed

#### Example

How many 2-item unordered selections can be made from `A`, `B`, `C` if repetition is allowed?

```text
n = 3
k = 2

(n + k - 1)Ck = (3 + 2 - 1)C2
(n + k - 1)Ck = 4C2
4C2 = 4! / (2! * (4 - 2)!)
4C2 = 4! / (2! * 2!)
4C2 = 24 / (2 * 2)
4C2 = 6
```

The combinations are:

```text
AA
AB
AC
BB
BC
CC
```

## Subsets
