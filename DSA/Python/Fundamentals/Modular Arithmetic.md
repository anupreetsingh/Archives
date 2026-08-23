# Modular Arithmetic

Modular arithmetic works with remainders. For a positive modulus $M$, applying the modulo operation to an integer converts it to a representative in the range

$$
0, 1, 2, \ldots, M-1.
$$

In other words, for every integer $a$, there are unique integers $q$ and $r$ such that

$$
a=qM+r, \qquad 0\le r<M.
$$

Here, $qM$ is an integer multiple of $M$, while $r$ is the part left over. Subtracting the remainder gives

$$
a-r=qM.
$$

Therefore, $a-r$ is divisible by $M$. The modulo operation removes the complete-multiple part $qM$ and returns the remainder $r$:

$$
a\bmod M=r.
$$

For example, with modulus $7$:

$$
17\bmod 7=3
$$

because $17=2(7)+3$. Negative integers are also represented by a number in the same range:

$$
-4\bmod 7=3
$$

because $-4=-1(7)+3$.

Python follows this convention when $M$ is positive.

```python
17 % 7   # 3
-4 % 7   # 3
```

## Congruence

Two integers are **congruent modulo $M$** when the modulo operation converts both of them to the same representative. We write

$$
a\equiv b\pmod M.
$$

The following statements are equivalent:

$$
a\equiv b\pmod M
$$

$$
a\bmod M=b\bmod M
$$

In the opening example, $17\bmod7=3$ and $3\bmod7=3$. Therefore, $17\equiv3\pmod7$.

Congruence therefore groups all integers with the same remainder into one equivalence class. Modular arithmetic operates on these classes rather than on the original integers.

## Addition, Subtraction, and Multiplication

Modulo can be applied before or after addition, subtraction, and multiplication:

$$
(a+b)\bmod M
=\big((a\bmod M)+(b\bmod M)\big)\bmod M,
$$

$$
(a-b)\bmod M
=\big((a\bmod M)-(b\bmod M)\big)\bmod M,
$$

and

$$
(ab)\bmod M
=\big((a\bmod M)(b\bmod M)\big)\bmod M.
$$

This is useful because the operands can be reduced to numbers between $0$ and $M-1$ before performing a larger calculation.

For example, using $17\equiv3\pmod7$ and $12\equiv5\pmod7$:

$$
(17+12)\bmod7=(3+5)\bmod7=1,
$$

$$
(17-12)\bmod7=(3-5)\bmod7=5,
$$

and

$$
(17\cdot12)\bmod7=(3\cdot5)\bmod7=1.
$$

### Why the Operations Can Be Split

Let

$$
a=q_1M+r_1
\qquad\text{and}\qquad
b=q_2M+r_2,
$$

where $r_1=a\bmod M$ and $r_2=b\bmod M$.

For addition,

$$
a+b=(q_1+q_2)M+(r_1+r_2).
$$

The term $(q_1+q_2)M$ is a multiple of $M$, so it contributes a remainder of $0$. Therefore,

$$
a+b\equiv r_1+r_2\pmod M.
$$

For subtraction,

$$
a-b=(q_1-q_2)M+(r_1-r_2),
$$

so the same reasoning gives

$$
a-b\equiv r_1-r_2\pmod M.
$$

For multiplication,

$$
\begin{aligned}
ab
&=(q_1M+r_1)(q_2M+r_2)\\
&=q_1q_2M^2+q_1Mr_2+q_2Mr_1+r_1r_2\\
&=M(q_1q_2M+q_1r_2+q_2r_1)+r_1r_2.
\end{aligned}
$$

Every term except $r_1r_2$ contains a factor of $M$. Hence,

$$
ab\equiv r_1r_2\pmod M.
$$

The three proofs all rely on the same fact: adding or removing a multiple of the modulus does not change a number's remainder.

## Division and Modular Inverses

Division is different because it cannot generally be split across modulo:

$$
\frac{a}{b}\bmod M
\ne
\frac{a\bmod M}{b\bmod M}.
$$

In normal arithmetic, the multiplicative inverse $a^{-1}$ of a nonzero number $a$ is defined by

$$
a\cdot a^{-1}=1.
$$

Solving for the inverse gives the ordinary reciprocal

$$
a^{-1}=\frac{1}{a}.
$$

In modular arithmetic, $a^{-1}$ has a related but different definition. It is a number satisfying

$$
a\cdot a^{-1}\equiv1\pmod M,
$$

or equivalently,

$$
(a\cdot a^{-1})\bmod M=1\bmod M.
$$

The product does not have to equal $1$ exactly. It can equal $1$ plus any integer multiple of $M$ for some integer $k$.:

$$
a\cdot a^{-1}=1+kM
$$

Consequently, a modular inverse is generally an integer other than the ordinary reciprocal $1/a$.

For example, the ordinary inverse of $3$ is $1/3$, but the modular inverse of $3$ modulo $7$ is $5$ because

$$
3\cdot5=15=1+2(7),
$$

and therefore

$$
3\cdot5\equiv1\pmod7.
$$

The modular inverse of $3$ is the entire congruence class represented by $5$. Thus, every integer belonging to the class of $x\equiv5\pmod7$, such as $-2$, $5$, or $12$, represents the modular inverse of $3$ because each one satisfies $3x\equiv1\pmod7$.

When $b$ has a modular inverse, division by $b$ modulo $M$ means multiplication by $b^{-1}$:

$$
\frac{a}{b}\equiv a\,b^{-1}\pmod M.
$$

Thus,

$$
\begin{aligned}
\frac{4}{3}
&\equiv4\cdot3^{-1}\pmod7\\
&\equiv4\cdot5\pmod7\\
&\equiv20\pmod7\\
&\equiv6\pmod7.
\end{aligned}
$$

This result means that $6$ solves $3x\equiv4\pmod7$, since $3\cdot6\equiv4\pmod7$.

### When Division Is Possible

The inverse $b^{-1}\pmod M$ exists if and only if

$$
\gcd(b,M)=1.
$$

For $2^{-1}$ to exist in modulo $6$, it would have to satisfy

$$
2\cdot2^{-1}\equiv1\pmod6.
$$

However, multiplying $2$ by any integer produces an even number, whose remainder modulo $6$ can only be $0$, $2$, or $4$—never $1$. Therefore, an integer satifying $2^{-1}$ modulo $6$ does not exist.

Consequently, division by $2$ modulo $6$ is not uniquely defined. For example, the equation

$$
2x\equiv4\pmod6
$$

has both $x\equiv2$ and $x\equiv5$ as solutions.

This is why ordinary cancellation can fail in modular arithmetic:

$$
2\cdot2\equiv2\cdot5\pmod6,
$$

but

$$
2\not\equiv5\pmod6.
$$

The common factor $2$ cannot be cancelled because it has no inverse modulo $6$.

### Prime Moduli and Fermat's Little Theorem

When the modulus is a prime $p$, every integer $a$ not divisible by $p$ has a modular inverse. Fermat's little theorem states that

$$
a^{p-1}\equiv1\pmod p
\qquad\text{when }p\nmid a.
$$

Since

$$
a^{p-1}=a\cdot a^{p-2},
$$

Fermat's little theorem also gives

$$
a\cdot a^{p-2}\equiv1\pmod p.
$$

Therefore,

$$
a^{-1}\equiv a^{p-2}\pmod p.
$$

For $a=3$ and $p=7$,

$$
3^6\equiv1\pmod7
$$

and

$$
3^{-1}\equiv3^5\equiv5\pmod7.
$$

In Python, `pow(a, p - 2, p)` computes this inverse efficiently without first constructing the potentially enormous value $a^{p-2}$:

```python
p = 7
a = 3
inverse = pow(a, p - 2, p)  # 5

result = (4 * inverse) % p   # 4 / 3 (mod 7) = 6
```

The condition $p\nmid a$ is essential. Zero modulo $p$ has no inverse, even when the modulus is prime.
