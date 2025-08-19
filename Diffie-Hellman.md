# Diffie-Hellman Key Exchange

## Overview

The Diffie-Hellman key exchange is a method for two parties to securely establish a shared secret key over an insecure communication channel. Published in 1976 by Whitfield Diffie and Martin Hellman, it was one of the first practical examples of public key cryptography. The security of Diffie-Hellman relies on the discrete logarithm problem.

## Mathematical Foundation

### Parameters

The Diffie-Hellman protocol uses:
- **p**: A large prime number (modulus)
- **g**: A primitive root modulo p (generator)
- **a**: Alice's private key (random integer)
- **b**: Bob's private key (random integer)

### Key Exchange Process

1. **Public Parameters**: Alice and Bob agree on public values $(p, g)$

2. **Private Key Generation**:
   - Alice chooses random private key $a$ where $1 < a < p-1$
   - Bob chooses random private key $b$ where $1 < b < p-1$

3. **Public Key Calculation**:
   - Alice calculates her public key: $A = g^a \bmod p$
   - Bob calculates his public key: $B = g^b \bmod p$

4. **Public Key Exchange**: Alice and Bob exchange their public keys $A$ and $B$

5. **Shared Secret Calculation**:
   - Alice computes: $K = B^a \bmod p = (g^b)^a \bmod p = g^{ab} \bmod p$
   - Bob computes: $K = A^b \bmod p = (g^a)^b \bmod p = g^{ab} \bmod p$

Both parties now share the same secret key $K = g^{ab} \bmod p$.

### Security

The security of Diffie-Hellman relies on the **Discrete Logarithm Problem**: given $g$, $p$, and $g^x \bmod p$, it is computationally difficult to find $x$.

An eavesdropper knows $p$, $g$, $A = g^a \bmod p$, and $B = g^b \bmod p$, but cannot efficiently compute $g^{ab} \bmod p$ without knowing $a$ or $b$.

## Elliptic Curve Diffie-Hellman (ECDH)

ECDH uses elliptic curve cryptography for better security with smaller key sizes:

### Elliptic Curve Parameters
- **E**: Elliptic curve equation $y^2 = x^3 + ax + b \bmod p$
- **G**: Base point (generator) on the curve
- **n**: Order of point G

### ECDH Process
1. Alice chooses private key $d_A$ and computes public key $Q_A = d_A \cdot G$
2. Bob chooses private key $d_B$ and computes public key $Q_B = d_B \cdot G$
3. Shared secret: $K = d_A \cdot Q_B = d_B \cdot Q_A = d_A \cdot d_B \cdot G$

## Security Considerations

- Use sufficiently large prime p (at least 2048 bits for classic DH, 256 bits for ECDH)
- Ensure g is a primitive root modulo p
- Private keys must be truly random and kept secret
- Vulnerable to man-in-the-middle attacks without authentication
- Use ephemeral keys (DHE/ECDHE) for forward secrecy

## Python Implementation

```python
import random
import hashlib
from typing import Tuple

def gcd(a, b):
    """Calculate Greatest Common Divisor using Euclidean algorithm"""
    while b:
        a, b = b, a % b
    return a

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

def find_primitive_root(p):
    """Find a primitive root modulo prime p"""
    if not is_prime(p):
        return None
    
    # Find prime factors of p-1
    phi = p - 1
    factors = []
    n = phi
    
    # Find all prime factors of phi
    d = 2
    while d * d <= n:
        while n % d == 0:
            factors.append(d)
            n //= d
        d += 1
    if n > 1:
        factors.append(n)
    
    # Remove duplicates
    factors = list(set(factors))
    
    # Test candidates for primitive root
    for candidate in range(2, p):
        is_primitive = True
        for factor in factors:
            if pow(candidate, phi // factor, p) == 1:
                is_primitive = False
                break
        
        if is_primitive:
            return candidate
    
    return None

def generate_dh_parameters(bits=1024):
    """Generate Diffie-Hellman parameters (p, g)"""
    p = generate_prime(bits)
    g = find_primitive_root(p)
    if g is None:
        return generate_dh_parameters(bits)  # Try again
    return p, g

class DiffieHellman:
    """Diffie-Hellman key exchange implementation"""
    
    def __init__(self, p=None, g=None, bits=1024):
        if p is None or g is None:
            self.p, self.g = generate_dh_parameters(bits)
        else:
            self.p = p
            self.g = g
        
        # Generate private key
        self.private_key = random.randrange(2, self.p - 1)
        
        # Calculate public key
        self.public_key = pow(self.g, self.private_key, self.p)
    
    def get_public_key(self):
        """Return public key"""
        return self.public_key
    
    def get_shared_secret(self, other_public_key):
        """Calculate shared secret using other party's public key"""
        return pow(other_public_key, self.private_key, self.p)
    
    def get_parameters(self):
        """Return DH parameters (p, g)"""
        return self.p, self.g

# Elliptic Curve Point class for ECDH
class ECPoint:
    """Elliptic curve point for ECDH implementation"""
    
    def __init__(self, x, y, a, b, p):
        self.x = x
        self.y = y
        self.a = a  # curve parameter
        self.b = b  # curve parameter
        self.p = p  # prime modulus
        self.is_infinity = (x is None and y is None)
    
    def __add__(self, other):
        """Point addition on elliptic curve"""
        if self.is_infinity:
            return other
        if other.is_infinity:
            return self
        
        if self.x == other.x:
            if self.y == other.y:
                # Point doubling
                s = (3 * self.x * self.x + self.a) * pow(2 * self.y, self.p - 2, self.p) % self.p
            else:
                # Points are inverses
                return ECPoint(None, None, self.a, self.b, self.p)
        else:
            # Regular addition
            s = (other.y - self.y) * pow(other.x - self.x, self.p - 2, self.p) % self.p
        
        x3 = (s * s - self.x - other.x) % self.p
        y3 = (s * (self.x - x3) - self.y) % self.p
        
        return ECPoint(x3, y3, self.a, self.b, self.p)
    
    def __mul__(self, scalar):
        """Scalar multiplication using double-and-add"""
        if scalar == 0:
            return ECPoint(None, None, self.a, self.b, self.p)
        
        result = ECPoint(None, None, self.a, self.b, self.p)  # Point at infinity
        addend = self
        
        while scalar:
            if scalar & 1:
                result = result + addend
            addend = addend + addend
            scalar >>= 1
        
        return result
    
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

class ECDH:
    """Elliptic Curve Diffie-Hellman implementation using secp256k1-like curve"""
    
    def __init__(self):
        # Using simplified curve parameters (not secp256k1)
        self.p = 2**256 - 2**32 - 977  # Prime modulus
        self.a = 0  # Curve parameter
        self.b = 7  # Curve parameter
        
        # Generator point (simplified, not actual secp256k1 generator)
        self.g = ECPoint(
            55066263022277343669578718895168534326250603453777594175500187360389116729240,
            32670510020758816978083085130507043184471273380659243275938904335757337482424,
            self.a, self.b, self.p
        )
        
        # Generate private key
        self.private_key = random.randrange(1, self.p)
        
        # Calculate public key
        self.public_key = self.g * self.private_key
    
    def get_public_key(self):
        """Return public key point"""
        return self.public_key
    
    def get_shared_secret(self, other_public_key):
        """Calculate shared secret using other party's public key"""
        shared_point = other_public_key * self.private_key
        # Use x-coordinate as shared secret
        return shared_point.x

# Example usage and demonstration
if __name__ == "__main__":
    print("=== Classic Diffie-Hellman Key Exchange ===")
    
    # Generate common parameters
    print("Generating DH parameters...")
    p, g = generate_dh_parameters(512)  # Small size for demo
    print(f"Prime p: {p}")
    print(f"Generator g: {g}")
    
    # Alice's side
    alice = DiffieHellman(p, g)
    print(f"\nAlice's private key: {alice.private_key}")
    print(f"Alice's public key: {alice.public_key}")
    
    # Bob's side
    bob = DiffieHellman(p, g)
    print(f"\nBob's private key: {bob.private_key}")
    print(f"Bob's public key: {bob.public_key}")
    
    # Key exchange
    alice_shared = alice.get_shared_secret(bob.get_public_key())
    bob_shared = bob.get_shared_secret(alice.get_public_key())
    
    print(f"\nAlice computed shared secret: {alice_shared}")
    print(f"Bob computed shared secret: {bob_shared}")
    print(f"Shared secrets match: {alice_shared == bob_shared}")
    
    print("\n=== Elliptic Curve Diffie-Hellman (ECDH) ===")
    
    # Alice's ECDH
    alice_ecdh = ECDH()
    print(f"Alice's private key: {alice_ecdh.private_key}")
    print(f"Alice's public key: ({alice_ecdh.public_key.x}, {alice_ecdh.public_key.y})")
    
    # Bob's ECDH
    bob_ecdh = ECDH()
    print(f"\nBob's private key: {bob_ecdh.private_key}")
    print(f"Bob's public key: ({bob_ecdh.public_key.x}, {bob_ecdh.public_key.y})")
    
    # ECDH key exchange
    alice_ecdh_shared = alice_ecdh.get_shared_secret(bob_ecdh.get_public_key())
    bob_ecdh_shared = bob_ecdh.get_shared_secret(alice_ecdh.get_public_key())
    
    print(f"\nAlice computed ECDH shared secret: {alice_ecdh_shared}")
    print(f"Bob computed ECDH shared secret: {bob_ecdh_shared}")
    print(f"ECDH shared secrets match: {alice_ecdh_shared == bob_ecdh_shared}")
    
    # Demonstrate key derivation
    if alice_shared == bob_shared:
        # Derive AES key from shared secret
        shared_bytes = alice_shared.to_bytes((alice_shared.bit_length() + 7) // 8, 'big')
        aes_key = hashlib.sha256(shared_bytes).digest()
        print(f"\nDerived AES key (hex): {aes_key.hex()}")
```

## Applications

- **TLS/SSL**: Used in HTTPS for establishing secure connections
- **SSH**: Secure shell protocol for remote access
- **VPN**: Virtual private network protocols
- **Signal Protocol**: End-to-end encrypted messaging
- **Cryptocurrency**: Bitcoin and other blockchain protocols

## Variants

- **DHE (Diffie-Hellman Ephemeral)**: Uses temporary keys for forward secrecy
- **ECDHE (Elliptic Curve DHE)**: Elliptic curve version with ephemeral keys
- **X25519**: Modern elliptic curve (Curve25519) for key exchange
- **X448**: Another modern elliptic curve option

## References

- Diffie, W.; Hellman, M. (1976). "New directions in cryptography"
- [Diffie-Hellman Key Exchange - Wikipedia](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)
- RFC 2631: Diffie-Hellman Key Agreement Method
- RFC 7748: Elliptic Curves for Security