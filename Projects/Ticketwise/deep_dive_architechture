# TicketWise — Detailed Architecture Deep Dive

> This document explains **exactly** what happens at every layer of TicketWise, tracing real data through real code paths. Read this before your interview.

---

## Table of Contents

1. [The Big Picture — Why This Architecture Exists](#1-the-big-picture)
2. [Layer 1 — Frontend (React + Vite)](#2-layer-1--frontend)
   - 2.1 SPA Structure & Routing
   - 2.2 Route Guards — Three Types, Three Purposes
   - 2.3 Auth Token Lifecycle
   - 2.4 API Communication Pattern
   - 2.5 Ticket Dashboard — What Happens Client-Side
3. [Layer 2 — Backend (Node.js + Express 5)](#3-layer-2--backend)
   - 3.1 Server Bootstrap Sequence
   - 3.2 The `/api/inngest` Endpoint — What It Actually Is
   - 3.3 Auth Middleware — Exact Execution Path
   - 3.4 User Controller — Signup & Login Deep Dive
   - 3.5 Ticket Controller — Role-Based Query Logic
4. [Layer 3 — Inngest Event Queue](#4-layer-3--inngest-event-queue)
   - 4.1 What Inngest Actually Does Under the Hood
   - 4.2 Function 1: `on-user-signup`
   - 4.3 Function 2: `on-ticket-created` — All 5 Steps
   - 4.4 Step Isolation — Why It Matters
5. [Layer 4 — External Services](#5-layer-4--external-services)
   - 5.1 Resend Email API
   - 5.2 Google Gemini Flash API — Prompt Engineering
   - 5.3 AI Response Parsing & Safety Net
6. [Database Design — Schema Decisions](#6-database-design)
   - 6.1 User Schema
   - 6.2 Ticket Schema
   - 6.3 Mongoose `populate()` — How Joins Work in MongoDB
7. [Authentication & Authorization — Two-Layer Model](#7-authentication--authorization)
8. [Complete End-to-End Data Flows](#8-complete-end-to-end-data-flows)
   - 8.1 Signup Flow
   - 8.2 Ticket Creation Flow
   - 8.3 Ticket Fetch Flow (Role-Aware)
9. [Key Design Decisions — The Reasoning](#9-key-design-decisions)

---

## 1. The Big Picture

TicketWise follows a **decoupled, event-driven architecture**. The core principle is:

> **The HTTP request/response cycle does the minimum possible. Everything else is asynchronous.**

When a user submits a ticket, the backend does exactly two things before responding:
1. Saves the ticket to MongoDB.
2. Fires an event to Inngest.

That's it. The user gets a `201 Created` response immediately. Meanwhile, Inngest handles the slow work — AI analysis (~2–4 seconds), moderator matching (a DB query), and email delivery — completely outside the request cycle.

This means:
- **The user never waits for AI.** The ticket appears instantly in the UI.
- **Failures don't cascade.** If Gemini is down, the ticket is still saved. Inngest retries the AI call later.
- **The backend stays stateless.** No long-running processes, no timeouts, no hanging connections.

---

## 2. Layer 1 — Frontend

### 2.1 SPA Structure & Routing

TicketWise is a **Single Page Application** built with React 19 and Vite. There is only one actual HTML file served — `index.html`. React Router handles all navigation client-side, meaning the browser never does a full page reload after the initial load.

Five pages exist:

| Page | Route | Who Can See It |
|---|---|---|
| Login | `/login` | Only unauthenticated users |
| Signup | `/signup` | Only unauthenticated users |
| Tickets | `/` | All authenticated users |
| Ticket Detail | `/tickets/:id` | All authenticated users |
| Admin | `/admin` | Only users with `role: "admin"` |

### 2.2 Route Guards — Three Types, Three Purposes

Route guards are React wrapper components. They sit between the router and the actual page component, checking conditions before deciding what to render.

**ProtectedRoute** — Guards pages that require login:

```jsx
function ProtectedRoute({ children }) {
    const token = localStorage.getItem("token");
    if (!token) return <Navigate to="/login" replace />;
    return children;
}
```

What it does: Reads `localStorage` for a JWT. If the token is missing, it redirects to `/login`. If the token exists (even if it's expired!), it renders the page. The backend will reject expired tokens — the frontend just checks for presence.

**PublicRoute** — Guards login/signup from already-authenticated users:

```jsx
function PublicRoute({ children }) {
    const token = localStorage.getItem("token");
    if (token) return <Navigate to="/" replace />;
    return children;
}
```

What it does: If you're logged in and try to visit `/login`, you get bounced to `/` instead. This prevents the awkward situation of a logged-in user re-logging in.

**AdminRoute** — Guards the admin page from non-admins:

```jsx
function AdminRoute({ children }) {
    const user = JSON.parse(localStorage.getItem("user"));
    if (!user || user.role !== "admin") return <Navigate to="/" replace />;
    return children;
}
```

What it does: Reads the stored `user` object and checks its `role` field. Critical note: **this is UI-level protection only**. If someone manually puts `{"role":"admin"}` in their localStorage, they'll see the admin page, but every API call they make will fail because the backend checks the JWT independently.

### 2.3 Auth Token Lifecycle

```
1. User logs in → POST /api/auth/login
2. Backend returns { token, user }
3. Frontend stores:
   - localStorage.setItem("token", token)        ← JWT string
   - localStorage.setItem("user", JSON.stringify(user))  ← user object with role/email
4. Token is included in every subsequent API call as:
   Authorization: Bearer <token>
5. On logout:
   - localStorage.removeItem("token")
   - localStorage.removeItem("user")
   - Redirect to /login
```

`localStorage` persists across browser sessions (unlike `sessionStorage`). This means users stay logged in after closing and reopening the browser.

### 2.4 API Communication Pattern

All API calls use the native `fetch()` API — no Axios. The backend URL comes from Vite's environment variable system:

```js
const res = await fetch(`${import.meta.env.VITE_SERVER_URL}/api/tickets`, {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${localStorage.getItem("token")}`,
    },
    body: JSON.stringify({ title, description }),
});
```

`VITE_SERVER_URL` is defined in a `.env` file (e.g., `http://localhost:3000`). Vite injects it at build time — it's baked into the frontend bundle, not a runtime config.

### 2.5 Ticket Dashboard — What Happens Client-Side

The dashboard fetches all tickets once on mount (`useEffect`), then does all filtering/searching client-side:

- **Stats** — computed with `useMemo` to avoid recalculating on every render:
  ```js
  const stats = useMemo(() => ({
      total: tickets.length,
      active: tickets.filter(t => t.status === "IN_PROGRESS").length,
      resolved: tickets.filter(t => t.status === "DONE").length,
  }), [tickets]);
  ```

- **Search** — also `useMemo`, filters tickets array by `title` and `description` matching the search string.

- **Animations** — Framer Motion wraps each ticket card. Cards animate in when they first mount (a fade+slide effect). The "Create Ticket" form expands/collapses with an animated height transition.

---

## 3. Layer 2 — Backend

### 3.1 Server Bootstrap Sequence

When the Node.js process starts (`node index.js`), it runs these steps in order:

```
1. Import Express, create app
2. app.use(cors())          ← Allow cross-origin requests from the React frontend
3. app.use(express.json())  ← Parse JSON request bodies
4. Register route groups:
   app.use("/api/auth",    userRoutes)
   app.use("/api/tickets", ticketRoutes)
   app.use("/api/inngest", serve({ client: inngest, functions: [...] }))
5. mongoose.connect(MONGODB_URI)  ← Connect to MongoDB Atlas
6. app.listen(PORT)               ← Start accepting connections
```

Step 4 is important: routes are registered before the DB connects. Express is ready to accept requests immediately, but database queries won't work until Mongoose connects. In practice, the connection is fast enough that this never causes issues, but it's a nuance worth knowing.

### 3.2 The `/api/inngest` Endpoint — What It Actually Is

This is not a normal REST endpoint you call. It's a **webhook handler** registered by the Inngest SDK via `serve()`.

When Inngest Cloud wants to run a background function (e.g., `on-ticket-created`), it sends an HTTP POST to `https://your-server.com/api/inngest`. The `serve()` handler receives this, authenticates the request using a signing key, and executes the appropriate function.

So the flow is:
```
Your backend fires inngest.send(event)
    → Event goes to Inngest Cloud servers
    → Inngest Cloud calls back YOUR server at /api/inngest
    → Your server runs the function steps
    → Inngest Cloud manages retries, logging, etc.
```

This is why Inngest needs your server to be publicly accessible (or use a tunnel like ngrok in development).

### 3.3 Auth Middleware — Exact Execution Path

```js
export const authenticate = (req, res, next) => {
    // Step 1: Extract token from "Authorization: Bearer eyJhb..."
    const token = req.headers.authorization?.split(" ")[1];

    // Step 2: If no token → 401 immediately
    if (!token) return res.status(401).json({ error: "Access Denied." });

    // Step 3: Verify signature and decode payload
    // jwt.verify throws if token is expired or tampered with
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    // Step 4: Attach decoded payload to req so controllers can use it
    req.user = decoded;  // { _id, email, role, iat, exp }

    // Step 5: Pass control to the next middleware or controller
    next();
};
```

`jwt.verify()` does two things: it checks the signature (was this JWT created by us?), and it checks the expiry (`exp` claim). If either fails, it throws a `JsonWebTokenError` or `TokenExpiredError`. In Express 5, unhandled thrown errors are automatically passed to error-handling middleware — you don't need to wrap it in try/catch (though you should in production for custom error messages).

### 3.4 User Controller — Signup & Login Deep Dive

**Signup:**
```
1. Receive { email, password } from request body
2. bcrypt.hash(password, 10)
   ← The "10" is the salt rounds. Each round doubles hashing time.
   ← 10 rounds ≈ 65ms on modern hardware — slow enough to deter brute force
3. User.create({ email, hashedPassword, role: "user", skills: [] })
4. inngest.send({ name: "user/signup", data: { email } })
   ← Fire and forget — we don't await this for the response
5. res.status(201).json({ message: "User created" })
```

**Login:**
```
1. Receive { email, password }
2. User.findOne({ email })  ← Fetch user from DB
3. bcrypt.compare(password, user.password)
   ← bcrypt rehashes the input and compares with stored hash
   ← Never decrypts — hashing is one-way
4. If match: jwt.sign({ _id, email, role }, JWT_SECRET, { expiresIn: "7d" })
5. res.json({ token, user: { _id, email, role } })
```

### 3.5 Ticket Controller — Role-Based Query Logic

**Create Ticket:**
```js
const ticket = await Ticket.create({
    title: req.body.title,
    description: req.body.description,
    createdBy: req.user._id,  // ← from JWT payload
    status: "TODO",           // ← default, AI will change later
});

await inngest.send({
    name: "ticket/created",
    data: { ticketId: ticket._id.toString() }
});

res.status(201).json({ ticket });
```

**Fetch Tickets (Role-Based):**

```js
let tickets;

if (req.user.role !== "user") {
    // Admins/moderators: get ALL tickets with full relational data
    tickets = await Ticket.find({})
        .populate("assignedTo", "email _id")   // joins User collection
        .populate("createdBy",  "email _id");
} else {
    // Regular users: only their own tickets, limited fields
    tickets = await Ticket.find({ createdBy: req.user._id })
        .select("title description status createdAt");
    // No populate — they don't need to know who's assigned
}

res.json({ tickets });
```

The `.populate()` calls do a second MongoDB query behind the scenes — essentially a JOIN. When an admin fetches tickets, each `assignedTo` ObjectId gets replaced with the actual User document (email + id).

---

## 4. Layer 3 — Inngest Event Queue

### 4.1 What Inngest Actually Does Under the Hood

When you call `inngest.send({ name: "ticket/created", data: {...} })`, here's what happens:

1. The Inngest client makes an HTTP request to Inngest Cloud with the event payload.
2. Inngest Cloud receives it, timestamps it, and stores it.
3. Inngest Cloud identifies which registered functions listen for `ticket/created`.
4. Inngest Cloud calls back your server's `/api/inngest` endpoint to start the function.
5. Your function runs step by step. After each `step.run()` completes, its return value is **checkpointed** by Inngest Cloud.
6. If any step fails, Inngest retries only that step (using the checkpoint), not the whole function.

This is fundamentally different from a message queue like Redis/Bull. Inngest is a **durable execution platform** — it guarantees that each step runs exactly once (at-least-once with idempotency), even across server restarts.

### 4.2 Function 1: `on-user-signup`

```
Trigger: { name: "user/signup", data: { email } }
Config: retries: 2

Step 1 — "get-user-email":
    User.findOne({ email: { $regex: new RegExp(email, "i") } })
    Returns: full user document

Step 2 — "send-welcome-email":
    resend.emails.send({
        from: "TicketWise <onboarding@resend.dev>",
        to: [user.email],
        subject: "Welcome to TicketWise",
        text: "Your account has been created..."
    })
```

Why the regex for email lookup? Email matching with `$regex` and `"i"` flag makes it case-insensitive — `User@Example.com` will still find the user stored as `user@example.com`. A better production approach would be to lowercase emails on save.

### 4.3 Function 2: `on-ticket-created` — All 5 Steps

```
Trigger: { name: "ticket/created", data: { ticketId } }
Config: retries: 2

Step 1 — "fetch-ticket":
    Ticket.findById(ticketId)
    Returns: { title, description, createdBy, ... }
    Output flows into next steps as "ticket" variable

Step 2 — AI Analysis (NOT wrapped in step.run):
    aiAnalysis(ticket.title, ticket.description)
    ← Calls Gemini Flash REST API
    Returns: { summary, priority, helpfulNotes, relatedSkills }

    ⚠️ This step is NOT inside step.run() because @inngest/agent-kit
    uses Inngest steps internally. Nesting steps causes a NESTING_STEPS error.

Step 3 — "update-ticket-with-ai-data":
    Ticket.findByIdAndUpdate(ticketId, {
        priority:     aiResult.priority,
        helpfulNotes: aiResult.helpfulNotes,
        relatedSkills: aiResult.relatedSkills,
        summary:      aiResult.summary,
        status:       "TODO",
    })

Step 4 — "assign-moderator":
    // Build regex from relatedSkills array: "React|OAuth|Node.js"
    const skillPattern = relatedSkills.join("|");

    let moderator = await User.findOne({
        role: "moderator",
        skills: { $elemMatch: { $regex: skillPattern, $options: "i" } }
    });

    // Fallback chain:
    if (!moderator) moderator = await User.findOne({ role: "admin" });
    if (!moderator) { /* ticket stays unassigned */ return; }

    await Ticket.findByIdAndUpdate(ticketId, {
        assignedTo: moderator._id,
        status: "IN_PROGRESS",
    });

Step 5 — "send-email-notification":
    resend.emails.send({
        to: [moderator.email],
        subject: `New ticket assigned: ${ticket.title}`,
        text: `Priority: ${priority}\nNotes: ${helpfulNotes}...`
    })
```

### 4.4 Step Isolation — Why It Matters

Consider this scenario: Steps 1–3 succeed. Step 4 (assign-moderator) fails because MongoDB is briefly unreachable.

- **Without Inngest:** You'd need to re-run everything from Step 1 — re-fetch the ticket, call Gemini again (costs money, takes time), re-update the ticket. And if Step 4 fails again, you'd do it all again.

- **With Inngest:** Steps 1–3 are checkpointed. Inngest retries only Step 4. Gemini is never called twice. The ticket update is never duplicated. This is step isolation.

Each `step.run()` call saves its return value. On retry, Inngest replays the function but skips completed steps, injecting their saved outputs directly.

---

## 5. Layer 4 — External Services

### 5.1 Resend Email API

Resend is used instead of SMTP (Nodemailer) because:
- No mail server setup required
- Reliable delivery with tracking
- Clean API — one function call sends an email
- Free tier covers small projects

```js
const resend = new Resend(process.env.RESEND_API_KEY);
await resend.emails.send({
    from: "TicketWise <onboarding@resend.dev>",
    to: [recipientEmail],
    subject: "...",
    text: "...",
});
```

The `from` address uses Resend's shared domain (`resend.dev`) in development. For production, you'd verify your own domain.

### 5.2 Google Gemini Flash API — Prompt Engineering

The AI call goes directly to Gemini's REST API in `utils/ai.js`:

```js
const response = await fetch(
    `https://generativelanguage.googleapis.com/v1beta/models/gemini-flash:generateContent?key=${API_KEY}`,
    {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
            contents: [{ parts: [{ text: prompt }] }],
            generationConfig: { temperature: 0.1 }
        })
    }
);
```

**Temperature: 0.1** — This is the key setting. Temperature controls randomness:
- `temperature: 1.0` → creative, varied, sometimes hallucinated output
- `temperature: 0.1` → nearly deterministic, consistent JSON every time

For structured data extraction (not creative writing), low temperature is essential.

**The Prompt:**

```
You are an expert AI assistant that processes technical support tickets.
You MUST respond with ONLY a valid JSON object.
Do NOT include markdown, backticks, or any text outside the JSON.

Respond with this exact structure:
{
    "summary": "one or two sentence summary",
    "priority": "low" | "medium" | "high",
    "helpfulNotes": "actionable notes for the handler",
    "relatedSkills": ["skill1", "skill2"]
}

Ticket Title: {title}
Ticket Description: {description}
```

### 5.3 AI Response Parsing & Safety Net

Even with a tight prompt, models sometimes wrap JSON in markdown fences. The code defends against this:

```js
// Strip ```json ... ``` or ``` ... ``` wrappers
const cleaned = rawText.replace(/```json\n?|```\n?/g, "").trim();

let parsed;
try {
    parsed = JSON.parse(cleaned);
} catch (e) {
    // Fallback if JSON is still invalid
    parsed = {
        summary: "Unable to analyze ticket",
        priority: "medium",
        helpfulNotes: "",
        relatedSkills: []
    };
}

// Validate priority value
const validPriorities = ["low", "medium", "high"];
if (!validPriorities.includes(parsed.priority)) {
    parsed.priority = "medium";
}
```

This three-layer defense (prompt instruction + strip fences + validate values) makes the AI integration robust without needing a schema validation library.

---

## 6. Database Design

### 6.1 User Schema

```js
{
    email:     { type: String, required: true, unique: true },
    password:  { type: String, required: true },  // bcrypt hash, never plaintext
    role:      { type: String, enum: ["user","moderator","admin"], default: "user" },
    skills:    { type: [String], default: [] },   // ["React", "Node.js", "AWS"]
    createdAt: { type: Date, default: Date.now }
}
```

The `skills` array is the linchpin of the auto-assignment system. It's what gets matched against the AI's `relatedSkills` output. A moderator with `skills: ["React", "OAuth", "JWT"]` would be matched to a ticket where the AI identified `relatedSkills: ["JWT", "authentication"]`.

### 6.2 Ticket Schema

```js
{
    title:        { type: String, required: true },
    description:  { type: String, required: true },
    status:       { type: String, enum: ["TODO","IN_PROGRESS","DONE"], default: "TODO" },
    priority:     { type: String, enum: ["low","medium","high"] },  // set by AI
    createdBy:    { type: ObjectId, ref: "User", required: true },
    assignedTo:   { type: ObjectId, ref: "User", default: null },   // set by Inngest
    helpfulNotes: { type: String },      // set by AI
    relatedSkills:{ type: [String] },    // set by AI
    summary:      { type: String },      // set by AI
    deadline:     { type: Date },
    createdAt:    { type: Date, default: Date.now }
}
```

**Lifecycle of a ticket's status field:**
```
Created: status = "TODO"   (set by backend on create)
After AI updates: status = "TODO"   (Step 3 sets it, same value, confirms AI ran)
After assignment: status = "IN_PROGRESS"  (Step 4 sets this)
After resolution: status = "DONE"   (manually set by admin — not automated)
```

### 6.3 Mongoose `populate()` — How Joins Work in MongoDB

MongoDB is a document database — it has no native JOINs. Mongoose's `populate()` simulates joins with a second query.

```js
await Ticket.find({}).populate("assignedTo", "email _id")
```

Behind the scenes:
1. `Ticket.find({})` returns all tickets. `assignedTo` fields are raw ObjectIds.
2. Mongoose collects all unique `assignedTo` ObjectIds.
3. Mongoose runs `User.find({ _id: { $in: [id1, id2, ...] } }).select("email _id")`.
4. Mongoose replaces each ObjectId in the results with the matching User document.

This is 2 DB queries, not 1. For large datasets, this can be a performance concern.

---

## 7. Authentication & Authorization — Two-Layer Model

```
LAYER 1: FRONTEND (UX Protection)
┌─────────────────────────────────────────────────────┐
│ ProtectedRoute → checks localStorage for token     │
│ AdminRoute     → checks localStorage user.role     │
│ PublicRoute    → redirects logged-in users away     │
│                                                     │
│ Purpose: Prevent UI confusion, not real security    │
│ Weakness: Any user can manipulate localStorage      │
└─────────────────────────────────────────────────────┘

LAYER 2: BACKEND (Real Security)
┌─────────────────────────────────────────────────────┐
│ authenticate middleware:                            │
│   jwt.verify(token, JWT_SECRET)                    │
│   → Validates signature (was JWT made by us?)      │
│   → Validates expiry (has it expired?)             │
│   → Attaches decoded payload to req.user           │
│                                                     │
│ Controller logic:                                   │
│   req.user.role === "user"                         │
│   → Ticket.find({ createdBy: req.user._id })       │
│   → Only returns that user's own tickets           │
│                                                     │
│ Purpose: Actual security enforcement               │
│ Cannot be bypassed by client-side manipulation     │
└─────────────────────────────────────────────────────┘
```

The JWT contains the user's role, signed with `JWT_SECRET`. If someone tampers with the role in their token, `jwt.verify()` fails because the signature won't match. They can't forge a moderator token without knowing `JWT_SECRET`.

---

## 8. Complete End-to-End Data Flows

### 8.1 Signup Flow — Every Hop

```
Browser
  │── POST /api/auth/signup { email, password }
  │
Backend (Express)
  │── express.json() parses body
  │── userRoutes → signupController
  │── bcrypt.hash(password, 10) → hashedPassword
  │── User.create({ email, hashedPassword, role:"user", skills:[] })
  │── inngest.send({ name:"user/signup", data:{ email } })
  │      └── HTTP POST to Inngest Cloud (async, we don't wait)
  │── res.status(201).json({ message: "Created" })
  │
Browser
  │── Receives 201 → shows success message → redirects to /login

[Async — happening in background, user has no idea]
Inngest Cloud
  │── Receives "user/signup" event
  │── Calls back POST /api/inngest on our server
  │
Backend (Inngest handler)
  │── Step 1 "get-user-email": User.findOne({ email regex })
  │── Step 2 "send-welcome-email": resend.emails.send(...)
  │
Resend API
  └── Delivers email to user's inbox
```

### 8.2 Ticket Creation Flow — Every Hop

```
Browser
  │── POST /api/tickets { title, description }
  │── Header: Authorization: Bearer <jwt>
  │
Backend (Express)
  │── authenticate middleware:
  │     jwt.verify(token) → req.user = { _id, email, role }
  │── ticketController:
  │     Ticket.create({ title, description, createdBy: req.user._id, status:"TODO" })
  │     inngest.send({ name:"ticket/created", data:{ ticketId: ticket._id } })
  │── res.status(201).json({ ticket })
  │
Browser
  │── Receives 201 → adds ticket to UI immediately (status: TODO)

[Async — happening over next ~5–10 seconds]
Inngest Cloud
  │── Receives "ticket/created" event
  │── Calls POST /api/inngest → runs on-ticket-created

Backend (Inngest steps)
  │── Step 1 "fetch-ticket":
  │     Ticket.findById(ticketId) → { title, description }
  │── AI Analysis:
  │     POST to Gemini Flash API with prompt + ticket content
  │     Gemini returns JSON: { summary, priority, helpfulNotes, relatedSkills }
  │── Step 3 "update-ticket-with-ai-data":
  │     Ticket.findByIdAndUpdate(ticketId, {
  │         priority, helpfulNotes, relatedSkills, summary
  │     })
  │     [Ticket in DB now has AI data. If user refreshes, they see it.]
  │── Step 4 "assign-moderator":
  │     User.findOne({ role:"moderator", skills match relatedSkills })
  │     → if found: assignedTo = moderator._id, status = "IN_PROGRESS"
  │     → if not:  User.findOne({ role:"admin" }) as fallback
  │── Step 5 "send-email-notification":
  │     resend.emails.send({ to: moderator.email, ... ticket details })
  │
Resend API
  └── Delivers assignment email to moderator
```

### 8.3 Ticket Fetch Flow (Role-Aware)

```
GET /api/tickets
Authorization: Bearer <jwt>

authenticate middleware:
  jwt.verify(token) → req.user.role = "user" | "moderator" | "admin"

if role === "user":
  Ticket.find({ createdBy: req.user._id }).select("title description status createdAt")
  ← Only their own tickets, limited fields

if role === "moderator" or "admin":
  Ticket.find({})
    .populate("assignedTo", "email _id")
    .populate("createdBy",  "email _id")
  ← All tickets, full data with joined user info
```

---

## 9. Key Design Decisions — The Reasoning

### Why event-driven instead of synchronous AI calls?

If AI analysis happened synchronously inside `POST /api/tickets`:
- User waits 3–6 seconds for Gemini's response before getting confirmation.
- If Gemini is rate-limited or down, ticket creation fails entirely.
- Server holds a connection open for each in-flight AI request — doesn't scale.

With Inngest, these problems disappear. The API responds in milliseconds. Gemini failures are retried automatically. The user's experience is never degraded by AI slowness.

### Why MongoDB instead of PostgreSQL?

The ticket schema is flexible — AI fields (`relatedSkills`, `helpfulNotes`) only get added after the background job runs. A relational DB would require nullable columns or a separate table. MongoDB's document model handles this naturally — a ticket document just grows more fields as data arrives.

### Why JWT in localStorage instead of httpOnly cookies?

`localStorage` is accessible from JavaScript, making it vulnerable to XSS (Cross-Site Scripting). If an attacker injects a script into the page, they can steal the token.

`httpOnly` cookies can't be accessed from JavaScript — only the browser sends them automatically. This prevents token theft via XSS.

TicketWise uses `localStorage` for simplicity. In production, you'd switch to `httpOnly` cookies with `SameSite: Strict` and `Secure` flags.

### Why Gemini Flash instead of GPT-4?

For **structured extraction** (turning free text into JSON), you don't need a frontier reasoning model. You need:
- Consistent output format → achieved via prompt + low temperature
- Low latency → Flash is faster than GPT-4
- Low cost → Flash is cheaper per token

GPT-4 would give higher quality reasoning for complex tasks but is overkill for JSON extraction from short text.

### Why not wrap the AI call in `step.run()`?

The project uses `@inngest/agent-kit` which internally uses Inngest steps. Inngest disallows nesting `step.run()` inside another `step.run()` — it throws a `NESTING_STEPS` error. So the AI call sits between steps rather than inside one. This is a framework constraint, not a design choice.

### Why not use Redux or Zustand for state?

The app's state is simple: a list of tickets, a current user, some UI flags. React's built-in `useState` + `useEffect` + `useMemo` handles this without ceremony. Adding Redux would mean boilerplate (actions, reducers, selectors) for no real benefit at this scale.

---

*Built with React 19, Express 5, MongoDB Atlas, Inngest, Google Gemini Flash, and Resend.*
