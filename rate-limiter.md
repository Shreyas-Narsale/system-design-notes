## Rate Limiter
- a rate limiter limits the number of client requests allowed to be sent over a specified period.
- If the API request count exceeds the threshold defined by the rate limiter, all the excess calls are blocked.

## benefits
1. Almost all APIs published by large tech companies enforce some form of rate limiting. 
   ex. example, Twitter limits the number of tweets to 300 per 3 hours [2]. 
2. Reduce cost. Limiting excess requests for paid third party APIs. 

## Requirements
1. Accuracy
2. Low latency. The rate limiter should not slow down HTTP response time.
3. Use as little memory as possible
4. Distributed rate limiting. The rate limiter can be shared across multiple servers or processes.
5. Show clear exceptions to users when their requests are throttled.
6. High fault tolerance. If there are any problems with the rate limiter (for example, a cache server goes offline), it does not affect the entire system.

## Desgin
- create a rate limiter middleware, 
- if limit is exceeded, returns a HTTP status code 429 -> sent too many requests.
- rate limiting is usually implemented within a component called API gateway. I gateway is a fully managed service that supports rate limiting, SSL termination, authentication, IP whitelisting, servicing static content, etc

Sample Response:
```
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Retry-After: 30

{
  "error": "rate_limit_exceeded",
  "message": "Too many requests. Try again after 30 seconds."
}
```

## Algorithms for rate limiting
- Token bucket
- Leaking bucket
- Fixed window counter
- Sliding window log
- Sliding window counter

## Token bucket algorithm
- A token bucket is a container that has pre-defined capacity.
- At Start, Tokens are put in the bucket (means bucket is full with tokens)
- If there are enough tokens, we take one token out for each request, and the request goes through. 
- If there are not enough tokens, the request is dropped.
- at refil time, tokens are added to the bucket.
- accept two parameters:
  - bucket size: the maximum number of tokens in the bucket.
  - token refill rate (time interval): the rate at which tokens are added to the bucket.

ex. Bucket size = 5  
Refill rate = 1 token per second
- at t=0, bucket is full with 5 tokens.
- 5 Requests Come Immediately, Each request consumes 1 token:
  - Request 1 → 4 tokens left
  - Request 2 → 3 tokens left
  - Request 3 → 2 tokens left
  - Request 4 → 1 token left
  - Request 5 → 0 tokens left
- 6th Request Comes Immediately  
  Tokens = 0, dropped
- After 1 Second, as per refill rule: 1 token per second  
  Now, Tokens = 1 → One request can now pass.
- After 2 Seconds, as per refill rule: 1 token per second  
  Now, Tokens = 2 → Two requests can now pass

final colcusion:  
Short burst capacity + controlled average rate

formula:
```
var (
    capacity       = maximum tokens bucket can hold
    tokens         = current tokens in bucket
    refill_rate    = tokens added per second
    last_refill    = last time bucket was updated
    now            = current time
)

elapsed_time = now - last_refill
tokens_to_add = elapsed_time * refill_rate
tokens = min(capacity, tokens + tokens_to_add)
last_refill = now
if tokens >= 1:
    tokens -= 1
    allow request
else:
    reject request
```

## Leaking bucket algorithm
implemented with first-in-first-out (FIFO) queue
- When a request arrives, the system checks if the queue is full. If it is not full, the request is added to the queue.
- Otherwise, the request is dropped.
- Requests are pulled from the queue and processed at regular intervals.

two parameters:
- Bucket size: it is equal to the queue size. 
- Outflow rate: the rate at which requests are pulled from the queue.

cons:
- A burst of traffic fills up the queue with old requests, and if they are not processed in time, recent requests will be rate limited.

ex.:
Let’s take: 
- Bucket size = 5
- Outflow rate = 1 request/sec

At Time = 0  
Queue is empty.

Suddenly 7 Requests Arrive
- Request 1 → added
- Request 2 → added
- Request 3 → added
- Request 4 → added
- Request 5 → added
- Request 6 → dropped ❌ (queue full)
- Request 7 → dropped ❌

Queue now contains:  
[1,2,3,4,5]

After 1 Second:
- 1 request processed (FIFO)
- Processed → Request 1
- Queue now: [2,3,4,5]

After 2 Seconds:
- Processed → Request 2
- Queue now: [3,4,5]

formula:
```
var(
    capacity     = maximum queue size
    queue        = FIFO queue
    outflow_rate = requests processed per second
    last_process = last time a request was processed
)
elapsed_time = now - last_process
requests_to_process = elapsed_time * outflow_rate
remove min(queue_size, requests_to_process) from queue
If new request arrives:
    if queue_size < capacity:
        add request to queue
    else:
        drop request
last_process = now
```

## Fixed window counter algorithm
- number of requests in a fixed time window.
- fix-sized time windows and assign a counter for each window.
- If limit exceeded → reject requests
- When new window starts → counter resets

ex. Allow 5 requests per 10 seconds
- Count requests in current 10-second window
- If count > 5 → reject

formula:
```
var(
    limit       = max requests allowed per window
    window_size = duration of each window (seconds, minutes)
    count       = number of requests in current window
    t           = current timestamp
    window_start= start time of current window
)
window_start = floor(t / window_size) * window_size
ex.:
    window_size = 60 sec
    t = 125 sec → window_start = 120 sec

if count < limit:
    allow request
    count += 1
else:
    reject request

Count Reset Formula:
if t >= window_start + window_size:
    count = 0
    window_start = floor(t / window_size) * window_size
```

- issue:
  burst problem at boundary.
  1.10 to 1.20 - 5  
  1.20 to 1.30 - 5  
  But If we checkout 1.15 to 1.25, we will have 10 requests.

## Sliding window log algorithm
- he fixed window counter algorithm has a major issue: it allows more requests to go through at the edges of a window.  
- The Sliding Window Log algorithm is a rate limiting mechanism that tracks each individual request timestamp and decides whether to allow or reject a new request based on a moving time window.
- very accurate rate limiting, but uses more memory than other approaches.

ex. Allow 100 requests per 1 minute per user  
We must ensure:
- In any continuous 60-second window, a user cannot make more than 100 requests.
- Not just fixed minute blocks — but any rolling 60 seconds.
- means 1.00 to 2.00 - 100 request allowed  
  2.00 to 3.00 - 100 request allowed  
  also in 1.15 to 2.15 - 100 request allowed

Working:
1. Maintain a list (log) of timestamps of each request.
2. When a new request comes:
   - Get current time (now)
   - Remove all timestamps older than now - window_size
   - Check remaining timestamps count
   - If count < limit → allow request
   - Else → reject request

ex. 3 requests per 10 seconds
- Time 1s → Request → Allowed
- Time 3s → Request → Allowed
- Time 7s → Request → Allowed
- Time 8s → Request → Rejected ❌ (already 3 in last 10 sec)
- Time 12s → Request → Allowed ✅

At time 12:
- Window = 12 - 10 = 2
- Remove timestamps < 2
- So 1s request is removed
- Now we only have 3s, 7s
- Count = 2 → allow new request

formula:
```
1. Remove timestamps < (t - W)
2. If len(timestamps) < L
    allow request
    append current timestamp (t)
else
    reject request
```

Time & Space Complexity
- Time Complexity: Worst case O(n) per request (cleaning old logs)
- Space Complexity: O(n) where n = max number of requests in window

## Sliding window counter algorithm
- hybrid approach that combines the fixed window counter and sliding window log.
- best algorithm
- Instead of storing every request, we store only two counters:
  - Current window counter
  - Previous window counter
- Then we calculate a weighted average.

ex. Limit = 100 per 60 sec
- Previous window had 80 requests
- Current window has 20 requests
- We are 30 seconds into current window
- weight = (60 - 30) / 60 = 0.5
- effective = 80 * 0.5 + 20 = 60 
- Since 60 < 100, request is allowed

ex.2;
- 1.00 to 2.00 - 60 requests
- 2.00 to 2.30 - 20 requests
- We are 30 seconds into current window
- weight = (60 - 30) / 60 = 0.5 
- effective = 60 * 0.5 + 20 = 50 
- Since 50 < 100, request is allowed

formula:
```
L       = request limit
W       = window size
t       = current time
t0      = start time of current window
Δt      = t - t0   (elapsed time in current window)
C_prev  = request count in previous window
C_curr  = request count in current window

Weight of previous window:
weight = (W - Δt) / W

Effective request count:
effective = (C_prev * ((W - Δt) / W)) + C_curr

Allow condition:
if effective < L
    allow request
else
    reject request
```

## Rate limiter in a distributed environment
for Scabale high throught distributed system, we can use Redis to store the rate limit data.

store:
- INCR: It increases the stored counter by 1.
- EXPIRE: It sets a timeout for the counter. If the timeout expires, the counter is automatically deleted.

But, There are two challenges:
- Race condition
- Synchronization issue

