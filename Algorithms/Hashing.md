A fundamental process that converts input data (of any size) into a fixed-size value, typically a numerical representation. The primary goal is to enable efficient data retrieval, comparison, and integrity checks by creating a unique "fingerprint" for the input.

- **Core Problem Solved:** provides a fast and consistent way to map arbitrary data to a fixed-size value, enabling O(1) lookups in ideal scenarios and facilitating data integrity verification.
- **Basic Mechanism:** a **hash function** takes an input (or "key") and performs a series of computations to produce a hash value (often an integer). A good function distributes outputs uniformly across the possible range.
- **Properties of a Good Hash Function:**
    - **Deterministic:** the same input always produces the same hash value.
    - **Uniform Distribution:** outputs are spread evenly to minimize collisions.
    - **Efficiency:** computes the hash value quickly.
    - **Avalanche Effect:** a small change in the input results in a large, unpredictable change in the output.

```text
Example: Simple Hash Function on Strings

Input: "apple"
Hash Calculation: (a=1, p=16, l=12, e=5) -> 1+16+16+12+5 = 50
Final Hash (mod 10 for a small table): 50 % 10 = 0

Input: "banana"
Hash Calculation: (b=2, a=1, n=14) -> 2+1+14+1+14+1 = 33
Final Hash: 33 % 10 = 3

These hashes (0 and 3) become indices in an array (the hash table).
```

---

#### **Mathematical Foundation & Collisions**
The finite size of the hash value range inevitably leads to collisions, where two different inputs produce the same hash.
- **Pigeonhole Principle:** if you have more possible inputs than possible hash values, collisions are guaranteed. The goal is to minimize their impact.
- **Collision Probability:** for a hash function with `m` possible outputs, the probability of at least one collision after `n` hashes is approximately `1 - e^(-n²/2m)` (Birthday Paradox).
- **Handling Collisions:** this is not the responsibility of the hash function itself, but of the data structure using it (e.g., a hash table uses chaining or open addressing).

<hr class="hr-light" />

#### **Hash Function Implementation Considerations**
- **Choice of Function:** depends on the use case (cryptography vs. data structures).
    - **Non-Cryptographic:** prioritize speed and good distribution (e.g., MurmurHash, xxHash). Often simulated with a single good function and different seeds: `hash_i(value) = base_hash(value, seed_i)`.
    - **Cryptographic:** prioritize irreversibility and avalanche effect (e.g., SHA-256). Much slower.
- **Modulo Operation:** used to map the large hash value to a specific index within an array of size `m`. The quality of this operation depends on the uniformity of the original hash.

<hr class="hr-light" />

#### **Code Implementation**

##### Polynomial Hash (Base 31)
```cpp
class SimpleHash31 {
private:
    using ll = long long;
    const ll mod = 1000000007LL;
    const ll base = 31LL;
	
    ll hash_val = 0;
    int len = 0;
	
    vector<ll> pow_base;     // base^i % mod
    vector<ll> inv_pow_base; // base^{-i} % mod
	
    ll mod_pow(ll a, ll b) {
        ll res = 1;
        a %= mod;
        while (b > 0) {
            if (b & 1) res = (res * a) % mod;
            a = (a * a) % mod;
            b >>= 1;
        }
        return res;
    }
	
public:
    SimpleHash31(int max_len) {
        pow_base.resize(max_len + 1);
        inv_pow_base.resize(max_len + 1);
		
        pow_base[0] = 1;
        for (int i = 1; i <= max_len; i++)
            pow_base[i] = (pow_base[i-1] * base) % mod;
		
        ll inv_base = mod_pow(base, mod - 2);
        inv_pow_base[0] = 1;
        for (int i = 1; i <= max_len; i++)
            inv_pow_base[i] = (inv_pow_base[i-1] * inv_base) % mod;
    }
	
    // append to back
    void push_back(char c) {
        ll v = (c - 'a' + 1);
        hash_val = (hash_val + v * pow_base[len]) % mod;
        ++len;
    }
	
    // prepend to front
    void push_front(char c) {
        ll v = (c - 'a' + 1);
        hash_val = ( (hash_val * base) % mod + v ) % mod;
        ++len;
    }
	
    // remove from back (need to pass removed char)
    void pop_back(char c) {
        ll v = (c - 'a' + 1);
        hash_val = (hash_val - v * pow_base[len-1] % mod + mod) % mod;
        --len;
    }
	
    // remove from front (need to pass removed char)
    void pop_front(char c) {
        ll v = (c - 'a' + 1);
        hash_val = (hash_val - v + mod) % mod;
        hash_val = (hash_val * inv_pow_base[1]) % mod;
        --len;
    }
	
    ll get_hash() const { return hash_val; }
    int get_len() const { return len; }
};

int main() {
    SimpleHash31 h(100); // allow up to 100 chars
    
    string s = "hello";
    for (char c : s) h.push_back(c);
    
    cout << "String: " << s << "\n";
    cout << "Hash: " << h.get_hash() << "\n";
    
    h.pop_back('o');
    cout << "After pop_back('o'): " << h.get_hash() << "\n";
    
    h.push_front('a');
    cout << "After push_front('a'): " << h.get_hash() << "\n";
}
```

##### Complexity
- **Precomputation**: `O(max_len)`
- **push_back / push_front / pop_back / pop_front**: `O(1)`
- **Space**: `O(max_len)` for powers

<hr class="hr-light" />

##### Minimal MurmurHash3 (32-bit)
```cpp
uint32_t murmurhash3(const void* key, int len, uint32_t seed = 0) {
    const uint8_t* data = (const uint8_t*)key;
    const int nblocks = len / 4;
	
    uint32_t h1 = seed;
    const uint32_t c1 = 0xcc9e2d51u;
    const uint32_t c2 = 0x1b873593u;
	
    // body - process 4-byte blocks (little-endian)
    for (int i = 0; i < nblocks; ++i) {
        uint32_t k1 = (uint32_t)data[i*4] | ((uint32_t)data[i*4+1] << 8) |
                      ((uint32_t)data[i*4+2] << 16) | ((uint32_t)data[i*4+3] << 24);
	
        k1 *= c1;
        k1 = (k1 << 15) | (k1 >> 17);
        k1 *= c2;
	
        h1 ^= k1;
        h1 = (h1 << 13) | (h1 >> 19);
        h1 = h1 * 5 + 0xe6546b64u;
    }
	
    // tail
    const uint8_t* tail = data + nblocks * 4;
    uint32_t k1 = 0;
    switch (len & 3) {
        case 3: k1 ^= (uint32_t)tail[2] << 16;
        case 2: k1 ^= (uint32_t)tail[1] << 8;
        case 1:
            k1 ^= (uint32_t)tail[0];
            k1 *= c1;
            k1 = (k1 << 15) | (k1 >> 17);
            k1 *= c2;
            h1 ^= k1;
    }
	
    // finalization
    h1 ^= (uint32_t)len;
    h1 ^= h1 >> 16;
    h1 *= 0x85ebca6bu;
    h1 ^= h1 >> 13;
    h1 *= 0xc2b2ae35u;
    h1 ^= h1 >> 16;
	
    return h1;
}

int main() {
    string s = "hello world";
    uint32_t h = murmurhash3(s.data(), s.size(), 42);
    
    cout << "String: " << s << "\n";
    cout << "MurmurHash3: " << h << "\n";
}
```

**Avalanche Effect Note:** the multiple bit-shifting and mixing operations (lines with `k1 = (k1 << 15) | (k1 >> 17)`, etc.) are specifically designed to create the avalanche effect where small input changes cause large, unpredictable output changes.
##### Complexity
- **Hash computation**: `O(len)` (processes each 4-byte block, then tail)
- **Space**: `O(1)` (constant workspace)

<hr class="hr-light" />

##### Minimal xxHash (32-bit)
```cpp
static inline uint32_t read32le(const uint8_t* p) {
    return (uint32_t)p[0] | ((uint32_t)p[1] << 8) | ((uint32_t)p[2] << 16) | ((uint32_t)p[3] << 24);
}

uint32_t xxhash32(const void* input, int len, uint32_t seed = 0) {
    const uint8_t* p = (const uint8_t*)input;
    const uint8_t* end = p + len;
    uint32_t h32;
	
    if (len >= 16) {
        const uint8_t* limit = end - 16;
        uint32_t v1 = seed + 0x9e3779b1u + 0x85ebca77u;
        uint32_t v2 = seed + 0x85ebca77u;
        uint32_t v3 = seed + 0x0u;
        uint32_t v4 = seed - 0x9e3779b1u;
		
        do {
            v1 += read32le(p) * 0x9e3779b1u; v1 = (v1 << 13) | (v1 >> 19); v1 *= 0x85ebca77u; p += 4;
            v2 += read32le(p) * 0x9e3779b1u; v2 = (v2 << 13) | (v2 >> 19); v2 *= 0x85ebca77u; p += 4;
            v3 += read32le(p) * 0x9e3779b1u; v3 = (v3 << 13) | (v3 >> 19); v3 *= 0x85ebca77u; p += 4;
            v4 += read32le(p) * 0x9e3779b1u; v4 = (v4 << 13) | (v4 >> 19); v4 *= 0x85ebca77u; p += 4;
        } while (p <= limit);
		
        h32 = ((v1 << 1) | (v1 >> 31));
        h32 += ((v2 << 7) | (v2 >> 25));
        h32 += ((v3 << 12) | (v3 >> 20));
        h32 += ((v4 << 18) | (v4 >> 14));
    } else {
        h32 = seed + 0x165667b1u;
    }
	
    h32 += (uint32_t)len;
	
    // remaining 4-byte lanes
    while (p + 4 <= end) {
        h32 += read32le(p) * 0x9e3779b1u;
        h32 = (h32 << 13) | (h32 >> 19);
        h32 = h32 * 0x85ebca77u + 0xc2b2ae3du;
        p += 4;
    }
	
    // remaining bytes
    while (p < end) {
        h32 += (uint32_t)(*p) * 0x27d4eb2fu;
        h32 = (h32 << 11) | (h32 >> 21);
        h32 *= 0x9e3779b1u;
        ++p;
    }
	
    h32 ^= h32 >> 15;
    h32 *= 0x85ebca6bu;
    h32 ^= h32 >> 13;
    h32 *= 0xc2b2ae35u;
    h32 ^= h32 >> 16;
	
    return h32;
}

int main() {
    string s = "fast hashing demo";
    uint32_t h = xxhash32(s.data(), s.size(), 123);
    
    cout << "String: " << s << "\n";
    cout << "xxHash32: " << h << "\n";
}
```

##### Complexity
- **Hash computation**: `O(len)` (processes input in 16-byte chunks, then tail)
- **Space**: `O(1)`

<hr class="hr-light" />

##### Minimal SHA-256
```cpp
// SHA-256 constants (first 32 bits of the fractional parts of the cube roots of the first 64 primes)
static const uint32_t K[64] = {
  0x428a2f98u, 0x71374491u, 0xb5c0fbcfu, 0xe9b5dba5u, 0x3956c25bu, 0x59f111f1u, 0x923f82a4u, 0xab1c5ed5u,
  0xd807aa98u, 0x12835b01u, 0x243185beu, 0x550c7dc3u, 0x72be5d74u, 0x80deb1feu, 0x9bdc06a7u, 0xc19bf174u,
  0xe49b69c1u, 0xefbe4786u, 0x0fc19dc6u, 0x240ca1ccu, 0x2de92c6fu, 0x4a7484aau, 0x5cb0a9dcu, 0x76f988dau,
  0x983e5152u, 0xa831c66du, 0xb00327c8u, 0xbf597fc7u, 0xc6e00bf3u, 0xd5a79147u, 0x06ca6351u, 0x14292967u,
  0x27b70a85u, 0x2e1b2138u, 0x4d2c6dfcu, 0x53380d13u, 0x650a7354u, 0x766a0abbu, 0x81c2c92eu, 0x92722c85u,
  0xa2bfe8a1u, 0xa81a664bu, 0xc24b8b70u, 0xc76c51a3u, 0xd192e819u, 0xd6990624u, 0xf40e3585u, 0x106aa070u,
  0x19a4c116u, 0x1e376c08u, 0x2748774cu, 0x34b0bcb5u, 0x391c0cb3u, 0x4ed8aa4au, 0x5b9cca4fu, 0x682e6ff3u,
  0x748f82eeu, 0x78a5636fu, 0x84c87814u, 0x8cc70208u, 0x90befffau, 0xa4506cebu, 0xbef9a3f7u, 0xc67178f2u
};

static inline uint32_t rightRotate(uint32_t value, int bits) {
    return (value >> bits) | (value << (32 - bits));
}

void sha256(const void* input, int len, uint8_t output[32]) {
    // initial hash values
    uint32_t h[8] = {
        0x6a09e667u, 0xbb67ae85u, 0x3c6ef372u, 0xa54ff53au,
        0x510e527fu, 0x9b05688cu, 0x1f83d9abu, 0x5be0cd19u
    };
	
    // pre-processing (padding)
    uint64_t bitLen = (uint64_t)len * 8ULL;
    int newLen = (((len + 8) / 64) + 1) * 64;
    vector<uint8_t> msg(newLen, 0);
    memcpy(msg.data(), input, len);
    msg[len] = 0x80;
    // append 64-bit big-endian length
    for (int i = 0; i < 8; ++i) {
        msg[newLen - 8 + i] = (uint8_t)((bitLen >> (56 - i * 8)) & 0xFF);
    }
	
    // process each 512-bit chunk
    for (int chunk = 0; chunk < newLen; chunk += 64) {
        uint32_t w[64];
        // prepare message schedule (big-endian word assembly)
        for (int i = 0; i < 16; ++i) {
            int idx = chunk + i * 4;
            w[i] = ((uint32_t)msg[idx] << 24) | ((uint32_t)msg[idx + 1] << 16) |
                   ((uint32_t)msg[idx + 2] << 8) | ((uint32_t)msg[idx + 3]);
        }
        for (int i = 16; i < 64; ++i) {
            uint32_t s0 = rightRotate(w[i - 15], 7) ^ rightRotate(w[i - 15], 18) ^ (w[i - 15] >> 3);
            uint32_t s1 = rightRotate(w[i - 2], 17) ^ rightRotate(w[i - 2], 19) ^ (w[i - 2] >> 10);
            w[i] = w[i - 16] + s0 + w[i - 7] + s1;
        }
		
        uint32_t a = h[0], b = h[1], c = h[2], d = h[3];
        uint32_t e = h[4], f = h[5], g = h[6], hv = h[7];
		
        // compression loop
        for (int i = 0; i < 64; ++i) {
            uint32_t S1 = rightRotate(e, 6) ^ rightRotate(e, 11) ^ rightRotate(e, 25);
            uint32_t ch = (e & f) ^ ((~e) & g);
            uint32_t temp1 = hv + S1 + ch + K[i] + w[i];
            uint32_t S0 = rightRotate(a, 2) ^ rightRotate(a, 13) ^ rightRotate(a, 22);
            uint32_t maj = (a & b) ^ (a & c) ^ (b & c);
            uint32_t temp2 = S0 + maj;
			
            hv = g; g = f; f = e; e = d + temp1;
            d = c; c = b; b = a; a = temp1 + temp2;
        }
		
        h[0] += a; h[1] += b; h[2] += c; h[3] += d;
        h[4] += e; h[5] += f; h[6] += g; h[7] += hv;
    }
	
    // produce final hash (big-endian)
    for (int i = 0; i < 8; ++i) {
        output[i*4 + 0] = (uint8_t)((h[i] >> 24) & 0xFF);
        output[i*4 + 1] = (uint8_t)((h[i] >> 16) & 0xFF);
        output[i*4 + 2] = (uint8_t)((h[i] >> 8) & 0xFF);
        output[i*4 + 3] = (uint8_t)(h[i] & 0xFF);
    }
}

int main() {
    string s = "cryptographic hash";
    uint8_t digest[32];
    
    sha256(s.data(), s.size(), digest);
    
    cout << "String: " << s << "\n";
    cout << "SHA-256: ";
    for (int i = 0; i < 32; i++) printf("%02x", digest[i]);
    cout << "\n";
}
```

##### Complexity
- **Hash computation**: `O(len)` (processes message in 512-bit = 64-byte blocks, plus padding)
- **Space**: `O(1)` auxiliary arrays (`w[64]` + constants)

---

#### **Use Cases & Applications**
- **Data Structures:** the foundation of hash tables, sets, and caches.
- **Cryptography:** verifying data integrity (checksums, digital signatures).
- **Databases:** as discussed in the transcript, hashing is critical for **database joins**. The smaller relation is scanned, and a hash table is built in memory using the join key. Probing the second relation then becomes an O(1) operation per key, making the join efficient.
- **Distributed Systems:** sharding and partitioning data across servers. The key is hashed to determine which server or partition is responsible for it.

<hr class="hr-light" />

#### **Limitations & Considerations**
- **Collisions:** inevitable and must be handled by the surrounding system.
- **Fixed Output Range:** the hash value does not uniquely identify the original input.
- **Cost of Hashing:** while fast, hashing is an additional cost compared to a direct array index lookup.

<hr class="hr-light" />

#### **Key Takeaways**
- **Hash Functions:** deterministic functions that map arbitrary data to fixed-size values for efficient operations.
- **Collisions:** inevitable due to pigeonhole principle; handled by data structures (chaining, open addressing).
- **Trade-offs:** non-cryptographic (speed, uniformity) vs. Cryptographic (irreversibility, avalanche effect).
- **Applications:** hash tables, databases (joins), caching, integrity checks, distributed systems.