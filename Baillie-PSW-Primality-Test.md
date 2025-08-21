# Baillie-PSW Primality Test

## Overview

The Baillie-PSW primality test, developed by Robert Baillie, Carl Pomerance, John Selfridge, and Samuel Wagstaff Jr. in 1980, is a probabilistic primality test that combines a Miller-Rabin test with a Lucas probability test. It is notable for having **no known composite counterexamples** despite being a probabilistic test, making it extremely reliable in practice. The test is widely used in computational number theory and some cryptographic applications where a very high degree of confidence in primality is required.

## Mathematical Foundation

### Theoretical Background

The Baillie-PSW test combines two independent primality tests:
1. **Miller-Rabin test** with base 2
2. **Lucas probable prime test** with carefully chosen parameters

The strength comes from the fact that numbers that fool both tests simultaneously appear to be extremely rare (possibly non-existent).

### Miller-Rabin Component

The first component is a **strong probable prime test** (Miller-Rabin) with base 2:

For odd $n > 1$, write $n - 1 = 2^r \cdot d$ where $d$ is odd. Then $n$ is a **strong probable prime base 2** if either:
- $2^d \equiv 1 \pmod{n}$, or
- $2^{2^i d} \equiv -1 \pmod{n}$ for some $0 \leq i < r$

### Lucas Probable Prime Test

The Lucas test is based on **Lucas sequences**. Given integers P and Q, define the Lucas sequences:
```
Uₙ(P,Q) = (αⁿ - βⁿ) / (α - β)
Vₙ(P,Q) = αⁿ + βⁿ
```

where $\alpha$ and $\beta$ are roots of $x^2 - Px + Q = 0$.

The **discriminant** is $D = P^2 - 4Q$.

**Lucas Probable Prime Test**: An odd integer $n$ with $\gcd(n, Q) = 1$ is a **Lucas probable prime** with parameters $(P, Q)$ if:
```
U_{n - (D/n)}(P,Q) ≡ 0 (mod n)
```

where $\left(\frac{D}{n}\right)$ is the Jacobi symbol.

### Strong Lucas Probable Prime Test

For the Baillie-PSW test, we use the **strong Lucas probable prime test**:

Write $n + 1 = 2^s \cdot t$ where $t$ is odd. Then $n$ is a **strong Lucas probable prime** if either:
- $U_t(P,Q) \equiv 0 \pmod{n}$, or  
- $V_{2^j t}(P,Q) \equiv 0 \pmod{n}$ for some $0 \leq j < s$

### Parameter Selection

The Baillie-PSW test uses specific parameter selection:
1. Use the smallest $D$ such that $\left(\frac{D}{n}\right) = -1$ where $D = P^2 - 4Q$
2. Typically start with $P = 1, Q = \frac{1-D}{4}$ and $D = 5, 9, 13, 17, ...$
3. The first $D$ in the sequence $5, -7, 9, -11, 13, -15, ...$ such that $\left(\frac{D}{n}\right) = -1$

## Algorithm Description

**Baillie-PSW Test Algorithm**:

**Input**: Odd integer $n > 1$

**Steps**:
1. **Trial division**: Check if $n$ has small prime factors (optional optimization)
2. **Perfect power check**: Verify $n$ is not a perfect power (optional)
3. **Miller-Rabin base 2**: 
   - If $n$ fails, return COMPOSITE
   - If $n$ passes, continue
4. **Parameter selection**:
   - Find smallest $D > 0$ such that $\left(\frac{D}{n}\right) = -1$
   - Set $P = 1, Q = \frac{1-D}{4}$
5. **Strong Lucas test**:
   - If $n$ fails, return COMPOSITE  
   - If $n$ passes, return PROBABLY PRIME

**Result**: If both tests pass, $n$ is almost certainly prime.

## Remarkable Properties

### No Known Counterexamples
Despite being a probabilistic test, **no composite number is known to pass the Baillie-PSW test**. This is remarkable because:
- Miller-Rabin alone has infinitely many counterexamples for any fixed base
- Lucas tests alone have counterexamples
- The combination appears to eliminate all known counterexamples

### Theoretical Bounds
- It's proven that if counterexamples exist, the smallest one is greater than $10^{15}$
- Heuristic arguments suggest counterexamples are extremely rare
- Some variants have been proven to have no counterexamples below much larger bounds

## Python Implementation

```python
import math
import random
from typing import Tuple, Optional

def jacobi_symbol(a: int, n: int) -> int:
    """
    Compute Jacobi symbol (a/n)
    Returns -1, 0, or 1
    """
    if n <= 0 or n % 2 == 0:
        raise ValueError("n must be positive and odd")
    
    a = a % n
    result = 1
    
    while a != 0:
        while a % 2 == 0:
            a //= 2
            if n % 8 in [3, 5]:
                result = -result
        
        a, n = n, a
        if a % 4 == 3 and n % 4 == 3:
            result = -result
        
        a = a % n
    
    return result if n == 1 else 0

def gcd(a: int, b: int) -> int:
    """Greatest Common Divisor using Euclidean algorithm"""
    while b:
        a, b = b, a % b
    return a

def miller_rabin_base_2(n: int) -> bool:
    """
    Miller-Rabin primality test with base 2
    Returns True if n is probably prime, False if definitely composite
    """
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False
    
    # Write n-1 as 2^r * d
    r = 0
    d = n - 1
    while d % 2 == 0:
        r += 1
        d //= 2
    
    # Test with base 2
    x = pow(2, d, n)
    
    if x == 1 or x == n - 1:
        return True
    
    for _ in range(r - 1):
        x = pow(x, 2, n)
        if x == n - 1:
            return True
    
    return False

def lucas_sequence(n: int, P: int, Q: int, k: int) -> Tuple[int, int]:
    """
    Compute Lucas sequences U_k and V_k modulo n
    Returns (U_k mod n, V_k mod n)
    """
    if k == 0:
        return (0, 2)
    if k == 1:
        return (1, P)
    
    # Binary method for computing Lucas sequences
    U_prev, U_curr = 0, 1
    V_prev, V_curr = 2, P
    
    # Process bits of k from most significant to least significant
    for bit in bin(k)[3:]:  # Skip '0b' and first bit
        # Double step: compute U_{2m}, V_{2m}
        U_temp = (U_curr * V_curr) % n
        V_temp = (V_curr * V_curr - 2 * pow(Q, len(bin(k)[3:]), n)) % n
        
        if bit == '1':
            # Add step: compute U_{2m+1}, V_{2m+1}
            U_curr = ((P * U_temp + V_temp) * pow(2, n-2, n)) % n
            V_curr = ((P * V_temp + U_temp * (P*P - 4*Q)) * pow(2, n-2, n)) % n
        else:
            U_curr, V_curr = U_temp, V_temp
    
    return (U_curr, V_curr)

def lucas_sequence_efficient(n: int, P: int, Q: int, k: int) -> Tuple[int, int]:
    """
    More efficient Lucas sequence computation using doubling formulas
    Returns (U_k mod n, V_k mod n)
    """
    if k == 0:
        return (0, 2)
    if k == 1:
        return (1, P)
    
    # Use doubling formulas
    def double_step(U_m: int, V_m: int, Q_m: int) -> Tuple[int, int, int]:
        """Double step: compute U_{2m}, V_{2m}, Q^{2m}"""
        U_2m = (U_m * V_m) % n
        V_2m = (V_m * V_m - 2 * Q_m) % n
        Q_2m = (Q_m * Q_m) % n
        return U_2m, V_2m, Q_2m
    
    def add_step(U_m: int, V_m: int, Q_m: int) -> Tuple[int, int, int]:
        """Add step: compute U_{m+1}, V_{m+1}, Q^{m+1}"""
        U_m1 = ((P * U_m + V_m) * pow(2, n-2, n)) % n
        V_m1 = ((P * V_m + (P*P - 4*Q) * U_m) * pow(2, n-2, n)) % n
        Q_m1 = (Q_m * Q) % n
        return U_m1, V_m1, Q_m1
    
    # Binary ladder method
    U, V, Q_power = 1, P, Q
    
    for bit in bin(k)[3:]:  # Skip first bit (always 1)
        U, V, Q_power = double_step(U, V, Q_power)
        if bit == '1':
            U, V, Q_power = add_step(U, V, Q_power)
    
    return (U, V)

def strong_lucas_test(n: int, P: int, Q: int) -> bool:
    """
    Strong Lucas probable prime test
    Returns True if n passes the test
    """
    D = P * P - 4 * Q
    
    if gcd(n, Q) != 1:
        return False
    
    # Write n+1 = 2^s * t where t is odd
    s = 0
    t = n + 1
    while t % 2 == 0:
        s += 1
        t //= 2
    
    # Compute U_t, V_t
    U, V = lucas_sequence_efficient(n, P, Q, t)
    
    # Check if U_t ≡ 0 (mod n)
    if U == 0:
        return True
    
    # Check if V_{2^j * t} ≡ 0 (mod n) for some 0 ≤ j < s
    for j in range(s):
        if V == 0:
            return True
        # V_{2k} = V_k^2 - 2*Q^k
        V = (V * V - 2 * pow(Q, pow(2, j) * t, n)) % n
    
    return False

def find_lucas_parameters(n: int) -> Tuple[int, int, int]:
    """
    Find suitable parameters P, Q, D for Lucas test
    Returns (P, Q, D) where D is the discriminant
    """
    # Try D = 5, -7, 9, -11, 13, -15, ...
    for i in range(1, 100):  # Practical limit
        if i % 2 == 1:
            D = 4 * i + 1  # 5, 9, 13, 17, ...
        else:
            D = -(4 * i - 3)  # -7, -11, -15, -19, ...
        
        jacobi = jacobi_symbol(D, n)
        
        if jacobi == -1:
            P = 1
            Q = (1 - D) // 4
            return P, Q, D
    
    # Fallback (should rarely happen)
    return 1, -1, 5

def is_perfect_power(n: int) -> bool:
    """
    Check if n is a perfect power (n = a^k for k > 1)
    Returns True if n is a perfect power
    """
    if n < 2:
        return False
    
    # Check for small powers
    for k in range(2, int(math.log2(n)) + 1):
        # Binary search for kth root
        low, high = 1, int(n**(1/k)) + 1
        
        while low <= high:
            mid = (low + high) // 2
            power = mid ** k
            
            if power == n:
                return True
            elif power < n:
                low = mid + 1
            else:
                high = mid - 1
    
    return False

def small_prime_divisors(n: int, limit: int = 1000) -> bool:
    """
    Check for small prime divisors up to limit
    Returns True if n has a small prime divisor
    """
    small_primes = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]
    
    for p in small_primes:
        if p > limit:
            break
        if p > n:
            break
        if n % p == 0:
            return n == p  # True only if n itself is the small prime
    
    return False

def baillie_psw_test(n: int) -> bool:
    """
    Baillie-PSW primality test
    Returns True if n is probably prime, False if definitely composite
    """
    # Handle small cases
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False
    
    # Check for small prime divisors (optional optimization)
    if small_prime_divisors(n, 100):
        return n <= 100 and n in [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97]
    
    # Check for perfect powers (optional)
    if is_perfect_power(n):
        return False
    
    # Miller-Rabin test with base 2
    if not miller_rabin_base_2(n):
        return False
    
    # Find Lucas parameters
    P, Q, D = find_lucas_parameters(n)
    
    # Strong Lucas probable prime test
    return strong_lucas_test(n, P, Q)

def baillie_psw_detailed(n: int) -> dict:
    """
    Detailed Baillie-PSW test with step-by-step results
    Returns dictionary with detailed information
    """
    result = {
        "n": n,
        "steps": [],
        "final_result": False
    }
    
    # Step 1: Basic checks
    if n < 2:
        result["steps"].append("FAIL: n < 2")
        return result
    if n == 2:
        result["steps"].append("PASS: n = 2 (prime)")
        result["final_result"] = True
        return result
    if n % 2 == 0:
        result["steps"].append("FAIL: n is even")
        return result
    
    result["steps"].append("PASS: Basic checks")
    
    # Step 2: Small prime divisors
    if small_prime_divisors(n, 100):
        small_primes = [p for p in [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97] if n % p == 0]
        if n in [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97]:
            result["steps"].append(f"PASS: n = {n} is a small prime")
            result["final_result"] = True
        else:
            result["steps"].append(f"FAIL: n has small prime factors: {small_primes}")
        return result
    
    result["steps"].append("PASS: No small prime divisors")
    
    # Step 3: Perfect power check
    if is_perfect_power(n):
        result["steps"].append("FAIL: n is a perfect power")
        return result
    
    result["steps"].append("PASS: Not a perfect power")
    
    # Step 4: Miller-Rabin base 2
    mr_result = miller_rabin_base_2(n)
    if not mr_result:
        result["steps"].append("FAIL: Miller-Rabin base 2 test")
        return result
    
    result["steps"].append("PASS: Miller-Rabin base 2 test")
    
    # Step 5: Lucas parameter selection
    P, Q, D = find_lucas_parameters(n)
    result["lucas_params"] = {"P": P, "Q": Q, "D": D}
    result["steps"].append(f"Lucas parameters: P={P}, Q={Q}, D={D}")
    
    # Step 6: Strong Lucas test
    lucas_result = strong_lucas_test(n, P, Q)
    if not lucas_result:
        result["steps"].append("FAIL: Strong Lucas test")
        return result
    
    result["steps"].append("PASS: Strong Lucas test")
    result["final_result"] = True
    
    return result

def compare_primality_tests(test_numbers: list) -> None:
    """
    Compare Baillie-PSW with other primality tests
    """
    import time
    
    def miller_rabin_multiple_bases(n: int, k: int = 10) -> bool:
        """Miller-Rabin with multiple random bases"""
        if n < 2 or (n > 2 and n % 2 == 0):
            return n == 2
        
        # Write n-1 as 2^r * d
        r = 0
        d = n - 1
        while d % 2 == 0:
            r += 1
            d //= 2
        
        # Test k random bases
        for _ in range(k):
            a = random.randrange(2, n - 1)
            x = pow(a, d, n)
            
            if x == 1 or x == n - 1:
                continue
                
            for _ in range(r - 1):
                x = pow(x, 2, n)
                if x == n - 1:
                    break
            else:
                return False
        
        return True
    
    def fermat_test(n: int, k: int = 10) -> bool:
        """Fermat primality test with k bases"""
        if n < 2 or (n > 2 and n % 2 == 0):
            return n == 2
        
        for _ in range(k):
            a = random.randrange(2, n)
            if pow(a, n - 1, n) != 1:
                return False
        return True
    
    tests = [
        ("Fermat", fermat_test),
        ("Miller-Rabin", miller_rabin_multiple_bases),
        ("Baillie-PSW", baillie_psw_test)
    ]
    
    print(f"{'Number':<12} {'Fermat':<8} {'M-R':<8} {'B-PSW':<8} {'Fermat_T':<10} {'M-R_T':<10} {'B-PSW_T':<10}")
    print("-" * 75)
    
    for n in test_numbers:
        results = {}
        times = {}
        
        for name, test_func in tests:
            start_time = time.time()
            try:
                results[name] = test_func(n)
                times[name] = time.time() - start_time
            except:
                results[name] = False
                times[name] = 0
        
        fermat_str = "PRIME" if results["Fermat"] else "COMP"
        mr_str = "PRIME" if results["Miller-Rabin"] else "COMP"
        bpsw_str = "PRIME" if results["Baillie-PSW"] else "COMP"
        
        print(f"{n:<12} {fermat_str:<8} {mr_str:<8} {bpsw_str:<8} {times['Fermat']:<10.6f} {times['Miller-Rabin']:<10.6f} {times['Baillie-PSW']:<10.6f}")

# Example usage and demonstrations
if __name__ == "__main__":
    print("=== Baillie-PSW Primality Test ===\n")
    
    # Test known primes and composites
    test_cases = [
        (2, True, "Smallest prime"),
        (17, True, "Small prime"),
        (25, False, "Perfect square"),
        (91, False, "91 = 7 × 13"),
        (561, False, "Carmichael number"),
        (1009, True, "Medium prime"),
        (1729, False, "Ramanujan's taxi number"),
        (65537, True, "Fermat prime F_4"),
        (982451653, True, "Large prime")
    ]
    
    print("Testing known cases:")
    correct = 0
    total = len(test_cases)
    
    for n, expected, description in test_cases:
        result = baillie_psw_test(n)
        status = "✓" if result == expected else "✗"
        print(f"{status} {n:>10} ({description}): {result}")
        if result == expected:
            correct += 1
    
    print(f"\nAccuracy: {correct}/{total} ({100*correct/total:.1f}%)\n")
    
    # Detailed analysis of a specific case
    print("=== Detailed Analysis: Carmichael Number 561 ===")
    detailed = baillie_psw_detailed(561)
    
    print(f"Testing n = {detailed['n']}")
    for step in detailed['steps']:
        print(f"  {step}")
    print(f"Final result: {'PROBABLY PRIME' if detailed['final_result'] else 'COMPOSITE'}")
    
    if 'lucas_params' in detailed:
        params = detailed['lucas_params']
        print(f"Lucas parameters used: P={params['P']}, Q={params['Q']}, D={params['D']}")
    print()
    
    # Test some challenging numbers
    print("=== Testing Challenging Numbers ===")
    challenging = [
        (341, "Fermat pseudoprime base 2"),
        (1105, "Carmichael number"),
        (2465, "Carmichael number"),
        (2821, "Carmichael number"),
        (15841, "Carmichael number"),
        (29341, "Carmichael number"),
        (46657, "Strong pseudoprime base 2"),
        (252601, "Strong pseudoprime base 2")
    ]
    
    for n, description in challenging:
        result = baillie_psw_test(n)
        status = "PROBABLY PRIME" if result else "COMPOSITE"
        print(f"{n:>8} ({description}): {status}")
    print()
    
    # Performance comparison
    print("=== Performance Comparison ===")
    test_numbers = [1009, 1729, 2821, 46657, 65537, 982451653]
    compare_primality_tests(test_numbers)
    print()
    
    # Show the remarkable property: no known counterexamples
    print("=== Remarkable Property ===")
    print("The Baillie-PSW test has NO KNOWN COMPOSITE COUNTEREXAMPLES.")
    print("This means that despite being a probabilistic test, no composite")
    print("number has ever been found that passes both the Miller-Rabin")
    print("base-2 test and the strong Lucas test with the standard parameters.")
    print()
    print("Key facts:")
    print("- Any counterexample must be > 10^15")
    print("- Heuristic arguments suggest counterexamples are extremely rare")
    print("- Some variants proven to have no counterexamples below 2^64")
    print("- Widely used in computational number theory")
    
    # Demonstrate Lucas sequence computation
    print("\n=== Lucas Sequence Demonstration ===")
    n, P, Q = 97, 1, -1
    print(f"Computing Lucas sequences for n={n}, P={P}, Q={Q}")
    print(f"Discriminant D = P² - 4Q = {P*P - 4*Q}")
    
    for k in [1, 2, 5, 10, 20]:
        U, V = lucas_sequence_efficient(n, P, Q, k)
        print(f"U_{k} ≡ {U} (mod {n}), V_{k} ≡ {V} (mod {n})")
    
    print(f"\nJacobi symbol ({P*P - 4*Q}/{n}) = {jacobi_symbol(P*P - 4*Q, n)}")
```

## Theoretical Analysis

### Why Baillie-PSW Works So Well

The effectiveness of Baillie-PSW comes from combining two fundamentally different types of tests:

1. **Miller-Rabin base 2**: Based on the structure of $(\mathbb{Z}/n\mathbb{Z})^*$ as a group
2. **Strong Lucas test**: Based on recurrence relations and quadratic character

Numbers that fool both tests simultaneously would need to have very special arithmetic properties that appear to be extremely rare or non-existent.

### Computational Complexity

- **Time complexity**: $O(\log^3 n)$ per test, same as individual components
- **Space complexity**: $O(\log n)$ for intermediate computations
- **Practical performance**: Typically faster than multiple Miller-Rabin rounds

### Comparison with Other Tests

| Test | Error Rate | Known Counterexamples | Computational Cost |
|------|------------|----------------------|-------------------|
| Fermat | High | Carmichael numbers | $O(\log^3 n)$ |
| Miller-Rabin (1 round) | ≤ 1/4 | Infinitely many | $O(\log^3 n)$ |
| Miller-Rabin (k rounds) | ≤ 4^(-k) | Infinitely many | $O(k \log^3 n)$ |
| **Baillie-PSW** | **Unknown** | **None known** | $O(\log^3 n)$ |

## Applications

### Computational Number Theory
- **Integer factorization**: Verify primality of factors
- **Algebraic number theory**: Test norms and discriminants
- **Cryptographic research**: Analyze security of number-theoretic assumptions

### Mathematical Software
- **Computer algebra systems**: Mathematica, Sage, PARI/GP use variants of Baillie-PSW
- **Cryptographic libraries**: Some implementations use it for high-confidence primality testing
- **Research tools**: Standard in computational number theory research

### Special Investigations
- **Prime searching**: Used in searches for large primes with special properties
- **Sequence analysis**: Testing numbers in mathematical sequences
- **Conjecture verification**: Checking primality in various mathematical contexts

## Variants and Extensions

### BPSW Variants
- **Extra strong Lucas test**: Stronger version with different parameter selection
- **Quadratic Frobenius test**: Alternative to Lucas test using matrix operations
- **Combined tests**: Integration with additional primality tests

### Parameter Modifications
- **Different base selection**: Using bases other than 2 for Miller-Rabin component
- **Alternative discriminant sequences**: Different methods for choosing $D$
- **Optimization strategies**: Faster parameter finding algorithms

## Current Research

### Theoretical Questions
- **Existence of counterexamples**: Are there any composite numbers that pass Baillie-PSW?
- **Effective bounds**: How large must the smallest counterexample be?
- **Probabilistic analysis**: What is the actual error rate?

### Computational Searches
- Extensive computer searches have found no counterexamples up to very large bounds
- Distributed computing projects continue searching
- Special classes of numbers tested for potential counterexamples

## References

- Baillie, Robert; Wagstaff Jr., Samuel S. (1980). "Lucas Pseudoprimes"
- Pomerance, Carl; Selfridge, John L.; Wagstaff Jr., Samuel S. (1980). "The pseudoprimes to 25·10^9"
- Crandall, Richard; Pomerance, Carl (2005). "Prime Numbers: A Computational Perspective"
- Martin, Marcel (2018). "Strengthening the Baillie-PSW primality test"
- Jacobsen, Dana (2014). "Pseudoprime Statistics, Tables, and Data"