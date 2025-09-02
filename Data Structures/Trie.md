#### **Trie (Prefix Tree)**
A tree-like data structure optimized for storing and retrieving strings. Its power comes from sharing common prefixes among words, making it exceptionally efficient for prefix-based operations like autocomplete, spell check, and IP routing.
- **Node Structure:** each node represents a single character. The path from the root to a node spells out a prefix. A boolean flag marks the end of a complete word.
- **Properties:**
    - The root node represents the empty string.
    - Each node has an array (or map) of pointers to its child nodes, one for each possible character in the alphabet.
    - The depth of a node corresponds to the length of the prefix it represents.
- **Use Case:** ideal for applications involving prefix searches, such as autocomplete systems, dictionary implementations, and longest prefix matching in network routers.

```text
        (root)
       /   |   \
      c    d    a
     /     |     \
    a      o      p
   / \     |      |
  t*  r    g*     p*
   |   |          |
   *   t*         l
                  |
                  e*
```
- **Search for "cat":** traverse `c -> a -> t`. The `*` on `t` means a word ends here. Found.
- **Search for "car":** traverse `c -> a -> r`. The `r` node **lacks the `*` terminator**, meaning "car" is **not stored** as a complete word. Not found.
- **All words starting with "ca":** find the node `a` under `c`, then recursively collect all words in its subtree: "cat", "cart".

<hr class="hr-light" />

#### **Important Notes**
- The "Trie" is pronounced "try," to distinguish it from "tree."
- The primary advantage of a Trie is its **O(L)** search, insert, and delete time, where **L** is the length of the string, **independent of the total number of words** in the structure.
- The main disadvantage is its **space complexity**, as each node may contain an array of 26 (for letters) or 256 (for ASCII) pointers, many of which may be `nullptr`.
- **Compressed Tries** (like a Radix Tree or Patricia Trie) merge nodes with single children to significantly reduce memory overhead.

---

#### **Implementation**

```cpp
class Trie {
private:
    static constexpr int NUM_CHARS = 256; // Alphabet size: Extended ASCII

    struct Node {
        Node* children[NUM_CHARS];
        bool end;

        Node() : end(false) {
            for (int i = 0; i < NUM_CHARS; ++i) children[i] = nullptr;
        }
    };

    Node* root;

    bool noChildren(Node* node) const {
        for (int i = 0; i < NUM_CHARS; i++) if (node->children[i]) return false;
        return true;
    }

    void clear(Node* node) {
        if (!node) return;
        for (auto child : node->children) // Recursively delete all children
            clear(child);
        delete node; // Delete the current node after its children are deleted
    }

    bool deleteWordHelper(Node* node, const string& txt, int depth) {
        // Base Case: reached the end of the word.
        if (depth == txt.size()) {
            if (!node->end) return false; // Word doesn't exist.
            node->end = false; // Unmark the end-of-word flag.
            // If this node has no children, it's safe to delete it later.
            return noChildren(node);
        }

        char currentChar = txt[depth];
        // Recurse down the path of the next character.
        if (!node->children[currentChar]) return false; // Word doesn't exist.

        bool shouldDeleteChild = deleteWordHelper(node->children[currentChar], txt, depth + 1);

        // After recursing, check if the child node is obsolete.
        if (shouldDeleteChild) {
            delete node->children[currentChar];
            node->children[currentChar] = nullptr;
        }

        return (noChildren(node) && !node->end);
    }

    void countWords(Node* node, int& cnt) {
        if (!node) return;
        if (node->end) cnt++; // Count if this node marks a word.

        for (Node* child : node->children) countWords(child, cnt); // Recurse for all children.
    }

public:
    Trie() : root(new Node()) {} // O(1)

    bool isEmpty() const {
        return root == nullptr || noChildren(root);
    }

    bool insert(const string& txt) { // O(L), where L is the length of the word.
        Node* curr = root;
        for (char c : txt) {
            unsigned char idx = static_cast<unsigned char>(c); // Safe conversion
            if (!curr->children[idx])
                curr->children[idx] = new Node(); // Create a new node if the path doesn't exist.
            curr = curr->children[idx];
        }
        if (curr->end) return false; // Word already exists.
        curr->end = true; // Mark the end of the word.
        return true;
    }

    bool search(const string& txt) { // O(L), where L is the length of the word.
        Node* curr = root;
        for (char c : txt) {
            unsigned char idx = static_cast<unsigned char>(c);
            if (!curr->children[idx]) return false; // Path doesn't exist.
            curr = curr->children[idx];
        }
        return curr->end; // Check if the end-of-word flag is set.
    }

    bool deleteWord(const string& txt) { // O(L), same as search, plus the cost of deleting nodes.
        if (txt.empty()) return false;
        return deleteWordHelper(root, txt, 0);
        // Note: The root is never deleted by the helper function, which is correct.
    }

    void printTrie() { // O(N * NUM_CHARS), where N is the number of nodes.
        if (!root) return;
        
        queue<pair<Node*, string>> q;
        q.push({root, ""});

        while (!q.empty()) {
            pair<Node*, string> p = q.front();
			Node* node = p.first;
			string prefix = p.second;
            q.pop();

            if (node->end) cout << prefix << '\n'; // Print the word if found.

            // Push all children onto the queue, appending their character to the prefix.
            for (int i = 0; i < NUM_CHARS; i++) {
                if (node->children[i])
                    q.push({node->children[i], prefix + static_cast<char>(i)});
            }
        }
    }

    bool startsWith(const string& prefix) { // O(P), where P is the length of the prefix.
        Node* curr = root;
        for (char c : prefix) {
            unsigned char idx = static_cast<unsigned char>(c);
            if (!curr->children[idx]) return false; // Prefix path breaks.
            curr = curr->children[idx];
        }
        return true; // Successfully traversed the entire prefix.
    }

    int countWords() { // O(N), as it must visit every node.
        int cnt = 0;
        countWords(root, cnt);
        return cnt;
    }

    int countWordsWithPrefix(const string& prefix) { // O(L + S), where L is the length of the prefix and S is the size of the subtree.
        Node* curr = root;
        // Traverse to the node representing the prefix.
        for (char c : prefix) {
            unsigned char idx = static_cast<unsigned char>(c);
            if (!curr->children[idx]) return 0; // No words with this prefix.
            curr = curr->children[idx];
        }
        // Count all words in the subtree of the prefix node.
        int cnt = 0;
        countWords(curr, cnt);
        return cnt;
    }

    string longestPrefixMatch(const string& txt) { // O(L * N), where L is the length of the LCP and N is the number of string
        Node* curr = root;
        string lpm;
        for (char c : txt) {
            unsigned char idx = static_cast<unsigned char>(c);
            if (curr->children[idx]) {
                lpm += c;
                curr = curr->children[idx];
            } else {
                break; // Path ends, stop.
            }
        }
        // IMPORTANT: This returns the longest path, not necessarily a word.
        // To ensure it's a word, we would need to track the last valid "end" node.
        return lpm;
    }

    void clear() { // O(N * NUM_CHARS)
        clear(root);
        root = new Node(); // Reset to an empty trie.
    }

    ~Trie() { clear(root); }
};


int main() {
    Trie t;
	t.insert("apple");
	t.insert("app");
	t.insert("banana");
	t.insert("bat");
	t.insert("batman");
	t.insert("flag");
	
	cout << "Words in Trie:\n";
	t.printTrie();
	cout << "Number of words: " << t.countWords() << '\n';
	cout << "Words with prefix 'ba': " << t.countWordsWithPrefix("ba") << '\n';
	cout << "Longest prefix match for 'batmobile': " << t.longestPrefixMatch("batmobile") << '\n';
	
	vector<string> strs = { "flower", "flow", "flight" };
	cout << "Longest common prefix: " << t.longestCommonPrefix(strs) << '\n';
	
	cout << "\nSearch tests:\n";
	cout << "app -> " << (t.search("app") ? "Found" : "Not Found") << '\n';
	cout << "bat -> " << (t.search("bat") ? "Found" : "Not Found") << '\n';
	cout << "batmobile -> " << (t.search("batmobile") ? "Found" : "Not Found") << '\n';
	
	t.deleteWord("app");
	cout<< "app -> " << (t.search("app") ? "Found" : "Not Found") << '\n';
	
	cout << "Before clearing, is Trie empty? " << (t.isEmpty() ? "Yes" : "No") << '\n';
	t.clear();
	cout << "After clearing, is Trie empty? " << (t.isEmpty() ? "Yes" : "No") << '\n';
    return 0;
}
```

---

#### **Complexity Analysis**

- **Insert:** **O(L)**, where L is the length of the word being inserted.
- **Search (Exact):** **O(L)**, traverses one node per character.
- **Search (Prefix):** **O(P)**, where P is the length of the prefix.
- **Delete:** **O(L)**, similar to search, plus constant-time node deletion checks.
- **Longest Prefix Match:** **O(L)**, where L is the length of the input string. It performs a single traversal of the string.
- **Longest Common Prefix:** **O(L * N)**, where L is the length of the LCP and N is the number of strings. In the worst case, O(M * N), where M is the length of the shortest string.
- **Count Words with Prefix:** **O(L + S)**, O(L) to find the prefix node, plus O(S) to count all words in its subtree.
- **Memory Space:** **O(N * C)**, where N is the number of nodes and C is the alphabet size (NUM_CHARS).