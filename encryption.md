## Encryption
Encryption is the process of converting plain data (plaintext) into a scrambled format (ciphertext) so that only authorized parties can read it.
You need a key to encrypt and decrypt.

### Flow
Plaintext + Key → Encryption → Ciphertext  
Ciphertext + Key → Decryption → Plaintext

### Types
1. Symmetric Encryption
   - Same key used for encryption & decryption
   - Examples: AES (Advanced Encryption Standard), DES, 3DES
   - Flow:
     - Message → AES Encrypt with Key → Ciphertext
     - Ciphertext → AES Decrypt with Same Key → Original Message

2. Asymmetric Encryption
   - Uses two keys:
     - Public key → encrypt
     - Private key → decrypt
   - Examples: RSA, ECC
   - Flow:
     - Alice encrypts message with Bob's public key
     - Bob decrypts message with his private key

### Why Encryption is Important
Provides CIA: Confidentiality, Integrity, Authentication

### Real-time Use Cases
- HTTPS / SSL/TLS
- Messaging apps (WhatsApp, Signal)
- Secure file storage
- VPNs

