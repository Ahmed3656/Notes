

### **Floyd’s Cycle Detection Algorithm (Tortoise and Hare)**

- Uses two pointers:  
  - **Slow pointer (`slow`)** moves **one step** at a time.  
  - **Fast pointer (`fast`)** moves **two steps** at a time.  
- If a cycle exists, the two pointers will **eventually meet** inside the loop.  
- Time complexity: **O(n)**  
- Space complexity: **O(1)**  

---

## **Proof of Fast Catching Slow in a Cycle**  
1. When `fast` enters the cycle, it is always ahead of `slow`.  
2. Since `fast` moves twice as fast, it **reduces the gap by 1 node per step** inside the cycle.  
3. If the cycle length is `C`, `fast` will meet `slow` in at most `C` steps.  
4. Since `C ≤ n`, the worst-case time complexity is **O(n)**.

---

## **Finding the Start of the Loop**
Once `fast` and `slow` meet inside the loop:  
1. Let `x` be the **number of nodes before the cycle starts**.  
2. Let `C` be the **cycle length**.  
3. When they meet, `slow` has traveled `x + kC` steps, and `fast` has traveled `2(x + kC)` steps.  
4. Since `fast` took exactly **one full cycle more** than `slow`:  

   $$
   2(x + kC) - (x + kC) = C
   $$

   $$
   x + kC = C
   $$

   $$
   x = C - kC
   $$

   - This means the **meeting point is exactly `x` steps away** from the cycle’s start.  

### **Steps to Find the Entry Point**
1. Keep `slow` at the meeting point.  
2. Move a new pointer (`entry`) from `head`.  
3. Move `slow` and `entry` **one step at a time**.  
4. The first node where they meet is the **cycle start node**.  

---

## **Final Algorithm to Detect and Find Cycle Start**
```cpp
ListNode *detectCycle(ListNode *head) {
    ListNode *slow = head, *fast = head;
    
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) { // Cycle detected
            ListNode *entry = head;
            while (entry != slow) {
                entry = entry->next;
                slow = slow->next;
            }
            return entry; // Start of the cycle
        }
    }
    return nullptr; // No cycle
}
