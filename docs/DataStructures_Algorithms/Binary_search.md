# Binary search

## Big O notation and efficiency

Binary search will work on any sorted list of elements. It has a time complexity of O(Log n) (base of 2).
This is because the number of required steps it takes to find the element is proportional to the logarithm of n.

For example, take a list of 64 items. In the worst case scenario, the binary search will have to divide the list 6 times before it reaches its answer.
log64 = 6. So `n` can essentially be revised as the number of items in the list, in order to find the worst case scenario of dividing the algorithm.

`2x2x2x2x2x2 = 64`

---

## Implementation

```python
import math

sortedList = []
i = 0
while i < 50:
    sortedList.append(i * 2)
    i += 1
print(sortedList)

# Implement binary search on sortedList

target = 45

def binarySearch(target, sortedList):
    startIndex = 0
    endIndex = len(sortedList) - 1
    middleIndex = math.ceil((startIndex + endIndex) / 2)

    while startIndex <= endIndex:
        if target == sortedList[middleIndex]:
            return (middleIndex, sortedList[middleIndex])
        elif target > sortedList[middleIndex]:
            startIndex = middleIndex + 1
        elif target < sortedList[middleIndex]:
            endIndex = middleIndex - 1
        middleIndex = math.ceil((startIndex + endIndex) / 2)

    return -1
```
