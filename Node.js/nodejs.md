# ⚡ NODE.JS BACKEND INTERVIEW — MASTER REVISION NOTES
> 30-minute revision handbook · Production-focused · Interview-optimized

---

## 1. MOST ASKED NODE.JS INTERVIEW TOPICS

### 🔴 Very Frequently Asked
- Event Loop (phases, order, tricky outputs)
- Callbacks → Promises → async/await
- Microtasks vs Macrotasks (`process.nextTick`, `setImmediate`, `Promise`)
- Express.js middleware chain & error handling
- JWT authentication flow
- Clustering vs Worker Threads
- REST API design & HTTP status codes
- Error handling (async, centralized, unhandled rejections)
- Memory leaks & debugging in production

### 🟡 Frequently Asked
- Node.js architecture (libuv, V8, thread pool)
- Streams & Buffers
- Connection pooling
- Rate limiting & security (CORS, Helmet, XSS, CSRF)
- Redis caching basics
- Database: indexing, transactions, ORM vs raw queries
- `process.nextTick` vs `setImmediate` (output traps)

### 🟢 Rarely Asked
- Specific libuv internals beyond thread pool
- Native C++ addons
- HTTP/2 specifics
- Node.js REPL internals

---

## 2. WHAT IS NODE.JS?

**Interview answer:**
> "Node.js is a JavaScript runtime built on Chrome's V8 engine. It uses a non-blocking, event-driven I/O model, making it efficient for I/O-heavy, data-streaming, and real-time applications. It runs on a single thread but delegates I/O to the OS and CPU tasks to a thread pool via libuv."

| Concept | What it means |
|---|---|
| V8 Engine | Compiles JS to machine code |
| Non-blocking I/O | Doesn't wait for disk/network — registers callbacks |
| libuv | C library powering the event loop and thread pool |
| Event-driven | Code runs in response to events, not top-to-bottom |

**Why companies use Node.js:**
- High concurrency with low memory (vs Java/PHP thread-per-request)
- Shared JS codebase (frontend + backend)
- Great for REST APIs, real-time apps, microservices, streaming

---

## 3. NODE.JS ARCHITECTURE

**Interview answer:**
> "Node.js is single-threaded for JS execution but uses libuv's thread pool (default 4 threads) for file I/O, crypto, DNS. Network I/O uses the OS's async primitives (epoll/kqueue/IOCP) — no thread needed."

```
JS Code
   │
   ▼
V8 Engine (executes JS)
   │
   ▼
Node.js Bindings (C++ layer)
   │
   ▼
libuv
  ├── Event Loop (single thread)
  ├── Thread Pool (4 threads) → fs, crypto, dns, zlib
  └── OS Async APIs → network I/O (TCP, UDP)
```

**Thread pool tasks:** `fs.readFile`, `crypto.pbkdf2`, `dns.lookup`, `zlib`
**OS async tasks:** `http`, `net`, `child_process`

**Interviewer expects:** You know Node isn't purely single-threaded — libuv uses threads for CPU-bound I/O.

---

## 4. EVENT LOOP ⚡ (MOST IMPORTANT)

**Interview answer:**
> "The event loop is what allows Node.js to perform non-blocking I/O. It continuously checks queues and executes callbacks in a defined order across phases. Each iteration is called a 'tick'."

### Event Loop Phases (in order)

```
   ┌──────────────────────────┐
   │        timers            │  → setTimeout, setInterval callbacks
   ├──────────────────────────┤
   │     pending callbacks    │  → I/O errors deferred from last tick
   ├──────────────────────────┤
   │       idle, prepare      │  → internal only
   ├──────────────────────────┤
   │          poll            │  → retrieve new I/O events (BLOCKS HERE if empty)
   ├──────────────────────────┤
   │          check           │  → setImmediate callbacks
   ├──────────────────────────┤
   │     close callbacks      │  → socket.on('close', ...)
   └──────────────────────────┘
          ↑ loops back
```

**Between EVERY phase:** microtask queues drain first:
1. `process.nextTick` queue (highest priority)
2. Promise `.then` / microtask queue

### Execution Priority (fastest → slowest)
```
process.nextTick  >  Promise.then  >  setImmediate  >  setTimeout(fn, 0)
```

### Critical Rules
- `process.nextTick` runs **before** I/O callbacks, before poll
- `setImmediate` runs in **check phase** (after poll)
- `setTimeout(fn, 0)` runs in **timers phase** (before poll)
- Inside I/O callback: `setImmediate` fires before `setTimeout`

### ⚠️ Tricky Output Examples

**Example 1:**
```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
process.nextTick(() => console.log('4'));
console.log('5');
```
**Output:** `1, 5, 4, 3, 2`
> Sync → nextTick → Promise → setTimeout

**Example 2:**
```js
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
```
**Output:** Order is NON-DETERMINISTIC at top level.
But inside an I/O callback, `setImmediate` always fires first.

**Example 3:**
```js
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));
setImmediate(() => console.log('immediate'));
setTimeout(() => console.log('timeout'), 0);
```
**Output:** `nextTick → promise → timeout → immediate`
> *(timeout vs immediate order may vary at top level)*

---

## 5. CALLBACKS, PROMISES, ASYNC-AWAIT

### Comparison Table

| Feature | Callback | Promise | async/await |
|---|---|---|---|
| Error handling | `if(err)` check | `.catch()` | `try/catch` |
| Readability | Poor (nested) | Medium | Best |
| Chaining | Callback hell | `.then()` chain | Sequential |
| Debugging | Hard | Medium | Easy |
| Concurrent | Manual | `Promise.all` | `Promise.all` |

### Callback Hell
```js
getUser(id, (err, user) => {
  getPosts(user.id, (err, posts) => {
    getComments(posts[0].id, (err, comments) => {
      // hell...
    });
  });
});
```

### Promise Chaining
```js
getUser(id)
  .then(user => getPosts(user.id))
  .then(posts => getComments(posts[0].id))
  .catch(err => console.error(err));
```

### async/await (preferred)
```js
async function getData(id) {
  try {
    const user = await getUser(id);
    const posts = await getPosts(user.id);
    const comments = await getComments(posts[0].id);
    return comments;
  } catch (err) {
    console.error(err);
  }
}
```

### ⚠️ Common Mistakes
- Forgetting `await` → promise is returned, not resolved value
- Using `await` inside `forEach` → doesn't work as expected, use `for...of`
- Not catching errors in async functions → unhandled rejections
- `async/await` is syntactic sugar — it still uses promises underneath

```js
// WRONG: forEach with await
users.forEach(async (user) => {
  await sendEmail(user); // these run concurrently, not sequentially
});

// CORRECT:
for (const user of users) {
  await sendEmail(user); // sequential
}

// OR for parallel:
await Promise.all(users.map(user => sendEmail(user)));
```

---

## 6. MICROTASKS VS MACROTASKS

| Type | APIs | Queue |
|---|---|---|
| Microtask | `Promise.then`, `queueMicrotask`, `MutationObserver` | Microtask queue |
| Special Microtask | `process.nextTick` | nextTick queue (runs first) |
| Macrotask | `setTimeout`, `setInterval`, `setImmediate`, I/O | Callback/event queue |

**Rule:** After every macrotask, ALL microtasks drain before the next macrotask.

```
[Macrotask] → [All Microtasks] → [Macrotask] → [All Microtasks] → ...
             ↑ nextTick first, then Promise.then
```

**Output trap:**
```js
setTimeout(() => {
  console.log('timeout');
  Promise.resolve().then(() => console.log('promise in timeout'));
}, 0);

Promise.resolve().then(() => console.log('outer promise'));
```
**Output:** `outer promise → timeout → promise in timeout`

---

## 7. EXPRESS.JS INTERVIEW TOPICS

**What is middleware?**
> "Middleware functions are functions that have access to `req`, `res`, and `next`. They execute in order, can modify the request/response, and either end the cycle or pass to the next middleware."

### Middleware Flow
```
Request → middleware1 → middleware2 → route handler → response
                ↓ (if error thrown)
          error middleware (4 params: err, req, res, next)
```

### Types of Middleware
| Type | Example |
|---|---|
| Application-level | `app.use(logger)` |
| Router-level | `router.use(auth)` |
| Error-handling | `app.use((err, req, res, next) => {})` |
| Built-in | `express.json()`, `express.static()` |
| Third-party | `cors()`, `helmet()`, `morgan()` |

### Key Patterns
```js
// Error middleware — MUST have 4 params
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: err.message });
});

// next() passes to next middleware
// next(err) passes to error middleware

// Route params
app.get('/users/:id', (req, res) => {
  const { id } = req.params; // path param
  const { page } = req.query; // query param
  const body = req.body; // body (needs express.json())
});
```

### ⚠️ Common Mistakes
- Forgetting `express.json()` → `req.body` is undefined
- Not calling `next()` → request hangs
- Calling `res.send()` twice → "headers already sent" error
- Putting error middleware before routes

---

## 8. REST API DESIGN

### REST Principles
- **Stateless** — each request contains all info needed
- **Uniform interface** — consistent resource URLs
- **Client-server** — decoupled frontend/backend
- **Cacheable** — responses should define cacheability

### HTTP Methods & Idempotency

| Method | Action | Idempotent | Safe |
|---|---|---|---|
| GET | Read | ✅ | ✅ |
| POST | Create | ❌ | ❌ |
| PUT | Replace | ✅ | ❌ |
| PATCH | Partial update | ❌ | ❌ |
| DELETE | Delete | ✅ | ❌ |

### Status Codes (must know)
| Code | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 204 | No Content (DELETE success) |
| 400 | Bad Request |
| 401 | Unauthorized (not logged in) |
| 403 | Forbidden (logged in, no permission) |
| 404 | Not Found |
| 409 | Conflict |
| 422 | Unprocessable Entity (validation error) |
| 429 | Too Many Requests |
| 500 | Internal Server Error |

### Pagination
```js
// Offset-based
GET /users?page=2&limit=10

// Cursor-based (better for large datasets)
GET /users?cursor=abc123&limit=10
```

### Versioning
```
/api/v1/users   // URL versioning (most common)
/api/v2/users
```

---

## 9. AUTHENTICATION & AUTHORIZATION

### JWT Flow
```
Login → server signs payload with secret → returns JWT
Request → client sends JWT in Authorization header
Server → verifies signature → extracts payload → grants access
```

**JWT Structure:** `header.payload.signature` (base64 encoded, NOT encrypted)

```js
// Sign
const token = jwt.sign({ userId: user.id, role: 'admin' }, process.env.JWT_SECRET, { expiresIn: '15m' });

// Verify middleware
const auth = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
};
```

### JWT vs Session

| | JWT | Session |
|---|---|---|
| Storage | Client (localStorage/cookie) | Server (memory/Redis) |
| Stateless | ✅ | ❌ |
| Scalable | ✅ (no server state) | ❌ (needs sticky sessions or Redis) |
| Revocable | ❌ (until expiry) | ✅ (delete session) |
| Size | Larger (in token) | Small (just session ID) |

### Refresh Token Pattern
```
Access Token: short-lived (15min), stored in memory
Refresh Token: long-lived (7d), stored in httpOnly cookie
Flow: access token expires → send refresh token → get new access token
```

### Password Hashing
```js
// Never store plain text. Use bcrypt.
const hash = await bcrypt.hash(password, 12); // saltRounds=12
const match = await bcrypt.compare(plainPassword, hash);
```

### ⚠️ Security Mistakes
- Storing JWT in localStorage (XSS vulnerable) — use httpOnly cookies
- Weak JWT secrets
- Not expiring tokens
- Storing plain-text passwords

---

## 10. DATABASE & BACKEND CONCEPTS

### Connection Pooling
> "Instead of creating a new DB connection per request (expensive), a pool maintains a set of reusable connections. Requests borrow from the pool and return after use."

```js
// pg pool example
const pool = new Pool({ max: 10, idleTimeoutMillis: 30000 });
const result = await pool.query('SELECT * FROM users WHERE id = $1', [id]);
```

### Indexing Basics
- Index = fast lookup data structure (B-tree by default)
- Add indexes on: foreign keys, frequently queried columns, ORDER BY columns
- Too many indexes → slow writes
- **Composite index** order matters: `(a, b)` helps `WHERE a=?` and `WHERE a=? AND b=?` but NOT `WHERE b=?`

### Transactions
```js
const client = await pool.connect();
try {
  await client.query('BEGIN');
  await client.query('UPDATE accounts SET balance = balance - 100 WHERE id = 1');
  await client.query('UPDATE accounts SET balance = balance + 100 WHERE id = 2');
  await client.query('COMMIT');
} catch (err) {
  await client.query('ROLLBACK');
} finally {
  client.release();
}
```

### ORM vs Raw Queries
| | ORM (Sequelize/Prisma) | Raw SQL |
|---|---|---|
| Speed to write | Fast | Slow |
| Performance | Slower (N+1 risk) | Faster |
| Flexibility | Limited | Full |
| N+1 problem | Common pitfall | Manual control |

**N+1 problem:** Fetching 100 users then querying posts for each = 101 queries. Fix: use `include`/`JOIN`.

---

## 11. STREAMS & BUFFERS

**Interview answer:**
> "Streams allow processing data piece by piece instead of loading it all into memory. This is critical for large files or real-time data — you can start processing before the entire file is read."

### Stream Types
| Type | Description | Example |
|---|---|---|
| Readable | Source of data | `fs.createReadStream`, `http.IncomingMessage` |
| Writable | Destination | `fs.createWriteStream`, `http.ServerResponse` |
| Duplex | Both read & write | `net.Socket` |
| Transform | Modify data as it passes | `zlib.createGzip()` |

### Pipe (most common pattern)
```js
// Efficient file copy — never loads full file in memory
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));
```

### Why Streams Matter
- Upload a 1GB file without OOM crash
- Streaming video/audio
- Real-time log processing
- Transform large CSV → process row by row

### Buffers
> "A Buffer is a fixed-size chunk of memory for binary data (outside V8 heap). Used when dealing with raw bytes: file I/O, network packets, crypto."

```js
const buf = Buffer.from('hello', 'utf8');
console.log(buf.toString('hex')); // 68656c6c6f
console.log(buf.length); // 5 bytes
```

---

## 12. ERROR HANDLING

### Sync Errors
```js
try {
  JSON.parse('{bad}');
} catch (err) {
  console.error(err.message);
}
```

### Async Error Handling
```js
// async/await — ALWAYS wrap in try/catch or add .catch()
async function getData() {
  try {
    const data = await fetchFromDB();
    return data;
  } catch (err) {
    throw new AppError('DB fetch failed', 500);
  }
}

// Promise — attach .catch()
fetchData().then(process).catch(handleError);
```

### Centralized Express Error Middleware
```js
// Custom error class
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
  }
}

// Centralized handler (last middleware)
app.use((err, req, res, next) => {
  const status = err.statusCode || 500;
  res.status(status).json({
    success: false,
    message: err.message || 'Internal Server Error'
  });
});

// Async wrapper (avoid try/catch in every route)
const asyncHandler = fn => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get('/users', asyncHandler(async (req, res) => {
  const users = await User.findAll(); // any throw → goes to error middleware
  res.json(users);
}));
```

### Unhandled Rejections (Production must-have)
```js
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection:', reason);
  process.exit(1); // or graceful shutdown
});

process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  process.exit(1);
});
```

---

## 13. SCALING NODE.JS APPLICATIONS

### Clustering
> "Node.js is single-threaded, so it can't use multiple CPU cores by default. Clustering forks N worker processes (typically = CPU count), each with its own event loop, sharing the same port."

```js
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  cluster.on('exit', (worker) => cluster.fork()); // restart crashed workers
} else {
  require('./app'); // each worker runs the app
}
```

### Clustering vs Worker Threads

| | Cluster | Worker Threads |
|---|---|---|
| Use case | Scale I/O-bound apps across CPUs | CPU-bound tasks (image processing, ML) |
| Memory | Separate memory per worker | Shared memory (`SharedArrayBuffer`) |
| Isolation | Full process isolation | Same process |
| Communication | IPC (message passing) | `postMessage` / `SharedArrayBuffer` |

### Horizontal Scaling
```
Client → Load Balancer (Nginx/AWS ALB)
             ├── Node instance 1
             ├── Node instance 2
             └── Node instance 3
                      ↓
                Redis (shared sessions/cache)
                      ↓
                Database
```

### Redis Caching
```js
// Cache-aside pattern
async function getUser(id) {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);

  const user = await db.findUser(id);
  await redis.setex(`user:${id}`, 3600, JSON.stringify(user)); // TTL: 1hr
  return user;
}
```

### Production Scaling Checklist
- Use PM2 in cluster mode (easier than raw cluster)
- Redis for session storage (not in-memory)
- Redis for rate limiting counters
- Database connection pooling
- Horizontal scaling behind load balancer
- CDN for static assets

---

## 14. SECURITY INTERVIEW TOPICS

### CORS
```js
const cors = require('cors');
app.use(cors({
  origin: ['https://myapp.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  credentials: true // allow cookies
}));
```

### Rate Limiting
```js
const rateLimit = require('express-rate-limit');
app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  message: { error: 'Too many requests' }
}));
```

### Helmet (HTTP security headers)
```js
app.use(helmet()); // sets X-Frame-Options, X-XSS-Protection, HSTS, etc.
```

### Attack Vectors & Fixes

| Attack | Description | Fix |
|---|---|---|
| SQL Injection | Malicious SQL in input | Parameterized queries / ORM |
| NoSQL Injection | `{$gt: ''}` in MongoDB queries | Validate/sanitize input, `express-mongo-sanitize` |
| XSS | JS injected into pages | Sanitize output, CSP headers, `helmet` |
| CSRF | Forged cross-site requests | CSRF tokens, SameSite cookies |
| Brute Force | Password guessing | Rate limiting, account lockout |

```js
// SQL Injection: NEVER do this
db.query(`SELECT * FROM users WHERE email = '${email}'`); // 💀

// CORRECT: parameterized
db.query('SELECT * FROM users WHERE email = $1', [email]); // ✅
```

---

## 15. COMMON NODE.JS INTERVIEW TRAPS

### 1. Blocking the Event Loop
```js
// ❌ Blocks event loop — no other requests served during this
app.get('/compute', (req, res) => {
  const result = heavyCpuComputation(); // synchronous, blocks!
  res.json(result);
});

// ✅ Offload to Worker Thread
const { Worker } = require('worker_threads');
```

### 2. `fs.readFileSync` in Production
```js
// ❌ Blocks all requests while reading file
const config = fs.readFileSync('./config.json');

// ✅
const config = await fs.promises.readFile('./config.json');
```

### 3. Sending Multiple Responses
```js
// ❌ "Cannot set headers after they are sent"
app.get('/', (req, res) => {
  if (!auth) res.status(401).json({ error: 'Unauthorized' }); // missing return!
  res.json({ data: '...' }); // still executes!
});

// ✅ Always return after sending
if (!auth) return res.status(401).json({ error: 'Unauthorized' });
```

### 4. Memory Leaks
- Global variables holding large data
- Uncleaned event listeners (`emitter.on` without `removeListener`)
- Closures keeping references alive
- Caching without TTL / cache eviction

```js
// Detecting: node --inspect app.js → Chrome DevTools → Memory snapshot
```

### 5. Unhandled Promise Rejections
```js
// ❌ Silent failure
async function riskyOp() {
  await mightFail(); // if it throws, nothing happens
}
riskyOp(); // called without .catch() or await

// ✅
riskyOp().catch(console.error);
// OR use process.on('unhandledRejection', ...)
```

### 6. `await` in forEach
```js
// ❌ All run in parallel, not sequential
items.forEach(async (item) => await process(item));

// ✅ Sequential
for (const item of items) await process(item);

// ✅ Parallel (intentional)
await Promise.all(items.map(item => process(item)));
```

---

## 16. TOP OUTPUT-BASED QUESTIONS

### Q1 — Classic event loop order
```js
console.log('A');
setTimeout(() => console.log('B'), 0);
Promise.resolve().then(() => console.log('C'));
process.nextTick(() => console.log('D'));
console.log('E');
```
**Output:** `A E D C B`
> Sync → nextTick → Promise → setTimeout

---

### Q2 — Nested nextTick
```js
process.nextTick(() => {
  console.log('nextTick 1');
  process.nextTick(() => console.log('nextTick 2'));
});
Promise.resolve().then(() => console.log('promise'));
```
**Output:** `nextTick 1 → nextTick 2 → promise`
> nextTick drains fully before Promise queue starts

---

### Q3 — setImmediate vs setTimeout in I/O
```js
const fs = require('fs');
fs.readFile('./file', () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
});
```
**Output:** `immediate → timeout` (always)
> Inside I/O callback, check phase (setImmediate) comes before next timers phase

---

### Q4 — Promise chaining order
```js
Promise.resolve()
  .then(() => { console.log(1); return Promise.resolve(2); })
  .then(v => console.log(v));

Promise.resolve()
  .then(() => console.log(3))
  .then(() => console.log(4));
```
**Output:** `1 → 3 → 2 → 4`
> `return Promise.resolve()` adds an extra microtask tick

---

### Q5 — Async/await trap
```js
async function foo() {
  console.log('A');
  await Promise.resolve();
  console.log('B');
}
console.log('C');
foo();
console.log('D');
```
**Output:** `C → A → D → B`
> `await` yields control back; 'D' runs before 'B'

---

### Q6 — Middleware order trap
```js
app.use((req, res, next) => { console.log('1'); next(); });
app.get('/', (req, res, next) => { console.log('2'); next(); });
app.use((req, res, next) => { console.log('3'); res.send('done'); });
```
**GET /** → `1 → 2 → 3`

---

### Q7 — Closure trap
```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```
**Output:** `3 3 3` (var is function-scoped, not block-scoped)
**Fix:** Use `let` instead of `var` → outputs `0 1 2`

---

### Q8 — Promise.all vs sequential
```js
// How long does this take if each takes 1s?
const [a, b, c] = await Promise.all([
  delay(1000), delay(1000), delay(1000)
]);
// ~1 second (parallel)

// vs sequential:
const a = await delay(1000);
const b = await delay(1000);
const c = await delay(1000);
// ~3 seconds
```

---

## 17. TOP INTERVIEW Q&A (SHORT ANSWERS)

**Q: What is Node.js?**
> "A JS runtime built on V8, using non-blocking I/O and an event-driven model, ideal for I/O-heavy and real-time applications."

**Q: Why is Node.js single-threaded?**
> "JavaScript was designed as a single-threaded language to avoid concurrency complexity. Node handles concurrency through the event loop and async I/O, not threads."

**Q: What is the event loop?**
> "A mechanism that allows Node to handle async operations. It runs in phases — timers, poll, check — picking up callbacks from queues and executing them without blocking."

**Q: process.nextTick vs setImmediate?**
> "`process.nextTick` fires at the end of the current operation, before any I/O or timers. `setImmediate` fires in the check phase after the poll phase. nextTick is faster/higher priority."

**Q: Promise vs async/await?**
> "async/await is syntactic sugar over Promises. It makes async code read like synchronous code. Both use the microtask queue. async/await makes error handling and debugging easier."

**Q: What are Streams?**
> "Streams process data chunk by chunk instead of loading it all into memory. Critical for large files, video streaming, and real-time data. Four types: Readable, Writable, Duplex, Transform."

**Q: What is middleware in Express?**
> "Functions that intercept the request-response cycle. They receive req, res, and next. They can modify the request, authenticate users, log, or terminate the cycle."

**Q: JWT vs Session?**
> "JWT is stateless — the server doesn't store anything, the client holds the token. Sessions are stateful — server stores session data. JWT scales better horizontally; sessions are easier to revoke."

**Q: Clustering vs Worker Threads?**
> "Clustering forks multiple processes to use multiple CPU cores for I/O-bound apps. Worker Threads run JS in parallel threads within the same process for CPU-bound tasks."

**Q: Why is Node.js fast?**
> "V8 compiles JS to machine code, the event loop avoids thread-per-request overhead, and libuv handles async I/O efficiently using OS primitives."

**Q: What is libuv?**
> "A C library that provides the event loop, thread pool, and async I/O for Node.js. It abstracts OS-level async APIs (epoll, kqueue, IOCP)."

**Q: How does Node handle concurrency?**
> "Through the event loop — I/O operations are delegated to the OS or thread pool, and callbacks are queued and executed when ready, all on a single JS thread."

---

## 18. PROJECT-BASED INTERVIEW QUESTIONS

### How to answer using your projects

**Q: How did you handle authentication in your project?**
> "I implemented JWT-based auth with short-lived access tokens (15min) and refresh tokens stored in httpOnly cookies. Passwords are hashed with bcrypt. The auth middleware verifies the token on every protected route."

**Q: How did you scale your API?**
> "Used PM2 in cluster mode to utilize all CPU cores, Redis for caching frequent queries and session storage, and added a rate limiter. For the DB, we used connection pooling with pg-pool."

**Q: How did you handle errors in production?**
> "Centralized error middleware in Express, custom AppError class with status codes, async wrapper to catch all async errors, and `process.on('unhandledRejection')` for uncaught cases. All errors logged to a monitoring service."

**Q: How did you handle background jobs?**
> "Used Bull (Redis-based queue) for background jobs like email sending and report generation. Jobs are retried on failure, and we have dead-letter queues for inspection."

**Q: How did you handle DB performance?**
> "Added indexes on frequently queried columns, used connection pooling, cached common queries in Redis, and optimized N+1 problems by using eager loading/JOINs."

---

## 19. DEBUGGING & PRODUCTION QUESTIONS

### Memory Leaks
**Signs:** Memory grows over time, eventual OOM crash
**Causes:** Uncleaned event listeners, global state, closures, unbounded caches
**Debug:**
```bash
node --inspect app.js
# Open chrome://inspect → Take heap snapshots → Compare over time
```

### High CPU Usage
**Signs:** Event loop blocked, all requests slow
**Causes:** Sync code, CPU-heavy computation in main thread
**Fix:** Move to Worker Thread, or queue as background job

### Slow APIs
**Debug approach:**
1. Add timing logs to identify slow operations
2. Check DB query performance (EXPLAIN ANALYZE)
3. Check N+1 queries
4. Check missing Redis cache
5. Profile with `--prof` flag → `node --prof-process`

### Logging Strategy
```js
// Use structured logging (Winston/Pino)
const logger = require('pino')();

logger.info({ userId: user.id, action: 'login' }, 'User logged in');
logger.error({ err, requestId }, 'Request failed');

// Log levels: error > warn > info > debug > trace
// In production: info and above
```

### Debugging Async Code
```js
// Source maps for stack traces
// Use --async-stack-traces flag (Node 12+)
// Add requestId to all log entries for tracing
const asyncLocalStorage = new AsyncLocalStorage(); // track context across async calls
```

---

## 20. ⚡ 5-MINUTE FINAL REVISION SHEET

### Event Loop (30 seconds)
```
Sync code → nextTick → Promise.then → setTimeout/setImmediate → I/O callbacks
Inside I/O: setImmediate BEFORE setTimeout
Top level: setTimeout vs setImmediate = NON-DETERMINISTIC
```

### Async Recap (20 seconds)
- `async/await` = Promises + syntactic sugar
- `await` in `forEach` = WRONG, use `for...of`
- Parallel = `Promise.all([...])`
- Always `.catch()` or `try/catch`

### Middleware Recap (15 seconds)
- `app.use()` = applies to all routes
- Must call `next()` or send response
- Error middleware = 4 params `(err, req, res, next)`
- Order matters — put error handler last

### JWT Flow (20 seconds)
```
Login → sign({userId}, secret, {expiresIn}) → return token
Request → Authorization: Bearer <token>
Server → jwt.verify(token, secret) → extract payload
```

### Common Traps (30 seconds)
- `readFileSync` in routes = blocks event loop
- Missing `return` before `res.send` = double response error
- `var` in loops with setTimeout = all log same value
- `await` inside `forEach` = runs concurrently
- Storing JWT in localStorage = XSS vulnerable

### Backend One-Liners
- **Node is fast because:** event loop + non-blocking I/O + V8 JIT
- **Cluster for:** multiple CPU cores, I/O-bound scaling
- **Worker Threads for:** CPU-heavy tasks (image/video processing)
- **Redis for:** caching, sessions, rate limiting, pub/sub, queues
- **Connection pool because:** new DB connection is expensive (~100ms)
- **Streams because:** process large data without loading into memory
- **bcrypt because:** slow by design (resistant to brute force)
- **JWT stateless:** server stores nothing, scales horizontally
- **process.nextTick:** highest priority, fires before I/O
- **setImmediate:** check phase, after poll, perfect for post-I/O work

---

> 💡 **Last 60 seconds:** Read section 20, then mentally trace one event loop example. You're ready.
