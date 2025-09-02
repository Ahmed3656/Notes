#### **B-Tree (Balanced Tree)**
A self-balancing tree data structure that maintains sorted data and allows for efficient insertion, deletion, and search operations. It is optimized for systems that read and write large blocks of data, like databases and file systems.
- **Node Structure:** each node can contain more than one key and more than two child pointers. The number of keys defines the node's capacity.
- **Properties:**
	- All leaves are at the same depth.
	- A node with `k` keys has `k+1` children.
	- Keys in a node are sorted, and the subtree between two keys `K_i` and `K_{i+1}` contains all keys in the range `(K_i, K_{i+1})`.
- **Data Storage:** both internal nodes and leaf nodes store data (or pointers to data).
- **Use Case:** primarily used when random access is expensive (e.g., disk-based storage) to minimize the number of disk I/O operations.

```text
      [50]
     /     \
[10, 30]   [70, 90]    <-- Internal Nodes can hold data!
 |   |     |   |   |
[5] [20] [40][60][100] <-- Leaf Nodes
```
- **Search for 40:** you find it in an internal node. The search stops early.
- **Range query for 30-60:** you must jump between different branches of the tree. Slow.

<hr class="hr-light" />

#### **B+Tree**
An advanced variation of the B-Tree that is the _de facto_ standard index structure in modern database systems and file systems.
- **Node Structure:** similar to a B-Tree, with a crucial distinction in the role of nodes.
- **Key Differences from B-Tree:**
	- **Data is only stored in leaf nodes.** Internal nodes only store keys and act as a roadmap to the leaves.
	- **Leaf nodes are linked together in a sorted, bidirectional linked list.**
- **Advantages:**
	- **Efficient Range Queries & Full Table Scans:** the linked list of leaves allows for very fast in-order traversals and range queries without needing to traverse back up the tree.
	- **Higher Fan-out:** since internal nodes only hold keys (not data pointers), they can hold more keys per node, resulting in a shorter, bushier tree and fewer disk seeks.
	- **More Stable:** all insertions and deletions only happen at the leaf level, making the structure more predictable.

```text
      [50]                <-- Internal Node: Only Keys & Pointers
     /     \
[10, 30]   [70, 90]       <-- Internal Nodes: Only Keys & Pointers
 |   |     |   |   |
[5]→[10,20]→[30,40]→[50,60]→[70,80]→[90,100]  <-- Leaf Nodes: Keys + Data → Linked
```
- **Search for 40:** you must traverse to a leaf node to find it.
- **Range query for 30-60:** find 30, then just walk the linked list → 40, 50, 60. **Blazing fast!**

<hr class="hr-light" />

#### **Comparison & Summary**

|Feature|B-Tree|B+Tree|
|---|---|---|
|**Data Pointers**|In both internal and leaf nodes.|**Only in leaf nodes.** Internal nodes are purely navigational.|
|**Leaf Node Links**|No|**Yes.** Leaves are linked in a sequential, doubly-linked list.|
|**Tree Height**|Slightly taller for the same order|Shorter and bushier due to higher fan-out in internal nodes.|
|**Access Speed**|Faster for accessing a single key|Comparable for a single key; **significantly faster for range queries.**|
|**Used in**|Some databases, file systems|**Virtually all modern relational databases** (MySQL, PostgreSQL, etc.).|

---

#### **Important Notes**

- The "B" stands for "Balanced," not "Binary." All leaf nodes are at the same depth.
- The primary goal of both structures is to **minimize disk I/O**. A single disk read can retrieve a large node containing many keys.
- The process of splitting a full node during insertion is what keeps the tree balanced.
- For **point queries** (finding a single record), both are excellent and perform similarly.
- For **range queries** (finding all records between X and Y), **B+Tree is vastly superior** due to the linear traversal of linked leaves.

---

#### **Implementations**

- **B-Tree**
	```cpp
	template<typename T, int MinDegree = 3>
	class BTree {
	private:
	    struct Node {
	        vector<T> keys;
	        vector<unique_ptr<Node>> children;
	        bool is_leaf;
	
	        explicit Node(bool leaf = true) : is_leaf(leaf) {
	            keys.reserve(2 * MinDegree - 1);
	            if (!leaf) {
	                children.reserve(2 * MinDegree);
	            }
	        }
	
	        bool is_full() const { // O(1) - constant check
	            return keys.size() == 2 * MinDegree - 1;
	        }
	
	        bool is_minimal() const { // O(1) - constant check
	            return keys.size() == MinDegree - 1;
	        }
	    };
	
	    unique_ptr<Node> root;
	
	    void split_child(Node* parent, size_t index) { // O(t) where t = MinDegree
	        auto full_child = parent->children[index].get();
	        auto new_child = make_unique<Node>(full_child->is_leaf);
	
	        // Move half the keys to new child - O(t)
	        auto mid = MinDegree - 1;
	        new_child->keys.assign(
	            full_child->keys.begin() + mid + 1,
	            full_child->keys.end()
	        );
	
	        // Move children if not leaf - O(t)
	        if (!full_child->is_leaf) {
	            new_child->children.reserve(MinDegree);
	            for (size_t i = mid + 1; i < full_child->children.size(); ++i) {
	                new_child->children.push_back(move(full_child->children[i]));
	            }
	            full_child->children.resize(mid + 1);
	        }
	
	        // Promote middle key to parent - O(1)
	        T promoted_key = full_child->keys[mid];
	        full_child->keys.resize(mid);
	
	        // Insert promoted key and new child into parent - O(t)
	        parent->keys.insert(parent->keys.begin() + index, promoted_key);
	        parent->children.insert(
	            parent->children.begin() + index + 1,
	            move(new_child)
	        );
	    }
	
	    void insert_non_full(Node* node, const T& key) { // O(t * log_t(n)) amortized
	        int i = static_cast<int>(node->keys.size()) - 1;
	
	        if (node->is_leaf) {
	            // Insert in sorted order - O(t)
	            node->keys.resize(node->keys.size() + 1);
	            while (i >= 0 && key < node->keys[i]) {
	                node->keys[i + 1] = move(node->keys[i]);
	                --i;
	            }
	            node->keys[i + 1] = key;
	        }
	        else {
	            // Find child to recurse into - O(t)
	            while (i >= 0 && key < node->keys[i]) {
	                --i;
	            }
	            ++i;
	
	            // Split if necessary, then recurse - O(t) + recursive call
	            if (node->children[i]->is_full()) {
	                split_child(node, i);
	                if (key > node->keys[i]) {
	                    ++i;
	                }
	            }
	            insert_non_full(node->children[i].get(), key);
	        }
	    }
	
	    Node* search_node(Node* node, const T& key) const { // O(log_t(n)) where n = total keys
	        if (!node) return nullptr;
	
	        // Linear search within node - O(t)
	        size_t i = 0;
	        while (i < node->keys.size() && key > node->keys[i]) {
	            ++i;
	        }
	
	        // Key found in current node - O(1)
	        if (i < node->keys.size() && key == node->keys[i]) {
	            return node;
	        }
	
	        // Reached leaf without finding key - O(1)
	        if (node->is_leaf) {
	            return nullptr;
	        }
	
	        // Recurse to appropriate child - O(log_t(n))
	        return search_node(node->children[i].get(), key);
	    }
	
	    void inorder_traversal(Node* node, vector<T>& result) const { // O(n) - visits all nodes once
	        if (!node) return;
	
	        size_t i = 0;
	        for (i = 0; i < node->keys.size(); ++i) {
	            if (!node->is_leaf) {
	                inorder_traversal(node->children[i].get(), result);
	            }
	            result.push_back(node->keys[i]);
	        }
	
	        if (!node->is_leaf) {
	            inorder_traversal(node->children[i].get(), result);
	        }
	    }

	public:
	    BTree() : root(make_unique<Node>()) {} // O(1)
	
	    void insert(const T& key) { // O(log_t(n)) amortized, O(t * log_t(n)) worst case
	        if (root->is_full()) {
	            auto new_root = make_unique<Node>(false);
	            new_root->children.push_back(move(root));
	            split_child(new_root.get(), 0);
	            root = move(new_root);
	        }
	        insert_non_full(root.get(), key);
	    }
	
	    bool search(const T& key) const { // O(log_t(n)) where n = total keys
	        return search_node(root.get(), key) != nullptr;
	    }
	
	    vector<T> get_sorted_keys() const { // O(n) - must visit all keys
	        vector<T> result;
	        inorder_traversal(root.get(), result);
	        return result;
	    }
	
	    void print() const { // O(n) - calls get_sorted_keys()
	        auto keys = get_sorted_keys();
	        cout << "B-Tree keys: ";
	        for (const auto& key : keys) {
	            cout << key << " ";
	        }
	        cout << "\n";
	    }
	};
	
	
	int main() {
		 BTree<int> btree;

		 // Insert some values
		 vector<int> values = { 10, 20, 5, 6, 12, 30, 7, 17, 25, 40, 35, 50 };
		 for (int val : values) {
		     btree.insert(val); // O(log_t(n)) per insertion
		     cout << "Inserted " << val << ": ";
		     btree.print(); // O(n) per print
		 }
		
		 // Search operations
		 cout << "\nSearch operations:\n";
		 for (int val : {5, 15, 25, 60}) {
		     cout << "Search " << val << ": "
		         << (btree.search(val) ? "Found" : "Not found") << "\n"; // O(log_t(n)) per search
		 }
		 
		 return 0;
	}
	```

<hr class="hr-light" />

- **B+Tree**
	```cpp
	template<typename T, int MinDegree = 3>
	class BPlusTree {
	private:
	    struct Node {
	        vector<T> keys;
	        bool is_leaf;
	
	        explicit Node(bool leaf = true) : is_leaf(leaf) {
	            keys.reserve(2 * MinDegree - 1);
	        }
	
	        virtual ~Node() = default;
	
	        bool is_full() const { // O(1) - constant check
	            return keys.size() == 2 * MinDegree - 1;
	        }
	
	        bool is_minimal() const { // O(1) - constant check
	            return keys.size() == MinDegree - 1;
	        }
	    };
	
	    struct InternalNode : public Node {
	        vector<unique_ptr<Node>> children;
	
	        InternalNode() : Node(false) { // O(1)
	            children.reserve(2 * MinDegree);
	        }
	    };
	
	    struct LeafNode : public Node {
	        unique_ptr<LeafNode> next;
	
	        LeafNode() : Node(true), next(nullptr) {} // O(1)
	    };
	
	    unique_ptr<Node> root;
	    LeafNode* leftmost_leaf;
	
	    void split_child(InternalNode* parent, size_t index) { // O(t) where t = MinDegree
	        auto child = parent->children[index].get();
	
	        if (child->is_leaf) {
	            auto leaf_child = static_cast<LeafNode*>(child);
	            auto new_leaf = make_unique<LeafNode>();
	
	            // Copy keys to new leaf - O(t)
	            auto mid = MinDegree;
	            new_leaf->keys.assign(
	                leaf_child->keys.begin() + mid,
	                leaf_child->keys.end()
	            );
	
	            // Update linked list - O(1)
	            new_leaf->next = move(leaf_child->next);
	            leaf_child->next = unique_ptr<LeafNode>(
	                static_cast<LeafNode*>(new_leaf.release())
	            );
	
	            leaf_child->keys.resize(mid);
	
	            // Promote first key of new leaf - O(t) for insertion
	            T promoted_key = static_cast<LeafNode*>(leaf_child->next.get())->keys[0];
	            parent->keys.insert(parent->keys.begin() + index, promoted_key);
	            parent->children.insert(
	                parent->children.begin() + index + 1,
	                unique_ptr<Node>(leaf_child->next.get())
	            );
	            leaf_child->next.release(); // Transfer ownership to parent
	        }
	        else {
	            auto internal_child = static_cast<InternalNode*>(child);
	            auto new_internal = make_unique<InternalNode>();
	
	            // Move keys and children - O(t)
	            auto mid = MinDegree - 1;
	            new_internal->keys.assign(
	                internal_child->keys.begin() + mid + 1,
	                internal_child->keys.end()
	            );
	
	            for (size_t i = mid + 1; i < internal_child->children.size(); ++i) {
	                new_internal->children.push_back(move(internal_child->children[i]));
	            }
	            internal_child->children.resize(mid + 1);
	
	            T promoted_key = internal_child->keys[mid];
	            internal_child->keys.resize(mid);
	
	            // Insert into parent - O(t)
	            parent->keys.insert(parent->keys.begin() + index, promoted_key);
	            parent->children.insert(
	                parent->children.begin() + index + 1,
	                move(new_internal)
	            );
	        }
	    }
	
	    void insert_non_full(Node* node, const T& key) { // O(t * log_t(n)) amortized
	        if (node->is_leaf) {
	            auto leaf = static_cast<LeafNode*>(node);
	            // Binary search + insertion - O(t)
	            auto pos = lower_bound(leaf->keys.begin(), leaf->keys.end(), key);
	            leaf->keys.insert(pos, key);
	        }
	        else {
	            auto internal = static_cast<InternalNode*>(node);
	            // Find appropriate child - O(t)
	            auto i = upper_bound(internal->keys.begin(), internal->keys.end(), key)
	                - internal->keys.begin();
	
	            // Split if needed, then recurse - O(t) + recursive call
	            if (internal->children[i]->is_full()) {
	                split_child(internal, i);
	                if (key > internal->keys[i]) {
	                    ++i;
	                }
	            }
	            insert_non_full(internal->children[i].get(), key);
	        }
	    }
	
	    LeafNode* find_leaf(const T& key) const { // O(log_t(n)) - traverse to leaf
	        Node* current = root.get();
	
	        while (!current->is_leaf) {
	            auto internal = static_cast<InternalNode*>(current);
	            // Binary search within node - O(t)
	            auto i = upper_bound(internal->keys.begin(), internal->keys.end(), key)
	                - internal->keys.begin();
	            current = internal->children[i].get();
	        }
	
	        return static_cast<LeafNode*>(current);
	    }
	
	public:
	    BPlusTree() : root(make_unique<LeafNode>()), leftmost_leaf(nullptr) { // O(1)
	        leftmost_leaf = static_cast<LeafNode*>(root.get());
	    }
	
	    void insert(const T& key) { // O(log_t(n)) amortized, O(t * log_t(n)) worst case
	        if (root->is_full()) {
	            auto new_root = make_unique<InternalNode>();
	            new_root->children.push_back(move(root));
	            split_child(new_root.get(), 0);
	            root = move(new_root);
	        }
	        insert_non_full(root.get(), key);
	    }
	
	    bool search(const T& key) const { // O(log_t(n)) - find leaf + binary search
	        auto leaf = find_leaf(key);
	        return binary_search(leaf->keys.begin(), leaf->keys.end(), key);
	    }
	
	    vector<T> range_query(const T& start, const T& end) const { // O(log_t(n) + k) where k = result size
	        vector<T> result;
	        auto leaf = find_leaf(start); // O(log_t(n)) to find starting leaf
	
	        // Sequential scan through linked leaves - O(k)
	        while (leaf) {
	            for (const auto& key : leaf->keys) {
	                if (key >= start && key <= end) {
	                    result.push_back(key);
	                }
	                else if (key > end) {
	                    return result;
	                }
	            }
	            leaf = leaf->next.get();
	        }
	
	        return result;
	    }
	
	    vector<T> get_sorted_keys() const { // O(n) - sequential scan of all leaves
	        vector<T> result;
	        auto leaf = leftmost_leaf;
	
	        while (leaf) {
	            for (const auto& key : leaf->keys) {
	                result.push_back(key);
	            }
	            leaf = leaf->next.get();
	        }
	
	        return result;
	    }
	
	    void print() const { // O(n) - calls get_sorted_keys()
	        auto keys = get_sorted_keys();
	        cout << "B+Tree keys: ";
	        for (const auto& key : keys) {
	            cout << key << " ";
	        }
	        cout << "\n";
	    }
	
	    void print_range(const T& start, const T& end) const { // O(log_t(n) + k) where k = result size
	        auto range_keys = range_query(start, end);
	        cout << "Range [" << start << ", " << end << "]: ";
	        for (const auto& key : range_keys) {
	            cout << key << " ";
	        }
	        cout << "\n";
	    }
	};
	
	
	int main() {
		BPlusTree<int> bplus_tree;

		// Insert same values
		vector<int> values = { 10, 20, 5, 6, 12, 30, 7, 17, 25, 40, 35, 50 };
		for (int val : values) {
		    bplus_tree.insert(val); // O(log_t(n)) per insertion
		    cout << "Inserted " << val << ": ";
		    bplus_tree.print(); // O(n) per print
		}
		
		// Search operations
		cout << "\nSearch operations:\n";
		for (int val : {5, 15, 25, 60}) {
		    cout << "Search " << val << ": "
		        << (bplus_tree.search(val) ? "Found" : "Not found") << "\n"; // O(log_t(n)) per search
		}
		
		// Range queries (B+Tree advantage)
		cout << "\nRange queries:\n";
		bplus_tree.print_range(10, 25); // O(log_t(n) + k) where k = keys in range
		bplus_tree.print_range(15, 35); // O(log_t(n) + k)
		bplus_tree.print_range(1, 8);   // O(log_t(n) + k)
		
		return 0;
	}
	```

---

#### **Complexity Analysis**
- **B-Tree Operations:**
	- Search: O(log_t(n)) where t = MinDegree, n = number of keys
	- Insert: O(log_t(n)) amortized, O(t * log_t(n)) worst case (due to splits)
	- Space: O(n) for storing keys, O(n/t) for internal structure

- **B+Tree Operations:**
	- Search: O(log_t(n)) - same as B-Tree
	- Insert: O(log_t(n)) amortized, O(t * log_t(n)) worst case
	- Range Query: O(log_t(n) + k) where k = number of results
	- Sequential Scan: O(n) - optimal for scanning all keys
	- Space: O(n) for keys + O(n/t) for internal nodes + linked list overhead