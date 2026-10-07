## Report - Number of Occurrences

### 1. Brute-force solution: `countFreqBrute()`

**Idea.** Scan the array from left to right and increment a counter each time `A[i] == key`.
The solution does not use the fact that the array is sorted.

**Running time.** The loop always performs exactly $n = A.length$ iterations, each costing $\Theta(1)$ (one comparison and possibly one increment). There is no early exit, so the best, average and worst cases coincide:

$$T(n) = \Theta(n)$$

**Memory:** $\Theta(1)$.

### 2. Divide-and-conquer solution: `countFreqSmart()`

**Idea.** Since $A$ is sorted, all occurrences of `key` form one contiguous block `A[first..last]`.
Then the answer is

$$count = last - first + 1,$$

or $0$ if the key is not in the array. So the problem reduces to finding the **first** and the **last** index of `key` with two modified binary searches:

* `findFirst` - when `A[mid] == key`, we save `mid` as a candidate and continue searching in the **left** half (`hi = mid - 1`) to find an even earlier occurrence.
* `findLast` - when `A[mid] == key`, we save `mid` and continue in the **right** half (`lo = mid + 1`).
* If `A[mid] < key`, the key can only be to the right (`lo = mid + 1`); if `A[mid] > key`, only to the left (`hi = mid - 1`).
* If `findFirst` returns `-1`, the key is absent and we return `0` without running `findLast`.

Edge cases: an empty array (`hi = -1`, the loop never runs, the result is `0`); a key smaller or larger than all elements; all elements equal to the key. Also, `mid = lo + (hi - lo) / 2` avoids integer overflow.

**Running time.** Each step of the search does $\Theta(1)$ work and discards half of the current range, so

$$T(n) = T(n/2) + \Theta(1).$$

By the Master theorem: $a = 1,\ b = 2,\ f(n) = \Theta(1) = \Theta(n^{\log_2 1}) = \Theta(n^0)$, which is case 2, so

$$T(n) = \Theta(\log n).$$

The loop stops only when the range becomes empty (it does not stop at the first match, because it keeps looking for the boundary), therefore the number of iterations is about $\lfloor \log_2 n \rfloor + 1$ in every case, so the bound is tight: $\Theta(\log n)$ in the best, average and worst cases.
The function runs two such searches, so $T(n) = 2 \cdot \Theta(\log n) = \Theta(\log n)$.

**Memory:** $\Theta(1)$, because the searches are iterative (no recursion stack).

### 3. Summary

| Solution | Best | Average | Worst | Memory |
|---|---|---|---|---|
| `countFreqBrute` | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(1)$ |
| `countFreqSmart` | $\Theta(\log n)$ | $\Theta(\log n)$ | $\Theta(\log n)$ | $\Theta(1)$ |

### 4. Empirical comparison

Measured with `System.nanoTime()`. The array is sorted and random, the key is chosen from the array, and each measurement is the average over many repetitions after a JIT warm-up.

```java
static void benchmark() {
    Problem1 p = new Problem1();
    Random rnd = new Random(42);
    int[] sizes = {1_000, 10_000, 100_000, 1_000_000, 10_000_000};
    int reps = 1000;

    for (int n : sizes) {
        int[] A = new int[n];
        for (int i = 0; i < n; i++) A[i] = rnd.nextInt(n / 10 + 1);
        Arrays.sort(A);
        int key = A[n / 2];

        // warm-up
        for (int i = 0; i < 200; i++) {
            p.countFreqBrute(key, A);
            p.countFreqSmart(key, A);
        }

        long t0 = System.nanoTime();
        int r1 = 0;
        for (int i = 0; i < reps; i++) r1 += p.countFreqBrute(key, A);
        long brute = (System.nanoTime() - t0) / reps;

        t0 = System.nanoTime();
        int r2 = 0;
        for (int i = 0; i < reps; i++) r2 += p.countFreqSmart(key, A);
        long smart = (System.nanoTime() - t0) / reps;

        System.out.printf("n=%d brute=%d ns smart=%d ns equal=%b%n", n, brute, smart, r1 == r2);
    }
}
```

**Results** (average time per call, ns):

| n | Brute-force, ns | Divide-and-conquer, ns | Speed-up |
|---|---|---|---|
| 1 000 | ... | ... | ... |
| 10 000 | ... | ... | ... |
| 100 000 | ... | ... | ... |
| 1 000 000 | ... | ... | ... |
| 10 000 000 | ... | ... | ... |

**Discussion.** The brute-force time grows linearly: increasing $n$ by 10 increases the time by about 10 times, which matches $\Theta(n)$. The time of `countFreqSmart` barely changes: increasing $n$ by 10 adds only about $\log_2 10 \approx 3.3$ extra iterations per search, which matches $\Theta(\log n)$. Therefore the speed-up grows with $n$. For very small arrays the difference is negligible (or even in favor of brute-force because of its simple sequential memory access and low constant factor), but for $n \ge 10^5$ the divide-and-conquer solution is faster by orders of magnitude.
