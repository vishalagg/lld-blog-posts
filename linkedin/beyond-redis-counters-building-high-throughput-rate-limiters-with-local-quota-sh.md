# LinkedIn Post — Beyond Redis Counters: Building High-Throughput Rate Limiters with Local Quota Sharding

_Generated: 2026-05-13_

---

Your rate limiter might be the very thing killing your application’s performance. 🛑

The default move for most engineers is to reach for Redis and increment a global key. It works fine during a quiet Tuesday, but the moment a marketing campaign hits or a botnet finds your API, that single Redis key becomes the hottest spot in your infrastructure. I’ve seen production systems—much like what Uber faced before building their Gringotts service—where p99 latency spiked from 20ms to over 500ms while CPU and memory stayed at a comfortable 15%. The bottleneck wasn't the application logic; it was the network round-trips and the lock contention of thousands of service instances trying to update the exact same counter at once. We often optimize our business logic to the microsecond, only to throw it all away by making every single request wait on a remote network call just to "ask for permission."

Think of it like a busy restaurant kitchen. If every chef had to walk to the central walk-in fridge to grab a single egg for every omelet, the kitchen would grind to a halt at the fridge door. Instead, each station has a small cooler. They grab a carton of 12 eggs at the start of the shift and only return to the central fridge when they run low. In engineering, we call this Local Quota Sharding. By letting each service instance "borrow" a batch of tokens and store them in a local AtomicLong, you move the heavy lifting to a background thread. If you're actively preparing for senior roles, I've found that hands-on LLD practice at https://javalld.com bridges exactly this gap between theory and system interviews. It forces you to think about these concurrency trade-offs and distributed state management before they bite you in production. 🍳

The magic happens when you shift the "hot path"—the code that executes for every user request—from a blocking network operation to a wait-free memory operation using CAS (Compare-And-Swap). Your request handling thread checks the local bucket, decrements it, and moves on—all in nanoseconds. Meanwhile, a background task monitors the consumption rate and proactively fetches the next batch from the global store before the local supply hits zero. This strategy does trade off some "strictness"—you might slightly over-provision if several nodes fetch batches simultaneously—but in high-throughput systems, the jump in availability and the reduction in tail latency are almost always worth the trade-off of "eventual" rate limiting. 🚀

Read the full breakdown here → https://javalld.com/blog/beyond-redis-counters-building-high-throughput-rate-limiters-with-local-quota-sh

#SystemDesign #Java #Concurrency #SoftwareArchitecture #Redis
