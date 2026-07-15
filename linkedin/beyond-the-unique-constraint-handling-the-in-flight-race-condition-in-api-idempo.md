# LinkedIn Post — Beyond the Unique Constraint: Handling the In-Flight Race Condition in API Idempotency

_Generated: 2026-07-15_

---

Have you ever realized that a standard database unique constraint is completely useless against concurrent, in-flight API requests? 🛑 Most engineers treat idempotency as a simple "read-then-write" database check, assuming a unique key will catch duplicates. But when a slow downstream payment gateway or third-party API is involved, that window of vulnerability opens wide. If a client retries a request before the first thread has finished executing and saved the result, your database has no record of it yet. Both threads see a green light, both proceed to the gateway, and your customer gets charged twice.

To solve this, we have to stop viewing idempotency as a static lookup table and start treating it as a dynamic state machine with a "Work-in-Progress" pegboard. Imagine a busy restaurant kitchen where a customer orders a custom cake; if the cashier only checks the "Finished Cakes" shelf and sees nothing, they might mistakenly start baking a brand-new cake from scratch. Instead, they need a pegboard showing that the cake is currently *in-progress*, forcing subsequent inquiries to wait. In distributed systems, this means atomically reserving the "In-Progress" state (using Redis `SETNX` or a database lock) the millisecond a request arrives. If you're actively preparing for senior roles, I've found that hands-on LLD practice at https://javalld.com bridges exactly this gap between theoretical system design and writing thread-safe, resilient code. ⏳

I once saw this exact race condition trigger a severe cascading failure in a high-volume fintech platform during a major promotional sale. The mobile app automatically retried failed network requests after 150ms, but our payment processor was taking up to 2 seconds to respond under heavy load. Because we relied on a simple unique constraint that was only written *after* a successful payment, thousands of duplicate requests bypassed the check, resulting in over $50,000 in double-charges and a massive backlog of manual refunds. The correct mental model is to lock the key immediately upon entry, and if a second thread hits an in-progress lock, force it to poll and block rather than failing or proceeding. When the first thread finishes, it updates the record to "Success" and caches the response, allowing the waiting thread to cleanly wake up and return the cached result. 💡

Read the full breakdown here → https://javalld.com/blog/beyond-the-unique-constraint-handling-the-in-flight-race-condition-in-api-idempo

#SoftwareArchitecture #SystemDesign #Java #Concurrency #BackendEngineering
