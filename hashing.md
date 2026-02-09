## Hashing
- Hashing is a process that converts any input into a fixed-length string of characters, called a hash.
- Hashing is one-way: you cannot reverse it.
- The same input always produces the same hash, but a small change → completely different hash.

### Flow
Input (any size) → Hash Function → Hash (fixed size)

### Examples
- MD5
- SHA-256
- SHA-512

### Real-time Example
- Store hashed passwords in the database.
- When a user logs in, hash the entered password and compare it with the stored hash.
- This protects passwords if the database is leaked.

### Why Hash + Salt
Without salt:
- Same password → same hash
- Two users with "password123" → same hash
- Attacker can see duplicates

With salt:
- Salt is a random value added to the password before hashing.
- More secure; prevents same hash for same password.
- Store the salt in the database along with the hash.

