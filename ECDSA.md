# ECDSA (Elliptic Curve Digital Signature Algorithm)

## Overview

ECDSA is a Digital Signature Algorithm (DSA) that uses keys derived from Elliptic Curve Cryptography (ECC). It is a particularly efficient algorithm based on Public Key Cryptography (PKC), offering the same level of security as RSA with much smaller key sizes.

## Mathematical Foundation

### Elliptic Curves

An elliptic curve over a finite field 𝔽ₚ (where p is a prime) is defined by the equation:

```
y² = x³ + ax + b (mod p)
```

where $4a^3 + 27b^2 \not\equiv 0 \pmod{p}$ (to ensure the curve is non-singular).

### Point Addition

For two points $P = (x_1, y_1)$ and $Q = (x_2, y_2)$ on the elliptic curve:

**Case 1: Point Doubling** (when P = Q):
```
λ = (3x₁² + a) / (2y₁) (mod p)
```

**Case 2: Point Addition** (when P ≠ Q):
```
λ = (y₂ - y₁) / (x₂ - x₁) (mod p)
```

**Result** P + Q = (x₃, y₃):
```
x₃ = λ² - x₁ - x₂ (mod p)
y₃ = λ(x₁ - x₃) - y₁ (mod p)
```

### Scalar Multiplication

For a point $P$ and scalar $k$:
$$kP = P + P + \cdots + P \text{ (k times)}$$

This is efficiently computed using the "double-and-add" method.

## ECDSA Algorithm

### Key Generation

1. **Select domain parameters:**
   - Prime $p$ (field size)
   - Elliptic curve parameters $a$ and $b$
   - Base point $G = (x_G, y_G)$ of order $n$
   - Cofactor $h = \#E(\mathbb{F}_p)/n$

2. **Generate private key:**
   - Choose random integer $d$ where $1 \leq d \leq n-1$

3. **Compute public key:**
   - $Q = dG$ (point multiplication)

### Signature Generation

To sign message $m$:

1. **Compute message hash:** $e = \text{Hash}(m)$
2. **Convert to integer:** $z = $ leftmost bits of $e$
3. **Generate random nonce:** $k$ where $1 \leq k \leq n-1$
4. **Calculate point:** $(x_1, y_1) = kG$
5. **Compute $r$:** $r = x_1 \bmod n$ (if $r = 0$, choose new $k$)
6. **Compute $s$:** $s = k^{-1}(z + rd) \bmod n$ (if $s = 0$, choose new $k$)
7. **Signature:** $(r, s)$

### Signature Verification

To verify signature $(r, s)$ on message $m$:

1. **Verify range:** $1 \leq r, s \leq n-1$
2. **Compute message hash:** $e = \text{Hash}(m)$
3. **Convert to integer:** $z = $ leftmost bits of $e$
4. **Calculate:** $w = s^{-1} \bmod n$
5. **Calculate:** $u_1 = zw \bmod n$ and $u_2 = rw \bmod n$
6. **Calculate point:** $(x_1, y_1) = u_1G + u_2Q$
7. **Verify:** Signature is valid if $r \equiv x_1 \pmod{n}$

## Popular Elliptic Curves

### secp256k1 (used by Bitcoin)
- **Prime:** $p = 2^{256} - 2^{32} - 2^9 - 2^8 - 2^7 - 2^6 - 2^4 - 1$
- **Curve:** $y^2 = x^3 + 7$
- **Generator point:** $G = (x_G, y_G)$ where:
  - $x_G = $ `0x79BE667EF9DCBBAC55A06295CE870B07029BFCDB2DCE28D959F2815B16F81798`
  - $y_G = $ `0x483ADA7726A3C4655DA4FBFC0E1108A8FD17B448A68554199C47D08FFB10D4B8`

### secp256r1 / P-256 (NIST standard)
- **Prime:** $p = 2^{256} - 2^{224} + 2^{192} + 2^{96} - 1$
- **Curve:** $y^2 = x^3 - 3x + b$ where:
  - $b = $ `0x5AC635D8AA3A93E7B3EBBD55769886BC651D06B0CC53B0F63BCE3C3E27D2604B`

## Security Considerations

- **Key size comparison:** 256-bit ECDSA ≈ 3072-bit RSA security
- **Vulnerability to quantum computers:** Shor's algorithm affects both RSA and ECDSA
- **Nonce reuse attack:** Using same $k$ for different messages reveals private key
- **Side-channel attacks:** Timing and power analysis can leak information

## Python Implementation

```python
import hashlib
import secrets
from dataclasses import dataclass
from typing import Tuple, Optional

@dataclass
class EllipticCurve:
    """Elliptic curve parameters"""
    p: int  # Prime modulus
    a: int  # Curve parameter a
    b: int  # Curve parameter b
    g: Tuple[int, int]  # Generator point
    n: int  # Order of generator point

@dataclass
class Point:
    """Point on elliptic curve"""
    x: Optional[int]
    y: Optional[int]
    
    def is_infinity(self) -> bool:
        return self.x is None and self.y is None

# secp256k1 parameters (Bitcoin curve)
SECP256K1 = EllipticCurve(
    p=0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEFFFFFC2F,
    a=0,
    b=7,
    g=(0x79BE667EF9DCBBAC55A06295CE870B07029BFCDB2DCE28D959F2815B16F81798,
       0x483ADA7726A3C4655DA4FBFC0E1108A8FD17B448A68554199C47D08FFB10D4B8),
    n=0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141
)

def mod_inverse(a: int, m: int) -> int:
    """Calculate modular multiplicative inverse using Extended Euclidean Algorithm"""
    if a < 0:
        a = (a % m + m) % m
    
    # Extended Euclidean Algorithm
    def extended_gcd(a: int, b: int) -> Tuple[int, int, int]:
        if a == 0:
            return b, 0, 1
        gcd, x1, y1 = extended_gcd(b % a, a)
        x = y1 - (b // a) * x1
        y = x1
        return gcd, x, y
    
    gcd, x, _ = extended_gcd(a, m)
    if gcd != 1:
        raise ValueError("Modular inverse does not exist")
    
    return (x % m + m) % m

def point_add(p1: Point, p2: Point, curve: EllipticCurve) -> Point:
    """Add two points on elliptic curve"""
    # Handle point at infinity
    if p1.is_infinity():
        return p2
    if p2.is_infinity():
        return p1
    
    # Handle point doubling
    if p1.x == p2.x:
        if p1.y == p2.y:
            # Point doubling: P + P = 2P
            s = (3 * p1.x * p1.x + curve.a) * mod_inverse(2 * p1.y, curve.p) % curve.p
        else:
            # Points are inverses: P + (-P) = O
            return Point(None, None)
    else:
        # Point addition: P + Q
        s = (p2.y - p1.y) * mod_inverse(p2.x - p1.x, curve.p) % curve.p
    
    # Calculate new point
    x3 = (s * s - p1.x - p2.x) % curve.p
    y3 = (s * (p1.x - x3) - p1.y) % curve.p
    
    return Point(x3, y3)

def point_multiply(k: int, point: Point, curve: EllipticCurve) -> Point:
    """Scalar multiplication: k * P using double-and-add method"""
    if k == 0:
        return Point(None, None)  # Point at infinity
    if k == 1:
        return point
    
    result = Point(None, None)  # Start with point at infinity
    addend = point
    
    while k:
        if k & 1:  # If k is odd
            result = point_add(result, addend, curve)
        addend = point_add(addend, addend, curve)  # Double
        k >>= 1  # Divide by 2
    
    return result

def generate_keypair(curve: EllipticCurve = SECP256K1) -> Tuple[int, Point]:
    """Generate ECDSA key pair"""
    # Generate private key (random integer)
    private_key = secrets.randbelow(curve.n - 1) + 1
    
    # Generate public key (point multiplication)
    generator = Point(curve.g[0], curve.g[1])
    public_key = point_multiply(private_key, generator, curve)
    
    return private_key, public_key

def sign_message(message: bytes, private_key: int, curve: EllipticCurve = SECP256K1) -> Tuple[int, int]:
    """Sign message using ECDSA"""
    # Hash the message
    message_hash = hashlib.sha256(message).digest()
    z = int.from_bytes(message_hash, 'big')
    
    # Ensure z is within curve order
    if z.bit_length() > curve.n.bit_length():
        z = z >> (z.bit_length() - curve.n.bit_length())
    
    generator = Point(curve.g[0], curve.g[1])
    
    while True:
        # Generate random nonce k
        k = secrets.randbelow(curve.n - 1) + 1
        
        # Calculate r = (k * G).x mod n
        point_k = point_multiply(k, generator, curve)
        if point_k.is_infinity():
            continue
        
        r = point_k.x % curve.n
        if r == 0:
            continue
        
        # Calculate s = k^(-1) * (z + r * private_key) mod n
        k_inv = mod_inverse(k, curve.n)
        s = (k_inv * (z + r * private_key)) % curve.n
        if s == 0:
            continue
        
        return r, s

def verify_signature(message: bytes, signature: Tuple[int, int], public_key: Point, 
                    curve: EllipticCurve = SECP256K1) -> bool:
    """Verify ECDSA signature"""
    r, s = signature
    
    # Check signature bounds
    if not (1 <= r <= curve.n - 1) or not (1 <= s <= curve.n - 1):
        return False
    
    # Hash the message
    message_hash = hashlib.sha256(message).digest()
    z = int.from_bytes(message_hash, 'big')
    
    # Ensure z is within curve order
    if z.bit_length() > curve.n.bit_length():
        z = z >> (z.bit_length() - curve.n.bit_length())
    
    # Calculate w = s^(-1) mod n
    try:
        w = mod_inverse(s, curve.n)
    except ValueError:
        return False
    
    # Calculate u1 = z * w mod n and u2 = r * w mod n
    u1 = (z * w) % curve.n
    u2 = (r * w) % curve.n
    
    # Calculate point (x1, y1) = u1 * G + u2 * Q
    generator = Point(curve.g[0], curve.g[1])
    point1 = point_multiply(u1, generator, curve)
    point2 = point_multiply(u2, public_key, curve)
    point_sum = point_add(point1, point2, curve)
    
    if point_sum.is_infinity():
        return False
    
    # Verify that r ≡ x1 (mod n)
    return r == (point_sum.x % curve.n)

def point_to_bytes(point: Point, compressed: bool = True) -> bytes:
    """Convert point to bytes representation"""
    if point.is_infinity():
        return b'\x00'
    
    if compressed:
        # Compressed format: 0x02/0x03 + x-coordinate
        prefix = b'\x02' if point.y % 2 == 0 else b'\x03'
        return prefix + point.x.to_bytes(32, 'big')
    else:
        # Uncompressed format: 0x04 + x-coordinate + y-coordinate
        return b'\x04' + point.x.to_bytes(32, 'big') + point.y.to_bytes(32, 'big')

def bytes_to_point(data: bytes, curve: EllipticCurve = SECP256K1) -> Point:
    """Convert bytes to point representation"""
    if len(data) == 1 and data[0] == 0:
        return Point(None, None)  # Point at infinity
    
    if len(data) == 33 and data[0] in (0x02, 0x03):
        # Compressed format
        x = int.from_bytes(data[1:], 'big')
        
        # Calculate y from curve equation: y² = x³ + ax + b
        y_squared = (pow(x, 3, curve.p) + curve.a * x + curve.b) % curve.p
        y = pow(y_squared, (curve.p + 1) // 4, curve.p)  # Works for p ≡ 3 (mod 4)
        
        # Choose correct y based on parity
        if y % 2 != data[0] - 0x02:
            y = curve.p - y
        
        return Point(x, y)
    
    elif len(data) == 65 and data[0] == 0x04:
        # Uncompressed format
        x = int.from_bytes(data[1:33], 'big')
        y = int.from_bytes(data[33:], 'big')
        return Point(x, y)
    
    else:
        raise ValueError("Invalid point format")

# Example usage
if __name__ == "__main__":
    print("Generating ECDSA key pair...")
    private_key, public_key = generate_keypair()
    
    print(f"Private key: {hex(private_key)}")
    print(f"Public key: ({hex(public_key.x)}, {hex(public_key.y)})")
    
    # Sign a message
    message = b"Hello, ECDSA!"
    print(f"\nSigning message: {message}")
    
    signature = sign_message(message, private_key)
    print(f"Signature (r, s): ({hex(signature[0])}, {hex(signature[1])})")
    
    # Verify the signature
    is_valid = verify_signature(message, signature, public_key)
    print(f"Signature valid: {is_valid}")
    
    # Test with wrong message
    wrong_message = b"Hello, ECDSA modified!"
    is_valid_wrong = verify_signature(wrong_message, signature, public_key)
    print(f"Wrong message signature valid: {is_valid_wrong}")
    
    # Convert public key to compressed bytes
    pub_key_bytes = point_to_bytes(public_key, compressed=True)
    print(f"\nCompressed public key: {pub_key_bytes.hex()}")
    
    # Convert back to point
    restored_point = bytes_to_point(pub_key_bytes)
    print(f"Restored public key matches: {public_key.x == restored_point.x and public_key.y == restored_point.y}")
```

## EdDSA and Ed25519

### EdDSA (Edwards-curve Digital Signature Algorithm)

EdDSA is a modern signature scheme using twisted Edwards curves, designed to be faster and more secure than traditional ECDSA.

### Ed25519 Parameters

**Prime:** $q = 2^{255} - 19$

**Twisted Edwards Curve:** $-x^2 + y^2 = 1 - \frac{121665}{121666}x^2y^2$

**Advantages over ECDSA:**
- Deterministic signatures (no nonce required)
- Faster signature generation and verification
- Built-in protection against many side-channel attacks
- Smaller signatures (64 bytes vs 70+ bytes for ECDSA)

## Applications

- **Bitcoin and Cryptocurrencies:** secp256k1 curve
- **TLS/SSL:** P-256, P-384, P-521 curves
- **SSH:** Ed25519 for modern implementations
- **Signal Protocol:** Curve25519 for key agreement
- **Smart Cards:** Smaller curves for resource-constrained devices

## References

- [EdDSA - Ed25519 - Wikipedia](https://en.wikipedia.org/wiki/EdDSA#Ed25519)
- [Elliptic Curve Cryptography - Wikipedia](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography)
- RFC 6979: Deterministic Usage of the Digital Signature Algorithm (DSA) and Elliptic Curve Digital Signature Algorithm (ECDSA)
- RFC 8032: Edwards-Curve Digital Signature Algorithm (EdDSA)
- [SEC 2: Recommended Elliptic Curve Domain Parameters](https://www.secg.org/sec2-v2.pdf)