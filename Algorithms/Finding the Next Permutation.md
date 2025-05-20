
### **Concept**
The **Next Permutation** algorithm rearranges numbers into the next lexicographically greater permutation. If no greater permutation exists, it rearranges them into the smallest possible order (ascending order).

### **Steps**
### **1. Finding the Breakpoint**
- Traverse the array from right to left.
- Identify the first element `nums[i]` where `nums[i] < nums[i + 1]`. This is the **breakpoint**.
- If no such element exists, the array is in descending order → Reverse it to get the smallest permutation.

### **2. Finding the Smallest Larger Element**
- Traverse the array from right to left.
- Find the smallest element `nums[j]` that is **greater than** `nums[i]`.
- Swap `nums[i]` and `nums[j]`.

### **3. Sorting the Right Side**
- Reverse the portion of the array right of `i` to get the next smallest permutation.

### **Edge Cases**
- **Already the largest permutation** → Reverse the array to get the smallest permutation.
- **Single-element array** → No change needed.

### **Complexity Analysis**
- **Finding the breakpoint**: `O(N)`
- **Finding the smallest larger element**: `O(N)`
- **Reversing the suffix**: `O(N)`
- **Total complexity**: `O(N)`

### **Code Implementation**
```cpp
int l = nums.size() - 1, r = nums.size();
for (; l > 0; l--) {
    if (nums[l] > nums[l - 1]) break;
}
l--;

if (l == -1) {
    sort(nums.begin(), nums.end());
} else {
    while (--r) {
        if (nums[r] > nums[l]) {
            swap(nums[l], nums[r]);
            sort(nums.begin() + l + 1, nums.end());
            break;
        }
    }
}