## Quadratic complexity

https://leetcode.com/problems/two-sum

> **Follow-up:** Can you come up with an algorithm that is less than O(n²) time complexity?

### TypeScript (2012)

```typescript
function twoSum(nums: number[], target: number): [number, number] | undefined {
    let n = nums.length;

    for (let i = 0; i < n; i++) { //   0   1   2   3    O(n)
        let a = nums[i];          //  15  16  23  42

        for (let j = i + 1; j < n; j++) { //  1   2   3   2   3   3    O(n*(n-1)/2) = O(n²)
            let b = nums[j];              // 16  23  42  23  42  42

            if (a + b === target) return [i, j];
        }
    }
    return undefined;
}

console.log(twoSum([15, 16, 23, 42], 58)); // [1, 3]
```

```
a + b = target
    b = target - a
```

```typescript
function twoSum(nums: number[], target: number): [number, number] | undefined {
    let n = nums.length;

    for (let i = 0; i < n; i++) {
        let a = nums[i];
        let b = target - a;

        // Given b, find j such that b === nums[j]
        for (let j = i + 1; j < n; j++) {
            if (b === nums[j]) {
                return [i, j];
            }
        }
        // Can we accelerate this search? b => j
    }
    return undefined;
}

console.log(twoSum([15, 16, 23, 42], 58)); // [1, 3]
```

## Map/Dictionary optimization

### TypeScript (2012)

```typescript
function twoSum(nums: number[], target: number): [number, number] | undefined {
    let n = nums.length;
    //                       b, j
    let index = new Map<number, number>();

    for (let j = 0; j < n; j++) { //   0   1   2   3    O(n)
        let b = nums[j];          //  15  16  23  42
        index.set(b, j);
    }
    console.log(index);   // Map(4) { 15 => 0, 16 => 1, 23 => 2, 42 => 3 }

    for (let i = 0; i < n; i++) { //   0   1   2   3    O(n)
        let a = nums[i];          //  15  16  23  42
        let b = target - a;       //  43  42  35  16
        let j = index.get(b);     //       3       1
        if (j !== undefined) return [i, j];
    }
    return undefined;
}

console.log(twoSum([15, 16, 23, 42], 58)); // [1, 3]
```

### Python (1991)

```python
def two_sum(nums, target):
    #                           [(0, 15), (1, 16), (2, 23), (3, 42)]
    index = {b: j for (j, b) in enumerate(nums)}
    #       {15: 0, 16: 1, 23: 2, 42: 3}

    for (i, a) in enumerate(nums):
        b = target - a
        j = index.get(b)
        if j != None:
            return (i, j)

print(two_sum([15, 16, 23, 42], 58)) # (1, 3)
```

### C# (2000)

```csharp
using System;
using System.Linq;

(int, int) twoSum(int[] nums, int target) {

    var index = nums.Select((b, j) => (b, j))
                    .ToDictionary(bj => bj.b, bj => bj.j);

    for (int i = 0; i < nums.Length; i++) {
        int a = nums[i];
        int b = target - a;
        if (index.TryGetValue(b, out int j)) return (i, j);
    }
    return (0, 0);
}

Console.WriteLine(twoSum(new int[]{15, 16, 23, 42}, 58)); // (1, 3)
```

### Java (1995)

```java
import java.util.stream.Collectors;
import java.util.stream.IntStream;

class Java {
    record IntPair(int first, int second) {
    }

    static IntPair twoSum(int[] nums, int target) {

        var index = IntStream.range(0, nums.length)
                .boxed()
                .collect(Collectors.toMap(j -> nums[j], j -> j));

        for (int i = 0; i < nums.length; i++) {
            int a = nums[i];
            int b = target - a;
            Integer j = index.get(b);
            if (j != null) return new IntPair(i, j);
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

    val index = nums.mapIndexed { j, b -> b to j }.toMap()

    nums.forEachIndexed { i, a ->
        val b = target - a
        index[b]?.let { j ->
            return Pair(i, j)
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

    index := make(map[int]int)

    for j, b := range nums {
        index[b] = j
    }

    for i, a := range nums {
        b := target - a
        if j, ok := index[b]; ok {
            return i, j
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
use std::collections::HashMap;

fn two_sum(nums: &[i32], target: i32) -> (usize, usize) {

    let mut index = HashMap::new();

    for (j, b) in nums.iter().enumerate() {
        index.insert(b, j);
    }

    for (i, a) in nums.iter().enumerate() {
        let b = target - a;
        if let Some(rj) = index.get(&b) {
            return (i, *rj);
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
#include <unordered_map>
#include <vector>

auto twoSum(const std::vector<int>& nums, int target) -> std::pair<size_t, size_t> {
    size_t n = nums.size();
    std::unordered_map<int, size_t> index;

    for (size_t j = 0; j < n; j++) {
        int b = nums[j];
        index.emplace(b, j);
    }
    for (size_t i = 0; i < n; i++) {
        int a = nums[i];
        int b = target - a;
        if (auto it = index.find(b); it != index.end()) {
            auto [b, j] = *it;
            return {i, j};
        }
    }
    // undefined behavior
}

int main() {
    auto [i, j] = twoSum({15, 16, 23, 42}, 58);
    std::cout << i << ", " << j << "\n"; // 1, 3
}
```

### Clojure (2007), dialect of LISP (1958)

```clojure
(defn two-sum [nums target]
  (let [n     (count nums)
        index (zipmap nums (range n))]
    (first
      (for [i (range n)
            :let [a (nums i)
                  b (- target a)
                  j (index b)]
            :when j]
        [i j]))))
; nil

(two-sum [15 16 23 42] 58) ; [1 3]
```
