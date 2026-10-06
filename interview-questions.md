# Backend Architecture Interview Questions & Scenarios

## System Design Questions

### 1. URL Shortening Service
**Requirements:**
- Generate short URLs from long URLs
- Handle high redirect traffic
- Custom short URLs option
- Analytics on clicks

**Key Components:**
- Hash generation (Base62 encoding)
- Database for URL mapping
- Cache for popular URLs
- Load balancer for redirect traffic
- Analytics pipeline

**Follow-up Questions:**
- How would you handle collisions?
- What's your caching strategy?
- How do you ensure availability?
- How would you scale to billions of URLs?

### 2. Social Media Feed
**Requirements:**
- Generate personalized feeds for users
- Handle millions of concurrent users
- Real-time updates
- Support posts, likes, comments

**Key Components:**
- Fan-out on write (pre-compute feeds)
- Graph database for social graph
- Cache for hot feeds
- Message queue for real-time updates
- CDN for media content

**Follow-up Questions:**
- How do you handle feed generation for celebrities?
- What's your strategy for read vs write optimization?
- How do you ensure consistency?
- How would you handle trending topics?

### 3. Ride-Sharing App
**Requirements:**
- Match riders with drivers
- Real-time location tracking
- Surge pricing during peak hours
- Payment processing

**Key Components:**
- Geospatial indexing
- Real-time location updates
- Dynamic pricing algorithm
- Payment gateway integration
- Rating system

**Follow-up Questions:**
- How do you handle real-time location updates?
- What's your matching algorithm?
- How do you handle payment failures?
- How would you scale during rush hour?

## Technical Deep Dive Questions

### Database Design

**Q: When would you choose SQL vs NoSQL?**
- **SQL**: Financial transactions, user data, relationships matter
- **NoSQL**: Big data, unstructured data, high write throughput
- **Hybrid**: User profiles in SQL, activity logs in NoSQL

**Q: How would you design database sharding?**
- **Horizontal partitioning** based on user ID hash
- **Consistent hashing** for even distribution
- **Rebalancing strategy** for hot spots
- **Cross-shard queries** handling

**Q: Explain database indexing strategies**
- **B-tree indexes** for range queries
- **Hash indexes** for exact matches
- **Composite indexes** for multi-column queries
- **Covering indexes** to avoid table scans

### Caching Strategies

**Q: Design a multi-layer caching strategy**
- **Browser cache**: Static assets, long TTL
- **CDN cache**: Geographic distribution
- **Application cache**: Redis for hot data
- **Database cache**: Query result caching

**Q: How would you handle cache invalidation?**
- **TTL-based**: Automatic expiration
- **Event-driven**: Invalidate on data changes
- **Write-through**: Update cache on writes
- **Manual**: Admin-triggered invalidation

**Q: What's your cache warming strategy?**
- **Pre-load popular content**
- **Background refresh jobs**
- **Predictive caching based on patterns**
- **User-specific cache warming**

### Message Queues

**Q: When would you use Kafka vs RabbitMQ?**
- **Kafka**: High throughput, event streaming, log aggregation
- **RabbitMQ**: Complex routing, reliable delivery, task queues
- **Hybrid**: RabbitMQ for tasks, Kafka for events

**Q: How would you handle message ordering?**
- **Single partition per ordering key**
- **Sequence numbers in messages**
- **Consumer-side ordering buffer**
- **Idempotent processing**

**Q: What's your dead letter queue strategy?**
- **Retry with exponential backoff**
- **Manual inspection and reprocessing**
- **Alert on dead letter accumulation**
- **Automatic archival for compliance**

## Scalability Scenarios

### 1. E-commerce Flash Sale
**Scenario**: 1M users trying to buy 1000 items in 5 minutes

**Challenges**:
- Inventory management
- Payment processing
- Database locking
- Network congestion

**Solutions**:
- **Queue-based processing**: Buffer purchase requests
- **Redis for inventory**: Fast atomic operations
- **Circuit breakers**: Prevent cascading failures
- **CDN for static content**: Reduce load

### 2. Live Streaming Platform
**Scenario**: 10M concurrent viewers during major event

**Challenges**:
- Video distribution
- Real-time chat
- Low latency
- Geographic distribution

**Solutions**:
- **Edge computing**: Distribute video processing
- **WebSockets for chat**: Real-time communication
- **Adaptive bitrate**: Adjust quality based on bandwidth
- **Multi-CDN strategy**: Redundant distribution

### 3. Social Media Viral Content
**Scenario**: Post goes viral, 100K likes per second

**Challenges**:
- Database write throughput
- Feed generation
- Notification system
- Analytics processing

**Solutions**:
- **Async processing**: Queue likes for batch processing
- **Fan-out on write**: Pre-compute affected feeds
- **Rate limiting**: Protect downstream services
- **Stream processing**: Real-time analytics

## Performance Optimization

### Database Optimization

**Q: How would you optimize slow queries?**
1. **Analyze execution plan** - Identify bottlenecks
2. **Add appropriate indexes** - Based on query patterns
3. **Rewrite queries** - Eliminate subqueries, optimize joins
4. **Denormalize data** - Trade space for performance
5. **Partition tables** - Reduce scan size

**Q: What's your connection pooling strategy?**
- **HikariCP**: High-performance connection pool
- **Pool sizing**: Based on concurrency requirements
- **Timeout configuration**: Prevent connection exhaustion
- **Monitoring**: Track pool utilization

### API Performance

**Q: How would you optimize API response time?**
- **Response caching**: Cache frequent requests
- **Request batching**: Combine multiple operations
- **Compression**: Gzip responses
- **Pagination**: Limit result sets
- **Async processing**: Background heavy operations

**Q: Rate limiting implementation?**
- **Token bucket algorithm**: Allow bursts
- **Redis-based counters**: Distributed rate limiting
- **Hierarchical limits**: User, endpoint, global
- **Graceful degradation**: Queue vs reject

## Security & Reliability

### Authentication & Authorization

**Q: JWT vs Session-based authentication?**
- **JWT**: Stateless, good for microservices
- **Sessions**: Server-side, easier revocation
- **Hybrid**: JWT for auth, sessions for sensitive ops

**Q: How would you implement OAuth 2.0?**
- **Authorization code flow**: Web applications
- **PKCE**: Mobile/single-page apps
- **Client credentials**: Service-to-service
- **Token refresh**: Maintain session continuity

### High Availability

**Q: How would you achieve 99.99% uptime?**
- **Multi-region deployment**: Geographic redundancy
- **Load balancers**: Distribute traffic, health checks
- **Circuit breakers**: Prevent cascading failures
- **Auto-scaling**: Handle traffic variations
- **Disaster recovery**: Backup and restore procedures

**Q: Database failover strategy?**
- **Primary-replica setup**: Automatic promotion
- **Connection failover**: Client retry logic
- **Data consistency**: Synchronous replication
- **Monitoring**: Replication lag alerts

## Monitoring & Observability

### Metrics & Alerting

**Q: What metrics would you monitor?**
- **RED**: Rate, Errors, Duration (application)
- **USE**: Utilization, Saturation, Errors (infrastructure)
- **Business metrics**: User engagement, conversion rates
- **SLI/SLO**: Service level indicators/objectives

**Q: Alerting strategy?**
- **Threshold-based**: CPU > 80%, memory > 90%
- **Anomaly detection**: Unusual patterns
- **Business impact**: Revenue-impacting alerts
- **Escalation**: Tiered alert routing

### Debugging & Troubleshooting

**Q: How would you debug a production issue?**
1. **Check metrics**: Identify anomalies
2. **Review logs**: Find error patterns
3. **Trace requests**: Follow user journey
4. **Reproduce issue**: Staging environment
5. **Implement fix**: Test and deploy

**Q: Post-mortem process?**
- **Timeline**: Detailed event sequence
- **Root cause**: Identify contributing factors
- **Action items**: Prevent recurrence
- **Blameless culture**: Focus on systems, not people

## Code Architecture

### Microservices Design

**Q: When would you split a monolith?**
- **Team growth**: Multiple teams working on same codebase
- **Deployment frequency**: Different services need different schedules
- **Technology diversity**: Different requirements for different parts
- **Fault isolation**: One component affecting others

**Q: How would you handle distributed transactions?**
- **Saga pattern**: Sequence of compensating transactions
- **Event sourcing**: Rebuild state from events
- **Two-phase commit**: Coordinated commit (rarely used)
- **Best effort**: Accept eventual consistency

### API Design

**Q: REST vs GraphQL?**
- **REST**: Simple, cacheable, standardized
- **GraphQL**: Flexible, single endpoint, reduced over-fetching
- **Hybrid**: GraphQL for complex queries, REST for simple operations

**Q: API versioning strategy?**
- **URL versioning**: /v1/users, /v2/users
- **Header versioning**: Accept: application/vnd.api+json;version=1
- **Parameter versioning**: ?version=1
- **Backward compatibility**: Maintain old versions

## Practical Coding Questions

### 1. Rate Limiter Implementation
```python
class RateLimiter:
    def __init__(self, max_requests, time_window):
        self.max_requests = max_requests
        self.time_window = time_window
        self.requests = {}
    
    def is_allowed(self, user_id):
        now = time.time()
        user_requests = self.requests.get(user_id, [])
        
        # Remove old requests
        user_requests = [req_time for req_time in user_requests 
                        if now - req_time < self.time_window]
        
        if len(user_requests) >= self.max_requests:
            return False
        
        user_requests.append(now)
        self.requests[user_id] = user_requests
        return True
```

### 2. Cache Implementation
```python
class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = {}
        self.order = []
    
    def get(self, key):
        if key in self.cache:
            self.order.remove(key)
            self.order.append(key)
            return self.cache[key]
        return None
    
    def put(self, key, value):
        if key in self.cache:
            self.order.remove(key)
        elif len(self.cache) >= self.capacity:
            oldest = self.order.pop(0)
            del self.cache[oldest]
        
        self.cache[key] = value
        self.order.append(key)
```

### 3. Message Queue Consumer
```python
class MessageConsumer:
    def __init__(self, queue, max_workers=5):
        self.queue = queue
        self.max_workers = max_workers
        self.workers = []
    
    def start(self):
        for i in range(self.max_workers):
            worker = threading.Thread(target=self._process_messages)
            worker.daemon = True
            worker.start()
            self.workers.append(worker)
    
    def _process_messages(self):
        while True:
            try:
                message = self.queue.get(timeout=1)
                self._handle_message(message)
                self.queue.task_done()
            except Empty:
                continue
            except Exception as e:
                logger.error(f"Error processing message: {e}")
    
    def _handle_message(self, message):
        # Process message logic here
        pass
```

## Behavioral Questions

### Team Collaboration
- "How do you handle disagreements about technical decisions?"
- "Describe a time you had to convince your team to adopt a new technology"
- "How do you ensure code quality in a fast-paced environment?"

### Problem Solving
- "Tell me about a complex technical problem you solved"
- "How do you approach debugging production issues?"
- "Describe a time you had to make a trade-off between performance and features"

### Learning & Growth
- "How do you stay updated with new technologies?"
- "What's the most challenging technical concept you've learned recently?"
- "How do you approach learning a new codebase?"

## Tips for Success

### During the Interview
1. **Clarify requirements** before starting design
2. **Think out loud** to show your thought process
3. **Consider trade-offs** for every decision
4. **Ask questions** about constraints and scale
5. **Structure your answer** with clear sections

### Preparation Strategy
1. **Practice whiteboarding** system designs
2. **Study common patterns** and when to use them
3. **Understand trade-offs** between different approaches
4. **Review your past projects** for examples
5. **Stay current** with industry trends and best practices

### Common Mistakes to Avoid
1. **Jumping to solutions** without understanding requirements
2. **Ignoring constraints** like budget or timeline
3. **Over-engineering** simple problems
4. **Not considering edge cases** and failure scenarios
5. **Forgetting about monitoring** and observability
