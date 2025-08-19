# DSA (Digital Signature Algorithm)

## Overview

DSA is a Federal Information Processing Standard for digital signatures, based on the mathematical concept of modular exponentiation and the discrete logarithm problem. DSA was proposed by the National Institute of Standards and Technology (NIST) in 1991 and became a standard in 1994.

## Mathematical Foundation

### Parameters

DSA uses the following parameters:
- **p**: A prime modulus of length L bits (where L is a multiple of 64 between 512 and 1024)
- **q**: A prime divisor of (p-1) of length 160 bits
- **g**: A generator of order q modulo p, where $1 < g < p$
- **h**: An arbitrary integer satisfying $1 < h < p-1$ such that $g = h^{(p-1)/q} \bmod p$

### Key Generation

1. **Choose domain parameters** $(p, q, g)$
2. **Choose private key** $x$ randomly from $\{1, 2, ..., q-1\}$
3. **Calculate public key** $y$:
   $$y = g^x \bmod p$$

**Public Key:** $(p, q, g, y)$  
**Private Key:** $(p, q, g, x)$

### Digital Signature Generation

To sign a message $m$:

1. **Choose random** $k$ from $\{1, 2, ..., q-1\}$
2. **Calculate** $r$:
   $$r = (g^k \bmod p) \bmod q$$
3. **Calculate** $s$:
   $$s = k^{-1}(H(m) + xr) \bmod q$$
   where $H(m)$ is the hash of message $m$

**Signature:** $(r, s)$

### Digital Signature Verification

To verify signature $(r, s)$ on message $m$:

1. **Verify** $0 < r < q$ and $0 < s < q$
2. **Calculate** $w$:
   $$w = s^{-1} \bmod q$$
3. **Calculate** $u_1$ and $u_2$:
   $$u_1 = H(m) \cdot w \bmod q$$
   $$u_2 = r \cdot w \bmod q$$
4. **Calculate** $v$:
   $$v = ((g^{u_1} \cdot y^{u_2}) \bmod p) \bmod q$$
5. **Accept if** $v = r$

## Security Considerations

- DSA security relies on the discrete logarithm problem
- The random value $k$ must be kept secret and never reused
- Modern implementations typically use SHA-256 or stronger hash functions
- Key sizes of 2048-bit $p$ and 256-bit $q$ are recommended for new applications

## Python Implementation

```python
import hashlib
import random
import math

def gcd(a, b):
    """Calculate Greatest Common Divisor using Euclidean algorithm"""
    while b:
        a, b = b, a % b
    return a

def mod_inverse(a, m):
    """Calculate modular multiplicative inverse using Extended Euclidean Algorithm"""
    if gcd(a, m) != 1:
        return None
    
    def extended_gcd(a, b):
        if a == 0:
            return b, 0, 1
        gcd_val, x1, y1 = extended_gcd(b % a, a)
        x = y1 - (b // a) * x1
        y = x1
        return gcd_val, x, y
    
    _, x, _ = extended_gcd(a, m)
    return (x % m + m) % m

def is_prime(n, k=10):
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

def generate_prime(bits):
    """Generate a random prime number of specified bit length"""
    while True:
        candidate = random.getrandbits(bits)
        candidate |= (1 << bits - 1) | 1
        
        if is_prime(candidate):
            return candidate

def generate_dsa_parameters(L=1024, N=160):
    """Generate DSA domain parameters (p, q, g)"""
    # Generate q (N-bit prime)
    q = generate_prime(N)
    
    # Generate p (L-bit prime such that q divides p-1)
    while True:
        # Generate random L-bit number
        p = random.getrandbits(L)
        p |= (1 << L - 1) | 1  # Ensure L bits and odd
        
        # Adjust p so that p-1 is divisible by q
        remainder = (p - 1) % q
        p = p - remainder
        
        if p.bit_length() == L and is_prime(p):
            break
    
    # Find generator g
    h = 2
    while True:
        g = pow(h, (p - 1) // q, p)
        if g > 1:
            break
        h += 1
    
    return p, q, g

def generate_dsa_keypair(p, q, g):
    """Generate DSA key pair"""
    # Private key: random integer in [1, q-1]
    x = random.randrange(1, q)
    
    # Public key: y = g^x mod p
    y = pow(g, x, p)
    
    return (p, q, g, y), (p, q, g, x)  # public_key, private_key

def dsa_sign(message, private_key, hash_func=hashlib.sha256):
    """Sign message using DSA private key"""
    p, q, g, x = private_key
    
    # Hash the message
    if isinstance(message, str):
        message = message.encode()
    h = int.from_bytes(hash_func(message).digest(), 'big') % q
    
    while True:
        # Choose random k
        k = random.randrange(1, q)
        
        # Calculate r
        r = pow(g, k, p) % q
        if r == 0:
            continue
        
        # Calculate s
        k_inv = mod_inverse(k, q)
        if k_inv is None:
            continue
            
        s = (k_inv * (h + x * r)) % q
        if s == 0:
            continue
            
        return r, s

def dsa_verify(message, signature, public_key, hash_func=hashlib.sha256):
    """Verify DSA signature"""
    p, q, g, y = public_key
    r, s = signature
    
    # Check signature bounds
    if not (0 < r < q and 0 < s < q):
        return False
    
    # Hash the message
    if isinstance(message, str):
        message = message.encode()
    h = int.from_bytes(hash_func(message).digest(), 'big') % q
    
    # Calculate w
    w = mod_inverse(s, q)
    if w is None:
        return False
    
    # Calculate u1 and u2
    u1 = (h * w) % q
    u2 = (r * w) % q
    
    # Calculate v
    v = ((pow(g, u1, p) * pow(y, u2, p)) % p) % q
    
    return v == r

# Example usage
if __name__ == "__main__":
    print("Generating DSA parameters...")
    p, q, g = generate_dsa_parameters(1024, 160)
    print(f"p: {p}")
    print(f"q: {q}")
    print(f"g: {g}")
    
    print("\nGenerating DSA key pair...")
    public_key, private_key = generate_dsa_keypair(p, q, g)
    
    # Sign a message
    message = "Hello, DSA!"
    print(f"\nMessage: {message}")
    
    signature = dsa_sign(message, private_key)
    print(f"Signature (r, s): {signature}")
    
    # Verify signature
    is_valid = dsa_verify(message, signature, public_key)
    print(f"Signature valid: {is_valid}")
    
    # Test with tampered message
    tampered_message = "Hello, DSA modified!"
    is_valid_tampered = dsa_verify(tampered_message, signature, public_key)
    print(f"Tampered message signature valid: {is_valid_tampered}")
```

## Advantages and Disadvantages

### Advantages
- Smaller signature size compared to RSA
- Well-studied and standardized algorithm
- Good performance for signature generation

### Disadvantages
- Cannot be used for encryption (signatures only)
- Requires good random number generation
- Vulnerable to poor implementation of random k selection

## References

- FIPS 186-4: Digital Signature Standard (DSS)
- [DSA Algorithm - Wikipedia](https://en.wikipedia.org/wiki/Digital_Signature_Algorithm)
- RFC 6979: Deterministic Usage of DSA and ECDSA