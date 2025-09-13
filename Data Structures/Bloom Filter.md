A space-efficient probabilistic data structure used to test whether an element is a member of a set. It provides definitive "no" answers but may return false positives, making it ideal for pre-filtering expensive operations like database or disk lookups.

- **Core Problem Solved:** avoids expensive lookups for non-existent items by providing a memory-efficient way to check if an element **definitely is not** in a set.
- **Basic Structure:** consists of a fixed-size **bit array** (typically ranging from kilobytes to megabytes) and **multiple independent hash functions**.
- **Operations:**
    - **Insertion:** hash the element with each hash function, compute positions modulo array size, and set those bits to 1.
    - **Query:** hash the element and check all corresponding bits. If any bit is 0, the element is **definitely not** in the set. If all are 1, the element **may be** in the set (with a chance of false positives).
- **Key Properties:**
    - **No False Negatives:** if an element was added, the filter will always report it as present.
    - **False Positives Possible:** different elements may hash to the same bits, so a "maybe" result requires a definitive secondary check.
    - **Extremely Efficient:** O(1) time complexity for inserts and checks, with very low memory overhead.
```text
Bloom Filter with m=16 bits, k=3 hash functions

Insert "apple":
   h1("apple") % 16 = 3
   h2("apple") % 16 = 10
   h3("apple") % 16 = 14
   Set bits [3, 10, 14] to 1

Insert "banana":
   h1("banana") % 16 = 10
   h2("banana") % 16 = 12
   h3("banana") % 16 = 15
   Set bits [10, 12, 15] to 1

Query "orange":
   h1("orange") % 16 = 3
   h2("orange") % 16 = 12
   h3("orange") % 16 = 15
   Check bits [3, 12, 15] → All are 1 → "Maybe present" (False Positive!)

Query "grape":
   h1("grape") % 16 = 2 → Bit 2 is 0 → "Definitely not present"
```

<hr class="hr-light" />

#### **Mathematical Foundation & Parameter Tuning**

The behavior and efficiency of a Bloom filter are governed by three parameters, with mathematical relationships that allow precise tuning for specific use cases.

- **Parameters:**
    - `n`: number of elements expected to be inserted
    - `m`: size of the bit array (in bits)
    - `k`: number of different hash functions used
- **False Positive Rate Approximation:**  
    The probability of a false positive is approximately:
	$$\text{FPR} \approx \left(1 - e^{- \frac{k \cdot n}{m}}\right)^k$$
- **Optimal Hash Function Count:**  
	For given `m` and `n`, the optimal number of hash functions that minimizes the false positive rate is:
	$$k_{\text{optimal}} = \frac{m}{n} \ln (2)$$
- **Minimal False Positive Rate:**  
	Using the optimal `k`, the minimal achievable false positive rate becomes:
	$$\text{FPR}_{\text{min}} \approx (0.6185)^{\frac{m}{n}}$$
<hr class="hr-light" />

#### **Practical Sizing: The 1-2% Rule**
In practice, you start by choosing an acceptable false positive rate, then calculate the required filter size.

- **Size Calculation Formula:**  
    For a desired false positive rate `FPR`, the required bits per element is:
    $$m = - \frac{n \cdot \ln(\text{FPR})}{(\ln (2))^2}$$
- **Practical Examples:**
	- **For 1% FPR (FPR = 0.01):**
	$$\frac{m}{n} = - \frac{\ln 0.01}{(\ln 2)^2} \approx 9.6 \text{ bits per element}$$
	- _For 1 million elements: ~10 million bits ≈ 1.25 MB_
	- **For 0.1% FPR (FPR = 0.001):**
	$$\frac{m}{n} = - \frac{\ln 0.001}{(\ln 2)^2} \approx 14.4 \text{ bits per element}$$
	_For 1 million elements: ~14.4 million bits ≈ 1.8 MB_
- **Industry Standard:** Database systems typically use false positive rates between 0.1% and 1%.

<hr class="hr-light" />

#### **Advanced Variants**

- **Counting Bloom Filter:**
    - **Purpose:** extends the standard Bloom filter to support **deletion** by replacing each bit with a small counter (typically 4 bits).
    - **Mechanism:** insertion increments counters, deletion decrements them. An element is considered present if all its counters are > 0.
    - **Trade-off:** supports deletion at the cost of **significantly increased memory overhead** (typically 4× the storage) and a small risk of counter overflow.
- **Scalable Bloom Filters:**
    - **Purpose:** automatically grows to accommodate more elements while maintaining a consistent false positive rate.
    - **Mechanism:** uses multiple Bloom filters with increasing sizes and stricter false positive rates for each new layer.

<hr class="hr-light" />

#### **Implementation Considerations**

- **Hash Function Selection:**
    - Goal is **independent, uniformly distributed** outputs.
    - **Practical Implementation:** often simulated using a **single high-quality hash function** (e.g., MurmurHash, xxHash) with `k` different seeds:
    ```cpp
    hash_i(value) = murmurhash(value, seed_i) % m
    ```
    
    - The filter relies on requiring collisions across **all** `k` functions for a false positive.
- **Memory vs Accuracy Trade-off:**
    - The false positive rate can be reduced by using a larger bit array or more hash functions, at the cost of increased memory and computation.
    - **Saturation Risk:** as more items are added, the false positive rate increases. Once all bits are set, every check returns "maybe," rendering the filter useless.

<hr class="hr-light" />

#### **Use Cases & Applications**

- **Database Systems:**
    - Avoiding unnecessary disk I/O for non-existent keys in SSTables (LevelDB, RocksDB).
    - Reducing database queries for non-existent records.
- **Network Systems:**
    - Network routers for packet filtering.
    - Web caching systems to avoid cache misses.
- **Distributed Systems:**
    - Apache Cassandra and Redis use them to reduce expensive operations.
    - Content delivery networks (CDNs) for cache sharing.
- **Security Applications:**
    - Password checkers to prevent common passwords.
    - Malware and phishing URL detection.

<hr class="hr-light" />

#### **Limitations & Considerations**

- **No Deletion Support:** standard Bloom filters do not support deletion (though Counting Bloom filters do).
- **False Positives:** always present, requiring secondary verification for "maybe" results.
- **Fixed Capacity:** performance degrades as the filter approaches capacity.
- **No Data Retrieval:** can only test membership; cannot retrieve the actual stored elements.

---

#### **Implementation**

```cpp
#include <functional>
#include <random>

class BloomFilter {
private:
    vector<bool> bits;
    size_t size;
    int num_hashes;

    // Primary hash function
    size_t hash1(const string& key) const {
        hash<string> hasher;
        return hasher(key) % size;
    }

    // Secondary hash function (DJB2 algorithm)
    size_t hash2(const string& key) const {
        size_t hash = 5381;
        for (char c : key) {
            hash = ((hash << 5) + hash) + c; // hash * 33 + c
        }
        return hash % size;
    }

public:
    BloomFilter(size_t expected_elements, double false_positive_rate = 0.01) {
        // Calculate optimal parameters using Bloom filter formulas
        size = static_cast<size_t>(-(expected_elements * log(false_positive_rate)) /
            pow(log(2), 2));
        num_hashes = static_cast<int>(ceil((static_cast<double>(size) /
            expected_elements) * log(2)));
        bits.resize(size, false);
    }

    void add(const string& key) {
        size_t h1 = hash1(key);
        size_t h2 = hash2(key);

        for (int i = 0; i < num_hashes; i++) {
            size_t index = (h1 + i * h2) % size;
            bits[index] = true;
        }
    }

    bool might_contain(const string& key) const {
        size_t h1 = hash1(key);
        size_t h2 = hash2(key);

        for (int i = 0; i < num_hashes; i++) {
            size_t index = (h1 + i * h2) % size;
            if (!bits[index]) {
                return false;
            }
        }
        return true;
    }

    // Utility function to estimate current false positive rate
    double estimate_false_positive_rate(size_t inserted_elements) const {
        return pow(1 - exp(-static_cast<double>(num_hashes) *
            inserted_elements / size), num_hashes);
    }

    void print_stats(size_t inserted_elements) const {
        cout << "Bloom Filter Statistics:\n";
        cout << " - Bit array size: " << size << " bits\n";
        cout << " - Hash functions: " << num_hashes << "\n";
        cout << " - Elements inserted: " << inserted_elements << "\n";
        cout << " - Estimated FPR: "
            << estimate_false_positive_rate(inserted_elements) * 100
            << "%\n";

        // Count set bits to estimate utilization
        size_t set_bits = count(bits.begin(), bits.end(), true);
        cout << " - Bit utilization: "
            << (static_cast<double>(set_bits) / size) * 100 << "%\n";
    }
};

int main() {
	cout << "=== Bloom Filter Test ===\n\n";

	// Create Bloom filter for 100 elements with 1% target FPR
	BloomFilter bf(100, 0.01);
	
	// Insert some items
	vector<string> items_to_insert = {
	    "apple", "banana", "orange", "grape", "melon",
	    "pear", "kiwi", "berry", "mango", "pineapple",
	    "username_john", "username_sarah", "email_test@example.com",
	    "user_12345", "session_abc123", "file_document.pdf"
	};
	
	cout << "Inserting " << items_to_insert.size() << " items...\n";
	for (const auto& item : items_to_insert) {
	    bf.add(item);
	}
	
	// Test 1: verify inserted items are found
	cout << "\n1. Testing inserted items (should all return true):\n";
	for (const auto& item : items_to_insert) {
	    bool found = bf.might_contain(item);
	    cout << "   " << item << ": " << (found ? "FOUND" : "NOT FOUND");
	    if (!found) cout << "FALSE NEGATIVE!";
	    cout << "\n";
	}
	
	// Test 2: test definitely non-existent items
	vector<string> non_existent_items = {
	    "dragonfruit", "username_nonexistent", "ghost_item",
	    "never_inserted", "phantom_value", "imaginary_fruit"
	};
	
	cout << "\n2. Testing non-existent items:\n";
	for (const auto& item : non_existent_items) {
	    bool found = bf.might_contain(item);
	    cout << "   " << item << ": " << (found ? "MAYBE (false positive)" : "NOT FOUND");
	    cout << "\n";
	}
	
	// Test 3: generate random test items to estimate actual false positive rate
	cout << "\n3. Testing random items to estimate false positive rate:\n";
	random_device rd;
	mt19937 gen(rd());
	uniform_int_distribution<> dis(10000, 99999);
	
	int tests = 1000;
	int false_positives = 0;
	
	for (int i = 0; i < tests; i++) {
	    string random_item = "test_item_" + to_string(dis(gen));
	    if (bf.might_contain(random_item)) {
	        false_positives++;
	    }
	}
	
	double actual_fpr = static_cast<double>(false_positives) / tests * 100;
	cout << "   Tested " << tests << " random non-existent items\n";
	cout << "   False positives: " << false_positives << "\n";
	cout << "   Actual FPR: " << actual_fpr << "%\n";
	
	cout << "\n4. Bloom Filter Statistics:\n";
	bf.print_stats(items_to_insert.size());
}
```

---

#### **Complexity Analysis**

- **Time Complexity:**
    - Insertion: O(k) where k = number of hash functions
    - Query: O(k) where k = number of hash functions
    - Both operations are effectively O(1) since k is typically small (3-10)
- **Space Complexity:**
    - O(m) where m = size of bit array
    - Typically 10-15 bits per element for 1% false positive rate
- **Optimal Parameters:**
    - Hash functions: k = (m/n) × ln(2)
    - Memory: m = - (n × ln(FPR)) / (ln(2))²

---

#### **Comparison with Other Data Structures**

| Feature              | Bloom Filter          | Hash Table | Binary Search Tree |
| -------------------- | --------------------- | ---------- | ------------------ |
| **Space Efficiency** | Excellent             | Good       | Fair               |
| **Insert Time**      | O(1)                  | O(1)       | O(log n)           |
| **Query Time**       | O(1)                  | O(1)       | O(log n)           |
| **False Positives**  | Yes                   | No         | No                 |
| **False Negatives**  | No                    | No         | No                 |
| **Deletion Support** | No (without variants) | Yes        | Yes                |
| **Data Retrieval**   | No                    | Yes        | Yes                |
