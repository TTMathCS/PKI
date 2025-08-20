# SHA (Secure Hash Algorithm)

## Overview

SHA is a family of cryptographic hash functions designed by the National Security Agency (NSA) and published by the National Institute of Standards and Technology (NIST). SHA functions produce fixed-size hash values from variable-length input data and are widely used in digital signatures, message authentication codes, and other security applications.

## SHA Family Variants

- **SHA-0**: Original version (1993) - withdrawn due to vulnerabilities
- **SHA-1**: 160-bit hash (1995) - deprecated due to collision attacks
- **SHA-2**: Family including SHA-224, SHA-256, SHA-384, SHA-512 (2001)
- **SHA-3**: Latest standard (2015) - based on Keccak algorithm

## Mathematical Foundation

### SHA-256 Algorithm

SHA-256 is the most commonly used variant, producing a 256-bit hash value.

#### Constants and Initial Values

**Initial hash values** (first 32 bits of fractional parts of square roots of first 8 primes):
$$H_0 = \text{6a09e667}, H_1 = \text{bb67ae85}, H_2 = \text{3c6ef372}, H_3 = \text{a54ff53a}$$
$$H_4 = \text{510e527f}, H_5 = \text{9b05688c}, H_6 = \text{1f83d9ab}, H_7 = \text{5be0cd19}$$

**Round constants** $K_t$ (first 32 bits of fractional parts of cube roots of first 64 primes):
$$K_0 = \text{428a2f98}, K_1 = \text{71374491}, ..., K_{63} = \text{c67178f2}$$

#### Preprocessing

1. **Padding**: Append bit '1' followed by zeros, then 64-bit message length
2. **Message length**: Must be ≡ 448 (mod 512) after padding

#### Processing

For each 512-bit message block:

1. **Message schedule**: Expand 16 words to 64 words
   **Message Schedule Formula:**
   
   For $0 \leq t \leq 15$:
   $$W_t = M_t$$
   
   For $16 \leq t \leq 63$:
   $$W_t = \sigma_1(W_{t-2}) + W_{t-7} + \sigma_0(W_{t-15}) + W_{t-16}$$

2. **Logical functions**:
   - $\text{Ch}(x,y,z) = (x \land y) \oplus (\lnot x \land z)$
   - $\text{Maj}(x,y,z) = (x \land y) \oplus (x \land z) \oplus (y \land z)$
   - $\Sigma_0(x) = \text{ROTR}^2(x) \oplus \text{ROTR}^{13}(x) \oplus \text{ROTR}^{22}(x)$
   - $\Sigma_1(x) = \text{ROTR}^6(x) \oplus \text{ROTR}^{11}(x) \oplus \text{ROTR}^{25}(x)$
   - $\sigma_0(x) = \text{ROTR}^7(x) \oplus \text{ROTR}^{18}(x) \oplus \text{SHR}^3(x)$
   - $\sigma_1(x) = \text{ROTR}^{17}(x) \oplus \text{ROTR}^{19}(x) \oplus \text{SHR}^{10}(x)$

3. **Main loop** (64 rounds):
   $$T_1 = h + \Sigma_1(e) + \text{Ch}(e,f,g) + K_t + W_t$$
   $$T_2 = \Sigma_0(a) + \text{Maj}(a,b,c)$$
   $$h = g, g = f, f = e, e = d + T_1$$
   $$d = c, c = b, b = a, a = T_1 + T_2$$

## Security Properties

- **One-way function**: Computationally infeasible to find input from hash
- **Collision resistance**: Hard to find two inputs with same hash
- **Avalanche effect**: Small input change causes large output change
- **Deterministic**: Same input always produces same hash
- **Fixed output size**: Regardless of input length

## Python Implementation

```python
import struct
from typing import List, Union

def _rotr(value: int, amount: int, width: int = 32) -> int:
    """Rotate right"""
    return ((value >> amount) | (value << (width - amount))) & ((1 << width) - 1)

def _shr(value: int, amount: int) -> int:
    """Shift right"""
    return value >> amount

def _ch(x: int, y: int, z: int) -> int:
    """Choice function"""
    return (x & y) ^ (~x & z)

def _maj(x: int, y: int, z: int) -> int:
    """Majority function"""
    return (x & y) ^ (x & z) ^ (y & z)

def _sigma0(x: int) -> int:
    """SHA-256 sigma0 function"""
    return _rotr(x, 7) ^ _rotr(x, 18) ^ _shr(x, 3)

def _sigma1(x: int) -> int:
    """SHA-256 sigma1 function"""
    return _rotr(x, 17) ^ _rotr(x, 19) ^ _shr(x, 10)

def _big_sigma0(x: int) -> int:
    """SHA-256 big sigma0 function"""
    return _rotr(x, 2) ^ _rotr(x, 13) ^ _rotr(x, 22)

def _big_sigma1(x: int) -> int:
    """SHA-256 big sigma1 function"""
    return _rotr(x, 6) ^ _rotr(x, 11) ^ _rotr(x, 25)

class SHA256:
    """SHA-256 hash implementation"""
    
    def __init__(self):
        # Initial hash values (first 32 bits of fractional parts of square roots of first 8 primes)
        self._h = [
            0x6a09e667, 0xbb67ae85, 0x3c6ef372, 0xa54ff53a,
            0x510e527f, 0x9b05688c, 0x1f83d9ab, 0x5be0cd19
        ]
        
        # Round constants (first 32 bits of fractional parts of cube roots of first 64 primes)
        self._k = [
            0x428a2f98, 0x71374491, 0xb5c0fbcf, 0xe9b5dba5, 0x3956c25b, 0x59f111f1, 0x923f82a4, 0xab1c5ed5,
            0xd807aa98, 0x12835b01, 0x243185be, 0x550c7dc3, 0x72be5d74, 0x80deb1fe, 0x9bdc06a7, 0xc19bf174,
            0xe49b69c1, 0xefbe4786, 0x0fc19dc6, 0x240ca1cc, 0x2de92c6f, 0x4a7484aa, 0x5cb0a9dc, 0x76f988da,
            0x983e5152, 0xa831c66d, 0xb00327c8, 0xbf597fc7, 0xc6e00bf3, 0xd5a79147, 0x06ca6351, 0x14292967,
            0x27b70a85, 0x2e1b2138, 0x4d2c6dfc, 0x53380d13, 0x650a7354, 0x766a0abb, 0x81c2c92e, 0x92722c85,
            0xa2bfe8a1, 0xa81a664b, 0xc24b8b70, 0xc76c51a3, 0xd192e819, 0xd6990624, 0xf40e3585, 0x106aa070,
            0x19a4c116, 0x1e376c08, 0x2748774c, 0x34b0bcb5, 0x391c0cb3, 0x4ed8aa4a, 0x5b9cca4f, 0x682e6ff3,
            0x748f82ee, 0x78a5636f, 0x84c87814, 0x8cc70208, 0x90befffa, 0xa4506ceb, 0xbef9a3f7, 0xc67178f2
        ]
        
        self._buffer = b''
        self._message_length = 0
    
    def _pad_message(self, message: bytes) -> bytes:
        """Pad message according to SHA-256 specification"""
        msg_len = len(message)
        message += b'\x80'  # Append '1' bit
        
        # Pad with zeros until length ≡ 448 (mod 512)
        while (len(message) % 64) != 56:
            message += b'\x00'
        
        # Append original length in bits as 64-bit big-endian integer
        message += struct.pack('>Q', msg_len * 8)
        
        return message
    
    def _process_block(self, block: bytes):
        """Process a 512-bit block"""
        # Break chunk into sixteen 32-bit words
        w = list(struct.unpack('>16I', block))
        
        # Extend the sixteen 32-bit words into sixty-four 32-bit words
        for i in range(16, 64):
            s0 = _sigma0(w[i-15])
            s1 = _sigma1(w[i-2])
            w.append((w[i-16] + s0 + w[i-7] + s1) & 0xffffffff)
        
        # Initialize working variables
        a, b, c, d, e, f, g, h = self._h
        
        # Main loop
        for i in range(64):
            S1 = _big_sigma1(e)
            ch = _ch(e, f, g)
            temp1 = (h + S1 + ch + self._k[i] + w[i]) & 0xffffffff
            S0 = _big_sigma0(a)
            maj = _maj(a, b, c)
            temp2 = (S0 + maj) & 0xffffffff
            
            h = g
            g = f
            f = e
            e = (d + temp1) & 0xffffffff
            d = c
            c = b
            b = a
            a = (temp1 + temp2) & 0xffffffff
        
        # Add this chunk's hash to result so far
        self._h[0] = (self._h[0] + a) & 0xffffffff
        self._h[1] = (self._h[1] + b) & 0xffffffff
        self._h[2] = (self._h[2] + c) & 0xffffffff
        self._h[3] = (self._h[3] + d) & 0xffffffff
        self._h[4] = (self._h[4] + e) & 0xffffffff
        self._h[5] = (self._h[5] + f) & 0xffffffff
        self._h[6] = (self._h[6] + g) & 0xffffffff
        self._h[7] = (self._h[7] + h) & 0xffffffff
    
    def update(self, data: bytes):
        """Update hash with new data"""
        self._buffer += data
        self._message_length += len(data)
        
        # Process complete 512-bit blocks
        while len(self._buffer) >= 64:
            self._process_block(self._buffer[:64])
            self._buffer = self._buffer[64:]
    
    def digest(self) -> bytes:
        """Get final hash digest"""
        # Pad the remaining buffer
        padded = self._pad_message(self._buffer)
        
        # Process remaining blocks
        for i in range(0, len(padded), 64):
            self._process_block(padded[i:i+64])
        
        # Return final hash as bytes
        return struct.pack('>8I', *self._h)
    
    def hexdigest(self) -> str:
        """Get final hash digest as hex string"""
        return self.digest().hex()

class SHA1:
    """SHA-1 hash implementation (deprecated - for educational purposes only)"""
    
    def __init__(self):
        # Initial hash values
        self._h = [0x67452301, 0xEFCDAB89, 0x98BADCFE, 0x10325476, 0xC3D2E1F0]
        self._buffer = b''
        self._message_length = 0
    
    def _f(self, t: int, b: int, c: int, d: int) -> int:
        """SHA-1 f function"""
        if 0 <= t <= 19:
            return (b & c) | (~b & d)
        elif 20 <= t <= 39:
            return b ^ c ^ d
        elif 40 <= t <= 59:
            return (b & c) | (b & d) | (c & d)
        else:  # 60 <= t <= 79
            return b ^ c ^ d
    
    def _k(self, t: int) -> int:
        """SHA-1 K constant"""
        if 0 <= t <= 19:
            return 0x5A827999
        elif 20 <= t <= 39:
            return 0x6ED9EBA1
        elif 40 <= t <= 59:
            return 0x8F1BBCDC
        else:  # 60 <= t <= 79
            return 0xCA62C1D6
    
    def _pad_message(self, message: bytes) -> bytes:
        """Pad message according to SHA-1 specification"""
        msg_len = len(message)
        message += b'\x80'
        
        while (len(message) % 64) != 56:
            message += b'\x00'
        
        message += struct.pack('>Q', msg_len * 8)
        return message
    
    def _process_block(self, block: bytes):
        """Process a 512-bit block"""
        w = list(struct.unpack('>16I', block))
        
        # Extend words
        for i in range(16, 80):
            w.append(_rotr(w[i-3] ^ w[i-8] ^ w[i-14] ^ w[i-16], 31))
        
        # Initialize working variables
        a, b, c, d, e = self._h
        
        # Main loop
        for t in range(80):
            temp = (_rotr(a, 27) + self._f(t, b, c, d) + e + self._k(t) + w[t]) & 0xffffffff
            e = d
            d = c
            c = _rotr(b, 30)
            b = a
            a = temp
        
        # Add to hash
        self._h[0] = (self._h[0] + a) & 0xffffffff
        self._h[1] = (self._h[1] + b) & 0xffffffff
        self._h[2] = (self._h[2] + c) & 0xffffffff
        self._h[3] = (self._h[3] + d) & 0xffffffff
        self._h[4] = (self._h[4] + e) & 0xffffffff
    
    def update(self, data: bytes):
        """Update hash with new data"""
        self._buffer += data
        self._message_length += len(data)
        
        while len(self._buffer) >= 64:
            self._process_block(self._buffer[:64])
            self._buffer = self._buffer[64:]
    
    def digest(self) -> bytes:
        """Get final hash digest"""
        padded = self._pad_message(self._buffer)
        
        for i in range(0, len(padded), 64):
            self._process_block(padded[i:i+64])
        
        return struct.pack('>5I', *self._h)
    
    def hexdigest(self) -> str:
        """Get final hash digest as hex string"""
        return self.digest().hex()

def sha256(data: Union[str, bytes]) -> str:
    """Convenience function for SHA-256 hashing"""
    if isinstance(data, str):
        data = data.encode('utf-8')
    
    hasher = SHA256()
    hasher.update(data)
    return hasher.hexdigest()

def sha1(data: Union[str, bytes]) -> str:
    """Convenience function for SHA-1 hashing (deprecated)"""
    if isinstance(data, str):
        data = data.encode('utf-8')
    
    hasher = SHA1()
    hasher.update(data)
    return hasher.hexdigest()

# HMAC implementation using SHA-256
def hmac_sha256(key: bytes, message: bytes) -> bytes:
    """HMAC-SHA256 implementation"""
    block_size = 64  # SHA-256 block size
    
    # Adjust key length
    if len(key) > block_size:
        key = SHA256().update(key).digest()
    if len(key) < block_size:
        key = key + b'\x00' * (block_size - len(key))
    
    # Create inner and outer padding
    o_key_pad = bytes(k ^ 0x5c for k in key)
    i_key_pad = bytes(k ^ 0x36 for k in key)
    
    # Compute HMAC
    inner_hash = SHA256()
    inner_hash.update(i_key_pad + message)
    inner_result = inner_hash.digest()
    
    outer_hash = SHA256()
    outer_hash.update(o_key_pad + inner_result)
    
    return outer_hash.digest()

# Example usage and testing
if __name__ == "__main__":
    # Test SHA-256
    print("=== SHA-256 Tests ===")
    
    test_vectors = [
        ("", "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"),
        ("abc", "ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad"),
        ("The quick brown fox jumps over the lazy dog", 
         "d7a8fbb307d7809469ca9abcb0082e4f8d5651e46d3cdb762d02d0bf37c9e592"),
    ]
    
    for input_str, expected in test_vectors:
        result = sha256(input_str)
        print(f"Input: '{input_str}'")
        print(f"Expected: {expected}")
        print(f"Got:      {result}")
        print(f"Match: {result == expected}\n")
    
    # Test incremental hashing
    print("=== Incremental Hashing ===")
    hasher = SHA256()
    hasher.update(b"Hello, ")
    hasher.update(b"World!")
    incremental_result = hasher.hexdigest()
    
    direct_result = sha256("Hello, World!")
    print(f"Incremental: {incremental_result}")
    print(f"Direct:      {direct_result}")
    print(f"Match: {incremental_result == direct_result}\n")
    
    # Test SHA-1 (deprecated)
    print("=== SHA-1 Test (Deprecated) ===")
    sha1_result = sha1("abc")
    expected_sha1 = "a9993e364706816aba3e25717850c26c9cd0d89d"
    print(f"SHA-1 of 'abc': {sha1_result}")
    print(f"Expected:       {expected_sha1}")
    print(f"Match: {sha1_result == expected_sha1}\n")
    
    # Test HMAC-SHA256
    print("=== HMAC-SHA256 Test ===")
    key = b"secret_key"
    message = b"Hello, HMAC!"
    hmac_result = hmac_sha256(key, message)
    print(f"HMAC-SHA256: {hmac_result.hex()}")
    
    # Performance test
    print("=== Performance Test ===")
    import time
    
    large_data = b"A" * 1000000  # 1MB of data
    start_time = time.time()
    hash_result = sha256(large_data)
    end_time = time.time()
    
    print(f"Hashed 1MB in {end_time - start_time:.4f} seconds")
    print(f"Hash: {hash_result[:64]}...")
```

## Applications

### Digital Signatures
Hash functions are used to create message digests that are then signed:
- **RSA**: Sign SHA-256 hash of message
- **ECDSA**: Sign SHA-256 hash of message
- **DSA**: Sign SHA-1 or SHA-256 hash of message

### Password Storage
Store salted hashes instead of plaintext passwords:
```python
import os
password_hash = sha256(password + salt)
```

### Data Integrity
Verify data hasn't been modified:
- **File checksums**: Verify file integrity
- **Git commits**: Track changes in version control
- **Blockchain**: Link blocks in cryptocurrency

### Message Authentication (HMAC)
Authenticate messages using shared secret:
$$\text{HMAC}(K, m) = H((K \oplus opad) \| H((K \oplus ipad) \| m))$$

## Security Considerations

### SHA-1 Deprecation
- **Collision attacks**: Google's SHAttered attack (2017)
- **Not recommended**: For new applications
- **Legacy support**: Only where absolutely necessary

### SHA-2 Security
- **Current standard**: No practical attacks known
- **Collision resistance**: ~2^128 operations for SHA-256
- **Preimage resistance**: ~2^256 operations for SHA-256

### SHA-3 Advantages
- **Different construction**: Sponge-based (Keccak)
- **Quantum resistance**: Better theoretical foundation
- **Flexibility**: Variable output length

## Common Vulnerabilities

- **Length extension attacks**: Vulnerable hash functions (not SHA-3)
- **Rainbow tables**: Precomputed hash lookups
- **Timing attacks**: Constant-time comparison needed
- **Weak salts**: Predictable or short salts

## References

- FIPS 180-4: Secure Hash Standard (SHS)
- [SHA-2 Algorithm - Wikipedia](https://en.wikipedia.org/wiki/SHA-2)
- [SHA-3 Algorithm - Wikipedia](https://en.wikipedia.org/wiki/SHA-3)
- RFC 3174: US Secure Hash Algorithm 1 (SHA1)
- RFC 6234: US Secure Hash Algorithms