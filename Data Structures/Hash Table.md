A data structure that implements an associative array, mapping keys to values. It uses a hash function to compute an index into an array of "buckets" or "slots" from which the desired value can be found. It provides average O(1) time complexity for insertions, deletions, and lookups.

- **Core Problem Solved:** extends the O(1) random access capability of arrays to keys of any data type (strings, objects, etc.) by using [hashing](../Algorithms/Hashing.md) to translate a key into an array index.
- **Basic Structure:** consists of a **backing array** (allocated in RAM for byte-addressability) and a **hash function**. Each array index holds a bucket, which may contain multiple key-value pairs due to collisions.
- **Operations:**
    - **Insertion:** hash the key to get an index, store the key-value pair at that index (handling collisions as necessary).
    - **Lookup:** hash the key to get an index, search the bucket at that index for the key.
    - **Deletion:** hash the key to get an index, remove the key-value pair from the bucket.

```text
Hash Table with size m=10, using modulo hashing and chaining.

Insert ("apple", "red"):
   hash("apple") = 50 -> index = 50 % 10 = 0
   Store ("apple", "red") in bucket 0.

Insert ("banana", "yellow"):
   hash("banana") = 33 -> index = 33 % 10 = 3
   Store ("banana", "yellow") in bucket 3.

Insert ("grape", "purple"):
   hash("grape") = 25 -> index = 25 % 10 = 5
   Store ("grape", "purple") in bucket 5.

Lookup "apple":
   hash("apple") = 50 -> index = 0
   Find "apple" in bucket 0 -> returns "red".

Lookup "orange":
   hash("orange") = 63 -> index = 3
   Search bucket 3 -> no "orange" found -> key does not exist.
```

<hr class="hr-light" />

#### **Collision Resolution Strategies**
- **Separate Chaining:** each bucket is a linked list (or other structure) containing all key-value pairs that hash to that index. Simple but requires extra memory for pointers.
- **Open Addressing:** all entries are stored directly in the array itself. When a collision occurs, the table is probed sequentially until an empty slot is found.
    - **Linear Probing:** check the next slot `(index + 1) % m`.
    - **Quadratic Probing:** check slots `(index + i²) % m` for i=1, 2, 3...
    - **Double Hashing:** use a second hash function to determine the probe step.

<hr class="hr-light" />

#### **Mathematical Foundation & Performance**
The performance of a hash table is directly tied to its **load factor** (λ).
- **Load Factor:** λ = n / m, where `n` is the number of entries and `m` is the number of buckets.
- **Time Complexity:**
    - **Average Case (Insert, Delete, Lookup):** O(1), assuming a good hash function and a low load factor (typically λ < 0.7).
    - **Worst Case:** O(n), if many collisions occur (e.g., all keys hash to the same bucket).
- **Resizing (Rehashing):** when the load factor exceeds a threshold, the table must be resized (usually doubled) and all existing entries must be rehashed and inserted into the new, larger table. This is an O(n) operation but amortized over many insertions, it keeps average operations O(1).

<hr class="hr-light" />

#### **Implementation Considerations**
- **Memory vs. Performance:** a larger table (lower load factor) reduces collisions but increases memory overhead. The "1-2% rule" from [Bloom Filters](./Bloom%20Filter.md) doesn't apply directly; instead, aim for a load factor of 0.5 to 0.75.
- **Hash Function Quality:** critical for performance. A poor function leads to clustering and degrades performance to O(n).
- **Memory Locality:** open addressing can have better cache performance than chaining because data is stored in a contiguous array.

<hr class="hr-light" />

#### **Security Considerations**
- **Hash Collision Attacks (Hash DoS):**  
    In web applications, an attacker can craft many keys that hash to the same value using a predictable hash function. This forces the hash table into worst-case O(n) lookup/insert behavior, causing server slowdowns or denial-of-service (DoS).
- **Real-World Response:**
    - Python, Ruby, and other languages switched to randomized or cryptographic-quality hash functions (like **SipHash**) to defend against collision attacks.
    - Many frameworks also impose limits on the maximum number of entries or chain lengths in a hash table to mitigate attack surfaces.
- **Trade-off:**  
    Cryptographic hashes are slower than simple modular arithmetic, so they’re often used selectively where untrusted input is expected.

<hr class="hr-light" />

#### **Advanced Variants & Concepts**
- **Consistent Hashing:** as detailed in the transcript, this variant minimizes the number of keys that need to be remapped when a hash table is resized (e.g., when adding or removing servers in a distributed system). Keys are hashed to a circle, and each server is responsible for a range of hashes. When a server is added, only keys from its immediate neighbor are moved, not the entire table.
- **Cuckoo Hashing:** uses multiple hash functions and tables to guarantee O(1) worst-case lookup time.

<hr class="hr-light" />

#### **Applications in Databases & Distributed Systems**
Hash tables are a fundamental building block for high-performance systems.

- **Symbol Tables in Compilers:**  hash tables store variable and function identifiers with O(1) lookup during parsing and semantic analysis.
- **DNS Caching:**  operating systems and DNS resolvers use hash tables to cache domain-to-IP lookups for fast resolution.
- **In-Memory Key-Value Stores:**  [Redis and Memcached](../Database/14.%20NoSQL%20Cache%20Architecture%20(In-Memory%20Databases).md) are essentially distributed hash tables optimized for low-latency key-based retrieval.
- **Database Joins (Hash Join):** this is a primary application. The process, as described, is:
    1. **Build Phase:** the smaller of the two tables to be joined is scanned. A hash table is built in RAM using the join key. This is a costly O(n) operation, but it's a one-time cost.
    2. **Probe Phase:** the larger table is scanned. For each row, the join key is hashed. The resulting index is used to probe the in-memory hash table in O(1) time to find matching rows. This avoids expensive O(n) scans for each key.
- **Caching:** in-memory key-value stores (like Redis) are essentially distributed hash tables.
- **Sharding & Partitioning:** distributed databases use hashing to determine which partition or server a piece of data belongs to. A key is hashed, and the output determines the responsible shard. The major limitation, as discussed, is that adding/removing a shard (changing `m` in the modulo operation) requires a massive, expensive reshuffling of data, a problem mitigated by **consistent hashing**.

<hr class="hr-light" />

#### **Limitations & Considerations**
- **Memory Bound:** the entire hash table (or a significant portion) must fit in RAM for optimal performance. This is the "biggest, biggest problem" highlighted in the transcript. If the dataset is too large, the hash table becomes useless for in-memory operations.
- **Cost of Building:** constructing the hash table is an O(n) operation. This is acceptable for operations like joins where the cost is amortized over many probes, but it's a fixed cost that must be considered.
- **Resizing Overhead:** dynamic resizing (rehashing) is a computationally expensive operation that can cause latency spikes.
- **Clustering:** poor hash functions or specific data patterns can lead to clusters of entries, degrading performance to O(n).

---

#### **Implementation (Separate Chaining)**

```cpp
template<typename Key, typename Value, typename Hash = hash<Key>>
class HashMap {
private:
    static constexpr double MAX_LOAD_FACTOR = 0.75;
    static constexpr size_t INITIAL_CAPACITY = 16;
    
    struct Node {
        Key key;
        Value value;
        size_t cached_hash;  // Cache hash to avoid re-computation during resize
        
        Node(const Key& k, const Value& v, size_t h) 
            : key(k), value(v), cached_hash(h) {}
    };
    
    vector<list<Node>> buckets_;
    size_t size_ = 0;
    Hash hasher_;
    
    size_t bucket_index(size_t hash) const {
        return hash & (buckets_.size() - 1);  // Power-of-two sizing for fast modulo
    }
    
    void rehash(size_t new_capacity) {
        vector<list<Node>> new_buckets(new_capacity);
        
        for (auto& bucket : buckets_) {
            for (auto& node : bucket) {
                size_t new_index = node.cached_hash & (new_capacity - 1);
                new_buckets[new_index].splice(new_buckets[new_index].end(), 
                                            bucket, 
                                            find(bucket.begin(), bucket.end(), node));
            }
        }
        
        buckets_ = move(new_buckets);
    }
    
    void grow_if_needed() {
        if (static_cast<double>(size_) / buckets_.size() > MAX_LOAD_FACTOR) {
            rehash(buckets_.size() * 2);
        }
    }

public:
    HashMap() : buckets_(INITIAL_CAPACITY) {}
    
    explicit HashMap(size_t capacity) : buckets_(max(INITIAL_CAPACITY, capacity)) {}
    
    // Core API
    bool insert(const Key& key, Value value) {
        grow_if_needed();
        
        size_t hash = hasher_(key);
        size_t index = bucket_index(hash);
        
        auto& bucket = buckets_[index];
        for (auto& node : bucket) {
            if (node.key == key) {
                node.value = move(value);  // Update existing
                return false;
            }
        }
        
        bucket.emplace_back(key, move(value), hash);
        size_++;
        return true;
    }
    
    // Modern C++: perfect forwarding
    template<typename K, typename V>
    bool emplace(K&& key, V&& value) {
        grow_if_needed();
        
        size_t hash = hasher_(key);
        size_t index = bucket_index(hash);
        
        auto& bucket = buckets_[index];
        for (auto& node : bucket) {
            if (node.key == key) {
                node.value = forward<V>(value);
                return false;
            }
        }
        
        bucket.emplace_back(forward<K>(key), forward<V>(value), hash);
        size_++;
        return true;
    }
    
    Value* find(const Key& key) {
        size_t hash = hasher_(key);
        size_t index = bucket_index(hash);
        
        for (auto& node : buckets_[index]) {
            if (node.key == key) {
                return &node.value;
            }
        }
        return nullptr;
    }
    
    const Value* find(const Key& key) const {
        return const_cast<HashMap*>(this)->find(key);
    }
    
    Value& operator[](const Key& key) {
        if (auto* val = find(key)) {
            return *val;
        }
        insert(key, Value{});
        return *find(key);
    }
    
    bool erase(const Key& key) {
        size_t hash = hasher_(key);
        size_t index = bucket_index(hash);
        
        auto& bucket = buckets_[index];
        for (auto it = bucket.begin(); it != bucket.end(); ++it) {
            if (it->key == key) {
                bucket.erase(it);
                size_--;
                return true;
            }
        }
        return false;
    }
    
    // Capacity
    size_t size() const { return size_; }
    bool empty() const { return size_ == 0; }
    size_t bucket_count() const { return buckets_.size(); }
    
    // Memory management
    void clear() {
        for (auto& bucket : buckets_) {
            bucket.clear();
        }
        size_ = 0;
    }
    
    void reserve(size_t capacity) {
        size_t required_buckets = static_cast<size_t>(capacity / MAX_LOAD_FACTOR) + 1;
        if (required_buckets > buckets_.size()) {
            rehash(required_buckets);
        }
    }
    
    // Statistics (useful for debugging)
    struct Stats {
        size_t size;
        size_t bucket_count;
        double load_factor;
        size_t max_chain_length;
        size_t empty_buckets;
    };
    
    Stats get_stats() const {
        Stats stats{};
        stats.size = size_;
        stats.bucket_count = buckets_.size();
        stats.load_factor = static_cast<double>(size_) / buckets_.size();
        
        for (const auto& bucket : buckets_) {
            if (bucket.empty()) {
                stats.empty_buckets++;
            }
            stats.max_chain_length = max(stats.max_chain_length, bucket.size());
        }
        
        return stats;
    }
    
    // Iterators (simplified version)
    class iterator {
    private:
        using BucketsType = vector<list<Node>>;
        BucketsType* buckets_;
        size_t bucket_index_;
        typename list<Node>::iterator node_it_;
        
        void advance_to_next_bucket() {
            while (bucket_index_ < buckets_->size() && 
                   node_it_ == (*buckets_)[bucket_index_].end()) {
                bucket_index_++;
                if (bucket_index_ < buckets_->size()) {
                    node_it_ = (*buckets_)[bucket_index_].begin();
                }
            }
        }
        
    public:
        iterator(BucketsType* buckets, size_t index, typename list<Node>::iterator it)
            : buckets_(buckets), bucket_index_(index), node_it_(it) {
            advance_to_next_bucket();
        }
        
        pair<const Key&, Value&> operator*() {
            return {node_it_->key, node_it_->value};
        }
        
        iterator& operator++() {
            ++node_it_;
            advance_to_next_bucket();
            return *this;
        }
        
        bool operator!=(const iterator& other) const {
            return node_it_ != other.node_it_;
        }
    };
    
    iterator begin() { return iterator(&buckets_, 0, buckets_[0].begin()); }
    iterator end() { return iterator(&buckets_, buckets_.size(), {}); }
};

// Example usage
int main() {
    HashMap<string, int> map;
    
    // Insert elements
    map.insert("apple", 5);
    map.emplace("banana", 3);  // More efficient
    
    // Access elements
    map["cherry"] = 7;  // Auto-inserts if not found
    
    // Find elements
    if (auto* value = map.find("apple")) {
        cout << "Found apple: " << *value << "\n";
    }
    
    // Iterate
    for (auto& [key, value] : map) {
        cout << key << ": " << value << "\n";
    }
    
    // Get statistics
    auto stats = map.get_stats();
    cout << "Load factor: " << stats.load_factor << "\n";
    cout << "Max chain: " << stats.max_chain_length << "\n";
    
    return 0;
}
```

<hr class="hr-light" />

#### **Complexity Analysis**
- **Time Complexity (Average Case):**
    - Insertion: O(1)
    - Deletion: O(1)
    - Lookup: O(1)
- **Time Complexity (Worst Case):**
    - All Operations: O(n) (with many collisions)
- **Space Complexity:** O(n + m), where n is the number of elements and m is the number of buckets.

|Operation|Average Case|Worst Case|Notes|
|---|---|---|---|
|**Insert**|O(1)|O(n)|Amortized O(1) with resizing|
|**Lookup**|O(1)|O(n)|Depends on hash distribution|
|**Delete**|O(1)|O(n)|Depends on collision resolution|
|**Resize**|O(n)|O(n)|Amortized over many operations|
|**Space**|O(n + m)|O(n + m)|m buckets, n elements|

**Note:** the O(1) average case assumes a good hash function and load factor λ < threshold (typically λ < 0.75). The actual expected cost is O(1 + λ), but since λ is kept constant via resizing, this simplifies to O(1).