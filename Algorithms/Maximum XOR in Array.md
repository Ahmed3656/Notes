
### **Concept**
To work with problems that ask for the **maximum XOR obtainable** from subsets or subsequences:
- We build a **linear basis (XOR basis)** from the numbers.
- The basis represents all possible XOR combinations.
- Then we greedily construct the **maximum XOR value** using the basis.

---

### **Steps**

### **1. Building the Basis**
- A basis is an array where each slot corresponds to a **bit position**.
- For each number:
    - Start from the **highest bit down to the lowest**.
    - If that bit is not yet in the basis, store the number there.
    - If it is already in the basis, XOR the number with the stored one and continue.
- This way, every new number either adds a new independent vector or reduces to `0`.

### **2. Constructing Maximum XOR**
- Initialize `res = 0`.
- Again go from **highest bit down to lowest**:
    - Check if `res ^ basis[i]` is larger than `res`.
    - If yes, update `res`.
- This ensures we always try to flip the biggest possible bit first, guaranteeing the maximum.

---

### **Explanation**
1. **Basis as a Vector Space**
    - Each number is like a vector in binary space.
    - The basis keeps the **minimal independent set** of numbers.
    - Any XOR of subsequences can be formed from the basis.
2. **Insertion Process**
    - If the number introduces a new high bit, it expands the space.
    - Otherwise, it collapses down (XOR cancels).
3. **Greedy Maximization**
    - By scanning from high bit to low bit, we always maximize the result at each step.
    - The final `res` is the **largest representable XOR**.

<hr class="hr-light" />

### **Mathematical View (Gaussian Elimination Analogy)**
- Think of each number as a **set of bits**, like a row of switches, where 1 = “on” and 0 = “off”.
- The **highest bit that’s on** in a number is like its **special marker**, we call it a **pivot**.
- When we try to add a number `x` to the basis:
    - If there’s already a number in the basis with the same pivot, we **XOR `x` with that basis number**.
    - This **cancels out that pivot** in `x` and may reveal a **new highest bit**.
    - If `x` becomes `0`, it means everything in `x` is already covered by the basis, so it adds **no new information**.
- Doing this for every number is like **cleaning up duplicates and keeping only the essential parts**.
- By the end, the basis has numbers with **unique highest bits**, and any combination of the original numbers can be made by XORing some subset of the basis.

---

### **Complexity Analysis**
- **Building the basis**: `O(N * B)` where `B` = number of bits (log of max value).
- **Maximization**: `O(B)`.
- **Total**: `O(N * B)`.