---
title: "Count-Min Sketch: Estimating Event Frequencies in Massive Data Streams"
date: 2025-02-14
draft: false
comments:
  - text: "If the overflow happens on the lower end of the range, it's technically called 'underflow'"
  - text: "In C++ for example, signed integer overflow is undefined behaviour"
---

In many real-world applications, we encounter scenarios where we need to analyze a continuous stream of events. Consider these examples:

*   **Social Media:** Identifying the most trending hashtags at any given moment.
*   **Caching:** Determining which data should be prioritized for caching to improve performance.

Analyzing these scenarios requires us to efficiently count the frequency of events. But what happens when we're dealing with a massive, potentially infinite, stream of data?

### The Challenge of Massive Data

A naive approach might involve using hash tables or binary search trees.  Each time an event occurs, we could increment its count in a hash table or store the keys with frequency values in an ordered manner using a binary search tree.

However, these simple approaches run into problems when dealing with massive datasets:

*   **Space Complexity:** Hash tables and binary search trees both have a linear space complexity, O(n), where n is the number of unique events.  This means the memory required grows linearly with the number of distinct items in the stream.
*   **Limited Space:** What if we have a restricted amount of memory available?  We need a data structure that can handle massive data streams while using a sub-linear amount of space.

### Enter the Count-Min Sketch

The Count-Min Sketch (CMS) is a probabilistic data structure designed to address these challenges. It allows us to:

*   Analyze potentially infinite streams of events.
*   Handle massive amounts of data.
*   Work within the constraints of limited space.
*   Perform fast analysis.

**The key idea is to achieve sub-linear space complexity.**

### How Count-Min Sketch Works

The Count-Min Sketch is a probabilistic algorithm used for estimating values. It's particularly useful for:

*   Calculating the simple frequency of an event.
*   Identifying frequent items in a stream.
*   Computing quantities.

At its core, the CMS uses a matrix of counters and a set of hash functions.

**Structure:**

Imagine a table (matrix) with `d` rows (depth) and `w` columns (width). Each cell in this table is a counter, initialized to zero. We also have `d` independent hash functions, each mapping an item from the data stream to a column in the table.

|         | 0      | 1      | 2      | 3      | 4      | 5      | 6      |
| :------ | :----- | :----- | :----- | :----- | :----- | :----- | :----- |
| **H1**  | 0      | 0      | 0      | 0      | 0      | 0      | 0      |
| **H2**  | 0      | 0      | 0      | 0      | 0      | 0      | 0      |
| **H3**  | 0      | 0      | 0      | 0      | 0      | 0      | 0      |
| **H4**  | 0      | 0      | 0      | 0      | 0      | 0      | 0      |

**Adding an Item:**

When an item arrives in the stream, we hash it using each of the `d` hash functions. Each hash function tells us which column in its corresponding row to update. We then increment the counter in that cell.

**Example:**

Let's say our stream is `[A, B, K, A, A, K, S...]` and we have the following hash values:

*   H1(A) = 1, H2(A) = 6, H3(A) = 3, H4(A) = 1
*   H1(B) = 1, H2(B) = 2, H3(B) = 4, H4(B) = 6
*   H1(K) = 3, H2(K) = 4, H3(K) = 1, H4(K) = 6
*   H1(S) = 6, H2(S) = 2, H3(S) = 4, H4(S) = 1
*   H1(E) = 6, H2(E) = 0, H3(E) = 0, H4(E) = 2

After processing `A, B, K, A, A, K, S, E`, our table might look like this:

|         | 0   | 1   | 2   | 3   | 4   | 5   | 6   |
| :------ | :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| **H1**  | 0   | 4   | 0   | 2   | 0   | 0   | 2   |
| **H2**  | 1   | 0   | 2   | 0   | 2   | 0   | 3   |
| **H3**  | 1   | 2   | 0   | 3   | 2   | 0   | 0   |
| **H4**  | 0   | 4   | 1   | 0   | 0   | 0   | 3   |

**Estimating the Count:**

To estimate the frequency of an item, we hash it using each of the `d` hash functions again.  We then find the *minimum* value among the counters at those locations.  This minimum value is our estimate.

For example, to estimate the count of 'S':

*   H1(S) = 6, H2(S) = 2, H3(S) = 4, H4(S) = 1

So we look up the values in our table:

*   Table = 2
*   Table = 2
*   Table = 2
*   Table = 4

The minimum of `[2, 2, 2, 4]` is 2.  Therefore, our estimate for the number of times 'S' appeared in the stream is 2.

**Why the Minimum?**

The reason we take the *minimum* is that collisions can only *inflate* the counts.  Because multiple items might hash to the same location, the counter value will always be greater than or equal to the true frequency of any individual item that hashes to that location.

### Potential for collisions

Collisions are inevitable, especially with a large stream of data and a limited table size.

**Count-Min Sketch Implementation in Python**

```
import numpy as np
import math

class CountMinSketch:
    def __init__(self, epsilon, delta):
        """
        :param epsilon: Error factor in frequency estimation
        :param delta: Error probability
        """
        self.epsilon = epsilon
        self.delta = delta
        
        # Calculate width and depth based on epsilon and delta
        self.width = math.ceil(math.e / epsilon)
        self.depth = math.ceil(math.log(1 / delta))
        
        self.table = np.zeros((self.depth, self.width), dtype=int)
        self.hash_functions = [
            lambda x, s=seed: hash((s, x)) % self.width for seed in range(self.depth)
        ]

    def add(self, key):
        for i in range(self.depth):
            self.table[i][self.hash_functions[i](key)] += 1

    def estimate(self, key):
        return min(self.table[i][self.hash_functions[i](key)] for i in range(self.depth))

    def get_error_bound(self, elements):
        """
        Returns the theoretical error bound.
        """
        return self.epsilon * elements
```

### Probability and Error

Because the Count-Min Sketch is a probabilistic data structure, there's a chance of error in our frequency estimates. The accuracy of the CMS is controlled by two parameters:

*   **ε (epsilon):**  Represents the error factor in the frequency estimate (the precision of the data structure).
*   **δ (delta):** Represents the probability of error (the chance that the estimate exceeds the error specified by ε).

The dimensions of the matrix (width `w` and depth `d`) are determined by these parameters:

*   width  `w =  ⌈e / ε⌉`
*   depth `d = ⌈ln 1 / δ⌉`

Where `e` is Euler's number (approximately 2.71828).

**Guarantee:**

If we add `m` elements to the CMS, and `â` is the estimated count of an element `x`, and `a` is the actual count of `x`, then:

`Pr(â <= a + ε * m) >= 1 - δ`

In other words, with probability at least `1 - δ`, our estimate `â` will be no more than `ε * m` greater than the true count `a`.

### Conclusion

The Count-Min Sketch provides an efficient way to estimate event frequencies in massive data streams using sub-linear space. While it is a probabilistic data structure and thus introduces some error, the error can be controlled by adjusting the parameters ε and δ to suit the specific application's requirements. This makes it a valuable tool for various applications, including trend analysis, anomaly detection, and data mining.

### References

*   CORMODE, G.; MUTHUKRISHNAN, S. **An improved data stream summary: the count-min sketch and its applications. Journal of Algorithms**, v. 55, n. 1, p. 58-75, abr. 2005.

*   CORMODE, G.; MUTHUKRISHNAN, S. **What's hot and what's not: tracking most frequent items dynamically. ACM Transactions on Database Systems**, v. 30, n. 1, p. 249-278, mar. 2005.

*   SCHWARZ, K. Count-Min Sketches. **[Available here](https://web.stanford.edu/class/archive/cs/cs166/cs166.1206/lectures/10/Slides10.pdf)**. Acessed 20 jan. 2025.
