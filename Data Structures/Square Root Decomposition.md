
**Square Root Decomposition** is one of the foundational techniques for solving **dynamic range query** problems. It's used when we want to compute an operation (e.g. sum, maximum, minimum, GCD, LCM, etc.) over a subarray, even while the array is dynamically changing (i.e., updated between queries).

---

### How It Works

We divide the array into blocks of equal size and precompute the desired operation for each block. Then, to answer a query over a range `[L, R]`, we:

1. **Use precomputed values** from the fully-covered blocks inside the range.
2. **Manually compute** the operation for elements in partially-covered blocks (edges).

This way, we only do heavy computation for at most **two partial blocks**, and the rest are handled in constant time per block.

---

### Complexity Analysis

Let:
- $( x )$ = block size (variable),
- $( n )$ = total number of elements in the array,
- $( \frac{n}{x} )$= number of blocks.

Then the total query time is:

$$
T(x) = x + \frac{n}{x}
$$

To minimize \( T(x) \), we derive:

$$
T'(x) = 1 - \frac{n}{x^2}
$$

Set derivative to zero:

$$
1 - \frac{n}{x^2} = 0 \\
\Rightarrow \frac{n}{x^2} = 1 \\
\Rightarrow x^2 = n \\
\Rightarrow x = \sqrt{n}
$$

Check if this is a minimum using the second derivative:

$$
T''(x) = \frac{2n}{x^3}
$$

Substituting \( x = \sqrt{n} \):

$$
T''(\sqrt{n}) = \frac{2n}{(\sqrt{n})^3} = \frac{2}{\sqrt{n}} > 0
$$

So the function has a **local minimum** at $( x = \sqrt{n} )$.

---

### Summary

- **Optimal block size**: $( \sqrt{n} )$
- **Query complexity**:  
  $$
  T(\sqrt{n}) = \sqrt{n} + \frac{n}{\sqrt{n}} = 2\sqrt{n}
  $$
- **Time complexity**: $O(\sqrt{n})$ per query

This technique efficiently balances block scanning and block skipping, making it a simple yet powerful tool for range queries on dynamic arrays.
