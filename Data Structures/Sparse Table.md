#### **Sparse Table**
A data structure designed to answer range queries efficiently on static arrays. It precomputes the answers for all intervals of length $2^k$ (power-of-two lengths), enabling fast query responses by combining these precomputed values.
- **Key Idea:** precompute and store the answers for all contiguous subarrays starting at each index $i$ with length $2^k$. Queries are answered by combining the results of two (or more) overlapping intervals that cover the query range.
- **Properties:**
    - **Static Data:** the underlying array must be static (no updates); if updates are needed, consider a Segment Tree or Fenwick Tree.
    - **Idempotent Functions:** functions like min, max, gcd, and OR are idempotent ($f(x, x) = x$), allowing overlapping intervals without affecting the result. Non-idempotent functions (like sum) require careful handling.
    - **Time Trade-off:** offers $O(1)$ or $O(\log n)$ query time with $O(n \log n)$ precomputation time and space.
- **Use Cases:** range minimum/maximum queries (RMQ), range gcd, range OR, and any range query with an idempotent function.

<hr class="hr-light" />

#### **Important Notes**
- **Idempotence:** functions like min, max, gcd, and OR are idempotent, meaning that applying the function to the same value multiple times doesn't change the result (e.g., $\min(a, a) = a$). This allows overlapping intervals without affecting the query result.
- **Non-idempotent Handling:** for functions like sum, the query method must break the range into non-overlapping intervals to avoid double-counting. This results in $O(\log n)$ query time.
- **Efficient Logarithm Precomputation:** precomputing the $\text{logTable}$ array allows $O(1)$ access to the required power during queries, making the idempotent query truly $O(1)$.
- **Applications Beyond RMQ:** the sparse table can be adapted for any associative function, but idempotence is key for $O(1)$ queries.

---

#### **Implementation**
- **Precomputation (Building the Table):**
    - Let $T[i][k]$ represent the answer for the range starting at index $i$ with length $2^k$.
    - Base case: $T[i][0] = a[i]$ (intervals of length 1).
    - For larger $k$, combine two smaller intervals: $T[i][k] = \text{merge}(T[i][k-1], T[i + 2^{k-1}][k-1])$.
- **Querying:**
    - For idempotent functions (e.g., min, max):
        - Let $k = \lfloor \log_2(r - l + 1) \rfloor$.
        - Answer: $\text{merge}(T[l][k], T[r - 2^k + 1][k])$.
    - For non-idempotent functions (e.g., sum):
        - Decompose the range into disjoint intervals of power-of-two lengths.
        - Iterate over the bits of the range length and combine the results.

```cpp
class SparseTable {
private:
    vector<vector<int>> T; // Sparse table: T[i][k] = answer for [i, i + 2^k - 1]
    int n;                 // Size of the input array
    vector<int> logTable;  // Precomputed logarithms for faster query indexing

    // Merge function: defines the operation (e.g., min, max, gcd, sum)
    int merge(int a, int b) {
        return min(a, b);   // For Range Minimum Query (RMQ)
        // return max(a, b); // For Range Maximum Query
        // return a + b;     // For Range Sum Query (requires non-idempotent handling)
    }

public:
    SparseTable(const vector<int>& arr) {
        n = arr.size();
        int maxK = log2(n) + 1; // Maximum power needed: ceil(log2(n))
        T.assign(n, vector<int>(maxK));

        // Precompute logarithm table for efficient query indexing
        logTable.resize(n + 1);
        logTable[0] = 0; // log2(0) is undefined, but we set to 0 for safety
        logTable[1] = 0;
        for (int i = 2; i <= n; i++)
            logTable[i] = logTable[i / 2] + 1;

        // Build the sparse table
        for (int i = 0; i < n; i++)
            T[i][0] = arr[i]; // Base case: intervals of length 1

        for (int k = 1; k < maxK; k++) {
            for (int i = 0; i + (1 << k) <= n; i++) {
                int nextIndex = i + (1 << (k - 1));
                T[i][k] = merge(T[i][k - 1], T[nextIndex][k - 1]);
            }
        }
    }

    // Query for non-idempotent functions (sum): O(log n)
    int query(int l, int r) {
        int ans = 0; // For sum: start with 0. For min, start with INT_MAX; for max, INT_MIN.
        // int ans = INT_MAX; // For min query
        // int ans = INT_MIN; // For max query
        for (int k = logTable[r - l + 1]; k >= 0; k--) {
            if (l + (1 << k) - 1 <= r) {
                ans = merge(ans, T[l][k]);
                l += (1 << k);
            }
        }
        return ans;
    }

    // Query for idempotent functions (min, max, gcd, OR): O(1)
    int queryIdempotent(int l, int r) {
        // l and r are 0-indexed, inclusive
        int length = r - l + 1;
        int k = logTable[length]; // Largest power of two <= length
        return merge(T[l][k], T[r - (1 << k) + 1][k]);
    }

    // Utility function to print the sparse table (for debugging)
    void printTable() {
        int maxK = T[0].size();
        for (int i = 0; i < n; i++) {
            cout << "i=" << i << ": ";
            for (int k = 0; k < maxK && i + (1 << k) <= n; k++) {
                cout << T[i][k] << " ";
            }
            cout << endl;
        }
    }
};


int main() {
	 vector<int> arr = { 1, 3, 2, 5, 4, 7, 9, 6 };
	 SparseTable st(arr);
	
	 cout << "Sparse Table (for RMQ):" << endl;
	 st.printTable();
	
	 cout << "\nQuery examples (idempotent - min):" << endl;
	 cout << "min in [0, 2]: " << st.queryIdempotent(0, 2) << endl; // min(1,3,2)=1
	 cout << "min in [2, 5]: " << st.queryIdempotent(2, 5) << endl; // min(2,5,4,7)=2
	 cout << "min in [1, 7]: " << st.queryIdempotent(1, 7) << endl; // min(3,2,5,4,7,9,6)=2
	 
	 // Note: for non-idempotent functions, use queryNonIdempotent.
     // Example for sum (uncomment the sum merge function and use):
     // cout << "sum in [0, 2]: " << st.queryNonIdempotent(0, 2) << endl; // 1+3+2=6
	 
	 return 0;
}
```

#### **Complexity Analysis**
- **Precomputation (Building the Table):**
    - **Time:** $O(n \log n)$, as we iterate over $n$ indices and $\log n$ powers.
    - **Space:** $O(n \log n)$, to store the table of size $n \times \lceil \log n \rceil$.
- **Query:**
    - **Idempotent Functions (e.g., min, max, gcd):** $O(1)$, by combining two precomputed intervals.
    - **Non-idempotent Functions (e.g., sum):** $O(\log n)$, by decomposing the range into $\log n$ disjoint intervals.
- **Notes:**
    - The sparse table is **static**; updates require rebuilding the entire table ($O(n \log n)$ time per update).
    - For **range sum queries**, a Fenwick Tree or Segment Tree ($O(\log n)$ per query and update) is often preferred if updates are needed.
    - The logarithm table precomputation accelerates query indexing by providing immediate access to the largest power of two less than or equal to the range length.