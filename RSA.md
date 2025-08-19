# RSA (Rivest–Shamir–Adleman) Algorithm

## Overview

RSA is a public-key cryptographic algorithm that uses prime numbers and modular arithmetic for encryption and digital signatures. The security of RSA relies on the practical difficulty of factoring the product of two large prime numbers, known as the "factoring problem".

## Mathematical Foundation

### Key Generation

1. **Choose two prime numbers** $p$ and $q$, then calculate:
   $$n = p \times q$$

2. **Calculate Euler's totient function:**
   $$\phi(n) = (p-1)(q-1)$$

3. **Choose an integer $e$** such that:
   - $1 < e < \phi(n)$
   - $\gcd(e, \phi(n)) = 1$ (e is coprime to $\phi(n)$)

4. **Calculate the private key $d$** such that:
   $$d \times e \equiv 1 \pmod{\phi(n)}$$
   
   This can be computed using the Extended Euclidean Algorithm.

### Encryption and Decryption

**Public Key:** $(n, e)$  
**Private Key:** $(n, d)$

**Encryption:** For message $m$ where $0 \leq m < n$:
$$c = m^e \pmod{n}$$

**Decryption:** For ciphertext $c$:
$$m = c^d \pmod{n}$$

### Digital Signatures

**Signing:** For message $m$:
$$s = m^d \pmod{n}$$

**Verification:** Check if:
$$m \equiv s^e \pmod{n}$$

## Security Considerations

- Modern RSA typically uses key sizes of 2048 or 4096 bits
- The difficulty of factoring large composite numbers is the foundation of RSA security
- No efficient classical algorithms exist for factoring large numbers (quantum computers pose a future threat)

## Prime Number Generation Algorithms

RSA relies on finding large prime numbers. Common algorithms include:

- **Miller-Rabin primality test** - probabilistic primality testing
- **Sieve of Eratosthenes** - systematic prime finding for smaller numbers
- **Baillie-PSW primality test** - deterministic for practical ranges
- **Fermat primality test** - simple but less reliable
- **Mersenne primes** - primes of the form $2^p - 1$

## Visualization

![RSA Algorithm Visualization](resources/RSA_Algorithm.png)

## Python Implementation

```python
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
    
    # Extended Euclidean Algorithm
    def extended_gcd(a, b):
        if a == 0:
            return b, 0, 1
        gcd, x1, y1 = extended_gcd(b % a, a)
        x = y1 - (b // a) * x1
        y = x1
        return gcd, x, y
    
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
        # Ensure odd number and proper bit length
        candidate |= (1 << bits - 1) | 1
        
        if is_prime(candidate):
            return candidate

def generate_keypair(bits=1024):
    """Generate RSA public/private key pair"""
    # Generate two distinct prime numbers
    p = generate_prime(bits // 2)
    q = generate_prime(bits // 2)
    while p == q:
        q = generate_prime(bits // 2)
    
    # Calculate n and phi(n)
    n = p * q
    phi_n = (p - 1) * (q - 1)
    
    # Choose e (commonly 65537)
    e = 65537
    while gcd(e, phi_n) != 1:
        e += 2
    
    # Calculate d (private exponent)
    d = mod_inverse(e, phi_n)
    
    # Return public and private keys
    public_key = (n, e)
    private_key = (n, d)
    
    return public_key, private_key

def encrypt(message, public_key):
    """Encrypt message using RSA public key"""
    n, e = public_key
    # Convert message to integer (for demo purposes)
    if isinstance(message, str):
        message_int = int.from_bytes(message.encode(), 'big')
    else:
        message_int = message
    
    if message_int >= n:
        raise ValueError("Message too large for key size")
    
    ciphertext = pow(message_int, e, n)
    return ciphertext

def decrypt(ciphertext, private_key):
    """Decrypt ciphertext using RSA private key"""
    n, d = private_key
    message_int = pow(ciphertext, d, n)
    return message_int

def sign(message, private_key):
    """Sign message using RSA private key"""
    n, d = private_key
    if isinstance(message, str):
        message_int = int.from_bytes(message.encode(), 'big')
    else:
        message_int = message
    
    if message_int >= n:
        raise ValueError("Message too large for key size")
    
    signature = pow(message_int, d, n)
    return signature

def verify(message, signature, public_key):
    """Verify signature using RSA public key"""
    n, e = public_key
    if isinstance(message, str):
        message_int = int.from_bytes(message.encode(), 'big')
    else:
        message_int = message
    
    verified_message = pow(signature, e, n)
    return verified_message == message_int

# Example usage
if __name__ == "__main__":
    print("Generating RSA key pair...")
    public_key, private_key = generate_keypair(1024)
    
    print(f"Public key (n, e): ({public_key[0]}, {public_key[1]})")
    print(f"Private key (n, d): ({private_key[0]}, {private_key[1]})")
    
    # Encryption/Decryption example
    message = 12345
    print(f"\nOriginal message: {message}")
    
    encrypted = encrypt(message, public_key)
    print(f"Encrypted: {encrypted}")
    
    decrypted = decrypt(encrypted, private_key)
    print(f"Decrypted: {decrypted}")
    
    # Digital signature example
    signature = sign(message, private_key)
    print(f"\nSignature: {signature}")
    
    is_valid = verify(message, signature, public_key)
    print(f"Signature valid: {is_valid}")
```

## Common Questions

**Q: Is it possible to pre-calculate all prime numbers and save them in a rainbow table for fast lookup?**

**A:** No, this is not feasible. Prime numbers are randomly selected during key pair generation. Modern RSA uses primes that are 2^1024 or 2^2048 bits in size, making it computationally impossible to find and store all such prime numbers (there isn't enough storage capacity in existence to hold them all).

## References

- [RSA Algorithm Explained - YouTube](https://www.youtube.com/watch?v=qph77bTKJTM)
- [Public-key cryptography - Wikipedia](https://en.wikipedia.org/wiki/Public-key_cryptography)
- RFC 8017: PKCS #1: RSA Cryptography Specifications Version 2.2