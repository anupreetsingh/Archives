# Interpreting Signed Binary Numbers in Two's Complement

1. Check the MSB (Most Significant Bit):

   - If MSB == 0:
       - → The number is non-negative.
       - → Just read the bits as normal binary to get the value.

   - If MSB == 1:
       - → The number is negative.
       - → The bits are stored in two's complement form.
       - → To find its absolute value:
           - a) Invert all the bits (one's complement).
           - b) Add 1 to the result.
           - c) The result is the magnitude (absolute value).
       - → Apply a negative sign to that magnitude.

## Example (8-bit)

```text
00001011 → MSB=0 → +11
11110101 → MSB=1 → invert(00001010), add 1 → 00001011 = 11 → -11
```

## Value Range

- For unsigned representation, if you have n bits to represent an integer the maximum possible value is `2^n - 1`, giving range: `0 to 2^n - 1`.
- For signed representation, since you use the MSB for sign (it's 1 less bit available):
  - For n bits, range is `-2^(n-1) ... +2^(n-1)-1`
  - Example: 8-bit range = `-128 ... +127`
