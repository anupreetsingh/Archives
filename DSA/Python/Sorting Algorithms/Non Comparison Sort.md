# Non Comparison Sort

A non-comparison sort determines the order of elements using properties of their keys, such as integer values or individual digits, rather than comparing pairs of elements to decide their relative order. These properties let it place values into positions or groups directly.

Its efficiency depends on the key range or the number of digits, so a non-comparison sort is not automatically linear for every input. Comparisons used for setup or loop control, such as finding the maximum value, do not make it a comparison sort.

## Counting Sort

Counting sort counts how many times each integer value occurs, then uses those counts to determine where each value belongs in the sorted result.

It is called counting sort because the frequency of each value determines how many positions that value occupies.

In this version, we sort integers into a new list in non-descending order. Each value maps to the count-array index `value - min_value`, so negative integers are also supported. The required range is `k = max_value - min_value + 1`, including values that do not appear in the input.

After counting, we convert the frequencies into cumulative counts. Each cumulative count tells us how many input values are less than or equal to that value, so subtracting one gives its last available output index. After placing an occurrence, we decrease its count to obtain the next available position.

We traverse the input from right to left because we fill each value's output positions from right to left. This preserves the original relative order of equal values, making the implementation stable.

For `[21, 13, 21, 12, 13]`, the range runs from `12` through `21`. The nonzero frequencies are `12: 1`, `13: 2`, and `21: 2`. Their cumulative counts are `1`, `3`, and `5`, so the last available indices for those values are initially `0`, `2`, and `4`.

- Time Complexity: O(n + k), because we scan the n values and the k count-array entries.
- Space Complexity: O(n + k), because we allocate an output list of n values and a count array of size k.

Counting sort works well when the integer range is small relative to the input size. A few values spread across a huge range would require a large count array even though most entries would remain zero.

```python
def counting_sort(nums):
    if not nums:
        return []

    min_value = min(nums)
    max_value = max(nums)
    k = max_value - min_value + 1
    counts = [0] * k

    # Map each integer to an index starting at zero.
    for num in nums:
        counts[num - min_value] += 1

    # Convert frequencies into cumulative counts.
    for i in range(1, k):
        counts[i] += counts[i - 1]

    res = [0] * len(nums)
    for num in reversed(nums):
        idx = num - min_value
        counts[idx] -= 1
        res[counts[idx]] = num

    return res


print(counting_sort([21, 13, 21, 12, 13]))  # [12, 13, 13, 21, 21]
```

## Radix Sort

Radix sort orders values one digit position at a time. Each pass groups values by the digit at the current position, and the passes together establish the order of the complete values.

It is called radix sort because the radix is the number base: decimal digits use radix 10, while binary digits use radix 2.

In this version, we sort nonnegative integers into a new list using least significant digit (LSD) radix sort. We start with the ones digit, then process the tens, hundreds, and subsequent positions until we have processed every digit of the maximum value. Shorter numbers effectively have leading zeros.

Each pass applies the stable placement process from [Counting Sort](#counting-sort) to the current digit. Stability is essential here: when two values have the same current digit, their order from the previously processed lower digits must be preserved. After each pass, the values are sorted by all digit positions processed so far.

Using the same input as above:

| Pass | Result |
| --- | --- |
| Original input | `[21, 13, 21, 12, 13]` |
| Ones digit | `[21, 21, 12, 13, 13]` |
| Tens digit | `[12, 13, 13, 21, 21]` |

During the tens pass, `12` remains before the two `13` values because the ones pass already established their order and all three have the same tens digit.

For digit position `exp`, we extract the current digit with `(num // exp) % base`. With `base = 10`, the values of `exp` are `1`, `10`, `100`, and so on.

- Time Complexity: O(d(n + b)), where d is the number of digits in the maximum value and b is the base, because each of the d passes processes n values and b counts. With base 10, this simplifies to O(dn).
- Space Complexity: O(n + b), because each pass uses an output list of n values and b counts, and earlier passes do not need to be retained.

Unlike counting sort over whole values, each pass needs only b count entries regardless of how large the values are. Larger values instead increase the number of digit passes. These bounds assume digit extraction and arithmetic take constant time; Python's arbitrarily large integers can add arithmetic cost.

```python
def radix_sort(nums):
    if not nums:
        return []
    if min(nums) < 0:
        raise ValueError("This implementation requires nonnegative integers.")

    base = 10
    max_value = max(nums)
    res = nums.copy()
    exp = 1

    # Stable counting sort on one digit position.
    def sort_by_digit(values, exp):
        counts = [0] * base

        for num in values:
            digit = (num // exp) % base
            counts[digit] += 1

        for i in range(1, base):
            counts[i] += counts[i - 1]

        output = [0] * len(values)
        for num in reversed(values):
            digit = (num // exp) % base
            counts[digit] -= 1
            output[counts[digit]] = num

        return output

    while max_value // exp > 0:
        res = sort_by_digit(res, exp)
        exp *= base

    return res


print(radix_sort([21, 13, 21, 12, 13]))  # [12, 13, 13, 21, 21]
```
