# LinkedIn Post — Designing Idempotency for In-Flight Retries: Handling the Race Conditions We Usually Ignore

_Generated: 2026-07-20_

---

Most engineers think idempotency is as simple as a quick read-before-write check in their database. But what happens when the first request is still running, and the client times out and retries? 🚨 This is where the most expensive double-charging bugs hide in production.

To build a truly resilient system, you cannot treat idempotency as a binary "exists or doesn't exist" check. You must track three distinct states for any given idempotency key: IN_PROGRESS, SUCCESS, and FAILED. Think of a busy restaurant kitchen where a waiter brings a duplicate ticket because they are worried about a delay. A smart chef doesn’t cook a second identical steak; they see the first ticket is already on the grill, tell the waiter to wait, and serve both from that single preparation. In-flight retries require this exact same coordination—we must transition the state atomically and force concurrent retries to block and wait for the original execution to resolve. 🍳

I once saw a payment gateway integration melt down during a Black Friday flash sale because of this exact race condition. The third-party gateway started experiencing 8-second latencies, causing client apps to timeout at 5 seconds and immediately retry. Because the original threads were still executing in the background, the database had no record of a completed transaction yet, leading to thousands of duplicate charges and a massive cleanup effort. If you are actively preparing for senior roles or designing these kinds of high-concurrency systems, I've found that hands-on LLD practice at https://javalld.com bridges exactly this gap between high-level theory and real-world implementation. Learning to orchestrate these transitions safely using tools like Redis or PostgreSQL locking is what separates junior implementation from senior system design. 💻

The correct mental model is the atomic lock-and-create pattern. When a request comes in, you must use an atomic database operation—like a Redis SET NX or a SQL INSERT with ON CONFLICT—to claim the key. If Thread A wins, it executes the business logic and saves the response. If Thread B loses the race, it must poll the state of that key until Thread A either completes the work or fails and releases the lock. This prevents duplicate executions entirely while safely returning the cached response to the retried client once it is ready. 🔒

Read the full breakdown here → https://javalld.com/blog/designing-idempotency-for-in-flight-retries-handling-the-race-conditions-we-usua

#SystemDesign #SoftwareArchitecture #Java #Concurrency #BackendDevelopment
