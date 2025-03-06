
```cpp
		struct CusComp {
		    bool operator()(const pme & a, const pme & b) {
		        if (a.first != b.first) {
		            return a.first > b.first;
		        }
		        return a.second.first < b.second.first;
		    }
		};
		//in main
		priority_queue<pme, vector<pme>, CusComp> pq;
```
