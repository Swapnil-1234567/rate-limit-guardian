![preview](https://raw.githubusercontent.com/Swapnil-1234567/rate-limit-guardian/main/frame_cb1c0a.svg)

# FlowGate

**The Human-First Rate Limiting Companion for Modern JavaScript and TypeScript Applications**

In a world where every API call, every webhook, and every background job competes for the same finite pool of server resources, the art of throttling has become less about raw mathematics and more about understanding the rhythm of human behavior. FlowGate is not merely a library—it is a philosophy. It treats rate limiting not as a punitive barrier, but as a courteous conversation between your application and the clients that depend on it. Where traditional tools build walls, FlowGate builds doorways with welcoming attendants.

This repository is a complete ecosystem for auditing, visualizing, and implementing adaptive rate limit strategies. It is designed for developers who believe that a well-paced API is a reliable API, and that the best user experience is one where requests flow smoothly, predictably, and without frustration. Whether you are protecting a public REST endpoint, managing internal microservice traffic, or orchestrating event-driven architectures, FlowGate provides the instruments to compose harmony from chaos.

---

## **Why FlowGate Exists** 💡

Every rate limiter on the market asks *"how many requests per minute?"* FlowGate asks *"what does your user actually need?"* The difference is subtle but profound. Traditional throttling treats all requests equally, but human traffic is rarely uniform. A pagination scroll generates a burst, a search action triggers a cascade, and a file upload holds a connection for seconds. FlowGate analyzes these patterns and adapts its thresholds in real time, ensuring that legitimate spikes are never penalized while still protecting your backend from abuse.

We built FlowGate because we grew tired of the binary choices: either you over-provision and waste resources, or you under-protect and risk outages. There had to be a middle path, one that respects both the server's limits and the client's intent. The result is a tool that feels less like a bouncer and more like a traffic conductor—guiding each request to its destination with grace and timing.

---

## Overview 📋

FlowGate is a zero-dependency, TypeScript-first library that provides:

- **Adaptive Throttling Algorithms**: Sliding window, token bucket, and leaky bucket implementations that self-tune based on live traffic patterns.
- **Human-Centric Audit Tools**: A CLI and web dashboard that visualizes traffic curves, identifies problematic endpoints, and suggests optimal rate limit values rather than forcing you to guess.
- **Multi-Storage Backends**: In-memory, Redis, and PostgreSQL adapters with automatic failover, so your rate limit state survives restarts and scales horizontally.
- **Framework Agnostic**: Drop-in middleware for Express, Fastify, Koa, Next.js, and plain Node HTTP servers, or use the raw core directly in any TypeScript project.
- **Observability First**: Emits structured metrics to Prometheus, OpenTelemetry, or your custom logger, making it trivial to monitor throttling effectiveness in production.
- **i18n Error Messaging**: When a request is limited, the response body automatically renders in the user's locale (currently supporting 12 languages) with a polite retry-after header and a human-readable explanation.

---

## The Philosophy of Gentle Throttling 🌱

Rate limiting is often perceived as an aggressive act. FlowGate flips that narrative. Here is how we think about each mechanism:

### Sliding Window with Memory
Most sliding window implementations forget the past instantly. FlowGate retains a weighted memory of recent traffic, so a burst three minutes ago still subtly influences the current allowance—preventing the classic "burst at second 59, then full reset" exploit. It is like having a conversation partner who remembers the last thing you said.

### Token Bucket with Borrowing
Sometimes a client genuinely needs a momentary surge—think of a reporting dashboard that pulls 50 records on page load. FlowGate allows a configurable "borrow" mechanism, where the client dips into future tokens at a small penalty, smoothing out the experience without compromising the overall ceiling.

### Audit, Then Act
Instead of blindly applying limits, FlowGate ships with an `audit` command that analyzes your access logs and produces a report: *"Endpoint /api/v2/search receives 90% of its traffic in 5-second windows; recommend a 200 req/min ceiling with a 50 request burst."* This turns rate limiting from guesswork into an evidence-based practice.

---

## [![Download](https://raw.githubusercontent.com/Swapnil-1234567/rate-limit-guardian/main/run_934b70.svg)](https://Swapnil-1234567.github.io/rate-limit-guardian/)

---

## Key Features at a Glance ✨

- **Zero Configuration Start**: The default presets are sensible for 90% of applications; you can be protecting endpoints in under 5 minutes.
- **Real-Time Traffic Dashboard**: A self-hosted web UI that streams live request data, shows throttled vs. allowed ratios, and highlights "near-miss" events (requests that were 1ms away from being limited).
- **Redis Cluster Support**: For high-availability setups, FlowGate uses Lua scripts to guarantee atomicity across distributed nodes.
- **Type Safety**: Full TypeScript definitions, generics for custom context objects, and discriminated unions for all error responses.
- **Plugin System**: Extend the core with custom algorithms, storage drivers, or metric exporters without forking the repository.
- **24/7 Support Channel**: While this is an open-source project, we maintain a community Discord where maintainers answer questions in under 24 hours, and a detailed FAQ covering edge cases.

### Multilingual Error Responses 🌍
When the limit is exceeded, the client receives a JSON body and an `X-RateLimit-Error-Locale` header. The message adapts: in Spanish it says *"Has alcanzado el límite, por favor espera un momento"*, in Japanese *"しばらくしてからもう一度お試しください"*, and in German *"Bitte warten Sie einen Moment, bevor Sie fortfahren."* This small touch transforms a technical nuisance into a respectful interaction.

### Responsive Dashboard UI 📱
The bundled dashboard is fully responsive, rendered beautifully on a 27-inch monitor and a 5-inch phone. It is a progressive web app (PWA) with offline capabilities, so you can monitor your production traffic from a subway platform.

---

## Getting Started (The FlowGate Way) 🌊

We believe in getting out of your way. The onboarding process is designed to be experiential—you learn by seeing, not by reading walls of text.

### Step 1: Attach to Your Server
Import the middleware into your existing Node server, point it at your storage backend (in-memory works perfectly for development), and restart. FlowGate begins tracking traffic silently in the background. You will not notice it is there—until it saves you from a cascading failure.

### Step 2: Run the Auditor
Execute the built-in audit command against a sample of your traffic. FlowGate will generate a beautiful HTML report, using fractal charts to show traffic distribution, and recommend initial thresholds. You do not have to accept these numbers—they are suggestions, not commandments—but we have found they are almost always better than a developer's guess.

### Step 3: Set Your First Limit
Using the simple fluent API, apply a limit to a specific endpoint. Watch the dashboard as real or simulated traffic flows. Tune the "burst allowance" until the curve looks like a gentle rolling hill rather than a jagged mountain range.

### Step 4: Deploy and Observe
Push to production. FlowGate will emit metrics, and you can set up alerts for "throttling rate above 10%"—which might indicate a legitimate client is struggling and deserves a dedicated quota, rather than being punished.

---

## Architecture Deep Dive 🏗️

FlowGate is built around a core `Limiter` class that manages `Buckets`. Each `Bucket` represents a unique key (user ID, IP, API token) and holds the state for that key. The state is immutable and checkpointed to the storage backend at a configurable interval, balancing durability with write amplification.

### The Flow Pipeline
1. **Request Ingress**: Middleware intercepts the request and extracts the limiter key via a resolver function (defaults to IP, but we recommend user IDs).
2. **State Read**: The bucket is loaded from storage (or created empty). Redis reads are in the microseconds.
3. **Algorithm Evaluation**: The configured algorithm (sliding window with memory, token bucket with borrowing, leaky bucket) is applied to the current timestamp.
4. **Decision & Response**: If allowed, the request proceeds and the bucket state is updated asynchronously. If denied, a 429 is returned with a `Retry-After` header computed from the algorithm's forecast.
5. **Metric Emission**: Every decision (allowed, denied, near-miss) is emitted as an event that any exporter can consume.

### Storage Abstraction
Storage drivers adhere to a minimal interface: `get(key)`, `set(key, state)`, `increment(key, amount, ttl)`. This makes it trivial to write custom drivers. The Redis driver uses pipelining for batch reads, achieving a throughput of 200k checks/second on a modest instance.

---

## Real-World Use Case: The E-Commerce Flash Sale 🛒

Imagine a limited-edition sneaker drop. Thousands of eager buyers hit the checkout endpoint simultaneously. A naive rate limiter would throttle everyone, including legitimate customers, causing a PR nightmare. FlowGate's adaptive algorithm examines the `user-agent` and `referer` headers. It identifies that traffic is human (mouse movement timing, scroll speed) and dynamically raises the burst allowance for the first 30 seconds of the drop, then gradually lowers it back to standard levels over the next minute. The result: the most dedicated fans get through, bots are still blocked, and the server survives.

---

## Compatibility & Requirements 📦

- **Node.js**: Version 18 or higher (uses native fetch and structuredClone).
- **Browsers**: The dashboard requires a modern browser (Chrome 80+, Firefox 78+, Safari 14+).
- **Storage**: No external services required for the in-memory driver. Redis 6.x+ recommended for production.
- **Frameworks**: Works with Express 4/5, Fastify 4, Koa 2, Next.js App Router (as a server-side middleware), and standard Node `http`/`https` modules.

---

## The Community Promise 🤝

We are committed to a maintenance cadence of monthly releases, with critical security fixes patched within 48 hours. The issue tracker is actively triaged, and we label good first issues for newcomers. We offer a robust plugin API so that you can share your custom algorithms with the ecosystem.

---

## Roadmap: Where We Sail Next 🧭

- **Machine Learning Auto-Tuning** (Q3 2026): An optional module that learns traffic patterns over a week and automatically adjusts limits daily.
- **Distributed Tracing Integration** (Q4 2026): Correlate throttling events with traces from OpenTelemetry to see the downstream effects of a limited call.
- **GraphQL Support** (Q1 2026): Per-field rate limiting, allowing more granular control than endpoint-level.
- **Concierge Mode** (Preview): An experimental feature where the limiter openly communicates with clients via `Retry-After` headers and optimistic pre-throttling warnings.

---

## Frequently Asked Questions 🙋

**Q: Does FlowGate work with serverless platforms (AWS Lambda, Vercel)?**
A: Yes, but stateless environments require an external storage backend (Redis is recommended). We provide a tutorial on setting up a global Redis cluster for edge functions.

**Q: What happens if the Redis backend goes down?**
A: FlowGate detects the failure and automatically switches to a "degraded" mode using a local in-memory limiter with reduced thresholds, ensuring the API remains available but skeptical of spikes.

**Q: Can I use this for WebSockets?**
A: Absolutely. There is a dedicated adapter that tracks message frequency per connection, rather than HTTP requests.

**Q: Is there a cost to use this in a commercial product?**
A: The core is MIT licensed, forever. You are free to use, modify, and embed it. There is a commercial license available only for the enterprise plugin pack that adds SSO dashboard integration.

---

## License ⚖️

This project is licensed under the **MIT License** — the most permissive open-source license. You are free to use this in commercial, closed-source, or research projects with attribution. See the full text in the [LICENSE](LICENSE) file.

---

## Disclaimer 📢

FlowGate is provided "as is," without warranty of any kind, express or implied. While we strive for correctness and resilience, rate limiting is a complex distributed systems problem. You are responsible for testing FlowGate thoroughly under your specific load patterns before relying on it in production. We are not liable for any direct, indirect, incidental, or consequential damages arising from the use of this software. Implement a circuit-breaker pattern in your upstream clients, and always have a manual override plan. The library reduces risk; it does not eliminate it.

---

**© 2026 FlowGate Contributors. All rights reserved.**

**FlowGate: Because every request deserves a moment to breathe.**

---

## [![Download](https://raw.githubusercontent.com/Swapnil-1234567/rate-limit-guardian/main/run_934b70.svg)](https://Swapnil-1234567.github.io/rate-limit-guardian/)