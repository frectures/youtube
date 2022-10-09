## Fibonacci

1. Generate *all* Fibonacci numbers (lazy)
2. Populate list with those below 1000 (eager)

```
0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987
```

### Python

```python
from itertools import takewhile

def fibonacci():
    a = 0
    yield a

    b = 1
    yield b

    while True:         # a b c
        c = a + b       # 3 5 8
        yield c

        (a, b) = (b, c) # 5 8
  
first17 = list(takewhile(lambda x: x < 1000, fibonacci()))
```

### JavaScript

```javascript
function* fibonacci() {
    let a = 0n;
    yield a;

    let b = BigInt(1);
    yield b;

    while (true) {       // a b c
        const c = a + b; // 3 5 8
        yield c;

        [a, b] = [b, c]; // 5 8
    }
}

const Generator = function*(){}().constructor;

Generator.prototype.takeWhile = function*(predicate) {
    for (const x of this) {
        if (!predicate(x)) break;
        yield x;
    }
}

const first17 = [...fibonacci().takeWhile(x => x < 1000n)];
```

### C#

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Numerics;

IEnumerable<BigInteger> fibonacci()
{
    BigInteger a = 0;
    yield return a;

    BigInteger b = new BigInteger(1);
    yield return b;

    while (true)
    {                         // a b c
        BigInteger c = a + b; // 3 5 8
        yield return c;

        (a, b) = (b, c);      // 5 8
    }
}

var first17 = fibonacci().TakeWhile(x => x < new BigInteger(1000)).ToList();
```

### Kotlin

```kotlin
import java.math.BigInteger
import java.math.BigInteger.ZERO
import java.math.BigInteger.ONE

fun fibonacci(): Sequence<BigInteger> = sequence {
    var a = ZERO
    yield(a)

    var b = ONE
    yield(b)

    while (true) {    // a b c
        val c = a + b // 3 5 8
        yield(c)

        a = b         // 5
        b = c         //   8
    }
}

val first17 = fibonacci().takeWhile { it < 1000.toBigInteger() }.toList()

// transition to Java
fun fibonacci() = generateSequence(Pair(ZERO, ONE  )) {
                         (a, b) -> Pair(b   , a + b)
                   //    (3, 5) ->     (5   , 8    )
}.map { it.first } //     3             5
```

```
(0, 1)(1, 2)(3, 5)(8, 13)(21, 34)(55, 89)(144, 233)(377, 610)
   (1, 1)(2, 3)(5, 8)(13, 21)(34, 55)(89, 144)(233, 377)(610, 987)
```

### Java

```java
import java.math.BigInteger;
import java.util.List;
import java.util.stream.Stream;

import static java.math.BigInteger.ONE;
import static java.math.BigInteger.ZERO;

class Java {
    record Pair(BigInteger a, BigInteger b) {
    }

    static Stream<BigInteger> fibonacci() {
        return Stream
                .iterate(new Pair(ZERO  , ONE),
                 ab   -> new Pair(ab.b(), ab.a().add(ab.b())))
        //     (3, 5) ->         (   5  , 8  )
                .map(Pair::a);
        //      3                    5
    }

    static List<BigInteger> first17 = fibonacci()
            .takeWhile(fib -> fib.compareTo(BigInteger.valueOf(1000)) < 0)
            .toList();
}
```

### Clojure

```clojure
(defn fibonacci []
  (->>                   [0N 1N    ]
    (iterate (fn [[a b]] [b (+ a b)]))
    ;             [3 5]  [5  8     ]
    (map first))) ;3      5

(def first17
  (into []
    (take-while (fn [x] (< x 1000N)))
    (fibonacci)))

;; transition to Haskell
(def fi-doh-nacci (lazy-cat [0N 1N] (map + fi-doh-nacci (rest fi-doh-nacci))))
```

|                |       |       |     |   |   |    |    |    |    |    |     |     |     |     |     |
| -------------- | ----- | ----- | --- | - | - | -- | -- | -- | -- | -- | --- | --- | --- | --- | --- |
| `fi-doh-nacci` | **0** | **1** | *1* | 2 | 3 |  5 |  8 | 13 | 21 | 34 |  55 |  89 | 144 | 233 | 377 |
| `(rest )`      | **1** |  *1*  |  2  | 3 | 5 |  8 | 13 | 21 | 34 | 55 |  89 | 144 | 233 | 377 | 610 |
| `(map + )`     |  *1*  |   2   |  3  | 5 | 8 | 13 | 21 | 34 | 55 | 89 | 144 | 233 | 377 | 610 | 987 |

Lazy sequences are meant to be *consumed*, not *stored*:

```
user=> (nth (fibonacci)  1000000)
; takes 15 seconds to compute 208988 digits

user=> (nth fi-doh-nacci 1000000)
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
```

### Haskell

```haskell
fi_doh_nacci = 0 : 1 : zipWith (+) fi_doh_nacci (tail fi_doh_nacci)

first17 = takeWhile (< 1000) fi_doh_nacci
```
