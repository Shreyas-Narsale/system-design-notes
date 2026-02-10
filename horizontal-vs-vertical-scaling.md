## Vertical Scaling vs Horizontal Scaling

### Vertical Scaling (Scale Up)
- Add more power (CPU, RAM, etc.) to a single server.
- Common for monolithic applications.

Useful for:
- Apps that store state locally:
  - File-based systems
  - In-memory caches
  - Local session storage
- Low-traffic applications
- More in-memory operations

Drawbacks:
- Single point of failure
- Downtime during scaling
- Poor fault tolerance (hardware failure, OS crash, or disk issue brings down the entire system)

---

### Horizontal Scaling (Scale Out)
- Add more servers to scale the application.
- Often done using Kubernetes with multi-node clusters.

Benefits:
- Better for large-scale applications
- Handles high traffic

Drawbacks:
- Requires stateless application design
- Data consistency challenges
- Harder debugging and monitoring
- Higher operational cost at small scale

