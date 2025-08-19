# Miller-Rabin Primality Test

## Overview

The Miller-Rabin primality test is a probabilistic algorithm used to determine whether a given number is likely to be prime. Developed by Gary L. Miller in 1976 and later improved by Michael O. Rabin in 1980, it is one of the most widely used primality testing algorithms due to its efficiency and reliability. Unlike deterministic tests, Miller-Rabin can give false positives but never false negatives, making it suitable for cryptographic applications where speed is crucial.

## Mathematical Foundation

### Theoretical Background

The Miller-Rabin test is based on **Fermat's Little Theorem** and properties of **quadratic residues**.

**Fermat's Little Theorem**: If $p$ is prime and $a$ is not divisible by $p$, then:
$$a^{p-1} \equiv 1 \pmod{p}$$

**Miller-Rabin Extension**: For an odd prime $p$, we can write $p - 1 = 2^r \cdot d$ where $d$ is odd. Then for any $a$ not divisible by $p$, either:
1. $a^d \equiv 1 \pmod{p}$, or  
2. $a^{2^i \cdot d} \equiv -1 \pmod{p}$ for some $0 \leq i < r$

### Algorithm Description

**Input**: An odd integer $n > 3$ to test for primality, and a parameter $k$ (number of rounds)

**Steps**:

1. **Decomposition**: Write $n - 1 = 2^r \cdot d$ where $d$ is odd
2. **Witness Loop**: Repeat $k$ times:
   - Choose random $a$ where $2 \leq a \leq n - 2$
   - Compute $x = a^d \bmod n$
   - If $x = 1$ or $x = n - 1$, continue to next iteration
   - For $i = 1$ to $r - 1$:
     - Set $x = x^2 \bmod n$
     - If $x = n - 1$, continue to next iteration
   - If we reach here, $n$ is composite (return "composite")
3. **Result**: If all $k$ rounds pass, return "probably prime"

### Error Analysis

The Miller-Rabin test has a **one-sided error**:
- If it returns "composite", the number is definitely composite
- If it returns "probably prime", there's a small chance it's actually composite

**Error Probability**: For a composite number $n$, the probability that a single round fails to detect compositeness is at most $\frac{1}{4}$.

After $k$ rounds, the probability of error is at most $(\frac{1}{4})^k = 4^{-k}$.

**Examples**:
- $k = 10$: Error probability ≤ $2^{-20} \approx 9.5 \times 10^{-7}$
- $k = 20$: Error probability ≤ $2^{-40} \approx 9.1 \times 10^{-13}$

### Witnesses and Strong Liars

- **Witness**: A value $a$ that proves $n$ is composite
- **Strong liar**: A value $a$ that makes a composite $n$ appear prime

For most composite numbers, the fraction of strong liars is much smaller than $\frac{1}{4}$, making the test very reliable in practice.

## Implementation Details

### Optimization Techniques

1. **Small Prime Division**: Check divisibility by small primes first
2. **Base Selection**: Use specific bases for deterministic results in certain ranges
3. **Fast Exponentiation**: Use binary exponentiation for efficiency
4. **Early Termination**: Stop as soon as compositeness is proven

### Deterministic Variants

For numbers up to certain limits, specific sets of bases guarantee correctness:
- $n < 2,047$: bases $\{2\}$
- $n < 1,373,653$: bases $\{2, 3\}$  
- $n < 9,080,191$: bases $\{31, 73\}$
- $n < 25,326,001$: bases $\{2, 3, 5\}$
- $n < 3,215,031,751$: bases $\{2, 3, 5, 7\}$

## Python Implementation

```python
import random
import math
from typing import List, Tuple, Optional

def miller_rabin_single(n: int, a: int) -> bool:
    """
    Single round of Miller-Rabin test with witness a
    Returns True if n passes the test (probably prime)
    Returns False if n fails the test (definitely composite)
    """
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False
    
    # Write n-1 as 2^r * d where d is odd
    r = 0
    d = n - 1
    while d % 2 == 0:
        r += 1
        d //= 2
    
    # Compute a^d mod n
    x = pow(a, d, n)
    
    # If x ≡ 1 (mod n), then n passes this round
    if x == 1:
        return True
    
    # Check if x ≡ -1 (mod n) for any of the r-1 squarings
    for _ in range(r):
        if x == n - 1:
            return True
        x = pow(x, 2, n)
    
    # If we get here, a is a witness to n's compositeness
    return False

def miller_rabin(n: int, k: int = 10) -> bool:
    """
    Miller-Rabin primality test with k rounds
    Returns True if n is probably prime
    Returns False if n is definitely composite
    """
    if n < 2:
        return False
    if n == 2 or n == 3:
        return True
    if n % 2 == 0:
        return False
    
    # Perform k rounds of testing
    for _ in range(k):
        # Choose random witness a in range [2, n-2]
        a = random.randrange(2, n - 1)
        
        if not miller_rabin_single(n, a):
            return False  # Composite
    
    return True  # Probably prime

def deterministic_miller_rabin(n: int) -> bool:
    """
    Deterministic Miller-Rabin test using precomputed witness sets
    Guaranteed correct for numbers up to certain bounds
    """
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False
    
    # Deterministic witness sets for various ranges
    if n < 2047:
        witnesses = [2]
    elif n < 1373653:
        witnesses = [2, 3]
    elif n < 9080191:
        witnesses = [31, 73]
    elif n < 25326001:
        witnesses = [2, 3, 5]
    elif n < 3215031751:
        witnesses = [2, 3, 5, 7]
    elif n < 4759123141:
        witnesses = [2, 7, 61]
    elif n < 1122004669633:
        witnesses = [2, 13, 23, 1662803]
    elif n < 2152302898747:
        witnesses = [2, 3, 5, 7, 11]
    elif n < 3474749660383:
        witnesses = [2, 3, 5, 7, 11, 13]
    elif n < 341550071728321:
        witnesses = [2, 3, 5, 7, 11, 13, 17]
    else:
        # Fall back to probabilistic test for very large numbers
        return miller_rabin(n, 20)
    
    # Test with each witness
    for a in witnesses:
        if a >= n:
            continue
        if not miller_rabin_single(n, a):
            return False
    
    return True

def optimized_miller_rabin(n: int, k: int = 10) -> bool:
    """
    Optimized Miller-Rabin with preliminary checks
    """
    # Handle small cases
    if n < 2:
        return False
    if n < 4:
        return True  # 2 and 3 are prime
    if n % 2 == 0:
        return False
    
    # Check divisibility by small primes
    small_primes = [3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]
    for p in small_primes:
        if n == p:
            return True
        if n % p == 0:
            return False
    
    # Use deterministic test if possible
    if n < 341550071728321:
        return deterministic_miller_rabin(n)
    
    # Otherwise use probabilistic test
    return miller_rabin(n, k)

def find_witnesses(n: int, max_witnesses: int = 100) -> List[int]:
    """
    Find witnesses that prove n is composite
    Returns empty list if no witnesses found (n might be prime)
    """
    if n < 2 or n == 2:
        return []
    if n % 2 == 0:
        return [2]
    
    witnesses = []
    for a in range(2, min(n - 1, max_witnesses + 2)):
        if not miller_rabin_single(n, a):
            witnesses.append(a)
    
    return witnesses

def miller_rabin_detailed(n: int, a: int) -> dict:
    """
    Detailed Miller-Rabin test showing all intermediate steps
    Returns dictionary with computation details
    """
    if n < 2:
        return {"error": "n must be >= 2"}
    if n % 2 == 0:
        return {"error": "n must be odd", "result": "composite"}
    
    # Decompose n-1 = 2^r * d
    r = 0
    d = n - 1
    while d % 2 == 0:
        r += 1
        d //= 2
    
    result = {
        "n": n,
        "witness": a,
        "decomposition": f"{n-1} = 2^{r} × {d}",
        "r": r,
        "d": d,
        "sequence": []
    }
    
    # Compute a^d mod n
    x = pow(a, d, n)
    result["sequence"].append(f"a^d = {a}^{d} ≡ {x} (mod {n})")
    
    if x == 1:
        result["result"] = "probably prime"
        result["reason"] = f"a^d ≡ 1 (mod n)"
        return result
    
    if x == n - 1:
        result["result"] = "probably prime"
        result["reason"] = f"a^d ≡ -1 (mod n)"
        return result
    
    # Square x repeatedly
    for i in range(1, r):
        x = pow(x, 2, n)
        result["sequence"].append(f"x^2 ≡ {x} (mod {n})")
        
        if x == 1:
            result["result"] = "composite"
            result["reason"] = f"Found x^2 ≡ 1 (mod n) where x ≢ ±1 (mod n)"
            return result
        
        if x == n - 1:
            result["result"] = "probably prime"
            result["reason"] = f"Found x ≡ -1 (mod n) after {i} squarings"
            return result
    
    result["result"] = "composite"
    result["reason"] = f"No x ≡ -1 (mod n) found in sequence"
    return result

def carmichael_test() -> List[int]:
    """
    Test Miller-Rabin on Carmichael numbers (known composites that fool Fermat test)
    """
    # Small Carmichael numbers
    carmichael_numbers = [561, 1105, 1729, 2465, 2821, 6601, 8911, 10585, 15841, 29341]
    
    results = []
    for n in carmichael_numbers:
        # Test with multiple rounds
        is_prime_mr = miller_rabin(n, 10)
        
        # Find witnesses
        witnesses = find_witnesses(n, 20)
        
        results.append({
            "number": n,
            "miller_rabin_result": is_prime_mr,
            "is_actually_prime": False,
            "witnesses_found": len(witnesses),
            "first_witness": witnesses[0] if witnesses else None
        })
    
    return results

def performance_comparison(numbers: List[int]) -> List[dict]:
    """
    Compare performance of different primality tests
    """
    import time
    
    results = []
    
    for n in numbers:
        result = {"number": n, "digits": len(str(n))}
        
        # Trial division (for comparison, small numbers only)
        if n < 10**6:
            start = time.time()
            trial_result = is_prime_trial_division(n)
            trial_time = time.time() - start
            result["trial_division"] = {"result": trial_result, "time": trial_time}
        
        # Miller-Rabin (probabilistic)
        start = time.time()
        mr_result = miller_rabin(n, 10)
        mr_time = time.time() - start
        result["miller_rabin"] = {"result": mr_result, "time": mr_time}
        
        # Deterministic Miller-Rabin
        start = time.time()
        det_result = deterministic_miller_rabin(n)
        det_time = time.time() - start
        result["deterministic_mr"] = {"result": det_result, "time": det_time}
        
        results.append(result)
    
    return results

def is_prime_trial_division(n: int) -> bool:
    """Trial division primality test for comparison"""
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False
    
    for i in range(3, int(math.sqrt(n)) + 1, 2):
        if n % i == 0:
            return False
    return True

def generate_probable_primes(bits: int, count: int = 10) -> List[int]:
    """
    Generate probable primes using Miller-Rabin
    Used in cryptographic key generation
    """
    primes = []
    attempts = 0
    max_attempts = count * 100
    
    while len(primes) < count and attempts < max_attempts:
        # Generate random odd number of specified bit length
        candidate = random.getrandbits(bits)
        candidate |= (1 << bits - 1) | 1  # Ensure proper bit length and odd
        
        # Test with Miller-Rabin
        if miller_rabin(candidate, 20):
            primes.append(candidate)
        
        attempts += 1
    
    return primes

# Example usage and demonstrations
if __name__ == "__main__":
    print("=== Miller-Rabin Primality Test ===\n")
    
    # Test known primes and composites
    test_cases = [
        (17, True, "Small prime"),
        (25, False, "Perfect square"),
        (97, True, "Small prime"), 
        (100, False, "Even composite"),
        (561, False, "Carmichael number"),
        (1009, True, "Medium prime"),
        (1024, False, "Power of 2"),
        (65537, True, "Fermat prime F_2")
    ]
    
    print("Testing known cases:")
    for n, expected, description in test_cases:
        result = optimized_miller_rabin(n, 10)
        status = "✓" if result == expected else "✗"
        print(f"{status} {n:>6} ({description}): {result}")
    print()
    
    # Detailed analysis of a specific case
    print("=== Detailed Miller-Rabin Analysis ===")
    n = 561  # Carmichael number
    a = 2    # Known witness
    
    detail = miller_rabin_detailed(n, a)
    print(f"Testing n = {detail['n']} with witness a = {detail['witness']}")
    print(f"Decomposition: {detail['decomposition']}")
    print("Computation sequence:")
    for step in detail['sequence']:
        print(f"  {step}")
    print(f"Result: {detail['result']}")
    print(f"Reason: {detail['reason']}\n")
    
    # Test Carmichael numbers
    print("=== Carmichael Numbers Test ===")
    carmichael_results = carmichael_test()
    
    print("Testing Miller-Rabin against Carmichael numbers:")
    for result in carmichael_results:
        n = result['number']
        mr_result = result['miller_rabin_result']
        witnesses = result['witnesses_found']
        first_witness = result['first_witness']
        
        status = "✓" if not mr_result else "✗"  # Should detect composite
        print(f"{status} {n:>5}: Miller-Rabin says {'prime' if mr_result else 'composite'}")
        print(f"       Witnesses found: {witnesses}, first witness: {first_witness}")
    print()
    
    # Performance comparison
    print("=== Performance Comparison ===")
    test_numbers = [1009, 10007, 100003, 982451653, 9876543211]
    
    perf_results = performance_comparison(test_numbers)
    
    print("Comparing different primality tests:")
    print(f"{'Number':<12} {'Digits':<6} {'Trial Div':<12} {'Miller-Rabin':<14} {'Deterministic':<14}")
    print("-" * 70)
    
    for result in perf_results:
        n = result['number']
        digits = result['digits']
        
        trial = result.get('trial_division', {})
        mr = result['miller_rabin']
        det = result['deterministic_mr']
        
        trial_str = f"{trial['time']:.6f}s" if trial else "N/A"
        mr_str = f"{mr['time']:.6f}s"
        det_str = f"{det['time']:.6f}s"
        
        print(f"{n:<12} {digits:<6} {trial_str:<12} {mr_str:<14} {det_str:<14}")
    print()
    
    # Generate probable primes
    print("=== Probable Prime Generation ===")
    print("Generating 512-bit probable primes:")
    
    probable_primes = generate_probable_primes(512, 5)
    for i, p in enumerate(probable_primes, 1):
        print(f"{i}. {p}")
        print(f"   Hex: {hex(p)}")
        print(f"   Bits: {p.bit_length()}")
        
        # Double-check with deterministic test if possible
        if p < 341550071728321:
            det_check = deterministic_miller_rabin(p)
            print(f"   Deterministic verification: {det_check}")
        print()
    
    # Error probability demonstration
    print("=== Error Probability Analysis ===")
    print("Miller-Rabin error probability by number of rounds:")
    
    for k in [1, 5, 10, 20, 40]:
        error_prob = (0.25) ** k
        print(f"k = {k:>2}: Error probability ≤ {error_prob:.2e} (≈ 1 in {1/error_prob:.0e})")
```

## Applications in Cryptography

### RSA Key Generation
```python
def generate_rsa_prime(bits: int) -> int:
    """Generate a prime for RSA using Miller-Rabin"""
    while True:
        candidate = random.getrandbits(bits)
        candidate |= (1 << bits - 1) | 1  # Ensure proper bit length and odd
        
        # Additional checks for RSA security
        if candidate % 3 == 0 or candidate % 5 == 0:
            continue
            
        if miller_rabin(candidate, 40):  # High confidence for cryptographic use
            return candidate
```

### Elliptic Curve Cryptography
Miller-Rabin is used to verify that curve parameters are prime, ensuring the mathematical properties required for secure elliptic curve operations.

### Digital Signature Schemes
DSA and ECDSA require prime moduli and curve parameters, all verified using Miller-Rabin testing.

## Advantages and Limitations

### Advantages
- **Fast**: $O(\log^3 n)$ time complexity per round
- **Reliable**: Very low error probability with sufficient rounds
- **Scalable**: Works efficiently for very large numbers
- **Implementable**: Simple algorithm suitable for various platforms

### Limitations
- **Probabilistic**: Cannot provide absolute certainty (except in deterministic variant)
- **One-sided error**: May incorrectly identify composites as prime
- **Random dependency**: Requires good random number generation

## Comparison with Other Tests

| Test | Type | Complexity | Reliability | Use Case |
|------|------|------------|-------------|-----------|
| Trial Division | Deterministic | $O(\sqrt{n})$ | Perfect | Small numbers |
| Fermat Test | Probabilistic | $O(\log^3 n)$ | Poor (Carmichael numbers) | Educational |
| Miller-Rabin | Probabilistic | $O(\log^3 n)$ | Excellent | General purpose |
| AKS Test | Deterministic | $O(\log^{12} n)$ | Perfect | Theoretical interest |
| Lucas-Lehmer | Deterministic | $O(\log^3 n)$ | Perfect | Mersenne numbers only |

## Historical Context

- **1976**: Gary Miller develops deterministic version assuming Generalized Riemann Hypothesis
- **1980**: Michael Rabin creates unconditional probabilistic version
- **Present**: Most widely used primality test in practical applications

## Security Considerations

### Cryptographic Standards
- **FIPS 186-4**: Recommends Miller-Rabin for DSA prime generation
- **RFC 4086**: Guidelines for random number generation in Miller-Rabin
- **Common Criteria**: Security evaluations often require Miller-Rabin verification

### Implementation Security
- Use cryptographically secure random number generators
- Perform sufficient rounds (typically 40+ for cryptographic applications)
- Implement constant-time operations to prevent timing attacks
- Validate all inputs to prevent malicious inputs

## References

- Miller, Gary L. (1976). "Riemann's Hypothesis and tests for primality"
- Rabin, Michael O. (1980). "Probabilistic algorithm for testing primality"
- Handbook of Applied Cryptography, Chapter 4
- FIPS 186-4: Digital Signature Standard
- Crandall, Richard; Pomerance, Carl (2005). "Prime Numbers: A Computational Perspective"