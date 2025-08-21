# Fermat Primes

## Overview

Fermat primes are prime numbers of the form $F_n = 2^{2^n} + 1$, where $n$ is a non-negative integer. Named after Pierre de Fermat (1601-1665), these primes have a unique mathematical structure and fascinating connections to geometry, particularly in the construction of regular polygons. Unlike Mersenne primes, only five Fermat primes are known, and it is conjectured that no others exist.

## Mathematical Foundation

### Definition

A **Fermat number** is a positive integer of the form:
```
Fₙ = 2^(2^n) + 1
```

where $n \geq 0$ is a non-negative integer.

A **Fermat prime** is a Fermat number that is also prime.

### Known Fermat Numbers

The first few Fermat numbers are:

| $n$ | $2^n$ | $F_n = 2^{2^n} + 1$ | Value | Status |
|-----|-------|-------------------|--------|---------|
| 0 | 1 | $2^2 + 1$ | 3 | Prime |
| 1 | 2 | $2^4 + 1$ | 17 | Prime |
| 2 | 4 | $2^{16} + 1$ | 65537 | Prime |
| 3 | 8 | $2^{256} + 1$ | $2^{256} + 1$ | Prime |
| 4 | 16 | $2^{65536} + 1$ | $2^{65536} + 1$ | Prime |
| 5 | 32 | $2^{2^{32}} + 1$ | $2^{4294967296} + 1$ | Composite |

### Fundamental Properties

**Theorem 1**: If $F_n = 2^{2^n} + 1$ is prime, then $n$ must be a power of 2.

**Proof**: If $n = ab$ where $a$ is odd and $a > 1$, then $2^n + 1$ divides $2^{an} + 1 = 2^{2^n} + 1$, making $F_n$ composite.

**Theorem 2**: Fermat numbers are pairwise coprime.
```
gcd(Fₘ, Fₙ) = 1 for m ≠ n
```

**Proof**: For $m < n$, we have $F_m$ divides $2^{2^n} - 1$, but $\gcd(2^{2^n} - 1, 2^{2^n} + 1) = 1$.

### Connection to Regular Polygons

**Gauss-Wantzel Theorem**: A regular p-sided polygon is constructible with compass and straightedge if and only if:
```
p = 2^k · p₁ · p₂ · ... · pᵣ
```

where $k \geq 0$ and $p_1, p_2, \ldots, p_r$ are distinct Fermat primes.

**Constructible polygons** based on known Fermat primes:
- Triangle ($p = 3 = F_0$)
- Pentagon ($p = 5 = F_1$) 
- 15-gon ($p = 15 = 3 \times 5 = F_0 \times F_1$)
- 17-gon ($p = 17 = F_1$)
- 257-gon ($p = 257 = F_2$)
- 65537-gon ($p = 65537 = F_2$)

### Primality Testing for Fermat Numbers

For large Fermat numbers, specialized tests are used:

**Pépin's Test**: For n ≥ 1, Fₙ is prime if and only if:
```
3^((Fₙ - 1)/2) ≡ -1 (mod Fₙ)
```

This test is based on the fact that 3 is a quadratic non-residue modulo $F_n$ when $F_n$ is prime.

## Python Implementation

```python
import math
import random
from typing import List, Tuple, Optional, Dict

def fermat_number(n: int) -> int:
    """Calculate the nth Fermat number F_n = 2^(2^n) + 1"""
    if n < 0:
        raise ValueError("n must be non-negative")
    
    # For large n, this will be computationally expensive
    if n > 4:
        print(f"Warning: F_{n} is extremely large ({2**n} bits)")
        return None
    
    return (1 << (1 << n)) + 1

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

def miller_rabin(n: int, k: int = 10) -> bool:
    """Miller-Rabin primality test"""
    if n < 2:
        return False
    if n == 2 or n == 3:
        return True
    if n % 2 == 0:
        return False
    
    # Write n-1 as d * 2^r
    r = 0
    d = n - 1
    while d % 2 == 0:
        r += 1
        d //= 2
    
    # Perform k rounds of testing
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

def pepin_test(n: int) -> bool:
    """
    Pépin's test for Fermat number primality
    For F_n where n >= 1, F_n is prime iff 3^((F_n - 1)/2) ≡ -1 (mod F_n)
    """
    if n < 1:
        return False
    
    F_n = fermat_number(n)
    if F_n is None:
        print(f"Cannot compute F_{n} - too large")
        return None
    
    if F_n == 3:  # F_0 = 3, special case
        return True
    
    # Test: 3^((F_n - 1)/2) ≡ -1 (mod F_n)
    exponent = (F_n - 1) // 2
    result = pow(3, exponent, F_n)
    
    return result == F_n - 1  # -1 ≡ F_n - 1 (mod F_n)

def factorize_fermat_number(n: int, max_factor: int = 1000000) -> Optional[List[int]]:
    """
    Attempt to find factors of Fermat number F_n
    Uses trial division up to max_factor
    """
    F_n = fermat_number(n)
    if F_n is None:
        return None
    
    factors = []
    temp = F_n
    
    # Check small factors
    for p in range(3, min(max_factor, int(math.sqrt(F_n)) + 1), 2):
        if not is_prime_basic(p):
            continue
        
        # For Fermat numbers, any prime factor p must satisfy p ≡ 1 (mod 2^(n+2))
        if n >= 2 and p % (1 << (n + 2)) != 1:
            continue
        
        while temp % p == 0:
            factors.append(p)
            temp //= p
            
        if temp == 1:
            break
    
    if temp > 1 and temp != F_n:
        factors.append(temp)
    
    return factors if len(factors) > 1 else None

def known_fermat_primes() -> List[Tuple[int, int]]:
    """Return list of known Fermat primes as (n, F_n) tuples"""
    return [
        (0, 3),
        (1, 17),
        (2, 65537),
        (3, fermat_number(3)),
        (4, fermat_number(4))
    ]

def verify_fermat_prime(n: int) -> Dict:
    """
    Comprehensive verification of whether F_n is prime
    Returns dictionary with test results
    """
    F_n = fermat_number(n)
    if F_n is None:
        return {"error": f"F_{n} too large to compute"}
    
    result = {
        "n": n,
        "F_n": F_n,
        "decimal_digits": len(str(F_n)),
        "binary_length": F_n.bit_length(),
        "hex_representation": hex(F_n)
    }
    
    # Basic primality test for small numbers
    if F_n < 10**6:
        result["basic_primality"] = is_prime_basic(F_n)
    
    # Miller-Rabin test
    if F_n < 10**9:
        result["miller_rabin"] = miller_rabin(F_n, 20)
    
    # Pépin's test (most reliable for Fermat numbers)
    if n >= 1:
        pepin_result = pepin_test(n)
        result["pepin_test"] = pepin_result
        result["is_prime"] = pepin_result
    else:
        result["is_prime"] = F_n == 3  # F_0 = 3
    
    # Try to find small factors if composite
    if not result.get("is_prime", False):
        factors = factorize_fermat_number(n, 100000)
        if factors:
            result["small_factors"] = factors
    
    return result

def constructible_polygon_sides(max_sides: int = 1000000) -> List[int]:
    """
    Find all constructible regular polygon side counts up to max_sides
    Based on known Fermat primes
    """
    # Known Fermat primes
    fermat_primes = [3, 17, 65537]  # F_0, F_1, F_2
    # Note: F_3 and F_4 are too large for practical polygon construction
    
    constructible = set()
    
    # All combinations of 2^k * product of distinct Fermat primes
    for k in range(20):  # 2^k factor
        power_of_2 = 1 << k
        
        if power_of_2 > max_sides:
            break
        
        # Generate all subsets of Fermat primes
        for mask in range(1 << len(fermat_primes)):
            product = power_of_2
            
            for i in range(len(fermat_primes)):
                if mask & (1 << i):
                    product *= fermat_primes[i]
                    
                if product > max_sides:
                    break
            
            if product <= max_sides:
                constructible.add(product)
    
    return sorted(list(constructible))

def analyze_fermat_pattern() -> None:
    """Analyze the pattern in Fermat numbers and their properties"""
    print("=== Fermat Numbers Analysis ===")
    
    for n in range(6):  # Analyze F_0 through F_5
        if n <= 4:
            F_n = fermat_number(n)
            result = verify_fermat_prime(n)
            
            print(f"F_{n} = 2^(2^{n}) + 1 = 2^{2**n} + 1 = {F_n}")
            print(f"  Decimal digits: {result['decimal_digits']}")
            print(f"  Binary length: {result['binary_length']} bits")
            print(f"  Is prime: {result.get('is_prime', 'Unknown')}")
            
            if 'small_factors' in result:
                factors = result['small_factors']
                print(f"  Factorization: {' × '.join(map(str, factors))}")
            
            print()
        else:
            print(f"F_{n} = 2^(2^{n}) + 1 = 2^{2**n} + 1")
            print(f"  This number has {2**n} bits and is too large to compute")
            print(f"  Status: Composite (first factor found by others)")
            print()

def fermat_prime_applications() -> None:
    """Demonstrate applications of Fermat primes"""
    print("=== Applications of Fermat Primes ===")
    
    # Constructible polygons
    print("Constructible regular polygons (sides ≤ 100):")
    constructible = constructible_polygon_sides(100)
    
    for sides in constructible:
        # Find the factorization
        temp = sides
        factors = []
        
        # Extract powers of 2
        k = 0
        while temp % 2 == 0:
            temp //= 2
            k += 1
        
        if k > 0:
            factors.append(f"2^{k}")
        
        # Extract Fermat primes
        for fp in [3, 17, 65537]:
            if temp % fp == 0:
                factors.append(str(fp))
                temp //= fp
        
        factorization = " × ".join(factors) if factors else "1"
        print(f"  {sides}-gon: {factorization}")
    
    print("\nFamous constructible polygons:")
    famous = {
        3: "Triangle (known since antiquity)",
        5: "Pentagon (known since antiquity)", 
        15: "15-gon (3 × 5)",
        17: "17-gon (constructed by Gauss, 1796)",
        257: "257-gon (first constructed by Richelot, 1832)",
        65537: "65537-gon (theoretically constructible)"
    }
    
    for sides, description in famous.items():
        if sides in constructible:
            print(f"  {sides}: {description}")

def historical_significance() -> None:
    """Discuss historical significance of Fermat primes"""
    print("=== Historical Significance ===")
    
    timeline = [
        (1640, "Pierre de Fermat", "Conjectured that all F_n are prime"),
        (1732, "Leonhard Euler", "Found that F_5 = 641 × 6700417 (composite)"),
        (1796, "Carl Friedrich Gauss", "Proved constructibility theorem for regular polygons"),
        (1832, "Friedrich Julius Richelot", "First to construct regular 257-gon"),
        (1975, "Morrison & Brillhart", "Factored F_6 completely"),
        (1980, "Brent & Pollard", "Factored F_7 completely"),
    ]
    
    for year, mathematician, achievement in timeline:
        print(f"  {year}: {mathematician} - {achievement}")

# Example usage and demonstrations
if __name__ == "__main__":
    print("=== Fermat Primes Analysis ===\n")
    
    # Verify known Fermat primes
    print("Verifying known Fermat primes:")
    for n in range(5):
        result = verify_fermat_prime(n)
        F_n = result["F_n"]
        is_prime = result.get("is_prime", False)
        
        print(f"F_{n} = {F_n}")
        print(f"  Digits: {result['decimal_digits']}")
        print(f"  Is prime: {is_prime}")
        
        if 'pepin_test' in result:
            print(f"  Pépin's test: {'PASS' if result['pepin_test'] else 'FAIL'}")
        
        if 'small_factors' in result:
            factors = result['small_factors']
            print(f"  Factors: {factors}")
            # Verify factorization
            product = 1
            for factor in factors:
                product *= factor
            print(f"  Verification: {product} == {F_n} ? {product == F_n}")
        
        print()
    
    # Demonstrate Pépin's test in detail
    print("=== Pépin's Test Demonstration ===")
    n = 2  # Test F_2 = 65537
    F_n = fermat_number(n)
    print(f"Testing F_{n} = {F_n} using Pépin's test:")
    print(f"Computing 3^{(F_n-1)//2} mod {F_n}...")
    
    result = pow(3, (F_n - 1) // 2, F_n)
    expected = F_n - 1  # -1 mod F_n
    
    print(f"Result: {result}")
    print(f"Expected: {expected} (≡ -1 mod {F_n})")
    print(f"Test result: {'PRIME' if result == expected else 'COMPOSITE'}")
    print()
    
    # Show the rapid growth of Fermat numbers
    print("=== Growth Rate of Fermat Numbers ===")
    print("The Fermat numbers grow extremely rapidly:")
    
    for n in range(8):
        if n <= 4:
            F_n = fermat_number(n)
            print(f"F_{n} = 2^{2**n} + 1 = {F_n}")
        else:
            bits = 2**n
            decimal_digits = int(bits * math.log10(2)) + 1
            print(f"F_{n} = 2^{bits} + 1")
            print(f"    ≈ {decimal_digits:,} decimal digits")
            print(f"    ≈ {bits:,} bits")
    
    print("\n" + "="*50)
    analyze_fermat_pattern()
    fermat_prime_applications()
    historical_significance()
```

## Historical Development

### Pierre de Fermat's Conjecture (1640)
Fermat originally conjectured that all numbers of the form $F_n = 2^{2^n} + 1$ are prime. This was one of the first major conjectures in number theory.

### Euler's Discovery (1732)
Leonhard Euler proved Fermat wrong by showing that:
```
F₅ = 2^32 + 1 = 4,294,967,297 = 641 × 6,700,417
```

Euler found this factorization using the observation that $641 = 5 \times 2^7 + 1$ and $641 = 5^4 + 2^4$.

### Gauss and Constructible Polygons (1796)
At age 19, Carl Friedrich Gauss proved the remarkable connection between Fermat primes and constructible regular polygons, revolutionizing both number theory and geometry.

## Current Status and Open Problems

### Known Composite Fermat Numbers
All Fermat numbers $F_n$ for $n = 5, 6, 7, 8, 9, 10, 11$ have been proven composite:

- $F_5 = 641 \times 6,700,417$
- $F_6 = 274,177 \times 67,280,421,310,721$
- $F_7$ has a 39-digit factor and a 252-digit factor
- $F_8$ has multiple known factors
- $F_9$ has a 49-digit factor
- $F_{10}$ has a 40-digit factor  
- $F_{11}$ has a 39-digit factor

### Fermat's Last Prime Conjecture
**Conjecture**: The only Fermat primes are $F_0 = 3$, $F_1 = 17$, $F_2 = 65,537$, $F_3$, and $F_4$.

**Evidence**: 
- No additional Fermat primes have been found despite extensive searching
- Heuristic arguments suggest the probability of finding more is extremely low
- The numbers grow so rapidly that verification becomes computationally infeasible

### Computational Challenges
- $F_5$ has 10 decimal digits
- $F_{10}$ has approximately $10^{308}$ decimal digits
- $F_{20}$ would have more digits than there are atoms in the observable universe

## Applications and Significance

### Cryptography
- **RSA Key Generation**: Small Fermat primes (especially 65537) are commonly used as public exponents
- **Discrete Fourier Transform**: Fast algorithms often use Fermat number moduli
- **Pseudorandom Generation**: Some generators exploit properties of Fermat numbers

### Computer Science
- **Number Theoretic Transforms**: Used in polynomial multiplication
- **Error-Correcting Codes**: Some codes are constructed using Fermat prime fields
- **Algorithm Analysis**: Fermat numbers appear in complexity analysis

### Pure Mathematics
- **Algebraic Number Theory**: Connection to cyclotomic fields
- **Galois Theory**: Regular polygon construction relates to solvable groups
- **Analytic Number Theory**: Distribution and growth rate studies

## Connections to Other Mathematical Objects

### Mersenne Primes Relationship
While both are special forms of primes, Fermat and Mersenne primes have opposite behaviors:
- **Mersenne**: $2^p - 1$ (subtraction), many known examples
- **Fermat**: $2^{2^n} + 1$ (addition), very few known examples

### Perfect Numbers
Unlike Mersenne primes, Fermat primes do not directly generate perfect numbers, but they have connections to other special number classes.

### Regular Polygons
The Gauss-Wantzel theorem creates a unique link between prime number theory and Euclidean geometry that exists nowhere else in mathematics.

## References

- Gauss, Carl Friedrich (1796). "Disquisitiones Arithmeticae"
- Hardy, G.H.; Wright, E.M. "An Introduction to the Theory of Numbers"
- Rosen, Kenneth H. "Elementary Number Theory and Its Applications"
- Crandall, Richard; Pomerance, Carl (2005). "Prime Numbers: A Computational Perspective"
- Weisstein, Eric W. "Fermat Prime" MathWorld--A Wolfram Web Resource