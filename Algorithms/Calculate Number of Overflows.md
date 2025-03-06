
### Multiplication Overflow Calculation

### Concept
To determine how many times a number overflows when multiplying two or more numbers, we:  
- Calculate the total number of bits each number requires.  
- Divide this by the bit width of the datatype we are using.  
- The remainder represents the result within the bit width, while the quotient gives the number of overflows.  

## **Mathematical Formula**  
The number of overflows can be computed as:  

$$
\text{overflows} = \left\lfloor \frac{\log_2 (a) + \log_2 (b)}{\text{bit width of type}} \right\rfloor
$$

## **Explanation**  
1. **Logarithmic Calculation**  
   - The number of bits required to store a number `x` is given by `log_2(x)`.  
   - For a multiplication `a * b`, the total required bits is `log_2(a) + log_2(b)`.  

2. **Determining Overflows**  
   - We divide the total bits by the datatype's bit width.  
   - The integer quotient gives the number of times an overflow occurs.  
   - The remainder determines the actual result within the bit width.  

## **Example Calculation**  
For `a = 300` and `b = 500` using a `16-bit` integer type:  

1. Compute the bit requirement:  
   $$
   \log_2(300) \approx 8.23, \quad \log_2(500) \approx 8.96
   $$
   $$
   \log_2(300) + \log_2(500) \approx 17.19
   $$

2. Divide by the bit width:  
   $$
   \frac{17.19}{16} \approx 1.07
   $$  

3. Result:  
   - **Overflows**: `1`  
   - **Remaining Bits in Range**  

## **Code Implementation**  
```cpp
#include <cmath>
#include <iostream>

int countOverflows(int a, int b, int bitWidth) {
    double totalBits = log2(a) + log2(b);
    return static_cast<int>(totalBits / bitWidth);
}

int main() {
    int a = 300, b = 500, bitWidth = 16;
    std::cout << "Overflows: " << countOverflows(a, b, bitWidth) << std::endl;
    return 0;
}
