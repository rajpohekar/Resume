# TicketWise — Access Control & Likely Interview Questions

---

## The Exact Question You Were Asked

> **"How did you ensure that a user can only see their page, and an admin can only see their page?"**

This is really asking three things at once:
1. How do you stop unauthenticated people from accessing any page?
2. How do you stop a regular user from reaching admin-only pages?
3. How do you stop a regular user from seeing other users' data — even if they guess the URL?

TicketWise answers all three, at **two separate levels: frontend and backend.**

---

## Part 1 — Frontend Access Control (React Route Guards)

The frontend uses three wrapper components in `App.jsx`. None of them render their children unless a condition is met — otherwise they redirect using React Router's `<Navigate>`.

---

### Guard 1 — `ProtectedRoute` (Are you logged in at all?)

```jsx
function ProtectedRoute({ children }) {
    const token = localStorage.getItem("token");
    if (!token) {
        return <Navigate to="/login" replace />;
    }
    return children;
}
```

**What it does:** Checks if a JWT token exists in `localStorage`. If there's no token, it redirects to `/login`. If there is a token, it renders the page.

**Which routes use it:** `/` (Tickets), `/tickets/:id`, and `/admin` — every private page.

**Interview answer for this guard:**
> "Every protected route is wrapped in a `ProtectedRoute` component. Before rendering the page, it checks `localStorage` for a JWT token. If the token isn't there, React Router redirects the user to `/login` immediately — they never see the page content at all."

---

### Guard 2 — `AdminRoute` (Are you an admin?)

```jsx
const isAdmin = () => {
    const userString = localStorage.getItem("user");
    if (!userString) return false;
    try {
        const user = JSON.parse(userString);
        return user?.role === "admin";
    } catch (error) {
        return false;
    }
};

function AdminRoute({ children }) {
    if (!isAdmin()) {
        return <Navigate to="/" replace />;
    }
    return children;
}
```

**What it does:** Reads the `user` object stored in `localStorage` after login. Parses it and checks if `role === "admin"`. If not, redirects to `/` (the tickets dashboard).

**Which route uses it:** Only `/admin`.

**Notice the layering on the admin route:**

```jsx
<Route
    path="/admin"
    element={
        <ProtectedRoute>      {/* First: are you logged in? */}
            <AdminRoute>      {/* Second: are you an admin? */}
                <Admin />
            </AdminRoute>
        </ProtectedRoute>
    }
/>
```

A user must pass **both guards** to reach the Admin page. A non-logged-in user hits `ProtectedRoute` and gets sent to `/login`. A logged-in non-admin user passes `ProtectedRoute` but fails `AdminRoute` and gets sent back to `/`.

**Interview answer for this guard:**
> "The admin route has two layers — `ProtectedRoute` checks for a token, and `AdminRoute` checks the role stored in `localStorage`. Both must pass. A regular logged-in user who tries to navigate to `/admin` manually will be silently redirected to the home page."

---

### Guard 3 — `PublicRoute` (Are you already logged in trying to go back to login?)

```jsx
function PublicRoute({ children }) {
    const token = localStorage.getItem("token");
    if (token) {
        return <Navigate to="/" replace />;
    }
    return children;
}
```

**What it does:** Prevents already-logged-in users from accessing `/login` and `/signup`. If they have a token, they're redirected to `/`.

**Interview answer:**
> "If a logged-in user tries to navigate to `/login` or `/signup`, `PublicRoute` detects the existing token in `localStorage` and redirects them to the dashboard. This prevents weird UX like being 'logged in twice'."

---

### The Wildcard Route

```jsx
<Route path="*" element={<Navigate to="/" replace />} />
```

Any unknown URL (`/random-page`, `/hack-me`, etc.) is redirected to `/`. That then hits `ProtectedRoute`, which sends non-logged-in users to `/login`. No 404 page needed.

---

## Part 2 — Backend Access Control (The Real Security Layer)

**This is the critical part.** The frontend guards are just UX. A person can open DevTools, manually set a token in `localStorage`, and navigate to `/admin` — the React guard only checks if `role === "admin"` in the stored user object, which can be tampered with client-side.

That's why **every sensitive operation is also enforced on the backend**, regardless of what the frontend sends.

---

### Step 1 — JWT Auth Middleware (Applied to every protected route)

```js
export const authenticate = (req, res, next) => {
    const token = req.headers.authorization?.split(" ")[1];

    if (!token) {
        return res.status(401).json({ error: "Access Denied. No token found." });
    }

    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        req.user = decoded;  // { _id, role } — extracted from the signed token
        next();
    } catch (error) {
        res.status(401).json({ error: "Invalid token" });
    }
};
```

**What it does:**
1. Extracts the JWT from the `Authorization: Bearer <token>` header.
2. Calls `jwt.verify()` with the server-side secret key.
3. If valid: attaches the decoded payload `{ _id, role }` to `req.user` and calls `next()`.
4. If invalid or missing: returns 401. The request never reaches the controller.

**Key point:** The `role` in `req.user` comes from the **signed JWT**, not from what the frontend sends in the request body. A user cannot forge their role by modifying `localStorage` — the backend derives role from the token's cryptographic signature.

---

### Step 2 — Role-Based Data Access in Controllers

Even after passing the auth middleware, the controller decides **what data to return** based on `req.user.role`.

#### In `getTickets` controller:

```js
const user = req.user;

if (user.role !== "user") {
    // Moderator or Admin: sees ALL tickets, with populated references
    tickets = await Ticket.find({})
        .populate("assignedTo", ["email", "_id"])
        .populate("createdBy", ["email", "_id"])
        .sort({ createdAt: -1 });
} else {
    // Regular user: sees ONLY their own tickets, limited fields
    tickets = await Ticket.find({ createdBy: user._id })
        .select("title description status createdAt")
        .sort({ createdAt: -1 });
}
```

**What this means:**
- A regular user hitting `GET /api/tickets` only gets tickets where `createdBy === their own _id`. Even if they send a valid token, they cannot see another user's tickets.
- A moderator or admin gets all tickets with full populated data (who created it, who it's assigned to).

#### In `getTicket` controller (single ticket):

```js
if (user.role !== "user") {
    ticket = await Ticket.findById(req.params.id)  // Any ticket by ID
        .populate("assignedTo", ["email", "_id"])
        .populate("createdBy", ["email", "_id"]);
} else {
    ticket = await Ticket.findOne({
        createdBy: user._id,    // Must belong to the requesting user
        _id: req.params.id,     // AND match the ID in the URL
    });
}
```

**What this means:** If a regular user tries to access `/api/tickets/someOtherUsersTicketId`, the query finds nothing (because `createdBy` won't match their `_id`), and the backend returns 404. The user cannot even discover that the ticket exists.

#### In `getUsers` and `updateUser` controllers:

```js
if (req.user?.role !== "admin") {
    return res.status(403).json({ error: "Forbidden" });
}
```

Hardcoded admin-only check. Even a moderator with a valid token gets 403 if they try to list or update users.

---

## The Two-Layer Security Summary (Say This in the Interview)

| What | Frontend (UX layer) | Backend (Security layer) |
|---|---|---|
| Not logged in | `ProtectedRoute` → redirect to `/login` | Auth middleware → 401 |
| Logged-in non-admin on `/admin` | `AdminRoute` → redirect to `/` | `req.user.role !== "admin"` → 403 |
| User trying to see other user's ticket | No frontend guard (can try the API directly) | `Ticket.findOne({ createdBy: user._id, _id })` → 404 |
| Data scope per role | UI only shows relevant tickets | DB query filtered by `createdBy: user._id` |

**One-line answer for the interview:**
> "Access control is enforced at two levels. On the frontend, React Router wrapper components check `localStorage` for a token and role before rendering any page. On the backend, a JWT middleware verifies every request cryptographically, and controllers use `req.user.role` — derived from the signed token, not from anything the client sends — to filter both what data is returned and what operations are allowed."

---

## Part 3 — Other Questions That Will Definitely Be Asked

These are the most common follow-ups based on your project's specific implementation.

---

### Q1: Why store the JWT in `localStorage`? Isn't that insecure?

**The honest answer (show you know the tradeoff):**
> "Yes, `localStorage` is vulnerable to XSS (Cross-Site Scripting) attacks — malicious JavaScript on the page can read it. The more secure approach is to store the JWT in an `httpOnly` cookie, which JavaScript cannot access at all. I used `localStorage` here for simplicity, but in production I would switch to `httpOnly` cookies and add CSRF protection."

---

### Q2: What's inside the JWT? Who puts the role in it?

**Answer:**
> "When a user logs in or signs up, the backend calls `jwt.sign({ _id: user._id, role: user.role }, JWT_SECRET)`. So the JWT payload contains the user's database ID and their role. When the auth middleware decodes the token on any protected request, it gets the role directly from this cryptographically signed payload — not from anything the frontend sends. The client can't tamper with it without invalidating the signature."

---

### Q3: What if someone deletes their token from `localStorage` but is still logged in on the backend?

**Answer:**
> "JWTs are stateless — the backend doesn't keep a session store. If the token is deleted from `localStorage`, the frontend will redirect the user to login (because `ProtectedRoute` won't find a token), but the token itself is technically still valid until it expires. Our tokens have no expiry set (`jwt.sign` is called without `expiresIn`), which is actually a weakness. In production I'd set an expiry like `{ expiresIn: '7d' }` and implement token refresh."

---

### Q4: What happens if a regular user manually calls `GET /api/tickets/someOtherTicketId` using Postman or curl with their own valid token?

**Answer:**
> "They'd get a 404. The backend query for a regular user is `Ticket.findOne({ createdBy: user._id, _id: req.params.id })`. Both conditions must match. Since the ticket belongs to a different user, `createdBy` won't match the requesting user's `_id`, so Mongoose returns null and the controller sends a 404. The user can't even confirm the ticket exists."

---

### Q5: What is the difference between 401 and 403 in your app?

**Answer:**
> "401 Unauthorized means 'I don't know who you are' — no token or invalid token. 403 Forbidden means 'I know who you are, but you're not allowed to do this' — valid token, but wrong role. In the app, the auth middleware throws 401. Controllers like `getUsers` and `updateUser` throw 403 when a non-admin tries to access admin functionality."

---

### Q6: How does the moderator skill-matching work? Could a wrong person be assigned?

**Answer:**
> "The AI returns an array of related skills like `['React', 'OAuth']`. We then query MongoDB for a moderator whose `skills` array contains at least one match using `$elemMatch` with a case-insensitive regex. If no moderator matches, we fall back to any admin. If there's no admin, the ticket stays unassigned with status 'TODO'. So the worst case is the ticket isn't assigned automatically — it never goes to the wrong person."

---

### Q7: Why do regular users get fewer fields when fetching their own tickets?

**Answer:**
> "We use Mongoose's `.select()` to limit which fields are returned. For regular users we select only `title, description, status, createdAt`. Admin-specific fields like `helpfulNotes`, `relatedSkills`, `assignedTo`, and the AI summary are excluded. This is defense in depth — even if something went wrong with the role check, users wouldn't receive internal operational data that's meant only for moderators."

---

### Q8: What happens if someone sends a request with a forged `role: "admin"` in the request body?

**Answer:**
> "Nothing happens — the backend completely ignores any `role` in the request body for authorization decisions. The role is read only from `req.user`, which is set by the `authenticate` middleware by decoding the JWT. The JWT is signed with a secret key only the server knows, so a forged role in the body has no effect."

---

### Q9: How does logout work? Is the token invalidated?

**Answer:**
> "Logout is client-side only — the frontend removes the token from `localStorage`. On the backend, the logout endpoint doesn't actually invalidate the token because JWTs are stateless and we don't have a token blacklist. This is a known limitation. The token technically remains valid until it would expire (which in our case it never does, since we didn't set `expiresIn`). In production, you'd either use short-lived tokens with refresh tokens, or maintain a server-side token blacklist in Redis."

---

### Q10: What's the difference between `moderator` and `admin` in your system?

**Answer:**
> "In the data access layer, both moderator and admin see all tickets with full data — the check is `user.role !== 'user'`. The difference is in user management: only an admin can call `GET /api/auth/users` to list all users, and only an admin can call `POST /api/auth/update-user` to change another user's role and skills. The Admin page on the frontend is also gated behind `AdminRoute` which checks for `role === 'admin'` specifically. Moderators are essentially read-only power users who receive ticket assignments."

---

### Q11: What if two moderators have the same skills? Who gets the ticket?

**Answer:**
> "MongoDB's `findOne()` returns the first matching document based on the natural order in the collection (insertion order by default). There's no round-robin or load balancing logic — the first moderator with a matching skill in the database gets assigned. This is a simplification. In a production system you'd want to track current workload and distribute based on who has the fewest open tickets."

---

### Q12: Why is there no `expiresIn` on the JWT?

**Answer:**
> "That's actually a bug/oversight in the current implementation. `jwt.sign({ _id, role }, JWT_SECRET)` without an `expiresIn` option creates a token that never expires. This means a stolen token stays valid forever. The fix is to add `{ expiresIn: '7d' }` and implement a token refresh mechanism so users aren't forced to log in every 7 days."

---

## Quick Reference — Access Control Decision Tree

```
Request arrives at backend
    │
    ├─ No token / invalid token → 401 Unauthorized
    │
    └─ Valid token → req.user = { _id, role }
           │
           ├─ GET /api/tickets/
           │       ├─ role === "user" → find({ createdBy: user._id }).select(limited fields)
           │       └─ role !== "user" → find({}).populate(all fields)
           │
           ├─ GET /api/tickets/:id
           │       ├─ role === "user" → findOne({ createdBy: user._id, _id: params.id })
           │       └─ role !== "user" → findById(params.id)
           │
           ├─ GET /api/auth/users
           │       ├─ role === "admin" → return all users (no passwords)
           │       └─ role !== "admin" → 403 Forbidden
           │
           └─ POST /api/auth/update-user
                   ├─ role === "admin" → update user
                   └─ role !== "admin" → 403 Forbidden
```

---

*Print this, read it twice, and you'll own any access control question they throw at you.*
