## Beginner Language

https://www.google.com/search?q=beginner+language+site:reddit.com

- What's the best starter language for a beginner programmer?
- What programming language should I start with first?
- What is the best/first coding language that a person should learn?
- ...

Picking the first language matters less than you may think:

1. **Fundamental concepts** transfer between languages:
   - Functions
   - Variables
   - Collections
   - Loops
   - Conditionals
   - ...
2. **Problem solving** is:
   - initially (mostly) language-independent
   - much harder

## LeetCode Problem 1. "Two Sum"

https://leetcode.com/problems/two-sum

- Python, JavaScript, TypeScript
- C#, Java, Kotlin
- Go, Rust, C++, C
- Clojure, Haskell

> Given an array of integers `nums` and an integer `target`,<br>
> return indices of the two numbers such that they add up to `target`.

```
          0   1   2   3
  nums:  15  16  23  42
target:  58
```

> You may assume that each input would have exactly one solution,<br>
> and you may not use the same element twice.<br>
> You can return the answer in any order.

**Idea**: Check all `a + b` two sums, where `a` left of `b`

```
          0   1   2   3
          i   j
          i       j
          i           j
              i   j
              i       j
                  i   j
                      i
              *       *
  nums:  15  16  23  42
          a   b
          a       b
          a           b
              a   b
              a       b
                  a   b
                      a
```

### Python (1991)

```python
def two_sum(nums, target):
    n = len(nums)

    for i in range(0, n):
        a = nums[i]

        for j in range(i + 1, n):
            b = nums[j]

            if a + b == target:
                return (i, j)
    # return None

print(two_sum([15, 16, 23, 42], 58)) # (1, 3)
```

### JavaScript (1995)

```javascript
function twoSum(nums, target) {
    let n = nums.length;

    for (let i = 0; i < n; i++) {
        let a = nums[i];

        for (let j = i + 1; j < n; j++) {
            let b = nums[j];

            if (a + b === target) return [i, j];
        }
    }
    // return undefined;
}

console.log(twoSum([15, 16, 23, 42], 58)); // [1, 3]
```

### TypeScript (2012)

```typescript
function twoSum(nums: number[], target: number): [number, number] | undefined {
    let n = nums.length;

    for (let i = 0; i < n; i++) {
        let a = nums[i];

        for (let j = i + 1; j < n; j++) {
            let b = nums[j];

            if (a + b === target) return [i, j];
        }
    }
    return undefined;
}

console.log(twoSum([15, 16, 23, 42], 58)); // [1, 3]
```

### C# (2000)

```csharp
using System;

(int, int) twoSum(int[] nums, int target) {
    int n = nums.Length;

    for (int i = 0; i < n; i++) {
        int a = nums[i];

        for (int j = i + 1; j < n; j++) {
            int b = nums[j];

            if (a + b == target) return (i, j);
        }
    }
    return (0, 0);
}

Console.WriteLine(twoSum(new int[]{15, 16, 23, 42}, 58)); // (1, 3)
```

### Java (1995)

```java
class Java {
    record IntPair(int first, int second) {
    }

    static IntPair twoSum(int[] nums, int target) {
        int n = nums.length;

        for (int i = 0; i < n; i++) {
            int a = nums[i];

            for (int j = i + 1; j < n; j++) {
                int b = nums[j];

                if (a + b == target) return new IntPair(i, j);
            }
        }
        return null;
    }

    public static void main(String[] args) {
        System.out.println(twoSum(new int[]{15, 16, 23, 42}, 58)); // IntPair[first=1, second=3]
    }
}
```

### Kotlin (2011)

```kotlin
fun twoSum(nums: IntArray, target: Int): Pair<Int, Int>? {
    val n = nums.size

    for (i in 0 until n) {
        val a = nums[i]

        for (j in i+1 until n) {
            val b = nums[j]

            if (a + b == target) return Pair(i, j)
        }
    }
    return null
}

fun main() {
    println(twoSum(intArrayOf(15, 16, 23, 42), 58)) // (1, 3)
}
```

### Go (2009)

```go
package main

import "fmt"

func twoSum(nums []int, target int) (int, int) {
    n := len(nums)

    for i := 0; i < n; i++ {
        a := nums[i]

        for j := i + 1; j < n; j++ {
            b := nums[j]

            if a + b == target {
                return i, j
            }
        }
    }
    return 0, 0
}

func main() {
    fmt.Println(twoSum([]int{15, 16, 23, 42}, 58)) // 1 3
}
```

### Rust (2010)

```rust
fn two_sum(nums: &[i32], target: i32) -> (usize, usize) {
    let n = nums.len();

    for i in 0..n {
        let a = nums[i];

        for j in i+1..n {
            let b = nums[j];

            if a + b == target {
                return (i, j);
            }
        }
    }
    return (0, 0);
}

fn main() {
    println!("{:?}", two_sum(&[15, 16, 23, 42], 58)); // (1, 3)
}
```

### C++ (1983)

```c++
#include <iostream>
#include <utility>
#include <vector>

auto twoSum(const std::vector<int>& nums, int target) -> std::pair<size_t, size_t> {
    size_t n = nums.size();

    for (size_t i = 0; i < n; i++) {
        int a = nums[i];

        for (size_t j = i + 1; j < n; j++) {
            int b = nums[j];

            if (a + b == target) return {i, j};
        }
    }
    // undefined behavior
}

int main() {
    auto [i, j] = twoSum({15, 16, 23, 42}, 58);
    std::cout << i << ", " << j << "\n"; // 1, 3
}
```

### C (1972)

```c
#include <stdio.h>

typedef struct { size_t first, second; } SizePair;

SizePair twoSum(int nums[], size_t n, int target) {

    for (size_t i = 0; i < n; i++) {
        int a = nums[i];

        for (size_t j = i + 1; j < n; j++) {
            int b = nums[j];

            if (a + b == target) {
                SizePair result = {i, j};
                return result;
            }
        }
    }
    // undefined behavior
}

int main() {
    int lost[] = {15, 16, 23, 42};
    SizePair ij = twoSum(lost, 4, 58);
    printf("%d, %d\n", ij.first, ij.second); // 1, 3
}
```

### Clojure (2007), dialect of LISP (1958)

```clojure
(defn two-sum [nums target]
  (let [n (count nums)]
    (first
      (for [i (range      0  n)  :let [x (nums i)]
            j (range (inc i) n)  :let [y (nums j)]
            :when (= target (+ x y))]
        [i j]))))
; nil

(two-sum [15 16 23 42] 58) ; [1 3]
```

### Haskell (1990)

```haskell
import Data.List (tails)

pairs :: [t] -> [(t, t)]
pairs    list = [(a, b) | (a:bs) <- tails list,
                           b     <- bs]

twoSum :: [Int] -> Int -> (Int, Int)
twoSum    nums  target = head $
    let indexedNums = zip [0..length nums - 1] nums
    in  [(i, j) | ((i, a), (j, b)) <- pairs indexedNums,
                  a + b == target]
-- Prelude.head: empty list

main :: IO ()
main = print $ twoSum [15, 16, 23, 42] 58 -- (1, 3)
```
