# Complete Backend Interview Preparation Handbook

Tailored for JavaScript, Node.js, Express.js, REST APIs, JWT/RBAC, MySQL, MongoDB, Razorpay, Inngest, Docker, logging, monitoring, AI/Text-to-SQL backend safety, and backend system design.

Last updated: 2026-07-17

**Important honesty rule:** this handbook separates project-backed claims from recommended production improvements. If you cannot prove that a feature was implemented in your code or resume, say "In a production version, I would..." instead of pretending it already exists.

## How To Use This Guide

**For an interview happening very soon**

Read the Interview Strategy, Backend Mastery Path, HTTP/API fundamentals, Express architecture, authentication, security, database basics, and your project grilling sections first. Practice the strong answers out loud. Do not try to memorize every code snippet; memorize the reasoning pattern: request flow, validation, authorization, database correctness, failure handling, and tests.

**For deep long-term mastery**

Work topic by topic. For every major concept, cover the definition, why it exists, internals, implementation, failure cases, testing, performance, alternatives, trade-offs, interview answer, exercise, and solution. Build the capstone gradually.

**For project-specific interview preparation**

Use the project grilling section. Mark each answer as one of three categories: actually implemented, recommended production improvement, or theoretical knowledge. Interviewers respect honest boundaries more than fake certainty.

**For hands-on implementation practice**

Use the exercises and capstone. Implement one module at a time: auth, queries, payments, webhooks, background jobs, tests, Docker, and OpenAPI. Then explain your code as if an interviewer is reading it line by line.

## Table Of Contents

1. Interview Strategy
2. Backend Mastery Path
3. JavaScript For Backend Interviews
4. HTTP And Networking Fundamentals
5. API Fundamentals And REST Design
6. Node.js Internals
7. Express.js Architecture And Production Module
8. TypeScript For Backend Interviews
9. Authentication And Authorization
10. Backend Security And OWASP API Top 10
11. SQL And MySQL
12. MongoDB
13. API Testing Handbook
14. Background Jobs, Messaging, And Inngest
15. Razorpay And Payment Backend Design
16. Docker, Deployment, And Production Engineering
17. Logging, Monitoring, And Observability
18. Alternative Communication Styles
19. AI And Text-to-SQL Backend Safety
20. Backend System Design Playbook
21. Resume-Based Project Grilling
22. Hands-On Practice Lab
23. Capstone Production-Style Backend
24. Practical Checklists
25. Mock Interview Bank: 200+ Questions
26. Official References
27. Final Interview Mantra

# 1. Interview Strategy

## The Answer Shape That Works

Strong backend answers usually follow this structure:

```text
Simple definition
-> why it exists
-> internal/request flow
-> implementation detail
-> trade-off
-> failure case
-> how to test
-> project connection
```

Example:

```text
JWT is a signed token containing claims like user ID and role. It exists so an API can authenticate requests without a server-side session lookup on every request. In Express, login creates a short-lived access token; auth middleware verifies it and attaches req.user. The trade-off is revocation, so in production I would use short access-token expiry, refresh-token rotation, and server-side refresh-token records. I would test missing token, invalid token, expired token, wrong role, and ownership checks.
```

## How To Avoid Fabricating

Use these labels in project answers:

- **Actually implemented:** "In my project, I implemented JWT authentication and RBAC middleware."
- **Recommended production improvement:** "In a production version, I would add refresh-token rotation and reuse detection."
- **Theoretical knowledge:** "I understand that RS256 is useful when multiple services need to verify tokens using a public key, although I have not needed it in my project."

## What Interviewers Are Really Checking

- Can you trace a request from client to database and back?
- Do you know where validation, auth, business logic, and DB access belong?
- Can you select correct status codes?
- Can you prevent common security bugs?
- Can you reason about indexes, transactions, and consistency?
- Can you test failure paths, not only happy paths?
- Can you discuss production concerns without over-engineering?
- Can you defend your resume projects honestly?

# 2. Backend Mastery Path

This replaces the old time-based plan. Use it as a map, not a calendar.

## Must Know

You must be able to explain these in every backend interview:

- Client-server model, DNS, TCP, TLS, HTTP lifecycle.
- HTTP methods, safe methods, idempotency, status codes.
- REST resource design, filtering, sorting, pagination, versioning, error contracts.
- Express request lifecycle, middleware order, route matching, controllers, services, repositories.
- JavaScript async model, promises, async/await, event loop basics.
- JWT authentication, RBAC, ownership authorization, 401 vs 403.
- SQL basics, joins, indexes, transactions, ACID.
- MongoDB documents, embedding vs referencing, indexes.
- Basic API testing with Supertest or Postman.
- Security basics: injection, CORS, CSRF, mass assignment, rate limiting, sensitive logs.
- Docker basics and environment configuration.

## Must Implement

You should be able to code these without copying blindly:

- Express app with `app.js` and `server.js`.
- Request validation using Zod.
- Centralized error handling with `AppError`.
- Auth middleware that verifies JWT and sets `req.user`.
- RBAC middleware and service-level ownership checks.
- CRUD endpoint with pagination and filtering.
- SQL queries using placeholders.
- Mongoose schema with indexes and safe response shaping.
- Unit tests for services and integration tests for routes.
- Idempotency record for payment-like APIs.
- Razorpay signature verification and webhook raw-body verification.
- Inngest event function with idempotent steps.
- Dockerfile and Docker Compose for API plus database.

## Deep Follow-Ups

Use these when interviewers push deeper:

- HTTP caching: `Cache-Control`, `ETag`, `Last-Modified`, `Vary`, conditional requests.
- HTTP/1.1 vs HTTP/2 vs HTTP/3.
- Event loop phases, microtasks, `process.nextTick`, `setImmediate`, thread pool.
- Streams, buffers, backpressure, worker threads, memory leaks.
- Refresh-token rotation, reuse detection, session/device management.
- OWASP API Security Top 10.
- SQL isolation levels, locks, deadlocks, query plans, N+1.
- MongoDB aggregation, partial/TTL indexes, `populate` trade-offs, `lean()`.
- Transactional outbox, at-least-once delivery, DLQ, retry jitter.
- Production metrics, p95/p99 latency, tracing, SLO/SLA/error budgets.

## Advanced/Optional

Recognize and compare these even if you have not implemented them:

- GraphQL, gRPC, WebSockets, Server-Sent Events, long polling.
- OAuth 2.0 and OpenID Connect internals.
- Kafka, RabbitMQ, BullMQ compared with Inngest.
- Distributed locks and cache stampede prevention.
- Sharding, read replicas, replication lag.
- Kubernetes, blue-green deployments, canary deployments.
- Multi-tenant authorization and row-level security.
- Advanced Text-to-SQL guardrails and query sandboxing.

# 3. JavaScript For Backend Interviews

JavaScript is your primary interview language. You should sound comfortable with both the language and its backend runtime behavior.

## Values, References, And Mutation

Primitives such as strings, numbers, booleans, `null`, `undefined`, `bigint`, and symbols are handled by value. Objects, arrays, functions, maps, sets, and dates are reference values.

```javascript
const user = { name: "Vineet" };
const sameUser = user;
sameUser.name = "Updated";

console.log(user.name); // Updated
```

A common backend bug is mutating request-derived objects and accidentally reusing them. Prefer explicit DTO shaping:

```javascript
function toPublicUser(user) {
  return {
    id: user.id,
    name: user.name,
    email: user.email,
    role: user.role
  };
}
```

## `var`, `let`, And `const`

- `var` is function-scoped and hoisted as `undefined`.
- `let` is block-scoped and cannot be used before initialization.
- `const` is block-scoped and prevents reassignment, but object contents can still mutate.

Use `const` by default, `let` when reassignment is needed, and avoid `var` in modern backend code.

## Closures In Express

Closures are functions that keep access to variables from their lexical scope.

```javascript
export function authorize(...allowedRoles) {
  return function authorizationMiddleware(req, res, next) {
    if (!req.user) {
      return res.status(401).json({
        error: { code: "AUTHENTICATION_REQUIRED", message: "Please log in" }
      });
    }

    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        error: { code: "FORBIDDEN", message: "You do not have permission" }
      });
    }

    next();
  };
}
```

`allowedRoles` remains available because of closure.

## Promises And Async/Await

`async/await` is syntax over promises. A rejected promise becomes a thrown error at the `await` line.

```javascript
async function getUserOrThrow(userId, userRepository) {
  const user = await userRepository.findById(userId);
  if (!user) {
    throw new AppError(404, "USER_NOT_FOUND", "User not found");
  }
  return user;
}
```

Do not forget to `await` promises whose errors you expect to catch:

```javascript
try {
  await emailService.sendWelcomeEmail(user.email);
} catch (error) {
  logger.warn({ error }, "Welcome email failed");
}
```

## Promise Combinators

- `Promise.all`: fail-fast, good for independent required operations.
- `Promise.allSettled`: collect all results, good for optional side effects.
- `Promise.race`: first settled promise wins.
- `Promise.any`: first fulfilled promise wins.

```javascript
const [profile, orders] = await Promise.all([
  userRepository.findPublicProfile(userId),
  orderRepository.findRecentByUserId(userId)
]);
```

## Error Objects

In modern JavaScript, caught values can be anything. Treat unknown errors carefully:

```javascript
function getErrorMessage(error) {
  if (error instanceof Error) {
    return error.message;
  }
  return "Unknown error";
}
```

## Common JavaScript Backend Mistakes

- Forgetting `await` and returning before work completes.
- Using `forEach` with async callbacks and expecting it to await.
- Mutating request body and passing it across layers.
- Using `JSON.parse` on untrusted huge payloads without body size limits.
- Swallowing errors in catch blocks.
- Keeping unbounded data in global arrays or maps.

## Exercise

Predict the output:

```javascript
async function run() {
  console.log("A");
  Promise.resolve().then(() => console.log("B"));
  await Promise.resolve();
  console.log("C");
}

run();
console.log("D");
```

**Solution**

```text
A
D
B
C
```

`run()` logs `A`, schedules a microtask for `B`, reaches `await`, and returns a pending promise to the caller. The outer script logs `D`. The microtask queue then runs the `.then` callback (`B`) and the continuation after `await` (`C`) in order.

# 4. HTTP And Networking Fundamentals

## Complete HTTP Request Lifecycle

        **Simple Definition**

        The HTTP request lifecycle is the journey from a client deciding to call a URL to the backend returning a response.

        **Why It Exists**

        It exists because web systems are distributed. The browser/client, DNS, transport connection, TLS, proxies, load balancers, application server, and database each contribute to latency and failure modes.

        **Internal Working**

        A client parses the URL, resolves the hostname through DNS, opens a transport connection, performs TLS for HTTPS, sends HTTP bytes, receives status/headers/body, and may reuse the connection through keep-alive. In production the request often passes through CDNs, reverse proxies, API gateways, and load balancers before Express sees it.

        **Request Or Execution Flow**

        React fetch -> DNS cache/recursive resolver -> TCP handshake or QUIC setup -> TLS handshake -> HTTP request -> reverse proxy -> Express middleware -> controller -> service -> repository -> DB -> JSON response -> client parses response.

        **Real Node.js/Express Implementation**

        ```javascript
app.get("/api/v1/users/:userId", authenticate, async (req, res, next) => {
  try {
    const user = await userService.getPublicUser(req.params.userId, req.user);
    return res.status(200).json({ data: user });
  } catch (error) {
    next(error);
  }
});
```

        **Example Request And Response**

        ```http
GET /api/v1/users/42 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer <access-token>
X-Request-ID: req_123

HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: private, no-store
X-Request-ID: req_123

{
  "data": {
    "id": "42",
    "name": "Vineet"
  }
}
```

        **Common Mistakes**

        - Thinking the request starts at Express.
- Ignoring DNS/TLS/proxy failures.
- Not setting timeouts.
- Trusting proxy headers without configuring trusted proxies.

        **Security Concerns**

        - Always use HTTPS for token-bearing APIs.
- Do not log authorization headers.
- Trust `X-Forwarded-*` only from known proxies.
- Use request-size limits before JSON parsing large bodies.

        **Failure Scenarios**

        - DNS failure means no request reaches your server.
- TCP/TLS timeout causes client-visible failure.
- Proxy timeout can happen while Node is still processing.
- DB timeout should become a controlled 503/500, not a hanging request.

        **Testing Strategy**

        - Use Supertest for Express behavior.
- Use integration tests for auth and status codes.
- Use load tests to observe latency and timeout behavior.
- Use proxy/local environment tests if relying on `trust proxy`.

        **Performance Concerns**

        - TLS handshakes and no keep-alive increase latency.
- Large JSON bodies increase parsing time and memory.
- Slow DB calls dominate request latency.
- Compression saves bandwidth but costs CPU.

        **Alternatives**

        - HTTP/1.1, HTTP/2, and HTTP/3 transport choices.
- REST, GraphQL, RPC, gRPC for API style.
- Polling, SSE, WebSockets for update delivery.

        **Trade-Offs**

        The API semantics may stay the same while transport changes. HTTP/2/3 improve connection behavior, but they do not fix poor endpoint design, missing indexes, or unsafe authorization.

        **Strong Interview Answer**

        ```text
        When a frontend calls an API, the browser resolves DNS, opens a TCP connection or QUIC connection, performs TLS for HTTPS, sends the HTTP request, and the request passes through infrastructure before Express handles middleware, controller, service, repository, and database work. The response returns with status, headers, and body. I think about failures and latency at every step.
        ```

        **Interviewer Follow-Ups With Answers**

        - **What part of this flow does Express control?** Only the application-level handling after the request reaches the Node process.
- **Where can a timeout occur?** Client, proxy/load balancer, Node server, database, or external provider.

        **Connection To Your Projects**

        This connects to every project: ResolveAI query creation, Razorpay payment callbacks, and Text-to-SQL all begin as HTTP requests and need validation, authorization, and controlled failure handling.

        **Small Exercise**

        Draw the lifecycle for `POST /api/v1/queries` and mark where authentication, validation, DB write, Inngest event publication, and response happen.

        **Exercise Solution / Expected Reasoning**

        Authentication and validation happen in Express middleware; DB write happens in service/repository; event publication should happen after successful persistence or through an outbox; response should return 201/202 with a stable contract.

## HTTP Versions, Headers, Caching, And Conditional Requests

        **Simple Definition**

        HTTP defines request/response semantics; versions such as HTTP/1.1, HTTP/2, and HTTP/3 change transport behavior more than API resource design.

        **Why It Exists**

        These features exist to reduce latency, bandwidth, and unnecessary server work while keeping clients and caches correct.

        **Internal Working**

        HTTP/1.1 supports persistent connections and textual messages. HTTP/2 multiplexes streams over a TCP connection. HTTP/3 uses QUIC over UDP, reducing transport-level head-of-line blocking. Headers carry metadata. Caches reuse responses according to `Cache-Control`; validators like `ETag` and `Last-Modified` let clients ask whether a stale response changed.

        **Request Or Execution Flow**

        Client sends `Accept`, `Accept-Encoding`, and cache validators -> server selects representation -> server returns `Content-Type`, `Cache-Control`, `ETag`, optional compressed body -> later client sends `If-None-Match` -> server returns 304 if unchanged.

        **Real Node.js/Express Implementation**

        ```javascript
import crypto from "node:crypto";

app.get("/api/v1/public-courses/:courseId", async (req, res, next) => {
  try {
    const course = await courseService.getPublicCourse(req.params.courseId);
    const body = JSON.stringify({ data: course });
    const etag = `"${crypto.createHash("sha256").update(body).digest("hex")}"`;

    res.set("ETag", etag);
    res.set("Cache-Control", "public, max-age=60");
    res.set("Vary", "Accept-Encoding");

    if (req.get("If-None-Match") === etag) {
      return res.status(304).end();
    }

    return res.type("application/json").send(body);
  } catch (error) {
    next(error);
  }
});
```

        **Example Request And Response**

        ```http
GET /api/v1/public-courses/CS101
Accept: application/json
If-None-Match: "abc123"

HTTP/1.1 304 Not Modified
ETag: "abc123"
Cache-Control: public, max-age=60
```

        **Common Mistakes**

        - Caching private user data in shared caches.
- Forgetting `Vary` when response changes by `Accept-Language` or encoding.
- Sending a body with 304 or 204.
- Assuming HTTP/2 changes REST design rules.

        **Security Concerns**

        - Use `Cache-Control: no-store` for tokens, payment responses, and sensitive personalized data.
- Avoid caching authorization-dependent responses publicly.
- Do not include secrets in headers or URLs.
- Be careful with cookies and shared caches.

        **Failure Scenarios**

        - Stale cache after update.
- ETag generated from wrong representation.
- Compressed response without correct `Content-Encoding`.
- Proxy strips or rewrites headers.

        **Testing Strategy**

        - Test cache headers for sensitive endpoints.
- Test 304 conditional flow.
- Test `Content-Type` and unsupported media types.
- Test that login responses use `no-store`.

        **Performance Concerns**

        - Caching reduces DB load.
- Compression reduces bandwidth but costs CPU.
- ETag generation from large bodies can cost CPU.
- HTTP/2 multiplexing helps many concurrent requests.

        **Alternatives**

        - Application-level Redis caching.
- CDN caching.
- Browser cache.
- No caching for sensitive dynamic APIs.

        **Trade-Offs**

        HTTP caching is powerful for public or stable resources, but dangerous for personalized data. Application caches such as Redis give more control but need invalidation logic.

        **Strong Interview Answer**

        ```text
        HTTP headers are part of the API contract. I use `Content-Type` and `Accept` for representation, `Cache-Control` for reuse policy, `ETag` or `Last-Modified` for conditional requests, and security headers for protection. I avoid shared caching for sensitive authenticated responses.
        ```

        **Interviewer Follow-Ups With Answers**

        - **When use `ETag`?** For validating whether a representation changed and for optimistic concurrency in some APIs.
- **What does `Vary` do?** It tells caches which request headers affect the cached representation.

        **Connection To Your Projects**

        Public plant/course pages in Vrikshami-like projects could use caching; authenticated profile, payment, and admin APIs should usually use `no-store`.

        **Small Exercise**

        Design caching headers for `GET /api/v1/profile` and `GET /api/v1/public-courses/CS101`.

        **Exercise Solution / Expected Reasoning**

        `/profile` should use `Cache-Control: no-store` because it is personalized. Public course detail can use `public, max-age=60` with ETag if stale data for a minute is acceptable.

# 5. API Fundamentals And REST Design

## REST API Design, Pagination, Bulk APIs, And OpenAPI

        **Simple Definition**

        REST API design means exposing resources through predictable URLs, HTTP methods, request/response schemas, and error contracts.

        **Why It Exists**

        It exists so clients can use APIs consistently without knowing backend internals.

        **Internal Working**

        Resources represent domain objects. Methods express operation semantics. Query parameters refine collection reads. Status codes express result. OpenAPI documents the contract in machine-readable form.

        **Request Or Execution Flow**

        Client sends endpoint request -> route matches method/path -> middleware authenticates/validates -> controller calls service -> service enforces rules -> repository queries -> response envelope returned.

        **Real Node.js/Express Implementation**

        ```javascript
router.get(
  "/queries",
  authenticate,
  validate(listQueriesSchema),
  asyncHandler(async (req, res) => {
    const result = await queryService.listQueries({
      actor: req.user,
      filters: req.validated.query
    });

    res.status(200).json({
      data: result.items,
      pagination: result.pagination
    });
  })
);
```

        **Example Request And Response**

        ```http
GET /api/v1/queries?status=open&priority=high&page=1&limit=20&sort=-createdAt
Authorization: Bearer <token>

HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    {
      "id": "query_501",
      "title": "Unable to register",
      "status": "open",
      "priority": "high"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "totalItems": 37,
    "totalPages": 2
  }
}
```

        **Common Mistakes**

        - Using verbs everywhere like `/getUser`.
- Returning unlimited lists.
- Returning inconsistent error shapes.
- Breaking old clients without versioning.
- Allowing arbitrary fields including secrets.

        **Security Concerns**

        - Check ownership for IDs in route params.
- Whitelist filters/sort fields.
- Limit page size.
- Use `Idempotency-Key` for retryable writes.
- Never expose password hashes, refresh tokens, or internal notes.

        **Failure Scenarios**

        - Large offset query becomes slow.
- Bulk API partially succeeds.
- Long-running request times out.
- Client retries a POST and creates duplicates.
- Old client breaks after field rename.

        **Testing Strategy**

        - Contract tests for response shape.
- Validation tests for filters/page/limit.
- Authorization tests for ownership.
- Idempotency tests for POST operations.
- OpenAPI schema validation where useful.

        **Performance Concerns**

        - Cursor pagination for large changing datasets.
- Stable sort fields and indexes.
- Bulk APIs may need async jobs.
- Field selection can reduce payload but needs safe whitelisting.

        **Alternatives**

        - GraphQL for client-selected graph data.
- RPC/gRPC for action-oriented service calls.
- Webhooks for provider callbacks.

        **Trade-Offs**

        REST is simple and widely understood, but it can over-fetch or under-fetch. GraphQL gives flexible client queries but moves complexity to schema/resolvers. RPC/gRPC gives strong service contracts but is less browser-native.

        **Strong Interview Answer**

        ```text
        I design REST APIs around resources, use HTTP methods for operations, query parameters for collection controls, stable pagination, consistent errors, and OpenAPI for contracts. I introduce new versions only for breaking changes and keep backward compatibility for existing clients.
        ```

        **Interviewer Follow-Ups With Answers**

        - **Offset or cursor pagination?** Offset for simple admin tables; cursor for large or frequently changing data.
- **When return 202?** For long-running async work such as AI processing or report generation.

        **Connection To Your Projects**

        ResolveAI query listing needs filtering, sorting, pagination, ownership checks, and possibly cursor pagination if the dataset grows.

        **Small Exercise**

        Design endpoints for assigning a query, listing assigned queries, and adding a reply.

        **Exercise Solution / Expected Reasoning**

        `GET /api/v1/queries?assignedTo=me&status=open&page=1&limit=20`, `POST /api/v1/queries/:queryId/assignments`, and `POST /api/v1/queries/:queryId/replies`. Assignment is a command/resource creation; reply is a child resource.

## API Design Exercise: Online Course Enrollment

**Attempt First**

Design APIs for listing courses, enrolling a student, listing enrollments, cancelling enrollment, and handling duplicate enrollment.

**Solved Design**

```http
GET /api/v1/courses?department=computer&page=1&limit=20
GET /api/v1/courses/:courseId
POST /api/v1/courses/:courseId/enrollments
GET /api/v1/courses/:courseId/enrollments
DELETE /api/v1/courses/:courseId/enrollments/:studentId
```

Enrollment request:

```json
{
  "studentId": "stu_42"
}
```

Status codes:

```text
201 -> enrollment created
401 -> missing/invalid auth
403 -> cannot enroll this student
404 -> course or student not found
409 -> already enrolled
422 -> enrollment closed or invalid data
```

Database constraint:

```sql
UNIQUE KEY uniq_course_student (course_id, student_id)
```

This unique key is not just an optimization. It protects correctness under concurrent duplicate requests.

# 6. Node.js Internals

## V8, Node.js, libuv, Event Loop, And Microtasks

        **Simple Definition**

        Node.js is a JavaScript runtime. V8 executes JavaScript; Node provides backend APIs; libuv provides the event loop and async I/O support.

        **Why It Exists**

        This architecture lets JavaScript handle many concurrent I/O operations without creating one thread per request.

        **Internal Working**

        V8 runs JS on the main thread. Async network I/O is usually handled by the OS. Some file system, DNS, crypto, and zlib operations use libuv's thread pool. The event loop processes phases such as timers, poll, check, and close callbacks. Promise microtasks and `process.nextTick` run with special priority between phases/operations.

        **Request Or Execution Flow**

        Synchronous JS -> schedule timers/promises/I/O -> call stack empties -> nextTick queue -> promise microtasks -> event loop phases -> callbacks execute -> more microtasks -> repeat.

        **Real Node.js/Express Implementation**

        ```javascript
import fs from "node:fs";

console.log("A");

setTimeout(() => console.log("timeout"), 0);
setImmediate(() => console.log("immediate"));
Promise.resolve().then(() => console.log("promise"));
process.nextTick(() => console.log("nextTick"));

fs.readFile(new URL(import.meta.url), () => {
  console.log("io");
  setTimeout(() => console.log("io-timeout"), 0);
  setImmediate(() => console.log("io-immediate"));
});

console.log("B");
```

        **Example Request And Response**

        ```text
Common first lines:
A
B
nextTick
promise

At top level, timeout vs immediate order can vary.
Inside the I/O callback, setImmediate usually runs before setTimeout(0).
```

        **Common Mistakes**

        - Saying Node creates one thread per request.
- Running CPU-heavy loops in request handlers.
- Recursive `process.nextTick` starving I/O.
- Assuming top-level `setTimeout(0)` always beats `setImmediate`.

        **Security Concerns**

        - CPU-blocking code can create denial of service.
- ReDoS blocks the event loop.
- Unbounded buffers can exhaust memory.
- Unhandled errors can crash the process.

        **Failure Scenarios**

        - Thread pool saturation delays fs/crypto/dns operations.
- Unhandled rejection crashes or destabilizes process depending config/version.
- Long poll callback delays timers.
- Memory leak increases GC pressure and latency.

        **Testing Strategy**

        - Output-prediction tests for event loop understanding.
- Unit tests for async error paths.
- Load tests that expose event-loop delay.
- Use fake timers carefully for timer logic.

        **Performance Concerns**

        - Use `perf_hooks.monitorEventLoopDelay` or APM for event-loop lag.
- Move CPU work to worker threads or jobs.
- Stream large data instead of buffering.
- Tune libuv thread pool only with measurement.

        **Alternatives**

        - Worker threads for CPU-bound JS.
- Child processes for isolated processes.
- Multiple Node processes behind a load balancer.
- Native services for heavy compute.

        **Trade-Offs**

        Node is excellent for I/O-heavy APIs, but the main JS thread is a shared resource. The trade-off is simple concurrency for I/O, but CPU-heavy work must be isolated.

        **Strong Interview Answer**

        ```text
        Node.js uses V8 to run JavaScript, Node APIs for backend capabilities, and libuv for the event loop and async I/O. It can handle many concurrent requests because I/O does not block the main thread, but CPU-heavy JavaScript blocks the event loop, so I move such work to workers, child processes, or background jobs.
        ```

        **Interviewer Follow-Ups With Answers**

        - **What changed in Node 20/libuv 1.45 around timers?** Timers run after poll in each event-loop iteration, with compatibility behavior before entering the loop; this can affect `setImmediate`/timer timing.
- **Where do promise callbacks run?** In the microtask queue, after current synchronous work and before normal macrotask phase callbacks.

        **Connection To Your Projects**

        ResolveAI AI processing should not run CPU-heavy work in the API request. It should be a background job or external AI call with timeout and retry.

        **Small Exercise**

        Predict output for `console.log(1); setTimeout(...); Promise.resolve().then(...); process.nextTick(...); console.log(2);`.

        **Exercise Solution / Expected Reasoning**

        `1`, `2`, `nextTick`, promise callback, then timer. Synchronous code first, then nextTick queue, then promise microtasks, then timer phase.

## Streams, Buffers, Backpressure, Cancellation, Workers, And Graceful Shutdown

        **Simple Definition**

        Streams process data chunk by chunk; buffers store raw bytes; backpressure prevents producers from overwhelming consumers; cancellation stops work that no longer matters.

        **Why It Exists**

        These exist because APIs often handle files, network data, uploads, downloads, and long external calls without loading everything into memory.

        **Internal Working**

        Readable streams emit chunks. Writable streams accept chunks and return `false` when internal buffer is full. Producers should wait for `drain`. `AbortController` passes cancellation signals. Worker threads run CPU-heavy JS away from the main thread. Graceful shutdown stops accepting new work and closes resources.

        **Request Or Execution Flow**

        Request starts upload -> file stream writes to storage -> backpressure pauses reads -> client disconnect triggers abort -> server cleans partial upload. On SIGTERM -> server stops accepting -> waits in-flight -> closes DB/Redis -> exits.

        **Real Node.js/Express Implementation**

        ```javascript
import { pipeline } from "node:stream/promises";
import fs from "node:fs";

app.post("/api/v1/uploads", authenticate, async (req, res, next) => {
  const destination = `/safe/uploads/${crypto.randomUUID()}.bin`;

  try {
    await pipeline(req, fs.createWriteStream(destination));
    return res.status(201).json({ data: { stored: true } });
  } catch (error) {
    next(error);
  }
});

const server = app.listen(process.env.PORT || 3000);

process.on("SIGTERM", () => {
  server.close(async () => {
    await db.end();
    await redis.quit();
    process.exit(0);
  });
});
```

        **Example Request And Response**

        ```http
POST /api/v1/uploads
Content-Type: application/octet-stream
Content-Length: 1048576

HTTP/1.1 201 Created

{
  "data": {
    "stored": true
  }
}
```

        **Common Mistakes**

        - Using `fs.readFile` for huge files.
- Ignoring stream errors.
- Ignoring `.write()` backpressure.
- Leaving servers accepting traffic during shutdown.
- Not cancelling external calls when client disconnects.

        **Security Concerns**

        - Validate file type and size.
- Avoid path traversal in file names.
- Do not store uploads under user-provided names.
- Scan dangerous file types if needed.
- Use request-size limits.

        **Failure Scenarios**

        - Client disconnects mid-upload.
- Disk full.
- External API never responds.
- Worker crashes.
- Shutdown kills in-flight payment update.

        **Testing Strategy**

        - Test large-file path with streams.
- Test upload size limit.
- Test client abort where feasible.
- Test SIGTERM graceful shutdown in integration/staging.
- Test worker failure fallback.

        **Performance Concerns**

        - Streaming lowers memory usage.
- Backpressure prevents heap growth.
- Worker threads add overhead, so use only for CPU-heavy work.
- Graceful shutdown prevents failed in-flight requests during deploy.

        **Alternatives**

        - Direct-to-object-storage uploads.
- Background processing queue.
- Child process for isolation.
- External service for heavy processing.

        **Trade-Offs**

        Streams are more complex than buffers but scale better. Workers avoid event-loop blocking but require message passing and lifecycle management.

        **Strong Interview Answer**

        ```text
        For large files or long-running data transfer, I use streams and respect backpressure instead of buffering everything. For CPU-heavy work, I use workers or background jobs. For production shutdown, I stop accepting new requests, finish in-flight work, close dependencies, and then exit.
        ```

        **Interviewer Follow-Ups With Answers**

        - **What is backpressure?** A signal that the consumer cannot keep up, so the producer should slow down.
- **Why not use worker threads for database queries?** Database calls are I/O-bound; async I/O already handles them well.

        **Connection To Your Projects**

        Certificate generation after payment may be a background job. File uploads in a plant-adoption/certificate project should stream to storage and validate file metadata.

        **Small Exercise**

        Find the bug: an upload endpoint reads the entire file into memory before checking size.

        **Exercise Solution / Expected Reasoning**

        The endpoint can exhaust memory. Enforce size limits before/during reading, stream to storage, and reject oversized requests early.

## Node.js Output Prediction Drills

### Drill 1

```javascript
console.log("start");

setTimeout(() => console.log("timeout"), 0);
setImmediate(() => console.log("immediate"));

Promise.resolve().then(() => console.log("promise"));
process.nextTick(() => console.log("nextTick"));

console.log("end");
```

**Expected Reasoning**

`start` and `end` are synchronous. `process.nextTick` runs before promise microtasks. The top-level order between `setTimeout(0)` and `setImmediate` can vary because it depends on timing and event-loop entry.

### Drill 2

```javascript
import fs from "node:fs";

fs.readFile(new URL(import.meta.url), () => {
  setTimeout(() => console.log("timeout"), 0);
  setImmediate(() => console.log("immediate"));
});
```

**Expected Reasoning**

Inside an I/O callback, `setImmediate` normally runs before `setTimeout(0)` because the immediate callback is queued for the check phase after poll.

### Drill 3

```javascript
async function f() {
  console.log("A");
  await null;
  console.log("B");
}

console.log("C");
f();
console.log("D");
```

**Expected Output**

```text
C
A
D
B
```

# 7. Express.js Architecture And Production Module

## Express Request Lifecycle, Middleware, And Layered Architecture

        **Simple Definition**

        Express architecture organizes request handling into middleware, routes, controllers, services, repositories, and centralized errors.

        **Why It Exists**

        It exists to keep HTTP concerns separate from business rules and database details, making the backend testable and maintainable.

        **Internal Working**

        Express receives a request, runs application middleware in registration order, matches routers/routes in order, runs route middleware, calls handler/controller, and forwards errors to error middleware with four parameters. Express 5 automatically forwards rejected promises from async handlers; Express 4 commonly needs wrappers.

        **Request Or Execution Flow**

        Request -> `express.json` -> request ID -> logger -> rate limiter -> router -> auth -> validation -> controller -> service -> repository -> response -> finish logger. Error path -> `next(error)` -> error middleware.

        **Real Node.js/Express Implementation**

        ```javascript
// src/utils/asyncHandler.js
export function asyncHandler(handler) {
  return function wrapped(req, res, next) {
    Promise.resolve(handler(req, res, next)).catch(next);
  };
}

// src/utils/AppError.js
export class AppError extends Error {
  constructor(statusCode, code, message, details = []) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.details = details;
    this.isOperational = true;
  }
}
```

        **Example Request And Response**

        ```http
POST /api/v1/queries
Authorization: Bearer <student-token>
Content-Type: application/json

{
  "title": "Unable to register",
  "description": "Portal error",
  "category": "academic"
}

HTTP/1.1 201 Created

{
  "data": {
    "id": "query_501",
    "status": "submitted"
  }
}
```

        **Common Mistakes**

        - Putting business logic in routes.
- Forgetting `return` after response.
- Calling `next()` after response.
- Registering 404/error middleware before routes.
- Putting generic `/:id` route before `/search`.

        **Security Concerns**

        - Validate every input server-side.
- Do not trust `req.body.role`.
- Shape responses to avoid sensitive fields.
- Use rate limits for auth/payment/AI endpoints.
- Use file upload limits.

        **Failure Scenarios**

        - Middleware branch hangs request.
- Service throws unhandled error.
- Database timeout.
- JSON parse error.
- Route shadowing causes wrong handler.

        **Testing Strategy**

        - Unit test services with fake repositories.
- Integration test route with middleware.
- Test validation, auth, forbidden, conflict, not found, and database failure.
- Test error response shape.

        **Performance Concerns**

        - Avoid unnecessary middleware on hot paths.
- Keep controllers thin.
- Use indexes for repository queries.
- Add body size limits.
- Stream files instead of buffering.

        **Alternatives**

        - Fastify for higher-performance schema-driven APIs.
- NestJS for opinionated architecture.
- Koa/Hono for lighter middleware models.

        **Trade-Offs**

        Express is flexible and simple, but you must enforce structure yourself. Frameworks like NestJS impose patterns but add learning and abstraction.

        **Strong Interview Answer**

        ```text
        In Express, I keep routes thin, middleware handles cross-cutting concerns, controllers handle HTTP input/output, services contain business rules, and repositories handle database access. Errors go to centralized middleware so responses are consistent and logs include request IDs.
        ```

        **Interviewer Follow-Ups With Answers**

        - **Why separate app.js and server.js?** Tests can import app without listening on a port; server.js handles startup and dependencies.
- **How does Express recognize error middleware?** By the four-argument signature `(error, req, res, next)`.

        **Connection To Your Projects**

        Your ResolveAI and Razorpay APIs are easiest to defend if explained through route -> middleware -> controller -> service -> repository -> external provider/job.

        **Small Exercise**

        Fix a middleware that sends 401 but still reaches the controller.

        **Exercise Solution / Expected Reasoning**

        Add `return` before `res.status(401).json(...)` or use `return next(error)`. Do not call `next()` after sending the response.

## Complete Production-Style Express Module: Queries

This module is intentionally JavaScript-first. It shows consistent imports, defined helpers, validation, DTO shaping, ownership checks, repository boundaries, and tests later in the guide.

### Folder Structure

```text
src/
  app.js
  server.js
  config/env.js
  middleware/authenticate.js
  middleware/authorize.js
  middleware/errorHandler.js
  middleware/requestId.js
  middleware/validate.js
  modules/queries/query.routes.js
  modules/queries/query.controller.js
  modules/queries/query.service.js
  modules/queries/query.repository.js
  modules/queries/query.schema.js
  modules/queries/query.dto.js
  utils/AppError.js
  utils/asyncHandler.js
```

### `src/config/env.js`

```javascript
import { z } from "zod";

const envSchema = z.object({
  NODE_ENV: z.enum(["development", "test", "production"]).default("development"),
  PORT: z.coerce.number().int().positive().default(3000),
  JWT_ACCESS_SECRET: z.string().min(32),
  DATABASE_URL: z.string().min(1)
});

export const env = envSchema.parse(process.env);
```

### `src/app.js`

```javascript
import express from "express";
import helmet from "helmet";
import { requestId } from "./middleware/requestId.js";
import { errorHandler } from "./middleware/errorHandler.js";
import { notFound } from "./middleware/notFound.js";
import { queryRouter } from "./modules/queries/query.routes.js";

export function createApp(dependencies) {
  const app = express();

  app.use(helmet());
  app.use(express.json({ limit: "1mb" }));
  app.use(requestId);

  app.get("/health", (req, res) => res.status(200).json({ status: "ok" }));
  app.get("/ready", async (req, res) => {
    const dbReady = await dependencies.health.checkDatabase();
    return res.status(dbReady ? 200 : 503).json({
      status: dbReady ? "ready" : "not_ready"
    });
  });

  app.use("/api/v1/queries", queryRouter(dependencies));

  app.use(notFound);
  app.use(errorHandler);

  return app;
}
```

### `src/modules/queries/query.schema.js`

```javascript
import { z } from "zod";

export const createQuerySchema = z.object({
  body: z.object({
    title: z.string().trim().min(5).max(150),
    description: z.string().trim().min(10).max(5000),
    category: z.enum(["academic", "technical", "administrative", "other"])
  }),
  params: z.object({}),
  query: z.object({})
});

export const listQueriesSchema = z.object({
  body: z.object({}),
  params: z.object({}),
  query: z.object({
    status: z.enum(["submitted", "assigned", "in_progress", "resolved", "closed"]).optional(),
    priority: z.enum(["low", "medium", "high"]).optional(),
    page: z.coerce.number().int().positive().default(1),
    limit: z.coerce.number().int().positive().max(100).default(20),
    sort: z.enum(["createdAt", "-createdAt", "priority", "-priority"]).default("-createdAt")
  })
});
```

### `src/modules/queries/query.dto.js`

```javascript
export function toQueryDto(query) {
  return {
    id: String(query.id),
    title: query.title,
    description: query.description,
    category: query.category,
    status: query.status,
    priority: query.priority,
    createdBy: String(query.createdBy),
    assignedTo: query.assignedTo ? String(query.assignedTo) : null,
    createdAt: query.createdAt
  };
}
```

### `src/modules/queries/query.service.js`

```javascript
import { AppError } from "../../utils/AppError.js";
import { toQueryDto } from "./query.dto.js";

export function createQueryService({ queryRepository, eventPublisher }) {
  async function createQuery({ actor, input }) {
    if (actor.role !== "student" && actor.role !== "admin") {
      throw new AppError(403, "FORBIDDEN", "Only students or admins can create queries");
    }

    const query = await queryRepository.create({
      title: input.title,
      description: input.description,
      category: input.category,
      status: "submitted",
      priority: "medium",
      createdBy: actor.id
    });

    await eventPublisher.publish({
      id: `query-created-${query.id}`,
      name: "query/created",
      data: {
        queryId: String(query.id),
        studentId: String(actor.id)
      }
    });

    return toQueryDto(query);
  }

  async function listQueries({ actor, filters }) {
    const scopedFilters = { ...filters };

    if (actor.role === "student") {
      scopedFilters.createdBy = actor.id;
    }

    if (actor.role === "moderator") {
      scopedFilters.assignedTo = actor.id;
    }

    const result = await queryRepository.findMany(scopedFilters);

    return {
      items: result.items.map(toQueryDto),
      pagination: result.pagination
    };
  }

  async function getQuery({ actor, queryId }) {
    const query = await queryRepository.findById(queryId);

    if (!query) {
      throw new AppError(404, "QUERY_NOT_FOUND", "Query not found");
    }

    const isOwner = String(query.createdBy) === String(actor.id);
    const isAssignedModerator = String(query.assignedTo) === String(actor.id);
    const isAdmin = actor.role === "admin";

    if (!isOwner && !isAssignedModerator && !isAdmin) {
      throw new AppError(403, "FORBIDDEN", "You cannot access this query");
    }

    return toQueryDto(query);
  }

  return { createQuery, listQueries, getQuery };
}
```

### `src/modules/queries/query.routes.js`

```javascript
import express from "express";
import { authenticate } from "../../middleware/authenticate.js";
import { validate } from "../../middleware/validate.js";
import { asyncHandler } from "../../utils/asyncHandler.js";
import { createQuerySchema, listQueriesSchema } from "./query.schema.js";
import { createQueryService } from "./query.service.js";

export function queryRouter(dependencies) {
  const router = express.Router();
  const queryService = createQueryService(dependencies);

  router.get(
    "/",
    authenticate(dependencies),
    validate(listQueriesSchema),
    asyncHandler(async (req, res) => {
      const result = await queryService.listQueries({
        actor: req.user,
        filters: req.validated.query
      });

      res.status(200).json({
        data: result.items,
        pagination: result.pagination
      });
    })
  );

  router.post(
    "/",
    authenticate(dependencies),
    validate(createQuerySchema),
    asyncHandler(async (req, res) => {
      const query = await queryService.createQuery({
        actor: req.user,
        input: req.validated.body
      });

      res.status(201).json({ data: query });
    })
  );

  router.get(
    "/:queryId",
    authenticate(dependencies),
    asyncHandler(async (req, res) => {
      const query = await queryService.getQuery({
        actor: req.user,
        queryId: req.params.queryId
      });

      res.status(200).json({ data: query });
    })
  );

  return router;
}
```

### Common Debugging Exercises

- Route `/search` returns query by ID because `/:queryId` was registered first. Fix by registering `/search` before `/:queryId`.
- Validation sends 422 but controller still runs. Fix missing `return`.
- Error middleware returns stack trace in production. Fix by using safe messages and request IDs.
- `req.user` undefined in authorization middleware. Fix middleware order: authenticate before authorize.

# 8. TypeScript For Backend Interviews

TypeScript is not the main language of this handbook, but backend interviewers often ask it because Node.js teams use it heavily.

## Why TypeScript Is Used In Node Backends

TypeScript catches many mistakes before runtime:

- DTO shape mismatches.
- Repository return type mistakes.
- Missing fields in service calls.
- Unsafe `any` usage.
- Refactor errors across modules.

But TypeScript does **not** validate real HTTP input at runtime. Use Zod/Joi/Ajv for runtime validation.

## Primitive And Object Types

```typescript
const name: string = "Vineet";
const age: number = 22;
const active: boolean = true;
const tags: string[] = ["node", "api"];

type UserDto = {
  id: string;
  name: string;
  email: string;
  role: "student" | "moderator" | "admin";
};
```

## Interfaces Vs Type Aliases

```typescript
interface UserRepository {
  findById(id: string): Promise<UserDto | null>;
}

type ApiResponse<T> = {
  data: T;
  requestId: string;
};
```

Use interfaces for object contracts that may be implemented. Use type aliases for unions, intersections, utility compositions, and generic response shapes.

## Union, Intersection, And Narrowing

```typescript
type PaymentStatus = "created" | "pending" | "captured" | "failed";

type AuthenticatedRequest = Request & {
  user: {
    id: string;
    role: "student" | "admin";
  };
};

function canRefund(status: PaymentStatus) {
  return status === "captured";
}
```

## `unknown` Vs `any`

```typescript
function getMessage(error: unknown): string {
  if (error instanceof Error) {
    return error.message;
  }
  return "Unknown error";
}
```

`any` disables checking. `unknown` forces safe narrowing.

## Typed Express Request And Middleware

```typescript
import type { Request, Response, NextFunction } from "express";

type AuthUser = {
  id: string;
  role: "student" | "moderator" | "admin";
};

type AuthenticatedRequest = Request & {
  user: AuthUser;
};

function authorize(...roles: AuthUser["role"][]) {
  return function middleware(req: Request, res: Response, next: NextFunction) {
    const user = (req as AuthenticatedRequest).user;

    if (!user) {
      return res.status(401).json({
        error: { code: "AUTHENTICATION_REQUIRED", message: "Please log in" }
      });
    }

    if (!roles.includes(user.role)) {
      return res.status(403).json({
        error: { code: "FORBIDDEN", message: "Access denied" }
      });
    }

    next();
  };
}
```

In larger apps, use Express declaration merging carefully:

```typescript
declare global {
  namespace Express {
    interface Request {
      user?: AuthUser;
    }
  }
}
```

## Typed Environment Configuration

```typescript
import { z } from "zod";

const envSchema = z.object({
  NODE_ENV: z.enum(["development", "test", "production"]),
  JWT_ACCESS_SECRET: z.string().min(32),
  PORT: z.coerce.number().default(3000)
});

export const env = envSchema.parse(process.env);
export type Env = z.infer<typeof envSchema>;
```

## Repository Interfaces

```typescript
type CreateUserInput = {
  name: string;
  email: string;
  passwordHash: string;
  role: "student" | "admin";
};

interface UserRepository {
  findByEmail(email: string): Promise<UserDto | null>;
  create(input: CreateUserInput): Promise<UserDto>;
}
```

## Zod Runtime Validation Vs TypeScript Compile-Time Types

```typescript
const createUserSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  password: z.string().min(8)
});

type CreateUserBody = z.infer<typeof createUserSchema>;
```

TypeScript helps developers. Zod protects the running API from real client input.

## Common TypeScript Interview Questions

- **Why TypeScript?** Safer refactoring, clearer contracts, fewer runtime shape mistakes.
- **Does TypeScript replace validation?** No. It is erased at runtime.
- **`unknown` vs `any`?** `unknown` requires narrowing; `any` disables safety.
- **Generic use case?** Reusable `ApiResponse<T>` or `Repository<T>`.
- **Utility types?** `Pick`, `Omit`, `Partial`, `Required`, `Record`.

# 9. Authentication And Authorization

## Registration, Password Hashing, Login, Sessions, And JWT

        **Simple Definition**

        Authentication proves identity. Registration creates an identity; login verifies credentials; JWT or sessions represent authenticated state.

        **Why It Exists**

        It exists because protected APIs must know who is making a request before applying permissions.

        **Internal Working**

        Registration validates input, checks duplicate email, hashes password with salt/work factor, stores user. Login finds user, verifies password hash, creates session or tokens. JWT is signed; session ID references server-side state. Refresh tokens extend login without long-lived access tokens.

        **Request Or Execution Flow**

        Register -> validate -> duplicate check -> hash -> insert -> return public user. Login -> validate -> find user with password hash -> compare -> issue access/refresh -> set cookie or return token. Protected request -> verify token/session -> attach `req.user` -> authorize.

        **Real Node.js/Express Implementation**

        ```javascript
import crypto from "node:crypto";
import bcrypt from "bcrypt";
import jwt from "jsonwebtoken";
import { AppError } from "../utils/AppError.js";

function hashToken(token) {
  return crypto.createHash("sha256").update(token).digest("hex");
}

function addDays(date, days) {
  const copy = new Date(date);
  copy.setDate(copy.getDate() + days);
  return copy;
}

function toPublicUser(user) {
  return {
    id: String(user.id),
    name: user.name,
    email: user.email,
    role: user.role
  };
}

export async function register(input, { userRepository }) {
  const existing = await userRepository.findByEmail(input.email);
  if (existing) {
    throw new AppError(409, "EMAIL_ALREADY_EXISTS", "Email already exists");
  }

  const passwordHash = await bcrypt.hash(input.password, 12);
  const user = await userRepository.create({
    name: input.name,
    email: input.email.toLowerCase(),
    passwordHash,
    role: "student"
  });

  return toPublicUser(user);
}

export async function login(input, { userRepository, refreshTokenRepository }) {
  const user = await userRepository.findByEmailWithPassword(input.email.toLowerCase());
  if (!user || !(await bcrypt.compare(input.password, user.passwordHash))) {
    throw new AppError(401, "INVALID_CREDENTIALS", "Invalid email or password");
  }

  const accessToken = jwt.sign(
    { sub: String(user.id), role: user.role },
    process.env.JWT_ACCESS_SECRET,
    { expiresIn: "15m", algorithm: "HS256" }
  );

  const refreshToken = crypto.randomBytes(32).toString("hex");
  await refreshTokenRepository.storeHashedToken({
    userId: user.id,
    tokenHash: hashToken(refreshToken),
    familyId: crypto.randomUUID(),
    expiresAt: addDays(new Date(), 30)
  });

  return { accessToken, refreshToken, user: toPublicUser(user) };
}
```

        **Example Request And Response**

        ```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "vineet@example.com",
  "password": "CorrectPassword123"
}

HTTP/1.1 200 OK
Set-Cookie: refreshToken=<opaque>; HttpOnly; Secure; SameSite=Lax; Path=/api/v1/auth/refresh
Cache-Control: no-store

{
  "data": {
    "accessToken": "<jwt>",
    "user": {
      "id": "user_42",
      "name": "Vineet",
      "email": "vineet@example.com",
      "role": "student"
    }
  }
}
```

        **Common Mistakes**

        - Storing plain passwords.
- Returning password hash.
- Different error for unknown email vs wrong password.
- Long-lived access tokens.
- Putting secrets in JWT payload.
- Storing refresh tokens unhashed.

        **Security Concerns**

        - Use HTTPS.
- Hash passwords with bcrypt/Argon2/scrypt.
- Use `HttpOnly`, `Secure`, `SameSite` cookies for refresh tokens if cookie-based.
- Protect refresh endpoint from CSRF if using cookies.
- Rate limit login.
- Use generic credential errors.

        **Failure Scenarios**

        - Database insert fails after hashing.
- Refresh token reused after theft.
- Access token expired.
- JWT secret leaked.
- Clock skew affects token expiry.
- User disabled after token issued.

        **Testing Strategy**

        - Registration validation and duplicate tests.
- Login success and invalid credential tests.
- Password hash not returned.
- Expired token returns 401.
- Refresh rotation reuse detection.
- Logout revokes refresh token.

        **Performance Concerns**

        - bcrypt work factor affects CPU/thread pool.
- JWT verification is cheap but not free.
- Session DB/Redis lookup adds latency.
- Rate limiting needs shared store in multi-instance deployment.

        **Alternatives**

        - Server-side sessions.
- JWT access tokens with refresh tokens.
- OAuth/OIDC login.
- API keys for machine clients.
- mTLS/service tokens for service-to-service.

        **Trade-Offs**

        Sessions are easier to revoke but need shared server-side storage. JWTs scale statelessly but revocation requires short expiry, denylist, or refresh-token control.

        **Strong Interview Answer**

        ```text
        For authentication, I validate credentials, hash passwords with a slow salted hash, return only public user fields, and use either sessions or short-lived JWT access tokens. In production I prefer refresh-token rotation and hashed refresh-token storage so logout and reuse detection are possible.
        ```

        **Interviewer Follow-Ups With Answers**

        - **JWT encrypted?** A normal signed JWT is not encrypted. Do not put secrets in it.
- **HS256 vs RS256?** HS256 uses shared secret; RS256 uses private/public key pair and is better when many services verify tokens.

        **Connection To Your Projects**

        Your resume mentions JWT/RBAC. It is safe to say you used JWT and role checks if implemented. For refresh rotation/device management, phrase as production improvement unless you actually built it.

        **Small Exercise**

        Design register/login/refresh/logout contracts with correct status codes.

        **Exercise Solution / Expected Reasoning**

        Register: `POST /auth/register` -> 201 or 409/422. Login: `POST /auth/login` -> 200 or 401/422. Refresh: `POST /auth/refresh` -> 200 or 401, rotates token. Logout: `POST /auth/logout` -> 204 and revokes refresh token.

## RBAC, Permission-Based Access, Ownership, Multi-Tenant Auth, OAuth/OIDC, And API Keys

        **Simple Definition**

        Authorization decides what an authenticated identity can do.

        **Why It Exists**

        It exists because knowing who the user is is not enough; the backend must enforce role, permission, ownership, tenant, and service boundaries.

        **Internal Working**

        RBAC checks broad roles. Permission-based access maps roles/users to granular actions. Ownership checks compare resource owner/assignee/tenant to actor. Multi-tenant auth ensures users cannot cross tenant boundaries. OAuth delegates access; OIDC handles login identity. API keys authenticate machines/projects.

        **Request Or Execution Flow**

        Authenticate -> attach actor -> route-level RBAC -> load resource -> check ownership/tenant/permission -> perform action -> audit.

        **Real Node.js/Express Implementation**

        ```javascript
export async function resolveQuery({ actor, queryId }, { queryRepository }) {
  const query = await queryRepository.findById(queryId);
  if (!query) {
    throw new AppError(404, "QUERY_NOT_FOUND", "Query not found");
  }

  const isAssignedModerator =
    actor.role === "moderator" && String(query.assignedTo) === String(actor.id);
  const isAdmin = actor.role === "admin";

  if (!isAssignedModerator && !isAdmin) {
    throw new AppError(403, "FORBIDDEN", "Only assigned moderators or admins can resolve");
  }

  if (query.status === "resolved") {
    throw new AppError(409, "QUERY_ALREADY_RESOLVED", "Query is already resolved");
  }

  return queryRepository.updateStatus(queryId, "resolved");
}
```

        **Example Request And Response**

        ```http
PATCH /api/v1/queries/query_501
Authorization: Bearer <moderator-token>

{
  "status": "resolved"
}

403 Forbidden
{
  "error": {
    "code": "FORBIDDEN",
    "message": "Only assigned moderators or admins can resolve",
    "requestId": "req_123"
  }
}
```

        **Common Mistakes**

        - Only checking role and not ownership.
- Trusting `tenantId` from request body.
- Using 401 when user is authenticated but lacks permission.
- Putting all permission logic in frontend.
- Using one global API key without scopes.

        **Security Concerns**

        - Check authorization on every resource access.
- Use server-derived tenant ID.
- Audit sensitive permission changes.
- Hash API keys at rest.
- Scope and rotate API keys.

        **Failure Scenarios**

        - User changes URL ID to access another resource.
- Admin role accidentally assigned by mass assignment.
- Tenant filter missing from query.
- OAuth callback state missing and vulnerable to CSRF.

        **Testing Strategy**

        - Test wrong role -> 403.
- Test owner vs non-owner.
- Test tenant isolation.
- Test disabled user/token revocation behavior.
- Test API key scopes.

        **Performance Concerns**

        - Fine-grained permission checks add DB/cache lookups.
- Tenant filters need composite indexes.
- Permission matrices can become complex; centralize them.

        **Alternatives**

        - RBAC for simple apps.
- ABAC/policy engines for complex conditions.
- Database row-level security.
- OAuth/OIDC for third-party identity/delegation.

        **Trade-Offs**

        RBAC is simple and explainable, but ownership and tenant checks are still required. Permission systems scale better for large products but add complexity.

        **Strong Interview Answer**

        ```text
        I treat authentication and authorization separately. After authentication creates `req.user`, route middleware can check broad roles, but the service must load the resource and enforce ownership or tenant access. For large systems, I would move from only roles to scoped permissions or policy-based authorization.
        ```

        **Interviewer Follow-Ups With Answers**

        - **401 or 403 for wrong role?** 403, because the user is authenticated but not allowed.
- **What is OIDC?** An identity layer on OAuth 2.0 used for login and ID tokens.

        **Connection To Your Projects**

        ResolveAI needs RBAC for student/moderator/admin and ownership checks for own/assigned queries. Multi-tenant checks would be a production improvement if colleges/organizations share one deployment.

        **Small Exercise**

        A student calls `GET /queries/query_999` where another student owns the query. What should happen?

        **Exercise Solution / Expected Reasoning**

        After loading the query, service checks `createdBy`. If not owner/admin/allowed moderator, return 403 or 404 depending on whether you want to hide existence.

## Auth API Contracts And Test Cases

### Register

```http
POST /api/v1/auth/register
Content-Type: application/json

{
  "name": "Vineet",
  "email": "vineet@example.com",
  "password": "StrongPassword123"
}
```

Success: `201 Created`. Validation: `422`. Duplicate email: `409`.

### Login

Success: `200 OK`. Invalid credentials: `401`. Rate limited: `429`.

### Refresh

```http
POST /api/v1/auth/refresh
Cookie: refreshToken=<opaque>
```

Success returns a new access token and rotates refresh token. If a previously used refresh token appears again, revoke the token family and return `401`.

### Logout

```http
POST /api/v1/auth/logout
Cookie: refreshToken=<opaque>
```

Return `204 No Content`; revoke the refresh token.

### Complete Test Matrix

- Register valid user -> 201, password hash not returned.
- Register duplicate email -> 409.
- Register weak password -> 422.
- Login valid credentials -> 200, access token returned, refresh cookie set.
- Login wrong password -> 401 generic message.
- Login unknown email -> 401 same generic message.
- Login repeated failures -> 429 after threshold.
- Refresh valid token -> 200 and token rotated.
- Refresh old reused token -> 401 and family revoked.
- Logout valid token -> 204 and token revoked.
- Protected route missing access token -> 401.
- Protected route wrong role -> 403.
- Protected route wrong owner -> 403 or 404 by policy.

# 10. Backend Security And OWASP API Top 10

## OWASP API Security Top 10: Interview Handbook

For every risk, use this pattern: definition, vulnerable API, attack, secure implementation, test, interview answer.

### API1:2023 Broken Object Level Authorization

**Definition**

The API fails to verify that the authenticated user can access the specific object.

**Vulnerable API Example**

```text
GET /api/v1/users/43/orders with user 42's token.
```

**Attack Scenario**

Attacker changes IDs and reads another user's data.

**Secure Implementation**

Load the resource and check owner/tenant/role before returning it.

**Test Case**

Use integration tests where user A tries to access user B's resource.

**Interview Answer**

```text
I never trust object IDs from the client. Authentication identifies the actor, but the service must check ownership or tenant access for each object.
```

### API2:2023 Broken Authentication

**Definition**

Authentication is weak, incorrectly implemented, or easy to bypass.

**Vulnerable API Example**

```text
Long-lived tokens, no password hashing, no token expiry.
```

**Attack Scenario**

Stolen token remains valid forever.

**Secure Implementation**

Hash passwords, expire tokens, rotate refresh tokens, rate limit login, and verify JWT correctly.

**Test Case**

Test missing, invalid, expired, and reused tokens.

**Interview Answer**

```text
Authentication must protect credentials, token lifecycle, and brute-force paths, not just issue a token.
```

### API3:2023 Broken Object Property Level Authorization

**Definition**

API exposes or allows modification of fields the user should not access.

**Vulnerable API Example**

```text
User update accepts `{role:'admin'}`.
```

**Attack Scenario**

Attacker upgrades own role.

**Secure Implementation**

Whitelist input fields and shape output DTOs.

**Test Case**

Test that `role`, `passwordHash`, and internal fields cannot be set/read.

**Interview Answer**

```text
I use DTOs and explicit field mapping instead of blindly spreading request bodies or returning DB documents.
```

### API4:2023 Unrestricted Resource Consumption

**Definition**

API allows excessive requests, payloads, or expensive operations.

**Vulnerable API Example**

```text
Unlimited AI generation endpoint.
```

**Attack Scenario**

Attacker drains API quota or CPU.

**Secure Implementation**

Rate limits, body limits, timeouts, pagination, and quotas.

**Test Case**

Load and abuse tests for large payloads and repeated calls.

**Interview Answer**

```text
Every expensive endpoint needs limits and observability.
```

### API5:2023 Broken Function Level Authorization

**Definition**

User can call functions outside their role.

**Vulnerable API Example**

```text
Student calls DELETE /admin/users/42.
```

**Attack Scenario**

Attacker invokes admin operation.

**Secure Implementation**

Route-level RBAC plus service checks.

**Test Case**

Test every privileged endpoint with low-role token.

**Interview Answer**

```text
Function-level auth protects actions; object-level auth protects specific records.
```

### API6:2023 Unrestricted Access To Sensitive Business Flows

**Definition**

Business workflow lacks anti-abuse controls.

**Vulnerable API Example**

```text
Unlimited coupon redemption or payment attempt creation.
```

**Attack Scenario**

Automation abuses business process.

**Secure Implementation**

Rate limits, idempotency, fraud checks, monitoring, and workflow state machine.

**Test Case**

Test duplicate/repeated business actions.

**Interview Answer**

```text
Business flows need domain-specific limits beyond generic auth.
```

### API7:2023 Server Side Request Forgery

**Definition**

Server fetches attacker-controlled URLs.

**Vulnerable API Example**

```text
POST /fetch-image {url:'http://169.254.169.254/...'}
```

**Attack Scenario**

Attacker reaches internal metadata service.

**Secure Implementation**

Allowlist domains, block private IPs, resolve DNS safely, set timeouts.

**Test Case**

Security tests with localhost/private IP URLs.

**Interview Answer**

```text
Never let arbitrary user URLs become unrestricted server-side requests.
```

### API8:2023 Security Misconfiguration

**Definition**

Unsafe defaults or missing hardening.

**Vulnerable API Example**

```text
Stack traces in production, permissive CORS.
```

**Attack Scenario**

Attacker learns internals or accesses from unwanted origins.

**Secure Implementation**

Helmet, safe errors, secure CORS, no default credentials, environment validation.

**Test Case**

Deployment config tests and security scan.

**Interview Answer**

```text
Production config should fail closed and avoid leaking internals.
```

### API9:2023 Improper Inventory Management

**Definition**

Unknown old APIs remain exposed.

**Vulnerable API Example**

```text
Old /api/v0/admin route still active.
```

**Attack Scenario**

Attacker uses forgotten weaker endpoint.

**Secure Implementation**

Maintain API inventory, versioning, deprecation, and route scanning.

**Test Case**

Test exposed routes against documented inventory.

**Interview Answer**

```text
You cannot secure APIs you do not know exist.
```

### API10:2023 Unsafe Consumption Of APIs

**Definition**

Backend trusts third-party APIs or webhooks too much.

**Vulnerable API Example**

```text
Payment webhook processed without signature.
```

**Attack Scenario**

Fake provider callback marks order paid.

**Secure Implementation**

Verify signatures, validate schemas, set timeouts, handle retries/idempotency.

**Test Case**

Invalid signature and duplicate webhook tests.

**Interview Answer**

```text
External data is still untrusted input.
```

## Injection, CORS, CSRF, SSRF, Mass Assignment, File Attacks, ReDoS, Secrets, And Secure Logging

        **Simple Definition**

        Backend security is the practice of treating all external input as untrusted and enforcing validation, authorization, least privilege, and safe defaults.

        **Why It Exists**

        It exists because APIs expose business actions and data to networks, browsers, users, third-party providers, and automated attackers.

        **Internal Working**

        Security controls are layered: input validation, authentication, authorization, database parameterization, body limits, rate limits, output encoding, safe headers, secret management, dependency scanning, and monitoring.

        **Request Or Execution Flow**

        Request arrives -> body size limit -> parse -> validate schema -> authenticate -> authorize -> parameterized query -> safe response shaping -> structured log without secrets -> metrics/alerts.

        **Real Node.js/Express Implementation**

        ```javascript
const [rows] = await db.execute(
  `SELECT id, name, email
   FROM users
   WHERE email = ?
   LIMIT 1`,
  [email]
);

app.use(express.json({ limit: "1mb" }));
app.use(helmet());
```

        **Example Request And Response**

        ```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "' OR 1=1 --",
  "password": "anything"
}

HTTP/1.1 401 Unauthorized
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid email or password"
  }
}
```

        **Common Mistakes**

        - String-concatenated SQL.
- Allowing Mongo operators in JSON fields.
- Permissive CORS with credentials.
- Parsing Razorpay webhook before signature verification.
- Logging tokens.
- Unsafe regex on unbounded input.

        **Security Concerns**

        - Use parameterized SQL.
- Validate NoSQL field types.
- Use CSRF controls for cookie auth.
- Verify webhook signatures with raw body.
- Use timing-safe comparison for signatures.
- Keep secrets out of git and logs.

        **Failure Scenarios**

        - SQL injection dumps users.
- NoSQL injection bypasses login.
- CSRF performs state-changing request.
- SSRF reaches internal services.
- ReDoS blocks event loop.
- Leaked key allows provider access.

        **Testing Strategy**

        - Security unit tests for validators.
- Integration tests for ownership and role access.
- Invalid webhook signature test.
- Large body test.
- Dependency audit in CI.
- Fuzz tests for risky parsers where useful.

        **Performance Concerns**

        - Security checks cost CPU but prevent larger incidents.
- Regex and JSON parsing can block event loop.
- Rate limiting needs Redis for multi-instance consistency.
- Hashing passwords consumes CPU/thread pool.

        **Alternatives**

        - Framework security middleware.
- API gateway controls.
- WAF as extra layer.
- Database row-level security.
- Static analysis and dependency scanning.

        **Trade-Offs**

        Security is defense in depth. A WAF or gateway does not replace correct application authorization. Validation does not replace DB constraints. Prompt safety does not replace SQL validation.

        **Strong Interview Answer**

        ```text
        I treat every input as untrusted, validate it, use parameterized queries, enforce both role and ownership authorization, verify webhooks, avoid sensitive logs, apply rate limits and size limits, and test negative/security paths explicitly.
        ```

        **Interviewer Follow-Ups With Answers**

        - **Does CORS protect your API from curl?** No. CORS is enforced by browsers; backend auth is still required.
- **Why timing-safe compare?** To avoid leaking signature comparison information through timing differences.

        **Connection To Your Projects**

        Payment webhooks and Text-to-SQL are high-risk areas: verify signatures for payments, and never execute generated SQL without parsing and read-only restrictions.

        **Small Exercise**

        Find the bug: `User.create(req.body)` in registration.

        **Exercise Solution / Expected Reasoning**

        This is mass assignment. Whitelist fields and set role server-side. Also hash password and return a DTO without passwordHash.

# 11. SQL And MySQL

## Relational Modeling, Keys, Normalization, Joins, CTEs, Window Functions, And N+1

        **Simple Definition**

        Relational modeling stores structured data in related tables with keys and constraints.

        **Why It Exists**

        It exists to represent relationships consistently, avoid duplication, and let the database enforce integrity.

        **Internal Working**

        Primary keys identify rows; foreign keys connect tables; unique keys enforce domain rules; composite keys enforce multi-column uniqueness. Normalization reduces redundancy. Joins combine related rows. CTEs name subqueries. Window functions compute over row sets without collapsing them.

        **Request Or Execution Flow**

        Design entities -> identify relationships -> choose keys -> create constraints -> write queries -> inspect plans -> add indexes matching access patterns.

        **Real Node.js/Express Implementation**

        ```sql
CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE,
  role VARCHAR(50) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE queries (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(150) NOT NULL,
  status VARCHAR(30) NOT NULL,
  created_by BIGINT NOT NULL,
  assigned_to BIGINT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_queries_created_by FOREIGN KEY (created_by) REFERENCES users(id),
  CONSTRAINT fk_queries_assigned_to FOREIGN KEY (assigned_to) REFERENCES users(id)
);
```

        **Example Request And Response**

        ```sql
WITH open_queries AS (
  SELECT q.id, q.title, q.created_at, u.email AS student_email
  FROM queries q
  JOIN users u ON u.id = q.created_by
  WHERE q.status = 'submitted'
)
SELECT *,
       ROW_NUMBER() OVER (ORDER BY created_at DESC) AS row_num
FROM open_queries
ORDER BY created_at DESC
LIMIT 20;
```

        **Common Mistakes**

        - No foreign keys where integrity matters.
- Storing comma-separated IDs.
- N+1 queries for child records.
- Over-normalizing simple read models.
- Using natural keys that change as foreign keys.

        **Security Concerns**

        - Use least-privilege DB users.
- Parameterize queries.
- Avoid exposing internal IDs if business requires opaque IDs.
- Do not put sensitive fields in views used by public APIs.

        **Failure Scenarios**

        - Foreign key violation.
- Duplicate unique key under concurrency.
- Join returns too many rows due wrong condition.
- N+1 causes slow endpoint.
- Migration breaks existing data.

        **Testing Strategy**

        - Schema migration tests.
- Repository integration tests.
- Constraint violation tests.
- N+1 tests through query counting if tooling supports.
- SQL injection tests.

        **Performance Concerns**

        - Joins need indexes on join/filter columns.
- Window functions can sort large datasets.
- CTEs improve readability but still need plan inspection.
- N+1 round trips hurt latency.

        **Alternatives**

        - MongoDB document modeling.
- Denormalized read tables.
- Materialized views.
- Search engines for text-heavy queries.

        **Trade-Offs**

        Relational modeling gives strong integrity but can require joins and migrations. Denormalization improves reads but adds update complexity.

        **Strong Interview Answer**

        ```text
        In SQL, I model entities with primary keys, relationships with foreign keys, and business uniqueness with unique constraints. I normalize to avoid duplication, denormalize only intentionally, and use joins/CTEs/window functions when they match query needs. I watch for N+1 and use `EXPLAIN` for performance.
        ```

        **Interviewer Follow-Ups With Answers**

        - **What is 3NF?** No repeating groups, no partial dependency on part of a composite key, and no transitive dependency on non-key attributes.
- **How fix N+1?** Use joins, batching, eager loading, or a query designed to fetch children in one round trip.

        **Connection To Your Projects**

        Text-to-SQL likely uses MySQL. You should explain schema metadata, joins, and safe query validation. Payments also benefit from relational constraints.

        **Small Exercise**

        Design tables for students enrolling in courses.

        **Exercise Solution / Expected Reasoning**

        Use `students`, `courses`, and `enrollments(student_id, course_id, created_at)` with `UNIQUE(student_id, course_id)` and foreign keys.

## Indexes, EXPLAIN, Transactions, ACID, Isolation, Locks, Deadlocks, Pooling, Replication, Sharding, And Optimization

        **Simple Definition**

        Indexes speed up access patterns; transactions group changes; isolation controls concurrent visibility; pooling manages connections.

        **Why It Exists**

        They exist because real APIs need both performance and correctness under concurrent traffic.

        **Internal Working**

        MySQL/InnoDB commonly uses B+tree indexes. Composite indexes follow leftmost-prefix behavior. `EXPLAIN` shows access plans. Transactions use undo/redo logs and locks/MVCC. Isolation levels trade consistency and concurrency. Connection pools reuse database connections. Replication copies data to replicas with possible lag.

        **Request Or Execution Flow**

        Request -> get connection from pool -> start transaction if needed -> locking/queries -> commit/rollback -> release connection. For reads -> query plan uses indexes or scans -> return rows.

        **Real Node.js/Express Implementation**

        ```javascript
export async function markOrderPaid({ orderId, paymentId, amountPaise }, db) {
  const connection = await db.getConnection();
  try {
    await connection.beginTransaction();

    const [orders] = await connection.execute(
      `SELECT id, payment_status, amount_paise
       FROM orders
       WHERE id = ?
       FOR UPDATE`,
      [orderId]
    );

    const order = orders[0];
    if (!order) throw new AppError(404, "ORDER_NOT_FOUND", "Order not found");
    if (order.payment_status === "paid") {
      await connection.rollback();
      return { alreadyPaid: true };
    }
    if (order.amount_paise !== amountPaise) {
      throw new AppError(422, "AMOUNT_MISMATCH", "Payment amount mismatch");
    }

    await connection.execute(
      `INSERT INTO payments (order_id, provider_payment_id, amount_paise)
       VALUES (?, ?, ?)`,
      [orderId, paymentId, amountPaise]
    );

    await connection.execute(
      `UPDATE orders SET payment_status = 'paid' WHERE id = ?`,
      [orderId]
    );

    await connection.commit();
    return { alreadyPaid: false };
  } catch (error) {
    await connection.rollback();
    throw error;
  } finally {
    connection.release();
  }
}
```

        **Example Request And Response**

        ```sql
CREATE INDEX idx_queries_assigned_status_created
ON queries (assigned_to, status, created_at);

EXPLAIN
SELECT id, title, status
FROM queries
WHERE assigned_to = ? AND status = ?
ORDER BY created_at DESC
LIMIT 20;
```

        **Common Mistakes**

        - Indexing every column.
- Ignoring composite index order.
- Holding transactions open while calling external APIs.
- Not handling deadlock retries.
- Oversized connection pool overwhelming DB.

        **Security Concerns**

        - Use parameterized queries.
- Limit DB user permissions.
- Do not log raw sensitive SQL parameters.
- Protect backups and replication credentials.

        **Failure Scenarios**

        - Deadlock under concurrent updates.
- Replica lag after write.
- Pool exhaustion.
- Long transaction blocks writes.
- Full table scan under load.

        **Testing Strategy**

        - Repository tests with real DB.
- Concurrent transaction tests.
- Deadlock retry tests where practical.
- EXPLAIN review for important queries.
- Migration rollback tests.

        **Performance Concerns**

        - Indexes improve reads but slow writes.
- Large pools increase DB load.
- Isolation levels affect lock behavior.
- Read replicas scale reads but can be stale.
- Sharding adds operational complexity.

        **Alternatives**

        - Optimistic locking with version columns.
- Pessimistic locks with `FOR UPDATE`.
- Redis locks for cross-resource coordination, carefully.
- CQRS/read models for read-heavy systems.

        **Trade-Offs**

        High isolation and locks improve correctness but reduce concurrency. Read replicas improve scale but introduce stale reads. Sharding scales writes but complicates queries and transactions.

        **Strong Interview Answer**

        ```text
        I choose indexes from query patterns, verify with `EXPLAIN`, and use transactions when multiple changes must succeed together. I keep transactions short, avoid external calls inside them, handle deadlocks with retries, and size pools based on DB capacity rather than API traffic alone.
        ```

        **Interviewer Follow-Ups With Answers**

        - **What is leftmost-prefix rule?** A composite index is most useful from its leftmost columns onward.
- **Dirty vs non-repeatable vs phantom read?** Dirty reads uncommitted data; non-repeatable reads see a changed row; phantom reads see new rows matching a repeated predicate.

        **Connection To Your Projects**

        Razorpay payment updates should use transactions/unique constraints. Text-to-SQL should use read-only users, `EXPLAIN` awareness, limits, and timeouts.

        **Small Exercise**

        Choose an index for `GET /queries?assignedTo=42&status=open&sort=-createdAt`.

        **Exercise Solution / Expected Reasoning**

        Use composite index `(assigned_to, status, created_at)` or `(assigned_to, status, created_at DESC)` depending DB support and query needs. Equality filters first, sort/range last.

## SQL Query Exercises

### Exercise 1: Find Duplicate Emails

```sql
SELECT email, COUNT(*) AS total
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

### Exercise 2: Latest Order Per User

```sql
SELECT *
FROM (
  SELECT o.*,
         ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC, id DESC) AS rn
  FROM orders o
) ranked
WHERE rn = 1;
```

### Exercise 3: Avoid N+1 For Users And Order Counts

```sql
SELECT u.id, u.name, COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name
ORDER BY u.id
LIMIT 20;
```

### Exercise 4: Payment Idempotency Constraint

```sql
CREATE TABLE idempotency_keys (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  idempotency_key VARCHAR(255) NOT NULL,
  request_hash CHAR(64) NOT NULL,
  status VARCHAR(30) NOT NULL,
  response_json JSON NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  UNIQUE KEY uniq_idempotency_key (idempotency_key)
);
```

# 12. MongoDB

## MongoDB Modeling, BSON, ObjectId, Embedding, Referencing, Mongoose, Populate, And Lean Queries

        **Simple Definition**

        MongoDB stores BSON documents in collections. Schema design is based on access patterns, document growth, and consistency needs.

        **Why It Exists**

        It exists to model flexible document-shaped data and iterate quickly while still supporting indexes, validation, aggregation, and transactions.

        **Internal Working**

        Documents can embed related subdocuments or reference other documents by ID. Mongoose adds schemas, validation, middleware, and `populate`, but MongoDB itself stores documents. `lean()` returns plain objects and skips hydration. ObjectId is an identifier, not a permission check.

        **Request Or Execution Flow**

        API request -> validate input -> service decides ownership/business rule -> repository creates/finds documents -> Mongoose validates/casts -> MongoDB stores BSON -> response DTO returned.

        **Real Node.js/Express Implementation**

        ```javascript
const querySchema = new mongoose.Schema(
  {
    title: { type: String, required: true, trim: true, maxlength: 150 },
    description: { type: String, required: true, maxlength: 5000 },
    status: {
      type: String,
      enum: ["submitted", "assigned", "in_progress", "resolved", "closed"],
      default: "submitted"
    },
    priority: { type: String, enum: ["low", "medium", "high"], default: "medium" },
    createdBy: { type: mongoose.Schema.Types.ObjectId, ref: "User", required: true },
    assignedTo: { type: mongoose.Schema.Types.ObjectId, ref: "User" }
  },
  { timestamps: true, versionKey: "__v" }
);

querySchema.index({ assignedTo: 1, status: 1, createdAt: -1 });
querySchema.index({ createdBy: 1, createdAt: -1 });
querySchema.index({ status: 1, priority: 1, createdAt: -1 });
```

        **Example Request And Response**

        ```javascript
const queries = await Query.find({ assignedTo, status: "submitted" })
  .sort({ createdAt: -1, _id: -1 })
  .limit(20)
  .lean();
```

        **Common Mistakes**

        - Embedding unbounded arrays.
- Using `populate` for large lists without limits.
- Assuming Mongoose validation replaces API validation.
- Returning full documents with internal fields.
- Using ObjectId existence as authorization.

        **Security Concerns**

        - Validate ObjectId format and ownership.
- Prevent Mongo operator injection with schema validation.
- Hide password hashes with select false and DTOs.
- Use least-privilege DB users.
- Avoid exposing internal document structure.

        **Failure Scenarios**

        - Document grows beyond practical size.
- Populate causes many queries/large memory.
- Duplicate documents without unique index.
- Optimistic concurrency conflict.
- Unindexed query scans collection.

        **Testing Strategy**

        - Repository tests with test MongoDB.
- Validation tests.
- Ownership tests.
- Index-aware query tests.
- NoSQL injection tests.

        **Performance Concerns**

        - Use `lean()` for read-only list endpoints.
- Use compound indexes in query order.
- Avoid unbounded arrays.
- Use aggregation carefully with indexes.
- Monitor slow queries.

        **Alternatives**

        - MySQL for relational/transactional models.
- Embedding for bounded child data.
- Referencing for large/shared child data.
- Aggregation pipeline vs application joins.

        **Trade-Offs**

        MongoDB is flexible and fast for document access patterns, but schema mistakes can become expensive. SQL gives stronger relational constraints and joins.

        **Strong Interview Answer**

        ```text
        In MongoDB, I design around access patterns. I embed small bounded data that is read with the parent, reference large or shared data, add indexes for real queries, use DTOs for safe responses, and use transactions only when multi-document atomicity is required.
        ```

        **Interviewer Follow-Ups With Answers**

        - **When use `populate`?** For convenient reference loading in small controlled cases; avoid overusing it for large lists.
- **What is `lean()`?** A Mongoose option returning plain objects, improving read performance when document methods are not needed.

        **Connection To Your Projects**

        ResolveAI query documents fit MongoDB if query data is document-like and evolves. Payments can be done in MongoDB too, but SQL often makes transactional constraints easier to explain.

        **Small Exercise**

        Model query replies. Embed or reference?

        **Exercise Solution / Expected Reasoning**

        If replies are few and always loaded with the query, embed. If replies can grow large, need pagination, moderation, or independent search, use a `query_replies` collection with `queryId` index.

## MongoDB Indexes, Aggregation, Transactions, Atomic Updates, Pagination, Replication, Sharding, Change Streams, And Performance

        **Simple Definition**

        MongoDB performance and correctness depend on indexes, atomic document operations, controlled transactions, and query-aware schema design.

        **Why It Exists**

        These features exist to keep document databases fast and reliable under real workloads.

        **Internal Working**

        Indexes map fields to document locations. Compound index order matters. Unique indexes enforce constraints; partial indexes index only matching documents; TTL indexes expire documents. Single-document updates are atomic. Transactions handle multi-document atomicity. Replication provides availability; sharding distributes data; change streams observe changes.

        **Request Or Execution Flow**

        Query -> planner chooses index or collection scan -> documents returned/aggregated -> cursor pagination uses stable sort key -> updates use atomic operators or transactions.

        **Real Node.js/Express Implementation**

        ```javascript
// Cursor pagination
async function listAssignedQueries({ assignedTo, status, after, limit = 20 }) {
  const filter = { assignedTo, status };

  if (after) {
    filter.$or = [
      { createdAt: { $lt: after.createdAt } },
      { createdAt: after.createdAt, _id: { $lt: after.id } }
    ];
  }

  return Query.find(filter)
    .sort({ createdAt: -1, _id: -1 })
    .limit(limit)
    .lean();
}

// Atomic update
await Query.updateOne(
  { _id: queryId, status: { $ne: "resolved" } },
  { $set: { status: "resolved", resolvedAt: new Date() } }
);
```

        **Example Request And Response**

        ```javascript
db.queries.aggregate([
  { $match: { status: "resolved" } },
  { $group: { _id: "$assignedTo", totalResolved: { $sum: 1 } } },
  { $sort: { totalResolved: -1 } }
]);
```

        **Common Mistakes**

        - Index fields in wrong order.
- No unique index for idempotency.
- Using skip/limit deep pagination on huge collections.
- Multi-document transaction for simple single-document update.
- Ignoring replica lag.

        **Security Concerns**

        - Do not expose connection strings.
- Validate aggregation inputs.
- Restrict user-controlled sort fields.
- Use read/write concerns intentionally for critical writes.
- Block Mongo operator injection.

        **Failure Scenarios**

        - TTL deletes data unexpectedly if used on audit records.
- Shard key causes hot shard.
- Change stream resume token lost.
- Transaction retries needed.
- Aggregation exceeds memory/time.

        **Testing Strategy**

        - Explain plans for hot queries.
- Unique index conflict tests.
- Pagination stability tests.
- Transaction failure tests.
- Mongo injection tests.

        **Performance Concerns**

        - Covered queries avoid fetching full docs.
- Compound indexes should match filter/sort.
- Aggregation stages should filter early.
- Lean queries reduce Mongoose overhead.
- Sharding adds routing overhead if shard key is poor.

        **Alternatives**

        - SQL for complex joins.
- Search engine for full-text ranking.
- Redis for cache/queues.
- Application-level denormalized read models.

        **Trade-Offs**

        MongoDB makes atomic document updates easy, but cross-document consistency and joins need more care. Sharding helps scale but requires good shard key design.

        **Strong Interview Answer**

        ```text
        For MongoDB performance, I choose indexes from query patterns, use compound indexes for filter and sort, avoid unbounded arrays, use cursor pagination for large lists, and use transactions only when multiple documents must commit together.
        ```

        **Interviewer Follow-Ups With Answers**

        - **What is TTL index for?** Automatic expiry for temporary documents like sessions or OTPs, not audit/payment records.
- **What is a covered query?** A query satisfied from the index because filter/projection fields are all in the index.

        **Connection To Your Projects**

        Query-resolution lists need indexes by `createdBy`, `assignedTo/status`, and `status/priority`. Plant-adoption payments need unique provider IDs and audit history.

        **Small Exercise**

        Design Mongo schema for plant adoption orders and payments.

        **Exercise Solution / Expected Reasoning**

        Use `orders` with user/plant/status/amount, `paymentAttempts` or embedded bounded attempts if small, unique `razorpayPaymentId`, and indexes on user/status/createdAt. Keep audit events separate if they can grow.

# 13. API Testing Handbook

## Testing Pyramid, Test Doubles, Isolation, Determinism, Async, Concurrency, And External APIs

        **Simple Definition**

        API testing verifies that backend behavior is correct across units, integrations, contracts, security paths, performance paths, and production smoke checks.

        **Why It Exists**

        It exists because backend bugs often hide in failure cases: auth, validation, concurrency, retries, duplicate requests, external provider failures, and database transactions.

        **Internal Working**

        Unit tests isolate services with fakes/stubs. Integration tests run Express routes with middleware and a test DB. E2E tests cover full flows. Contract tests verify API shapes. Load/security tests test non-functional behavior. Deterministic tests control time, randomness, data, and external providers.

        **Request Or Execution Flow**

        Arrange fixtures -> act through service or HTTP -> assert status/body/DB side effects/external calls -> cleanup. For concurrency, run parallel requests and assert one side effect.

        **Real Node.js/Express Implementation**

        ```javascript
import test from "node:test";
import assert from "node:assert/strict";
import request from "supertest";

test("student cannot access another student's query", async () => {
  const app = createTestApp();
  const token = createTestAccessToken({ sub: "student_1", role: "student" });

  await request(app)
    .get("/api/v1/queries/query_owned_by_student_2")
    .set("Authorization", `Bearer ${token}`)
    .expect(403);
});
```

        **Example Request And Response**

        ```text
Test result should verify:
- HTTP status.
- Response error code.
- No forbidden data in body.
- No unintended DB change.
```

        **Common Mistakes**

        - Only testing happy path.
- Mocking the code under test too much.
- Using production DB.
- Tests depend on real time/external APIs.
- No cleanup between tests.
- Ignoring concurrency.

        **Security Concerns**

        - Test auth bypass attempts.
- Test ownership failures.
- Test injection payloads.
- Test large request bodies.
- Test invalid webhook signatures.
- Never use real secrets in tests.

        **Failure Scenarios**

        - Flaky tests due shared state.
- Race condition only appears under parallel requests.
- External provider outage breaks tests.
- Transactions not rolled back.
- Fake timers not restored.

        **Testing Strategy**

        - Unit tests for services.
- Integration tests for routes.
- Contract tests for API schema.
- Security tests for negative paths.
- Performance smoke tests for hot endpoints.
- Manual Postman only as supplement.

        **Performance Concerns**

        - Too many slow E2E tests hurt feedback.
- Use testcontainers/Dockerized DB for realism.
- Use fakes for fast unit tests.
- Parallel tests need isolated data.

        **Alternatives**

        - Node's built-in `node:test`.
- Jest/Vitest.
- Supertest for HTTP.
- Postman/Newman for collections.
- k6/Artillery for load.
- Testcontainers or Docker Compose for DBs.

        **Trade-Offs**

        Fast unit tests give feedback; integration tests give confidence. Mocking all dependencies is fast but can miss wiring bugs. Real DB tests are slower but catch schema/query issues.

        **Strong Interview Answer**

        ```text
        I test backend APIs at multiple levels. Services get unit tests with fake repositories. Routes get integration tests with Supertest and a test database. I test negative paths: missing auth, wrong role, invalid input, not found, conflict, external failure, idempotency, duplicate webhooks, and concurrency.
        ```

        **Interviewer Follow-Ups With Answers**

        - **How test payment provider?** Mock Razorpay SDK/API and separately test signature verification with known payloads.
- **How test retries?** Use fake timers or controlled failing stub, assert retry count and final state.

        **Connection To Your Projects**

        Your Razorpay, Inngest, and Text-to-SQL projects need tests beyond normal CRUD: duplicate payment/webhook, job retries, AI failure, SQL safety validation.

        **Small Exercise**

        Write test cases for refresh-token rotation.

        **Exercise Solution / Expected Reasoning**

        Test valid refresh rotates token, old token reuse is rejected, token family revoked, missing cookie 401, expired refresh 401, and logout-revoked token 401.

## Complete Testing Example For Queries Module

The snippets below use test helpers with obvious responsibilities: `createTestApp()` builds an Express app with fake or test dependencies, `createTestDependencies()` returns repositories/providers configured for tests, and `createTestAccessToken()` signs a short-lived JWT using the test secret. In a real repository, keep these helpers in `tests/helpers`.

### Service Unit Test

```javascript
import test from "node:test";
import assert from "node:assert/strict";
import { createQueryService } from "../src/modules/queries/query.service.js";

test("createQuery rejects moderator-created query", async () => {
  const service = createQueryService({
    queryRepository: {
      create: async () => {
        throw new Error("should not create");
      }
    },
    eventPublisher: {
      publish: async () => {}
    }
  });

  await assert.rejects(
    () => service.createQuery({
      actor: { id: "mod_1", role: "moderator" },
      input: {
        title: "Cannot access portal",
        description: "Portal gives error while registering",
        category: "technical"
      }
    }),
    { code: "FORBIDDEN" }
  );
});
```

### Integration Test With Supertest

```javascript
import test from "node:test";
import request from "supertest";
import { createApp } from "../src/app.js";

test("POST /queries validates title", async () => {
  const app = createApp(createTestDependencies());
  const token = createTestAccessToken({ sub: "student_1", role: "student" });

  await request(app)
    .post("/api/v1/queries")
    .set("Authorization", `Bearer ${token}`)
    .send({
      title: "bad",
      description: "Portal gives error while registering",
      category: "technical"
    })
    .expect(422);
});
```

### Authentication Test

```javascript
await request(app)
  .post("/api/v1/queries")
  .send(validQueryBody)
  .expect(401);
```

### Authorization Test

```javascript
const moderatorToken = createTestAccessToken({ sub: "mod_1", role: "moderator" });

await request(app)
  .post("/api/v1/queries")
  .set("Authorization", `Bearer ${moderatorToken}`)
  .send(validQueryBody)
  .expect(403);
```

### Not-Found And Conflict Tests

- `GET /queries/missing` -> 404.
- `PATCH /queries/resolved` to resolve again -> 409.

### Database Failure Test

Make repository throw an error and assert:

- Status is 500 or mapped operational code.
- Response does not expose stack trace.
- `requestId` is present.

### External API Failure Test

If event publishing fails after query creation, decide policy:

- If event is required, fail request and roll back/write outbox.
- If event is async best-effort, persist query and mark processing pending.

Test the chosen policy explicitly.

### Idempotency Test

Send two identical POST payment requests with same idempotency key. Assert:

- One payment attempt row.
- Same response body.
- Same provider order ID.

### Concurrent Duplicate-Request Test

```javascript
const [a, b] = await Promise.all([
  request(app).post("/api/v1/payments").set("Idempotency-Key", key).send(body),
  request(app).post("/api/v1/payments").set("Idempotency-Key", key).send(body)
]);

assert.equal(a.status, 201);
assert.ok([200, 201, 409].includes(b.status));
assert.equal(await countPaymentAttempts(key), 1);
```

The exact second status depends on design: return stored result, wait for first, or return conflict/in-progress.

# 14. Background Jobs, Messaging, And Inngest

## Background Jobs, Delivery Semantics, Idempotent Consumers, Retries, DLQ, Outbox, And Inngest

        **Simple Definition**

        Background jobs run slow, retryable, scheduled, or side-effect-heavy work outside the request-response path.

        **Why It Exists**

        They exist to keep APIs responsive and make failures recoverable, especially for email, AI processing, webhooks, certificate generation, and reconciliation.

        **Internal Working**

        A producer creates a job/event. A consumer handles it. At-most-once delivery may lose work; at-least-once may duplicate work, so consumers must be idempotent. Retries use backoff and jitter. DLQs store exhausted jobs. Transactional outbox solves the race between DB write and event publication.

        **Request Or Execution Flow**

        API writes DB -> stores outbox/event or sends Inngest event -> returns 201/202 -> worker/function runs steps -> retries transient failures -> updates job state -> logs/alerts failures.

        **Real Node.js/Express Implementation**

        ```javascript
export const classifyQuery = inngest.createFunction(
  {
    id: "classify-query",
    idempotency: "event.data.queryId",
    retries: 3,
    triggers: [{ event: "query/created" }]
  },
  async ({ event, step }) => {
    const query = await step.run("load-query", () =>
      queryRepository.findById(event.data.queryId)
    );

    const classification = await step.run("classify-with-ai", () =>
      aiService.classify(query.title, query.description)
    );

    await step.run("save-classification", () =>
      queryRepository.saveClassification(query.id, classification)
    );

    await step.run("notify-assignee-once", () =>
      notificationService.sendOnce({
        dedupeKey: `query-assigned:${query.id}`,
        queryId: query.id
      })
    );
  }
);
```

        **Example Request And Response**

        ```http
POST /api/v1/reports

HTTP/1.1 202 Accepted
{
  "data": {
    "jobId": "job_123",
    "status": "queued",
    "statusUrl": "/api/v1/jobs/job_123"
  }
}
```

        **Common Mistakes**

        - Doing slow work inside request.
- Assuming exactly-once delivery.
- Non-idempotent email/payment side effects.
- Retrying permanent validation errors.
- Publishing event after DB write with no recovery plan.

        **Security Concerns**

        - Do not put secrets in job payloads.
- Authorize job-triggering endpoints.
- Validate webhook/job payloads.
- Avoid logging sensitive payloads.
- Protect internal job dashboards.

        **Failure Scenarios**

        - Job fails after all retries.
- Duplicate event delivered.
- Event published but DB transaction rolled back.
- DB committed but event publish failed.
- Jobs run out of order.
- Queue backlog grows.

        **Testing Strategy**

        - Unit test job handler with fake services.
- Test retryable vs non-retryable errors.
- Test duplicate delivery.
- Test DLQ/failure status.
- Test outbox publisher.
- Test scheduled reconciliation.

        **Performance Concerns**

        - Set concurrency limits.
- Monitor queue depth and job latency.
- Use backoff+jitter.
- Avoid too many tiny steps or one huge opaque step.
- Keep payloads small.

        **Alternatives**

        - Inngest for durable functions.
- BullMQ with Redis for queues.
- RabbitMQ for message routing.
- Kafka for high-throughput event streams.
- Cron for scheduled tasks.

        **Trade-Offs**

        Inngest simplifies durable workflows and steps. BullMQ gives direct Redis queue control. RabbitMQ fits routing/worker queues. Kafka fits event streaming. The right tool depends on delivery, ordering, throughput, and operational comfort.

        **Strong Interview Answer**

        ```text
        I use background jobs for slow or retryable work. Because most queues are at-least-once, my consumers are idempotent using unique keys and state checks. I use retries with backoff, DLQs for exhausted failures, and an outbox pattern when DB writes and event publication must stay consistent.
        ```

        **Interviewer Follow-Ups With Answers**

        - **Why is exactly once hard?** Networks and crashes make it difficult to know whether a side effect happened; practical systems use at-least-once plus idempotency.
- **What is outbox pattern?** Write business data and an event record in one DB transaction, then a worker publishes the event.

        **Connection To Your Projects**

        ResolveAI can use Inngest for AI classification and notifications. Certificate generation after Razorpay payment can also run as a job.

        **Small Exercise**

        Design a retry-safe email notification job.

        **Exercise Solution / Expected Reasoning**

        Create a notification row with unique `dedupeKey`, send email, mark sent. If retry happens, check existing sent row and skip duplicate. Use backoff and DLQ after max retries.

# 15. Razorpay And Payment Backend Design

## Razorpay Payment State Machine, Signature Verification, Webhooks, Idempotency, Reconciliation, Refunds, And Auditing

        **Simple Definition**

        Payment backend design coordinates local order state with Razorpay order/payment state safely.

        **Why It Exists**

        It exists because money workflows must handle retries, duplicate callbacks, webhooks arriving out of order, database failures, and auditability.

        **Internal Working**

        The backend creates a local order/payment attempt, creates a Razorpay order server-side, returns checkout details, verifies client callback signature, processes webhooks with raw-body signature verification, updates local state idempotently, reconciles mismatches, and records audit events.

        **Request Or Execution Flow**

        Local order created -> Razorpay order created -> checkout opens -> payment authorized/captured/failed -> client callback -> backend verifies -> webhook confirms -> DB transaction updates order/payment -> certificate/job triggered -> reconciliation handles gaps.

        **Real Node.js/Express Implementation**

        ```javascript
import crypto from "node:crypto";

export function verifyCheckoutSignature({ orderId, paymentId, signature, secret }) {
  const expected = crypto
    .createHmac("sha256", secret)
    .update(`${orderId}|${paymentId}`)
    .digest("hex");

  if (typeof signature !== "string" || signature.length !== expected.length) {
    return false;
  }

  return crypto.timingSafeEqual(Buffer.from(expected), Buffer.from(signature));
}

export function verifyWebhookSignature({ rawBody, signature, secret }) {
  const expected = crypto
    .createHmac("sha256", secret)
    .update(rawBody)
    .digest("hex");

  if (typeof signature !== "string" || signature.length !== expected.length) {
    return false;
  }

  return crypto.timingSafeEqual(Buffer.from(expected), Buffer.from(signature));
}
```

        **Example Request And Response**

        ```http
POST /api/v1/orders/order_501/payment-attempts
Authorization: Bearer <token>
Idempotency-Key: order_501_payment_1

HTTP/1.1 201 Created
{
  "data": {
    "paymentAttemptId": "pa_123",
    "razorpayOrderId": "order_RZP123",
    "amountPaise": 100000,
    "currency": "INR",
    "status": "created"
  }
}
```

        **Common Mistakes**

        - Trusting frontend amount.
- Marking paid without signature verification.
- Ignoring webhooks.
- Processing duplicate webhook twice.
- No unique constraint on provider payment ID.
- Calling Razorpay inside DB transaction for too long.

        **Security Concerns**

        - Keep Razorpay secret server-side only.
- Verify checkout and webhook signatures.
- Use raw webhook body.
- Use timing-safe compare.
- Validate amount/currency against local order.
- Do not log secrets or full card/payment sensitive data.

        **Failure Scenarios**

        - Payment captured but DB update failed.
- DB committed but response failed, client retries.
- Webhook duplicate.
- Webhook out of order.
- Refund partially succeeds.
- Razorpay API unavailable.

        **Testing Strategy**

        - Mock Razorpay order creation.
- Signature verification test with known payload.
- Invalid signature test.
- Duplicate callback/webhook tests.
- Concurrent idempotency test.
- Reconciliation job test.
- Refund state test.

        **Performance Concerns**

        - Payment endpoints should be low latency and reliable.
- Use short DB transactions.
- Webhook processing should acknowledge fast and enqueue heavy work.
- Unique constraints prevent duplicate writes under concurrency.

        **Alternatives**

        - Other payment gateways with similar server-side verification.
- UPI/manual reconciliation flows.
- Provider-hosted checkout vs custom checkout.

        **Trade-Offs**

        Provider checkout reduces compliance burden, but you must still own backend correctness. Webhooks are reliable final signals, but they can duplicate or arrive out of order.

        **Strong Interview Answer**

        ```text
        For Razorpay, I create orders on the backend, store local payment attempts, verify checkout signatures server-side, process signed webhooks idempotently, store amounts in paise, use unique constraints on provider IDs, and reconcile provider/local state when failures happen.
        ```

        **Interviewer Follow-Ups With Answers**

        - **What if webhook arrives before client callback?** Webhook processing should be independent and idempotent; state machine handles either order.
- **Authorized vs captured?** Authorized means approved/held; captured means funds collected. Auto-capture may make them close together.

        **Connection To Your Projects**

        Your Vrikshami project mentions Razorpay. Claim signature verification only if implemented; otherwise say production version would verify callbacks and webhooks with idempotency and reconciliation.

        **Small Exercise**

        Handle duplicate `payment.captured` webhook.

        **Exercise Solution / Expected Reasoning**

        Verify signature, read provider event ID/payment ID, insert event with unique constraint, load payment/order, if already captured return 200, otherwise update in transaction and append audit event.

## Payment State Machine

```text
created
  -> pending
  -> authorized
  -> captured
  -> failed
  -> refunded
  -> partially_refunded
```

Recommended local states:

- `created`: local attempt exists, provider order may be created.
- `pending`: checkout started or awaiting provider final state.
- `authorized`: payment authorized but not captured.
- `captured`: funds captured and order can be fulfilled.
- `failed`: payment failed or expired.
- `refunded`: full amount refunded.
- `partially_refunded`: part of amount refunded.

## Text Sequence Diagram

```text
Client -> API: POST /orders/:id/payment-attempts + Idempotency-Key
API -> DB: create local payment_attempt(created)
API -> Razorpay: create order(amount, currency, receipt)
Razorpay -> API: razorpay_order_id
API -> DB: store provider order id
API -> Client: checkout details
Client -> Razorpay Checkout: user pays
Razorpay Checkout -> Client: payment_id, order_id, signature
Client -> API: POST /payments/verify
API -> API: verify HMAC signature
API -> DB: transaction mark attempt captured if valid
Razorpay -> API webhook: payment.captured
API -> API: verify raw-body webhook signature
API -> DB: idempotently process event
API -> Inngest: generate certificate / notify
```

## Failure-Case Matrix

| Failure | Risk | Correct Handling |
|---|---|---|
| Client retries create attempt | Duplicate payment order | Idempotency key with request hash and unique constraint |
| Payment succeeds, API response lost | User retries | Same idempotency key returns stored result |
| Payment captured, DB update fails | Local order pending | Webhook retry or reconciliation job repairs |
| DB updated, response fails | Client sees failure though paid | Retry returns paid state; UI polls status |
| Duplicate webhook | Duplicate side effects | Unique event/payment ID and idempotent consumer |
| Webhooks out of order | Invalid state transition | State machine ignores stale transitions or records for review |
| Amount mismatch | Fraud/bug | Reject verification and alert |
| Refund partial | Wrong order status | Track refund amount and state separately |

# 16. Docker, Deployment, And Production Engineering

## Environments, Config, Secrets, Docker, Compose, CI/CD, Deployment, Scaling, Caching, Timeouts, Retries, Circuit Breakers, And Rollbacks

        **Simple Definition**

        Production engineering is everything needed to run a backend safely outside your laptop.

        **Why It Exists**

        It exists because correct code can still fail due bad config, missing secrets, unsafe deploys, no health checks, overloaded dependencies, or no observability.

        **Internal Working**

        Apps run across development, test, staging, and production. Config is injected by environment. Secrets come from secret managers. Docker images package runtime. CI runs tests/scans. CD deploys with rollback. Health/readiness/liveness endpoints guide orchestration. Redis caches hot data. Timeouts/retries/circuit breakers protect dependencies.

        **Request Or Execution Flow**

        Commit -> CI install/test/lint/security -> build Docker image -> deploy staging -> smoke test -> canary/blue-green production -> monitor metrics/logs -> rollback if error budget burns.

        **Real Node.js/Express Implementation**

        ```dockerfile
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev

FROM node:22-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production
USER node
COPY --chown=node:node --from=deps /app/node_modules ./node_modules
COPY --chown=node:node . .
EXPOSE 3000
CMD ["node", "src/server.js"]
```

        **Example Request And Response**

        ```yaml
services:
  api:
    build: .
    ports:
      - "3000:3000"
    env_file:
      - .env
    depends_on:
      - mysql
      - redis
  mysql:
    image: mysql:8.4
    environment:
      MYSQL_DATABASE: app
      MYSQL_ROOT_PASSWORD: local-dev-only
  redis:
    image: redis:7-alpine
```

        **Common Mistakes**

        - Putting secrets in image.
- Using production DB in tests.
- No readiness endpoint.
- Retrying every failure blindly.
- No rollback plan.
- Running as root without need.

        **Security Concerns**

        - Do not bake secrets into Docker images.
- Use least privilege.
- Scan dependencies/images.
- Validate env at startup.
- Separate staging and production credentials.
- Use secure headers and TLS.

        **Failure Scenarios**

        - Bad migration breaks deploy.
- Cache stampede after expiry.
- Retry storm during provider outage.
- Pool exhaustion.
- Canary shows high errors.
- Rollback incompatible with DB migration.

        **Testing Strategy**

        - CI unit/integration tests.
- Docker build test.
- Smoke tests after deploy.
- Migration tests.
- Load tests for key endpoints.
- Chaos/failure drills at awareness level.

        **Performance Concerns**

        - Cache-aside reduces DB load.
- Cache invalidation is hard.
- Connection pool too large hurts DB.
- p95/p99 latency reveal tail problems.
- Horizontal scaling needs stateless servers.

        **Alternatives**

        - Blue-green deployment.
- Canary deployment.
- Rolling deploy.
- Feature flags.
- Serverless/functions for some workloads.
- Kubernetes for orchestration awareness.

        **Trade-Offs**

        More infrastructure improves reliability only if operated well. Start simple: stateless API, good config, health checks, tests, logs, metrics, and safe deploys.

        **Strong Interview Answer**

        ```text
        In production I separate environments, validate configuration, keep secrets out of code/images, containerize with Docker, expose health/readiness checks, use structured logs and metrics, set timeouts/retries carefully, cache hot reads, and deploy with rollback strategy.
        ```

        **Interviewer Follow-Ups With Answers**

        - **Readiness vs liveness?** Readiness means can receive traffic; liveness means process is alive and should not be restarted unless wedged.
- **Why p95/p99?** Averages hide slow tail latency that users experience.

        **Connection To Your Projects**

        Docker/logging/monitoring are resume topics. Explain them as production foundations, and be honest about whether you used full CI/CD, canary, or Kubernetes.

        **Small Exercise**

        Design a readiness endpoint for an API using MySQL and Redis.

        **Exercise Solution / Expected Reasoning**

        Check DB and Redis with short timeout. Return 200 if both required dependencies are ready; return 503 with safe status if not. Do not expose credentials/errors.

# 17. Logging, Monitoring, And Observability

## Structured Logs, Request IDs, Metrics, Tracing, Alerts, SLO/SLA, And Sensitive Data

        **Simple Definition**

        Observability is the ability to understand what a backend is doing from logs, metrics, traces, and alerts.

        **Why It Exists**

        It exists because production failures must be diagnosed without attaching a debugger to a live request.

        **Internal Working**

        Request ID middleware assigns/carries correlation IDs. Structured logs record JSON fields. Metrics count requests, errors, latency, queue depth, and business events. Traces connect calls across services. Alerts fire on symptoms. SLOs define reliability targets; error budgets measure allowed unreliability.

        **Request Or Execution Flow**

        Request enters -> request ID stored in req/AsyncLocalStorage -> logs include requestId/userId/path/status/duration -> metrics record latency/error -> traces propagate to DB/provider/job -> alert if thresholds breached.

        **Real Node.js/Express Implementation**

        ```javascript
import { AsyncLocalStorage } from "node:async_hooks";
import crypto from "node:crypto";

export const requestContext = new AsyncLocalStorage();

export function requestId(req, res, next) {
  const requestIdValue = req.get("X-Request-ID") || crypto.randomUUID();
  res.set("X-Request-ID", requestIdValue);

  requestContext.run({ requestId: requestIdValue }, next);
}

export function logInfo(message, fields = {}) {
  const context = requestContext.getStore() || {};
  console.log(JSON.stringify({ level: "info", message, ...context, ...fields }));
}
```

        **Example Request And Response**

        ```json
{
  "level": "info",
  "message": "request completed",
  "requestId": "req_123",
  "method": "POST",
  "path": "/api/v1/payments",
  "statusCode": 201,
  "durationMs": 82,
  "userId": "user_42"
}
```

        **Common Mistakes**

        - Plain unstructured logs only.
- Logging tokens/passwords.
- No request ID.
- Alerting on every small blip.
- Only monitoring CPU, not business failures.

        **Security Concerns**

        - Redact secrets.
- Avoid logging full auth headers.
- Be careful with PII.
- Protect log access.
- Do not expose internal errors to clients.

        **Failure Scenarios**

        - Cannot trace payment failure.
- Logs too noisy.
- Missing job request context.
- High 5xx not alerted.
- p99 latency spike hidden by average.

        **Testing Strategy**

        - Test request ID appears in error response.
- Test logger redaction.
- Smoke test metrics endpoint if exposed internally.
- Alert rules tested in staging where possible.

        **Performance Concerns**

        - Logging synchronously too much can slow hot paths.
- High-cardinality metrics can be expensive.
- Trace sampling balances detail and cost.

        **Alternatives**

        - APM tools.
- OpenTelemetry.
- Cloud provider logs/metrics.
- Self-hosted Prometheus/Grafana/Loki/ELK.

        **Trade-Offs**

        More observability costs storage and attention. Track high-signal logs and metrics first: request latency, error rate, dependency health, and business-critical failures.

        **Strong Interview Answer**

        ```text
        I include request IDs in responses and logs, use structured logs without secrets, monitor latency percentiles and error rates, trace important flows like payments/jobs, and alert on user-impacting symptoms rather than only infrastructure metrics.
        ```

        **Interviewer Follow-Ups With Answers**

        - **What is error budget?** Allowed unreliability under an SLO; if burned, prioritize reliability over new features.
- **Why AsyncLocalStorage?** It carries request context across async calls without manually passing requestId everywhere.

        **Connection To Your Projects**

        For Razorpay, log payment attempt ID, order ID, provider payment ID, request ID, and state transition, but never secrets. For Inngest, log job ID and step names.

        **Small Exercise**

        What should be logged for a failed payment verification?

        **Exercise Solution / Expected Reasoning**

        Log requestId, userId, local order/paymentAttempt ID, provider order/payment ID if present, error code, and safe reason. Do not log Razorpay secret, full token, or sensitive card data.

# 18. Alternative Communication Styles

## REST

Use REST for resource-oriented CRUD and standard web APIs. It is browser-friendly, cache-friendly, and easy to document with OpenAPI. Do not use REST alone when clients need flexible graph-shaped queries or real-time bidirectional communication.

## GraphQL

Use GraphQL when clients need flexible field selection across related data and you can manage schema/resolver complexity. Avoid it if your team mainly needs simple CRUD and does not want resolver authorization/N+1 complexity.

## RPC

RPC exposes actions directly, such as `CreatePayment` or `ResolveQuery`. It fits command-heavy internal APIs. It is less resource-oriented and can become inconsistent if not documented.

## gRPC

gRPC uses Protocol Buffers and HTTP/2. It is excellent for internal service-to-service communication with strong contracts and streaming support. Browser support and debugging are less simple than REST.

## WebSockets

Use WebSockets for bidirectional real-time communication such as chat, collaborative editing, live dashboards, or multiplayer experiences. Avoid them for simple request-response flows.

## Server-Sent Events

SSE is server-to-client streaming over HTTP. It is simpler than WebSockets for notifications/progress updates where the client does not need to send frequent messages back.

## Webhooks

Webhooks let external systems notify your backend. Use them for payment events, provider callbacks, and asynchronous integration. Always verify signatures and handle duplicates.

## Polling And Long Polling

Polling is simple and reliable for occasional updates but can waste requests. Long polling holds a request open until data is available or timeout, useful when WebSockets/SSE are not available.

# 19. AI And Text-to-SQL Backend Safety

## AI/Text-to-SQL Architecture, Validation, Read-Only Execution, Prompt Injection, Self-Correction, And Auditing

        **Simple Definition**

        Text-to-SQL converts natural language questions into SQL, but the backend must treat model output as untrusted code-like text.

        **Why It Exists**

        It exists to let users query data without writing SQL, but it introduces serious safety risks if generated SQL is executed directly.

        **Internal Working**

        A safe pipeline authenticates the user, retrieves allowed schema context, asks the model for SQL, parses SQL into an AST, allows only SELECT, restricts tables/columns, enforces tenant filters and LIMIT, runs with read-only DB user, applies timeout, logs/audits the query, and limits correction attempts.

        **Request Or Execution Flow**

        Question -> auth/tenant -> schema retrieval/vector search -> SQL generation -> parse/validate -> add constraints -> execute read-only with timeout -> return limited result -> audit.

        **Real Node.js/Express Implementation**

        ```javascript
export async function runTextToSql({ actor, question }, deps) {
  const schemaContext = await deps.schemaRetriever.getAllowedSchema(actor);
  const generated = await deps.ai.generateSql({ question, schemaContext });

  const parsed = deps.sqlParser.parse(generated.sql);

  deps.sqlPolicy.assertSelectOnly(parsed);
  deps.sqlPolicy.assertAllowedTables(parsed, schemaContext.allowedTables);
  deps.sqlPolicy.assertAllowedColumns(parsed, schemaContext.allowedColumns);
  deps.sqlPolicy.assertHasLimit(parsed, 100);
  deps.sqlPolicy.assertTenantFilter(parsed, actor.tenantId);

  return deps.readOnlyDb.queryWithTimeout(generated.sql, [], { timeoutMs: 5000 });
}
```

        **Example Request And Response**

        ```http
POST /api/v1/analytics/questions
Authorization: Bearer <token>

{
  "question": "Show top 5 courses by paid enrollments"
}

HTTP/1.1 200 OK
{
  "data": {
    "sql": "SELECT ... LIMIT 5",
    "rows": []
  }
}
```

        **Common Mistakes**

        - Executing model output directly.
- Using write-capable DB user.
- Relying only on prompt instructions.
- No tenant filter.
- No LIMIT/timeout.
- Infinite self-correction loop.

        **Security Concerns**

        - Read-only DB credentials.
- Parse and validate AST.
- Allowlist tables/columns.
- Enforce tenant/organization filters.
- Block multiple statements.
- Audit generated SQL.
- Avoid exposing sensitive schema.

        **Failure Scenarios**

        - Prompt injection asks for DROP TABLE.
- Model hallucinates table.
- Query scans huge table.
- Tenant data leak.
- Self-correction loops forever.
- Read replica lag returns stale data.

        **Testing Strategy**

        - Unit test SQL policy with malicious queries.
- Integration test read-only DB permissions.
- Timeout tests.
- Tenant isolation tests.
- Prompt injection tests.
- Max-attempt self-correction test.

        **Performance Concerns**

        - Schema retrieval improves accuracy.
- LIMIT and indexes protect DB.
- Complex analytical queries can be expensive.
- Run against replicas or analytics warehouse when possible.

        **Alternatives**

        - Predefined reports.
- Query builder UI.
- Semantic layer/metrics store.
- Human approval for generated SQL.
- Data warehouse sandbox.

        **Trade-Offs**

        Text-to-SQL is powerful but high-risk. Deterministic guardrails matter more than model prompts. Safer alternatives are predefined reports or a constrained query builder.

        **Strong Interview Answer**

        ```text
        For Text-to-SQL, I would never directly execute AI output. I would generate SQL using schema context, parse it, allow only safe SELECT queries, restrict tables and columns, enforce tenant filters and LIMIT, run with read-only credentials and timeout, and audit every query.
        ```

        **Interviewer Follow-Ups With Answers**

        - **Why FAISS/vector search?** To retrieve relevant schema docs/examples so the model generates better SQL; it does not replace validation.
- **How terminate self-correction?** Set a max attempt count, usually 2 or 3, then return a safe failure.

        **Connection To Your Projects**

        This directly connects to your Text-to-SQL project. Be very clear: model generation is not the safety boundary; validation and DB permissions are.

        **Small Exercise**

        Classify this generated SQL: `SELECT * FROM users; DROP TABLE payments;`.

        **Exercise Solution / Expected Reasoning**

        Reject it because it contains multiple statements and a destructive command. Even if the first SELECT is valid, the whole generated output is unsafe.

# 20. Backend System Design Playbook

## URL Shortener

        **Clarifying Questions**

        - Who are the users and what actions are most frequent?
        - Is the system read-heavy, write-heavy, or balanced?
        - What is the required consistency level?
        - What failure should be handled without manual intervention?
        - What audit or compliance requirements exist?

        **Functional Requirements**

        Create short URLs, redirect to long URLs, optionally track clicks and expiry.

        **Non-Functional Requirements**

        - Low latency for common reads.
        - Correctness for state-changing writes.
        - Idempotent write APIs where retries are likely.
        - Clear authentication and authorization.
        - Structured logs, metrics, alerts, and audit records.
        - Horizontal scaling path for API servers.

        **Scale Estimates**

        In an interview, start with assumptions instead of fake precision. For example: "If we expect 10,000 daily active users and each user performs 20 reads and 2 writes per day, reads dominate. I would first optimize indexes and caching for reads, while keeping writes transactionally correct." Update the design if the interviewer gives concrete numbers.

        **API Contracts**

        ```http
POST /api/v1/urls
GET /:shortCode
GET /api/v1/urls/:shortCode/stats
```

        **Data Model**

        `urls(id, short_code unique, long_url, created_by, expires_at, created_at)` and `clicks(id, url_id, ip_hash, user_agent, created_at)`.

        **Indexes**

        - Index foreign keys used in joins or lookups.
        - Add composite indexes matching the most common filter and sort pattern.
        - Add unique indexes for idempotency keys, public identifiers, and external provider IDs.
        - Re-check index usefulness with `EXPLAIN` or database-specific explain tools.

        **High-Level Architecture**

        ```text
        Client
        -> Load balancer / reverse proxy
        -> Express API
        -> Service layer
        -> Database
        -> Redis cache where useful
        -> Background jobs for slow or retryable work
        -> External providers when needed
        ```

        **Detailed Request Flow**

        Create validates URL and generates unique code. Redirect reads code, checks expiry, optionally logs click asynchronously, and returns 302/301.

        **Authentication And Authorization**

        Use JWT or session authentication for users. Use RBAC for broad route permissions and ownership checks in the service layer for resource-level authorization. For service-to-service calls, use signed API keys, mTLS, or short-lived service tokens depending on infrastructure maturity.

        **Caching**

        Cache hot read-only or slowly changing data. Avoid caching personalized sensitive responses in shared caches unless the cache key includes all authorization dimensions and the response is safe to reuse.

        **Concurrency**

        Protect critical writes with unique constraints, transactions, optimistic version columns, or locks depending on the operation. Assume retries and duplicate requests will happen.

        **Idempotency**

        Use idempotency keys for externally retried write operations. Store request hash, status, response, and an `in_progress` state to handle concurrent duplicates.

        **Failure Handling**

        If DB is down, redirects fail unless cached. Use Redis/CDN cache for hot codes and async click logging.

        **Consistency**

        Use strong consistency for money, ownership, and permission changes. Use eventual consistency for notifications, analytics, search indexing, and emails.

        **Security**

        Validate inputs, enforce ownership, use parameterized queries, avoid sensitive logs, rate limit expensive actions, and expose only safe response fields.

        **Testing**

        Test happy path, validation failure, auth failure, authorization failure, duplicate requests, external provider failure, database failure, and concurrency cases.

        **Monitoring**

        Track request latency, error rate, dependency latency, queue depth, job failures, and business-specific counters.

        **Scaling Plan**

        Start with stateless API instances, proper indexes, and background jobs. Add Redis caching, read replicas, partitioning, or sharding only when metrics justify them.

        **Trade-Offs**

        The simplest correct design is usually better than a distributed design too early. Add queues, caches, and replicas when they solve a concrete bottleneck or reliability problem.

        **Interview Follow-Up Questions**

        - **301 or 302?** 301 for permanent stable links, 302 if destination may change.
- **How avoid code collision?** Unique constraint and retry generation.

## Authentication Service

        **Clarifying Questions**

        - Who are the users and what actions are most frequent?
        - Is the system read-heavy, write-heavy, or balanced?
        - What is the required consistency level?
        - What failure should be handled without manual intervention?
        - What audit or compliance requirements exist?

        **Functional Requirements**

        Register, login, refresh, logout, manage devices/sessions.

        **Non-Functional Requirements**

        - Low latency for common reads.
        - Correctness for state-changing writes.
        - Idempotent write APIs where retries are likely.
        - Clear authentication and authorization.
        - Structured logs, metrics, alerts, and audit records.
        - Horizontal scaling path for API servers.

        **Scale Estimates**

        In an interview, start with assumptions instead of fake precision. For example: "If we expect 10,000 daily active users and each user performs 20 reads and 2 writes per day, reads dominate. I would first optimize indexes and caching for reads, while keeping writes transactionally correct." Update the design if the interviewer gives concrete numbers.

        **API Contracts**

        ```http
POST /auth/register
POST /auth/login
POST /auth/refresh
POST /auth/logout
GET /auth/sessions
DELETE /auth/sessions/:id
```

        **Data Model**

        `users`, `refresh_tokens`, `login_attempts`, `password_reset_tokens`, `audit_logs`.

        **Indexes**

        - Index foreign keys used in joins or lookups.
        - Add composite indexes matching the most common filter and sort pattern.
        - Add unique indexes for idempotency keys, public identifiers, and external provider IDs.
        - Re-check index usefulness with `EXPLAIN` or database-specific explain tools.

        **High-Level Architecture**

        ```text
        Client
        -> Load balancer / reverse proxy
        -> Express API
        -> Service layer
        -> Database
        -> Redis cache where useful
        -> Background jobs for slow or retryable work
        -> External providers when needed
        ```

        **Detailed Request Flow**

        Login verifies password, issues access token, rotates refresh token, stores hashed refresh token, returns public user.

        **Authentication And Authorization**

        Use JWT or session authentication for users. Use RBAC for broad route permissions and ownership checks in the service layer for resource-level authorization. For service-to-service calls, use signed API keys, mTLS, or short-lived service tokens depending on infrastructure maturity.

        **Caching**

        Cache hot read-only or slowly changing data. Avoid caching personalized sensitive responses in shared caches unless the cache key includes all authorization dimensions and the response is safe to reuse.

        **Concurrency**

        Protect critical writes with unique constraints, transactions, optimistic version columns, or locks depending on the operation. Assume retries and duplicate requests will happen.

        **Idempotency**

        Use idempotency keys for externally retried write operations. Store request hash, status, response, and an `in_progress` state to handle concurrent duplicates.

        **Failure Handling**

        If refresh reuse detected, revoke family and force re-login.

        **Consistency**

        Use strong consistency for money, ownership, and permission changes. Use eventual consistency for notifications, analytics, search indexing, and emails.

        **Security**

        Validate inputs, enforce ownership, use parameterized queries, avoid sensitive logs, rate limit expensive actions, and expose only safe response fields.

        **Testing**

        Test happy path, validation failure, auth failure, authorization failure, duplicate requests, external provider failure, database failure, and concurrency cases.

        **Monitoring**

        Track request latency, error rate, dependency latency, queue depth, job failures, and business-specific counters.

        **Scaling Plan**

        Start with stateless API instances, proper indexes, and background jobs. Add Redis caching, read replicas, partitioning, or sharding only when metrics justify them.

        **Trade-Offs**

        The simplest correct design is usually better than a distributed design too early. Add queues, caches, and replicas when they solve a concrete bottleneck or reliability problem.

        **Interview Follow-Up Questions**

        - **JWT or sessions?** JWT for stateless APIs; sessions for easier revocation.
- **How prevent brute force?** Rate limits and generic errors.

## Query-Resolution Platform

        **Clarifying Questions**

        - Who are the users and what actions are most frequent?
        - Is the system read-heavy, write-heavy, or balanced?
        - What is the required consistency level?
        - What failure should be handled without manual intervention?
        - What audit or compliance requirements exist?

        **Functional Requirements**

        Students create queries, moderators resolve, admins assign, jobs classify/notify.

        **Non-Functional Requirements**

        - Low latency for common reads.
        - Correctness for state-changing writes.
        - Idempotent write APIs where retries are likely.
        - Clear authentication and authorization.
        - Structured logs, metrics, alerts, and audit records.
        - Horizontal scaling path for API servers.

        **Scale Estimates**

        In an interview, start with assumptions instead of fake precision. For example: "If we expect 10,000 daily active users and each user performs 20 reads and 2 writes per day, reads dominate. I would first optimize indexes and caching for reads, while keeping writes transactionally correct." Update the design if the interviewer gives concrete numbers.

        **API Contracts**

        ```http
POST /queries
GET /queries?assignedTo=me&status=open
GET /queries/:id
POST /queries/:id/replies
POST /queries/:id/assignments
PATCH /queries/:id
```

        **Data Model**

        `queries`, `query_replies`, `assignments`, `users`, `notifications`, indexes on owner/assignee/status.

        **Indexes**

        - Index foreign keys used in joins or lookups.
        - Add composite indexes matching the most common filter and sort pattern.
        - Add unique indexes for idempotency keys, public identifiers, and external provider IDs.
        - Re-check index usefulness with `EXPLAIN` or database-specific explain tools.

        **High-Level Architecture**

        ```text
        Client
        -> Load balancer / reverse proxy
        -> Express API
        -> Service layer
        -> Database
        -> Redis cache where useful
        -> Background jobs for slow or retryable work
        -> External providers when needed
        ```

        **Detailed Request Flow**

        Student creates query -> DB insert -> Inngest classification -> assignment -> notification -> moderator reply -> resolve.

        **Authentication And Authorization**

        Use JWT or session authentication for users. Use RBAC for broad route permissions and ownership checks in the service layer for resource-level authorization. For service-to-service calls, use signed API keys, mTLS, or short-lived service tokens depending on infrastructure maturity.

        **Caching**

        Cache hot read-only or slowly changing data. Avoid caching personalized sensitive responses in shared caches unless the cache key includes all authorization dimensions and the response is safe to reuse.

        **Concurrency**

        Protect critical writes with unique constraints, transactions, optimistic version columns, or locks depending on the operation. Assume retries and duplicate requests will happen.

        **Idempotency**

        Use idempotency keys for externally retried write operations. Store request hash, status, response, and an `in_progress` state to handle concurrent duplicates.

        **Failure Handling**

        AI failure marks classification failed but query remains. Duplicate notification prevented by dedupe key.

        **Consistency**

        Use strong consistency for money, ownership, and permission changes. Use eventual consistency for notifications, analytics, search indexing, and emails.

        **Security**

        Validate inputs, enforce ownership, use parameterized queries, avoid sensitive logs, rate limit expensive actions, and expose only safe response fields.

        **Testing**

        Test happy path, validation failure, auth failure, authorization failure, duplicate requests, external provider failure, database failure, and concurrency cases.

        **Monitoring**

        Track request latency, error rate, dependency latency, queue depth, job failures, and business-specific counters.

        **Scaling Plan**

        Start with stateless API instances, proper indexes, and background jobs. Add Redis caching, read replicas, partitioning, or sharding only when metrics justify them.

        **Trade-Offs**

        The simplest correct design is usually better than a distributed design too early. Add queues, caches, and replicas when they solve a concrete bottleneck or reliability problem.

        **Interview Follow-Up Questions**

        - **How authorize?** Student own, moderator assigned, admin all.
- **Why background jobs?** AI/email are slow and retryable.

## Payment Service

        **Clarifying Questions**

        - Who are the users and what actions are most frequent?
        - Is the system read-heavy, write-heavy, or balanced?
        - What is the required consistency level?
        - What failure should be handled without manual intervention?
        - What audit or compliance requirements exist?

        **Functional Requirements**

        Create payment attempts, verify callbacks, process webhooks, refunds, reconciliation.

        **Non-Functional Requirements**

        - Low latency for common reads.
        - Correctness for state-changing writes.
        - Idempotent write APIs where retries are likely.
        - Clear authentication and authorization.
        - Structured logs, metrics, alerts, and audit records.
        - Horizontal scaling path for API servers.

        **Scale Estimates**

        In an interview, start with assumptions instead of fake precision. For example: "If we expect 10,000 daily active users and each user performs 20 reads and 2 writes per day, reads dominate. I would first optimize indexes and caching for reads, while keeping writes transactionally correct." Update the design if the interviewer gives concrete numbers.

        **API Contracts**

        ```http
POST /orders/:id/payment-attempts
POST /payments/verify
POST /webhooks/razorpay
GET /orders/:id/payment-status
POST /payments/:id/refunds
```

        **Data Model**

        `orders`, `payment_attempts`, `payments`, `payment_events`, `refunds`, `idempotency_keys`.

        **Indexes**

        - Index foreign keys used in joins or lookups.
        - Add composite indexes matching the most common filter and sort pattern.
        - Add unique indexes for idempotency keys, public identifiers, and external provider IDs.
        - Re-check index usefulness with `EXPLAIN` or database-specific explain tools.

        **High-Level Architecture**

        ```text
        Client
        -> Load balancer / reverse proxy
        -> Express API
        -> Service layer
        -> Database
        -> Redis cache where useful
        -> Background jobs for slow or retryable work
        -> External providers when needed
        ```

        **Detailed Request Flow**

        Create local attempt -> Razorpay order -> checkout -> verify signature -> webhook confirms -> transaction updates -> audit.

        **Authentication And Authorization**

        Use JWT or session authentication for users. Use RBAC for broad route permissions and ownership checks in the service layer for resource-level authorization. For service-to-service calls, use signed API keys, mTLS, or short-lived service tokens depending on infrastructure maturity.

        **Caching**

        Cache hot read-only or slowly changing data. Avoid caching personalized sensitive responses in shared caches unless the cache key includes all authorization dimensions and the response is safe to reuse.

        **Concurrency**

        Protect critical writes with unique constraints, transactions, optimistic version columns, or locks depending on the operation. Assume retries and duplicate requests will happen.

        **Idempotency**

        Use idempotency keys for externally retried write operations. Store request hash, status, response, and an `in_progress` state to handle concurrent duplicates.

        **Failure Handling**

        Webhook duplicate/out-of-order handled idempotently. Reconciliation repairs local/provider mismatch.

        **Consistency**

        Use strong consistency for money, ownership, and permission changes. Use eventual consistency for notifications, analytics, search indexing, and emails.

        **Security**

        Validate inputs, enforce ownership, use parameterized queries, avoid sensitive logs, rate limit expensive actions, and expose only safe response fields.

        **Testing**

        Test happy path, validation failure, auth failure, authorization failure, duplicate requests, external provider failure, database failure, and concurrency cases.

        **Monitoring**

        Track request latency, error rate, dependency latency, queue depth, job failures, and business-specific counters.

        **Scaling Plan**

        Start with stateless API instances, proper indexes, and background jobs. Add Redis caching, read replicas, partitioning, or sharding only when metrics justify them.

        **Trade-Offs**

        The simplest correct design is usually better than a distributed design too early. Add queues, caches, and replicas when they solve a concrete bottleneck or reliability problem.

        **Interview Follow-Up Questions**

        - **Where use idempotency?** Create attempts, verification, refunds, webhook events.
- **Why audit?** Money workflows require traceability.

## Notification Service

        **Clarifying Questions**

        - Who are the users and what actions are most frequent?
        - Is the system read-heavy, write-heavy, or balanced?
        - What is the required consistency level?
        - What failure should be handled without manual intervention?
        - What audit or compliance requirements exist?

        **Functional Requirements**

        Send email/SMS/in-app notifications with retries and deduplication.

        **Non-Functional Requirements**

        - Low latency for common reads.
        - Correctness for state-changing writes.
        - Idempotent write APIs where retries are likely.
        - Clear authentication and authorization.
        - Structured logs, metrics, alerts, and audit records.
        - Horizontal scaling path for API servers.

        **Scale Estimates**

        In an interview, start with assumptions instead of fake precision. For example: "If we expect 10,000 daily active users and each user performs 20 reads and 2 writes per day, reads dominate. I would first optimize indexes and caching for reads, while keeping writes transactionally correct." Update the design if the interviewer gives concrete numbers.

        **API Contracts**

        ```http
POST /notifications
GET /notifications/:id
GET /users/me/notifications
PATCH /notifications/:id/read
```

        **Data Model**

        `notifications(id, type, recipient_id, status, dedupe_key unique)`, `notification_attempts`.

        **Indexes**

        - Index foreign keys used in joins or lookups.
        - Add composite indexes matching the most common filter and sort pattern.
        - Add unique indexes for idempotency keys, public identifiers, and external provider IDs.
        - Re-check index usefulness with `EXPLAIN` or database-specific explain tools.

        **High-Level Architecture**

        ```text
        Client
        -> Load balancer / reverse proxy
        -> Express API
        -> Service layer
        -> Database
        -> Redis cache where useful
        -> Background jobs for slow or retryable work
        -> External providers when needed
        ```

        **Detailed Request Flow**

        API/job creates notification -> worker sends provider call -> update status -> retry failures.

        **Authentication And Authorization**

        Use JWT or session authentication for users. Use RBAC for broad route permissions and ownership checks in the service layer for resource-level authorization. For service-to-service calls, use signed API keys, mTLS, or short-lived service tokens depending on infrastructure maturity.

        **Caching**

        Cache hot read-only or slowly changing data. Avoid caching personalized sensitive responses in shared caches unless the cache key includes all authorization dimensions and the response is safe to reuse.

        **Concurrency**

        Protect critical writes with unique constraints, transactions, optimistic version columns, or locks depending on the operation. Assume retries and duplicate requests will happen.

        **Idempotency**

        Use idempotency keys for externally retried write operations. Store request hash, status, response, and an `in_progress` state to handle concurrent duplicates.

        **Failure Handling**

        Provider outage moves exhausted jobs to DLQ; dedupe key prevents duplicate emails.

        **Consistency**

        Use strong consistency for money, ownership, and permission changes. Use eventual consistency for notifications, analytics, search indexing, and emails.

        **Security**

        Validate inputs, enforce ownership, use parameterized queries, avoid sensitive logs, rate limit expensive actions, and expose only safe response fields.

        **Testing**

        Test happy path, validation failure, auth failure, authorization failure, duplicate requests, external provider failure, database failure, and concurrency cases.

        **Monitoring**

        Track request latency, error rate, dependency latency, queue depth, job failures, and business-specific counters.

        **Scaling Plan**

        Start with stateless API instances, proper indexes, and background jobs. Add Redis caching, read replicas, partitioning, or sharding only when metrics justify them.

        **Trade-Offs**

        The simplest correct design is usually better than a distributed design too early. Add queues, caches, and replicas when they solve a concrete bottleneck or reliability problem.

        **Interview Follow-Up Questions**

        - **Sync or async?** Usually async.
- **How avoid duplicates?** Unique dedupe key.

## File Upload Service

        **Clarifying Questions**

        - Who are the users and what actions are most frequent?
        - Is the system read-heavy, write-heavy, or balanced?
        - What is the required consistency level?
        - What failure should be handled without manual intervention?
        - What audit or compliance requirements exist?

        **Functional Requirements**

        Upload user files safely and process them.

        **Non-Functional Requirements**

        - Low latency for common reads.
        - Correctness for state-changing writes.
        - Idempotent write APIs where retries are likely.
        - Clear authentication and authorization.
        - Structured logs, metrics, alerts, and audit records.
        - Horizontal scaling path for API servers.

        **Scale Estimates**

        In an interview, start with assumptions instead of fake precision. For example: "If we expect 10,000 daily active users and each user performs 20 reads and 2 writes per day, reads dominate. I would first optimize indexes and caching for reads, while keeping writes transactionally correct." Update the design if the interviewer gives concrete numbers.

        **API Contracts**

        ```http
POST /files/upload-url
POST /files/:id/complete
GET /files/:id
DELETE /files/:id
```

        **Data Model**

        `files(id, owner_id, storage_key, status, size, mime_type, checksum)`.

        **Indexes**

        - Index foreign keys used in joins or lookups.
        - Add composite indexes matching the most common filter and sort pattern.
        - Add unique indexes for idempotency keys, public identifiers, and external provider IDs.
        - Re-check index usefulness with `EXPLAIN` or database-specific explain tools.

        **High-Level Architecture**

        ```text
        Client
        -> Load balancer / reverse proxy
        -> Express API
        -> Service layer
        -> Database
        -> Redis cache where useful
        -> Background jobs for slow or retryable work
        -> External providers when needed
        ```

        **Detailed Request Flow**

        Backend returns signed URL -> client uploads to object storage -> completion triggers scan/job -> file becomes available.

        **Authentication And Authorization**

        Use JWT or session authentication for users. Use RBAC for broad route permissions and ownership checks in the service layer for resource-level authorization. For service-to-service calls, use signed API keys, mTLS, or short-lived service tokens depending on infrastructure maturity.

        **Caching**

        Cache hot read-only or slowly changing data. Avoid caching personalized sensitive responses in shared caches unless the cache key includes all authorization dimensions and the response is safe to reuse.

        **Concurrency**

        Protect critical writes with unique constraints, transactions, optimistic version columns, or locks depending on the operation. Assume retries and duplicate requests will happen.

        **Idempotency**

        Use idempotency keys for externally retried write operations. Store request hash, status, response, and an `in_progress` state to handle concurrent duplicates.

        **Failure Handling**

        If processing fails, status failed and retry/cleanup job handles orphaned file.

        **Consistency**

        Use strong consistency for money, ownership, and permission changes. Use eventual consistency for notifications, analytics, search indexing, and emails.

        **Security**

        Validate inputs, enforce ownership, use parameterized queries, avoid sensitive logs, rate limit expensive actions, and expose only safe response fields.

        **Testing**

        Test happy path, validation failure, auth failure, authorization failure, duplicate requests, external provider failure, database failure, and concurrency cases.

        **Monitoring**

        Track request latency, error rate, dependency latency, queue depth, job failures, and business-specific counters.

        **Scaling Plan**

        Start with stateless API instances, proper indexes, and background jobs. Add Redis caching, read replicas, partitioning, or sharding only when metrics justify them.

        **Trade-Offs**

        The simplest correct design is usually better than a distributed design too early. Add queues, caches, and replicas when they solve a concrete bottleneck or reliability problem.

        **Interview Follow-Up Questions**

        - **Why direct upload?** Avoid loading app servers with large files.
- **How secure?** Size/type limits, random keys, scanning.

## Rate Limiter

        **Clarifying Questions**

        - Who are the users and what actions are most frequent?
        - Is the system read-heavy, write-heavy, or balanced?
        - What is the required consistency level?
        - What failure should be handled without manual intervention?
        - What audit or compliance requirements exist?

        **Functional Requirements**

        Limit request frequency by IP/user/API key.

        **Non-Functional Requirements**

        - Low latency for common reads.
        - Correctness for state-changing writes.
        - Idempotent write APIs where retries are likely.
        - Clear authentication and authorization.
        - Structured logs, metrics, alerts, and audit records.
        - Horizontal scaling path for API servers.

        **Scale Estimates**

        In an interview, start with assumptions instead of fake precision. For example: "If we expect 10,000 daily active users and each user performs 20 reads and 2 writes per day, reads dominate. I would first optimize indexes and caching for reads, while keeping writes transactionally correct." Update the design if the interviewer gives concrete numbers.

        **API Contracts**

        ```http
Any endpoint returns 429 with Retry-After when limited
```

        **Data Model**

        Redis keys for counters/buckets, optional DB configs per plan.

        **Indexes**

        - Index foreign keys used in joins or lookups.
        - Add composite indexes matching the most common filter and sort pattern.
        - Add unique indexes for idempotency keys, public identifiers, and external provider IDs.
        - Re-check index usefulness with `EXPLAIN` or database-specific explain tools.

        **High-Level Architecture**

        ```text
        Client
        -> Load balancer / reverse proxy
        -> Express API
        -> Service layer
        -> Database
        -> Redis cache where useful
        -> Background jobs for slow or retryable work
        -> External providers when needed
        ```

        **Detailed Request Flow**

        Request computes key -> atomic Redis increment/token bucket -> allow or reject.

        **Authentication And Authorization**

        Use JWT or session authentication for users. Use RBAC for broad route permissions and ownership checks in the service layer for resource-level authorization. For service-to-service calls, use signed API keys, mTLS, or short-lived service tokens depending on infrastructure maturity.

        **Caching**

        Cache hot read-only or slowly changing data. Avoid caching personalized sensitive responses in shared caches unless the cache key includes all authorization dimensions and the response is safe to reuse.

        **Concurrency**

        Protect critical writes with unique constraints, transactions, optimistic version columns, or locks depending on the operation. Assume retries and duplicate requests will happen.

        **Idempotency**

        Use idempotency keys for externally retried write operations. Store request hash, status, response, and an `in_progress` state to handle concurrent duplicates.

        **Failure Handling**

        If Redis down, decide fail-open for non-critical or fail-closed for sensitive endpoints.

        **Consistency**

        Use strong consistency for money, ownership, and permission changes. Use eventual consistency for notifications, analytics, search indexing, and emails.

        **Security**

        Validate inputs, enforce ownership, use parameterized queries, avoid sensitive logs, rate limit expensive actions, and expose only safe response fields.

        **Testing**

        Test happy path, validation failure, auth failure, authorization failure, duplicate requests, external provider failure, database failure, and concurrency cases.

        **Monitoring**

        Track request latency, error rate, dependency latency, queue depth, job failures, and business-specific counters.

        **Scaling Plan**

        Start with stateless API instances, proper indexes, and background jobs. Add Redis caching, read replicas, partitioning, or sharding only when metrics justify them.

        **Trade-Offs**

        The simplest correct design is usually better than a distributed design too early. Add queues, caches, and replicas when they solve a concrete bottleneck or reliability problem.

        **Interview Follow-Up Questions**

        - **Fixed vs sliding window?** Fixed simple but bursty; sliding/token bucket smoother.
- **Multi-instance?** Use shared Redis.

## Background Job System

        **Clarifying Questions**

        - Who are the users and what actions are most frequent?
        - Is the system read-heavy, write-heavy, or balanced?
        - What is the required consistency level?
        - What failure should be handled without manual intervention?
        - What audit or compliance requirements exist?

        **Functional Requirements**

        Accept jobs, run workers, retry, schedule, monitor.

        **Non-Functional Requirements**

        - Low latency for common reads.
        - Correctness for state-changing writes.
        - Idempotent write APIs where retries are likely.
        - Clear authentication and authorization.
        - Structured logs, metrics, alerts, and audit records.
        - Horizontal scaling path for API servers.

        **Scale Estimates**

        In an interview, start with assumptions instead of fake precision. For example: "If we expect 10,000 daily active users and each user performs 20 reads and 2 writes per day, reads dominate. I would first optimize indexes and caching for reads, while keeping writes transactionally correct." Update the design if the interviewer gives concrete numbers.

        **API Contracts**

        ```http
POST /jobs
GET /jobs/:id
POST /jobs/:id/cancel
```

        **Data Model**

        `jobs`, `job_attempts`, `dead_letter_jobs`, `scheduled_jobs`.

        **Indexes**

        - Index foreign keys used in joins or lookups.
        - Add composite indexes matching the most common filter and sort pattern.
        - Add unique indexes for idempotency keys, public identifiers, and external provider IDs.
        - Re-check index usefulness with `EXPLAIN` or database-specific explain tools.

        **High-Level Architecture**

        ```text
        Client
        -> Load balancer / reverse proxy
        -> Express API
        -> Service layer
        -> Database
        -> Redis cache where useful
        -> Background jobs for slow or retryable work
        -> External providers when needed
        ```

        **Detailed Request Flow**

        Producer inserts job -> worker claims -> executes -> retries with backoff -> marks complete/failed.

        **Authentication And Authorization**

        Use JWT or session authentication for users. Use RBAC for broad route permissions and ownership checks in the service layer for resource-level authorization. For service-to-service calls, use signed API keys, mTLS, or short-lived service tokens depending on infrastructure maturity.

        **Caching**

        Cache hot read-only or slowly changing data. Avoid caching personalized sensitive responses in shared caches unless the cache key includes all authorization dimensions and the response is safe to reuse.

        **Concurrency**

        Protect critical writes with unique constraints, transactions, optimistic version columns, or locks depending on the operation. Assume retries and duplicate requests will happen.

        **Idempotency**

        Use idempotency keys for externally retried write operations. Store request hash, status, response, and an `in_progress` state to handle concurrent duplicates.

        **Failure Handling**

        Crash during job needs visibility timeout/lease and idempotent handler.

        **Consistency**

        Use strong consistency for money, ownership, and permission changes. Use eventual consistency for notifications, analytics, search indexing, and emails.

        **Security**

        Validate inputs, enforce ownership, use parameterized queries, avoid sensitive logs, rate limit expensive actions, and expose only safe response fields.

        **Testing**

        Test happy path, validation failure, auth failure, authorization failure, duplicate requests, external provider failure, database failure, and concurrency cases.

        **Monitoring**

        Track request latency, error rate, dependency latency, queue depth, job failures, and business-specific counters.

        **Scaling Plan**

        Start with stateless API instances, proper indexes, and background jobs. Add Redis caching, read replicas, partitioning, or sharding only when metrics justify them.

        **Trade-Offs**

        The simplest correct design is usually better than a distributed design too early. Add queues, caches, and replicas when they solve a concrete bottleneck or reliability problem.

        **Interview Follow-Up Questions**

        - **Exactly once?** Hard; use at-least-once plus idempotency.
- **What is DLQ?** Failed jobs after retries.

## Order-Management API

        **Clarifying Questions**

        - Who are the users and what actions are most frequent?
        - Is the system read-heavy, write-heavy, or balanced?
        - What is the required consistency level?
        - What failure should be handled without manual intervention?
        - What audit or compliance requirements exist?

        **Functional Requirements**

        Create orders, update status, pay, cancel, list history.

        **Non-Functional Requirements**

        - Low latency for common reads.
        - Correctness for state-changing writes.
        - Idempotent write APIs where retries are likely.
        - Clear authentication and authorization.
        - Structured logs, metrics, alerts, and audit records.
        - Horizontal scaling path for API servers.

        **Scale Estimates**

        In an interview, start with assumptions instead of fake precision. For example: "If we expect 10,000 daily active users and each user performs 20 reads and 2 writes per day, reads dominate. I would first optimize indexes and caching for reads, while keeping writes transactionally correct." Update the design if the interviewer gives concrete numbers.

        **API Contracts**

        ```http
POST /orders
GET /orders/:id
PATCH /orders/:id
POST /orders/:id/cancellations
```

        **Data Model**

        `orders`, `order_items`, `inventory_reservations`, `payments`, `order_events`.

        **Indexes**

        - Index foreign keys used in joins or lookups.
        - Add composite indexes matching the most common filter and sort pattern.
        - Add unique indexes for idempotency keys, public identifiers, and external provider IDs.
        - Re-check index usefulness with `EXPLAIN` or database-specific explain tools.

        **High-Level Architecture**

        ```text
        Client
        -> Load balancer / reverse proxy
        -> Express API
        -> Service layer
        -> Database
        -> Redis cache where useful
        -> Background jobs for slow or retryable work
        -> External providers when needed
        ```

        **Detailed Request Flow**

        Create order validates items -> reserves inventory -> creates order -> payment attempt -> state transitions.

        **Authentication And Authorization**

        Use JWT or session authentication for users. Use RBAC for broad route permissions and ownership checks in the service layer for resource-level authorization. For service-to-service calls, use signed API keys, mTLS, or short-lived service tokens depending on infrastructure maturity.

        **Caching**

        Cache hot read-only or slowly changing data. Avoid caching personalized sensitive responses in shared caches unless the cache key includes all authorization dimensions and the response is safe to reuse.

        **Concurrency**

        Protect critical writes with unique constraints, transactions, optimistic version columns, or locks depending on the operation. Assume retries and duplicate requests will happen.

        **Idempotency**

        Use idempotency keys for externally retried write operations. Store request hash, status, response, and an `in_progress` state to handle concurrent duplicates.

        **Failure Handling**

        Concurrent purchase needs inventory locking or reservation constraints.

        **Consistency**

        Use strong consistency for money, ownership, and permission changes. Use eventual consistency for notifications, analytics, search indexing, and emails.

        **Security**

        Validate inputs, enforce ownership, use parameterized queries, avoid sensitive logs, rate limit expensive actions, and expose only safe response fields.

        **Testing**

        Test happy path, validation failure, auth failure, authorization failure, duplicate requests, external provider failure, database failure, and concurrency cases.

        **Monitoring**

        Track request latency, error rate, dependency latency, queue depth, job failures, and business-specific counters.

        **Scaling Plan**

        Start with stateless API instances, proper indexes, and background jobs. Add Redis caching, read replicas, partitioning, or sharding only when metrics justify them.

        **Trade-Offs**

        The simplest correct design is usually better than a distributed design too early. Add queues, caches, and replicas when they solve a concrete bottleneck or reliability problem.

        **Interview Follow-Up Questions**

        - **How model state?** Order state machine.
- **How avoid oversell?** Transactions/locks or reservation system.

## Webhook-Processing System

        **Clarifying Questions**

        - Who are the users and what actions are most frequent?
        - Is the system read-heavy, write-heavy, or balanced?
        - What is the required consistency level?
        - What failure should be handled without manual intervention?
        - What audit or compliance requirements exist?

        **Functional Requirements**

        Receive external events, verify, store, process idempotently.

        **Non-Functional Requirements**

        - Low latency for common reads.
        - Correctness for state-changing writes.
        - Idempotent write APIs where retries are likely.
        - Clear authentication and authorization.
        - Structured logs, metrics, alerts, and audit records.
        - Horizontal scaling path for API servers.

        **Scale Estimates**

        In an interview, start with assumptions instead of fake precision. For example: "If we expect 10,000 daily active users and each user performs 20 reads and 2 writes per day, reads dominate. I would first optimize indexes and caching for reads, while keeping writes transactionally correct." Update the design if the interviewer gives concrete numbers.

        **API Contracts**

        ```http
POST /webhooks/:provider
GET /webhook-events/:id
```

        **Data Model**

        `webhook_events(provider, event_id unique, status, raw_payload_hash)`.

        **Indexes**

        - Index foreign keys used in joins or lookups.
        - Add composite indexes matching the most common filter and sort pattern.
        - Add unique indexes for idempotency keys, public identifiers, and external provider IDs.
        - Re-check index usefulness with `EXPLAIN` or database-specific explain tools.

        **High-Level Architecture**

        ```text
        Client
        -> Load balancer / reverse proxy
        -> Express API
        -> Service layer
        -> Database
        -> Redis cache where useful
        -> Background jobs for slow or retryable work
        -> External providers when needed
        ```

        **Detailed Request Flow**

        Raw request -> verify signature -> store event -> enqueue processing -> return 200 -> worker updates domain state.

        **Authentication And Authorization**

        Use JWT or session authentication for users. Use RBAC for broad route permissions and ownership checks in the service layer for resource-level authorization. For service-to-service calls, use signed API keys, mTLS, or short-lived service tokens depending on infrastructure maturity.

        **Caching**

        Cache hot read-only or slowly changing data. Avoid caching personalized sensitive responses in shared caches unless the cache key includes all authorization dimensions and the response is safe to reuse.

        **Concurrency**

        Protect critical writes with unique constraints, transactions, optimistic version columns, or locks depending on the operation. Assume retries and duplicate requests will happen.

        **Idempotency**

        Use idempotency keys for externally retried write operations. Store request hash, status, response, and an `in_progress` state to handle concurrent duplicates.

        **Failure Handling**

        Duplicate event returns 200. Invalid signature returns 400. Processing failure retries.

        **Consistency**

        Use strong consistency for money, ownership, and permission changes. Use eventual consistency for notifications, analytics, search indexing, and emails.

        **Security**

        Validate inputs, enforce ownership, use parameterized queries, avoid sensitive logs, rate limit expensive actions, and expose only safe response fields.

        **Testing**

        Test happy path, validation failure, auth failure, authorization failure, duplicate requests, external provider failure, database failure, and concurrency cases.

        **Monitoring**

        Track request latency, error rate, dependency latency, queue depth, job failures, and business-specific counters.

        **Scaling Plan**

        Start with stateless API instances, proper indexes, and background jobs. Add Redis caching, read replicas, partitioning, or sharding only when metrics justify them.

        **Trade-Offs**

        The simplest correct design is usually better than a distributed design too early. Add queues, caches, and replicas when they solve a concrete bottleneck or reliability problem.

        **Interview Follow-Up Questions**

        - **Why respond fast?** Provider retries if timeout.
- **Why raw body?** Signature computed over exact bytes.

## Text-to-SQL Backend

        **Clarifying Questions**

        - Who are the users and what actions are most frequent?
        - Is the system read-heavy, write-heavy, or balanced?
        - What is the required consistency level?
        - What failure should be handled without manual intervention?
        - What audit or compliance requirements exist?

        **Functional Requirements**

        Answer natural language questions safely with generated SQL.

        **Non-Functional Requirements**

        - Low latency for common reads.
        - Correctness for state-changing writes.
        - Idempotent write APIs where retries are likely.
        - Clear authentication and authorization.
        - Structured logs, metrics, alerts, and audit records.
        - Horizontal scaling path for API servers.

        **Scale Estimates**

        In an interview, start with assumptions instead of fake precision. For example: "If we expect 10,000 daily active users and each user performs 20 reads and 2 writes per day, reads dominate. I would first optimize indexes and caching for reads, while keeping writes transactionally correct." Update the design if the interviewer gives concrete numbers.

        **API Contracts**

        ```http
POST /analytics/questions
GET /analytics/questions/:id
```

        **Data Model**

        `query_audits`, schema metadata tables/docs, optional vector index.

        **Indexes**

        - Index foreign keys used in joins or lookups.
        - Add composite indexes matching the most common filter and sort pattern.
        - Add unique indexes for idempotency keys, public identifiers, and external provider IDs.
        - Re-check index usefulness with `EXPLAIN` or database-specific explain tools.

        **High-Level Architecture**

        ```text
        Client
        -> Load balancer / reverse proxy
        -> Express API
        -> Service layer
        -> Database
        -> Redis cache where useful
        -> Background jobs for slow or retryable work
        -> External providers when needed
        ```

        **Detailed Request Flow**

        Auth -> schema retrieval -> LLM -> parse/validate -> read-only execute -> audit -> response.

        **Authentication And Authorization**

        Use JWT or session authentication for users. Use RBAC for broad route permissions and ownership checks in the service layer for resource-level authorization. For service-to-service calls, use signed API keys, mTLS, or short-lived service tokens depending on infrastructure maturity.

        **Caching**

        Cache hot read-only or slowly changing data. Avoid caching personalized sensitive responses in shared caches unless the cache key includes all authorization dimensions and the response is safe to reuse.

        **Concurrency**

        Protect critical writes with unique constraints, transactions, optimistic version columns, or locks depending on the operation. Assume retries and duplicate requests will happen.

        **Idempotency**

        Use idempotency keys for externally retried write operations. Store request hash, status, response, and an `in_progress` state to handle concurrent duplicates.

        **Failure Handling**

        Unsafe SQL rejected; expensive query timed out; self-correction limited.

        **Consistency**

        Use strong consistency for money, ownership, and permission changes. Use eventual consistency for notifications, analytics, search indexing, and emails.

        **Security**

        Validate inputs, enforce ownership, use parameterized queries, avoid sensitive logs, rate limit expensive actions, and expose only safe response fields.

        **Testing**

        Test happy path, validation failure, auth failure, authorization failure, duplicate requests, external provider failure, database failure, and concurrency cases.

        **Monitoring**

        Track request latency, error rate, dependency latency, queue depth, job failures, and business-specific counters.

        **Scaling Plan**

        Start with stateless API instances, proper indexes, and background jobs. Add Redis caching, read replicas, partitioning, or sharding only when metrics justify them.

        **Trade-Offs**

        The simplest correct design is usually better than a distributed design too early. Add queues, caches, and replicas when they solve a concrete bottleneck or reliability problem.

        **Interview Follow-Up Questions**

        - **Why read-only user?** Defense in depth.
- **How handle tenant?** Enforce tenant filter deterministically.

## Audit-Log Service

        **Clarifying Questions**

        - Who are the users and what actions are most frequent?
        - Is the system read-heavy, write-heavy, or balanced?
        - What is the required consistency level?
        - What failure should be handled without manual intervention?
        - What audit or compliance requirements exist?

        **Functional Requirements**

        Append-only record of important actions.

        **Non-Functional Requirements**

        - Low latency for common reads.
        - Correctness for state-changing writes.
        - Idempotent write APIs where retries are likely.
        - Clear authentication and authorization.
        - Structured logs, metrics, alerts, and audit records.
        - Horizontal scaling path for API servers.

        **Scale Estimates**

        In an interview, start with assumptions instead of fake precision. For example: "If we expect 10,000 daily active users and each user performs 20 reads and 2 writes per day, reads dominate. I would first optimize indexes and caching for reads, while keeping writes transactionally correct." Update the design if the interviewer gives concrete numbers.

        **API Contracts**

        ```http
GET /audit-logs?actor=&resource=&from=&to=
```

        **Data Model**

        `audit_logs(id, actor_id, action, resource_type, resource_id, metadata_json, request_id, created_at)`.

        **Indexes**

        - Index foreign keys used in joins or lookups.
        - Add composite indexes matching the most common filter and sort pattern.
        - Add unique indexes for idempotency keys, public identifiers, and external provider IDs.
        - Re-check index usefulness with `EXPLAIN` or database-specific explain tools.

        **High-Level Architecture**

        ```text
        Client
        -> Load balancer / reverse proxy
        -> Express API
        -> Service layer
        -> Database
        -> Redis cache where useful
        -> Background jobs for slow or retryable work
        -> External providers when needed
        ```

        **Detailed Request Flow**

        Service emits audit event inside transaction or outbox; logs are queryable by admin.

        **Authentication And Authorization**

        Use JWT or session authentication for users. Use RBAC for broad route permissions and ownership checks in the service layer for resource-level authorization. For service-to-service calls, use signed API keys, mTLS, or short-lived service tokens depending on infrastructure maturity.

        **Caching**

        Cache hot read-only or slowly changing data. Avoid caching personalized sensitive responses in shared caches unless the cache key includes all authorization dimensions and the response is safe to reuse.

        **Concurrency**

        Protect critical writes with unique constraints, transactions, optimistic version columns, or locks depending on the operation. Assume retries and duplicate requests will happen.

        **Idempotency**

        Use idempotency keys for externally retried write operations. Store request hash, status, response, and an `in_progress` state to handle concurrent duplicates.

        **Failure Handling**

        Audit write failure policy depends on criticality: block sensitive actions or outbox retry.

        **Consistency**

        Use strong consistency for money, ownership, and permission changes. Use eventual consistency for notifications, analytics, search indexing, and emails.

        **Security**

        Validate inputs, enforce ownership, use parameterized queries, avoid sensitive logs, rate limit expensive actions, and expose only safe response fields.

        **Testing**

        Test happy path, validation failure, auth failure, authorization failure, duplicate requests, external provider failure, database failure, and concurrency cases.

        **Monitoring**

        Track request latency, error rate, dependency latency, queue depth, job failures, and business-specific counters.

        **Scaling Plan**

        Start with stateless API instances, proper indexes, and background jobs. Add Redis caching, read replicas, partitioning, or sharding only when metrics justify them.

        **Trade-Offs**

        The simplest correct design is usually better than a distributed design too early. Add queues, caches, and replicas when they solve a concrete bottleneck or reliability problem.

        **Interview Follow-Up Questions**

        - **Mutable?** Audit logs should be append-only.
- **Sensitive data?** Redact secrets/PII where not needed.

# 21. Resume-Based Project Grilling

## Project Claim Legend

- **Actually implemented:** only say this if your code/resume proves it.
- **Recommended production improvement:** say "In a production version, I would..."
- **Theoretical knowledge:** say "I understand the approach, though I have not implemented it fully."

## ResolveAI

        **30-Second Explanation**

        ResolveAI is best explained as a backend project where you should connect user actions to API design, database modeling, authentication, authorization, failure handling, and production improvements.

        **90-Second Explanation**

        Start with the user problem, then explain the main backend flow: request enters Express, middleware authenticates and validates, the controller calls a service, the service applies business rules, the repository persists data, and background jobs or external providers handle slow or third-party work. Finish by naming one trade-off and one production improvement.

        **Architecture**

        ```text
        Client -> Express API -> Middleware -> Controller -> Service -> Repository -> Database
                                                 -> Background jobs / external providers where needed
        ```

        **What Was Actually Implemented Or Is Safe To Claim From The Known Background**

        - JWT authentication and RBAC are part of the known project background.
- Asynchronous processing with Inngest is part of the known project background.
- A student query-resolution domain is part of the known background.
- MongoDB/MySQL exposure and logging/monitoring are part of the overall resume context.

        **Production Improvements To Phrase Carefully**

        - In a production version, I would add refresh-token rotation and reuse detection if not already present.
- In a production version, I would add transactional outbox for query-created events if event publication must never be lost.
- In a production version, I would add unique notification dedupe keys.
- In a production version, I would add full integration tests for ownership and moderator assignment.
- In a production version, I would add dashboards for failed AI jobs and queue latency.

        **Data Flow**

        Explain the endpoint, request validation, service rule, database write/read, asynchronous side effect if any, and final response.

        **API Design**

        Prefer resource-oriented endpoints, consistent status codes, `data` and `error` envelopes, pagination for list endpoints, and idempotency for retryable writes.

        **Database Choice**

        Say why MongoDB or MySQL fit that project, and mention when the other would be better.

        **Authentication And Authorization**

        Use JWT/session language accurately. Distinguish RBAC from ownership checks.

        **Security**

        Mention validation, injection prevention, least privilege, secret handling, and no sensitive logs.

        **Failure Handling**

        Discuss retries, idempotency, webhooks, dead-letter handling, and reconciliation depending on project.

        **Testing**

        Mention unit tests for services, integration tests for APIs, auth/authorization tests, validation tests, and failure tests.

        **Scalability**

        Discuss stateless APIs, indexes, caching, queues, and database scaling only after correctness.

        **Likely Interviewer Questions**

        ### Q1. Why did you use JWT?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q2. Where was JWT stored?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q3. How did RBAC work?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q4. How did you prevent students from seeing another student's query?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q5. How did Inngest fit into the workflow?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q6. What happens if AI classification fails?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q7. How would you prevent duplicate notifications?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q8. What status codes did query APIs return?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q9. How would you paginate assigned queries?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q10. What indexes would you add?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q11. How would you test moderator authorization?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q12. How would you handle query reassignment?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q13. What is the difference between validation and business rules in this project?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q14. How would you log query processing?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q15. How would you monitor failed jobs?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q16. How would you design replies?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q17. Would you embed replies or reference them?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q18. How would you secure file attachments if added?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q19. How would you scale to many students?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q20. How would you handle duplicate query submissions?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q21. What would you improve in production?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q22. What would you put in OpenAPI docs?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q23. How would you handle admin analytics?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q24. How would you avoid leaking private query data?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q25. How would you deploy it with Docker?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

## Vrikshami / Razorpay Plant-Adoption Project

        **30-Second Explanation**

        Vrikshami / Razorpay Plant-Adoption Project is best explained as a backend project where you should connect user actions to API design, database modeling, authentication, authorization, failure handling, and production improvements.

        **90-Second Explanation**

        Start with the user problem, then explain the main backend flow: request enters Express, middleware authenticates and validates, the controller calls a service, the service applies business rules, the repository persists data, and background jobs or external providers handle slow or third-party work. Finish by naming one trade-off and one production improvement.

        **Architecture**

        ```text
        Client -> Express API -> Middleware -> Controller -> Service -> Repository -> Database
                                                 -> Background jobs / external providers where needed
        ```

        **What Was Actually Implemented Or Is Safe To Claim From The Known Background**

        - Razorpay integration is part of the known project background.
- MongoDB and Express-style backend work are part of the known project background.
- Certificate generation after payment is mentioned in the prior prep context.

        **Production Improvements To Phrase Carefully**

        - In a production version, I would verify both checkout and webhook signatures server-side if not already implemented.
- In a production version, I would add idempotency keys and unique provider payment constraints.
- In a production version, I would add reconciliation job for provider/local mismatches.
- In a production version, I would generate certificates asynchronously after confirmed capture.
- In a production version, I would add payment audit events and dashboards.

        **Data Flow**

        Explain the endpoint, request validation, service rule, database write/read, asynchronous side effect if any, and final response.

        **API Design**

        Prefer resource-oriented endpoints, consistent status codes, `data` and `error` envelopes, pagination for list endpoints, and idempotency for retryable writes.

        **Database Choice**

        Say why MongoDB or MySQL fit that project, and mention when the other would be better.

        **Authentication And Authorization**

        Use JWT/session language accurately. Distinguish RBAC from ownership checks.

        **Security**

        Mention validation, injection prevention, least privilege, secret handling, and no sensitive logs.

        **Failure Handling**

        Discuss retries, idempotency, webhooks, dead-letter handling, and reconciliation depending on project.

        **Testing**

        Mention unit tests for services, integration tests for APIs, auth/authorization tests, validation tests, and failure tests.

        **Scalability**

        Discuss stateless APIs, indexes, caching, queues, and database scaling only after correctness.

        **Likely Interviewer Questions**

        ### Q1. How does Razorpay order creation work?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q2. Why create payment order on backend?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q3. How do you verify checkout signature?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q4. How do you verify webhook signature?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q5. What if payment succeeds but DB update fails?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q6. What if DB update succeeds but response fails?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q7. How do you prevent duplicate payment attempts?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q8. How do you handle duplicate webhooks?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q9. What states can a payment have?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q10. Why store amount in paise?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q11. How do refunds work?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q12. What audit records would you store?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q13. How would you test Razorpay without real payments?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q14. How would you protect Razorpay secrets?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q15. How would you handle certificate generation?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q16. Should certificate generation be synchronous?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q17. How would you reconcile payments daily?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q18. What unique constraints are needed?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q19. How would you handle amount mismatch?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q20. How would you monitor payment failures?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q21. What status codes should payment APIs return?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q22. How would you design payment OpenAPI docs?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q23. How would you handle partial refund?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q24. What would you improve in production?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q25. How would you explain the full payment flow in 90 seconds?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

## AI/Text-to-SQL System

        **30-Second Explanation**

        AI/Text-to-SQL System is best explained as a backend project where you should connect user actions to API design, database modeling, authentication, authorization, failure handling, and production improvements.

        **90-Second Explanation**

        Start with the user problem, then explain the main backend flow: request enters Express, middleware authenticates and validates, the controller calls a service, the service applies business rules, the repository persists data, and background jobs or external providers handle slow or third-party work. Finish by naming one trade-off and one production improvement.

        **Architecture**

        ```text
        Client -> Express API -> Middleware -> Controller -> Service -> Repository -> Database
                                                 -> Background jobs / external providers where needed
        ```

        **What Was Actually Implemented Or Is Safe To Claim From The Known Background**

        - Text-to-SQL, SQL validation, MySQL, and FAISS are part of the known project background.
- Agent routing/self-correction were mentioned in the prior prep context.

        **Production Improvements To Phrase Carefully**

        - In a production version, I would parse generated SQL into an AST and enforce a deterministic allowlist.
- In a production version, I would use a read-only database user with query timeout and row limit.
- In a production version, I would add tenant filters deterministically outside the model.
- In a production version, I would audit every generated query and validation decision.
- In a production version, I would run the system against a replica/warehouse rather than production OLTP where appropriate.

        **Data Flow**

        Explain the endpoint, request validation, service rule, database write/read, asynchronous side effect if any, and final response.

        **API Design**

        Prefer resource-oriented endpoints, consistent status codes, `data` and `error` envelopes, pagination for list endpoints, and idempotency for retryable writes.

        **Database Choice**

        Say why MongoDB or MySQL fit that project, and mention when the other would be better.

        **Authentication And Authorization**

        Use JWT/session language accurately. Distinguish RBAC from ownership checks.

        **Security**

        Mention validation, injection prevention, least privilege, secret handling, and no sensitive logs.

        **Failure Handling**

        Discuss retries, idempotency, webhooks, dead-letter handling, and reconciliation depending on project.

        **Testing**

        Mention unit tests for services, integration tests for APIs, auth/authorization tests, validation tests, and failure tests.

        **Scalability**

        Discuss stateless APIs, indexes, caching, queues, and database scaling only after correctness.

        **Likely Interviewer Questions**

        ### Q1. How does the system convert text to SQL?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q2. Why is AI-generated SQL dangerous?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q3. How did the router select an agent?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q4. How would you prevent unsafe SQL?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q5. Why use read-only DB user?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q6. How do you validate generated SQL?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q7. How do you enforce tenant filters?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q8. How do you block multiple statements?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q9. How do you limit expensive queries?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q10. What if SQL fails?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q11. How does self-correction terminate?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q12. Why use FAISS/vector search?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q13. What schema context is given to the model?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q14. How do you prevent prompt injection?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q15. How do you audit generated queries?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q16. How do you handle hallucinated columns?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q17. How would you test SQL policy?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q18. How do you protect sensitive tables?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q19. How do you return results safely?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q20. How would you cache schema metadata?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q21. What would you improve in production?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q22. How do you compare MySQL vs MongoDB for this project?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q23. What indexes matter for generated queries?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q24. How do you monitor query cost?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q25. How would you explain this to a non-AI backend interviewer?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

## Internship / General Backend Experience

        **30-Second Explanation**

        Internship / General Backend Experience is best explained as a backend project where you should connect user actions to API design, database modeling, authentication, authorization, failure handling, and production improvements.

        **90-Second Explanation**

        Start with the user problem, then explain the main backend flow: request enters Express, middleware authenticates and validates, the controller calls a service, the service applies business rules, the repository persists data, and background jobs or external providers handle slow or third-party work. Finish by naming one trade-off and one production improvement.

        **Architecture**

        ```text
        Client -> Express API -> Middleware -> Controller -> Service -> Repository -> Database
                                                 -> Background jobs / external providers where needed
        ```

        **What Was Actually Implemented Or Is Safe To Claim From The Known Background**

        - The exact implementation details are not provided in the current file.
- Use this section as a careful answer template rather than a list of claims.

        **Production Improvements To Phrase Carefully**

        - In a production version, I would prepare a truthful list of endpoints, modules, bugs, tests, and deployment work you personally touched.
- In a production version, I would bring one concrete debugging story.
- In a production version, I would bring one concrete trade-off story.
- In a production version, I would avoid claiming ownership of team-level work you only observed.

        **Data Flow**

        Explain the endpoint, request validation, service rule, database write/read, asynchronous side effect if any, and final response.

        **API Design**

        Prefer resource-oriented endpoints, consistent status codes, `data` and `error` envelopes, pagination for list endpoints, and idempotency for retryable writes.

        **Database Choice**

        Say why MongoDB or MySQL fit that project, and mention when the other would be better.

        **Authentication And Authorization**

        Use JWT/session language accurately. Distinguish RBAC from ownership checks.

        **Security**

        Mention validation, injection prevention, least privilege, secret handling, and no sensitive logs.

        **Failure Handling**

        Discuss retries, idempotency, webhooks, dead-letter handling, and reconciliation depending on project.

        **Testing**

        Mention unit tests for services, integration tests for APIs, auth/authorization tests, validation tests, and failure tests.

        **Scalability**

        Discuss stateless APIs, indexes, caching, queues, and database scaling only after correctness.

        **Likely Interviewer Questions**

        ### Q1. What backend work did you do in the internship?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q2. How did you collaborate with frontend or other teams?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q3. What API design decisions did you make?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q4. What testing did you add?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q5. What production issue did you learn from?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q6. How did you handle logs?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q7. How did you handle Docker or deployment?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q8. How did you review code?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q9. What would you improve if you had more time?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q10. How did you learn a new codebase?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q11. How did you debug errors?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q12. How did you protect secrets?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q13. How did you validate inputs?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q14. What database did you use?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q15. What was the hardest part?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q16. How did you prioritize tasks?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q17. How did you handle feedback?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q18. How did you measure success?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q19. What did you personally implement?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q20. What did the team implement?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q21. What would you not claim?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q22. How did you document APIs?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q23. How did you handle branches/PRs?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q24. How would you scale that work?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

### Q25. What did you learn about production backends?

**Strong Answer**

Use the actual implementation if you can prove it from your code or resume. If not, say: "In the version I built, I handled the core flow. In a production version, I would add stronger validation, idempotency, observability, and failure recovery." Then answer with the recommended design.

**Follow-Up Direction**

Expect the interviewer to ask about data model, authorization, failure cases, tests, and scaling for this answer.

# 22. Hands-On Practice Lab

## Exercise 1: Design An Endpoint

**Question**

Design an endpoint for moderators to list their open high-priority queries.

**Solution**

```http
GET /api/v1/queries?assignedTo=me&status=open&priority=high&page=1&limit=20&sort=-createdAt
Authorization: Bearer <moderator-token>
```

Service must ignore arbitrary `assignedTo` for moderators and derive it from `req.user.id`.

## Exercise 2: Find The Security Problem

```javascript
router.patch("/users/:id", authenticate, async (req, res) => {
  const user = await User.findByIdAndUpdate(req.params.id, req.body, { new: true });
  res.json({ data: user });
});
```

**Solution**

Problems: no ownership/role authorization, mass assignment, no validation, returns raw user, possible passwordHash/internal fields. Fix with validation, explicit fields, service authorization, and DTO.

## Exercise 3: Predict Node Output

```javascript
console.log("1");
Promise.resolve().then(() => console.log("2"));
process.nextTick(() => console.log("3"));
console.log("4");
```

**Solution**

```text
1
4
3
2
```

## Exercise 4: Fix Broken Middleware

```javascript
function auth(req, res, next) {
  if (!req.get("Authorization")) {
    res.status(401).json({ error: "Missing token" });
  }
  next();
}
```

**Solution**

```javascript
function auth(req, res, next) {
  if (!req.get("Authorization")) {
    return res.status(401).json({
      error: { code: "AUTHENTICATION_REQUIRED", message: "Missing token" }
    });
  }
  next();
}
```

## Exercise 5: Improve A SQL Query

Query:

```sql
SELECT * FROM queries
WHERE assigned_to = 42 AND status = 'open'
ORDER BY created_at DESC
LIMIT 20;
```

**Solution**

Avoid `SELECT *` and add index:

```sql
CREATE INDEX idx_queries_assigned_status_created
ON queries (assigned_to, status, created_at);

SELECT id, title, status, priority, created_at
FROM queries
WHERE assigned_to = ?
  AND status = ?
ORDER BY created_at DESC
LIMIT ?;
```

## Exercise 6: Design MongoDB Schema

**Question**

Design query replies for potentially thousands of replies.

**Solution**

Use separate collection:

```javascript
{
  _id,
  queryId,
  authorId,
  message,
  createdAt
}
```

Index `{ queryId: 1, createdAt: 1 }`.

## Exercise 7: Write An API Test

**Question**

Test that a student cannot resolve a query.

**Solution**

```javascript
await request(app)
  .patch("/api/v1/queries/query_501")
  .set("Authorization", `Bearer ${studentToken}`)
  .send({ status: "resolved" })
  .expect(403);
```

## Exercise 8: Handle Duplicate Payment

**Solution**

Use `Idempotency-Key`, request hash, `in_progress/completed` states, unique constraint, and same response replay.

## Exercise 9: Handle Duplicate Webhook

**Solution**

Verify signature, store provider event ID in unique column, return 200 for duplicate, process only first event.

## Exercise 10: Debug Hanging Express Request

**Solution**

Check middleware branches for no `return`, no `next`, no response, unresolved promise, blocked event loop, or external call without timeout.

## Exercise 11: Debug Event-Loop Blocking

**Solution**

Capture CPU profile/event-loop delay. Look for synchronous loops, huge JSON operations, unsafe regex, sync filesystem, or CPU-heavy crypto. Move work to worker/job and limit input.

## Exercise 12: Design Retry-Safe Background Job

**Solution**

Make side effects idempotent with unique dedupe keys, store job state, retry transient errors with backoff/jitter, do not retry validation errors, move exhausted jobs to DLQ.

# 23. Capstone Production-Style Backend

## Project: QueryCourse Payments Platform

A capstone combining query-resolution, course enrollment, Razorpay-style payments, background jobs, JWT/RBAC, Redis, tests, Docker, and OpenAPI.

## Folder Structure

```text
src/
  app.js
  server.js
  config/env.js
  config/db.js
  config/redis.js
  middleware/
    requestId.js
    logger.js
    authenticate.js
    authorize.js
    validate.js
    rateLimit.js
    errorHandler.js
    notFound.js
  modules/
    auth/
    users/
    queries/
    courses/
    orders/
    payments/
    webhooks/
    jobs/
  jobs/inngest.js
  utils/
    AppError.js
    asyncHandler.js
    crypto.js
  openapi/openapi.yaml
tests/
  unit/
  integration/
Dockerfile
docker-compose.yml
.dockerignore
```

## Database Schema: MySQL Option

```sql
CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  role VARCHAR(30) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE refresh_tokens (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT NOT NULL,
  token_hash CHAR(64) NOT NULL UNIQUE,
  family_id CHAR(36) NOT NULL,
  revoked_at TIMESTAMP NULL,
  expires_at TIMESTAMP NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE queries (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(150) NOT NULL,
  description TEXT NOT NULL,
  status VARCHAR(30) NOT NULL,
  priority VARCHAR(20) NOT NULL,
  created_by BIGINT NOT NULL,
  assigned_to BIGINT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (created_by) REFERENCES users(id),
  FOREIGN KEY (assigned_to) REFERENCES users(id),
  INDEX idx_queries_created (created_by, created_at),
  INDEX idx_queries_assigned_status (assigned_to, status, created_at)
);

CREATE TABLE orders (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT NOT NULL,
  amount_paise INT NOT NULL,
  currency CHAR(3) NOT NULL,
  status VARCHAR(30) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE payment_attempts (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  order_id BIGINT NOT NULL,
  idempotency_key VARCHAR(255) NOT NULL UNIQUE,
  request_hash CHAR(64) NOT NULL,
  razorpay_order_id VARCHAR(255) NULL UNIQUE,
  status VARCHAR(30) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (order_id) REFERENCES orders(id)
);

CREATE TABLE payment_events (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  provider VARCHAR(50) NOT NULL,
  provider_event_id VARCHAR(255) NOT NULL,
  provider_payment_id VARCHAR(255) NULL,
  event_type VARCHAR(100) NOT NULL,
  processed_at TIMESTAMP NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  UNIQUE KEY uniq_provider_event (provider, provider_event_id)
);
```

## API Contracts

```http
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
GET /api/v1/queries
POST /api/v1/queries
GET /api/v1/queries/:queryId
POST /api/v1/queries/:queryId/replies
PATCH /api/v1/queries/:queryId
POST /api/v1/orders/:orderId/payment-attempts
POST /api/v1/payments/verify
POST /api/v1/webhooks/razorpay
GET /api/v1/jobs/:jobId
```

## Main Flows

- Register/login issues JWT access token and refresh cookie.
- Query creation validates body, checks role, stores query, emits Inngest event.
- Query listing scopes by role: student own, moderator assigned, admin all.
- Payment attempt uses idempotency key, local DB row, Razorpay order, and response replay.
- Webhook endpoint uses raw body, HMAC verification, event unique key, and background processing.
- Jobs update query classification, send notifications once, and generate certificates after captured payment.

## Security Design

- Zod validation on every request.
- JWT auth plus RBAC plus ownership checks.
- Password hashes only, no plaintext.
- Refresh tokens hashed at rest.
- Parameterized SQL.
- Razorpay signatures verified server-side.
- Rate limits on auth, payment, AI endpoints.
- Request body limits.
- Structured logs with redaction.

## Test Strategy

- Unit tests for auth service, query service, payment state machine.
- Integration tests for auth routes, query routes, payment routes.
- Webhook tests with valid/invalid signature and duplicate event.
- Concurrency tests for idempotency.
- Database transaction failure tests.
- Inngest/job handler tests with fake services.
- Dockerized MySQL/Redis for integration tests.

## Docker Setup

Use the Dockerfile and Compose approach from the production section. `.dockerignore` must include `.env`, `node_modules`, `.git`, `coverage`, and logs.

## OpenAPI Sketch

```yaml
openapi: 3.1.0
info:
  title: QueryCourse API
  version: 1.0.0
paths:
  /api/v1/queries:
    post:
      security:
        - bearerAuth: []
      responses:
        "201":
          description: Query created
        "401":
          description: Authentication required
        "403":
          description: Forbidden
        "422":
          description: Validation failed
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

## Capstone Interview Questions

- Why did you choose layered architecture?
- Where is validation done?
- Why does auth middleware not make ownership decisions alone?
- How does payment idempotency work?
- What happens if webhook arrives twice?
- What happens if Inngest job fails?
- How do you test refresh-token rotation?
- What indexes support query listing?
- How do you prevent SQL injection?
- How would you deploy and monitor this backend?

# 24. Practical Checklists

## API Design Checklist

- Resources are nouns.
- Methods match semantics.
- Status codes are accurate.
- Request/response schemas are documented.
- Errors use `{ error: { code, message, details, requestId } }`.
- Pagination exists for lists.
- Sort/filter fields are whitelisted.
- Auth and ownership rules are clear.
- Breaking changes use versioning/deprecation.
- OpenAPI is updated.

## Express Checklist

- `app.js` separate from `server.js`.
- Middleware order is correct.
- Routes are thin.
- Controllers handle HTTP.
- Services handle business rules.
- Repositories handle DB.
- DTOs shape responses.
- Errors centralized.
- Request IDs in logs/responses.
- Health/readiness/liveness endpoints exist.

## Security Audit Checklist

- Backend validation for every endpoint.
- Parameterized SQL.
- Mongo operator injection blocked.
- Passwords hashed.
- JWT expiry and verification.
- Refresh token storage/revocation.
- RBAC and ownership checks.
- CORS configured narrowly.
- CSRF considered for cookies.
- Rate limits.
- Request size limits.
- Webhook signatures verified.
- Secrets not committed/logged.
- Dependency scan.
- Secure headers.

## Payment Checklist

- Server calculates amount.
- Amount stored in smallest unit.
- Local payment attempt stored.
- Provider order created server-side.
- Checkout signature verified.
- Webhook raw-body signature verified.
- Idempotency key used.
- Unique provider payment/event IDs.
- State machine handles duplicates/out-of-order.
- Audit events stored.
- Reconciliation job exists.

## Testing Checklist

- Unit tests for services.
- Integration tests for routes.
- Auth/authorization tests.
- Validation tests.
- Not-found/conflict tests.
- Database failure tests.
- External provider failure tests.
- Concurrency/idempotency tests.
- Duplicate webhook tests.
- Load/security smoke tests where relevant.

# 25. Mock Interview Bank: 200+ Questions

_Total questions: 249. Each question includes basic answer, strong answer, follow-ups, common wrong answer, and example._

## HTTP and Networking

### [HTTP and Networking] What happens when you call an HTTPS API?

**Basic Answer**

DNS, TCP, TLS, HTTP request, server processing, response.

**Strong Answer**

The browser resolves DNS, opens TCP, negotiates TLS, sends HTTP, the request passes through proxies/load balancers, Express handles middleware/controllers, and a response returns. Mention timeouts, keep-alive, and status codes.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only saying 'frontend calls backend'.

**Practical Example**

Trace `GET /api/v1/users/42` from React to Express and back.

### [HTTP and Networking] What is DNS?

**Basic Answer**

DNS maps domains to IP addresses.

**Strong Answer**

DNS lets clients resolve names like `api.example.com` to addresses, with caching at browser, OS, resolver, and authoritative levels.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Calling DNS a database inside the backend.

**Practical Example**

A failed DNS lookup means the request never reaches Express.

### [HTTP and Networking] What is the TCP handshake?

**Basic Answer**

SYN, SYN-ACK, ACK.

**Strong Answer**

TCP establishes a reliable ordered connection before HTTP/1.1 or HTTP/2 data is exchanged, unless using QUIC/HTTP/3 over UDP.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying TCP encrypts data.

**Practical Example**

Slow handshakes increase latency without keep-alive.

### [HTTP and Networking] Why HTTPS?

**Basic Answer**

Encryption and integrity.

**Strong Answer**

HTTPS uses TLS to encrypt traffic, authenticate the server, and protect token-bearing headers from network attackers.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying HTTPS replaces authentication.

**Practical Example**

JWT over HTTP can be stolen on the network.

### [HTTP and Networking] HTTP/1.1 vs HTTP/2 vs HTTP/3?

**Basic Answer**

Newer versions improve transport behavior.

**Strong Answer**

HTTP/1.1 uses textual messages and limited multiplexing; HTTP/2 multiplexes streams over one TCP connection; HTTP/3 uses QUIC over UDP to reduce head-of-line blocking at transport level.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying REST only works on HTTP/1.1.

**Practical Example**

API semantics stay similar across versions.

### [HTTP and Networking] What is keep-alive?

**Basic Answer**

Reuse TCP connections.

**Strong Answer**

Persistent connections avoid repeating TCP/TLS setup for each request, improving latency and reducing server/client overhead.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying it keeps user sessions alive.

**Practical Example**

Node's HTTP agent can reuse connections to upstream APIs.

### [HTTP and Networking] What is content negotiation?

**Basic Answer**

Client and server agree response format.

**Strong Answer**

Clients send `Accept`, `Accept-Language`, or `Accept-Encoding`; servers choose representation and should use `Vary` when caches must distinguish versions.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Confusing it with authentication.

**Practical Example**

Return JSON for `Accept: application/json`.

### [HTTP and Networking] What is an ETag?

**Basic Answer**

A resource version validator.

**Strong Answer**

ETag lets a client make conditional requests with `If-None-Match`; unchanged resources can return `304 Not Modified` without a body.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying ETag is an auth token.

**Practical Example**

Use ETag for cache validation or optimistic concurrency.

### [HTTP and Networking] What is Cache-Control?

**Basic Answer**

HTTP caching policy.

**Strong Answer**

It tells browsers/proxies whether and how long a response can be reused, with directives like `no-store`, `private`, `max-age`, and `s-maxage`.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Caching every authenticated response publicly.

**Practical Example**

Use `Cache-Control: no-store` for login responses.

### [HTTP and Networking] What are proxy headers?

**Basic Answer**

Headers added by proxies.

**Strong Answer**

`X-Forwarded-For`, `X-Forwarded-Proto`, and `Forwarded` help apps know original client IP/protocol, but must only be trusted from known proxies.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Trusting user-supplied `X-Forwarded-For` from the internet.

**Practical Example**

Express `trust proxy` must be configured carefully.

## API Fundamentals

### [API Fundamentals] What is an API contract?

**Basic Answer**

The documented request/response agreement.

**Strong Answer**

It defines method, path, headers, auth, body, query params, status codes, response shape, and errors; OpenAPI can formalize it.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only listing endpoints without schemas.

**Practical Example**

A frontend can build safely from a stable OpenAPI spec.

### [API Fundamentals] Route params vs query params vs body?

**Basic Answer**

Resource identity vs modifiers vs payload.

**Strong Answer**

Path params identify a resource, query params filter/sort/page collections, body carries creation/update data.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using body for GET filters everywhere.

**Practical Example**

`GET /tickets?status=open&page=1`.

### [API Fundamentals] What is idempotency?

**Basic Answer**

Same request repeated gives same final effect.

**Strong Answer**

Idempotency matters for retries after timeouts, especially payments. Use idempotency keys and unique constraints for non-idempotent operations.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying idempotent means same status code every time.

**Practical Example**

Second duplicate payment request returns first payment result.

### [API Fundamentals] Why correct status codes?

**Basic Answer**

They communicate outcome.

**Strong Answer**

Clients, monitors, retry logic, and tests rely on accurate HTTP status codes; do not return 200 for errors.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Always returning 200 with `success:false`.

**Practical Example**

Use 409 for duplicate email.

### [API Fundamentals] What is backward compatibility?

**Basic Answer**

Old clients keep working.

**Strong Answer**

Additive changes are safer; breaking changes require versioning, deprecation, or migration plans.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Changing response fields without warning.

**Practical Example**

Add optional `middleName`, do not rename `name` silently.

## REST Design

### [REST Design] How do you design REST endpoints?

**Basic Answer**

Use resources and HTTP methods.

**Strong Answer**

Use plural nouns, stable resource identifiers, query params for collection controls, consistent response envelopes, and correct status codes.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using `/getAllUsers` and POST for every action.

**Practical Example**

`GET /api/v1/users/:userId`.

### [REST Design] When are verbs acceptable?

**Basic Answer**

For commands not cleanly modeled as CRUD.

**Strong Answer**

Login, cancel, generate, or business actions can be commands, though modeling them as resources is often cleaner.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Putting verbs in every URL.

**Practical Example**

`POST /orders/:id/cancellations`.

### [REST Design] Offset vs cursor pagination?

**Basic Answer**

Offset uses page/limit; cursor uses position.

**Strong Answer**

Offset is simple and supports page jumps; cursor is better for large, changing datasets with stable ordering.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Returning all rows.

**Practical Example**

`GET /feed?limit=20&after=cursor`.

### [REST Design] How do you version APIs?

**Basic Answer**

Usually URL or headers.

**Strong Answer**

Use `/api/v1` for clarity; introduce new versions for breaking changes and deprecate old versions with communication.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Versioning every tiny change.

**Practical Example**

Changing `name` to `firstName/lastName` is breaking.

### [REST Design] How do you design errors?

**Basic Answer**

Consistent error object.

**Strong Answer**

Use machine-readable `code`, safe human `message`, optional `details`, and `requestId`.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Leaking stack traces.

**Practical Example**

`VALIDATION_FAILED` with field details.

## Node.js Internals

### [Node.js Internals] What does V8 do?

**Basic Answer**

Runs JavaScript.

**Strong Answer**

V8 parses, compiles, optimizes, and garbage-collects JavaScript; Node adds runtime APIs and libuv handles async I/O/event loop.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying V8 handles database queries.

**Practical Example**

CPU-heavy JS runs on the main V8 thread.

### [Node.js Internals] What is libuv?

**Basic Answer**

Node's async I/O library.

**Strong Answer**

libuv provides event loop, networking abstractions, file-system thread pool work, timers, and cross-platform async primitives.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying libuv is a database driver.

**Practical Example**

Thread pool saturation affects fs/crypto/dns work.

### [Node.js Internals] What is the event loop?

**Basic Answer**

Scheduler for async callbacks.

**Strong Answer**

It processes phases such as timers, poll, check, and close callbacks, while microtasks run between turns and `nextTick` has special priority.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying it creates one thread per request.

**Practical Example**

A long loop blocks all callbacks.

### [Node.js Internals] `setImmediate` vs `setTimeout(0)`?

**Basic Answer**

Different scheduling mechanisms.

**Strong Answer**

`setImmediate` runs in check phase; `setTimeout` runs after timer threshold. Top-level order can vary; inside I/O, immediate usually runs first.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying timeout always runs first.

**Practical Example**

Inside `fs.readFile` callback, immediate before timeout.

### [Node.js Internals] What is backpressure?

**Basic Answer**

Producer is faster than consumer.

**Strong Answer**

Streams signal backpressure when writes return false; producers should pause until `drain` to avoid memory growth.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Ignoring return value of `.write()`.

**Practical Example**

Large file upload should stream, not buffer entire file.

### [Node.js Internals] When use worker threads?

**Basic Answer**

CPU-bound work.

**Strong Answer**

Worker threads move CPU-heavy JavaScript off the main event loop; I/O-bound API work usually does not need them.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using workers for every DB query.

**Practical Example**

Hashing large files can use workers.

### [Node.js Internals] What is AsyncLocalStorage?

**Basic Answer**

Async request context storage.

**Strong Answer**

It preserves context like request ID across async calls without passing it through every function.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using global mutable variables for request state.

**Practical Example**

Logger can read request ID from ALS.

### [Node.js Internals] How do memory leaks happen?

**Basic Answer**

References prevent GC.

**Strong Answer**

Leaks occur via unbounded caches, event listeners, timers, global arrays, closures, or retaining request objects.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying GC prevents all leaks.

**Practical Example**

A Map keyed by user ID without TTL grows forever.

### [Node.js Internals] Unhandled rejection vs uncaught exception?

**Basic Answer**

Async promise error vs sync exception not caught.

**Strong Answer**

Both indicate bugs or unhandled failures; log, alert, and often restart gracefully because process state may be unsafe.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Ignoring them in production.

**Practical Example**

Add process handlers but fix root cause.

### [Node.js Internals] CPU-bound vs I/O-bound?

**Basic Answer**

Computation vs waiting.

**Strong Answer**

I/O-bound work waits on DB/network/file; CPU-bound work uses the JS thread. Node excels at I/O-bound workloads.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying Node is always fastest.

**Practical Example**

Report generation may need worker/queue.

## Express

### [Express] What is Express middleware?

**Basic Answer**

Functions in request flow.

**Strong Answer**

Middleware runs in registered order and can parse, authenticate, validate, rate limit, log, respond, or pass errors.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Forgetting `next()` and hanging requests.

**Practical Example**

Auth middleware sets `req.user`.

### [Express] Controller vs service?

**Basic Answer**

HTTP vs business logic.

**Strong Answer**

Controllers translate HTTP to service calls; services enforce business rules and coordinate repositories/providers.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Putting SQL and payment code directly in route.

**Practical Example**

Service checks order can be paid.

### [Express] Express 4 vs 5 async errors?

**Basic Answer**

Promise handling differs.

**Strong Answer**

Express 5 forwards rejected route promises to error middleware; Express 4 commonly needs an async wrapper.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Assuming all Express versions behave the same.

**Practical Example**

`asyncHandler(fn).catch(next)` in Express 4.

### [Express] Why response shaping?

**Basic Answer**

Avoid leaking internals.

**Strong Answer**

DTO/serializer logic returns only public fields and decouples API contracts from DB schema.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Returning Mongoose document with passwordHash.

**Practical Example**

Map user to `{id,name,email,role}`.

### [Express] How to debug hanging request?

**Basic Answer**

Find missing response/next.

**Strong Answer**

Check middleware branches that neither return response nor call `next`, long awaits, blocked event loop, and unresolved promises.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Increasing timeout blindly.

**Practical Example**

Missing return in validation middleware.

## JavaScript

### [JavaScript] `var` vs `let` vs `const`?

**Basic Answer**

Scope and reassignment.

**Strong Answer**

`var` is function-scoped; `let` and `const` are block-scoped; `const` prevents reassignment, not object mutation.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying const makes object immutable.

**Practical Example**

`const user = {}; user.name = 'A'` works.

### [JavaScript] What is closure?

**Basic Answer**

Function remembers lexical scope.

**Strong Answer**

Closures let inner functions access outer variables after outer execution, useful for factories/middleware but can retain memory.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Calling every callback a closure without explanation.

**Practical Example**

`authorize('admin')` returns middleware using `allowedRoles`.

### [JavaScript] Promise states?

**Basic Answer**

Pending, fulfilled, rejected.

**Strong Answer**

Promises represent async completion; errors propagate through rejection and `await` throws rejected reasons.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using try/catch around un-awaited promise.

**Practical Example**

`await service.create()` in controller.

### [JavaScript] `Promise.all` vs `allSettled`?

**Basic Answer**

Fail-fast vs collect all.

**Strong Answer**

`Promise.all` rejects on first rejection; `allSettled` waits for every promise and reports each result.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using `all` for optional independent notifications.

**Practical Example**

Fetch user/orders in parallel with `all`.

### [JavaScript] What is optional chaining?

**Basic Answer**

Safe nested access.

**Strong Answer**

It returns undefined instead of throwing when a left operand is nullish.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using it to hide required missing data.

**Practical Example**

`req.user?.id`.

## TypeScript

### [TypeScript] Why TypeScript in backend?

**Basic Answer**

Static types reduce mistakes.

**Strong Answer**

It catches contract mismatches earlier, documents DTOs, improves refactoring, and works with runtime validation like Zod.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying TS replaces validation.

**Practical Example**

Request body still needs Zod at runtime.

### [TypeScript] `unknown` vs `any`?

**Basic Answer**

`unknown` forces narrowing.

**Strong Answer**

`any` disables checking; `unknown` requires you to prove type before using it.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using any everywhere.

**Practical Example**

Catch errors as unknown and narrow.

### [TypeScript] Interface vs type alias?

**Basic Answer**

Both describe shapes.

**Strong Answer**

Interfaces are extendable object contracts; type aliases handle unions/intersections/tuples well. Use consistently.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying one is always better.

**Practical Example**

`type ApiResult<T> = {data:T}`.

### [TypeScript] What are generics?

**Basic Answer**

Reusable types with parameters.

**Strong Answer**

Generics preserve type information across functions/classes like repositories or API responses.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using `any` instead.

**Practical Example**

`ApiResponse<UserDto>`.

### [TypeScript] Zod vs TypeScript?

**Basic Answer**

Runtime vs compile-time.

**Strong Answer**

TypeScript disappears at runtime; Zod validates real inputs and can infer TypeScript types.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Trusting TS for request body.

**Practical Example**

`createUserSchema.parse(req.body)`.

## Authentication

### [Authentication] Authentication vs authorization?

**Basic Answer**

Identity vs permission.

**Strong Answer**

Authentication proves who the user is; authorization decides what that user may do.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using 403 for missing token.

**Practical Example**

401 missing token, 403 wrong role.

### [Authentication] How does JWT work?

**Basic Answer**

Signed claims token.

**Strong Answer**

JWT has header, payload, signature. Server verifies signature, expiry, issuer/audience if used, then trusts claims.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying JWT is encrypted by default.

**Practical Example**

Payload contains `sub` and `role`, not password.

### [Authentication] HS256 vs RS256?

**Basic Answer**

Shared secret vs key pair.

**Strong Answer**

HS256 uses one secret to sign/verify; RS256 signs with private key and verifies with public key, useful across services.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Publishing HS256 secret.

**Practical Example**

Auth service signs, API verifies public key.

### [Authentication] Refresh token rotation?

**Basic Answer**

Issue new refresh token each use.

**Strong Answer**

Rotation detects reuse: if an old token is used again, revoke the token family because it may be stolen.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Long-lived non-rotating tokens forever.

**Practical Example**

Store hashed refresh token with family ID.

### [Authentication] Session vs JWT?

**Basic Answer**

Server state vs stateless token.

**Strong Answer**

Sessions are easy to revoke but need shared store; JWTs scale statelessly but revocation is harder.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying JWT is always better.

**Practical Example**

Admin dashboards often work well with sessions.

### [Authentication] What is OIDC?

**Basic Answer**

Identity layer over OAuth 2.0.

**Strong Answer**

OIDC adds ID tokens and standardized user identity claims for login flows.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Calling OAuth login authentication by itself.

**Practical Example**

Login with Google returns identity through OIDC.

### [Authentication] How prevent brute force?

**Basic Answer**

Rate limit and monitor.

**Strong Answer**

Use per-IP/per-account limits, progressive delay, CAPTCHA after risk, generic errors, and alerting; avoid easy account enumeration.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Locking accounts forever.

**Practical Example**

5 failed login attempts -> cooldown.

## Security

### [Security] What is BOLA?

**Basic Answer**

Broken object-level authorization.

**Strong Answer**

A user accesses another user's resource by changing IDs; fix with ownership checks after loading resource.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only checking JWT exists.

**Practical Example**

`GET /orders/otherUserOrder`.

### [Security] What is mass assignment?

**Basic Answer**

Client sets fields it should not.

**Strong Answer**

Avoid spreading request body into models; whitelist fields.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Allowing `role:'admin'` from signup body.

**Practical Example**

Create user with explicit role student.

### [Security] What is SSRF?

**Basic Answer**

Server requests attacker-controlled URL.

**Strong Answer**

Attackers make backend reach internal services; validate destinations, block private IP ranges, and use allowlists.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only checking URL starts with http.

**Practical Example**

Image fetch endpoint accesses metadata IP.

### [Security] What is prototype pollution?

**Basic Answer**

Changing object prototypes.

**Strong Answer**

Unsafe merge of user objects can set `__proto__`; use safe parsers, validation, and avoid deep merging untrusted input.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Assuming JSON is harmless.

**Practical Example**

`{"__proto__":{"admin":true}}`.

### [Security] What is ReDoS?

**Basic Answer**

Regex denial of service.

**Strong Answer**

Catastrophic regex backtracking on crafted input blocks the event loop; use safe regexes and length limits.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Running complex regex on unlimited input.

**Practical Example**

Email validation regex freezes API.

## SQL

### [SQL] Primary vs foreign key?

**Basic Answer**

Identity vs relationship.

**Strong Answer**

Primary key uniquely identifies row; foreign key references another table and enforces referential integrity.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using email as all foreign keys.

**Practical Example**

`orders.user_id -> users.id`.

### [SQL] What is normalization?

**Basic Answer**

Reduce duplication.

**Strong Answer**

1NF/2NF/3NF reduce repeating groups, partial dependencies, and transitive dependencies; denormalize intentionally for reads.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Normalizing until queries are unusable.

**Practical Example**

Separate users and orders.

### [SQL] What is a B-tree index?

**Basic Answer**

Sorted tree for lookup/range.

**Strong Answer**

B-tree indexes keep keys ordered, enabling equality, range, and order-by operations efficiently.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying index is a hash map always.

**Practical Example**

`WHERE user_id=? ORDER BY created_at`.

### [SQL] What is leftmost-prefix rule?

**Basic Answer**

Composite index usage starts from left.

**Strong Answer**

Index `(a,b,c)` helps filters on `a`, `a,b`, `a,b,c`, but not usually `b` alone.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Creating `(status,created_at)` for queries by created_at only.

**Practical Example**

Choose order by equality filters then sort/range.

### [SQL] What is `EXPLAIN`?

**Basic Answer**

Query plan inspection.

**Strong Answer**

It shows how MySQL plans access tables, indexes, join order, and estimated rows so you can optimize queries.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Guessing indexes blindly.

**Practical Example**

Check if query uses full table scan.

### [SQL] What is N+1?

**Basic Answer**

One query plus many child queries.

**Strong Answer**

Fetching list then querying each child individually causes many round trips; fix joins, batching, or eager loading.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Adding more DB connections.

**Practical Example**

Load 20 users then 20 order queries.

### [SQL] Optimistic vs pessimistic locking?

**Basic Answer**

Version check vs lock first.

**Strong Answer**

Optimistic uses version/update condition and retries on conflict; pessimistic locks rows with `FOR UPDATE`.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Locking every read.

**Practical Example**

Seat booking may use row locks.

## MongoDB

### [MongoDB] Embedding vs referencing?

**Basic Answer**

Store together vs link IDs.

**Strong Answer**

Embed bounded data read with parent; reference large/shared/independently queried data.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Embedding unbounded arrays.

**Practical Example**

Replies may be separate if many.

### [MongoDB] What is ObjectId?

**Basic Answer**

MongoDB identifier type.

**Strong Answer**

It is a 12-byte BSON value with timestamp/random/counter parts; useful as ID but not an authorization mechanism.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Treating ObjectId as secret.

**Practical Example**

Validate format but still check ownership.

### [MongoDB] What are partial indexes?

**Basic Answer**

Index subset of documents.

**Strong Answer**

Partial indexes index documents matching a filter, reducing size and enforcing conditional uniqueness.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using many full indexes blindly.

**Practical Example**

Unique active subscription per user.

### [MongoDB] What are TTL indexes?

**Basic Answer**

Automatic expiry.

**Strong Answer**

TTL indexes delete documents after a time, useful for sessions, OTPs, or temporary records.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using TTL for financial audit records.

**Practical Example**

Expire password reset tokens.

### [MongoDB] What is `lean()`?

**Basic Answer**

Plain objects from Mongoose.

**Strong Answer**

`lean()` skips Mongoose document hydration, often faster for read-only responses but no document methods/getters.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using lean then expecting `.save()`.

**Practical Example**

List endpoints can use lean DTOs.

## Testing

### [Testing] Unit vs integration test?

**Basic Answer**

Isolation vs combined behavior.

**Strong Answer**

Unit tests isolate services; integration tests exercise route/middleware/database boundaries.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only manual Postman testing.

**Practical Example**

Supertest route test for POST /users.

### [Testing] Mock vs stub vs fake?

**Basic Answer**

Different test doubles.

**Strong Answer**

Mocks assert interactions, stubs return controlled answers, fakes are working simplified implementations.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Calling every double a mock.

**Practical Example**

Fake repository backed by Map.

### [Testing] How test idempotency?

**Basic Answer**

Repeat same request/key.

**Strong Answer**

Send same idempotency key twice, including concurrent requests, and assert one side effect and same response.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only testing one request.

**Practical Example**

Two payment calls create one payment row.

### [Testing] How test JWT expiry?

**Basic Answer**

Control time or use expired token.

**Strong Answer**

Generate token with past exp or mock timers; assert 401 and refresh path behavior.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Waiting real 15 minutes.

**Practical Example**

Use test clock.

### [Testing] Why deterministic tests?

**Basic Answer**

Same result every run.

**Strong Answer**

Control time, randomness, external APIs, and database state; flaky tests lose trust.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Depending on real email provider.

**Practical Example**

Mock Razorpay.

## Background Jobs

### [Background Jobs] Why background jobs?

**Basic Answer**

Slow/retryable work outside request.

**Strong Answer**

They keep APIs responsive and let side effects retry independently.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Doing email and AI inside request always.

**Practical Example**

Return 202 then process.

### [Background Jobs] At-most-once vs at-least-once?

**Basic Answer**

Delivery guarantees.

**Strong Answer**

At-most-once may lose work; at-least-once may duplicate work, so consumers must be idempotent.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying exactly once is easy.

**Practical Example**

Webhook consumer handles duplicate events.

### [Background Jobs] What is transactional outbox?

**Basic Answer**

DB write and event publication safety.

**Strong Answer**

Write business row and outbox event in same transaction; worker publishes event later, avoiding DB/event race.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Write DB then publish event with no recovery.

**Practical Example**

Order paid and payment event recorded atomically.

### [Background Jobs] What is DLQ?

**Basic Answer**

Dead-letter queue.

**Strong Answer**

Failed jobs after retries move to DLQ for inspection/replay.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Retrying forever.

**Practical Example**

AI classification failed after 3 attempts.

### [Background Jobs] Why jitter in retries?

**Basic Answer**

Avoid retry storms.

**Strong Answer**

Randomizing backoff prevents many jobs from retrying at same time after outage.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Fixed immediate retries.

**Practical Example**

Payment provider 503 recovery.

## Payments

### [Payments] Why create order server-side?

**Basic Answer**

Protect amount and identity.

**Strong Answer**

Backend calculates amount, creates Razorpay order, and returns order ID; frontend cannot decide trusted amount.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Trusting client amount.

**Practical Example**

Amount from DB cart/order.

### [Payments] How verify Razorpay signature?

**Basic Answer**

HMAC SHA256.

**Strong Answer**

Use server secret and expected payload, raw body for webhooks, timing-safe compare, and reject invalid signatures.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Verifying on frontend.

**Practical Example**

`order_id|payment_id` checkout signature.

### [Payments] How handle duplicate webhooks?

**Basic Answer**

Idempotent event processing.

**Strong Answer**

Use provider event ID/payment ID unique constraint; duplicate returns 200 without repeat side effect.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Returning 500 for duplicates.

**Practical Example**

`x-razorpay-event-id`.

### [Payments] Payment success but DB failure?

**Basic Answer**

Reconcile.

**Strong Answer**

Webhook retries or scheduled reconciliation fetches provider status and repairs local state. Local attempts and unique IDs make it safe.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Telling user to pay again.

**Practical Example**

Payment captured but order pending.

### [Payments] Why store audit records?

**Basic Answer**

Money needs traceability.

**Strong Answer**

Audit rows record provider events, state transitions, request IDs, and actor for debugging/refunds/disputes.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Overwriting status without history.

**Practical Example**

payment_events table.

## Docker

### [Docker] What is Docker image vs container?

**Basic Answer**

Template vs running instance.

**Strong Answer**

Image is immutable filesystem/config; container is a running process from image with isolated environment.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying Docker is a VM.

**Practical Example**

Build API image, run container.

### [Docker] Why `.dockerignore`?

**Basic Answer**

Reduce build context.

**Strong Answer**

It prevents copying node_modules, secrets, git data, and coverage into image build context.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Copying `.env` into image.

**Practical Example**

Add `.env` to dockerignore.

### [Docker] Why multi-stage build?

**Basic Answer**

Smaller safer images.

**Strong Answer**

Build dependencies/assets in one stage and copy only runtime needs to final image.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Shipping dev dependencies.

**Practical Example**

npm ci then runtime stage.

### [Docker] How pass secrets?

**Basic Answer**

Environment/secret manager.

**Strong Answer**

Do not bake secrets into image; inject through deployment secret mechanism.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

ARG JWT_SECRET in Dockerfile.

**Practical Example**

Kubernetes secret or platform env var.

### [Docker] Container health check?

**Basic Answer**

Detect unhealthy instance.

**Strong Answer**

Health endpoints let orchestrators remove/restart unhealthy containers.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only checking process exists.

**Practical Example**

`/ready` verifies DB.

## Production Engineering

### [Production Engineering] Readiness vs liveness?

**Basic Answer**

Ready for traffic vs alive.

**Strong Answer**

Readiness checks dependencies needed to serve traffic; liveness checks process health and should not fail for temporary dependency outage unless process is wedged.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

One `/health` for everything.

**Practical Example**

DB down -> not ready, still live.

### [Production Engineering] What are latency percentiles?

**Basic Answer**

Distribution, not average.

**Strong Answer**

p95/p99 show tail latency experienced by slow users; averages hide outliers.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only reporting average latency.

**Practical Example**

p99 spikes during DB lock.

### [Production Engineering] Timeouts vs retries?

**Basic Answer**

Bound waiting and retry transient failures.

**Strong Answer**

Set timeouts for external calls, retry only idempotent/transient operations with backoff/jitter.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Infinite retry on POST payment.

**Practical Example**

Retry 503 GET provider call.

### [Production Engineering] What is circuit breaker?

**Basic Answer**

Fail fast for unhealthy dependency.

**Strong Answer**

It opens after repeated failures, avoids overwhelming dependency, and later probes recovery.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Calling failed service every request forever.

**Practical Example**

AI provider outage.

### [Production Engineering] Blue-green vs canary?

**Basic Answer**

Deployment strategies.

**Strong Answer**

Blue-green switches traffic between full environments; canary gradually shifts a small percentage to new version.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Deploying to all users instantly with no rollback.

**Practical Example**

Canary payment code carefully.

## System Design

### [System Design] How design URL shortener?

**Basic Answer**

Map code to URL.

**Strong Answer**

APIs create code, redirect, and track analytics; use unique code, cache redirects, validate URLs, and rate limit creation.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Storing only in memory.

**Practical Example**

GET /:code redirects.

### [System Design] How design rate limiter?

**Basic Answer**

Track usage in window/bucket.

**Strong Answer**

Use Redis counters/token bucket/sliding window with atomic operations and return 429 with Retry-After.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Local memory only behind many servers.

**Practical Example**

Login limiter by IP+email.

### [System Design] How design notification service?

**Basic Answer**

Durable notification records and workers.

**Strong Answer**

API records request, workers send via provider with retries/idempotency, status tracking, and DLQ.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Sending synchronously only.

**Practical Example**

Email duplicate key.

### [System Design] How design audit log?

**Basic Answer**

Append-only event record.

**Strong Answer**

Store actor, action, resource, timestamp, metadata, request ID; avoid mutation and protect sensitive data.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Updating one last_action field.

**Practical Example**

Admin deleted user audit.

### [System Design] How design Text-to-SQL?

**Basic Answer**

LLM plus deterministic safety.

**Strong Answer**

Retrieve schema, generate SQL, parse/validate, enforce read-only/tenant/LIMIT, execute with read-only user and timeout.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Executing model output directly.

**Practical Example**

SELECT only allowed.

## Project Questions

### [Project Questions] Explain ResolveAI.

**Basic Answer**

Student query-resolution backend.

**Strong Answer**

Explain query creation, JWT/RBAC, assignment, async AI/notifications with Inngest, MongoDB/MySQL choice, and production improvements.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only saying it uses AI.

**Practical Example**

Design `/queries/:id/replies`.

### [Project Questions] Explain Vrikshami payments.

**Basic Answer**

Razorpay integration.

**Strong Answer**

Explain server-side order creation, checkout, signature verification, webhook confirmation, idempotency, and certificate generation.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Trusting frontend payment success.

**Practical Example**

Handle captured webhook duplicate.

### [Project Questions] Explain Text-to-SQL safety.

**Basic Answer**

Validated read-only AI SQL.

**Strong Answer**

Never execute model output blindly; parse, validate, restrict, enforce tenant/LIMIT, use read-only DB.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Prompt-only safety.

**Practical Example**

Block DROP TABLE.

### [Project Questions] What would you improve?

**Basic Answer**

Production hardening.

**Strong Answer**

Mention tests, idempotency, observability, security review, migrations, and load testing without pretending implemented.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Inventing features.

**Practical Example**

Say 'In production, I would...'

### [Project Questions] What was your hardest bug?

**Basic Answer**

Use STAR format.

**Strong Answer**

Pick a real bug if available. If not, discuss a plausible category carefully as a learning, not fabricated implementation.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Making up exact incidents.

**Practical Example**

Hanging request due missing next.

## Debugging Scenarios

### [Debugging Scenarios] API hangs forever.

**Basic Answer**

Missing response/next or unresolved await.

**Strong Answer**

Trace middleware order, add request ID logs, inspect branches, check event-loop blocking and dependency timeouts.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Restarting server only.

**Practical Example**

Validation error path forgot return.

### [Debugging Scenarios] CPU at 100%.

**Basic Answer**

Event loop blocked.

**Strong Answer**

Capture CPU profile, inspect synchronous loops/regex/JSON work, move CPU work to worker/job, add input limits.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Add DB index blindly.

**Practical Example**

ReDoS regex.

### [Debugging Scenarios] Database slow query.

**Basic Answer**

Bad plan/index/N+1.

**Strong Answer**

Use EXPLAIN, check indexes/cardinality, query shape, rows scanned, and round trips.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Increase server RAM first.

**Practical Example**

Missing `(user_id, created_at)` index.

### [Debugging Scenarios] Duplicate payments.

**Basic Answer**

Missing idempotency/constraints.

**Strong Answer**

Inspect callbacks/webhooks/retries, add unique payment IDs, idempotency records, and reconciliation.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Tell user not to click twice.

**Practical Example**

Concurrent duplicate request.

### [Debugging Scenarios] Logs show no request context.

**Basic Answer**

No correlation ID propagation.

**Strong Answer**

Add request ID middleware/AsyncLocalStorage and include IDs in service/job logs.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Search by timestamp only.

**Practical Example**

req_123 in error response.

### [HTTP and Networking] HEAD method

**Basic Answer**

GET headers without body

**Strong Answer**

Use HEAD to check metadata like existence or caching validators without transferring body.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying HEAD creates resources.

**Practical Example**

Health of static asset cache.

### [HTTP and Networking] OPTIONS method

**Basic Answer**

Supported communication options

**Strong Answer**

Browsers use OPTIONS for CORS preflight; APIs may return allowed methods/headers.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Confusing with GET.

**Practical Example**

Preflight for custom Authorization header.

### [HTTP and Networking] MIME type

**Basic Answer**

Content type label

**Strong Answer**

MIME types tell clients how to interpret bytes; APIs usually use `application/json`.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Ignoring Content-Type.

**Practical Example**

Reject XML if endpoint expects JSON.

### [HTTP and Networking] Compression

**Basic Answer**

Reduce payload size

**Strong Answer**

Gzip/br can reduce response size but costs CPU and should avoid already-compressed files.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Compressing tiny responses blindly.

**Practical Example**

Large JSON list compressed.

### [HTTP and Networking] Timeouts

**Basic Answer**

Limit waiting

**Strong Answer**

Clients and servers need connect/read/request timeouts to prevent resource exhaustion.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

No timeout anywhere.

**Practical Example**

External AI call times out.

### [API Fundamentals] Bulk API design

**Basic Answer**

Handle many operations

**Strong Answer**

Decide atomic all-or-nothing vs partial success; large jobs may return 202.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Always one huge synchronous request.

**Practical Example**

Bulk deactivate users.

### [API Fundamentals] Partial response

**Basic Answer**

Return selected fields

**Strong Answer**

Can reduce payload but must whitelist fields to avoid sensitive leakage.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Allow any DB field.

**Practical Example**

`fields=id,name`.

### [API Fundamentals] Deprecation

**Basic Answer**

Sunsetting old API

**Strong Answer**

Warn clients, document dates, keep old version during migration, monitor usage.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Deleting endpoint instantly.

**Practical Example**

`Deprecation` and docs.

### [REST Design] Nested route depth

**Basic Answer**

Relationship path

**Strong Answer**

Use shallow nesting; too many levels make authorization and routing awkward.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Five-level URLs.

**Practical Example**

`/tasks/:id/comments`.

### [REST Design] PUT create behavior

**Basic Answer**

Upsert possibility

**Strong Answer**

Some APIs allow PUT to known URI to create/replace; document clearly.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Assuming universal behavior.

**Practical Example**

`PUT /settings/user_42`.

### [Node.js Internals] Buffer

**Basic Answer**

Binary data container

**Strong Answer**

Buffers handle raw bytes for files, streams, crypto, and network data.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using string for arbitrary binary.

**Practical Example**

Webhook raw body buffer.

### [Node.js Internals] EventEmitter

**Basic Answer**

Pub/sub object

**Strong Answer**

It emits named events to listeners; handle `error` event to avoid crashes.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using it for distributed messaging.

**Practical Example**

Stream emits data/end/error.

### [Node.js Internals] AbortController

**Basic Answer**

Cancellation signal

**Strong Answer**

Use it to cancel fetches, timers, and long operations when request closes/timeouts.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Ignoring abandoned clients.

**Practical Example**

Cancel AI fetch after 10s.

### [Node.js Internals] Thread pool size

**Basic Answer**

libuv worker count

**Strong Answer**

fs, crypto, zlib, dns lookup can use libuv pool; saturation delays those tasks.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Increasing it always solves CPU.

**Practical Example**

Many bcrypt hashes delay fs.

### [Express] Route matching order

**Basic Answer**

First matching route wins

**Strong Answer**

Register specific routes before generic params and 404 after routers.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Putting `/:id` before `/search`.

**Practical Example**

`/users/search` treated as id.

### [Express] File upload security

**Basic Answer**

Validate file inputs

**Strong Answer**

Limit size/type, randomize names, store outside app, scan if needed.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Trust original filename.

**Practical Example**

Path traversal in filename.

### [Express] Rate limiting middleware

**Basic Answer**

Request quota

**Strong Answer**

Use Redis-backed limiter in multi-instance deployments.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

In-memory limiter in production cluster.

**Practical Example**

Login endpoint limiter.

### [JavaScript] Hoisting

**Basic Answer**

Declarations processed before execution

**Strong Answer**

`var` hoists initialized as undefined; `let/const` are in temporal dead zone.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using variable before declaration safely.

**Practical Example**

ReferenceError for `let`.

### [JavaScript] Event delegation

**Basic Answer**

Frontend concept

**Strong Answer**

Mention only if relevant; backend JS interviews focus more on async, promises, modules.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Over-answering backend with DOM.

**Practical Example**

N/A backend.

### [JavaScript] Modules CJS vs ESM

**Basic Answer**

Module systems

**Strong Answer**

CommonJS uses `require`; ESM uses `import/export`; Node supports both with config differences.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Mixing without config.

**Practical Example**

`type: module`.

### [TypeScript] Union type

**Basic Answer**

Value can be one of variants

**Strong Answer**

Use narrowing before accessing variant-specific fields.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Accessing property without narrowing.

**Practical Example**

`status: 'paid'|'failed'`.

### [TypeScript] Intersection type

**Basic Answer**

Combine types

**Strong Answer**

Useful for composing DTOs or middleware-augmented request types.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Confusing with union.

**Practical Example**

`User & Timestamps`.

### [TypeScript] Utility types

**Basic Answer**

Transform types

**Strong Answer**

`Pick`, `Omit`, `Partial`, `Required`, `Record` reduce duplication.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Rewriting every DTO manually.

**Practical Example**

`Omit<User,'passwordHash'>`.

### [Authentication] API keys

**Basic Answer**

Shared credential for clients/services

**Strong Answer**

Use for service/project access, hash at rest, scope permissions, rotate keys.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using one global key forever.

**Practical Example**

Webhook provider key.

### [Authentication] Service-to-service auth

**Basic Answer**

Auth between backends

**Strong Answer**

Use mTLS, signed JWTs, or scoped API keys; never trust internal network alone.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

No auth inside VPC.

**Practical Example**

Worker calls API.

### [Authentication] Account lockout tradeoff

**Basic Answer**

Security vs DoS

**Strong Answer**

Hard lockouts stop brute force but enable account denial; prefer throttling and risk-based controls.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Lock forever after 3 tries.

**Practical Example**

Temporary cooldown.

### [Security] CORS

**Basic Answer**

Browser response access control

**Strong Answer**

Configure allowed origins; still require auth and authorization.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

CORS as security for Postman/curl.

**Practical Example**

Credentials need explicit origin.

### [Security] CSRF

**Basic Answer**

Cross-site cookie abuse

**Strong Answer**

Use SameSite, CSRF token, origin checks for cookie-auth state changes.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Ignoring because API uses cookies.

**Practical Example**

POST transfer from malicious site.

### [Security] XSS in API

**Basic Answer**

Stored data rendered by frontend

**Strong Answer**

APIs should not return unsafe HTML unless sanitized; set HttpOnly cookies.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Assuming JSON prevents XSS.

**Practical Example**

Stored comment with script.

### [Security] Path traversal

**Basic Answer**

Access unintended files

**Strong Answer**

Normalize paths and restrict to allowed directory; do not trust file names.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Concatenate user path.

**Practical Example**

`../../.env`.

### [SQL] CTE

**Basic Answer**

Named temporary query

**Strong Answer**

CTEs make complex queries readable and can support recursive queries.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Always faster.

**Practical Example**

`WITH paid_orders AS (...)`.

### [SQL] Window functions

**Basic Answer**

Compute over rows

**Strong Answer**

Use for rank, running totals, pagination metadata without collapsing rows.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Confusing with GROUP BY.

**Practical Example**

`ROW_NUMBER() OVER`.

### [SQL] Views

**Basic Answer**

Saved query abstraction

**Strong Answer**

Can simplify access/reporting but may hide complexity and performance issues.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Treating as table always.

**Practical Example**

View for public user fields.

### [SQL] Stored procedures

**Basic Answer**

DB-side logic

**Strong Answer**

Useful in some orgs for encapsulated operations but can complicate versioning/testing.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Putting all business logic in DB by default.

**Practical Example**

Interview-level compare.

### [SQL] Replication lag

**Basic Answer**

Replica behind primary

**Strong Answer**

Reads from replica may be stale after writes; route read-your-write cases to primary.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Assuming replicas instant.

**Practical Example**

Order paid but replica shows pending.

### [MongoDB] Covered query

**Basic Answer**

Index contains all needed fields

**Strong Answer**

MongoDB can answer from index without fetching documents when projection matches index.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Projecting unindexed fields.

**Practical Example**

Find email by status with projection.

### [MongoDB] Change streams

**Basic Answer**

Watch DB changes

**Strong Answer**

Useful for reactive workflows but require replica set/sharded cluster and careful resume handling.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Replacing all queues.

**Practical Example**

Notify search indexer.

### [MongoDB] Sharding

**Basic Answer**

Horizontal partitioning

**Strong Answer**

Choose shard key carefully for distribution and query routing.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Shard too early.

**Practical Example**

Tenant-based shard key tradeoff.

### [Testing] Contract tests

**Basic Answer**

Consumer/provider agreement

**Strong Answer**

Verify API shape across teams/services; helpful with OpenAPI/Pact style workflows.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only unit tests.

**Practical Example**

Frontend expects `data.items`.

### [Testing] Smoke tests

**Basic Answer**

Basic deployment checks

**Strong Answer**

Run quick critical-path tests after deployment.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Full load test on every deploy.

**Practical Example**

Health + login + main read.

### [Testing] Load testing

**Basic Answer**

Performance under traffic

**Strong Answer**

Use k6/Artillery to test latency, throughput, and bottlenecks with realistic scenarios.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only hitting `/health`.

**Practical Example**

Payment flow load at low volume.

### [Background Jobs] Cron vs queue

**Basic Answer**

Scheduled vs event work

**Strong Answer**

Cron runs by schedule; queue jobs respond to events/tasks and retry.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using cron for every async event.

**Practical Example**

Daily reconciliation cron.

### [Background Jobs] Ordering

**Basic Answer**

Event sequence

**Strong Answer**

Queues may not guarantee global ordering; design with state machines and idempotent transitions.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Assuming webhooks arrive ordered.

**Practical Example**

Captured before authorized webhook.

### [Payments] Refunds

**Basic Answer**

Reverse payment partially/fully

**Strong Answer**

Track refund state, provider refund ID, amount, reason, and idempotency.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Just setting order unpaid.

**Practical Example**

Partial refund audit.

### [Payments] Authorized vs captured

**Basic Answer**

Reserved vs collected

**Strong Answer**

Authorized means payment approved; captured means funds captured. Auto-capture may combine flow.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Treating authorized as final always.

**Practical Example**

Ship only after captured.

### [Docker] Layer caching

**Basic Answer**

Reuse build layers

**Strong Answer**

Copy package files before source so dependency install cache is reused.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Copy all files before npm ci.

**Practical Example**

Faster rebuilds.

### [Docker] Non-root user

**Basic Answer**

Least privilege

**Strong Answer**

Run app as non-root inside container to reduce impact of compromise.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Root everywhere.

**Practical Example**

`USER node`.

### [Production Engineering] Feature flags

**Basic Answer**

Runtime behavior switch

**Strong Answer**

Enable gradual rollout/rollback without redeploy but manage flag debt.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using flags as permanent config mess.

**Practical Example**

Enable new checkout for 5%.

### [Production Engineering] SLO vs SLA

**Basic Answer**

Objective vs agreement

**Strong Answer**

SLO is internal reliability target; SLA is contractual promise; error budget is allowed unreliability.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using terms interchangeably.

**Practical Example**

99.9% monthly API availability.

### [System Design] Webhook processor

**Basic Answer**

Receive and process provider events

**Strong Answer**

Verify raw signature, store event, enqueue processing, idempotently update domain state.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Doing heavy work before response.

**Practical Example**

Razorpay webhook.

### [System Design] Order API

**Basic Answer**

Cart/order/payment workflow

**Strong Answer**

Use order state machine, payment attempts, inventory checks, transactions, idempotency.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

One `status` update from client.

**Practical Example**

Create order then pay.

### [Project Questions] How would you test ResolveAI?

**Basic Answer**

API and job tests

**Strong Answer**

Test create query, auth/ownership, assignment, job retry, duplicate notifications, and AI failure.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only manual test.

**Practical Example**

Mock AI provider.

### [Project Questions] Why MongoDB or MySQL?

**Basic Answer**

Data access fit

**Strong Answer**

Explain based on relationships, transactions, schema flexibility, and query patterns.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Because it is easy.

**Practical Example**

Payments often benefit from SQL transactions.

### [Debugging Scenarios] Webhook signature fails

**Basic Answer**

Raw body mismatch or wrong secret

**Strong Answer**

Ensure raw body, correct webhook secret, old secret during rotation, exact header, timing-safe compare.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Parsing body before HMAC.

**Practical Example**

Express raw route.

### [Debugging Scenarios] Tests flaky

**Basic Answer**

Uncontrolled state

**Strong Answer**

Check time, randomness, DB cleanup, concurrency, external APIs, and async completion.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Rerun until pass.

**Practical Example**

Use fixtures and fake timers.

### [HTTP and Networking] What is TLS handshake?

**Basic Answer**

Secure connection setup

**Strong Answer**

The client and server negotiate protocol version, cipher suite, certificates, and keys before encrypted HTTP data is exchanged.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying TLS is the same as JWT.

**Practical Example**

HTTPS request before API token is sent.

### [HTTP and Networking] What is head-of-line blocking?

**Basic Answer**

One blocked item delays others

**Strong Answer**

In HTTP/1.1 connection limits and TCP ordering can delay independent requests; HTTP/2 multiplexes streams but still uses TCP; HTTP/3 reduces transport-level blocking with QUIC.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying HTTP/2 has no possible blocking at all.

**Practical Example**

Many assets/API calls over one connection.

### [HTTP and Networking] What is `Vary` header?

**Basic Answer**

Cache key modifier

**Strong Answer**

`Vary` tells caches which request headers affect the response representation, such as `Accept-Encoding` or `Accept-Language`.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Ignoring it for content negotiation.

**Practical Example**

English and Hindi responses cached separately.

### [HTTP and Networking] What is 304?

**Basic Answer**

Not Modified

**Strong Answer**

A conditional request can return 304 without a body when cached content is still valid.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Returning JSON body with 304.

**Practical Example**

ETag validation.

### [HTTP and Networking] What is 415?

**Basic Answer**

Unsupported media type

**Strong Answer**

Return 415 when the endpoint requires JSON but receives unsupported content type.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using 500 for bad Content-Type.

**Practical Example**

Client sends text/plain to JSON endpoint.

### [API Fundamentals] What is 422?

**Basic Answer**

Semantic validation failure

**Strong Answer**

Use 422 when syntax is valid but fields violate validation or business constraints, if that is your team's convention.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using 422 for auth failure.

**Practical Example**

Invalid email format.

### [API Fundamentals] What is 409?

**Basic Answer**

State conflict

**Strong Answer**

Use 409 when request conflicts with current resource state, such as duplicate email or already-paid order.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using 500 for duplicate key.

**Practical Example**

Unique key violation mapped to EMAIL_ALREADY_EXISTS.

### [API Fundamentals] What is 429?

**Basic Answer**

Rate limit exceeded

**Strong Answer**

Return 429 with optional Retry-After when a client exceeds quota.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Blocking silently.

**Practical Example**

Login brute force limiter.

### [API Fundamentals] What is request id?

**Basic Answer**

Correlation identifier

**Strong Answer**

A request ID connects client error responses to backend logs and traces.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using timestamp only.

**Practical Example**

`X-Request-ID` response header.

### [API Fundamentals] What is API deprecation?

**Basic Answer**

Planned retirement

**Strong Answer**

Deprecation gives clients time and documentation to migrate before removal.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Removing route without notice.

**Practical Example**

v1 endpoint migration to v2.

### [REST Design] How design search?

**Basic Answer**

Text query endpoint

**Strong Answer**

Use query params like `search=term`, validate length, index/search appropriately, and paginate.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

SQL LIKE on huge table with no index for everything.

**Practical Example**

Search tickets by title.

### [REST Design] How design sorting?

**Basic Answer**

Whitelisted sort fields

**Strong Answer**

Allow only documented fields and directions to prevent injection or expensive sorts.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Passing raw sort into SQL.

**Practical Example**

`sort=-createdAt`.

### [REST Design] How design field selection?

**Basic Answer**

Client asks subset

**Strong Answer**

Whitelist public fields and reject sensitive or unknown fields.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Allowing `passwordHash` field.

**Practical Example**

`fields=id,name,email`.

### [REST Design] How design bulk partial success?

**Basic Answer**

Report item-level results

**Strong Answer**

Define whether operation is atomic; for partial success return successful and failed item lists or create async job.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Pretending all succeeded.

**Practical Example**

Bulk deactivate users.

### [REST Design] How design long-running operation?

**Basic Answer**

202 plus status URL

**Strong Answer**

Return 202 with job ID/status URL and process asynchronously.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Keeping request open for minutes.

**Practical Example**

AI report generation.

### [Node.js Internals] What is garbage collection?

**Basic Answer**

Memory cleanup

**Strong Answer**

V8 reclaims unreachable objects; leaks happen when objects remain reachable through references.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying GC means no memory issues.

**Practical Example**

Global Map grows forever.

### [Node.js Internals] How detect event-loop lag?

**Basic Answer**

Measure delay

**Strong Answer**

Use perf hooks/APM to measure delay, then profile CPU-heavy synchronous work.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Guessing from CPU only.

**Practical Example**

p99 latency high during regex.

### [Node.js Internals] What uses libuv thread pool?

**Basic Answer**

Some async native work

**Strong Answer**

File system, crypto, zlib, and some DNS operations can use the libuv thread pool.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

All network I/O uses thread pool.

**Practical Example**

Many bcrypt calls delay fs.

### [Node.js Internals] What is child_process?

**Basic Answer**

Run separate process

**Strong Answer**

Use child processes for isolated commands or CPU work with separate memory/process boundary.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Running shell commands with user input unsafely.

**Practical Example**

PDF generation process.

### [Node.js Internals] How handle uncaughtException?

**Basic Answer**

Log and shutdown

**Strong Answer**

An uncaught exception may leave process state unsafe; log, stop accepting traffic, and restart via supervisor.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Continue as if nothing happened.

**Practical Example**

Production crash handling.

### [Express] What is DTO?

**Basic Answer**

Data transfer object

**Strong Answer**

DTOs shape API responses and decouple DB schema from public contracts.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Returning raw ORM model.

**Practical Example**

Public user DTO excludes passwordHash.

### [Express] What is dependency injection?

**Basic Answer**

Pass dependencies in

**Strong Answer**

DI makes services testable by passing repositories/providers instead of importing globals everywhere.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Hard-coding every dependency.

**Practical Example**

Fake repository in unit test.

### [Express] Why validate env?

**Basic Answer**

Fail fast on bad config

**Strong Answer**

Validate required environment variables at startup to avoid runtime surprises.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Undefined JWT secret accepted.

**Practical Example**

Zod env schema.

### [Express] What is readiness endpoint?

**Basic Answer**

Traffic readiness check

**Strong Answer**

It verifies required dependencies and returns 503 when not ready.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only returning ok always.

**Practical Example**

DB unavailable -> 503.

### [Express] How handle JSON parse errors?

**Basic Answer**

Error middleware

**Strong Answer**

Express JSON parser forwards malformed body errors; map them to safe 400 responses.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Returning stack trace.

**Practical Example**

Malformed JSON request.

### [JavaScript] What is `this`?

**Basic Answer**

Call-site context

**Strong Answer**

`this` depends on how a function is called; arrow functions capture lexical this.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Assuming class method keeps this when passed as callback.

**Practical Example**

Bind controller methods or use functions.

### [JavaScript] Map vs object?

**Basic Answer**

Keyed collections

**Strong Answer**

Map supports any key type and stable size/iteration; objects are plain records with prototype concerns.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using object for untrusted arbitrary keys.

**Practical Example**

Use Map for in-memory fake repo.

### [JavaScript] Set use case?

**Basic Answer**

Unique values

**Strong Answer**

Set stores unique values, useful for dedupe in memory but not as production distributed dedupe.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using Set for multi-instance idempotency.

**Practical Example**

Deduplicate IDs in request.

### [JavaScript] Destructuring risk?

**Basic Answer**

Access undefined

**Strong Answer**

Destructure after validation or provide defaults to avoid runtime errors.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Destructuring req.body without checking.

**Practical Example**

Zod validates body first.

### [JavaScript] JSON pitfalls?

**Basic Answer**

Serialization limits

**Strong Answer**

JSON loses Date type, cannot represent BigInt, and parsing huge payloads costs memory/CPU.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Stringifying BigInt directly.

**Practical Example**

Return ISO date strings.

### [TypeScript] What is discriminated union?

**Basic Answer**

Union with tag field

**Strong Answer**

Use a common literal field to narrow variants safely.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Manual unsafe casts.

**Practical Example**

`{type:'card'} | {type:'upi'}`.

### [TypeScript] What is `never`?

**Basic Answer**

Impossible type

**Strong Answer**

`never` helps exhaustive checks in switches.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Ignoring new status values.

**Practical Example**

Payment status exhaustive switch.

### [TypeScript] What is `Record`?

**Basic Answer**

Object key/value type

**Strong Answer**

`Record<K,V>` represents an object with keys K and values V.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using any object.

**Practical Example**

`Record<Role, Permission[]>`.

### [TypeScript] What is `Partial`?

**Basic Answer**

All fields optional

**Strong Answer**

Useful for patch inputs but still validate allowed fields at runtime.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using Partial to accept unsafe body.

**Practical Example**

PATCH user DTO.

### [TypeScript] How type errors?

**Basic Answer**

Custom classes and narrowing

**Strong Answer**

Create typed error classes but still narrow unknown caught errors.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Assuming catch is Error.

**Practical Example**

AppError extends Error.

### [Authentication] What is token audience?

**Basic Answer**

Intended recipient

**Strong Answer**

JWT `aud` helps ensure token issued for this API is not accepted by another.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Ignoring issuer/audience in multi-service system.

**Practical Example**

Auth token for API A rejected by API B.

### [Authentication] What is token issuer?

**Basic Answer**

Who issued token

**Strong Answer**

`iss` identifies token issuer and should be verified in federated/multi-service systems.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Accepting any signed-looking token.

**Practical Example**

OIDC provider issuer.

### [Authentication] What is device management?

**Basic Answer**

Per-device sessions

**Strong Answer**

Store refresh token/session records per device so users can revoke one device.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

One token for all devices.

**Practical Example**

Logout from one browser.

### [Authentication] Why hash refresh tokens?

**Basic Answer**

Limit DB leak impact

**Strong Answer**

If DB leaks, raw refresh tokens should not be immediately usable.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Storing refresh token plaintext.

**Practical Example**

SHA-256 token hash lookup.

### [Authentication] What is SameSite?

**Basic Answer**

Cookie cross-site policy

**Strong Answer**

`SameSite` controls when browsers send cookies on cross-site requests and helps reduce CSRF.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Thinking it encrypts cookie.

**Practical Example**

SameSite=Lax refresh cookie.

### [Security] How prevent NoSQL injection?

**Basic Answer**

Validate types/operators

**Strong Answer**

Reject objects where strings are expected and sanitize Mongo operators.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Passing req.body directly into findOne.

**Practical Example**

Email must be string.

### [Security] How prevent SQL injection in ORDER BY?

**Basic Answer**

Whitelist identifiers

**Strong Answer**

Placeholders protect values, not identifiers; map client sort fields to known column names.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using `ORDER BY ${req.query.sort}`.

**Practical Example**

sort=-createdAt -> `created_at DESC`.

### [Security] How handle dependency vulnerabilities?

**Basic Answer**

Audit and update

**Strong Answer**

Use npm audit/SCA tools, pin/lock dependencies, patch quickly, and remove unused packages.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Ignoring lockfile alerts.

**Practical Example**

CI dependency scan.

### [Security] How secure file paths?

**Basic Answer**

Normalize and constrain

**Strong Answer**

Resolve paths and ensure they stay within intended directory; randomize storage keys.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Trust uploaded filename.

**Practical Example**

Prevent `../` traversal.

### [Security] How avoid sensitive logs?

**Basic Answer**

Redaction

**Strong Answer**

Redact auth headers, tokens, passwords, OTPs, card data, and secrets before logging.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Logging req.body for login.

**Practical Example**

Logger redaction middleware.

### [SQL] What is covering index?

**Basic Answer**

Index satisfies query

**Strong Answer**

If index contains all filtered/projected fields, DB may avoid table lookup.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Adding huge covering indexes everywhere.

**Practical Example**

Index `(user_id,status,created_at,id)`.

### [SQL] What is index selectivity?

**Basic Answer**

How well index narrows rows

**Strong Answer**

High-cardinality/selective columns often make better leading filters than low-cardinality columns, balanced with query patterns.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Indexing boolean alone.

**Practical Example**

status low selectivity.

### [SQL] What is optimistic concurrency?

**Basic Answer**

Update with version check

**Strong Answer**

Use version column in WHERE and fail/retry if no rows updated.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Overwriting lost updates.

**Practical Example**

`WHERE id=? AND version=?`.

### [SQL] What is pessimistic locking?

**Basic Answer**

Lock before changing

**Strong Answer**

Use `SELECT ... FOR UPDATE` inside transaction when conflicts are likely and correctness is critical.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Holding lock while calling external API.

**Practical Example**

Inventory reservation.

### [SQL] How handle migrations?

**Basic Answer**

Versioned schema changes

**Strong Answer**

Use migration tool, backward-compatible deploy steps, backups, and rollback strategy.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Manual production edits.

**Practical Example**

Add nullable column before code uses it.

### [MongoDB] What is BSON?

**Basic Answer**

Binary JSON-like storage

**Strong Answer**

BSON supports additional types like ObjectId and Date and is MongoDB's storage/wire format.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Saying Mongo stores plain JSON exactly.

**Practical Example**

Date stored as BSON date.

### [MongoDB] What is `$set`?

**Basic Answer**

Atomic field update

**Strong Answer**

Use update operators like `$set` to change fields atomically without replacing whole document.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Read-modify-write without condition for concurrent updates.

**Practical Example**

Set query status.

### [MongoDB] What is optimistic concurrency in Mongoose?

**Basic Answer**

Version key check

**Strong Answer**

Mongoose can use versioning to detect conflicting document saves.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Ignoring lost updates.

**Practical Example**

`__v` version conflict.

### [MongoDB] How debug slow Mongo query?

**Basic Answer**

Explain and profiler

**Strong Answer**

Use explain plans, indexes, profiler/slow query logs, and check scanned vs returned docs.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Adding random indexes.

**Practical Example**

Collection scan on assignedTo.

### [MongoDB] When use transaction?

**Basic Answer**

Multi-document atomicity

**Strong Answer**

Use when multiple documents must commit together; avoid for simple single-document atomic updates.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Wrapping every update in transaction.

**Practical Example**

Order and payment records.

### [Testing] How test time?

**Basic Answer**

Fake timers/clock injection

**Strong Answer**

Inject clock or use fake timers to test expiry/retry logic deterministically.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Sleeping real time.

**Practical Example**

JWT expiry tests.

### [Testing] How test randomness?

**Basic Answer**

Inject ID generator

**Strong Answer**

Pass deterministic ID generator in tests to assert outputs.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Asserting random UUID exact value.

**Practical Example**

Fake id `query_1`.

### [Testing] How clean test data?

**Basic Answer**

Isolation strategy

**Strong Answer**

Use transactions, truncation, unique schemas, or containerized DB reset.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Tests depend on previous order.

**Practical Example**

BeforeEach cleanup.

### [Testing] What is meaningful assertion?

**Basic Answer**

Assert behavior not implementation only

**Strong Answer**

Check status, body, DB side effects, and external calls where relevant.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only expecting 200.

**Practical Example**

Assert passwordHash absent.

### [Testing] How test background job?

**Basic Answer**

Call handler with fake event/deps

**Strong Answer**

Unit test job logic with fake services and integration test enqueue path.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Waiting for real queue in unit tests.

**Practical Example**

Fake Inngest step.

### [Background Jobs] What is job timeout?

**Basic Answer**

Max execution time

**Strong Answer**

Jobs should have timeouts so stuck providers or bugs do not occupy workers forever.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

No timeout.

**Practical Example**

AI call max 30s.

### [Background Jobs] What is concurrency limit?

**Basic Answer**

Max parallel jobs

**Strong Answer**

Limits protect DB/providers from too many simultaneous jobs.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Unlimited workers.

**Practical Example**

Only 5 payment reconciliation jobs.

### [Background Jobs] What is scheduled job?

**Basic Answer**

Run at time/interval

**Strong Answer**

Use for reconciliation, cleanup, summaries; make it idempotent because schedules can overlap.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Non-idempotent cron.

**Practical Example**

Daily payment reconciliation.

### [Background Jobs] What is job cancellation?

**Basic Answer**

Stop unneeded work

**Strong Answer**

Support cancellation flags/signals for long jobs where user cancels or job becomes obsolete.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Ignoring cancelled reports.

**Practical Example**

Cancel report generation.

### [Background Jobs] What is eventual consistency?

**Basic Answer**

State converges later

**Strong Answer**

Async side effects may lag behind primary write; expose pending states and reconcile failures.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Promising instant email/search update.

**Practical Example**

Query created before AI classification.

### [Payments] What is reconciliation?

**Basic Answer**

Compare provider and local state

**Strong Answer**

Scheduled job fetches provider records and repairs local mismatch.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Manual checking only.

**Practical Example**

Captured provider payment with pending local order.

### [Payments] What is idempotency request hash?

**Basic Answer**

Detect key misuse

**Strong Answer**

Store hash of key request body so same key with different payload is rejected.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Reusing key for different amount.

**Practical Example**

409 idempotency conflict.

### [Payments] How handle amount validation?

**Basic Answer**

Compare trusted server amount

**Strong Answer**

Backend calculates expected amount from order and rejects callback/provider data mismatch.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using amount from client.

**Practical Example**

Order amount 100000 paise.

### [Payments] How process refunds?

**Basic Answer**

State and audit

**Strong Answer**

Create refund request with idempotency, call provider, process webhook, update refund/payment state.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Deleting payment row.

**Practical Example**

Partial refund record.

### [Payments] How monitor payments?

**Basic Answer**

Business metrics

**Strong Answer**

Track attempts, success/failure rate, webhook failures, reconciliation mismatches, provider latency.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Only HTTP 500 count.

**Practical Example**

Alert on webhook signature failures spike.

### [Docker] What is image layer?

**Basic Answer**

Build filesystem layer

**Strong Answer**

Each Dockerfile instruction can create a layer; ordering affects cache reuse and image size.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

One giant unstructured Dockerfile.

**Practical Example**

Copy package files before source.

### [Docker] What is container security?

**Basic Answer**

Reduce blast radius

**Strong Answer**

Use non-root, small images, updated dependencies, read-only FS where possible, no secrets in image.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Root with secrets baked in.

**Practical Example**

node alpine runtime.

### [Docker] Why log to stdout?

**Basic Answer**

Container logging model

**Strong Answer**

Platforms collect stdout/stderr; file logs inside containers are harder to aggregate.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Writing only local files.

**Practical Example**

JSON logs to stdout.

### [Docker] What is Compose for?

**Basic Answer**

Multi-container local environment

**Strong Answer**

Docker Compose runs API, DB, Redis, etc. locally for development/testing.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using Compose as full production orchestrator always.

**Practical Example**

API + MySQL + Redis.

### [Docker] How handle migrations in Docker deploy?

**Basic Answer**

Separate migration step

**Strong Answer**

Run migrations as controlled job before/with deploy using backward-compatible strategy.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Run migrations randomly in every app instance.

**Practical Example**

CI/CD migration job.

### [Production Engineering] What is cache stampede?

**Basic Answer**

Many misses hit DB

**Strong Answer**

When cache expires, many requests recompute simultaneously; use locks, early refresh, jitter, stale-while-revalidate.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Same TTL for all hot keys.

**Practical Example**

Popular course page expiry.

### [Production Engineering] What is distributed lock?

**Basic Answer**

Cross-instance mutual exclusion

**Strong Answer**

Use carefully with TTL/fencing for rare coordination; prefer DB constraints when possible.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Using locks instead of unique constraints for payments.

**Practical Example**

Single reconciliation job.

### [Production Engineering] What is bulkhead?

**Basic Answer**

Failure isolation

**Strong Answer**

Separate resource pools so one dependency/path does not exhaust all capacity.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

One shared pool for all calls.

**Practical Example**

AI provider pool separate from payment pool.

### [Production Engineering] What is graceful degradation?

**Basic Answer**

Reduced functionality during failure

**Strong Answer**

Serve core features while optional dependency is down.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Whole app down because email fails.

**Practical Example**

Create query but notification pending.

### [Production Engineering] What is canary rollback trigger?

**Basic Answer**

Metrics-based rollback

**Strong Answer**

Rollback if canary error rate/latency/business metrics exceed threshold.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Rollback based on vibes.

**Practical Example**

Payment success drops.

### [System Design] Design autocomplete

**Basic Answer**

Fast prefix/search suggestions

**Strong Answer**

Use normalized search index/trie/search engine, rate limit, cache popular prefixes, protect privacy.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

LIKE '%term%' on huge DB only.

**Practical Example**

Course search suggestions.

### [System Design] Design leaderboard

**Basic Answer**

Ranked data

**Strong Answer**

Use sorted sets/cache/read model, handle ties, update asynchronously if acceptable.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Sorting entire DB every request.

**Practical Example**

Top moderators by resolved queries.

### [System Design] Design report generation

**Basic Answer**

Async job

**Strong Answer**

Return 202, job status, background worker, object storage for result, retries and expiry.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Synchronous 5-minute request.

**Practical Example**

Monthly payment report.

### [System Design] Design session management

**Basic Answer**

Device/session records

**Strong Answer**

Refresh tokens/session rows per device with revoke endpoint and audit.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

One JWT forever.

**Practical Example**

Logout all devices.

### [System Design] Design webhook replay

**Basic Answer**

Safe manual retry

**Strong Answer**

Store raw event/audit, allow admin replay through idempotent processor, track attempts.

**Likely Follow-Ups**

- How would you implement it in Express?
- What can go wrong?
- How would you test it?

**Common Wrong Answer**

Manually edit DB.

**Practical Example**

Replay failed payment event.

# 26. Official References

These primary or official references were checked or used while updating this handbook:

- Node.js event loop guide: https://nodejs.org/learn/asynchronous-work/event-loop-timers-and-nexttick
- Node.js test runner documentation: https://nodejs.org/api/test.html
- Express error handling guide: https://expressjs.com/en/guide/error-handling/
- OWASP API Security Top 10 2023: https://owasp.org/API-Security/editions/2023/en/0x11-t10/
- RFC 7519 JSON Web Token: https://www.rfc-editor.org/rfc/rfc7519
- RFC 8725 JWT Best Current Practices: https://www.rfc-editor.org/rfc/rfc8725
- MDN HTTP caching: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Caching
- MDN ETag header: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/ETag
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- TypeScript Handbook: https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
- Razorpay Node.js integration: https://razorpay.com/docs/payments/server-integration/nodejs/integration-steps/
- Razorpay webhook validation: https://razorpay.com/docs/webhooks/validate-test/
- Inngest error handling and retries: https://www.inngest.com/docs/guides/error-handling
- Inngest idempotency guide: https://www.inngest.com/docs/guides/handling-idempotency
- MySQL InnoDB isolation levels: https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html
- MySQL EXPLAIN statement: https://dev.mysql.com/doc/refman/8.4/en/explain.html
- MongoDB indexes: https://www.mongodb.com/docs/manual/indexes/
- MongoDB compound indexes: https://www.mongodb.com/docs/manual/core/indexes/index-types/index-compound/
- Jest mock functions: https://jestjs.io/docs/mock-functions
- Supertest repository/documentation: https://github.com/forwardemail/supertest
- Docker build best practices: https://docs.docker.com/build/building/best-practices/

# 27. Final Interview Mantra

For every backend answer, ask yourself:

```text
What is the API contract?
Who is authenticated?
Who is authorized?
What input is validated?
What database constraint protects correctness?
What happens if the request is duplicated?
What happens if a dependency fails?
How is this tested?
How is this logged and monitored?
What did I actually implement, and what would I improve in production?
```

If you answer with that mindset, you will sound like someone who can build and operate a real backend, not just repeat definitions.

