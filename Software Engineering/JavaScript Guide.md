# JavaScript Guide

## Functions

### Flexible Function Arity

**Arity** is the number of parameters a function declares. The term comes from the ending *-ary* in words such as *unary* (one), *binary* (two), and *ternary* (three), combined with *-ity*.

JavaScript has **flexible function arity**: which means it allows a call even if you have supply fewer or more arguments than the parameters in the function definition.

- **Missing arguments:** the corresponding parameters are initialized to `undefined`, unless they have default values.
- **Extra arguments:** arguments beyond the named parameters are unused unless the function accesses them through a rest parameter (`...args`) or, in regular functions, the `arguments` object.

For example, a function that multiplies a number by a factor:

```js
function multiply(number, factor = 2) {
  return number * factor;
}

multiply(5, 3);     // 15 — both arguments supplied
multiply(5);        // 10 — factor defaults to 2
multiply(5, 3, 4);  // 15 — the third argument is unused
multiply();         // NaN — number is undefined, so undefined * 2 yields NaN
```
