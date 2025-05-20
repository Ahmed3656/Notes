
### **Concept**
To efficiently search for a target value in a **rotated sorted array**, we:
- First **identify the pivot point** (smallest element) where the rotation occurred.
- Then **perform binary search** in the correct half.

### **Steps**
### **1. Finding the Pivot (Smallest Element)**
- A rotated sorted array consists of two sorted halves.
- The pivot is the **smallest element**, which also represents the **rotation point**.
- We use **binary search** to find it:
  - If `nums[mid] > nums[right]`, the pivot is in the **right half** → move `left = mid + 1`.
  - Otherwise, the pivot is in the **left half** → move `right = mid`.

#### **2. Determining the Search Range**
- The pivot **divides the array** into two sorted halves.
- If `target` lies in the range `[nums[pivot], nums[right]]`, search the **right half**.
- Otherwise, search the **left half**.

#### **3. Performing Binary Search**
- After selecting the correct half, perform **standard binary search**:
  - If `nums[mid] == target`, return `mid`.
  - If `nums[mid] > target`, search the **left half**.
  - If `nums[mid] < target`, search the **right half**.

### **Explanation**
1. **Pivot Finding**
   - The smallest element is the **turning point**.
   - We reduce the search space using binary search.

2. **Choosing Search Range**
   - The array is split into two sorted parts.
   - We determine in which half the `target` lies.

3. **Binary Search**
   - We apply **regular binary search** on the chosen half.

### **Edge Cases**
- **Single-element array** → The pivot is `nums[0]`.
- **Already sorted array (no rotation)** → The pivot is at index `0`.
- **Target is at pivot** → Directly return pivot index.
- **Target not present** → Return `-1`.

### **Complexity Analysis**
- **Finding pivot**: `O(log N)`
- **Binary search**: `O(log N)`
- **Total complexity**: `O(log N)`