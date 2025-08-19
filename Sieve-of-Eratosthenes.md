# Sieve of Eratosthenes

## Overview

The Sieve of Eratosthenes is one of the most efficient algorithms for finding all prime numbers up to a given limit. Named after the ancient Greek mathematician Eratosthenes of Cyrene (c. 276-194 BCE), this algorithm systematically eliminates composite numbers by marking multiples of each prime, leaving only the primes unmarked. It remains the gold standard for generating lists of primes and forms the foundation for many modern primality testing algorithms.

## Mathematical Foundation

### Algorithm Principle

The sieve works on the fundamental principle that **every composite number has a prime factor less than or equal to its square root**.

**Key Insight**: If $n$ is composite, then $n = a \times b$ where $a \leq \sqrt{n}$ and $b \geq \sqrt{n}$. Therefore, every composite number $n$ will be marked when we process the prime $a \leq \sqrt{n}$.

### Algorithm Description

**Input**: An integer $n \geq 2$ (upper limit)

**Output**: All prime numbers $p$ where $2 \leq p \leq n$

**Steps**:

1. **Initialize**: Create a boolean array $A[0..n]$ and set $A[i] = \text{true}$ for all $i \in [2, n]$
2. **Mark composites**: For each $i$ from $2$ to $\lfloor\sqrt{n}\rfloor$:
   - If $A[i] = \text{true}$ (i.e., $i$ is prime):
     - For each $j = i^2, i^2 + i, i^2 + 2i, \ldots$ while $j \leq n$:
       - Set $A[j] = \text{false}$
3. **Collect primes**: Return all indices $i$ where $A[i] = \text{true}$

### Mathematical Correctness

**Theorem**: The Sieve of Eratosthenes correctly identifies all primes up to $n$.

**Proof**:
- **No false positives**: If $A[i] = \text{true}$ at the end, then $i$ was never marked as a multiple of any prime $p < i$, so $i$ is prime.
- **No false negatives**: If $i$ is prime, it will never be marked since it has no prime factors other than itself.
- **All composites marked**: Every composite $i$ has a smallest prime factor $p \leq \sqrt{i}$, so $i$ will be marked when processing $p$.

## Complexity Analysis

### Time Complexity

The time complexity is $O(n \log \log n)$.

**Derivation**: The number of operations is proportional to:
$$\sum_{p \text{ prime}, p \leq \sqrt{n}} \frac{n}{p} = n \sum_{p \leq \sqrt{n}} \frac{1}{p}$$

By the **Prime Number Theorem**, this sum is asymptotically $O(\log \log n)$.

### Space Complexity

- **Basic version**: $O(n)$ space for the boolean array
- **Optimized version**: $O(n/2)$ by storing only odd numbers
- **Segmented version**: $O(\sqrt{n})$ space complexity

## Algorithm Variants

### Optimized Sieve (Skip Even Numbers)

Since all even numbers except 2 are composite, we can optimize by:
1. Handle 2 separately
2. Only sieve odd numbers from 3 onwards
3. Use array of size $n/2$ instead of $n$

### Segmented Sieve

For very large $n$, memory becomes a constraint. The segmented sieve processes the range $[2, n]$ in segments of size $O(\sqrt{n})$.

**Benefits**:
- Reduced memory usage: $O(\sqrt{n})$ instead of $O(n)$
- Better cache locality
- Can handle arbitrarily large ranges

### Wheel Factorization

Further optimize by skipping multiples of small primes (2, 3, 5, etc.):
- **2-wheel**: Skip even numbers (50% reduction)
- **2,3-wheel**: Skip multiples of 2 and 3 (66.7% reduction)  
- **2,3,5-wheel**: Skip multiples of 2, 3, and 5 (73.3% reduction)

## Python Implementation

```python
import math
import bisect
from typing import List, Iterator, Tuple

def sieve_of_eratosthenes(limit: int) -> List[int]:
    """
    Basic Sieve of Eratosthenes algorithm
    Returns list of all primes up to limit (inclusive)
    """
    if limit < 2:
        return []
    
    # Initialize sieve array - True means potentially prime
    is_prime = [True] * (limit + 1)
    is_prime[0] = is_prime[1] = False  # 0 and 1 are not prime
    
    # Sieve process
    for i in range(2, int(math.sqrt(limit)) + 1):
        if is_prime[i]:
            # Mark multiples of i starting from i^2
            for j in range(i * i, limit + 1, i):
                is_prime[j] = False
    
    # Collect primes
    return [i for i in range(2, limit + 1) if is_prime[i]]

def optimized_sieve(limit: int) -> List[int]:
    """
    Optimized sieve that skips even numbers
    More memory efficient and faster
    """
    if limit < 2:
        return []
    if limit == 2:
        return [2]
    
    # Handle 2 separately, then work with odd numbers only
    primes = [2]
    
    # Array for odd numbers: index i represents number 2*i + 3
    # So index 0 -> 3, index 1 -> 5, index 2 -> 7, etc.
    odd_limit = (limit - 3) // 2 + 1
    is_prime = [True] * odd_limit
    
    # Sieve odd numbers
    for i in range(int(math.sqrt(limit)) // 2):
        if is_prime[i]:
            prime = 2 * i + 3
            # Mark multiples starting from prime^2
            start = (prime * prime - 3) // 2
            for j in range(start, odd_limit, prime):
                is_prime[j] = False
    
    # Collect odd primes
    primes.extend([2 * i + 3 for i in range(odd_limit) if is_prime[i]])
    
    return primes

def segmented_sieve(limit: int) -> List[int]:
    """
    Segmented sieve for large limits
    Uses O(sqrt(n)) memory instead of O(n)
    """
    if limit < 2:
        return []
    
    segment_size = max(int(math.sqrt(limit)), 1000)
    
    # First find all primes up to sqrt(limit)
    sqrt_limit = int(math.sqrt(limit))
    small_primes = sieve_of_eratosthenes(sqrt_limit)
    
    all_primes = []
    
    # Process segments
    for segment_start in range(0, limit + 1, segment_size):
        segment_end = min(segment_start + segment_size - 1, limit)
        
        if segment_start == 0:
            # First segment: use small primes directly
            all_primes.extend([p for p in small_primes if p <= segment_end])
        else:
            # Initialize segment
            segment = [True] * (segment_end - segment_start + 1)
            
            # Mark multiples of each small prime
            for prime in small_primes:
                # Find first multiple of prime in this segment
                start = max(prime * prime, ((segment_start + prime - 1) // prime) * prime)
                
                # Mark multiples
                for j in range(start, segment_end + 1, prime):
                    segment[j - segment_start] = False
            
            # Collect primes from this segment
            for i, is_prime in enumerate(segment):
                if is_prime and segment_start + i > 1:
                    all_primes.append(segment_start + i)
    
    return all_primes

def wheel_sieve_2_3(limit: int) -> List[int]:
    """
    Wheel factorization sieve using 2,3-wheel
    Skips numbers divisible by 2 or 3
    """
    if limit < 2:
        return []
    
    primes = []
    if limit >= 2:
        primes.append(2)
    if limit >= 3:
        primes.append(3)
    
    if limit < 5:
        return primes
    
    # 2,3-wheel pattern: numbers of form 6k±1
    # Pattern repeats every 6: [1, 5] relative to 6k
    wheel = [4, 2]  # differences: 5-1=4, 7-5=2, 11-7=4, 13-11=2, ...
    
    # Generate candidates using wheel
    candidates = []
    candidate = 5  # First candidate after 2, 3
    wheel_index = 0
    
    while candidate <= limit:
        candidates.append(candidate)
        candidate += wheel[wheel_index]
        wheel_index = (wheel_index + 1) % 2
    
    # Sieve the candidates
    is_prime = [True] * len(candidates)
    
    for i, candidate in enumerate(candidates):
        if is_prime[i] and candidate * candidate <= limit:
            # Mark multiples
            for j in range(i, len(candidates)):
                if candidates[j] % candidate == 0 and j != i:
                    is_prime[j] = False
    
    # Collect wheel-sieved primes
    primes.extend([candidates[i] for i in range(len(candidates)) if is_prime[i]])
    
    return sorted(primes)

def sieve_with_statistics(limit: int) -> dict:
    """
    Sieve with detailed statistics about the process
    """
    if limit < 2:
        return {"primes": [], "stats": {"limit": limit, "prime_count": 0}}
    
    is_prime = [True] * (limit + 1)
    is_prime[0] = is_prime[1] = False
    
    stats = {
        "limit": limit,
        "sqrt_limit": int(math.sqrt(limit)),
        "operations": 0,
        "primes_used": [],
        "composites_marked": 0
    }
    
    # Sieve with statistics
    for i in range(2, int(math.sqrt(limit)) + 1):
        if is_prime[i]:
            stats["primes_used"].append(i)
            marked_count = 0
            
            for j in range(i * i, limit + 1, i):
                if is_prime[j]:
                    is_prime[j] = False
                    marked_count += 1
                stats["operations"] += 1
            
            stats["composites_marked"] += marked_count
    
    primes = [i for i in range(2, limit + 1) if is_prime[i]]
    
    stats["prime_count"] = len(primes)
    stats["composite_count"] = limit - 1 - len(primes)  # excluding 1
    stats["efficiency"] = stats["composites_marked"] / max(stats["operations"], 1)
    
    return {"primes": primes, "stats": stats}

def prime_counting_function(limit: int) -> int:
    """
    Count primes up to limit using sieve (π(x) function)
    """
    if limit < 2:
        return 0
    
    is_prime = [True] * (limit + 1)
    is_prime[0] = is_prime[1] = False
    
    for i in range(2, int(math.sqrt(limit)) + 1):
        if is_prime[i]:
            for j in range(i * i, limit + 1, i):
                is_prime[j] = False
    
    return sum(is_prime)

def nth_prime(n: int) -> int:
    """
    Find the nth prime number using sieve
    Uses prime number theorem for initial estimate
    """
    if n < 1:
        raise ValueError("n must be positive")
    if n == 1:
        return 2
    
    # Estimate upper bound using prime number theorem
    # π(x) ≈ x / ln(x), so x ≈ n * ln(n) for the nth prime
    if n < 6:
        estimate = 15
    else:
        estimate = int(n * (math.log(n) + math.log(math.log(n))))
    
    # Generate primes and find nth
    while True:
        primes = sieve_of_eratosthenes(estimate)
        if len(primes) >= n:
            return primes[n - 1]
        estimate *= 2  # Double the estimate and try again

def prime_gaps(limit: int) -> List[Tuple[int, int, int]]:
    """
    Find prime gaps up to limit
    Returns list of (prime1, prime2, gap) tuples
    """
    primes = sieve_of_eratosthenes(limit)
    gaps = []
    
    for i in range(len(primes) - 1):
        gap = primes[i + 1] - primes[i]
        gaps.append((primes[i], primes[i + 1], gap))
    
    return gaps

def twin_primes(limit: int) -> List[Tuple[int, int]]:
    """
    Find all twin primes up to limit
    Twin primes are pairs (p, p+2) where both are prime
    """
    primes_set = set(sieve_of_eratosthenes(limit))
    twin_primes_list = []
    
    for p in primes_set:
        if p + 2 in primes_set:
            twin_primes_list.append((p, p + 2))
    
    return sorted(twin_primes_list)

def goldbach_verification(limit: int) -> dict:
    """
    Verify Goldbach's conjecture for even numbers up to limit
    Every even number > 2 can be expressed as sum of two primes
    """
    primes = sieve_of_eratosthenes(limit)
    prime_set = set(primes)
    
    results = {
        "verified_count": 0,
        "total_even_numbers": 0,
        "counterexamples": [],
        "examples": {}
    }
    
    for n in range(4, limit + 1, 2):  # Even numbers starting from 4
        results["total_even_numbers"] += 1
        found_pair = False
        
        for p in primes:
            if p > n // 2:
                break
            if (n - p) in prime_set:
                results["verified_count"] += 1
                results["examples"][n] = (p, n - p)
                found_pair = True
                break
        
        if not found_pair:
            results["counterexamples"].append(n)
    
    results["success_rate"] = results["verified_count"] / max(results["total_even_numbers"], 1)
    
    return results

def benchmark_sieves(limit: int) -> dict:
    """
    Benchmark different sieve implementations
    """
    import time
    
    algorithms = [
        ("Basic Sieve", sieve_of_eratosthenes),
        ("Optimized Sieve", optimized_sieve),
        ("Segmented Sieve", segmented_sieve),
        ("Wheel Sieve (2,3)", wheel_sieve_2_3)
    ]
    
    results = {}
    
    for name, algorithm in algorithms:
        start_time = time.time()
        try:
            primes = algorithm(limit)
            end_time = time.time()
            
            results[name] = {
                "time": end_time - start_time,
                "prime_count": len(primes),
                "success": True,
                "memory_efficient": "Segmented" in name
            }
        except Exception as e:
            results[name] = {
                "error": str(e),
                "success": False
            }
    
    return results

# Example usage and demonstrations
if __name__ == "__main__":
    print("=== Sieve of Eratosthenes Demonstration ===\n")
    
    # Basic sieve demonstration
    limit = 100
    print(f"Finding all primes up to {limit}:")
    primes = sieve_of_eratosthenes(limit)
    print(f"Found {len(primes)} primes: {primes}\n")
    
    # Statistics demonstration
    print("=== Sieve Statistics ===")
    result = sieve_with_statistics(100)
    stats = result["stats"]
    
    print(f"Limit: {stats['limit']}")
    print(f"Primes found: {stats['prime_count']}")
    print(f"Composites found: {stats['composite_count']}")
    print(f"Operations performed: {stats['operations']}")
    print(f"Primes used for sieving: {stats['primes_used']}")
    print(f"Efficiency: {stats['efficiency']:.2%}\n")
    
    # Prime counting function
    print("=== Prime Counting Function π(x) ===")
    test_values = [10, 100, 1000, 10000]
    
    for x in test_values:
        count = prime_counting_function(x)
        ratio = x / count if count > 0 else float('inf')
        print(f"π({x:>5}) = {count:>4} (ratio x/π(x) = {ratio:.2f})")
    print()
    
    # Nth prime
    print("=== Finding Nth Prime ===")
    for n in [1, 10, 100, 1000]:
        prime = nth_prime(n)
        print(f"The {n}{'st' if n==1 else 'th'} prime is {prime}")
    print()
    
    # Prime gaps
    print("=== Prime Gaps Analysis ===")
    gaps = prime_gaps(100)
    gap_counts = {}
    max_gap = 0
    
    for p1, p2, gap in gaps:
        gap_counts[gap] = gap_counts.get(gap, 0) + 1
        max_gap = max(max_gap, gap)
    
    print("Gap distribution:")
    for gap in sorted(gap_counts.keys()):
        print(f"  Gap {gap}: {gap_counts[gap]} occurrences")
    
    print(f"Largest gap up to 100: {max_gap}\n")
    
    # Twin primes
    print("=== Twin Primes up to 100 ===")
    twins = twin_primes(100)
    print(f"Found {len(twins)} twin prime pairs:")
    for p1, p2 in twins:
        print(f"  ({p1}, {p2})")
    print()
    
    # Goldbach verification
    print("=== Goldbach Conjecture Verification ===")
    goldbach = goldbach_verification(100)
    
    print(f"Even numbers tested: {goldbach['total_even_numbers']}")
    print(f"Successfully expressed as sum of two primes: {goldbach['verified_count']}")
    print(f"Success rate: {goldbach['success_rate']:.2%}")
    
    if goldbach['counterexamples']:
        print(f"Counterexamples: {goldbach['counterexamples']}")
    else:
        print("No counterexamples found!")
    
    print("\nSome examples:")
    for n in sorted(list(goldbach['examples'].keys())[:10]):
        p1, p2 = goldbach['examples'][n]
        print(f"  {n} = {p1} + {p2}")
    print()
    
    # Performance benchmark
    print("=== Performance Benchmark ===")
    benchmark_limit = 100000
    benchmark_results = benchmark_sieves(benchmark_limit)
    
    print(f"Benchmarking sieves up to {benchmark_limit}:")
    print(f"{'Algorithm':<20} {'Time (s)':<10} {'Primes':<8} {'Status'}")
    print("-" * 50)
    
    for name, result in benchmark_results.items():
        if result["success"]:
            time_str = f"{result['time']:.4f}"
            count_str = str(result['prime_count'])
            status = "✓"
            if result.get('memory_efficient', False):
                status += " (Memory Efficient)"
        else:
            time_str = "N/A"
            count_str = "N/A"
            status = f"✗ {result['error']}"
        
        print(f"{name:<20} {time_str:<10} {count_str:<8} {status}")
```

## Applications

### Prime Number Generation
- **Cryptography**: Generate prime candidates for RSA key generation
- **Number Theory Research**: Study prime distribution patterns
- **Mathematical Computing**: Provide prime lists for various algorithms

### Algorithm Preprocessing
Many algorithms benefit from precomputed prime lists:
- **Prime factorization**: Trial division with sieved primes
- **Primality testing**: Preliminary divisibility checks
- **Number theoretic functions**: Euler's totient, Möbius function

### Mathematical Investigations
- **Prime gaps**: Study distribution of gaps between consecutive primes
- **Twin primes**: Find pairs of primes differing by 2
- **Goldbach conjecture**: Verify that even numbers are sums of two primes
- **Prime counting**: Estimate π(x) function behavior

## Optimizations and Variants

### Memory Optimizations
1. **Bit packing**: Store 8 boolean values per byte
2. **Odd-only storage**: Skip even numbers entirely
3. **Segmented processing**: Process in chunks to fit in cache

### Computational Optimizations
1. **Wheel factorization**: Skip multiples of small primes
2. **Multiple prime marking**: Mark multiples of several primes simultaneously
3. **Parallel processing**: Distribute segments across multiple cores

### Modern Variants
1. **Sieve of Sundaram**: Alternative approach using different elimination pattern
2. **Sieve of Atkin**: More complex but asymptotically faster for very large limits
3. **Incremental sieves**: Generate primes on-demand without storing entire array

## Comparison with Other Methods

| Method | Time Complexity | Space Complexity | Best Use Case |
|--------|----------------|------------------|---------------|
| Trial Division | O(n√n) | O(1) | Single prime tests |
| Sieve of Eratosthenes | O(n log log n) | O(n) | Multiple primes up to n |
| Segmented Sieve | O(n log log n) | O(√n) | Large limits, memory constrained |
| Miller-Rabin | O(k log³ n) | O(1) | Large single numbers |
| Wheel Sieve | O(n log log n) | O(n) | Dense prime generation |

## Historical Context

- **Ancient Greece (3rd century BCE)**: Eratosthenes develops the algorithm
- **1934**: Derrick Norman Lehmer creates mechanical sieve machines
- **1959**: First computer implementations appear
- **1970s-1980s**: Segmented and wheel factorization variants developed
- **Present**: Foundation for modern prime generation in cryptographic libraries

## References

- Eratosthenes of Cyrene (c. 276-194 BCE). Original algorithm concept
- Knuth, Donald E. "The Art of Computer Programming, Volume 2"  
- Crandall, Richard; Pomerance, Carl (2005). "Prime Numbers: A Computational Perspective"
- Rosen, Kenneth H. "Elementary Number Theory and Its Applications"
- Sorenson, Jonathan; Webster, Jonathan (2015). "Strong pseudoprimes to twelve prime bases"