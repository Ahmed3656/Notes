#### **B-Tree (Balanced Tree)**
A self-balancing tree data structure that maintains sorted data and allows for efficient insertion, deletion, and search operations. It is optimized for systems that read and write large blocks of data, like databases and file systems.
- **Node Structure:** each node can contain more than one key and more than two child pointers. The number of keys defines the node's capacity.
- **Properties:**
	- All leaves are at the same depth.
	- A node with `k` keys has `k+1` children.
	- Keys in a node are sorted, and the subtree between two keys `K_i` and `K_{i+1}` contains all keys in the range `(K_i, K_{i+1})`.
- **Data Storage:** both internal nodes and leaf nodes store data (or pointers to data).
- **Use Case:** primarily used when random access is expensive (e.g., disk-based storage) to minimize the number of disk I/O operations.

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