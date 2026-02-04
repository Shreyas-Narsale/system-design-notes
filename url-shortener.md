## URL Shortener

### Problem Statement

Assume URL `https://www.systeminterview.com/q=chatsystem&c=loggedin&v=v3&l=long` is the original URL. Your service creates an alias with shorter length: `https://tinyurl.com/y7keocwj`. If you click the alias, it redirects you to the original URL.

- **Write operation**: 100 million URLs are generated per day.  
  - Write operations per second: 100 million / 24 / 3600 = 1160
- **Read operation**: Assuming the ratio of read operations to write operations is 10:1, read operations per second: 1160 * 10 = 11,600
- **Total records**: Assuming the URL shortener service will run for 10 years, this means we must support  
  100 million * 365 * 10 = 365 billion records.
- **Average URL length**: Assume average URL length is 100.
- **Storage requirement over 10 years**:  
  365 billion * 100 bytes = 36.5 TB


## Solution

### URL Redirecting

- **Example**:
  - Short URL: `https://sho.rt/aZ9kL`
  - Long URL: `https://www.amazon.in/Samsung-Galaxy-Ultra-Storage-Black/dp`

Once a user hits the short URL in the browser (`https://sho.rt/aZ9kL`), it returns a 301 or 302 status code with a header update in the response:

- `Location: <long-url>` (for example, `https://www.amazon.in/Samsung-Galaxy-Ultra-Storage-Black/dp`)

Then the browser again hits the long URL with the same request data.

#### 301 Redirect

- “Permanently” moved to the long URL.
- As it is permanently redirected, the browser caches the response, and subsequent requests for the same URL will not be sent to the URL shortening service.
- Instead, requests are redirected to the long URL server directly.
- To reduce the server load, using a 301 redirect makes sense.

#### 302 Redirect

- “Temporarily” moved to the long URL.
- Subsequent requests for the same URL will be sent to the URL shortening service first.
- Then, they are redirected to the long URL server.
- If analytics is important, a 302 redirect is a better choice as it can track click rate and source of the click more easily.


## Data Model

`url_mapping`

- `id` (bigint, PK)
- `short_code` (varchar, UNIQUE)
- `long_url` (text)
- `created_at`
- `expires_at`


## Hash Function

A hash function is used to hash a long URL to a short URL, also known as `hashValue`.

- **hashValue**:
  - Consists of characters from `[0-9, a-z, A-Z]`, containing 10 + 26 + 26 = 62 possible characters.
  - As we need to support more than 365 billion URLs, we calculate:  
    (62^7 = 3,521,614,606,208 approx 3.5) trillion.
  - So the length of `hashValue` is 7.

We should implement a hash function that hashes a long URL to a 7-character string.

### 1. Hash + Collision Resolution

Use well-known hash functions like CRC32, MD5, or SHA-1.

- **Examples**:

  | Hash function | Hash value (Hexadecimal)                           |
  |---------------|----------------------------------------------------|
  | CRC32         | 5cb54054                                           |
  | MD5           | 5a62509a84df9ee03fe1230b9df8b84e                   |
  | SHA-1         | 0eeae7916c06853901d9ccbefbfcaf4de57ed85b           |

Even the shortest hash value (from CRC32) is too long (more than 7 characters). How can we make it shorter?

- The first approach is to collect the first 7 characters of a hash value.

**Issues**:

- Two different URLs can produce the same 7-character string (collision).

  - **Collision Resolution Strategy**:  
    If another long URL already uses this short URL, we change the input slightly and retry:
      - `hash(longURL + "1")`
      - `hash(longURL + "2")`
      - `hash(longURL + "3")`

- **Big drawback**:
  - Requires a DB lookup for every hash attempt.

### 2. Base 62 Conversion

First, from the long URL we create a unique ID using a distributed ID generator like Snowflake.  
Then we apply Base 62 conversion to the unique ID.

- Base 62 conversion is used as there are 62 possible characters for `hashValue`.

Let us understand base 10:

- **Base 10**:  
  11157 in base 10 → 11157

Now, Base 62 conversion:

- Convert 11157 to base 62:

  | Divisor | Value | Remainder | Representation in Base 62 |
  |---------|-------|-----------|----------------------------|
  | 62      | 11157 | 59        | X                          |
  | 62      | 179   | 55        | T                          |
  | 62      | 2     | 2         | 2                          |
  |         | 0     |           |                            |

So the hash is `XT2`.

- **Short URL**: `https://sho.rt/XT2`


### Comparison of the Two Approaches

| Aspect                                           | Hash + Collision Resolution                                | Base 62 Conversion                                                |
|--------------------------------------------------|------------------------------------------------------------|-------------------------------------------------------------------|
| Short URL length                                 | Fixed short URL length.                                    | Short URL length is not fixed; it goes up with the ID.           |
| Dependency on unique ID generator                | Does not need a unique ID generator.                       | Depends on a unique ID generator.                                |
| Collision                                        | Collision is possible and needs to be resolved.            | Collision is not possible because the ID is unique.              |
| Predictability of next short URL                 | Not possible to figure out the next available short URL    | Easy to figure out the next available short URL if ID increments |
|                                                  | because it doesn’t depend on ID.                           | by 1 for a new entry. This can be a security concern.            |


## Final Conclusion

We will use **Base 62 conversion with Snowflake-style IDs** (most popular distributed unique ID generator), used by: Twitter, Discord, Instagram.

- In the Base-62 approach, we don’t hash the URL. Instead, we generate a globally unique numeric ID using a distributed ID generator like Snowflake and convert it to Base-62.
- This guarantees uniqueness, avoids collisions, and simplifies the write path.
- If IDs increment, it is easy for attackers to guess URLs.
- To avoid this, we will **add a salt**.


## Final Scalable Architecture

### 1. Services

1️⃣ **URL Redirect API (Data Plane – HOT PATH)**  
- Purpose: ultra-fast redirects  
- No auth, no business logic

2️⃣ **URL Management API (Control Plane)**  
- Purpose: everything else  
  - URL generation  
  - User management  
  - Rate limits  
  - Analytics config  
  - Metadata  

### 2. High-Level Architecture

```text
                    ┌──────────────┐
                    │   Clients    │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │  API Gateway │
                    │   / Ingress  │
                    └──────┬───────┘
            ┌──────────────┴──────────────┐
            │                               │
   GET /{code}                        /user/
   (public)                          /shorten
                                     /urlManager
            │                               │
┌───────────▼───────────┐       ┌──────────▼───────────┐
│ URL Redirect Service  │       │ URL Management Svc   │
│ (read-heavy)          │       │ (write-heavy)        │
└───────────┬───────────┘       └──────────┬───────────┘
            │                               │
            │                               │
      ┌─────▼─────┐                         │
      │   Redis   │                         │
      │  Cache    │                         │
      └─────┬─────┘                         │
            │                               │
  Cache hit │                               │
  --------->│                               │
            │                               │
     Cache miss?                           │
            │                               │
            ▼                               │
     PostgreSQL (source of truth)          │
            │                               │
            ▼                               │
  Return longURL + Add to Redis            │
            │                               │
            ▼                               ▼
       301 / 302 Redirect             Persist shortURL + metadata
                                         Generate ID + Base62
                                         User management
                                         Rate limiting config
                                         Analytics events
```

## Database Requirements for Scalability

### 1. Caching & Hot Path Optimizations

- **Use Redis Cluster**:
  - For high QPS, use Redis Cluster to shard cache across multiple nodes.

### 2. Database Scaling

- **Postgres Read Replicas**:
  - The Redirect service could query read replicas if you decide to bypass Redis for rare cache misses.
  - The Management service always writes to the master node.

- **Sharding / Partitioning `urls` table**:
  - Partition by short code or another appropriate key.
  - Why?
    - The Redirect service only knows `short_code`.
    - The DB can route the query to the exact partition.
    - Zero fan-out.
  - This keeps queries fast and avoids a single-node bottleneck.

- **Replication strategy**:
  - **Rules**:
    - Writes: only to primary.
    - Reads (Redirect service): replicas ONLY.


## Deployment

### Kubernetes Scaling Infra Plan

- **Compute**
  - Separate Deployments:
    - `url-redirect` (read-heavy)
    - `url-management` (write-heavy)

- **Stateless containers**

