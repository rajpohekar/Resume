# ⚛️ THE COMPLETE REACT ENGINEERING HANDBOOK
### From Zero to Senior Engineer — Learning Guide + Interview Prep

> **How to use this handbook:** Read it top-to-bottom the first time. Then use section headers as a reference. Before interviews, jump to the Quick Revision Sheet (Section 25).

---

# TABLE OF CONTENTS

1. What is React?
2. How React Works Internally
3. Setting Up React
4. JSX In Depth
5. Components In Depth
6. Props In Depth
7. State In React
8. Event Handling
9. Conditional Rendering
10. Lists & Keys
11. Forms In React
12. Hooks — Complete Deep Dive
13. Component Lifecycle
14. React Router
15. API Calling & Data Fetching
16. State Management
17. Performance Optimization
18. React Internals & Advanced Topics
19. Error Handling
20. Folder Structure & Clean Architecture
21. Common React Interview Questions
22. Common React Pitfalls
23. Project-Based React Questions
24. Build a Complete Mental Model of React
25. 5-Minute Rapid Revision Sheet

---

# 1. WHAT IS REACT?

## 1.1 The Core Idea

Imagine you're building a news feed. Every second, new articles appear, likes update, comments stream in. The entire page is *alive*. 

Before React, you'd write code like:

```javascript
// Vanilla JS — manipulate the DOM by hand
document.getElementById('likeCount').innerText = newCount;
document.querySelector('.comment-list').appendChild(newNode);
```

This works — but it becomes **spaghetti code** at scale. You track every element manually, mutations happen everywhere, and bugs are hard to trace.

**React's answer:** Describe *what* the UI should look like for a given state. React figures out *how* to make it so.

```jsx
// React — describe the desired state
function LikeButton({ count }) {
  return <button>❤️ {count} Likes</button>;
}
```

When `count` changes, React automatically updates the DOM. You never touch the DOM directly.

---

## 1.2 A Brief History

| Year | Event |
|------|-------|
| 2011 | Jordan Walke (Facebook) builds FaxJS — internal prototype |
| 2013 | React open-sourced at JSConf US |
| 2015 | React Native released — write mobile apps in React |
| 2016 | React Fiber rewrite begins (async rendering) |
| 2017 | React 16 ships — Fiber becomes the core |
| 2019 | Hooks introduced in React 16.8 — massive paradigm shift |
| 2022 | React 18 — Concurrent Features, `useTransition`, Suspense |
| 2024+ | React Server Components mature, new compiler lands |

---

## 1.3 Problems React Solves

### Problem 1: Manual DOM Management is Painful

```
Traditional Web Dev:
  Event fires → Find DOM node → Mutate it → Hope nothing breaks

React:
  State changes → React diffs → React patches only what changed
```

### Problem 2: State and UI Get Out of Sync

In vanilla JS, the data in memory and what's on screen can diverge. React's core guarantee: **the UI is always a function of state.**

```
UI = f(state)
```

### Problem 3: Reusability is Hard

Vanilla JS widgets are hard to isolate and reuse. React components are self-contained units with their own state, logic, and markup.

---

## 1.4 Single Page Application (SPA) Concept

**Traditional Multi-Page App (MPA):**
```
User clicks link
  → Browser requests new HTML from server
    → Server sends entire new HTML page
      → Browser re-renders everything
        → Feels slow, full page flash
```

**Single Page Application (SPA):**
```
User clicks link
  → JavaScript intercepts the navigation
    → JS fetches only the data needed (JSON)
      → React updates only the parts that changed
        → Feels instant, no page flash
```

React is the UI engine for SPAs. The HTML page loads once; everything else is dynamic.

---

## 1.5 Component-Based Architecture

React decomposes UIs into a **tree of components**. Each component:
- Has its own logic
- Has its own state (optionally)
- Renders a piece of UI
- Can be reused anywhere

```
App
├── Header
│   ├── Logo
│   └── NavBar
│       ├── NavItem ("Home")
│       ├── NavItem ("About")
│       └── NavItem ("Contact")
├── MainContent
│   ├── Sidebar
│   └── ArticleList
│       ├── Article
│       ├── Article
│       └── Article
└── Footer
```

Each box is an independent, testable, reusable unit.

**Real-world analogy:** Like LEGO bricks. Each brick is a component. You assemble them to build anything, and a red brick works the same whether it's in a house or a car.

---

## 1.6 Declarative UI

**Imperative** (vanilla JS): Tell the browser *how* to do it step by step.

```javascript
// Imperative — tell DOM what to do manually
const list = document.createElement('ul');
items.forEach(item => {
  const li = document.createElement('li');
  li.textContent = item.name;
  list.appendChild(li);
});
document.body.appendChild(list);
```

**Declarative** (React): Describe *what* you want; React handles the how.

```jsx
// Declarative — describe the desired output
function ItemList({ items }) {
  return (
    <ul>
      {items.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}
```

Same result, but the React version is shorter, readable, and React automatically handles DOM updates when `items` changes.

---

## 1.7 Virtual DOM Overview

React keeps an in-memory copy of the real DOM called the **Virtual DOM**. When state changes:

```
1. React creates a new Virtual DOM tree
2. React compares it to the previous Virtual DOM (diffing)
3. React calculates the minimum changes needed
4. React applies ONLY those changes to the real DOM
```

This is called **reconciliation**. It's fast because operating on in-memory JavaScript objects is ~100x faster than touching the real DOM.

> **We'll go very deep on this in Section 2.**

---

## 1.8 React vs Vanilla JS

| Aspect | Vanilla JS | React |
|--------|-----------|-------|
| DOM manipulation | Manual | Automatic |
| State management | You manage | useState / stores |
| Component reuse | Hard | Built-in |
| Performance | Can be fast if careful | Optimized by default |
| Team scalability | Gets messy | Structured |
| Learning curve | Low | Medium |

---

## 1.9 React vs Angular vs Vue

| Feature | React | Angular | Vue |
|---------|-------|---------|-----|
| Type | Library (UI only) | Full framework | Progressive framework |
| Language | JavaScript/JSX | TypeScript (required) | JavaScript/TypeScript |
| Data binding | One-way | Two-way | Two-way (v-model) |
| Learning curve | Medium | High | Low |
| Bundle size | Small | Large | Small |
| Maintained by | Meta | Google | Community |
| Use case | Flexible, anything | Enterprise apps | Beginner-friendly |

**Analogy:**
- React = engine (you pick the chassis, wheels, paint)
- Angular = full car (everything decided for you)
- Vue = car with good defaults, but customizable

---

## 1.10 React Ecosystem Overview

React itself only handles the UI layer. The ecosystem fills the rest:

```
React (UI)
├── Routing         → React Router, TanStack Router
├── State Mgmt      → Redux Toolkit, Zustand, Jotai, Recoil
├── Data Fetching   → React Query, SWR, Apollo (GraphQL)
├── Forms           → React Hook Form, Formik
├── Styling         → Tailwind, CSS Modules, Styled Components
├── Testing         → Jest, React Testing Library, Vitest
├── SSR/SSG         → Next.js, Remix
├── Mobile          → React Native
└── Animation       → Framer Motion, React Spring
```

---

### 📝 QUICK REVISION — Section 1

- React = UI library that keeps UI in sync with state
- `UI = f(state)` — the core mental model
- Components = reusable, self-contained UI units
- Declarative = describe WHAT, not HOW
- Virtual DOM = in-memory DOM copy for fast diffing
- SPA = one HTML page, JS handles all navigation

---

# 2. HOW REACT WORKS INTERNALLY

> This section is critical for senior interviews. Understanding the internals separates good developers from great ones.

## 2.1 The Real DOM Problem

The **Real DOM** (Document Object Model) is the browser's representation of your HTML. Every DOM operation is expensive:

- Accessing a DOM node triggers layout calculations
- Modifying a DOM node can cause a **reflow** (recalculate layout) or **repaint** (redraw pixels)
- With hundreds of dynamic elements, this becomes a bottleneck

```
document.getElementById('item').style.color = 'red';
// ↑ This triggers:
//   1. DOM lookup (traverse tree)
//   2. Style recalculation
//   3. Possible reflow
//   4. Repaint
```

---

## 2.2 The Virtual DOM

The Virtual DOM is a **plain JavaScript object** that mirrors the real DOM structure.

```javascript
// Real DOM:
// <div class="box">
//   <h1>Hello</h1>
//   <p>World</p>
// </div>

// Virtual DOM equivalent:
{
  type: 'div',
  props: { className: 'box' },
  children: [
    { type: 'h1', props: {}, children: ['Hello'] },
    { type: 'p', props: {}, children: ['World'] }
  ]
}
```

Manipulating this JS object is lightning fast. No reflows, no repaints — it's just memory operations.

---

## 2.3 Reconciliation — The Diffing Algorithm

When state/props change, React:

```
Step 1: Re-render component → produces NEW virtual DOM tree
Step 2: Compare new vDOM vs old vDOM ("diffing")
Step 3: Find the minimal set of changes
Step 4: Apply ONLY those changes to the real DOM
```

### The Diffing Rules

React's diffing algorithm runs in **O(n)** time (not O(n³) like naive tree diffing) by using two heuristics:

**Heuristic 1: Elements of different types produce different trees.**

```jsx
// Old tree:           // New tree:
<div>                  <span>
  <Counter />            <Counter />
</div>                 </span>

// React sees div→span: DESTROY the entire subtree, build fresh
// Counter state is LOST
```

**Heuristic 2: Keys identify which list items changed.**

```jsx
// Without keys: React assumes position-based matching
// [A, B, C] → [X, A, B, C]: React thinks A changed to X, B changed to A...

// With keys: React tracks by identity
// key="a" moved to position 2? Just move the DOM node.
```

### Visual Diffing Flow

```
Previous vDOM:                New vDOM:
<ul>                          <ul>
  <li key="1">Alice</li>        <li key="1">Alice</li>
  <li key="2">Bob</li>    →     <li key="3">Charlie</li>  ← NEW
  <li key="3">Charlie</li>      <li key="2">Bob</li>      ← MOVED
</ul>                         </ul>

Diff result:
  - Insert <li key="3"> at position 2 (already exists, just move)
  - Move <li key="2"> to position 3
  - 0 DOM creations, 1 DOM move → very efficient
```

---

## 2.4 React Fiber Architecture

Before React 16, React's rendering was **synchronous and uninterruptible**. If a component tree was large, the main thread would be blocked, causing dropped frames (janky UI).

**Fiber** is React's internal reimplementation of the rendering engine, introduced in React 16.

### The Problem Fiber Solves

```
Old React (Stack Reconciler):
  Start rendering big component tree
    → Can't stop
    → Browser can't handle user input
    → UI feels frozen

Fiber:
  Start rendering big component tree
    → "Is there higher priority work? (user click, animation?)"
    → YES: Pause this render, handle high priority work
    → Resume render when ready
```

### Fiber Node

Each React element has a corresponding **Fiber node** — a JavaScript object with:

```javascript
{
  type,          // Component type (function, class, 'div', etc.)
  key,           // Key for reconciliation
  stateNode,     // DOM node or component instance
  return,        // Parent fiber
  child,         // First child fiber
  sibling,       // Next sibling fiber
  pendingProps,  // New props being applied
  memoizedProps, // Props from last render
  memoizedState, // State from last render
  updateQueue,   // Queue of pending state updates
  effectTag,     // What needs to happen (update, placement, deletion)
  alternate,     // Previous fiber (double buffering)
}
```

### Double Buffering

Fiber maintains two trees:
- **Current tree** — what's currently rendered on screen
- **Work-in-progress (WIP) tree** — the tree being built for the next render

```
         Current Tree            WIP Tree
         (on screen)            (being built)
         
         Root Fiber    ←→       Root Fiber WIP
           |                        |
         App Fiber    ←→       App Fiber WIP
           |                        |
         Header       ←→       Header WIP
```

When the WIP tree is complete, React swaps it to become the current tree in the **commit phase**. This ensures the UI is never in a half-rendered state.

---

## 2.5 The Two Phases of Rendering

React rendering happens in two distinct phases:

### Phase 1: Render Phase (Pure, can be interrupted)

```
Trigger (state/prop change)
  → React calls your component function
    → Component returns React elements (vDOM)
      → React builds the fiber tree
        → React diffs against previous fiber tree
          → React builds a list of "effects" (what needs changing)

✓ No real DOM changes here
✓ Can be interrupted and restarted (Fiber)
✓ Must be pure / side-effect free
```

### Phase 2: Commit Phase (Synchronous, cannot be interrupted)

```
React takes the list of effects
  → BeforeMutation: getSnapshotBeforeUpdate (class), passive cleanup
  → Mutation: ACTUALLY updates the real DOM
  → Layout: componentDidMount/Update, useLayoutEffect fires
  → Passive: useEffect fires (after paint)

✓ Happens all at once
✓ Cannot be interrupted (user sees consistent UI)
```

### Render Phase vs Commit Phase

```
┌──────────────────────────────────────────────────────┐
│                   RENDER PHASE                       │
│  (can be paused, resumed, restarted, thrown away)    │
│                                                      │
│  Component function runs                             │
│  JSX → React Elements → Fiber diffing               │
│  Effects collected                                   │
└──────────────────────────────────────────────────────┘
                          ↓
┌──────────────────────────────────────────────────────┐
│                   COMMIT PHASE                       │
│  (synchronous, cannot be interrupted)               │
│                                                      │
│  Real DOM mutations happen                           │
│  Refs attached                                       │
│  useLayoutEffect fires                               │
│  Browser paints                                      │
│  useEffect fires                                     │
└──────────────────────────────────────────────────────┘
```

---

## 2.6 Batching Updates

React **batches** multiple `setState` calls into a single render for performance.

```jsx
function handleClick() {
  setCount(c => c + 1);   // Does NOT render yet
  setName('Alice');        // Does NOT render yet
  setLoading(false);       // Does NOT render yet
  // ↑ React batches all three, renders ONCE
}
```

**Before React 18:** Batching only happened inside React event handlers. Async callbacks (setTimeout, Promises) triggered separate renders per setState.

**React 18:** Automatic batching everywhere, including async code:

```jsx
// React 18 — batched even in async
setTimeout(() => {
  setCount(c => c + 1);  // batched
  setName('Alice');       // batched
  // Only ONE re-render
}, 1000);
```

---

## 2.7 React Scheduling

React 18 introduced the **Scheduler** — a priority system for work:

```
Immediate    → Synchronous, must happen now (user input)
UserBlocking → High priority (clicks, typing)
Normal       → Default rendering
Low          → State updates from data fetching
Idle         → Work that can wait until browser is idle
```

```jsx
// useTransition marks an update as "low priority"
const [isPending, startTransition] = useTransition();

startTransition(() => {
  // This update can be interrupted if higher priority work arrives
  setSearchResults(filtered);
});
```

---

## 2.8 Interview: Explain How React Rendering Works

> **Interviewer:** "Can you walk me through what happens when you call setState?"

**Model Answer:**

"When `setState` is called, React doesn't immediately update the DOM. Instead, it enqueues an update. React then schedules a re-render using its Fiber scheduler.

During the render phase, React calls the component function, which returns a new virtual DOM tree expressed as React elements. React then performs reconciliation — it diffs the new fiber tree against the current one using its O(n) diffing algorithm. During this phase, React collects a list of effects: which DOM nodes need to be created, updated, or deleted.

The render phase can be interrupted by higher-priority work in React 18's Concurrent Mode.

Once the render phase is complete, React enters the commit phase, which is synchronous. React applies all DOM mutations in one go, then fires `useLayoutEffect`, then the browser paints, then `useEffect` fires.

This two-phase architecture ensures the UI is always consistent — you never see a half-rendered component."

---

### 📝 QUICK REVISION — Section 2

- Virtual DOM = plain JS objects mirroring real DOM — fast to manipulate
- Reconciliation = diffing old vs new vDOM → minimal real DOM changes
- Diffing heuristics: different element types = full rebuild; keys = identity tracking
- Fiber = unit of work; enables interruptible, priority-based rendering
- Double buffering: current tree (on screen) + WIP tree (being built)
- Render phase: pure, can be interrupted. Commit phase: DOM mutations, synchronous
- Batching: multiple setState calls → single render
- React 18: automatic batching everywhere + Scheduler for priority

---

# 3. SETTING UP REACT

## 3.1 Creating a React App

### Option A: Vite (Recommended for new projects)

```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

Why Vite?
- Extremely fast dev server (uses native ES modules)
- Fast Hot Module Replacement (HMR)
- Lean config
- Better DX than Create React App

### Option B: Create React App (Legacy)

```bash
npx create-react-app my-app
cd my-app
npm start
```

> CRA is largely deprecated. Prefer Vite or Next.js.

### Option C: Next.js (SSR/SSG projects)

```bash
npx create-next-app@latest my-app
```

---

## 3.2 npm vs npx

| | npm | npx |
|--|-----|-----|
| Purpose | Install packages | Execute packages without installing |
| Example | `npm install react` | `npx create-react-app my-app` |
| Installs to disk? | Yes | Temporary (if not installed) |
| Use case | Project dependencies | CLI tools, scaffolding |

---

## 3.3 Project Structure (Vite + React)

```
my-app/
├── public/              ← Static assets (index.html, favicon)
├── src/
│   ├── main.jsx         ← Entry point — mounts React to DOM
│   ├── App.jsx          ← Root component
│   ├── App.css
│   ├── components/      ← Reusable components
│   ├── pages/           ← Route-level components
│   ├── hooks/           ← Custom hooks
│   ├── services/        ← API calls
│   ├── utils/           ← Helper functions
│   └── context/         ← React context
├── index.html           ← Shell HTML (has <div id="root">)
├── vite.config.js
└── package.json
```

### Entry Point: main.jsx

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

// React 18: createRoot API
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

`React.StrictMode` runs component functions twice in development to catch side effects in the render phase. It has no effect in production.

---

## 3.4 package.json Explained

```json
{
  "name": "my-app",
  "scripts": {
    "dev": "vite",          // Start dev server
    "build": "vite build",  // Bundle for production
    "preview": "vite preview" // Preview production build
  },
  "dependencies": {
    "react": "^18.2.0",       // React core
    "react-dom": "^18.2.0"    // React DOM renderer
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.0.0", // Vite React plugin (Babel/SWC)
    "vite": "^4.4.5"
  }
}
```

- `dependencies` = required at runtime
- `devDependencies` = build tools, linters, test runners — not in production bundle

---

# 4. JSX IN DEPTH

## 4.1 What is JSX?

JSX (JavaScript XML) is **syntactic sugar** — it looks like HTML but is actually JavaScript.

```jsx
// What you write:
const element = <h1 className="title">Hello World</h1>;

// What Babel transforms it into:
const element = React.createElement(
  'h1',
  { className: 'title' },
  'Hello World'
);
```

JSX is **not** HTML. It's a syntax extension that gets compiled before it reaches the browser.

---

## 4.2 JSX Transpilation — How Babel Works

```
Your JSX code
     ↓
Babel (or SWC in Vite)
     ↓
React.createElement() calls
     ↓
JavaScript Objects (React Elements)
     ↓
React processes these during rendering
     ↓
Real DOM
```

### New JSX Transform (React 17+)

You no longer need `import React from 'react'` at the top of every file. The new JSX transform auto-imports the runtime:

```jsx
// Old way (before React 17)
import React from 'react'; // Required for JSX to work
function Hello() { return <div>Hi</div>; }

// New way (React 17+) — Babel handles the import automatically
function Hello() { return <div>Hi</div>; }
```

---

## 4.3 React.createElement() Deep Dive

Every JSX element compiles to a `React.createElement` call:

```javascript
React.createElement(type, props, ...children)
```

```jsx
// JSX:
<div className="box" onClick={handleClick}>
  <h1>Title</h1>
  <p>Body text</p>
</div>

// Compiled to:
React.createElement(
  'div',
  { className: 'box', onClick: handleClick },
  React.createElement('h1', null, 'Title'),
  React.createElement('p', null, 'Body text')
)

// Produces this object (React Element):
{
  type: 'div',
  props: {
    className: 'box',
    onClick: handleClick,
    children: [
      { type: 'h1', props: { children: 'Title' } },
      { type: 'p', props: { children: 'Body text' } }
    ]
  },
  key: null,
  ref: null
}
```

This plain JavaScript object is a **React Element** — a description of what to render, not an actual DOM node.

---

## 4.4 JSX Rules

### Rule 1: Return a single root element

```jsx
// ❌ Wrong — two root elements
return (
  <h1>Title</h1>
  <p>Body</p>
);

// ✅ Correct — wrapped in a div
return (
  <div>
    <h1>Title</h1>
    <p>Body</p>
  </div>
);

// ✅ Correct — use Fragment (no extra DOM node)
return (
  <>
    <h1>Title</h1>
    <p>Body</p>
  </>
);
```

**Why?** `React.createElement` returns a single object. You can't return two values from one call.

### Rule 2: Close all tags

```jsx
// ❌ Wrong — HTML allows unclosed img
<img src="photo.jpg">

// ✅ Correct — JSX requires self-closing
<img src="photo.jpg" />
```

### Rule 3: Use camelCase for attributes

```jsx
// HTML attribute → JSX attribute
class       → className    (class is a JS reserved word)
for         → htmlFor      (for is a JS reserved word)
onclick     → onClick
tabindex    → tabIndex
stroke-width → strokeWidth
```

### Rule 4: JavaScript expressions go in `{}`

```jsx
const name = 'Alice';
const isAdmin = true;

return (
  <div>
    <p>Hello, {name}!</p>                           {/* Variable */}
    <p>{2 + 2}</p>                                  {/* Expression */}
    <p>{isAdmin ? 'Admin' : 'User'}</p>             {/* Ternary */}
    <p>{name.toUpperCase()}</p>                     {/* Method call */}
    <button style={{ color: 'red', fontSize: 16 }}> {/* Object literal (double braces) */}
      Click
    </button>
  </div>
);
```

**Note the double braces** for inline styles: outer `{}` = JSX expression, inner `{}` = JavaScript object.

### Rule 5: No statements in JSX

```jsx
// ❌ Wrong — if is a statement, not an expression
return (
  <div>
    {if (condition) { <p>Yes</p> }}   // Syntax error
  </div>
);

// ✅ Correct — use ternary (expression)
return (
  <div>
    {condition ? <p>Yes</p> : null}
  </div>
);
```

---

## 4.5 Common JSX Mistakes

```jsx
// ❌ Mistake 1: Not wrapping JSX in parentheses (auto-semicolon insertion)
return
  <div>Hello</div>;  // Returns undefined! JS inserts semicolon after 'return'

// ✅ Fix:
return (
  <div>Hello</div>
);

// ❌ Mistake 2: Rendering false/null/undefined
const count = 0;
return <div>{count && <span>Items</span>}</div>;
// ↑ Renders "0" in the DOM! 0 is falsy but JSX renders numbers

// ✅ Fix: use explicit boolean
return <div>{count > 0 && <span>Items</span>}</div>;
// or
return <div>{Boolean(count) && <span>Items</span>}</div>;

// ❌ Mistake 3: Using object directly as JSX child
return <div>{myObject}</div>; // Error: Objects not valid as React child

// ✅ Fix: convert to string or extract properties
return <div>{myObject.name}</div>;
```

---

### 📝 QUICK REVISION — Section 4

- JSX = syntactic sugar → compiles to React.createElement()
- React.createElement(type, props, children) → plain JS object (React Element)
- JSX rules: single root, close all tags, camelCase attrs, expressions in {}
- Fragments `<>` avoid extra DOM nodes
- Double braces for inline styles: `style={{ color: 'red' }}`
- `0 && <Component>` renders "0" — use `count > 0 &&` instead

---

# 5. COMPONENTS IN DEPTH

## 5.1 What is a Component?

A component is a **JavaScript function that returns JSX**. It's the fundamental unit of React UIs.

```
Component = Function that takes props → Returns React elements
```

```jsx
// Simplest possible component
function Hello() {
  return <h1>Hello, World!</h1>;
}

// With props
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// Usage
<Greeting name="Alice" />
```

**Critical rule:** Component names **must start with a capital letter.**

```jsx
function button() { return <button>Click</button>; } // ❌ React treats as HTML element
function Button() { return <button>Click</button>; } // ✅ React treats as component
```

---

## 5.2 Functional Components

Modern React is 100% functional components + hooks. Clean, concise, testable.

```jsx
// Full example with different patterns
function UserCard({ user, onFollow }) {
  const isVerified = user.followers > 1000;

  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h2>
        {user.name}
        {isVerified && <span className="badge">✓</span>}
      </h2>
      <p>{user.bio}</p>
      <p>{user.followers} followers</p>
      <button onClick={() => onFollow(user.id)}>
        Follow
      </button>
    </div>
  );
}
```

---

## 5.3 Class Components (Legacy Knowledge)

Class components predate hooks. You'll see them in older codebases.

```jsx
import React, { Component } from 'react';

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 }; // State in constructor
  }

  increment = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>+</button>
      </div>
    );
  }
}
```

**Why we use functional components today:**
- Less boilerplate
- No `this` confusion
- Hooks provide everything class components offer
- Easier to test and reason about

---

## 5.4 Pure Components

A **pure component** renders the same output for the same props. No side effects, no external state.

```jsx
// Pure — output depends ONLY on props
function Price({ amount, currency }) {
  return <span>{currency}{amount.toFixed(2)}</span>;
}

// Impure — output depends on external state (current time)
function CurrentTime() {
  return <span>{new Date().toLocaleTimeString()}</span>; // Different every render
}
```

Pure components are safe to memoize (see `React.memo` in Section 12).

---

## 5.5 Component Composition

**Composition** is building complex UI from simple components, like LEGO.

```jsx
// Atomic components
function Avatar({ src, alt }) {
  return <img className="avatar" src={src} alt={alt} />;
}

function Badge({ text, color }) {
  return <span className={`badge badge-${color}`}>{text}</span>;
}

// Composed component
function UserProfile({ user }) {
  return (
    <div className="profile">
      <Avatar src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      {user.isPro && <Badge text="PRO" color="gold" />}
    </div>
  );
}
```

### The children Prop

React's built-in composition mechanism: pass JSX as children.

```jsx
// Card is a layout component that renders whatever children you give it
function Card({ title, children }) {
  return (
    <div className="card">
      <div className="card-header">{title}</div>
      <div className="card-body">{children}</div>
    </div>
  );
}

// Usage — children is anything between the tags
<Card title="User Info">
  <p>Name: Alice</p>
  <p>Role: Admin</p>
  <button>Edit</button>
</Card>
```

---

## 5.6 Smart vs Dumb Components (Container vs Presentational)

| Smart (Container) | Dumb (Presentational) |
|---|---|
| Contains logic & state | Pure display |
| Fetches data | Receives data via props |
| Passes data down | Renders UI |
| Less reusable | Highly reusable |
| Harder to test | Easy to test |

```jsx
// ❌ Mixed concern — hard to reuse
function UserList() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch('/api/users').then(r => r.json()).then(setUsers);
  }, []);

  return (
    <ul>
      {users.map(u => <li key={u.id}>{u.name} - {u.email}</li>)}
    </ul>
  );
}

// ✅ Separated concerns
// Smart: handles data
function UserListContainer() {
  const [users, setUsers] = useState([]);
  useEffect(() => {
    fetch('/api/users').then(r => r.json()).then(setUsers);
  }, []);
  return <UserList users={users} />;  // passes data down
}

// Dumb: pure display, highly reusable
function UserList({ users }) {
  return (
    <ul>
      {users.map(u => <li key={u.id}>{u.name} - {u.email}</li>)}
    </ul>
  );
}
```

---

## 5.7 Props Drilling (The Problem)

When data needs to pass through many layers of components:

```
App (has user data)
  └── Layout
        └── Sidebar
              └── UserMenu
                    └── Avatar  ← Actually needs the data
```

Every intermediate component must accept and forward the `user` prop — even if it doesn't use it. This is **props drilling**.

```jsx
// Props drilling — Layout and Sidebar don't use 'user' but must pass it
function App() {
  const user = { name: 'Alice', avatar: '...' };
  return <Layout user={user} />;
}

function Layout({ user }) {
  return <Sidebar user={user} />;  // Doesn't use user, just passes it
}

function Sidebar({ user }) {
  return <UserMenu user={user} />;  // Same
}

function UserMenu({ user }) {
  return <Avatar src={user.avatar} />;  // Finally uses it
}
```

**Solutions:** React Context API (Section 12.7) or state management libraries (Section 16).

---

### 📝 QUICK REVISION — Section 5

- Component = JS function returning JSX; name must start with capital letter
- Functional components + hooks = modern standard
- Class components = legacy, avoid in new code
- Pure component: same props → same output (safe to memoize)
- Composition: build complex UI from simple pieces; `children` prop enables layout components
- Smart = logic + data; Dumb = pure display
- Props drilling = passing data through many layers; solved by Context

---

# 6. PROPS IN DEPTH

## 6.1 What are Props?

Props (properties) are the way components communicate. They're **arguments passed to a component** from a parent.

```jsx
// Parent sends props
<UserCard
  name="Alice"
  age={28}
  isAdmin={true}
  onClick={() => alert('clicked')}
  style={{ color: 'blue' }}
/>

// Component receives them
function UserCard({ name, age, isAdmin, onClick, style }) {
  return (
    <div style={style} onClick={onClick}>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      {isAdmin && <span>Admin</span>}
    </div>
  );
}
```

Props can be **any JavaScript value**: strings, numbers, booleans, arrays, objects, functions, other components.

---

## 6.2 Unidirectional Data Flow

React enforces **one-way data flow**: data flows **down** from parent to child via props.

```
App
 ↓ props
  Header
    ↓ props
     NavBar
       ↓ props
        NavItem
```

Children **cannot** directly modify parent state. They communicate upward by calling **callback functions** passed as props.

```jsx
// Child calls parent's function to "lift state up"
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <Child
      count={count}
      onIncrement={() => setCount(c => c + 1)}  // Pass handler down
    />
  );
}

function Child({ count, onIncrement }) {
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={onIncrement}>+</button>  {/* Calls parent's function */}
    </div>
  );
}
```

---

## 6.3 Why Props are Immutable

Props are **read-only** within a component. You must never modify them.

```jsx
// ❌ NEVER do this
function Badge({ label }) {
  label = label.toUpperCase();  // Mutating a prop
  return <span>{label}</span>;
}

// ✅ Correct — derive a new value
function Badge({ label }) {
  const displayLabel = label.toUpperCase();  // Local variable
  return <span>{displayLabel}</span>;
}
```

**Why?** Immutability enables React's change detection. If you mutate props, React can't know what changed. It also makes data flow predictable — you always know where a prop came from.

---

## 6.4 Default Props

```jsx
// Method 1: Default parameters (recommended)
function Button({ label = 'Click Me', variant = 'primary', disabled = false }) {
  return (
    <button className={`btn btn-${variant}`} disabled={disabled}>
      {label}
    </button>
  );
}

// Usage — omitting props uses defaults
<Button />                         // "Click Me", primary, enabled
<Button label="Submit" />          // "Submit", primary, enabled
<Button variant="danger" disabled /> // "Click Me", danger, disabled
```

---

## 6.5 Prop Types (Runtime Validation)

```bash
npm install prop-types
```

```jsx
import PropTypes from 'prop-types';

function UserCard({ name, age, email, role }) {
  return <div>{name}</div>;
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number.isRequired,
  email: PropTypes.string,
  role: PropTypes.oneOf(['admin', 'user', 'moderator']),
};

UserCard.defaultProps = {
  email: 'Not provided',
  role: 'user',
};
```

> In TypeScript projects, you use **TypeScript interfaces** instead of PropTypes:

```tsx
interface UserCardProps {
  name: string;
  age: number;
  email?: string;
  role?: 'admin' | 'user' | 'moderator';
}

function UserCard({ name, age, email = 'Not provided', role = 'user' }: UserCardProps) {
  return <div>{name}</div>;
}
```

---

## 6.6 Spreading Props

```jsx
// Sometimes useful for passing all props to an underlying element
function FancyButton({ className, ...restProps }) {
  return (
    <button
      className={`fancy-btn ${className || ''}`}
      {...restProps}  // Passes onClick, disabled, type, etc.
    />
  );
}

// Usage
<FancyButton onClick={handleSubmit} disabled={isLoading} type="submit">
  Submit
</FancyButton>
```

**⚠️ Use carefully.** Spreading unknown props can pass invalid attributes to DOM elements, causing warnings.

---

## 6.7 Interview Questions — Props

**Q: Why are props immutable?**

> Immutability enables predictable data flow and allows React to efficiently detect changes. If props were mutable, React couldn't reliably determine when to re-render, and debugging would be a nightmare since you couldn't trace where a value came from.

**Q: How do you pass data from child to parent?**

> Through callback functions. The parent defines a function that updates its state, passes that function as a prop to the child, and the child calls it with the new data. This pattern is called "lifting state up."

**Q: What's the difference between props and state?**

> Props are external inputs — data passed into a component, read-only, owned by the parent. State is internal — data owned and managed by the component itself, can be changed by the component. Props flow down; state lives within.

---

### 📝 QUICK REVISION — Section 6

- Props = read-only arguments from parent to component
- Unidirectional flow: data goes down, events go up via callbacks
- Props are immutable — never mutate them, derive new values instead
- Default props via default parameters: `function Foo({ bar = 'default' })`
- Spread props `{...props}` — useful but use with care
- PropTypes for runtime validation; TypeScript for compile-time

# 7. STATE IN REACT

> State is the most important concept in React. Master this deeply.

## 7.1 What is State?

**State** is data that belongs to a component and can change over time. When state changes, React re-renders the component.

```
Props = data passed IN from outside (read-only)
State = data living INSIDE a component (can change)
```

**Real-world analogy:** A shopping cart. The *items* in the cart are state — they start empty, you add things, remove things. The cart re-renders to show current contents whenever state changes.

---

## 7.2 Why State Exists

Without state, components are static. They render once and never update.

```jsx
// No state — renders "0" forever, clicking does nothing
function BrokenCounter() {
  let count = 0;
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => count++}>+</button>  // Increments, but React doesn't know!
    </div>
  );
}

// With state — React knows when it changes, re-renders
function WorkingCounter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

`count++` modifies a local variable — React has no knowledge of it. `setCount(...)` tells React: "this value changed, please re-render."

---

## 7.3 useState Internals

```jsx
const [state, setState] = useState(initialValue);
```

**What useState actually does:**

1. On the first render, React allocates a "slot" in a linked list for this state
2. Returns `[currentValue, setterFunction]`
3. When `setState` is called, React enqueues an update
4. React schedules a re-render
5. On re-render, React reads the updated value from the slot
6. Returns the new value

```
Hook slots are stored per component, ordered by hook call order.
That's why you CANNOT call hooks conditionally (the order must be consistent).

Slot 0: useState(0)    → count
Slot 1: useState('')   → name
Slot 2: useEffect(...) → side effect
Slot 3: useState([])   → items
```

---

## 7.4 State Updates are Asynchronous

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    console.log(count); // Still logs 0! State update is queued, not immediate
  }
}
```

State doesn't change immediately when you call `setState`. The update is **enqueued** and the component re-renders on the next cycle. The new value is available in the *next* render.

---

## 7.5 Batching and the Stale Closure Problem

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleTripleIncrement() {
    // ❌ This only increments by 1, not 3!
    setCount(count + 1); // count is 0 → sets to 1
    setCount(count + 1); // count is STILL 0 (stale closure) → sets to 1
    setCount(count + 1); // count is STILL 0 → sets to 1
    // All three use the same stale 'count' value from this render
  }

  function handleCorrectTripleIncrement() {
    // ✅ Functional update — always uses the latest state
    setCount(c => c + 1); // c=0 → 1
    setCount(c => c + 1); // c=1 → 2
    setCount(c => c + 1); // c=2 → 3
  }
}
```

**Rule of thumb:** Whenever the new state depends on the previous state, use the **functional update form**: `setState(prev => prev + 1)`.

---

## 7.6 State Immutability

React compares state using **reference equality** (`===`). Mutating objects/arrays directly won't trigger re-renders.

```jsx
// ❌ Mutating state directly — React doesn't detect change
const [user, setUser] = useState({ name: 'Alice', age: 28 });

function birthday() {
  user.age++;          // Mutates existing object
  setUser(user);       // Same reference — React thinks nothing changed!
}

// ✅ Creating new object — React detects the new reference
function birthday() {
  setUser({ ...user, age: user.age + 1 });  // New object = re-render
}
```

### Updating Arrays in State

```jsx
const [items, setItems] = useState(['a', 'b', 'c']);

// Add item
setItems([...items, 'd']);                        // ✅ New array
setItems(prev => [...prev, 'd']);                  // ✅ Functional form

// Remove item
setItems(items.filter(item => item !== 'b'));       // ✅

// Update item
setItems(items.map(item =>
  item === 'b' ? 'UPDATED' : item               // ✅
));

// ❌ These don't trigger re-render
items.push('d');        setItems(items);            // Mutation!
items.splice(0, 1);     setItems(items);            // Mutation!
```

### Updating Nested Objects

```jsx
const [state, setState] = useState({
  user: { name: 'Alice', address: { city: 'NYC' } }
});

// ✅ Spread all levels
setState({
  ...state,
  user: {
    ...state.user,
    address: {
      ...state.user.address,
      city: 'LA'
    }
  }
});

// ✅ Easier with Immer (a popular library)
import { produce } from 'immer';
setState(produce(draft => { draft.user.address.city = 'LA'; }));
```

---

## 7.7 Multiple State Variables

```jsx
// ✅ Separate state variables for unrelated data
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
}

// ✅ Group related state into an object
function Form() {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    name: ''
  });

  function handleChange(field, value) {
    setFormData(prev => ({ ...prev, [field]: value }));
  }
}
```

---

## 7.8 Common State Bugs

```jsx
// Bug 1: Using state immediately after setting it
function handleSubmit() {
  setUser({ name: 'Alice' });
  console.log(user.name); // Still the OLD user — state isn't updated yet
  submitForm(user);        // Sends OLD data!
}
// Fix: use the value you're setting, not the state variable

// Bug 2: Infinite loop
function Component() {
  const [data, setData] = useState(null);
  setData(fetchData());  // ❌ Sets state during render → re-render → sets state again...
}
// Fix: use useEffect

// Bug 3: Stale closure in setTimeout/async
function Timer() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    setTimeout(() => {
      console.log(count); // Always logs 0! Captured in closure
    }, 1000);
  }, []); // ← Empty deps means 'count' is captured as 0 forever
}
// Fix: use functional update or include count in deps
```

---

### 📝 QUICK REVISION — Section 7

- State = internal data that changes; triggers re-render when updated
- `useState` stores state in a slot; order of hooks must be consistent
- State updates are asynchronous — new value available on NEXT render
- Use functional update `setState(prev => ...)` when new state depends on old
- Immutability: return new objects/arrays, never mutate existing ones
- `{...spread}` for objects, `[...spread]` for arrays to create new references

---

# 8. EVENT HANDLING

## 8.1 React's Synthetic Events

React wraps native DOM events in **SyntheticEvents** — a cross-browser normalized wrapper.

```jsx
function Button() {
  function handleClick(event) {
    // 'event' is a SyntheticEvent — same API across all browsers
    console.log(event.type);           // 'click'
    console.log(event.target);         // The DOM element clicked
    console.log(event.currentTarget);  // The element with the handler attached
    event.preventDefault();            // Prevent default browser behavior
    event.stopPropagation();           // Stop event bubbling
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

### Attaching Event Handlers

```jsx
// Inline arrow function (creates new function each render)
<button onClick={() => handleClick(item.id)}>Delete</button>

// Reference to function (more performant)
<button onClick={handleClick}>Submit</button>

// Passing arguments
<button onClick={(e) => handleDelete(item.id, e)}>Delete</button>
```

---

## 8.2 Common Events

```jsx
// Mouse events
<div onClick={handler}
     onDoubleClick={handler}
     onMouseEnter={handler}
     onMouseLeave={handler}
     onMouseMove={handler} />

// Keyboard events
<input onKeyDown={handler}   // Key pressed down
       onKeyUp={handler}     // Key released
       onKeyPress={handler}  // Deprecated, use onKeyDown
/>

// Form events
<input onChange={handler}    // Value changed
       onFocus={handler}     // Input focused
       onBlur={handler}      // Input lost focus
/>
<form onSubmit={handler} />

// Focus events
<input onFocus={handler} onBlur={handler} />
```

---

## 8.3 Event Bubbling and Capturing

Events bubble UP the DOM tree by default.

```
User clicks inner <span>
  → span's onClick fires
    → div's onClick fires
      → body's onClick fires
```

```jsx
function Parent() {
  return (
    <div onClick={() => console.log('div clicked')}>
      <button onClick={(e) => {
        console.log('button clicked');
        e.stopPropagation(); // Prevent event from reaching div
      }}>
        Click
      </button>
    </div>
  );
}
```

**Event capturing** (rare): events travel DOWN before bubbling up.

```jsx
// onClickCapture fires during capture phase (before onClick on children)
<div onClickCapture={() => console.log('captured!')}>
  <button onClick={() => console.log('button')}>Click</button>
</div>
// Order: "captured!" → "button"
```

---

## 8.4 preventDefault

```jsx
// Form submission — prevent default page reload
<form onSubmit={(e) => {
  e.preventDefault(); // Stop browser from navigating
  handleFormSubmit();
}}>

// Link — prevent navigation
<a href="/old" onClick={(e) => {
  e.preventDefault();
  navigate('/new');
}}>
```

---

# 9. CONDITIONAL RENDERING

## 9.1 If/Else (Return Early Pattern)

```jsx
function UserGreeting({ user, isLoading }) {
  if (isLoading) {
    return <Spinner />;  // Early return
  }

  if (!user) {
    return <LoginPrompt />;
  }

  return <h1>Welcome back, {user.name}!</h1>;
}
```

---

## 9.2 Ternary Operator

Best for choosing between two options inline:

```jsx
function ToggleButton({ isOn, onToggle }) {
  return (
    <button
      onClick={onToggle}
      className={isOn ? 'btn-on' : 'btn-off'}
    >
      {isOn ? 'Turn Off' : 'Turn On'}
    </button>
  );
}
```

---

## 9.3 Logical AND (`&&`)

Best for showing something or nothing:

```jsx
function Notification({ hasMessages, count }) {
  return (
    <header>
      <Logo />
      {hasMessages && <span className="badge">{count}</span>}
    </header>
  );
}
```

**⚠️ The 0 Gotcha:**

```jsx
// ❌ Renders "0" in the DOM
{messages.length && <MessageList messages={messages} />}

// ✅ Fix — use explicit boolean
{messages.length > 0 && <MessageList messages={messages} />}
{!!messages.length && <MessageList messages={messages} />}
```

---

## 9.4 Null Coalescing and Optional Rendering

```jsx
// Render null to render nothing (no DOM output)
function AdminPanel({ isAdmin }) {
  if (!isAdmin) return null;
  return <div>Admin Controls</div>;
}

// Nullish coalescing — render fallback if null/undefined
function UserName({ name }) {
  return <span>{name ?? 'Anonymous'}</span>;
}
```

---

# 10. LISTS & KEYS

## 10.1 Rendering Lists

```jsx
function ProductList({ products }) {
  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>
          <img src={product.image} alt={product.name} />
          <span>{product.name}</span>
          <span>${product.price}</span>
        </li>
      ))}
    </ul>
  );
}
```

---

## 10.2 Why Keys Exist (Deep Explanation)

Keys tell React **which item is which** during reconciliation.

Without keys, React uses position to match items:

```
Old list: [Apple, Banana, Cherry]    positions: [0, 1, 2]
New list: [Orange, Apple, Banana, Cherry]  positions: [0, 1, 2, 3]

React sees:
  Position 0: Apple → Orange (different! Update DOM)
  Position 1: Banana → Apple (different! Update DOM)
  Position 2: Cherry → Banana (different! Update DOM)
  Position 3: (new) Cherry   (Insert DOM node)

→ 3 DOM updates + 1 insert = expensive, and state is LOST
```

With keys, React uses identity:

```
Old: [key=apple: Apple, key=banana: Banana, key=cherry: Cherry]
New: [key=orange: Orange, key=apple: Apple, key=banana: Banana, key=cherry: Cherry]

React sees:
  key=orange: NEW → insert at position 0
  key=apple: EXISTS → move to position 1
  key=banana: EXISTS → move to position 2
  key=cherry: EXISTS → move to position 3

→ 1 insert + 3 moves = efficient, and state is PRESERVED
```

---

## 10.3 Key Rules

```jsx
// ✅ Use stable, unique IDs
{users.map(user => <User key={user.id} {...user} />)}

// ✅ Unique within siblings, not globally
{posts.map(post => (
  <div key={post.id}>
    {post.comments.map(comment => (
      <Comment key={comment.id} {...comment} />  // OK — different sibling group
    ))}
  </div>
))}
```

---

## 10.4 Index as Key — When It's Wrong

```jsx
// ❌ Using index as key — problematic when list changes
{todos.map((todo, index) => (
  <TodoItem key={index} todo={todo} />
))}
```

**The problem:** If items are reordered, filtered, or inserted, indices shift:

```
Original:  [key=0: Alice, key=1: Bob, key=2: Charlie]
Delete Bob: [key=0: Alice, key=1: Charlie]

React sees:
  key=1: "Bob" → "Charlie" (UPDATE — but Charlie already existed!)
  key=2: Charlie → removed (DELETE — Charlie wasn't removed!)

Result: Charlie has wrong state (Bob's leftover state), bugs everywhere
```

**When index is OK:**
- The list is static (never reordered, filtered, or inserted into)
- Items have no state

```jsx
// OK — static list of navigation items
{navItems.map((item, index) => (
  <NavLink key={index} href={item.href}>{item.label}</NavLink>
))}
```

---

### 📝 QUICK REVISION — Section 10

- `.map()` + `key` = the standard pattern for rendering lists
- Keys help React track item identity during reconciliation
- Keys must be unique among siblings, stable across renders
- Use item.id for keys — not index (unless list is static and never reordered)
- Wrong keys = incorrect state, buggy animations, wasted renders

---

# 11. FORMS IN REACT

## 11.1 Controlled Components

In a controlled component, React controls the input's value via state.

```jsx
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  function handleSubmit(e) {
    e.preventDefault();
    // email and password are in sync with inputs
    loginUser({ email, password });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}                           // Controlled by state
        onChange={e => setEmail(e.target.value)} // State updates on change
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={e => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

**The contract:** `value` + `onChange` = controlled. You're the single source of truth.

---

## 11.2 Uncontrolled Components

Uncontrolled components store their value in the DOM, not React state. Access via `ref`.

```jsx
import { useRef } from 'react';

function SearchForm() {
  const inputRef = useRef();

  function handleSubmit(e) {
    e.preventDefault();
    console.log(inputRef.current.value); // Read directly from DOM
  }

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} type="text" defaultValue="initial" />
      <button type="submit">Search</button>
    </form>
  );
}
```

| Controlled | Uncontrolled |
|---|---|
| State drives value | DOM drives value |
| React is source of truth | DOM is source of truth |
| Can validate on every keystroke | Validate on submit only |
| More code | Less code |
| Recommended for most cases | Good for file inputs, simple one-time reads |

---

## 11.3 Handling Multiple Inputs

```jsx
function ProfileForm() {
  const [form, setForm] = useState({
    name: '',
    email: '',
    age: '',
    role: 'user'
  });

  function handleChange(e) {
    const { name, value, type, checked } = e.target;
    setForm(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value  // Handle checkboxes
    }));
  }

  return (
    <form>
      <input name="name" value={form.name} onChange={handleChange} />
      <input name="email" value={form.email} onChange={handleChange} />
      <input name="age" type="number" value={form.age} onChange={handleChange} />
      <select name="role" value={form.role} onChange={handleChange}>
        <option value="user">User</option>
        <option value="admin">Admin</option>
      </select>
    </form>
  );
}
```

---

# 12. HOOKS — COMPLETE DEEP DIVE

## 12.0 Why Hooks Were Introduced

Before hooks (pre-2019), React had a critical problem: **you couldn't reuse stateful logic between components.**

If two components needed the same logic (e.g., "subscribe to window resize"), you had to use:
- **HOCs (Higher-Order Components)** — wrapper components that inject props
- **Render Props** — a pattern using a prop that's a function

Both patterns caused:
- **Wrapper hell** — deeply nested component trees
- **Prop confusion** — unclear where props came from
- **Hard to follow** — logic scattered across lifecycle methods

**Hooks solution:** Extract stateful logic into reusable JavaScript functions that can be called from any component.

```
Before Hooks:
  <DataProvider>
    <ThemeConsumer>
      <AuthConsumer>
        <RouterConsumer>
          <YourActualComponent />  ← Wrapped in 4 HOCs!
        </RouterConsumer>
      </AuthConsumer>
    </ThemeConsumer>
  </DataProvider>

After Hooks:
  function YourActualComponent() {
    const data = useData();
    const theme = useTheme();
    const auth = useAuth();
    const router = useRouter();
    // Clean!
  }
```

---

## 12.1 Rules of Hooks

React hooks work because of **ordered, consistent calls per render**. Violate these rules and React's internal bookkeeping breaks.

### Rule 1: Only call hooks at the top level

```jsx
// ❌ WRONG — conditional hook
function Component({ isLoggedIn }) {
  if (isLoggedIn) {
    const [user, setUser] = useState(null); // Different hook count between renders!
  }
}

// ❌ WRONG — hook inside loop
function List({ items }) {
  items.forEach(item => {
    const [selected, setSelected] = useState(false); // Unpredictable!
  });
}

// ✅ Always at the top level
function Component({ isLoggedIn }) {
  const [user, setUser] = useState(null); // Always called, every render
  if (isLoggedIn) {
    // Use the state here, don't declare it here
  }
}
```

### Rule 2: Only call hooks in React functions

```javascript
// ❌ WRONG — regular JS function
function getUser() {
  const [user] = useState(null); // Error! Not a component or custom hook
}

// ✅ OK — React component
function UserComponent() {
  const [user] = useState(null);
}

// ✅ OK — custom hook (must start with 'use')
function useUser() {
  const [user] = useState(null);
  return user;
}
```

---

## 12.2 useState — Deep Dive

```jsx
// Syntax
const [state, setState] = useState(initialValue);

// Initial value can be a function (lazy initialization — runs only once)
const [data, setData] = useState(() => {
  return JSON.parse(localStorage.getItem('data')) ?? [];
  // ↑ Only runs on first render, not every re-render
});
```

### Functional Updates (Important!)

```jsx
// ✅ Always use functional form when new state depends on old state
setCount(prev => prev + 1);

// This is critical in event handlers, timers, async functions
// where you might have a stale closure
useEffect(() => {
  const interval = setInterval(() => {
    setCount(c => c + 1); // ✅ Always gets latest count
    // setCount(count + 1); // ❌ count is stale (captured at effect setup)
  }, 1000);
  return () => clearInterval(interval);
}, []); // Empty deps — intentional
```

---

## 12.3 useEffect — Deep Dive

> The most important and most misused hook.

### What useEffect Does

`useEffect` synchronizes your component with an external system. It runs **after the component renders** and after the browser has painted.

```
Render → Paint to screen → useEffect runs
```

### Syntax and Dependency Array

```jsx
useEffect(() => {
  // Effect code
  return () => {
    // Cleanup (optional)
  };
}, [/* dependencies */]);
```

### The Three Modes

```jsx
// Mode 1: No dependency array — runs after EVERY render
useEffect(() => {
  document.title = `Count: ${count}`;
}); // ← no array; runs after every render

// Mode 2: Empty array — runs ONCE after mount
useEffect(() => {
  fetch('/api/data').then(setData);
  subscribeToEvents();
  return () => unsubscribeFromEvents(); // Cleanup on unmount
}, []); // ← empty array; once only

// Mode 3: With dependencies — runs when deps change
useEffect(() => {
  fetchUser(userId);
}, [userId]); // ← runs when userId changes
```

### Visual Execution Flow

```
Component Mounts:
  1. Component function runs (render phase)
  2. DOM updates
  3. Browser paints
  4. useEffect(fn, []) fires  ← mount effects

Dependency changes (e.g., userId changes):
  1. Component re-renders
  2. DOM updates
  3. Browser paints
  4. CLEANUP from previous effect runs ← important!
  5. New effect runs

Component Unmounts:
  1. CLEANUP function from last effect runs
  2. Component is removed from DOM
```

### The Cleanup Function

```jsx
// Crucial for preventing memory leaks
useEffect(() => {
  // Start something
  const subscription = subscribeToData(userId, setData);
  const timer = setInterval(refresh, 5000);

  // Return cleanup function
  return () => {
    subscription.unsubscribe(); // Stop subscription when userId changes or unmount
    clearInterval(timer);       // Stop timer
  };
}, [userId]);
```

**Without cleanup:**
```
userId changes:
  Old subscription (userId=1) still running
  New subscription (userId=2) starts
  → Two subscriptions active, data from wrong user may arrive!
```

**With cleanup:**
```
userId changes:
  Cleanup runs: subscription(userId=1).unsubscribe()
  New effect: subscription(userId=2) starts
  → Only one subscription, correct data
```

---

### Async in useEffect

```jsx
// ❌ Can't make useEffect's callback async directly
useEffect(async () => { // Returns a Promise, not cleanup fn!
  const data = await fetchData();
  setData(data);
}, []);

// ✅ Define async function inside
useEffect(() => {
  async function loadData() {
    try {
      const data = await fetchData();
      setData(data);
    } catch (error) {
      setError(error.message);
    }
  }
  loadData();
}, []);

// ✅ Or use IIFE
useEffect(() => {
  (async () => {
    const data = await fetchData();
    setData(data);
  })();
}, []);
```

---

### Missing Dependencies Warning

```jsx
// ⚠️ Warning: 'userId' used inside but not in deps
useEffect(() => {
  fetchUser(userId); // Uses userId
}, []); // ← Missing userId

// ✅ Fix: include all used values
useEffect(() => {
  fetchUser(userId);
}, [userId]);

// But what if you DON'T want to re-run when it changes?
// Use a ref to store the latest value
const userIdRef = useRef(userId);
useEffect(() => { userIdRef.current = userId; }, [userId]);

useEffect(() => {
  // Use ref inside — won't re-trigger effect
  fetchUser(userIdRef.current);
}, []); // Empty deps intentionally
```

---

### Infinite Loop in useEffect

```jsx
// ❌ Infinite loop: effect sets state → re-render → effect runs again
useEffect(() => {
  setData(processData(data)); // data changes → effect runs → sets data → data changes...
}, [data]); // ← data is both dependency AND being set

// ❌ Object as dependency (always "new" reference)
useEffect(() => {
  fetchConfig(options);
}, [options]); // options = {size: 10} — new object every render!

// ✅ Fix for object deps: destructure primitive values
useEffect(() => {
  fetchConfig({ size, page });
}, [size, page]); // Primitives compare by value
```

---

## 12.4 useRef — Deep Dive

`useRef` returns a mutable object `{ current: value }` that persists across renders **without causing re-renders**.

### Use 1: DOM References

```jsx
function TextInput() {
  const inputRef = useRef(null);

  function focusInput() {
    inputRef.current.focus(); // Direct DOM manipulation
  }

  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}
```

### Use 2: Storing Mutable Values (without re-render)

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef(null); // Store timer ID

  function start() {
    intervalRef.current = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
  }

  function stop() {
    clearInterval(intervalRef.current); // Access without re-render
  }

  return (
    <div>
      <p>{seconds}s</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```

### Use 3: Tracking Previous Values

```jsx
function usePrevious(value) {
  const prevRef = useRef();
  useEffect(() => {
    prevRef.current = value; // Update AFTER render
  }, [value]);
  return prevRef.current; // Returns previous value
}

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <p>Now: {count}, Before: {prevCount}</p>
  );
}
```

### ref vs state

| | useState | useRef |
|--|---------|--------|
| Triggers re-render? | ✅ Yes | ❌ No |
| Persists across renders? | ✅ Yes | ✅ Yes |
| Use for | UI data | DOM refs, timers, previous values |

---

## 12.5 useMemo — Deep Dive

`useMemo` memoizes the result of an **expensive computation**. It only recalculates when dependencies change.

```jsx
// Syntax
const memoizedValue = useMemo(() => compute(a, b), [a, b]);
```

### Without useMemo

```jsx
function ProductList({ products, filterText, sortBy }) {
  // This runs on EVERY render, even when only unrelated state changes
  const filteredAndSorted = products
    .filter(p => p.name.includes(filterText))
    .sort((a, b) => a[sortBy] - b[sortBy]);

  return filteredAndSorted.map(p => <Product key={p.id} {...p} />);
}
```

### With useMemo

```jsx
function ProductList({ products, filterText, sortBy }) {
  // Only recalculates when products, filterText, or sortBy change
  const filteredAndSorted = useMemo(() => {
    console.log('Computing...');
    return products
      .filter(p => p.name.includes(filterText))
      .sort((a, b) => a[sortBy] - b[sortBy]);
  }, [products, filterText, sortBy]);

  return filteredAndSorted.map(p => <Product key={p.id} {...p} />);
}
```

### When to Use useMemo

**Use it when:**
- Computation is genuinely expensive (large arrays, complex math)
- You've measured a performance problem with React DevTools
- Result is used as a dependency in useEffect/useCallback

**Don't use it when:**
- The computation is simple (string formatting, basic math)
- The component rarely re-renders
- Premature optimization adds complexity for no benefit

```jsx
// ❌ Premature — simple computation
const fullName = useMemo(() => `${first} ${last}`, [first, last]);

// ✅ Needed — sorting 10,000 items
const sortedData = useMemo(() => bigArray.sort(complexComparator), [bigArray]);
```

---

## 12.6 useCallback — Deep Dive

`useCallback` memoizes a **function reference**. It returns the same function object unless dependencies change.

```jsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

### Why Function References Matter

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // Every render creates a NEW function reference
  const handleDelete = (id) => deleteItem(id);

  return (
    <>
      <input value={name} onChange={e => setName(e.target.value)} />
      {/* Child re-renders every time name changes, even though handleDelete didn't change */}
      <ExpensiveChild onDelete={handleDelete} />
    </>
  );
}
```

When `name` changes, `Parent` re-renders, `handleDelete` is a new function → `ExpensiveChild` gets a new `onDelete` prop → `ExpensiveChild` re-renders unnecessarily.

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // Same function reference unless dependencies change
  const handleDelete = useCallback((id) => {
    deleteItem(id);
  }, []); // No deps — never changes

  return (
    <>
      <input value={name} onChange={e => setName(e.target.value)} />
      <ExpensiveChild onDelete={handleDelete} />
      {/* Now ExpensiveChild only re-renders when handleDelete changes */}
    </>
  );
}
```

**useCallback is most effective when paired with `React.memo`.**

---

## 12.7 React.memo — Deep Dive

`React.memo` is a Higher-Order Component that memoizes the rendered output of a component. It prevents re-renders when props haven't changed.

```jsx
// Without memo — re-renders whenever parent re-renders
function ExpensiveChild({ data, onClick }) {
  console.log('ExpensiveChild rendered');
  return <div onClick={onClick}>{data.map(renderItem)}</div>;
}

// With memo — only re-renders when props change (shallow comparison)
const ExpensiveChild = React.memo(function ExpensiveChild({ data, onClick }) {
  console.log('ExpensiveChild rendered');
  return <div onClick={onClick}>{data.map(renderItem)}</div>;
});
```

### Shallow Comparison

`React.memo` uses **shallow equality** — it compares each prop with `===`.

```jsx
// Primitives: compare by value ✅
{ count: 5 } → { count: 5 }  // Same value → no re-render

// Objects/Arrays: compare by reference ❌
{ items: [1,2,3] } → { items: [1,2,3] }  // New array reference → re-render!
```

This is why `useCallback` and `useMemo` are often needed alongside `React.memo`.

### The Full Pattern

```jsx
// 1. Memoize the child component
const ProductCard = React.memo(function ProductCard({ product, onAddToCart }) {
  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={() => onAddToCart(product.id)}>Add to Cart</button>
    </div>
  );
});

// 2. In parent: stabilize callback with useCallback
function ProductList({ products }) {
  const handleAddToCart = useCallback((id) => {
    addToCart(id); // setCart(prev => [...prev, id])
  }, []); // Or include setCart if needed

  return products.map(product => (
    <ProductCard
      key={product.id}
      product={product}
      onAddToCart={handleAddToCart} // Stable reference
    />
  ));
}
```

---

## 12.8 useContext — Deep Dive

Context lets you share data across the component tree without prop drilling.

### Step 1: Create Context

```jsx
import { createContext, useContext, useState } from 'react';

// Create context with a default value (used when no provider above)
const ThemeContext = createContext({
  theme: 'light',
  toggleTheme: () => {}
});
```

### Step 2: Provide Context

```jsx
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(t => t === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Wrap your app
function App() {
  return (
    <ThemeProvider>
      <Router>
        <PageContent />
      </Router>
    </ThemeProvider>
  );
}
```

### Step 3: Consume Context

```jsx
function ThemedButton({ children }) {
  // Any component in the tree can access this — no props drilling!
  const { theme, toggleTheme } = useContext(ThemeContext);

  return (
    <button
      className={`btn btn-${theme}`}
      onClick={toggleTheme}
    >
      {children}
    </button>
  );
}
```

### When Context Causes Performance Issues

Every component that `useContext` calls will re-render when the context value changes.

```jsx
// ❌ Problem: new object every render → all consumers re-render
function Provider({ children }) {
  const [user, setUser] = useState(null);
  return (
    <UserContext.Provider value={{ user, setUser }}> {/* New object each render! */}
      {children}
    </UserContext.Provider>
  );
}

// ✅ Fix: memoize the value
function Provider({ children }) {
  const [user, setUser] = useState(null);
  const value = useMemo(() => ({ user, setUser }), [user]); // Stable reference
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}
```

---

## 12.9 Custom Hooks — Deep Dive

Custom hooks are functions that start with `use` and can call other hooks. They extract and share **stateful logic** between components.

### Example 1: useFetch

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false; // Handle race conditions

    setLoading(true);
    setError(null);

    fetch(url)
      .then(res => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then(data => {
        if (!cancelled) setData(data);
      })
      .catch(err => {
        if (!cancelled) setError(err.message);
      })
      .finally(() => {
        if (!cancelled) setLoading(false);
      });

    return () => { cancelled = true; }; // Cleanup: cancel on unmount/url change
  }, [url]);

  return { data, loading, error };
}

// Usage — clean and reusable
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);

  if (loading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  return <Profile user={user} />;
}
```

### Example 2: useLocalStorage

```jsx
function useLocalStorage(key, initialValue) {
  const [stored, setStored] = useState(() => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = useCallback((value) => {
    try {
      const val = value instanceof Function ? value(stored) : value;
      setStored(val);
      localStorage.setItem(key, JSON.stringify(val));
    } catch (error) {
      console.error(error);
    }
  }, [key, stored]);

  return [stored, setValue];
}

// Usage
function App() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [user, setUser] = useLocalStorage('user', null);
}
```

### Example 3: useDebounce

```jsx
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Usage — search input that doesn't fire on every keystroke
function SearchBox() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 300);

  useEffect(() => {
    if (debouncedQuery) search(debouncedQuery); // Only fires 300ms after typing stops
  }, [debouncedQuery]);

  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

---

### 📝 QUICK REVISION — Section 12

- Hooks extract reusable stateful logic into functions
- Rule 1: Only call at top level (no conditionals/loops)
- Rule 2: Only call in React functions or custom hooks
- useState: state + setter; use functional form for derived state
- useEffect: sync with external systems; 3 modes (always/once/on-change); always clean up
- useRef: mutable value that doesn't cause re-render; DOM refs, timers
- useMemo: memoize expensive computation; returns cached value
- useCallback: memoize function reference; prevents child re-renders
- React.memo: skip re-render when props unchanged (shallow compare)
- useContext: consume context; re-renders on every context change
- Custom hooks: functions starting with `use` that encapsulate reusable stateful logic

---

# 13. COMPONENT LIFECYCLE

## 13.1 The Three Phases

Every React component goes through three lifecycle phases:

```
MOUNT           → Component added to DOM
   ↓
UPDATE          → Props or state change
   ↓
UNMOUNT         → Component removed from DOM
```

## 13.2 Lifecycle with Hooks (Functional Components)

```
MOUNT:
  Component function runs
  DOM is created
  ↓
  useLayoutEffect fires (before paint)
  ↓
  Browser paints
  ↓
  useEffect fires (after paint)

UPDATE (props/state change):
  Component function re-runs
  DOM updates
  ↓
  useLayoutEffect cleanup from prev render
  useLayoutEffect fires
  ↓
  Browser paints
  ↓
  useEffect cleanup from prev render
  useEffect fires

UNMOUNT:
  useLayoutEffect cleanup fires
  useEffect cleanup fires
  DOM node removed
```

## 13.3 Class Lifecycle vs Hooks Mapping

| Class Method | Hook Equivalent |
|---|---|
| `constructor` | `useState(initialValue)` |
| `componentDidMount` | `useEffect(() => { ... }, [])` |
| `componentDidUpdate` | `useEffect(() => { ... }, [deps])` |
| `componentWillUnmount` | `useEffect(() => { return cleanup; }, [])` |
| `shouldComponentUpdate` | `React.memo` + `useMemo` |
| `getSnapshotBeforeUpdate` | `useLayoutEffect` (before paint) |
| `getDerivedStateFromProps` | Compute during render, no hook needed |

```jsx
// Equivalent to componentDidMount
useEffect(() => {
  // Mount logic
}, []);

// Equivalent to componentDidUpdate for 'count'
useEffect(() => {
  // Runs when count changes
}, [count]);

// Equivalent to componentWillUnmount
useEffect(() => {
  return () => {
    // Cleanup on unmount
  };
}, []);

// Equivalent to componentDidMount + componentDidUpdate (every render)
useEffect(() => {
  // Runs after every render
});
```

## 13.4 useLayoutEffect vs useEffect

```
useEffect:    async, runs AFTER browser paint
              → No visual flash, doesn't block paint
              → Use for most side effects

useLayoutEffect: sync, runs BEFORE browser paint
                 → Can block paint if slow!
                 → Use when you need to measure/modify DOM before user sees it
```

```jsx
// useLayoutEffect — reads layout, prevents flash
function Tooltip({ targetRef }) {
  const tooltipRef = useRef();

  useLayoutEffect(() => {
    // Measure target position BEFORE browser paints
    const rect = targetRef.current.getBoundingClientRect();
    // Position tooltip next to target
    tooltipRef.current.style.top = `${rect.bottom}px`;
    tooltipRef.current.style.left = `${rect.left}px`;
  }, []);

  return <div ref={tooltipRef} className="tooltip">...</div>;
}
```

# 14. REACT ROUTER

## 14.1 SPA Routing Concept

In a SPA, navigation doesn't reload the page. React Router:
- Intercepts URL changes
- Matches URL to component
- Renders that component without page reload

```bash
npm install react-router-dom
```

---

## 14.2 Core Components

```jsx
import {
  BrowserRouter,
  Routes,
  Route,
  Link,
  NavLink,
  useNavigate,
  useParams,
  useLocation,
  Outlet
} from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users" element={<Users />} />
        <Route path="/users/:id" element={<UserDetail />} />
        <Route path="*" element={<NotFound />} />  {/* 404 */}
      </Routes>
    </BrowserRouter>
  );
}
```

---

## 14.3 Navigation

```jsx
// Link — declarative navigation (renders <a> tag)
<Link to="/about">About</Link>

// NavLink — Link with active styling
<NavLink
  to="/about"
  className={({ isActive }) => isActive ? 'nav-link active' : 'nav-link'}
>
  About
</NavLink>

// useNavigate — programmatic navigation
function LoginForm() {
  const navigate = useNavigate();

  async function handleLogin(data) {
    await login(data);
    navigate('/dashboard');             // Go to route
    navigate(-1);                       // Go back
    navigate('/profile', { replace: true }); // Replace history entry
  }
}
```

---

## 14.4 Dynamic Routes and URL Params

```jsx
// Route definition
<Route path="/users/:userId/posts/:postId" element={<PostDetail />} />

// Component reads params
function PostDetail() {
  const { userId, postId } = useParams();
  const { data: post } = useFetch(`/api/users/${userId}/posts/${postId}`);
  return <div>{post?.title}</div>;
}
```

### Query Params

```jsx
// URL: /products?category=electronics&sort=price

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();

  const category = searchParams.get('category');
  const sort = searchParams.get('sort');

  function handleSortChange(newSort) {
    setSearchParams({ category, sort: newSort });
  }
}
```

---

## 14.5 Nested Routes

```jsx
// Parent route renders <Outlet /> where children appear
function DashboardLayout() {
  return (
    <div>
      <Sidebar />
      <main>
        <Outlet />  {/* Child route renders here */}
      </main>
    </div>
  );
}

// Route config
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<DashboardHome />} />         {/* /dashboard */}
  <Route path="analytics" element={<Analytics />} /> {/* /dashboard/analytics */}
  <Route path="settings" element={<Settings />} />   {/* /dashboard/settings */}
</Route>
```

---

## 14.6 Protected Routes

```jsx
function ProtectedRoute({ children }) {
  const { user } = useAuth();
  const location = useLocation();

  if (!user) {
    // Redirect to login, remember where they were trying to go
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
}

// Usage
<Route
  path="/dashboard"
  element={
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  }
/>

// After login, redirect to intended page
function LoginPage() {
  const navigate = useNavigate();
  const location = useLocation();

  async function handleLogin(credentials) {
    await login(credentials);
    const from = location.state?.from?.pathname || '/dashboard';
    navigate(from, { replace: true }); // Go back where they came from
  }
}
```

---

# 15. API CALLING & DATA FETCHING

## 15.1 The Standard Pattern

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    setLoading(true);
    setError(null);

    fetch(`/api/users/${userId}`)
      .then(res => {
        if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
        return res.json();
      })
      .then(data => {
        if (!cancelled) setUser(data);
      })
      .catch(err => {
        if (!cancelled) setError(err.message);
      })
      .finally(() => {
        if (!cancelled) setLoading(false);
      });

    return () => { cancelled = true; }; // Cleanup
  }, [userId]);

  if (loading) return <Skeleton />;
  if (error) return <ErrorBanner message={error} />;
  if (!user) return null;
  return <Profile user={user} />;
}
```

---

## 15.2 Using Axios

```bash
npm install axios
```

```jsx
import axios from 'axios';

// Create configured instance
const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000,
  headers: { 'Content-Type': 'application/json' }
});

// Request interceptor — attach auth token
api.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Response interceptor — handle 401 globally
api.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      logout(); // Token expired
    }
    return Promise.reject(error);
  }
);

// Usage in component
useEffect(() => {
  api.get(`/users/${userId}`)
    .then(res => setUser(res.data)) // axios wraps response in .data
    .catch(err => setError(err.message));
}, [userId]);
```

---

## 15.3 React Query (Recommended for Production)

React Query handles caching, background refetching, loading/error states, and much more.

```bash
npm install @tanstack/react-query
```

```jsx
import { QueryClient, QueryClientProvider, useQuery, useMutation } from '@tanstack/react-query';

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <UserProfile userId={1} />
    </QueryClientProvider>
  );
}

function UserProfile({ userId }) {
  // Dramatically simpler than manual useEffect + state
  const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', userId],    // Cache key
    queryFn: () => fetchUser(userId),
    staleTime: 5 * 60 * 1000,     // Cache for 5 minutes
  });

  if (isLoading) return <Spinner />;
  if (error) return <Error />;
  return <Profile user={user} />;
}

// Mutation — POST/PUT/DELETE
function CreatePostForm() {
  const mutation = useMutation({
    mutationFn: (newPost) => api.post('/posts', newPost),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts'] }); // Refetch posts
    }
  });

  return (
    <form onSubmit={e => {
      e.preventDefault();
      mutation.mutate({ title: 'New Post', content: '...' });
    }}>
      <button disabled={mutation.isPending}>
        {mutation.isPending ? 'Saving...' : 'Save Post'}
      </button>
    </form>
  );
}
```

---

# 16. STATE MANAGEMENT

## 16.1 When to Use What

```
Local state (useState)
  → One component needs it
  → Simple toggle, form field, counter

Lifted state (useState in parent)
  → A few sibling components share it
  → Data flows down via props

Context API
  → Many components need it, avoiding prop drilling
  → Theme, auth, language, user preferences
  → NOT for high-frequency updates (performance)

Redux Toolkit / Zustand
  → Large complex state (shopping cart, multi-step forms)
  → Frequent updates across many components
  → Time-travel debugging needed

React Query / SWR
  → Server state (data from API)
  → Caching, background refetch, optimistic updates
```

---

## 16.2 Context API (Full Pattern)

```jsx
// auth-context.jsx
const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Check if user is logged in on mount
    checkAuth().then(user => {
      setUser(user);
      setLoading(false);
    });
  }, []);

  const login = useCallback(async (credentials) => {
    const user = await loginApi(credentials);
    setUser(user);
  }, []);

  const logout = useCallback(() => {
    logoutApi();
    setUser(null);
  }, []);

  const value = useMemo(
    () => ({ user, loading, login, logout }),
    [user, loading, login, logout]
  );

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook for consuming auth
export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
}
```

---

## 16.3 Zustand (Simple Global State)

```bash
npm install zustand
```

```jsx
import { create } from 'zustand';

// Define store
const useCartStore = create((set, get) => ({
  items: [],

  addItem: (product) => set(state => ({
    items: [...state.items, product]
  })),

  removeItem: (id) => set(state => ({
    items: state.items.filter(item => item.id !== id)
  })),

  getTotal: () => get().items.reduce((sum, item) => sum + item.price, 0),

  clearCart: () => set({ items: [] })
}));

// In any component — no context provider needed!
function CartIcon() {
  const items = useCartStore(state => state.items); // Subscribe to specific slice
  return <span>{items.length}</span>;
}

function ProductCard({ product }) {
  const addItem = useCartStore(state => state.addItem);
  return <button onClick={() => addItem(product)}>Add to Cart</button>;
}
```

---

## 16.4 Redux Toolkit (Complex State)

```bash
npm install @reduxjs/toolkit react-redux
```

```jsx
// store/cartSlice.js
import { createSlice } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [], status: 'idle' },
  reducers: {
    addItem: (state, action) => {
      // Immer allows direct mutation inside reducers!
      state.items.push(action.payload);
    },
    removeItem: (state, action) => {
      state.items = state.items.filter(i => i.id !== action.payload);
    }
  }
});

export const { addItem, removeItem } = cartSlice.actions;
export default cartSlice.reducer;

// store/index.js
import { configureStore } from '@reduxjs/toolkit';
const store = configureStore({
  reducer: { cart: cartReducer, user: userReducer }
});

// Usage
import { useSelector, useDispatch } from 'react-redux';

function Cart() {
  const items = useSelector(state => state.cart.items);
  const dispatch = useDispatch();

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.name}
          <button onClick={() => dispatch(removeItem(item.id))}>Remove</button>
        </li>
      ))}
    </ul>
  );
}
```

---

# 17. PERFORMANCE OPTIMIZATION

## 17.1 Understanding Re-renders

A component re-renders when:
1. Its **state** changes
2. Its **props** change
3. Its **parent** re-renders (even if props are the same!)
4. Its **context** changes

The goal isn't to eliminate re-renders (they're cheap) — it's to eliminate **unnecessary, expensive** re-renders.

---

## 17.2 The Optimization Toolkit

```
Problem: Component re-renders unnecessarily
  → React.memo: skip re-render when props unchanged

Problem: Expensive calculation runs every render
  → useMemo: cache result until dependencies change

Problem: Callback function creates new reference every render
  → useCallback: stabilize function reference

Problem: Large bundle loads slowly
  → React.lazy + Suspense: code splitting

Problem: Rendering 10,000 list items
  → Virtual list (react-window / react-virtual): only render visible items
```

---

## 17.3 Code Splitting with React.lazy

```jsx
import { lazy, Suspense } from 'react';

// These components are only loaded when needed
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Analytics = lazy(() => import('./pages/Analytics'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/analytics" element={<Analytics />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

Each route becomes a separate JS chunk — users only download what they visit.

---

## 17.4 List Virtualization

Rendering 10,000 `<div>` elements creates 10,000 DOM nodes — slow!

Virtualization renders only the **visible items** (say, 20 at a time).

```bash
npm install @tanstack/react-virtual
```

```jsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
  const parentRef = useRef();

  const rowVirtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,  // Row height in px
  });

  return (
    <div ref={parentRef} style={{ height: '500px', overflow: 'auto' }}>
      <div style={{ height: `${rowVirtualizer.getTotalSize()}px`, position: 'relative' }}>
        {rowVirtualizer.getVirtualItems().map(virtualItem => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: 0,
              transform: `translateY(${virtualItem.start}px)`,
              height: `${virtualItem.size}px`
            }}
          >
            {items[virtualItem.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## 17.5 Profiling with React DevTools

1. Install React DevTools browser extension
2. Open DevTools → "Profiler" tab
3. Click "Record", interact with your app, stop recording
4. Look for components that render often with long durations

```
Reading the flame chart:
  Wide bar = long render time (investigate)
  Tall stack = deep component tree (normal)
  Gray = not rendered in this commit
  Yellow/orange = re-rendered (check why)
```

---

## 17.6 Performance Checklist

```
□ Are you using production build? (not development)
□ Are keys stable? (not using index for dynamic lists)
□ Are callbacks memoized? (useCallback for child props)
□ Are heavy computations cached? (useMemo)
□ Are big components memoized? (React.memo)
□ Is code splitting used for routes/heavy components?
□ Are large lists virtualized?
□ Are images optimized and lazy loaded?
□ Are unnecessary useEffect deps included?
```

---

# 18. REACT INTERNALS & ADVANCED TOPICS

## 18.1 Concurrent Rendering (React 18)

Concurrent Mode allows React to work on multiple versions of the UI simultaneously, pausing and resuming work as needed.

```jsx
// useTransition — mark updates as "non-urgent"
function SearchPage() {
  const [input, setInput] = useState('');
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    setInput(e.target.value); // Urgent: update input immediately

    startTransition(() => {
      setQuery(e.target.value); // Non-urgent: can be deferred
    });
  }

  return (
    <>
      <input value={input} onChange={handleChange} />
      {isPending && <Spinner />}
      <SearchResults query={query} />  {/* May have stale results briefly */}
    </>
  );
}
```

---

## 18.2 Suspense

Suspense lets you declaratively handle loading states:

```jsx
// Data fetching with Suspense (React Query / Relay support this)
function App() {
  return (
    <ErrorBoundary fallback={<ErrorPage />}>
      <Suspense fallback={<PageSkeleton />}>
        <UserProfile />    {/* Suspends while loading */}
      </Suspense>
    </ErrorBoundary>
  );
}

// Lazy loading with Suspense
const HeavyComponent = lazy(() => import('./HeavyComponent'));

<Suspense fallback={<Spinner />}>
  <HeavyComponent />
</Suspense>
```

---

## 18.3 Server-Side Rendering (SSR) vs Client-Side Rendering (CSR)

```
CSR (React default):
  1. Browser downloads empty HTML
  2. Browser downloads JS bundle
  3. React runs in browser, builds DOM
  4. Page is visible + interactive

Pros: Instant subsequent navigation, rich interactions
Cons: Slow first paint, bad SEO (empty HTML initially)

SSR (Next.js):
  1. Browser requests page
  2. Server runs React, produces HTML
  3. Browser receives HTML with content
  4. React hydrates (attaches event handlers)

Pros: Fast first paint, good SEO
Cons: More server infrastructure, slower navigation (full request)
```

### Hydration

Hydration = process of attaching React's event listeners and state to server-rendered HTML.

```
Server sends HTML:
  <div id="root">
    <h1>Hello World</h1>
    <button>Click me</button>
  </div>

React "hydrates":
  Reads existing DOM
  Matches it to component tree
  Attaches onClick to <button>
  Component is now interactive
```

---

## 18.4 React Server Components (RSC)

Available in Next.js App Router. Server Components run **exclusively on the server** — no JS sent to client.

```jsx
// ServerComponent.jsx (runs on server, zero JS bundle cost)
async function UserList() {
  const users = await db.query('SELECT * FROM users'); // Direct DB access!
  return (
    <ul>
      {users.map(u => <li key={u.id}>{u.name}</li>)}
    </ul>
  );
}

// ClientComponent.jsx (must have 'use client' directive)
'use client';
function LikeButton({ postId }) {
  const [liked, setLiked] = useState(false); // useState works here
  return <button onClick={() => setLiked(l => !l)}>Like</button>;
}
```

---

# 19. ERROR HANDLING

## 19.1 Error Boundaries

Error boundaries catch JavaScript errors during rendering and display a fallback UI.

```jsx
import { Component } from 'react';

class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log to error monitoring service
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div>
          <h2>Something went wrong.</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary fallback={<ErrorPage />}>
  <RiskyComponent />
</ErrorBoundary>
```

> Note: Error boundaries only work in class components (as of React 18). Use a library like `react-error-boundary` for a hook-based approach.

```jsx
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div>
      <p>Error: {error.message}</p>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

<ErrorBoundary FallbackComponent={ErrorFallback}>
  <App />
</ErrorBoundary>
```

---

# 20. FOLDER STRUCTURE & CLEAN ARCHITECTURE

## 20.1 Scalable Project Structure

```
src/
├── components/           # Reusable UI components (no business logic)
│   ├── ui/               # Generic UI: Button, Input, Modal, Card
│   │   ├── Button/
│   │   │   ├── Button.jsx
│   │   │   ├── Button.module.css
│   │   │   └── Button.test.jsx
│   │   └── ...
│   └── layout/           # Layout components: Header, Footer, Sidebar
│
├── features/             # Feature-based modules (business logic)
│   ├── auth/
│   │   ├── components/   # Auth-specific UI
│   │   ├── hooks/        # useAuth, usePermissions
│   │   ├── services/     # authService.js (API calls)
│   │   └── store/        # Auth state (Zustand slice, Redux slice)
│   ├── products/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── services/
│   └── cart/
│
├── pages/                # Route-level components (assemble features)
│   ├── Home.jsx
│   ├── ProductDetail.jsx
│   └── Checkout.jsx
│
├── hooks/                # Shared custom hooks
│   ├── useDebounce.js
│   ├── useFetch.js
│   └── useLocalStorage.js
│
├── services/             # API layer (base config, shared calls)
│   └── api.js
│
├── utils/                # Pure utility functions
│   ├── formatDate.js
│   ├── formatCurrency.js
│   └── validators.js
│
├── context/              # Global contexts (if not using Zustand/Redux)
├── constants/            # App-wide constants, enums
├── types/                # TypeScript types/interfaces
├── App.jsx
└── main.jsx
```

---

# 21. COMMON REACT INTERVIEW QUESTIONS

## Q1: What is the Virtual DOM and how does it work?

**Answer:**
The Virtual DOM is an in-memory JavaScript object representation of the real DOM. When a component's state or props change, React:
1. Renders a new Virtual DOM tree
2. Compares it to the previous Virtual DOM (reconciliation/diffing)
3. Calculates the minimum set of real DOM updates needed
4. Applies those changes in a single batch

This is faster than directly manipulating the real DOM, because DOM operations are expensive (they trigger layout recalculation and repaints), while JavaScript object manipulation is cheap.

*Follow-up: "What algorithm does React use for diffing?"*
React's diffing runs in O(n) by making two assumptions: elements of different types produce different trees, and keys uniquely identify list items across renders.

---

## Q2: Explain the difference between useState and useRef.

| | useState | useRef |
|--|---------|--------|
| Triggers re-render? | ✅ Yes | ❌ No |
| Value persists? | ✅ Yes | ✅ Yes |
| When to use | UI data that should update the display | DOM refs, timers, previous values |

`useState` is for data that affects the UI. `useRef` is for mutable values that shouldn't trigger re-renders — like storing a timer ID, the previous value of a variable, or a direct DOM reference.

---

## Q3: What happens when you call setState?

React doesn't immediately update the DOM. It:
1. Enqueues the update
2. Schedules a re-render
3. Batches multiple setState calls in the same event handler
4. During the re-render, calls the component function
5. Reconciles the new virtual DOM against the previous
6. Applies minimal DOM changes in the commit phase

State is only updated on the *next* render, not immediately after calling setState.

---

## Q4: What are the rules of hooks and why do they exist?

**The rules:**
1. Only call hooks at the top level (not inside conditions/loops)
2. Only call hooks in React functions (components or custom hooks)

**Why?** React stores hooks in an ordered linked list, one per component instance. The order of hook calls must be identical on every render so React can correctly match each hook call to its stored value. If you call a hook conditionally, the count changes between renders and React reads wrong values from the wrong slots.

---

## Q5: Explain useEffect dependencies. What happens with an empty array vs no array vs specific deps?

```
useEffect(fn)           → runs after EVERY render
useEffect(fn, [])       → runs once after mount
useEffect(fn, [a, b])   → runs after mount + when a or b changes
```

The dependency array tells React when to re-run the effect. React compares deps using `Object.is` (similar to `===`). If any dep changed, the cleanup from the previous effect runs, then the new effect runs.

---

## Q6: What is reconciliation?

Reconciliation is React's algorithm for determining what changed between renders and updating the real DOM efficiently.

When state/props change:
1. React re-renders the component (calls the function)
2. Gets a new React element tree
3. Compares it to the previous tree using the diffing algorithm
4. Produces a list of "effects" — DOM operations to perform
5. Applies them in the commit phase

React's O(n) diffing uses two heuristics: different element types cause full subtree replacement; keys identify list items for efficient reordering.

---

## Q7: What is Fiber?

Fiber is React's internal reconciler architecture (since React 16). It represents each piece of work as a "fiber" — a JavaScript object with information about the component, its state, effects, and relationships to other fibers.

Key benefit: Fiber makes rendering **interruptible**. In React 18's Concurrent Mode, React can pause rendering mid-tree, handle higher-priority updates (like user input), then resume where it left off. This prevents the UI from feeling janky during heavy renders.

---

## Q8: Controlled vs Uncontrolled components?

**Controlled:** React state is the single source of truth for the input's value.
```jsx
<input value={state} onChange={e => setState(e.target.value)} />
```

**Uncontrolled:** The DOM is the source of truth; access value via ref.
```jsx
<input ref={inputRef} defaultValue="initial" />
// read: inputRef.current.value
```

Controlled is recommended for most cases — instant validation, programmatic value changes, consistent state. Uncontrolled is simpler for file inputs or integrating with non-React code.

---

## Q9: How do you prevent unnecessary re-renders?

1. **React.memo** — wrap component, skip re-render when props unchanged (shallow compare)
2. **useCallback** — memoize callbacks passed as props, so children don't see new references
3. **useMemo** — memoize expensive computations
4. **State colocation** — keep state close to where it's used, avoid lifting state unnecessarily high
5. **Context splitting** — separate frequently-changing context from rarely-changing context
6. **Keys** — use stable, unique keys in lists

---

## Q10: What is lifting state up?

When two sibling components need to share state, you "lift" that state up to their closest common ancestor, which then passes it down via props and passes callbacks for modification.

```jsx
function Parent() {
  const [count, setCount] = useState(0); // Lifted here

  return (
    <>
      <DisplayCount count={count} />
      <IncrementButton onIncrement={() => setCount(c => c + 1)} />
    </>
  );
}
```

---

# 22. COMMON REACT PITFALLS

## Pitfall 1: Infinite Re-render Loop

```jsx
// ❌ Sets state during render phase → infinite loop
function Component() {
  const [count, setCount] = useState(0);
  setCount(1); // Every render → triggers re-render → repeat forever!
}

// ✅ Fix: use useEffect for side effects after render
useEffect(() => { setCount(1); }, []); // Only on mount
```

## Pitfall 2: Object/Array as useEffect Dependency

```jsx
// ❌ New object reference every render → infinite loop
useEffect(() => {
  fetchData(options);
}, [options]); // options = { page: 1, size: 10 } — new reference each render!

// ✅ Fix: use primitive values
useEffect(() => {
  fetchData({ page, size });
}, [page, size]); // Primitives compare by value
```

## Pitfall 3: Stale Closure

```jsx
// ❌ Captures count=0 in closure, never updates
useEffect(() => {
  setInterval(() => {
    setCount(count + 1); // count is always 0 (stale)
  }, 1000);
}, []);

// ✅ Fix: functional update form
useEffect(() => {
  const timer = setInterval(() => {
    setCount(c => c + 1); // Always gets fresh value
  }, 1000);
  return () => clearInterval(timer);
}, []);
```

## Pitfall 4: Direct State Mutation

```jsx
// ❌ Mutates existing array — React doesn't detect change
const [items, setItems] = useState([]);
items.push(newItem); // Mutation!
setItems(items);     // Same reference → no re-render

// ✅ Create new array
setItems(prev => [...prev, newItem]);
```

## Pitfall 5: Missing Cleanup

```jsx
// ❌ Memory leak — subscription persists after unmount
useEffect(() => {
  const sub = eventEmitter.subscribe('event', handler);
  // No cleanup!
}, []);

// ✅ Always clean up subscriptions, timers, event listeners
useEffect(() => {
  const sub = eventEmitter.subscribe('event', handler);
  return () => sub.unsubscribe();
}, []);
```

## Pitfall 6: Using 0 with && Operator

```jsx
// ❌ Renders "0" in DOM when items.length = 0
{items.length && <List items={items} />}

// ✅ Explicit boolean check
{items.length > 0 && <List items={items} />}
```

## Pitfall 7: Not Using Keys (or Using Index)

```jsx
// ❌ Index as key for dynamic list
{items.map((item, index) => <Item key={index} {...item} />)}

// ✅ Stable unique ID
{items.map(item => <Item key={item.id} {...item} />)}
```

---

# 23. PROJECT-BASED REACT QUESTIONS

## Q: "How did you optimize performance in your React app?"

**Strong Answer:**
"On our e-commerce product listing page, we had a noticeable lag when typing in the search box because it was re-rendering 200+ product cards on every keystroke.

I profiled it with React DevTools Profiler and found two issues:
1. Each `ProductCard` was re-rendering even when its own data hadn't changed — because the parent re-rendered with new search state
2. The filter + sort computation was running on every render

I fixed this by:
- Wrapping `ProductCard` in `React.memo` to skip re-renders when product data is unchanged
- Memoizing the `onAddToCart` callback with `useCallback` so `ProductCard` didn't see new prop references
- Moving the filter/sort into `useMemo` so it only recalculated when `searchText` or `sortBy` changed

The result was a 4x reduction in render time from ~80ms to ~20ms per keystroke, measured in the profiler."

---

## Q: "How did you manage state in your app?"

**Strong Answer:**
"I used three layers of state:

1. **Local state** with `useState` for UI-specific things like dropdown open/close, modal visibility, and form inputs
2. **React Query** for server state — it handled caching, background refetch, and loading/error states for API data, which removed a lot of manual useEffect + loading/error state boilerplate
3. **Zustand** for shared client state like the shopping cart and user preferences that needed to be accessed in many unrelated components without prop drilling

I avoided using Redux because our state requirements didn't need time-travel debugging, and Zustand gave us 90% of the capability with 10% of the boilerplate."

---

## Q: "How did you structure your components?"

**Strong Answer:**
"I used a feature-based folder structure with a separation between UI components and feature components.

Pure UI components (Button, Modal, Input) live in `src/components/ui` — they have no business logic, accept props only, and are fully reusable.

Feature components are grouped by domain: `src/features/auth`, `src/features/products`, etc. Each feature has its own components, hooks, and service files.

Pages just assemble features — they're thin components that compose feature components together.

This made it easy to find code, isolate changes, and reuse UI components across features."

---

# 24. BUILD A COMPLETE MENTAL MODEL OF REACT

## The Core Mental Model

```
React = UI = f(state)

Your components describe WHAT the UI looks like for any given state.
React figures out HOW to make it so, efficiently.
```

## The Complete Data Flow

```
User Action (click, input, etc.)
         ↓
Event Handler fires
         ↓
State Update (setState)
         ↓
React schedules re-render
         ↓
Component function runs again
         ↓
New React Element tree produced
         ↓
Reconciliation (diff old vs new)
         ↓
Minimal DOM updates applied
         ↓
useLayoutEffect fires
         ↓
Browser paints
         ↓
useEffect fires
         ↓
User sees updated UI
```

## The Component Tree Mental Model

```
Every React app is a tree of components.
Each node = one component instance.
Each component = function that maps props+state → UI.

State lives at nodes. Data flows down via props.
Events flow up via callbacks.

When a node's state changes:
  → That node re-renders
  → All its children potentially re-render (unless memoized)
  → React diffs the subtree
  → Minimal DOM changes applied
```

## Connecting the Hooks

```
useState     → "I need some memory that persists and causes UI updates"
useEffect    → "After rendering, sync with the outside world"
useRef       → "I need memory that persists but NOT causing UI updates"
useMemo      → "Cache this expensive calculation until deps change"
useCallback  → "Cache this function reference until deps change"
React.memo   → "Skip re-rendering if my props haven't changed"
useContext   → "Reach up the tree to get data without prop drilling"
Custom hooks → "Extract this stateful logic to share it"
```

## The Decision Tree for State Location

```
Does only this component need the state?
  YES → useState in this component

Do sibling components need it?
  YES → Lift to parent, pass via props

Do many components across the tree need it?
  YES → Context API (for infrequent updates) OR Zustand/Redux (for frequent updates)

Is it server data (from an API)?
  YES → React Query / SWR (specialized for this)
```

---

# 25. 5-MINUTE RAPID REVISION SHEET

> Read this before walking into a frontend interview.

## Core Concept
```
React = declarative UI library
UI = f(state)  ← everything else follows from this
Virtual DOM → Reconciliation → Minimal real DOM updates
```

## Hooks Cheatsheet
```
useState(init)        → [value, setter]
useEffect(fn, deps)   → side effects after render; cleanup with return fn
useRef(init)          → { current: val }; no re-render; DOM refs
useMemo(fn, deps)     → cached computed value
useCallback(fn, deps) → cached function reference
useContext(ctx)       → consume context value
```

## useEffect Modes
```
useEffect(fn)          → every render
useEffect(fn, [])      → mount only
useEffect(fn, [dep])   → mount + when dep changes
return () => cleanup   → runs before next effect or on unmount
```

## Rendering Rules
```
Re-renders happen when: state changes, props change, parent re-renders, context changes
Prevent with: React.memo, useMemo, useCallback, state colocation
```

## Keys Rule
```
Keys = stable unique IDs for list items
Never use index for dynamic lists (items added/removed/reordered)
Wrong keys = state bugs + unnecessary DOM updates
```

## Reconciliation
```
React diffs old vDOM vs new vDOM
Different element types → full subtree rebuild (state lost)
Same type → update props in place (state preserved)
Keys → track items across reorders
```

## State Immutability
```
Objects: { ...old, field: newVal }
Arrays (add): [...old, newItem]
Arrays (remove): old.filter(i => i.id !== id)
Arrays (update): old.map(i => i.id === id ? {...i, field: newVal} : i)
NEVER mutate then set — same reference = no re-render
```

## Common Traps
```
0 && <X>           → renders "0"; use count > 0 && <X>
async useEffect    → define async fn inside, not on the callback
stale closure      → use functional setState: setCount(c => c + 1)
object in deps     → infinite loop; destructure primitives
mutating state     → no re-render; always return new reference
missing cleanup    → memory leaks; always clean timers/subscriptions
```

## Lifecycle (Hooks)
```
Mount:   render → useLayoutEffect → paint → useEffect
Update:  render → old LayoutEffect cleanup → useLayoutEffect → paint → old Effect cleanup → useEffect
Unmount: old LayoutEffect cleanup → old Effect cleanup → DOM removed
```

## Performance Hierarchy
```
1. Fix first: wrong keys, unnecessary state lifting
2. Then: React.memo + useCallback + useMemo
3. Then: Code splitting with React.lazy + Suspense
4. Then: List virtualization (react-virtual)
5. Always: Profile with React DevTools BEFORE optimizing
```

## Interview Confidence Phrases
```
"React batches state updates in event handlers for performance"
"Keys help React track list item identity during reconciliation"
"Fiber made rendering interruptible — React can pause for higher-priority work"
"useEffect's dependency array controls when the effect re-runs"
"Functional updates prevent stale closure bugs with async/timer state"
"React.memo + useCallback work together — memo for the child, callback for stable props"
```

---

*End of React Engineering Handbook. Build something great.* ⚛️
