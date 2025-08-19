# Pocklington Primality Test

## Overview

The Pocklington primality test, developed by Henry Cabourn Pocklington in 1914, is a deterministic primality test that can prove the primality of certain numbers when partial factorization of $n-1$ is known. Unlike probabilistic tests such as Miller-Rabin, Pocklington's test provides a definitive proof of primality for numbers of the form where a significant portion of $n-1$ can be factored. This makes it particularly valuable in number theory research and for proving the primality of specific classes of large numbers.

## Mathematical Foundation

### Theoretical Background

The Pocklington test is based on **Fermat's Little Theorem** and the structure of the multiplicative group modulo $n$.

**Fermat's Little Theorem**: If $p$ is prime and $\gcd(a, p) = 1$, then:
$$a^{p-1} \equiv 1 \pmod{p}$$

### Pocklington's Theorem

**Theorem (Pocklington, 1914)**: Let $n > 1$ and suppose that $n - 1 = F \cdot R$ where:
- $F > \sqrt{n}$ (the "factored part")
- $\gcd(F, R) = 1$
- For each prime divisor $q$ of $F$, there exists an integer $a$ such that:
  - $\gcd(a, n) = 1$
  - $a^{n-1} \equiv 1 \pmod{n}$
  - $\gcd(a^{(n-1)/q} - 1, n) = 1$

Then $n$ is prime.

### Extended Pocklington Test

For practical implementation, we often use a weaker version:

**Theorem (Sufficient Condition)**: Let $n > 1$ and $n - 1 = F \cdot R$ where $F > \sqrt{n}$. If there exists an integer $a$ such that:
1. $\gcd(a, n) = 1$
2. $a^{n-1} \equiv 1 \pmod{n}$
3. For each prime divisor $q$ of $F$: $\gcd(a^{(n-1)/q} - 1, n) = 1$

Then $n$ is prime.

### Algorithm Description

**Input**: 
- An integer $n > 1$ to test for primality
- A partial factorization $n - 1 = F \cdot R$ where $F$ is the factored part

**Precondition**: $F > \sqrt{n}$ (essential for the test to work)

**Steps**:
1. **Verify precondition**: Check that $F > \sqrt{n}$
2. **Find witness**: Find an integer $a$ with $\gcd(a, n) = 1$
3. **Fermat test**: Verify $a^{n-1} \equiv 1 \pmod{n}$
4. **Factor verification**: For each prime factor $q$ of $F$:
   - Compute $b = a^{(n-1)/q} \bmod n$
   - Verify $\gcd(b - 1, n) = 1$
5. **Conclusion**: If all conditions are satisfied, $n$ is prime

## Comparison with Other Tests

### Advantages
- **Deterministic**: Provides definitive proof of primality
- **Theoretical importance**: Essential for proving primality of special number forms
- **Partial factorization**: Only requires factorization of part of $n-1$
- **Mathematical rigor**: Based on solid group-theoretic foundations

### Disadvantages
- **Factorization requirement**: Needs partial factorization of $n-1$
- **Limited applicability**: Only works when $F > \sqrt{n}$
- **Computational complexity**: May be slower than probabilistic tests
- **Not general-purpose**: Cannot be applied to arbitrary numbers easily

## Applications

### Number Theory Research
- Proving primality of Mersenne numbers
- Establishing primality in sequences like Fermat numbers
- Verifying primality of numbers with special forms

### Cryptographic Applications
- Generating provable primes for cryptographic protocols
- Verifying primality of key generation parameters
- Research into prime generation algorithms

## Python Implementation

```python
import math
import random
from typing import List, Tuple, Optional, Dict
from collections import defaultdict

def gcd(a: int, b: int) -> int:
    """Calculate Greatest Common Divisor using Euclidean algorithm"""
    while b:
        a, b = b, a % b
    return a

def factor_trial_division(n: int, limit: int = 10000) -> List[Tuple[int, int]]:
    """
    Factor n using trial division up to limit
    Returns list of (prime, exponent) pairs
    """
    factors = []
    d = 2
    
    while d * d <= n and d <= limit:
        if n % d == 0:
            exp = 0
            while n % d == 0:
                n //= d
                exp += 1
            factors.append((d, exp))
        d += 1 if d == 2 else 2
    
    if n > 1:
        factors.append((n, 1))
    
    return factors

def is_prime_basic(n: int) -> bool:
    """Basic primality test for small numbers"""
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

def find_primitive_root_candidate(n: int, max_attempts: int = 100) -> Optional[int]:
    """
    Find a candidate that might work as a witness for Pocklington test
    Not necessarily a primitive root, but suitable for the test
    """
    for _ in range(max_attempts):
        a = random.randint(2, n - 1)
        if gcd(a, n) == 1:
            # Quick check: a^(n-1) mod n should be 1
            if pow(a, n - 1, n) == 1:
                return a
    return None

def pocklington_test(n: int, factorization: List[Tuple[int, int]]) -> Dict:
    """
    Pocklington primality test
    
    Args:
        n: Number to test for primality
        factorization: List of (prime, exponent) pairs representing factors of n-1
    
    Returns:
        Dictionary with test results
    """
    if n < 2:
        return {"result": False, "reason": "n must be >= 2"}
    
    if n == 2:
        return {"result": True, "reason": "2 is prime"}
    
    if n % 2 == 0:
        return {"result": False, "reason": "n is even"}
    
    # Calculate F (factored part) and R (unfactored part)
    F = 1
    prime_factors = []
    
    for prime, exp in factorization:
        F *= prime ** exp
        prime_factors.append(prime)
    
    if (n - 1) % F != 0:
        return {"result": False, "reason": "Invalid factorization: F does not divide n-1"}
    
    R = (n - 1) // F
    
    result = {
        "n": n,
        "n_minus_1": n - 1,
        "F": F,
        "R": R,
        "factorization": factorization,
        "prime_factors": prime_factors,
        "sqrt_n": math.sqrt(n)
    }
    
    # Check if F > sqrt(n) (essential condition)
    if F <= math.sqrt(n):
        result.update({
            "result": False,
            "reason": f"F = {F} <= sqrt(n) = {math.sqrt(n):.2f}, condition not satisfied"
        })
        return result
    
    result["F_condition"] = True
    
    # Find a suitable witness
    witness = find_primitive_root_candidate(n, 50)
    if witness is None:
        result.update({
            "result": False,
            "reason": "Could not find suitable witness"
        })
        return result
    
    result["witness"] = witness
    
    # Verify a^(n-1) ≡ 1 (mod n)
    fermat_check = pow(witness, n - 1, n)
    if fermat_check != 1:
        result.update({
            "result": False,
            "reason": f"Fermat test failed: {witness}^{n-1} ≡ {fermat_check} (mod {n})",
            "fermat_result": fermat_check
        })
        return result
    
    result["fermat_check"] = True
    
    # Check gcd(a^((n-1)/q) - 1, n) = 1 for each prime factor q of F
    gcd_checks = {}
    
    for q in prime_factors:
        exponent = (n - 1) // q
        a_power = pow(witness, exponent, n)
        gcd_result = gcd(a_power - 1, n)
        
        gcd_checks[q] = {
            "exponent": exponent,
            "a_power": a_power,
            "gcd": gcd_result
        }
        
        if gcd_result != 1:
            result.update({
                "result": False,
                "reason": f"GCD check failed for prime {q}: gcd({a_power} - 1, {n}) = {gcd_result}",
                "gcd_checks": gcd_checks
            })
            return result
    
    result.update({
        "result": True,
        "reason": "All Pocklington conditions satisfied",
        "gcd_checks": gcd_checks
    })
    
    return result

def pocklington_with_partial_factorization(n: int, known_factors: List[Tuple[int, int]] = None) -> Dict:
    """
    Attempt Pocklington test with automatic partial factorization
    """
    if known_factors is None:
        known_factors = []
    
    # Try to factor n-1 partially
    remaining = n - 1
    all_factors = list(known_factors)
    
    # Remove known factors
    for prime, exp in known_factors:
        for _ in range(exp):
            if remaining % prime == 0:
                remaining //= prime
            else:
                return {"result": False, "reason": "Invalid known factors provided"}
    
    # Try to find more factors
    additional_factors = factor_trial_division(remaining, limit=10000)
    all_factors.extend(additional_factors)
    
    # Calculate total factored part
    F = 1
    for prime, exp in all_factors:
        F *= prime ** exp
    
    result = {"factorization_attempt": all_factors, "F": F}
    
    if F <= math.sqrt(n):
        result.update({
            "result": False,
            "reason": f"Could not factor enough of n-1: F = {F} <= sqrt(n) = {math.sqrt(n):.2f}"
        })
        return result
    
    # Proceed with Pocklington test
    pocklington_result = pocklington_test(n, all_factors)
    result.update(pocklington_result)
    
    return result

def generate_pocklington_prime(bits: int, max_attempts: int = 1000) -> Optional[Tuple[int, List[Tuple[int, int]]]]:
    """
    Generate a prime suitable for Pocklington test
    Returns (prime, factorization_of_prime_minus_1) or None
    """
    for _ in range(max_attempts):
        # Generate a candidate where n-1 has a large known factor
        # Strategy: n = k*p + 1 where p is a large known prime
        
        # Choose a large prime p
        p_bits = bits // 2
        p = random.getrandbits(p_bits)
        p |= (1 << p_bits - 1) | 1
        
        # Find a prime p using simple trial
        while not is_prime_basic(p):
            p += 2
            if p.bit_length() > p_bits + 5:  # Avoid infinite loop
                break
        
        if not is_prime_basic(p):
            continue
        
        # Choose small k
        k = random.randint(1, 2**(bits - p_bits))
        n = k * p + 1
        
        if n.bit_length() != bits:
            continue
        
        # n-1 = k*p, so p is a large factor of n-1
        if p > math.sqrt(n):
            # Test if n is prime using Pocklington
            factorization = [(p, 1)]  # We know p divides n-1
            
            # Try to factor k
            k_factors = factor_trial_division(k, limit=1000)
            factorization.extend(k_factors)
            
            result = pocklington_test(n, factorization)
            if result["result"]:
                return n, factorization
    
    return None

def verify_known_primes() -> List[Dict]:
    """
    Verify some known primes using Pocklington test
    """
    # Test cases: (n, partial_factorization_of_n_minus_1)
    test_cases = [
        # Small primes for verification
        (5, [(2, 2)]),  # 5-1 = 4 = 2^2
        (13, [(4, 1), (3, 1)]),  # 13-1 = 12 = 4*3
        (17, [(16, 1)]),  # 17-1 = 16 = 2^4
        (97, [(32, 1), (3, 1)]),  # 97-1 = 96 = 32*3
        
        # Larger examples
        (257, [(256, 1)]),  # Fermat prime F_3: 257-1 = 256 = 2^8
    ]
    
    results = []
    
    for n, factors in test_cases:
        print(f"\nTesting n = {n}")
        result = pocklington_test(n, factors)
        results.append(result)
        
        if result["result"]:
            print(f"  ✓ {n} is PRIME (Pocklington test)")
        else:
            print(f"  ✗ {n} failed: {result['reason']}")
    
    return results

def compare_with_trial_division(test_numbers: List[int]) -> None:
    """
    Compare Pocklington test with trial division for known primes
    """
    import time
    
    print("\n=== Comparison: Pocklington vs Trial Division ===")
    print(f"{'Number':<8} {'Trial Div':<12} {'Pocklington':<12} {'TD Time':<10} {'Pock Time':<10}")
    print("-" * 65)
    
    for n in test_numbers:
        # Trial division
        start_time = time.time()
        td_result = is_prime_basic(n)
        td_time = time.time() - start_time
        
        # Pocklington (with automatic factorization attempt)
        start_time = time.time()
        pock_result = pocklington_with_partial_factorization(n)
        pock_time = time.time() - start_time
        
        td_str = "PRIME" if td_result else "COMPOSITE"
        pock_str = "PRIME" if pock_result.get("result", False) else "FAILED"
        
        print(f"{n:<8} {td_str:<12} {pock_str:<12} {td_time:.6f}s  {pock_time:.6f}s")

# Example usage and demonstrations
if __name__ == "__main__":
    print("=== Pocklington Primality Test ===\n")
    
    # Verify known primes
    known_results = verify_known_primes()
    
    # Detailed analysis of a specific case
    print("\n=== Detailed Analysis: Testing n = 97 ===")
    n = 97
    # 97 - 1 = 96 = 32 × 3 = 2^5 × 3
    factorization = [(2, 5), (3, 1)]  # 2^5 = 32, 3^1 = 3
    
    result = pocklington_test(n, factorization)
    
    print(f"n = {result['n']}")
    print(f"n - 1 = {result['n_minus_1']}")
    print(f"Factorization: {result['factorization']}")
    print(f"F (factored part) = {result['F']}")
    print(f"R (unfactored part) = {result['R']}")
    print(f"sqrt(n) ≈ {result['sqrt_n']:.2f}")
    print(f"F > sqrt(n)? {result.get('F_condition', False)}")
    
    if 'witness' in result:
        print(f"Witness: a = {result['witness']}")
        print(f"a^(n-1) mod n = {pow(result['witness'], n-1, n)} (should be 1)")
        
        print("\nGCD checks for prime factors:")
        for q, check in result['gcd_checks'].items():
            print(f"  q = {q}: a^((n-1)/q) = {check['a_power']}, gcd({check['a_power']}-1, {n}) = {check['gcd']}")
    
    print(f"\nResult: {result['reason']}")
    
    # Test with insufficient factorization
    print("\n=== Testing with Insufficient Factorization ===")
    n = 101
    insufficient_factors = [(4, 1)]  # 101-1 = 100, but we only know 4 divides it
    
    result = pocklington_test(n, insufficient_factors)
    print(f"Testing n = {n} with partial factorization {insufficient_factors}")
    print(f"F = {result.get('F', 'N/A')}, sqrt(n) ≈ {math.sqrt(n):.2f}")
    print(f"Result: {result['reason']}")
    
    # Generate a Pocklington-suitable prime
    print("\n=== Generating Pocklington-Suitable Prime ===")
    print("Attempting to generate a 64-bit prime suitable for Pocklington test...")
    
    generated = generate_pocklington_prime(64, max_attempts=100)
    
    if generated:
        prime, factorization = generated
        print(f"Generated prime: {prime}")
        print(f"Prime bit length: {prime.bit_length()}")
        print(f"Factorization of {prime}-1: {factorization}")
        
        # Verify it
        verification = pocklington_test(prime, factorization)
        print(f"Verification: {'PASSED' if verification['result'] else 'FAILED'}")
        
        if not verification['result']:
            print(f"Failure reason: {verification['reason']}")
    else:
        print("Failed to generate suitable prime in given attempts")
    
    # Compare performance
    print("\n=== Performance Comparison ===")
    test_numbers = [97, 101, 257, 1009, 2017]
    compare_with_trial_division(test_numbers)
    
    # Mathematical properties demonstration
    print("\n=== Mathematical Properties ===")
    
    print("Key insight: Pocklington test works because:")
    print("1. If n is prime, then (Z/nZ)* is cyclic of order n-1")
    print("2. For a generator g, the order of g^((n-1)/q) is exactly q")
    print("3. If gcd(g^((n-1)/q) - 1, n) > 1, then n shares a factor with this element")
    print("4. This contradicts primality unless the gcd is exactly 1")
    
    print(f"\nFactorization requirements:")
    print("- Must factor 'enough' of n-1 so that F > sqrt(n)")
    print("- This ensures that any potential factor of n would be detected")
    print("- The unfactored remainder R must be small enough to not affect the proof")
    
    # Show limitation
    print("\n=== Limitations Demonstration ===")
    
    difficult_numbers = [
        (91, "91 = 7 × 13, but 90 = 2 × 3² × 5 (small factors only)"),
        (111, "111 = 3 × 37, but 110 = 2 × 5 × 11 (small factors only)")
    ]
    
    for n, description in difficult_numbers:
        print(f"\n{description}")
        result = pocklington_with_partial_factorization(n)
        print(f"  Automatic factorization result: {result.get('reason', 'Unknown')}")
        if 'F' in result:
            print(f"  Best F achieved: {result['F']} (need > {math.sqrt(n):.2f})")
```

## Theoretical Significance

### Group Theory Connection
The Pocklington test leverages the structure of the multiplicative group $(\mathbb{Z}/n\mathbb{Z})^*$. For a prime $p$, this group is cyclic of order $p-1$, and the test essentially verifies this cyclicity by checking that potential generators have the correct order properties.

### Relationship to Other Tests
- **Fermat Test**: Pocklington includes Fermat's test as a component
- **Lucas Test**: Similar theoretical foundation but different approach
- **Miller-Rabin**: More general but probabilistic; Pocklington is deterministic but requires factorization

### Certificates of Primality
The Pocklington test provides a **certificate of primality** - the factorization of $n-1$ and the witness $a$ serve as a verifiable proof that $n$ is prime. This is valuable in theoretical computer science and cryptographic applications where proof of primality is required.

## Practical Considerations

### When to Use Pocklington Test
- **Special form numbers**: Numbers where $n-1$ has known large factors
- **Theoretical research**: When definitive proof of primality is needed
- **Cryptographic protocols**: Where primality certificates are required
- **Sequence verification**: Testing numbers in mathematical sequences with known structure

### Limitations in Practice
- **Factorization bottleneck**: Requires factoring significant portion of $n-1$
- **Not general purpose**: Cannot be applied to arbitrary large numbers efficiently
- **Computational overhead**: More complex than probabilistic tests

## Modern Applications

### Cryptographic Prime Generation
Some cryptographic applications require **provable primes** rather than probable primes. The Pocklington test, combined with modern factorization techniques, can provide the necessary certificates.

### Mathematical Software
Computer algebra systems often implement Pocklington-style tests for proving primality of numbers arising in mathematical research, particularly in number theory and algebraic geometry.

## References

- Pocklington, Henry Cabourn (1914). "The determination of the prime or composite nature of large numbers by Fermat's theorem"
- Brillhart, John; Lehmer, Derrick Henry; Selfridge, John (1975). "New primality criteria and factorizations of 2^m ± 1"
- Cohen, Henri (1993). "A Course in Computational Algebraic Number Theory"
- Crandall, Richard; Pomerance, Carl (2005). "Prime Numbers: A Computational Perspective"
- Williams, Hugh C. (1998). "Édouard Lucas and primality testing"