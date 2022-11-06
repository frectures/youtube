## 23. Merge k Sorted Lists (Hard)

https://leetcode.com/problems/merge-k-sorted-lists/

```
Input: [[1,4,5],[1,3,4],[2,6]]
Output: [1,1,2,3,4,4,5,6]
```

### C#

```csharp
using System;
using System.Collections.Generic;

List<int> mergeSortedLists(IEnumerable<IEnumerable<int>> sortedLists)
{
    var result = new List<int>();

    var queue = new PriorityQueue<IEnumerator<int>, int>();
    foreach (var list in sortedLists)
    {
        var enumerator = list.GetEnumerator();
        if (enumerator.MoveNext())
        {
            queue.Enqueue(enumerator, enumerator.Current);
        }
    }

    while (queue.Count > 0)
    {
        var enumerator = queue.Dequeue();
        result.Add(enumerator.Current);
        if (enumerator.MoveNext())
        {
            queue.Enqueue(enumerator, enumerator.Current);
        }
    }

    return result;
}

foreach (int x in mergeSortedLists(new int[4][]{new int[]{1,4,5},new int[]{1,3,4},new int[]{2,6},new int[]{}}))
{
    Console.WriteLine(x);
}
```

### Java

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.Iterator;
import java.util.List;
import java.util.PriorityQueue;

// remembers current element, inspired by C#
class Enumerator<T>
{
    private final Iterator<T> iterator;
    private T current;

    public Enumerator(Iterable<T> iterable)
    {
        iterator = iterable.iterator();
    }

    public boolean MoveNext()
    {
        if (!iterator.hasNext()) return false;

        current = iterator.next();
        return true;
    }

    public T Current()
    {
        return current;
    }
}

public class Merge
{
    public static List<Integer> mergeSortedLists(Iterable<Iterable<Integer>> sortedLists)
    {
        var result = new ArrayList<Integer>();

        var queue = new PriorityQueue<Enumerator<Integer>>(Comparator.comparing(Enumerator::Current));
        for (var list : sortedLists)
        {
            var enumerator = new Enumerator<>(list);
            if (enumerator.MoveNext())
            {
                queue.offer(enumerator);
            }
        }

        while (!queue.isEmpty())
        {
            var enumerator = queue.poll();
            result.add(enumerator.Current());
            if (enumerator.MoveNext())
            {
                queue.offer(enumerator);
            }
        }

        return result;
    }

    public static void main(String[] args)
    {
        for (int x : mergeSortedLists(List.of(List.of(1,4,5),List.of(1,3,4),List.of(2,6))))
        {
            System.out.println(x);
        }
    }
}
```

### C++

```cpp
#include <iostream>
#include <queue>
#include <vector>

using std::vector;

struct Slice
{
    vector<int>::const_iterator begin, end;
};

bool operator<(const Slice& a, const Slice& b)
{
    return *b.begin < *a.begin; // reverse priorities
}

vector<int> mergeSortedLists(const vector<vector<int>>& sortedLists)
{
    vector<int> result;

    std::priority_queue<Slice> queue;
    for (const auto& list : sortedLists)
    {
        if (!list.empty())
        {
            queue.emplace(list.begin(), list.end()); // --std=c++20
        }
    }

    while (!queue.empty())
    {
        auto slice = queue.top();
        queue.pop();
        result.push_back(*slice.begin);
        if (++slice.begin != slice.end)
        {
            queue.push(slice);
        }
    }

    return result;
}

int main()
{
    for (int x : mergeSortedLists({{1,4,5},{1,3,4},{2,6},{}}))
    {
        std::cout << x << "\n";
    }
}
```

### Clojure

```
#object[java.util.PriorityQueue [[1, 4, 5], [2, 6], [1, 3, 4]]]
#object[java.util.PriorityQueue [[1, 3, 4], [2, 6], [4, 5]]]
#object[java.util.PriorityQueue [[2, 6], [4, 5], [3, 4]]]
#object[java.util.PriorityQueue [[3, 4], [4, 5], [6]]]
#object[java.util.PriorityQueue [[4, 5], [6], [4]]]
#object[java.util.PriorityQueue [[4], [6], [5]]]
#object[java.util.PriorityQueue [[5], [6]]]
#object[java.util.PriorityQueue [[6]]]
#object[java.util.PriorityQueue []]```
```

https://github.com/fredoverflow/clopad/blob/master/samples/defpure.clj

```clj
(defpure merge-sorted-lists-imperative
  {[[2,6],[1,4,5],[1,3,4],[]] [1,1,2,3,4,4,5,6]}
  [& sorted-lists]
  (let [result (new ArrayList)
        queue  (new PriorityQueue (fn [^LinkedList xs, ^LinkedList ys]
                                    (< (. xs peek) (. ys peek))))]
    (. queue addAll (map #(new LinkedList %)
                      (remove empty? sorted-lists)))
    (loop []
      (println queue)
      (if (empty? queue)
        (vec result)
        (let [list (. queue poll)]
          (. result add (. list poll))
          (if-not (. list isEmpty)
            (. queue offer list))
          (recur))))))
```

```xml
<dependency>
    <groupId>org.clojure</groupId>
    <artifactId>data.priority-map</artifactId>
    <version>1.1.0</version>
</dependency>
```

```clj
{0 [1 4 5], 1 [1 3 4], 2 [2 6]}
{1 [1 3 4], 2 [2 6], 0 (4 5)}
{2 [2 6], 1 (3 4), 0 (4 5)}
{1 (3 4), 0 (4 5), 2 (6)}
{0 (4 5), 1 (4), 2 (6)}
{1 (4), 0 (5), 2 (6)}
{0 (5), 2 (6)}
{2 (6)}
{}
```

```clj
(defpure merge-sorted-lists-functional
  {[[2,6],[1,4,5],[1,3,4],[]] [1,1,2,3,4,4,5,6]}
  [& sorted-lists]
  (loop [result (transient [])
         queue  (into (priority-map-keyfn first)
                  (map-indexed vector
                    (remove empty? sorted-lists)))]
    (println queue)
    (if (empty? queue)
      (persistent! result)
      (let [[key [x & xs]] (peek queue)
            queue          (pop  queue)]
        (recur
          (conj! result x)
          (if xs (assoc queue key xs) queue))))))
```

https://clojure.org/reference/transducers

```clj
  (loop [result (transient [])
         queue  (into (priority-map-keyfn first)
                  (comp
                    (remove empty?)
                    (map-indexed vector))
                  sorted-lists)]
```
