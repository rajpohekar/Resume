# TicketWise — Complete Architecture Interview Guide

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [High-Level Architecture](#2-high-level-architecture)
3. [Layer 1 — Frontend (React + Vite)](#3-layer-1--frontend-react--vite)
4. [Layer 2 — Backend (Node.js + Express)](#4-layer-2--backend-nodejs--express)
5. [Layer 3 — Event Queue (Inngest)](#5-layer-3--event-queue-inngest)
6. [Layer 4 — External Services](#6-layer-4--external-services)
7. [Data Flow — End to End](#7-data-flow--end-to-end)
8. [Database Design](#8-database-design)
9. [Authentication & Authorization](#9-authentication--authorization)
10. [AI Integration](#10-ai-integration)
11. [Key Design Decisions](#11-key-design-decisions)
12. [Interview Q&A Cheat Sheet](#12-interview-qa-cheat-sheet)

---

## 1. Project Overview

**TicketWise** is an AI-powered support ticket management system. When a user submits a support ticket, an AI automatically analyzes it, assigns a priority, identifies the skills required to solve it, writes helpful notes for the handler, and automatically assigns the right moderator — all in the background without blocking the user.

**Tech Stack at a glance:**

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite, TailwindCSS, DaisyUI, Framer Motion |
| Backend | Node.js, Express 5, JWT, bcryptjs |
| Database | MongoDB (Mongoose ODM) |
| Event Queue | Inngest (background job orchestration) |
| AI | Google Gemini Flash API |
| Email | Resend API |

---

## 2. High-Level Architecture

```
┌──────────────────────────────────────────┐
│           FRONTEND (React + Vite)        │
│  Login | Signup | Tickets | Admin pages  │
│  Route guards (JWT from localStorage)    │
└────────────────┬─────────────────────────┘
                 │ REST API (Bearer token)
┌────────────────▼─────────────────────────┐
│         BACKEND (Express 5)              │
│  /api/auth  |  /api/tickets              │
│  Auth middleware → Controllers           │
│  Mongoose models → MongoDB Atlas         │
│  /api/inngest (event serving endpoint)   │
└────────────────┬─────────────────────────┘
                 │ inngest.send(event)
┌────────────────▼─────────────────────────┐
│           INNGEST (Event Queue)          │
│  on-user-signup function                 │
│  on-ticket-created function              │
│  Step functions with automatic retries   │
└────────────┬──────────────┬──────────────┘
             │              │
    ┌────────▼───┐  ┌───────▼──────────────┐
    │   Resend   │  │  Google Gemini Flash  │
    │ (Email API)│  │  (AI ticket analysis) │
    └────────────┘  └──────────────────────┘
```

The architecture follows a **decoupled event-driven pattern** — the API responds immediately to the user while all heavy processing (AI, email, assignment) happens asynchronously in the background.

---

## 3. Layer 1 — Frontend (React + Vite)

### 3.1 Pages

The frontend is a React Single Page Application (SPA) with five main pages:

| Page | Route | Purpose |
|---|---|---|
| Login | `/login` | User authentication |
| Signup | `/signup` | New user registration |
| Tickets | `/` | Dashboard — list all tickets, create new ones |
| Ticket Detail | `/tickets/:id` | View a single ticket's full details |
| Admin | `/admin` | Admin-only panel |

### 3.2 Route Guards

Three types of route protection are implemented using React Router:

**ProtectedRoute** — Checks if a JWT token exists in `localStorage`. If not, redirects to `/login`.

```jsx
function ProtectedRoute({ children }) {
    const token = localStorage.getItem("token");
    if (!token) return <Navigate to="/login" replace />;
    return children;
}
```

**PublicRoute** — If a user is already logged in (token exists), they cannot access login/signup — they get redirected to `/`.

**AdminRoute** — Reads the `user` object from `localStorage`, checks if `role === "admin"`. If not admin, redirects to `/`.

> **Interview tip:** The role check on the frontend is just UI-level protection. The backend also enforces role-based access. Frontend guards are for UX; backend guards are for real security.

### 3.3 State Management

No Redux or external state library is used. State is managed with React's built-in `useState` and `useEffect` hooks within each page component. The auth token and user object are stored in `localStorage` and read directly.

### 3.4 API Communication

All API calls use the native `fetch()` API with the `Authorization: Bearer <token>` header. The backend URL is read from the Vite environment variable `VITE_SERVER_URL`.

```js
const res = await fetch(`${import.meta.env.VITE_SERVER_URL}/tickets`, {
    method: "GET",
    headers: { Authorization: `Bearer ${token}` },
});
```

### 3.5 Ticket Dashboard Features

- **Stats panel:** Shows total, active (IN_PROGRESS), and resolved (DONE) ticket counts — computed client-side with `useMemo`.
- **Search:** Filters tickets by title + description using a `useMemo`-derived list.
- **Status filter:** Dropdown to filter by TODO / IN_PROGRESS / DONE.
- **Animated UI:** Framer Motion handles card entrance animations and form expand/collapse transitions.

---

## 4. Layer 2 — Backend (Node.js + Express)

### 4.1 Entry Point (`index.js`)

The server does four things on startup:

1. Sets up Express with CORS and JSON body parsing.
2. Registers three route groups: `/api/auth`, `/api/tickets`, `/api/inngest`.
3. Connects to MongoDB via Mongoose.
4. Starts listening on port 3000 (or `process.env.PORT`).

```js
app.use("/api/auth",    userRoutes);
app.use("/api/tickets", ticketRoutes);
app.use("/api/inngest", serve({ client: inngest, functions: [...] }));
```

The `/api/inngest` endpoint is not a normal REST endpoint — it's a **webhook handler** that Inngest Cloud calls to execute background functions on your server.

### 4.2 Auth Routes (`/api/auth`)

| Method | Route | Protected | Purpose |
|---|---|---|---|
| POST | `/api/auth/signup` | No | Register a new user |
| POST | `/api/auth/login` | No | Login, receive JWT |
| POST | `/api/auth/logout` | No | Client-side logout |
| GET | `/api/auth/users` | Yes | Get all users (admin) |
| POST | `/api/auth/update-user` | Yes | Update user role/skills |

### 4.3 Ticket Routes (`/api/tickets`)

| Method | Route | Protected | Purpose |
|---|---|---|---|
| POST | `/api/tickets/` | Yes | Create a new ticket |
| GET | `/api/tickets/` | Yes | Get all tickets (role-filtered) |
| GET | `/api/tickets/:id` | Yes | Get a single ticket |

### 4.4 Auth Middleware

Every protected route passes through this middleware before reaching the controller:

```js
export const authenticate = (req, res, next) => {
    const token = req.headers.authorization?.split(" ")[1]; // "Bearer <token>"
    if (!token) return res.status(401).json({ error: "Access Denied." });

    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded; // user payload now available in controllers
    next();
};
```

The decoded JWT payload (which includes `_id`, `email`, `role`) is attached to `req.user` so controllers know who is making the request.

### 4.5 Controllers

**User controller** handles:
- Password hashing with `bcryptjs` on signup.
- JWT generation with `jsonwebtoken` on login.
- Firing the `user/signup` Inngest event after successful registration.

**Ticket controller** handles:
- Creating a ticket document in MongoDB.
- Immediately firing the `ticket/created` Inngest event with the ticket ID.
- **Role-based data fetching:** admins/moderators get full ticket data with populated fields; regular users only see their own tickets with limited fields.

```js
// Role-based query logic
if (user.role !== "user") {
    tickets = await Ticket.find({}).populate("assignedTo").populate("createdBy");
} else {
    tickets = await Ticket.find({ createdBy: user._id }).select("title description status createdAt");
}
```

---

## 5. Layer 3 — Event Queue (Inngest)

### 5.1 What is Inngest?

Inngest is a **background job and event queue platform**. Instead of processing heavy tasks synchronously inside a request/response cycle, you fire an event and Inngest handles it reliably in the background — with automatic retries, step isolation, and observability.

Think of it like this:
- Traditional approach: User creates ticket → server does AI analysis → server assigns moderator → server sends email → then responds to user. The user waits 5+ seconds.
- TicketWise approach: User creates ticket → server saves to DB → fires event → **immediately responds to user**. Inngest handles everything else in the background.

### 5.2 Inngest Client

```js
// inngest/client.js
import { Inngest } from "inngest";
export const inngest = new Inngest({ id: "ai-ticket-assistant" });
```

This client is used both to **send events** (from controllers) and to **register functions** (via the `/api/inngest` endpoint).

### 5.3 Function 1 — `on-user-signup`

**Trigger:** `user/signup` event (fired after signup)

**Steps:**
1. `get-user-email` — Looks up the user in MongoDB by email (case-insensitive regex).
2. `send-welcome-email` — Calls Resend to send a welcome email.

**Retries:** 2 (configured on the function). If a step fails, Inngest retries only that step, not the whole function.

```js
inngest.createFunction(
    { id: "on-user-signup", retries: 2, triggers: [{ event: "user/signup" }] },
    async ({ event, step }) => {
        const user = await step.run("get-user-email", async () => { ... });
        await step.run("send-welcome-email", async () => { ... });
    }
);
```

### 5.4 Function 2 — `on-ticket-created`

**Trigger:** `ticket/created` event (fired after a ticket is saved)

**Steps (sequential — each step's output flows into the next):**

| Step | What it does |
|---|---|
| `fetch-ticket` | Fetches the full ticket document from MongoDB using the ticketId from the event |
| AI Analysis | Calls Google Gemini Flash with the ticket title + description |
| `update-ticket-with-ai-data` | Updates the ticket with: `priority`, `helpfulNotes`, `relatedSkills`, `summary`, status = "TODO" |
| `assign-moderator` | Queries MongoDB for a moderator whose skills match the ticket's `relatedSkills` (case-insensitive regex). Falls back to any admin if no match found. Updates ticket with `assignedTo` and status = "IN_PROGRESS" |
| `send-email-notification` | Sends the assigned moderator an email via Resend with ticket details |

> **Interview tip:** Inngest steps are isolated. If step 4 (assign-moderator) fails, Inngest retries only step 4, not the AI call (step 2). This makes the pipeline resilient and cost-efficient.

### 5.5 Why Inngest over a simple `setTimeout` or queue?

- **Durability:** Events survive server restarts.
- **Retries:** Failed steps retry automatically with backoff.
- **Observability:** Inngest dashboard shows every function run, step output, and error.
- **Step isolation:** Each `step.run(...)` is checkpointed — no double-processing on retry.

---

## 6. Layer 4 — External Services

### 6.1 Resend (Email API)

Resend is a modern transactional email API used instead of SMTP/Nodemailer (which is also installed but not used for delivery).

```js
const resend = new Resend(process.env.RESEND_API_KEY);
await resend.emails.send({
    from: 'TicketWise <onboarding@resend.dev>',
    to: [recipientEmail],
    subject: subject,
    text: body,
});
```

Two emails are sent in the system:
1. **Welcome email** — triggered by `on-user-signup`
2. **Assignment notification** — triggered by `on-ticket-created` after moderator is assigned

### 6.2 Google Gemini Flash API

Gemini is called directly via the REST API (not an SDK) inside `utils/ai.js`.

**What it receives:** A carefully crafted prompt containing the ticket's title and description.

**What it returns:** A strict JSON object:
```json
{
    "summary": "Short summary of the issue",
    "priority": "high",
    "helpfulNotes": "Check the OAuth token expiry config...",
    "relatedSkills": ["OAuth", "Node.js", "JWT"]
}
```

**How it's made reliable:**
- The prompt explicitly says "respond ONLY with valid JSON, no markdown fences"
- `temperature: 0.1` is set for deterministic, structured output
- The code strips any accidental markdown code fences before `JSON.parse()`
- Invalid responses fall back to safe defaults (priority = "medium", empty skills)

---

## 7. Data Flow — End to End

### 7.1 User Signup Flow

```
User fills Signup form
    → POST /api/auth/signup
    → bcrypt.hash(password)
    → User.create({email, hashedPassword, role: "user"})
    → inngest.send({ name: "user/signup", data: { email } })
    → Responds 201 to frontend immediately
    [Background]
    → Inngest: on-user-signup fires
    → Fetch user from DB
    → Resend: send welcome email
```

### 7.2 Ticket Creation Flow

```
User submits ticket form (title + description)
    → POST /api/tickets/
    → authenticate middleware verifies JWT
    → Ticket.create({ title, description, createdBy: req.user._id })
    → inngest.send({ name: "ticket/created", data: { ticketId } })
    → Responds 201 to frontend immediately (ticket shown in UI)
    [Background — Inngest orchestrates]
    → Step 1: Fetch ticket from MongoDB
    → Step 2: Call Gemini Flash API with title + description
    → Step 3: Update ticket with priority, skills, notes, summary
    → Step 4: Find moderator with matching skills (or fallback admin)
             → Update ticket: assignedTo + status = "IN_PROGRESS"
    → Step 5: Send email to assigned moderator via Resend
```

### 7.3 Ticket Fetch Flow (role-aware)

```
GET /api/tickets/
    → authenticate middleware
    → if role === "user"     → return only own tickets (limited fields)
    → if role !== "user"    → return ALL tickets (full data, populated refs)
```

---

## 8. Database Design

### 8.1 User Schema

```js
{
    email:     String (unique, required),
    password:  String (bcrypt hashed),
    role:      String (enum: "user" | "moderator" | "admin", default: "user"),
    skills:    [String],   // e.g. ["React", "Node.js", "AWS"]
    createdAt: Date
}
```

The `skills` array on a moderator is what the AI's `relatedSkills` is matched against during auto-assignment.

### 8.2 Ticket Schema

```js
{
    title:        String,
    description:  String,
    status:       String (default: "TODO"),  // "TODO" | "IN_PROGRESS" | "DONE"
    priority:     String,                    // "low" | "medium" | "high" — set by AI
    createdBy:    ObjectId → User,
    assignedTo:   ObjectId → User (null initially),
    helpfulNotes: String,                    // set by AI
    relatedSkills:[String],                  // set by AI
    deadline:     Date,
    createdAt:    Date
}
```

### 8.3 Relationship

```
User (1) ──────────── (many) Ticket [createdBy]
User (1) ──────────── (many) Ticket [assignedTo]
```

Both `createdBy` and `assignedTo` use Mongoose `populate()` to join user data (email, id) when fetched by admins/moderators.

---

## 9. Authentication & Authorization

### 9.1 Authentication (Who are you?)

1. User logs in with email + password.
2. Backend verifies password with `bcrypt.compare()`.
3. Backend signs a JWT: `jwt.sign({ _id, email, role }, JWT_SECRET)`.
4. Frontend stores the JWT in `localStorage`.
5. Every subsequent API request sends it as `Authorization: Bearer <token>`.
6. The `authenticate` middleware verifies and decodes it on every protected route.

### 9.2 Authorization (What can you do?)

| Role | Can do |
|---|---|
| `user` | Create tickets, view only their own tickets |
| `moderator` | View all tickets (full data), get assigned tickets |
| `admin` | Everything a moderator can + access `/admin` page |

Authorization is enforced **in two places:**
- **Frontend:** AdminRoute / ProtectedRoute components redirect unauthorized users (UX layer).
- **Backend:** Controllers check `req.user.role` before deciding what data to return (security layer).

---

## 10. AI Integration

### 10.1 The Prompt Strategy

The prompt is engineered to force structured JSON output:

```
You are an expert AI assistant that processes technical support tickets.
You MUST respond with ONLY a valid JSON object.
Do NOT include markdown, backticks, or any text outside the JSON.

{
  "summary": "...",
  "priority": "low|medium|high",
  "helpfulNotes": "...",
  "relatedSkills": ["..."]
}

Ticket Title: {title}
Ticket Description: {description}
```

Key engineering choices:
- `temperature: 0.1` — near-deterministic output, avoids creative hallucination
- Explicit JSON schema in the prompt
- Post-processing strips any accidental markdown fences
- Validation before use: if `priority` is not one of `["low","medium","high"]`, defaults to "medium"

### 10.2 Moderator Matching

After AI returns `relatedSkills: ["React", "OAuth"]`, the system queries:

```js
await User.findOne({
    role: "moderator",
    skills: {
        $elemMatch: {
            $regex: "React|OAuth",
            $options: "i"  // case-insensitive
        }
    }
});
```

If no moderator matches → finds any admin as fallback.
If no admin exists → ticket remains unassigned (status stays "TODO").

---

## 11. Key Design Decisions

### Why Inngest instead of a simple async function?

A plain `async` function in a request handler would fail silently if the server restarts mid-execution. Inngest persists the event, checkpoints each step, and retries failures automatically. This makes the pipeline production-grade.

### Why separate the frontend and backend into two directories?

The `ai-ticket-frontend/` and `ai-ticket-assistant/` are completely independent applications. The backend is a pure REST API — it could serve mobile apps or other clients without any changes. This is a clean separation of concerns.

### Why store JWT in localStorage instead of httpOnly cookies?

`localStorage` is simpler to implement but is vulnerable to XSS attacks. The production-grade alternative would be httpOnly cookies. This is a good **interview talking point** — you can acknowledge the tradeoff and mention the secure alternative.

### Why Gemini Flash and not GPT-4?

Gemini Flash is fast and cheap for structured extraction tasks. The ticket analysis doesn't need creative reasoning — it needs consistent JSON output. Low temperature + structured prompt = reliable results regardless of model.

### Why is the AI call NOT wrapped in `step.run()`?

The code comments this directly — `@inngest/agent-kit` uses Inngest steps internally, and wrapping it in another `step.run()` causes a `NESTING_STEPS` error. This is a real framework constraint worth knowing.

---

## 12. Interview Q&A Cheat Sheet

**Q: Explain your project in one sentence.**
> TicketWise is a full-stack AI-powered ticket management system where tickets are automatically triaged, prioritized, and assigned to the right moderator using Google Gemini, with all background processing handled asynchronously via Inngest.

**Q: How does the AI assignment work?**
> When a ticket is created, an Inngest event fires. In the background, Gemini Flash analyzes the ticket and returns a JSON with priority, related skills, and helpful notes. We then query MongoDB for a moderator whose skill set matches the AI-identified skills using a regex query. If no match is found, it falls back to an admin.

**Q: What happens if the AI call fails?**
> Inngest retries the entire function up to 2 times. If the AI still fails, we catch the error and set safe defaults — priority becomes "medium" and skills become an empty array. The ticket is still saved and can be manually assigned.

**Q: How is the app secured?**
> Authentication uses JWTs signed with a secret key. Every protected route passes through an auth middleware that verifies the token. Authorization is role-based — users only see their own tickets, while moderators and admins see everything. Role checks happen on the backend, not just the frontend.

**Q: Why did you use Inngest instead of just doing the AI call inside the route handler?**
> Doing it synchronously would make the user wait several seconds for Gemini to respond. With Inngest, the API responds immediately after saving the ticket, and all the heavy lifting — AI analysis, moderator assignment, email sending — happens in the background with full retry support and observability.

**Q: What would you improve if you had more time?**
> I'd move JWT storage from localStorage to httpOnly cookies to prevent XSS attacks. I'd add WebSocket notifications so the ticket status updates in real time on the frontend. I'd also add an admin UI to update ticket status to DONE, and add pagination for the ticket list.

**Q: How does role-based access work in your React frontend?**
> I have three route wrapper components — ProtectedRoute (checks for JWT token), PublicRoute (redirects logged-in users away from login/signup), and AdminRoute (reads the user object from localStorage and checks if role is admin). These are UI-level guards — the backend independently enforces roles on every API call.

**Q: Describe the data models.**
> There are two Mongoose models. User has email, hashed password, role (user/moderator/admin), and a skills array. Ticket has title, description, status, priority, createdBy and assignedTo (both ObjectId references to User), plus AI-generated fields: helpfulNotes, relatedSkills, and summary. The relationship is that a User can create many tickets and be assigned many tickets.

---

*Good luck with your interview! You built something genuinely impressive here.*
