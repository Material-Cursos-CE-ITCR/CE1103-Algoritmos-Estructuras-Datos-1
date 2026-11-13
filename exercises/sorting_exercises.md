# LeetCode Solutions (C#)

All solutions are in C#, written with simple, explicit steps.
Each problem lists its **input**, **output**, **constraints** and **examples** before the solution.

All problems should be solved accordingly using ideas developed in class for sorting and searching. 


---

## 1. Sort Colors

**Link:** https://leetcode.com/problems/sort-colors/

### Problem

You are given an array of objects colored red, white or blue. They are represented by the integers `0` (red), `1` (white) and `2` (blue). Sort the array **in place** so that objects of the same color are adjacent, in the order red, white, blue (that is, `0`s first, then `1`s, then `2`s). You must solve it **without using the library sort function**.

### Input

- `int[] nums`: the array to sort. It only contains the values `0`, `1` and `2`, in any order and with any number of repetitions.

### Output

- Nothing is returned (`void`). You must modify `nums` directly. LeetCode checks the contents of `nums` after your method finishes.

### Constraints

- $1 \le n \le 300$, where $n$ is `nums.Length`
- `nums[i]` is `0`, `1` or `2`

### Examples

| Input | Result in `nums` |
|---|---|
| `[2,0,2,1,1,0]` | `[0,0,1,1,2,2]` |
| `[2,0,1]` | `[0,1,2]` |

### Approach A: Counting (two passes)

Count how many 0s, 1s and 2s there are, then overwrite the array.

```csharp
public class Solution {
    public void SortColors(int[] nums) {
        int count0 = 0, count1 = 0, count2 = 0;

        // First pass: count each value
        foreach (int x in nums) {
            if (x == 0) count0++;
            else if (x == 1) count1++;
            else count2++;
        }

        // Second pass: overwrite the array
        int i = 0;
        for (int k = 0; k < count0; k++) {
            nums[i] = 0;
            i++;
        }
        for (int k = 0; k < count1; k++) {
            nums[i] = 1;
            i++;
        }
        for (int k = 0; k < count2; k++) {
            nums[i] = 2;
            i++;
        }
    }
}
```

**Complexity:** time $O(n)$, space $O(1)$.

### Approach B: Dutch national flag (one pass)

Three pointers split the array into zones:

- `[0, low)` holds 0s
- `[low, mid)` holds 1s
- `(high, n-1]` holds 2s
- `[mid, high]` is the unprocessed zone

```csharp
public class Solution {
    public void SortColors(int[] nums) {
        int low = 0;
        int mid = 0;
        int high = nums.Length - 1;

        while (mid <= high) {
            if (nums[mid] == 0) {
                // Send the 0 to the front zone
                Swap(nums, low, mid);
                low++;
                mid++;
            } else if (nums[mid] == 1) {
                // Already in the right zone
                mid++;
            } else {
                // Send the 2 to the back zone.
                // Do not advance mid: the swapped-in value is unchecked.
                Swap(nums, mid, high);
                high--;
            }
        }
    }

    private void Swap(int[] nums, int a, int b) {
        int temp = nums[a];
        nums[a] = nums[b];
        nums[b] = temp;
    }
}
```

**Complexity:** time $O(n)$, space $O(1)$.

---

## 2. Merge Sorted Array

**Link:** https://leetcode.com/problems/merge-sorted-array/

### Problem

You are given two integer arrays `nums1` and `nums2`, both sorted in non-decreasing order. Merge `nums2` into `nums1` so that `nums1` ends up as a single array sorted in non-decreasing order. The result is stored **inside `nums1`**, not returned.

To make this possible, `nums1` has extra room at the end: its length is $m + n$, where the first $m$ elements are the real values and the last $n$ elements are `0`s used only as empty placeholders. Those placeholder zeros are not real data and must be overwritten.

### Input

- `int[] nums1`: length $m + n$. The first $m$ elements are sorted real values; the last $n$ are `0` placeholders.
- `int m`: how many real elements `nums1` holds (the first $m$ positions).
- `int[] nums2`: length $n$, sorted.
- `int n`: how many elements `nums2` holds.

### Output

- Nothing is returned (`void`). `nums1` must contain all $m + n$ values in sorted order after the method finishes.

### Constraints

- `nums1.Length == m + n` and `nums2.Length == n`
- $0 \le m, n \le 200$ and $1 \le m + n \le 200$
- $-10^9 \le$ `nums1[i]`, `nums2[j]` $\le 10^9$

### Examples

| `nums1` | `m` | `nums2` | `n` | Result in `nums1` |
|---|---|---|---|---|
| `[1,2,3,0,0,0]` | 3 | `[2,5,6]` | 3 | `[1,2,2,3,5,6]` |
| `[1]` | 1 | `[]` | 0 | `[1]` |
| `[0]` | 0 | `[1]` | 1 | `[1]` |

In the first example, only `[1,2,3]` is real data in `nums1`; the three trailing `0`s are free space.

### Solution

**Idea:** Fill `nums1` from the back. The largest remaining element always goes to the last free position, so nothing useful gets overwritten.

```csharp
public class Solution {
    public void Merge(int[] nums1, int m, int[] nums2, int n) {
        int i = m - 1;        // last valid element of nums1
        int j = n - 1;        // last element of nums2
        int k = m + n - 1;    // last position of nums1 (where we write)

        while (j >= 0) {
            if (i >= 0 && nums1[i] > nums2[j]) {
                nums1[k] = nums1[i];
                i--;
            } else {
                nums1[k] = nums2[j];
                j--;
            }
            k--;
        }
        // If j < 0, the rest of nums1 is already in place.
    }
}
```

**Complexity:** time $O(m + n)$, space $O(1)$.

---

## 3. Height Checker

**Link:** https://leetcode.com/problems/height-checker/

### Problem

A school wants all students to stand in a line in non-decreasing order of height. The array `expected` is the heights sorted in non-decreasing order (the order the students should be in). The array `heights` is the order the students are actually standing in. Return the number of indices `i` where `heights[i] != expected[i]`.

Note that `expected` is not given to you: you have to build it yourself by sorting a copy of `heights`.

### Input

- `int[] heights`: `heights[i]` is the height of the $i$-th student in the current line.

### Output

- An `int`: the number of positions where the current line differs from the sorted line.

### Constraints

- $1 \le$ `heights.Length` $\le 100$
- $1 \le$ `heights[i]` $\le 100$

### Examples

| `heights` | `expected` (sorted copy) | Output | Why |
|---|---|---|---|
| `[1,1,4,2,1,3]` | `[1,1,1,2,3,4]` | 3 | Indices 2, 4 and 5 differ (4 vs 1, 1 vs 3, 3 vs 4) |
| `[5,1,2,3,4]` | `[1,2,3,4,5]` | 5 | Every index differs |
| `[1,2,3,4,5]` | `[1,2,3,4,5]` | 0 | Already sorted |

### Solution

**Idea:** Make a sorted copy and count the positions where the two arrays differ.

```csharp
public class Solution {
    public int HeightChecker(int[] heights) {
        int[] expected = (int[])heights.Clone();
        Array.Sort(expected);

        int count = 0;
        for (int i = 0; i < heights.Length; i++) {
            if (heights[i] != expected[i]) {
                count++;
            }
        }
        return count;
    }
}
```

**Complexity:** time $O(n \log n)$, space $O(n)$.

### Variant: counting sort

Heights are between 1 and 100, so we can build the expected order with counting sort in $O(n)$ time:

```csharp
public class Solution {
    public int HeightChecker(int[] heights) {
        int[] freq = new int[101];

        // Count how many times each height appears
        foreach (int h in heights) {
            freq[h]++;
        }

        int count = 0;
        int index = 0;  // position in the original array

        // Walk through heights in increasing order
        for (int h = 1; h <= 100; h++) {
            for (int k = 0; k < freq[h]; k++) {
                // Position "index" should hold height h
                if (heights[index] != h) {
                    count++;
                }
                index++;
            }
        }
        return count;
    }
}
```

**Complexity:** time $O(n + R)$ with $R = 100$, space $O(R)$.

---

## 4. Richest Customer Wealth

**Link:** https://leetcode.com/problems/richest-customer-wealth/

### Problem

You are given a grid where each row is a customer and each column is a bank. The value in each cell is how much money that customer has in that bank. A customer's **wealth** is the sum of the money they have across all their banks (the sum of their row). Return the wealth of the richest customer.

### Input

- `int[][] accounts`: an $m \times n$ grid. `accounts[i][j]` is the amount of money the $i$-th customer has in the $j$-th bank.
  - $m$ = number of customers (rows) = `accounts.Length`
  - $n$ = number of banks (columns) = `accounts[i].Length`

### Output

- An `int`: the maximum row sum.

### Constraints

- $1 \le m, n \le 50$
- $1 \le$ `accounts[i][j]` $\le 100$

### Examples

| `accounts` | Output | Why |
|---|---|---|
| `[[1,2,3],[3,2,1]]` | 6 | Both customers have $1+2+3 = 6$ |
| `[[1,5],[7,3],[3,5]]` | 10 | Row sums are 6, 10, 8; the max is 10 |
| `[[2,8,7],[7,1,3],[1,9,5]]` | 17 | Row sums are 17, 11, 15; the max is 17 |

### Solution

**Idea:** Sum each row and keep the maximum.

```csharp
public class Solution {
    public int MaximumWealth(int[][] accounts) {
        int best = 0;

        for (int i = 0; i < accounts.Length; i++) {
            int wealth = 0;

            // Sum all the banks of customer i
            for (int j = 0; j < accounts[i].Length; j++) {
                wealth += accounts[i][j];
            }

            if (wealth > best) {
                best = wealth;
            }
        }
        return best;
    }
}
```

**Complexity:** time $O(m \cdot n)$, space $O(1)$.

---

## 5. Sqrt(x)

**Link:** https://leetcode.com/problems/sqrtx/

### Problem

Given a non-negative integer `x`, return the square root of `x` **rounded down** to the nearest integer, that is, $\lfloor \sqrt{x} \rfloor$. The result must be a non-negative integer. You are **not allowed to use any built-in exponent function or operator** (for example `Math.Sqrt` or `Math.Pow`).

### Input

- `int x`: a non-negative integer.

### Output

- An `int`: the largest integer $r$ such that $r^2 \le x$. The decimal part is discarded, not rounded.

### Constraints

- $0 \le x \le 2^{31} - 1$

### Examples

| `x` | Output | Why |
|---|---|---|
| 4 | 2 | $\sqrt{4} = 2$ exactly |
| 8 | 2 | $\sqrt{8} = 2.82842\ldots$, and the decimal part is dropped |

### Solution

**Idea:** Binary search on the answer. We look for the largest $r$ such that $r^2 \le x$.

```csharp
public class Solution {
    public int MySqrt(int x) {
        long low = 0;
        long high = x;
        long answer = 0;

        while (low <= high) {
            long mid = low + (high - low) / 2;

            if (mid * mid <= x) {
                // mid is a valid candidate, try to find a bigger one
                answer = mid;
                low = mid + 1;
            } else {
                // mid is too big
                high = mid - 1;
            }
        }
        return (int)answer;
    }
}
```

**Note:** `mid * mid` can overflow a 32-bit `int` (for example $46341^2 > 2^{31}-1$), so we use `long`.

**Complexity:** time $O(\log x)$, space $O(1)$.

---

## 6. Two Sum

**Link:** https://leetcode.com/problems/two-sum/

### Problem

Given an array of integers `nums` and an integer `target`, return the **indices** (positions, not the values) of the two numbers that add up to `target`. You may assume there is exactly one valid answer, and you may not use the same element twice. The answer can be returned in any order.

### Input

- `int[] nums`: the array of integers. It can contain negative numbers and repeated values.
- `int target`: the sum you are looking for.

### Output

- An `int[]` with two elements: the indices `[i, j]` such that `nums[i] + nums[j] == target` and $i \ne j$.

### Constraints

- $2 \le$ `nums.Length` $\le 10^4$
- $-10^9 \le$ `nums[i]` $\le 10^9$
- $-10^9 \le$ `target` $\le 10^9$
- Exactly one valid answer exists.

### Examples

| `nums` | `target` | Output | Why |
|---|---|---|---|
| `[2,7,11,15]` | 9 | `[0,1]` | `nums[0] + nums[1] = 2 + 7 = 9` |
| `[3,2,4]` | 6 | `[1,2]` | `2 + 4 = 6` (not `[0,0]`, the same element cannot be used twice) |
| `[3,3]` | 6 | `[0,1]` | Two different elements with the same value |

### Solution

**Idea:** For each number $x$, we need $target - x$. Store the numbers already seen in a hash map (`value -> index`) and check if the complement is there.

```csharp
public class Solution {
    public int[] TwoSum(int[] nums, int target) {
        Dictionary<int, int> seen = new Dictionary<int, int>();  // value -> index

        for (int i = 0; i < nums.Length; i++) {
            int complement = target - nums[i];

            // Did we already see the number we need?
            if (seen.ContainsKey(complement)) {
                return new int[] { seen[complement], i };
            }

            // Save the current number for later elements
            seen[nums[i]] = i;
        }
        return new int[0];  // never reached, a solution always exists
    }
}
```

**Complexity:** time $O(n)$ on average, space $O(n)$.