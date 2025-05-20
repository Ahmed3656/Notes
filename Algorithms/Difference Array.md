
### **Concept**
A **Difference Array** allows efficient range updates on an array in constant time per update, followed by a single pass to reconstruct the updated array. It avoids repeated range traversal during updates.

---

### **Steps**

### **1. Initialize the Difference Array**
- Let the original array be `arr` of size `n`.
- Create a `diff` array of size `n + 1`, initialized to 0.
- For converting `arr` to `diff`:
  - `diff[0] = arr[0]`
  - `diff[i] = arr[i] - arr[i - 1]` for `i > 0`

### **2. Applying Range Updates**
- For a query that adds `val` to all elements from index `l` to `r`:
  - Do `diff[l] += val`
  - Do `diff[r + 1] -= val` *(if `r + 1` is within bounds)*

### **3. Reconstructing the Final Array**
- Reconstruct the updated array using prefix sums:
```cpp
arr[0] = diff[0];
for (int i = 1; i < n; i++) {
    arr[i] = arr[i - 1] + diff[i];
}
