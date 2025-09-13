A high-performance write-optimized data structure designed for write-heavy workloads. It uses an append-only write pattern and periodic compaction to maintain efficiency, making it ideal for modern database systems, key-value stores, and big data applications.

- **Core Philosophy:** transform random writes into sequential writes for better performance on modern storage media.
- **Structure:** consists of multiple levels (tiers) of data structures, typically with an in-memory component and multiple disk-based components.
- **Write Pattern:** all writes are initially appended to a log and stored in memory, then later merged with on-disk components.
- **Use Case:** excellent for write-intensive applications (logging, time-series data, IoT) where write throughput is more important than read latency.
```text
Memory:        [MemTable] (sorted in memory)
                  ↓
Disk:    [SSTable Level 0] → [SSTable Level 1] → [SSTable Level 2] → ...
                 (Newest)        (Older)           (Oldest, Largest)
```

- **Write "key5":** appended to WAL, then added to MemTable
- **Read "key5":** check MemTable —> Check Level 0 —> Check Level 1 —> ...
- **Range query for keys 10-20:** must check all levels and merge results

<hr class="hr-light" />

#### **Core Components**
1. **MemTable**
    - In-memory sorted data structure (usually skip list or balanced tree)
    - Serves as a write buffer
    - First place checked for reads
2. **Write-Ahead Log (WAL)**
    - Append-only log on disk
    - Provides durability (survives crashes)
    - Replayed on recovery to rebuild MemTable
3. **Sorted String Tables (SSTables)**
    - Immutable sorted files on disk
    - Organized in multiple levels (L0, L1, L2, ...)
    - Each level has size constraints and compaction policies
4. **Bloom Filters**
    - Probabilistic data structures to avoid unnecessary SSTable lookups
    - Tell us if a key _might_ be in an SSTable or _definitely isn't_
```text
+------------------------+-----------------------------------------+
|      Data Blocks       |               Index & Meta              |
+------------------------+-----------------------------------------+
| [BLOCK 1]              | [BLOOM FILTER]                          |
|   key100: value100     |   - Bits for fast "not found" checks    |
|   key105: value105     |                                         |
|   key110: value110     | [INDEX]                                 |
| ...                    |   - key100 -> offset to Block 1         |
| [BLOCK 2]              |   - key200 -> offset to Block 5         |
|   key200: value200     |   - key300 -> offset to Block 10        |
|   key205: value205     |   ...                                   |
| ...                    |                                         |
| [BLOCK N]              | [META]                                  |
|   key995: value995     |   - Min key, Max key                    |
|   key999: value999     |   - Compression type                    |
|                        |   - Version number                      |
+------------------------+-----------------------------------------+
```

**How a Read Works Internally:**
1. **Check Bloom Filter:** "Is key `key205` in this file?" If "no," skip this entire file instantly.
2. **Binary Search the In-Memory Index:** the index (a small map stored in RAM) says `key205` is in `Block 2`, which starts at byte offset `X`.
3. **Seek and Read One Block:** the system does a single disk seek to offset `X` and reads just that one compressed block of data (e.g., 4KB) into memory.
4. **Scan the Block:** it scans this small, sorted block in memory to find the exact key `key205`.

This is why reads are still efficient despite having many files. You don't read the whole SSTable. You read one 4KB block from it.

<hr class="hr-light" />

#### **How It Works**
**Write Path:**
1. Write appended to WAL for durability
2. Key-value pair inserted into MemTable
3. When MemTable reaches size threshold:
    - MemTable becomes immutable, new MemTable created
    - Immutable MemTable flushed to disk as SSTable (L0)
    - WAL is cleared and new WAL created

**Read Path:**
1. Check MemTable (current and immutable)
2. Check Bloom filters for each SSTable level
3. Check SSTables that might contain the key (from newest to oldest)
4. Return the most recent value found

**Compaction:**
- Process of merging SSTables from one level to the next
- Removes overwritten values and deleted keys
- Maintains sorted order and size constraints
- Different strategies:
	- **Size-Tiered Compaction (STC):** merge smaller SSTables into larger ones when a size threshold is reached; fewer compactions, higher read cost.
	- **Leveled Compaction (LC):** keep one sorted, non-overlapping SSTable per key range per level; more compactions, better read efficiency.
	- **Tiered + Leveled:** hybrid approach; balances write amplification and read performance by combining both strategies.
```text
Before Compaction:
Level 0: [A-D] [C-F] [E-H] [A-C]  ← Overlapping, unsorted
Level 1: [A-M]        [N-Z]       ← Non-overlapping, sorted

After Compaction:  
Level 0: (empty)
Level 1: [A-F] [G-M] [N-Z]        ← All non-overlapping, sorted
```

<hr class="hr-light" />

#### **Advantages & Trade-offs**
**Advantages:**
- **Excellent write throughput** (sequential append operations)
- **Predictable write performance** (no in-place updates)
- **Efficient for high-velocity data ingestion**
- **Natural support for compression** (immutable SSTables)

**Trade-offs:**
- **Higher read latency** (may need to check multiple levels)
- **Compaction overhead** (background I/O and CPU usage)
- **Space amplification** (multiple copies of data during compaction)
- **Write amplification** (data rewritten multiple times during compaction)

<hr class="hr-light" />

#### **Comparison with B+Tree**

|Feature|B+Tree|LSM Tree|
|---|---|---|
|**Write Pattern**|In-place updates|Append-only|
|**Write Performance**|Good (O(log n))|Excellent (O(1) amortized)|
|**Read Performance**|Excellent (O(log n))|Good to variable (O(k log n))|
|**Space Efficiency**|High|Lower (due to compaction)|
|**Background I/O**|Minimal|Significant (compaction)|
|**Best For**|Read-heavy, mixed workloads|Write-heavy, append-only|

---

#### **Important Notes**

- Primary design goal is to **maximize write throughput** on modern hardware
- **Bloom filters** are crucial for read performance optimization
- **Compaction strategy** significantly affects performance characteristics
- The term "tree" is misleading. There is no actual tree structure on disk like a B+Tree. The data is stored in immutable, sorted files **(SSTables)**, organized into **levels**.
	- **Level 0 (L0):** contains the newest SSTables, flushed directly from memory. Files in L0 can have **overlapping key ranges**. Finding a key here requires checking every file.
	- **Level 1 and Below:** SSTables in these levels have **non-overlapping key ranges**. Each SSTable in L1 covers a specific range (e.g., `[A-F]`, `[G-M]`, `[N-Z]`). This allows for efficient lookup by only checking one file per level.
- You can never "delete" data immediately from an immutable SSTable. Instead, a special **tombstone** record is written.
	- This tombstone propagates through the system like a normal write.
	- During a read, if a tombstone is found (and it's the most recent value for that key), the system returns "not found."
	- The **only time the data is physically deleted** is during **compaction**, when the tombstone and all older values for that key are dropped and not written to the new SSTable.
	- **This means a `DELETE` operation actually _consumes_ storage (the tombstone) until compaction cleans it up.** This is a key source of **space amplification**.
- Used in **Cassandra, RocksDB, LevelDB, HBase, and many modern databases**
- Range Queries
	- **Expensive:** must check multiple SSTables across all levels and merge results, especially if key ranges are fragmented.
	- **Memory & I/O:** requires scanning blocks from several SSTables, potentially leading to higher latency compared to point lookups.

---

#### **Implementation**

```cpp
// Bloom Filter implementation
class BloomFilter {
private:
    vector<bool> bits;
    size_t size;
    int num_hashes;

    // Simple hash functions
    size_t hash1(const string& key) const {
        hash<string> hasher;
        return hasher(key) % size;
    }

    size_t hash2(const string& key) const {
        size_t hash_val = 0;
        for (char c : key) {
            hash_val = (hash_val * 31) + c;
        }
        return hash_val % size;
    }

public:
    BloomFilter(size_t expected_elements, double false_positive_rate = 0.01) {
        size = static_cast<size_t>(-(expected_elements * log(false_positive_rate)) / pow(log(2), 2));
        num_hashes = static_cast<int>(ceil((size / expected_elements) * log(2)));
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
};

// SSTable implementation
class SSTable {
public:
    string filename;
    BloomFilter bloom_filter;
    map<string, string> data;
    set<string> keys;

    SSTable(const map<string, string>& data_map, const string& base_path)
        : bloom_filter(data_map.size(), 0.01) {
        // Generate unique filename
        static int counter = 0;
        filename = base_path + "/sstable_" + to_string(counter++) + ".dat";

        // Store data and build bloom filter
        data = data_map;
        for (const auto& entry : data) {
            keys.insert(entry.first);
            bloom_filter.add(entry.first);
        }

        // Write to disk (simplified)
        ofstream file(filename);
        for (const auto& entry : data) {
            file << entry.first << ":" << entry.second << "\n";
        }
    }

    string* get(const string& key) {
        if (!bloom_filter.might_contain(key)) {
            return NULL;
        }

        auto it = data.find(key);
        if (it != data.end()) {
            return &(it->second);
        }
        return NULL;
    }

    ~SSTable() {
        // Clean up file (in real implementation)
        remove(filename.c_str());
    }
};

// LSM Tree implementation
template<typename T = string>
class LSMTree {
private:
    // MemTable (using sorted map for simplicity)
    map<string, T> memtable;
    size_t memtable_size;
    size_t memtable_capacity;

    // SSTable levels
    vector<deque<shared_ptr<SSTable>>> levels;
    string storage_path;

    // Write-ahead log (simplified)
    ofstream wal;

    void flush_memtable() {
        if (memtable.empty()) return;

        // Create SSTable from memtable
        map<string, string> data;
        for (const auto& entry : memtable) {
            data[entry.first] = entry.second;
        }

        auto sstable = make_shared<SSTable>(data, storage_path);

        // Add to level 0
        if (levels.empty()) {
            levels.push_back(deque<shared_ptr<SSTable>>());
        }
        levels[0].push_front(sstable);

        // Clear memtable and WAL
        memtable.clear();
        memtable_size = 0;
        wal.close();
        wal.open(storage_path + "/wal.log", ios::trunc);

        // Trigger compaction if needed
        if (levels[0].size() > 4) { // Simple threshold
            compact(0);
        }
    }

    void compact(size_t level) {
        if (level >= levels.size()) {
            levels.push_back(deque<shared_ptr<SSTable>>());
        }

        if (level + 1 >= levels.size()) {
            levels.push_back(deque<shared_ptr<SSTable>>());
        }

        // Merge all SSTables from current level
        map<string, string> merged_data;
        for (auto& sstable : levels[level]) {
            for (const auto& entry : sstable->data) {
                merged_data[entry.first] = entry.second; // Newer values overwrite older ones
            }
        }

        // Create new SSTable for next level
        if (!merged_data.empty()) {
            auto new_sstable = make_shared<SSTable>(merged_data, storage_path);
            levels[level + 1].push_front(new_sstable);
        }

        // Clear current level
        levels[level].clear();
    }

public:
    LSMTree(size_t capacity = 1000, const string& path = "/tmp/lsm")
        : memtable_capacity(capacity), memtable_size(0), storage_path(path) {
        // Initialize WAL
        wal.open(path + "/wal.log", ios::app);
    }

    void put(const string& key, const T& value) {
        // Write to WAL
        wal << key << ":" << value << "\n";
        wal.flush();

        // Update memtable
        memtable[key] = value;
        memtable_size++;

        // Flush if capacity reached
        if (memtable_size >= memtable_capacity) {
            flush_memtable();
        }
    }

    T* get(const string& key) {
        // Check memtable first
        auto mem_it = memtable.find(key);
        if (mem_it != memtable.end()) {
            return &(mem_it->second);
        }

        // Check SSTables from newest to oldest
        for (auto& level : levels) {
            for (auto& sstable : level) {
                string* value = sstable->get(key);
                if (value != NULL) {
                    // Cast from string* to T* - this is safe for T=string
                    return reinterpret_cast<T*>(value);
                }
            }
        }

        return NULL;
    }

    vector<pair<string, T>> range_scan(const string& start, const string& end) {
        vector<pair<string, T>> result;

        // Check memtable
        auto it = memtable.lower_bound(start);
        while (it != memtable.end() && it->first <= end) {
            result.push_back(make_pair(it->first, it->second));
            ++it;
        }

        // Check SSTables (simplified - in real implementation would need to merge)
        for (auto& level : levels) {
            for (auto& sstable : level) {
                for (const auto& key : sstable->keys) {
                    if (key >= start && key <= end) {
                        string* value = sstable->get(key);
                        if (value != NULL) {
                            // Check if we already have a newer version
                            bool found_newer = false;
                            for (auto& result_entry : result) {
                                if (result_entry.first == key) {
                                    found_newer = true;
                                    break;
                                }
                            }
                            if (!found_newer) {
                                result.push_back(make_pair(key, *value));
                            }
                        }
                    }
                }
            }
        }

        // Sort by key
        sort(result.begin(), result.end(),
            [](const pair<string, T>& a, const pair<string, T>& b) {
                return a.first < b.first;
            });

        return result;
    }

    void print_stats() const {
        cout << "LSM Tree Statistics:\n";
        cout << "MemTable size: " << memtable_size << "/" << memtable_capacity << "\n";
        cout << "Levels: " << levels.size() << "\n";
        for (size_t i = 0; i < levels.size(); i++) {
            cout << "  Level " << i << ": " << levels[i].size() << " SSTables\n";
        }
    }
};

int main() {
    LSMTree<string> lsm_tree(3); // Small capacity for demonstration
    
    // Insert some values
    vector<pair<string, string>> test_data = {
        make_pair("key1", "value1"),
        make_pair("key2", "value2"),
        make_pair("key3", "value3"),
        make_pair("key4", "value4"),
        make_pair("key5", "value5")
    };
    
    for (const auto& entry : test_data) {
        lsm_tree.put(entry.first, entry.second);
        cout << "Inserted: " << entry.first << " -> " << entry.second << "\n";
    }
    
    // Search operations
    cout << "\nSearch operations:\n";
    const char* search_keys[] = {"key1", "key3", "key6"};
    for (int i = 0; i < 3; i++) {
        string* value = lsm_tree.get(search_keys[i]);
        if (value != NULL) {
            cout << "Found " << search_keys[i] << ": " << *value << "\n";
        } else {
            cout << search_keys[i] << ": Not found\n";
        }
    }
    
    // Range query
    cout << "\nRange query [key2, key4]:\n";
    auto range_result = lsm_tree.range_scan("key2", "key4");
    for (const auto& entry : range_result) {
        cout << entry.first << " -> " << entry.second << "\n";
    }
    
    // Print statistics
    cout << "\n";
    lsm_tree.print_stats();
    
    return 0;
}
```

---

#### **Complexity Analysis**

- **Write Operations:** O(1) amortized (append to WAL and MemTable)
- **Read Operations:** O(k log n) where k = number of SSTables checked
- **Space Complexity:** O(n) + compaction overhead
- **Compaction:** O(n) per compaction cycle, occurs in background
- **Range Queries:** O(n) in worst case, depends on data distribution

**Performance Characteristics:**
- **Write-optimized:** excellent for high-throughput writes
- **Read-optimized:** good for point queries with Bloom filters
- **Compaction overhead:** background process that affects overall performance
- **Tunable:** performance can be adjusted through compaction strategies and size thresholds