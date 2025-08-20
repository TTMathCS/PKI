# AES (Advanced Encryption Standard)

## Overview

AES is a symmetric encryption algorithm established as the encryption standard by the U.S. National Institute of Standards and Technology (NIST) in 2001. Also known as Rijndael, AES was developed by Belgian cryptographers Joan Daemen and Vincent Rijmen. It replaced the older Data Encryption Standard (DES) and is widely used worldwide for securing sensitive data.

## Mathematical Foundation

### Key Features
- **Block cipher**: Operates on fixed-size blocks of 128 bits
- **Symmetric**: Same key for encryption and decryption
- **Key sizes**: 128, 192, or 256 bits
- **Rounds**: 10 rounds (AES-128), 12 rounds (AES-192), 14 rounds (AES-256)

### State Matrix

AES operates on a 4×4 matrix of bytes called the **state**:

```
State = [s₀₀  s₀₁  s₀₂  s₀₃]
        [s₁₀  s₁₁  s₁₂  s₁₃]
        [s₂₀  s₂₁  s₂₂  s₂₃]
        [s₃₀  s₃₁  s₃₂  s₃₃]
```

### AES Round Operations

Each round (except the last) consists of four operations:

#### 1. SubBytes (Substitution)
Replace each byte with a corresponding byte from the S-box:
```
s'ᵢ,j = S-box[sᵢ,j]
```

#### 2. ShiftRows
Cyclically shift rows:
- Row 0: No shift
- Row 1: Shift left by 1 position
- Row 2: Shift left by 2 positions  
- Row 3: Shift left by 3 positions

#### 3. MixColumns
Matrix multiplication in $GF(2^8)$:
```
[s'₀ⱼ]   [2 3 1 1]   [s₀ⱼ]
[s'₁ⱼ] = [1 2 3 1] × [s₁ⱼ]
[s'₂ⱼ]   [1 1 2 3]   [s₂ⱼ]
[s'₃ⱼ]   [3 1 1 2]   [s₃ⱼ]
```

Each element is calculated as:
```
s'₀,j = 2·s₀,j + 3·s₁,j + 1·s₂,j + 1·s₃,j
s'₁,j = 1·s₀,j + 2·s₁,j + 3·s₂,j + 1·s₃,j
s'₂,j = 1·s₀,j + 1·s₁,j + 2·s₂,j + 3·s₃,j
s'₃,j = 3·s₀,j + 1·s₁,j + 1·s₂,j + 2·s₃,j
```

#### 4. AddRoundKey
XOR with round key:
```
s'ᵢ,j = sᵢ,j ⊕ kᵢ,j⁽ʳ⁾
```

### Key Schedule

The key schedule generates round keys from the original key using:
- **RotWord**: Rotate 4-byte word
- **SubWord**: Apply S-box to each byte
- **Rcon**: Round constant

For AES-128:
**Key Schedule Formula:**

If `i mod 4 = 0`:
```
W[i] = W[i-4] ⊕ SubWord(RotWord(W[i-1])) ⊕ Rcon[i/4]
```

Otherwise:
```
W[i] = W[i-4] ⊕ W[i-1]
```

## Security Considerations

- **Key size**: Use AES-256 for maximum security
- **Mode of operation**: Use authenticated modes like GCM or CCM
- **IV/Nonce**: Must be unique and unpredictable for each encryption
- **Padding**: PKCS#7 padding for modes requiring it
- **Side-channel attacks**: Use constant-time implementations

## Python Implementation

```python
import os
from typing import List, Tuple

# AES S-box
S_BOX = [
    0x63, 0x7c, 0x77, 0x7b, 0xf2, 0x6b, 0x6f, 0xc5, 0x30, 0x01, 0x67, 0x2b, 0xfe, 0xd7, 0xab, 0x76,
    0xca, 0x82, 0xc9, 0x7d, 0xfa, 0x59, 0x47, 0xf0, 0xad, 0xd4, 0xa2, 0xaf, 0x9c, 0xa4, 0x72, 0xc0,
    0xb7, 0xfd, 0x93, 0x26, 0x36, 0x3f, 0xf7, 0xcc, 0x34, 0xa5, 0xe5, 0xf1, 0x71, 0xd8, 0x31, 0x15,
    0x04, 0xc7, 0x23, 0xc3, 0x18, 0x96, 0x05, 0x9a, 0x07, 0x12, 0x80, 0xe2, 0xeb, 0x27, 0xb2, 0x75,
    0x09, 0x83, 0x2c, 0x1a, 0x1b, 0x6e, 0x5a, 0xa0, 0x52, 0x3b, 0xd6, 0xb3, 0x29, 0xe3, 0x2f, 0x84,
    0x53, 0xd1, 0x00, 0xed, 0x20, 0xfc, 0xb1, 0x5b, 0x6a, 0xcb, 0xbe, 0x39, 0x4a, 0x4c, 0x58, 0xcf,
    0xd0, 0xef, 0xaa, 0xfb, 0x43, 0x4d, 0x33, 0x85, 0x45, 0xf9, 0x02, 0x7f, 0x50, 0x3c, 0x9f, 0xa8,
    0x51, 0xa3, 0x40, 0x8f, 0x92, 0x9d, 0x38, 0xf5, 0xbc, 0xb6, 0xda, 0x21, 0x10, 0xff, 0xf3, 0xd2,
    0xcd, 0x0c, 0x13, 0xec, 0x5f, 0x97, 0x44, 0x17, 0xc4, 0xa7, 0x7e, 0x3d, 0x64, 0x5d, 0x19, 0x73,
    0x60, 0x81, 0x4f, 0xdc, 0x22, 0x2a, 0x90, 0x88, 0x46, 0xee, 0xb8, 0x14, 0xde, 0x5e, 0x0b, 0xdb,
    0xe0, 0x32, 0x3a, 0x0a, 0x49, 0x06, 0x24, 0x5c, 0xc2, 0xd3, 0xac, 0x62, 0x91, 0x95, 0xe4, 0x79,
    0xe7, 0xc8, 0x37, 0x6d, 0x8d, 0xd5, 0x4e, 0xa9, 0x6c, 0x56, 0xf4, 0xea, 0x65, 0x7a, 0xae, 0x08,
    0xba, 0x78, 0x25, 0x2e, 0x1c, 0xa6, 0xb4, 0xc6, 0xe8, 0xdd, 0x74, 0x1f, 0x4b, 0xbd, 0x8b, 0x8a,
    0x70, 0x3e, 0xb5, 0x66, 0x48, 0x03, 0xf6, 0x0e, 0x61, 0x35, 0x57, 0xb9, 0x86, 0xc1, 0x1d, 0x9e,
    0xe1, 0xf8, 0x98, 0x11, 0x69, 0xd9, 0x8e, 0x94, 0x9b, 0x1e, 0x87, 0xe9, 0xce, 0x55, 0x28, 0xdf,
    0x8c, 0xa1, 0x89, 0x0d, 0xbf, 0xe6, 0x42, 0x68, 0x41, 0x99, 0x2d, 0x0f, 0xb0, 0x54, 0xbb, 0x16
]

# AES inverse S-box
INV_S_BOX = [
    0x52, 0x09, 0x6a, 0xd5, 0x30, 0x36, 0xa5, 0x38, 0xbf, 0x40, 0xa3, 0x9e, 0x81, 0xf3, 0xd7, 0xfb,
    0x7c, 0xe3, 0x39, 0x82, 0x9b, 0x2f, 0xff, 0x87, 0x34, 0x8e, 0x43, 0x44, 0xc4, 0xde, 0xe9, 0xcb,
    0x54, 0x7b, 0x94, 0x32, 0xa6, 0xc2, 0x23, 0x3d, 0xee, 0x4c, 0x95, 0x0b, 0x42, 0xfa, 0xc3, 0x4e,
    0x08, 0x2e, 0xa1, 0x66, 0x28, 0xd9, 0x24, 0xb2, 0x76, 0x5b, 0xa2, 0x49, 0x6d, 0x8b, 0xd1, 0x25,
    0x72, 0xf8, 0xf6, 0x64, 0x86, 0x68, 0x98, 0x16, 0xd4, 0xa4, 0x5c, 0xcc, 0x5d, 0x65, 0xb6, 0x92,
    0x6c, 0x70, 0x48, 0x50, 0xfd, 0xed, 0xb9, 0xda, 0x5e, 0x15, 0x46, 0x57, 0xa7, 0x8d, 0x9d, 0x84,
    0x90, 0xd8, 0xab, 0x00, 0x8c, 0xbc, 0xd3, 0x0a, 0xf7, 0xe4, 0x58, 0x05, 0xb8, 0xb3, 0x45, 0x06,
    0xd0, 0x2c, 0x1e, 0x8f, 0xca, 0x3f, 0x0f, 0x02, 0xc1, 0xaf, 0xbd, 0x03, 0x01, 0x13, 0x8a, 0x6b,
    0x3a, 0x91, 0x11, 0x41, 0x4f, 0x67, 0xdc, 0xea, 0x97, 0xf2, 0xcf, 0xce, 0xf0, 0xb4, 0xe6, 0x73,
    0x96, 0xac, 0x74, 0x22, 0xe7, 0xad, 0x35, 0x85, 0xe2, 0xf9, 0x37, 0xe8, 0x1c, 0x75, 0xdf, 0x6e,
    0x47, 0xf1, 0x1a, 0x71, 0x1d, 0x29, 0xc5, 0x89, 0x6f, 0xb7, 0x62, 0x0e, 0xaa, 0x18, 0xbe, 0x1b,
    0xfc, 0x56, 0x3e, 0x4b, 0xc6, 0xd2, 0x79, 0x20, 0x9a, 0xdb, 0xc0, 0xfe, 0x78, 0xcd, 0x5a, 0xf4,
    0x1f, 0xdd, 0xa8, 0x33, 0x88, 0x07, 0xc7, 0x31, 0xb1, 0x12, 0x10, 0x59, 0x27, 0x80, 0xec, 0x5f,
    0x60, 0x51, 0x7f, 0xa9, 0x19, 0xb5, 0x4a, 0x0d, 0x2d, 0xe5, 0x7a, 0x9f, 0x93, 0xc9, 0x9c, 0xef,
    0xa0, 0xe0, 0x3b, 0x4d, 0xae, 0x2a, 0xf5, 0xb0, 0xc8, 0xeb, 0xbb, 0x3c, 0x83, 0x53, 0x99, 0x61,
    0x17, 0x2b, 0x04, 0x7e, 0xba, 0x77, 0xd6, 0x26, 0xe1, 0x69, 0x14, 0x63, 0x55, 0x21, 0x0c, 0x7d
]

# Round constants for key expansion
RCON = [0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x80, 0x1B, 0x36]

def gf_multiply(a: int, b: int) -> int:
    """Multiply two numbers in GF(2^8) with irreducible polynomial 0x11B"""
    result = 0
    for _ in range(8):
        if b & 1:
            result ^= a
        high_bit = a & 0x80
        a <<= 1
        if high_bit:
            a ^= 0x1B  # Irreducible polynomial
        b >>= 1
    return result & 0xFF

def sub_bytes(state: List[List[int]]) -> List[List[int]]:
    """Apply S-box substitution to each byte in the state"""
    return [[S_BOX[state[i][j]] for j in range(4)] for i in range(4)]

def inv_sub_bytes(state: List[List[int]]) -> List[List[int]]:
    """Apply inverse S-box substitution to each byte in the state"""
    return [[INV_S_BOX[state[i][j]] for j in range(4)] for i in range(4)]

def shift_rows(state: List[List[int]]) -> List[List[int]]:
    """Shift rows of the state matrix"""
    result = [row[:] for row in state]
    
    # Row 1: shift left by 1
    result[1] = state[1][1:] + state[1][:1]
    
    # Row 2: shift left by 2
    result[2] = state[2][2:] + state[2][:2]
    
    # Row 3: shift left by 3
    result[3] = state[3][3:] + state[3][:3]
    
    return result

def inv_shift_rows(state: List[List[int]]) -> List[List[int]]:
    """Inverse shift rows of the state matrix"""
    result = [row[:] for row in state]
    
    # Row 1: shift right by 1
    result[1] = state[1][-1:] + state[1][:-1]
    
    # Row 2: shift right by 2
    result[2] = state[2][-2:] + state[2][:-2]
    
    # Row 3: shift right by 3
    result[3] = state[3][-3:] + state[3][:-3]
    
    return result

def mix_columns(state: List[List[int]]) -> List[List[int]]:
    """Mix columns operation using matrix multiplication in GF(2^8)"""
    mix_matrix = [[2, 3, 1, 1], [1, 2, 3, 1], [1, 1, 2, 3], [3, 1, 1, 2]]
    result = [[0 for _ in range(4)] for _ in range(4)]
    
    for col in range(4):
        for row in range(4):
            for i in range(4):
                result[row][col] ^= gf_multiply(mix_matrix[row][i], state[i][col])
    
    return result

def inv_mix_columns(state: List[List[int]]) -> List[List[int]]:
    """Inverse mix columns operation"""
    inv_mix_matrix = [[14, 11, 13, 9], [9, 14, 11, 13], [13, 9, 14, 11], [11, 13, 9, 14]]
    result = [[0 for _ in range(4)] for _ in range(4)]
    
    for col in range(4):
        for row in range(4):
            for i in range(4):
                result[row][col] ^= gf_multiply(inv_mix_matrix[row][i], state[i][col])
    
    return result

def add_round_key(state: List[List[int]], round_key: List[List[int]]) -> List[List[int]]:
    """XOR state with round key"""
    return [[state[i][j] ^ round_key[i][j] for j in range(4)] for i in range(4)]

def key_expansion(key: bytes) -> List[List[List[int]]]:
    """Expand the key into round keys"""
    key_length = len(key)
    if key_length == 16:
        num_rounds = 10
        num_words = 44
    elif key_length == 24:
        num_rounds = 12
        num_words = 52
    elif key_length == 32:
        num_rounds = 14
        num_words = 60
    else:
        raise ValueError("Invalid key length")
    
    # Convert key to words (4 bytes each)
    w = []
    for i in range(0, key_length, 4):
        w.append(list(key[i:i+4]))
    
    # Expand key
    for i in range(key_length // 4, num_words):
        temp = w[i-1][:]
        
        if i % (key_length // 4) == 0:
            # RotWord
            temp = temp[1:] + temp[:1]
            # SubWord
            temp = [S_BOX[b] for b in temp]
            # XOR with Rcon
            temp[0] ^= RCON[(i // (key_length // 4)) - 1]
        elif key_length == 32 and i % 8 == 4:
            # For AES-256, apply SubWord every 8 words
            temp = [S_BOX[b] for b in temp]
        
        # XOR with word from key_length//4 positions back
        new_word = [w[i - key_length // 4][j] ^ temp[j] for j in range(4)]
        w.append(new_word)
    
    # Convert words to round keys
    round_keys = []
    for round_num in range(num_rounds + 1):
        round_key = [[0 for _ in range(4)] for _ in range(4)]
        for col in range(4):
            word = w[round_num * 4 + col]
            for row in range(4):
                round_key[row][col] = word[row]
        round_keys.append(round_key)
    
    return round_keys

def bytes_to_state(data: bytes) -> List[List[int]]:
    """Convert 16 bytes to 4x4 state matrix"""
    state = [[0 for _ in range(4)] for _ in range(4)]
    for i in range(16):
        state[i % 4][i // 4] = data[i]
    return state

def state_to_bytes(state: List[List[int]]) -> bytes:
    """Convert 4x4 state matrix to 16 bytes"""
    result = bytearray(16)
    for i in range(16):
        result[i] = state[i % 4][i // 4]
    return bytes(result)

def pkcs7_pad(data: bytes, block_size: int = 16) -> bytes:
    """Add PKCS#7 padding"""
    padding_length = block_size - (len(data) % block_size)
    padding = bytes([padding_length] * padding_length)
    return data + padding

def pkcs7_unpad(data: bytes) -> bytes:
    """Remove PKCS#7 padding"""
    if not data:
        raise ValueError("Cannot unpad empty data")
    
    padding_length = data[-1]
    if padding_length > len(data) or padding_length == 0:
        raise ValueError("Invalid padding")
    
    for i in range(padding_length):
        if data[-(i+1)] != padding_length:
            raise ValueError("Invalid padding")
    
    return data[:-padding_length]

class AES:
    """AES encryption/decryption implementation"""
    
    def __init__(self, key: bytes):
        if len(key) not in [16, 24, 32]:
            raise ValueError("Key must be 16, 24, or 32 bytes long")
        
        self.key = key
        self.round_keys = key_expansion(key)
        self.num_rounds = len(self.round_keys) - 1
    
    def encrypt_block(self, plaintext: bytes) -> bytes:
        """Encrypt a single 16-byte block"""
        if len(plaintext) != 16:
            raise ValueError("Block must be exactly 16 bytes")
        
        state = bytes_to_state(plaintext)
        
        # Initial round key addition
        state = add_round_key(state, self.round_keys[0])
        
        # Main rounds
        for round_num in range(1, self.num_rounds):
            state = sub_bytes(state)
            state = shift_rows(state)
            state = mix_columns(state)
            state = add_round_key(state, self.round_keys[round_num])
        
        # Final round (no MixColumns)
        state = sub_bytes(state)
        state = shift_rows(state)
        state = add_round_key(state, self.round_keys[self.num_rounds])
        
        return state_to_bytes(state)
    
    def decrypt_block(self, ciphertext: bytes) -> bytes:
        """Decrypt a single 16-byte block"""
        if len(ciphertext) != 16:
            raise ValueError("Block must be exactly 16 bytes")
        
        state = bytes_to_state(ciphertext)
        
        # Initial round key addition
        state = add_round_key(state, self.round_keys[self.num_rounds])
        
        # Main rounds (in reverse)
        for round_num in range(self.num_rounds - 1, 0, -1):
            state = inv_shift_rows(state)
            state = inv_sub_bytes(state)
            state = add_round_key(state, self.round_keys[round_num])
            state = inv_mix_columns(state)
        
        # Final round (no InvMixColumns)
        state = inv_shift_rows(state)
        state = inv_sub_bytes(state)
        state = add_round_key(state, self.round_keys[0])
        
        return state_to_bytes(state)
    
    def encrypt_ecb(self, plaintext: bytes) -> bytes:
        """Encrypt using ECB mode (not recommended for production)"""
        plaintext = pkcs7_pad(plaintext)
        ciphertext = b""
        
        for i in range(0, len(plaintext), 16):
            block = plaintext[i:i+16]
            ciphertext += self.encrypt_block(block)
        
        return ciphertext
    
    def decrypt_ecb(self, ciphertext: bytes) -> bytes:
        """Decrypt using ECB mode"""
        if len(ciphertext) % 16 != 0:
            raise ValueError("Ciphertext length must be multiple of 16")
        
        plaintext = b""
        for i in range(0, len(ciphertext), 16):
            block = ciphertext[i:i+16]
            plaintext += self.decrypt_block(block)
        
        return pkcs7_unpad(plaintext)
    
    def encrypt_cbc(self, plaintext: bytes, iv: bytes = None) -> Tuple[bytes, bytes]:
        """Encrypt using CBC mode"""
        if iv is None:
            iv = os.urandom(16)
        
        if len(iv) != 16:
            raise ValueError("IV must be 16 bytes")
        
        plaintext = pkcs7_pad(plaintext)
        ciphertext = b""
        previous_block = iv
        
        for i in range(0, len(plaintext), 16):
            block = plaintext[i:i+16]
            # XOR with previous ciphertext block (or IV for first block)
            xored_block = bytes(a ^ b for a, b in zip(block, previous_block))
            encrypted_block = self.encrypt_block(xored_block)
            ciphertext += encrypted_block
            previous_block = encrypted_block
        
        return ciphertext, iv
    
    def decrypt_cbc(self, ciphertext: bytes, iv: bytes) -> bytes:
        """Decrypt using CBC mode"""
        if len(ciphertext) % 16 != 0:
            raise ValueError("Ciphertext length must be multiple of 16")
        
        if len(iv) != 16:
            raise ValueError("IV must be 16 bytes")
        
        plaintext = b""
        previous_block = iv
        
        for i in range(0, len(ciphertext), 16):
            block = ciphertext[i:i+16]
            decrypted_block = self.decrypt_block(block)
            # XOR with previous ciphertext block (or IV for first block)
            plaintext_block = bytes(a ^ b for a, b in zip(decrypted_block, previous_block))
            plaintext += plaintext_block
            previous_block = block
        
        return pkcs7_unpad(plaintext)

# Example usage and testing
if __name__ == "__main__":
    # Test with different key sizes
    keys = [
        os.urandom(16),  # AES-128
        os.urandom(24),  # AES-192
        os.urandom(32),  # AES-256
    ]
    
    plaintext = b"Hello, AES! This is a test message for encryption."
    
    for i, key in enumerate([keys[0]]):  # Test with AES-128 for demo
        key_size = len(key) * 8
        print(f"=== AES-{key_size} Test ===")
        print(f"Key (hex): {key.hex()}")
        print(f"Plaintext: {plaintext.decode()}")
        
        aes = AES(key)
        
        # ECB mode test
        print("\n--- ECB Mode ---")
        ciphertext_ecb = aes.encrypt_ecb(plaintext)
        print(f"Ciphertext (hex): {ciphertext_ecb.hex()}")
        
        decrypted_ecb = aes.decrypt_ecb(ciphertext_ecb)
        print(f"Decrypted: {decrypted_ecb.decode()}")
        print(f"ECB successful: {plaintext == decrypted_ecb}")
        
        # CBC mode test
        print("\n--- CBC Mode ---")
        ciphertext_cbc, iv = aes.encrypt_cbc(plaintext)
        print(f"IV (hex): {iv.hex()}")
        print(f"Ciphertext (hex): {ciphertext_cbc.hex()}")
        
        decrypted_cbc = aes.decrypt_cbc(ciphertext_cbc, iv)
        print(f"Decrypted: {decrypted_cbc.decode()}")
        print(f"CBC successful: {plaintext == decrypted_cbc}")
```

## Modes of Operation

### Electronic Codebook (ECB)
- **Not recommended**: Identical plaintext blocks produce identical ciphertext
- **Use case**: Only for single-block encryption

### Cipher Block Chaining (CBC)
- **Secure**: Uses IV and chains blocks
- **Requires**: Random IV for each encryption
- **Padding**: PKCS#7 padding required

### Galois/Counter Mode (GCM)
- **Authenticated**: Provides both encryption and authentication
- **Performance**: Can be parallelized
- **Recommended**: For modern applications

### Counter Mode (CTR)
- **Stream cipher-like**: Converts block cipher to stream cipher
- **Parallelizable**: Both encryption and decryption
- **No padding**: Required

## Common Vulnerabilities

- **ECB Mode**: Reveals patterns in plaintext
- **Key Reuse**: Same key with predictable IV/nonce
- **Padding Oracle**: Attacks on CBC mode with padding
- **Side-channel**: Timing attacks, power analysis
- **Weak Keys**: All-zero or pattern keys

## References

- FIPS 197: Advanced Encryption Standard (AES)
- [AES Algorithm - Wikipedia](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)
- NIST SP 800-38A: Block Cipher Modes of Operation
- RFC 3565: Use of the AES Encryption Algorithm