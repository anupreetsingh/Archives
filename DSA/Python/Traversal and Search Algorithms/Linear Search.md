# Linear Search

Used in case of Unordered List. Checks each element one-by-one until you find the target or reach the end of the array.

```python
def linear_search(arr, target):
    for i in range(len(arr)):
        if arr[i] == target:
            return i
    return -1
```
