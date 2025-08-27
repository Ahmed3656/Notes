#### **On Disk (HDD/SSD)**
- Data is stored in **blocks/pages** (commonly **4KB** or **8KB**).
- When you need a single record or byte, the **entire page** containing it must be read from disk into memory.
- This is because **disks are optimized for sequential I/O**, not random byte-level access.
- After the page is loaded into RAM (buffer/cache), the database or OS retrieves the exact piece of data you requested.

#### **In RAM**
- RAM supports **random access at the word/byte level**.
- You directly compute the **physical memory address** using the **page table + offset** (virtual memory mapping).
- Only the required word/byte is read, so **no need to fetch surrounding data** unless your code explicitly asks for it.
- That's why RAM is far faster than disk; no large I/O operations are required.

#### **To Summarize**
- On disk → fetches **whole pages**, then extracts what you want.  
- In RAM → **direct random access** to the needed address, **no page-level read needed**.