# Mersenne Primes

## Overview

Mersenne primes are prime numbers that can be expressed in the form $M_p = 2^p - 1$, where $p$ is also a prime number. Named after French mathematician Marin Mersenne (1588-1648), these primes have fascinated mathematicians for centuries and play important roles in number theory, cryptography, and computer science.

## Mathematical Foundation

### Definition

A **Mersenne number** is a positive integer of the form:
```
Mₚ = 2^p - 1
```

where $p$ is a positive integer.

A **Mersenne prime** is a Mersenne number that is also prime. For $M_p$ to be prime, $p$ must itself be prime (though not all prime $p$ yield prime $M_p$).

### Fundamental Theorem

**Theorem**: If $M_n = 2^n - 1$ is prime, then $n$ must be prime.

**Proof**: If n = ab where a > 1 and b > 1, then:
```
2^n - 1 = 2^(ab) - 1 = (2^a)^b - 1
```

Since $(2^a)^b - 1$ is divisible by $2^a - 1$ (by the factorization $x^b - 1 = (x-1)(x^{b-1} + x^{b-2} + \cdots + 1)$), we have that $2^n - 1$ is composite.

### Lucas-Lehmer Test

The **Lucas-Lehmer test** is the most efficient known primality test specifically for Mersenne numbers.

**Algorithm**: For an odd prime $p$, $M_p = 2^p - 1$ is prime if and only if $M_p$ divides $S_{p-2}$, where:

**Lucas-Lehmer Sequence:**

For i = 0:
```
S₀ = 4
```

For i > 0:
```
Sᵢ = Sᵢ₋₁² - 2
```

**Mathematical Foundation**: The sequence Sᵢ is related to the recurrence:
```
Sᵢ = ((2 + √3)^(2^i) + (2 - √3)^(2^i)) / 2
```

### Perfect Numbers Connection

**Euclid-Euler Theorem**: An even number is perfect if and only if it has the form:
```
2^(p-1)(2^p - 1)
```

where $2^p - 1$ is a Mersenne prime.

Examples:
- $p = 2$: $M_2 = 3$, perfect number = $2^1 \cdot 3 = 6$
- $p = 3$: $M_3 = 7$, perfect number = $2^2 \cdot 7 = 28$
- $p = 5$: $M_5 = 31$, perfect number = $2^4 \cdot 31 = 496$

## Known Mersenne Primes

As of 2024, only 51 Mersenne primes are known. The first few are:

| $p$ | $M_p = 2^p - 1$ | Decimal Value | Discovery Year |
|-----|-----------------|---------------|----------------|
| 2   | $2^2 - 1 = 3$  | 3             | Ancient        |
| 3   | $2^3 - 1 = 7$  | 7             | Ancient        |
| 5   | $2^5 - 1 = 31$ | 31            | Ancient        |
| 7   | $2^7 - 1 = 127$| 127           | Ancient        |
| 13  | $2^{13} - 1$   | 8191          | 1456           |
| 17  | $2^{17} - 1$   | 131071        | 1588           |
| 19  | $2^{19} - 1$   | 524287        | 1588           |

The largest known Mersenne prime (as of 2023) is $2^{82589933} - 1$, discovered in 2018.

## Properties and Characteristics

### Divisibility Properties
- If $p$ is an odd prime, any prime factor $q$ of $M_p = 2^p - 1$ must satisfy $q \equiv 1 \pmod{8}$ or $q \equiv 7 \pmod{8}$
- Additionally, $q \equiv 1 \pmod{p}$

### Distribution
- Mersenne primes become increasingly rare as $p$ increases
- The probability that a random Mersenne number $M_p$ is prime is approximately $\frac{e^\gamma \log 2}{\log p} \approx \frac{1.78}{\log p}$

### Computational Importance
- Used in pseudorandom number generators (Mersenne Twister)
- Largest known primes are typically Mersenne primes due to efficient testing
- Important in distributed computing projects like GIMPS

## Python Implementation

```python
import math
from typing import List, Tuple, Optional

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
    import random
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

def lucas_lehmer_test(p: int) -> bool:
    """
    Lucas-Lehmer test for Mersenne prime M_p = 2^p - 1
    Returns True if M_p is prime, False otherwise
    """
    if p == 2:
        return True  # M_2 = 3 is prime
    
    if not is_prime_basic(p):
        return False  # p must be prime for M_p to potentially be prime
    
    # Calculate M_p = 2^p - 1
    M_p = (1 << p) - 1  # Efficient calculation of 2^p - 1
    
    # Lucas-Lehmer sequence
    S = 4
    for _ in range(p - 2):
        S = (S * S - 2) % M_p
    
    return S == 0

def find_mersenne_primes(max_p: int) -> List[Tuple[int, int]]:
    """
    Find all Mersenne primes M_p = 2^p - 1 where p <= max_p
    Returns list of tuples (p, M_p)
    """
    mersenne_primes = []
    
    # Check p = 2 separately (special case)
    if max_p >= 2:
        if lucas_lehmer_test(2):
            mersenne_primes.append((2, 3))
    
    # Check odd primes
    for p in range(3, max_p + 1, 2):
        if is_prime_basic(p):
            print(f"Testing p = {p}...")
            if lucas_lehmer_test(p):
                M_p = (1 << p) - 1
                mersenne_primes.append((p, M_p))
                print(f"Found Mersenne prime: M_{p} = {M_p}")
    
    return mersenne_primes

def mersenne_factorization_attempt(p: int, max_trials: int = 1000) -> Optional[List[int]]:
    """
    Attempt to find small factors of M_p = 2^p - 1
    Returns list of factors if composite, None if no small factors found
    """
    if not is_prime_basic(p):
        return None
    
    M_p = (1 << p) - 1
    factors = []
    
    # Check for small prime factors
    # Any prime factor q of M_p must satisfy specific congruence conditions
    for q in range(3, max_trials + 1, 2):
        if not is_prime_basic(q):
            continue
        
        # Check if q satisfies the necessary conditions
        if q % 8 not in [1, 7]:  # q ≡ 1 or 7 (mod 8)
            continue
        if q % p != 1:  # q ≡ 1 (mod p)
            continue
        
        if M_p % q == 0:
            factors.append(q)
            while M_p % q == 0:
                M_p //= q
            
            if M_p == 1:
                break
    
    if M_p > 1:
        factors.append(M_p)
    
    return factors if len(factors) > 1 else None

def perfect_number_from_mersenne(p: int) -> Optional[int]:
    """
    Generate perfect number from Mersenne prime M_p
    Returns 2^(p-1) * (2^p - 1) if M_p is prime, None otherwise
    """
    if lucas_lehmer_test(p):
        M_p = (1 << p) - 1
        return (1 << (p - 1)) * M_p
    return None

def mersenne_prime_properties(p: int) -> dict:
    """
    Analyze properties of Mersenne number M_p
    """
    if not is_prime_basic(p):
        return {"error": "p must be prime"}
    
    M_p = (1 << p) - 1
    is_mersenne_prime = lucas_lehmer_test(p)
    
    properties = {
        "p": p,
        "M_p": M_p,
        "is_prime": is_mersenne_prime,
        "decimal_digits": len(str(M_p)),
        "binary_representation": bin(M_p),
        "hex_representation": hex(M_p)
    }
    
    if is_mersenne_prime:
        perfect_num = perfect_number_from_mersenne(p)
        properties["perfect_number"] = perfect_num
        properties["perfect_number_digits"] = len(str(perfect_num)) if perfect_num else 0
    else:
        factors = mersenne_factorization_attempt(p)
        if factors:
            properties["small_factors"] = factors
    
    return properties

# Sieve-based approach for finding candidate primes
def sieve_of_eratosthenes(limit: int) -> List[int]:
    """Generate all primes up to limit using Sieve of Eratosthenes"""
    if limit < 2:
        return []
    
    sieve = [True] * (limit + 1)
    sieve[0] = sieve[1] = False
    
    for i in range(2, int(math.sqrt(limit)) + 1):
        if sieve[i]:
            for j in range(i * i, limit + 1, i):
                sieve[j] = False
    
    return [i for i in range(2, limit + 1) if sieve[i]]

def efficient_mersenne_search(max_p: int) -> List[dict]:
    """
    Efficient search for Mersenne primes using sieve for candidates
    """
    print(f"Searching for Mersenne primes up to p = {max_p}")
    
    # Get all prime candidates using sieve
    prime_candidates = sieve_of_eratosthenes(max_p)
    
    mersenne_primes = []
    
    for p in prime_candidates:
        print(f"Testing M_{p} = 2^{p} - 1...")
        
        if lucas_lehmer_test(p):
            properties = mersenne_prime_properties(p)
            mersenne_primes.append(properties)
            print(f"✓ M_{p} is prime! ({properties['decimal_digits']} digits)")
        else:
            print(f"✗ M_{p} is composite")
    
    return mersenne_primes

# Example usage and demonstration
if __name__ == "__main__":
    print("=== Mersenne Primes Finder ===\n")
    
    # Find small Mersenne primes
    print("Finding Mersenne primes for small values of p:")
    small_mersenne = efficient_mersenne_search(25)
    
    print(f"\nFound {len(small_mersenne)} Mersenne primes:")
    for mp in small_mersenne:
        p = mp['p']
        M_p = mp['M_p']
        digits = mp['decimal_digits']
        print(f"M_{p} = {M_p} ({digits} digits)")
        
        if 'perfect_number' in mp:
            perfect = mp['perfect_number']
            perfect_digits = mp['perfect_number_digits']
            print(f"  Corresponding perfect number: {perfect} ({perfect_digits} digits)")
        print()
    
    # Demonstrate Lucas-Lehmer test steps
    print("=== Lucas-Lehmer Test Demonstration ===")
    p = 5  # Test M_5 = 31
    print(f"Testing M_{p} = 2^{p} - 1 = {(1 << p) - 1}")
    
    M_p = (1 << p) - 1
    S = 4
    print(f"S_0 = {S}")
    
    for i in range(1, p - 1):
        S = (S * S - 2) % M_p
        print(f"S_{i} = {S}")
    
    print(f"Final result: S_{p-2} = {S}")
    print(f"M_{p} is {'prime' if S == 0 else 'composite'}")
    
    # Analyze a composite Mersenne number
    print("\n=== Composite Mersenne Number Analysis ===")
    p_composite = 11  # M_11 = 2047 = 23 × 89
    properties = mersenne_prime_properties(p_composite)
    
    print(f"M_{p_composite} = {properties['M_p']}")
    print(f"Is prime: {properties['is_prime']}")
    
    if 'small_factors' in properties:
        factors = properties['small_factors']
        print(f"Factorization: {' × '.join(map(str, factors))}")
        
        # Verify factorization
        product = 1
        for factor in factors:
            product *= factor
        print(f"Verification: {product} == {properties['M_p']} ? {product == properties['M_p']}")
    
    # Show progression of perfect numbers
    print("\n=== Perfect Numbers from Mersenne Primes ===")
    for mp in small_mersenne[:6]:  # First 6 Mersenne primes
        p = mp['p']
        if 'perfect_number' in mp:
            perfect = mp['perfect_number']
            print(f"M_{p} = {mp['M_p']} → Perfect number: {perfect}")
    
    # Demonstrate efficiency of Lucas-Lehmer test
    print("\n=== Lucas-Lehmer Test Efficiency ===")
    import time
    
    test_values = [13, 17, 19, 23, 29, 31]
    
    for p in test_values:
        M_p = (1 << p) - 1
        
        # Time Lucas-Lehmer test
        start_time = time.time()
        is_prime_ll = lucas_lehmer_test(p)
        ll_time = time.time() - start_time
        
        # Time general primality test (for comparison)
        start_time = time.time()
        is_prime_mr = miller_rabin(M_p, 10)
        mr_time = time.time() - start_time
        
        print(f"M_{p} ({len(str(M_p))} digits):")
        print(f"  Lucas-Lehmer: {is_prime_ll} ({ll_time:.6f}s)")
        print(f"  Miller-Rabin: {is_prime_mr} ({mr_time:.6f}s)")
        print(f"  Speedup: {mr_time/ll_time:.2f}x faster")
```

## Applications and Significance

### Cryptography
- **RSA Key Generation**: Large Mersenne primes can be used as factors
- **Pseudorandom Generators**: Mersenne Twister algorithm uses Mersenne prime modulus
- **Discrete Logarithm**: Security of some systems relies on difficulty in Mersenne prime fields

### Computational Mathematics
- **Benchmarking**: Testing computational limits of hardware
- **Distributed Computing**: GIMPS (Great Internet Mersenne Prime Search)
- **Algorithm Testing**: Verification of primality testing algorithms

### Mathematical Research
- **Number Theory**: Connection to perfect numbers and amicable numbers
- **Analytic Number Theory**: Distribution and density questions
- **Computational Complexity**: Efficiency of specialized algorithms

## Open Problems and Conjectures

### Infinitude Conjecture
**Question**: Are there infinitely many Mersenne primes?

**Status**: Open problem. Believed to be true but unproven.

### New Mersenne Conjecture
**Conjecture**: If two odd numbers $m$ and $n$ are such that $m < n$ and both $2^m - 1$ and $2^n - 1$ are prime, then $n/m < 2$ eventually fails.

### Catalan's Mersenne Conjecture
**Conjecture**: The only solution to $2^p - 1 = x^q$ in integers $p > 1$, $q > 1$, $x > 1$ is $p = 3$, $q = 2$, $x = 3$ (giving $2^3 - 1 = 7$ and $3^2 = 9$, but $7 \neq 9$).

## Records and GIMPS Project

### Great Internet Mersenne Prime Search (GIMPS)
- Distributed computing project founded in 1996
- Volunteers contribute CPU time to search for new Mersenne primes
- Has discovered all Mersenne primes with more than 300,000 digits

### Current Records (2024)
- **Largest known prime**: $2^{82,589,933} - 1$ (24,862,048 digits)
- **Discovery date**: December 21, 2018
- **Discoverer**: Patrick Laroche (GIMPS participant)

## References

- Lucas, Édouard (1878). "Théorie des nombres"
- Lehmer, Derrick Henry (1930). "An extended theory of Lucas' functions"
- GIMPS Project: https://www.mersenne.org/
- Knuth, Donald E. "The Art of Computer Programming, Volume 2"
- Crandall, Richard; Pomerance, Carl (2005). "Prime Numbers: A Computational Perspective"