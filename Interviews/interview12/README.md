# DevOps Interview Session 12
## Performance Tuning & Optimization (3-7 Years)

---

### Question 1: Application Performance Bottlenecks

**Interviewer:** Your API response time degraded from 100ms to 2s. How do you troubleshoot?

**Candidate:** Start with Application Insights or APM tool. Check:
1. **Dependency calls** - database, external APIs timing
2. **Database queries** - N+1 problems, missing indexes
3. **Memory** - garbage collection pauses
4. **CPU** - inefficient algorithms
5. **Network** - latency to dependencies

Found issue: N+1 query loading user profiles in loop. Fixed with eager loading, response time back to 100ms.

---

### Question 2: Database Query Optimization

**Interviewer:** Optimize a slow SQL query.

**Candidate:**
```sql
-- Slow query (3 seconds)
SELECT u.*, o.*, p.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
LEFT JOIN products p ON o.product_id = p.id
WHERE u.created_at > '2024-01-01'
AND u.status = 'active';

-- Optimization steps:
-- 1. Check execution plan
EXPLAIN ANALYZE ...;

-- 2. Add indexes
CREATE INDEX idx_users_created_status ON users(created_at, status);
CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_product ON orders(product_id);

-- 3. Select only needed columns
SELECT 
  u.id, u.name, u.email,
  o.id as order_id, o.total,
  p.name as product_name
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
LEFT JOIN products p ON o.product_id = p.id
WHERE u.created_at > '2024-01-01'
AND u.status = 'active';

-- Result: 3s -> 50ms
```

---

### Question 3: Caching Strategy Implementation

**Interviewer:** Design multi-layer caching for an e-commerce API.

**Candidate:**
```javascript
// Layer 1: In-memory cache (fastest)
const NodeCache = require('node-cache');
const memCache = new NodeCache({ stdTTL: 60 });

// Layer 2: Redis (shared across instances)
const redis = require('redis');
const redisClient = redis.createClient();

// Layer 3: CDN (static content)
// CloudFront caches at edge locations

async function getProduct(productId) {
  // Try memory cache
  let product = memCache.get(productId);
  if (product) return product;
  
  // Try Redis
  product = await redisClient.get(`product:${productId}`);
  if (product) {
    product = JSON.parse(product);
    memCache.set(productId, product);
    return product;
  }
  
  // Database (cache miss)
  product = await db.query('SELECT * FROM products WHERE id = ?', [productId]);
  
  // Populate caches
  await redisClient.setex(`product:${productId}`, 3600, JSON.stringify(product));
  memCache.set(productId, product);
  
  return product;
}

// Cache invalidation on update
async function updateProduct(productId, data) {
  await db.query('UPDATE products SET ... WHERE id = ?', [productId]);
  
  // Invalidate all layers
  memCache.del(productId);
  await redisClient.del(`product:${productId}`);
  // CDN purge via API
}
```

---

### Question 4: Container Resource Optimization

**Interviewer:** Your pods are being OOMKilled. How do you fix?

**Candidate:**
```yaml
# Before - no limits
spec:
  containers:
  - name: app
    image: myapp:v1
    # Gets killed when memory spike

# After - proper limits
spec:
  containers:
  - name: app
    image: myapp:v1
    resources:
      requests:
        memory: "256Mi"  # Guaranteed
        cpu: "250m"
      limits:
        memory: "512Mi"  # Max before OOM
        cpu: "500m"      # Throttled if exceeded
```

**Interviewer:** How did you determine the right values?

**Candidate:** Ran load tests, monitored with Prometheus, analyzed p95 memory usage. Set requests at p50, limits at p95 + 20% buffer. Also fixed memory leak in code using heap snapshots.

---

### Question 5-40: Performance & Optimization

**Q5:** CDN configuration best practices?
**A:** Cache static assets, set proper Cache-Control headers, edge caching for API (carefully), geo-distribution, cache invalidation strategy.

**Q6:** Load balancer optimization?
**A:** Connection pooling, keepalive, health checks tuning, session affinity if needed, SSL termination at LB.

**Q7:** Connection pooling configuration?
**A:** Pool size = ((core_count * 2) + effective_spindle_count). Monitor active vs idle connections, tune based on workload.

**Q8:** Memory leak detection?
**A:** Heap dumps, memory profiling, track object allocation, compare snapshots over time, use weak references.

**Q9:** CPU optimization techniques?
**A:** Profile hot paths, optimize algorithms (O(n²) → O(n log n)), parallelize work, reduce context switches, caching.

**Q10:** I/O optimization strategies?
**A:** Async I/O, buffering, batch operations, SSD over HDD, ReadyBoost, memory-mapped files.

**Q11:** Network latency reduction?
**A:** CDN, compression (gzip/brotli), HTTP/2, keep-alive, DNS prefetch, connection reuse.

**Q12:** Kubernetes HPA tuning?
**A:** Appropriate metrics (CPU, memory, custom), stabilization window, scale-up/down policies, min/max replicas.

**Q13:** Database connection pooling?
**A:** HikariCP for Java, pgBouncer for PostgreSQL. Pool size tuning, connection timeout, idle timeout, leak detection.

**Q14:** Image optimization?
**A:** Compression, WebP format, lazy loading, responsive images, CDN delivery, caching headers.

**Q15:** API response time optimization?
**A:** Database indexes, caching, pagination, field filtering, compression, async processing for slow operations.

**Q16:** Garbage collection tuning?
**A:** Choose GC algorithm (G1GC, ZGC), heap size, young/old generation ratio, GC logging, analyze pauses.

**Q17:** Docker image size reduction?
**A:** Multi-stage builds, alpine base, remove dev dependencies, .dockerignore, layer optimization.

**Q18:** Batch processing optimization?
**A:** Parallel processing, chunking, streaming, database bulk inserts, error handling, progress tracking.

**Q19:** Redis performance tuning?
**A:** Pipelining, connection pooling, persistence vs performance, maxmemory policy, key expiration.

**Q20:** Elasticsearch optimization?
**A:** Shard sizing, index lifecycle, query optimization, aggregation tuning, bulk indexing, refresh interval.

**Q21:** Message queue optimization?
**A:** Batch size, prefetch count, parallel consumers, dead letter queues, message TTL.

**Q22:** Frontend performance?
**A:** Code splitting, tree shaking, minification, lazy loading, service workers, critical CSS.

**Q23:** Database index strategy?
**A:** Analyze query patterns, composite indexes, covering indexes, index maintenance, avoid over-indexing.

**Q24:** Microservices communication optimization?
**A:** gRPC over REST, connection reuse, circuit breakers, bulkheading, timeouts, retries with backoff.

**Q25:** Storage performance tuning?
**A:** Premium SSD, file system selection, RAID configuration, partition alignment, I/O scheduler.

**Q26:** Kubernetes network performance?
**A:** CNI choice (Cilium for BPF), network policies overhead, service mesh tradeoffs, host networking for latency-critical.

**Q27:** Load testing methodology?
**A:** Baseline, ramp-up, sustained peak, spike test, soak test, stress test until failure.

**Q28:** API rate limiting strategy?
**A:** Token bucket algorithm, sliding window, per-user limits, burst allowance, 429 responses with Retry-After.

**Q29:** Static content optimization?
**A:** Minification, bundling, compression, cache headers, CDN, asset versioning.

**Q30:** Database read replica strategy?
**A:** Route reads to replicas, monitor replication lag, handle eventual consistency, connection string routing.

**Q31:** Monitoring overhead reduction?
**A:** Sample high-cardinality metrics, aggregation, metric retention policies, efficient exporters.

**Q32:** Log performance impact?
**A:** Appropriate log levels in production, async logging, structured logs, log sampling for high-volume.

**Q33:** Kubernetes resource quotas?
**A:** Namespace limits, LimitRange for defaults, prevent resource starvation, priorityClass for important workloads.

**Q34:** Application startup time?
**A:** Lazy initialization, parallel loading, connection pre-warming, readiness vs liveness probes.

**Q35:** Caching invalidation strategies?
**A:** TTL-based, event-driven, cache-aside pattern, write-through, write-behind, versioned keys.

**Q36:** Auto-scaling optimization?
**A:** KEDA for event-driven, cluster autoscaler for nodes, vertical pod autoscaler, predictive scaling.

**Q37:** Build pipeline optimization?
**A:** Docker layer caching, parallel stages, artifact reuse, incremental builds, distributed builds.

**Q38:** Security scanning performance?
**A:** Cache scan results, scan base images separately, parallel scans, exclude known-good files.

**Q39:** Test execution optimization?
**A:** Parallel execution, test categorization, fail fast, mock external dependencies, test data management.

**Q40:** Cost optimization strategies?
**A:** Reserved instances, spot instances, auto-shutdown dev/test, rightsizing, storage lifecycle policies.

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)


