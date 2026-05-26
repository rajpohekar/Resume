# ⚛️ Top 20 React Interview Questions for Fresher SDE

> Most asked questions in fresher/entry-level React interviews. Study these thoroughly before your interview.

---

## Q1. What is React and why is it used?

**Answer:**
React is a JavaScript library built by Meta for building user interfaces. It follows the core principle:

```
UI = f(state)
```

The UI is always a function of state — when state changes, React automatically updates the UI. It solves problems like manual DOM manipulation, state/UI getting out of sync, and poor component reusability.

**Key points to mention:** declarative UI, component-based architecture, Virtual DOM, unidirectional data flow.

---

## Q2. What is the Virtual DOM and how does it work?

**Answer:**
The Virtual DOM is an in-memory JavaScript object that mirrors the real DOM. When state/props change:

1. React creates a **new Virtual DOM tree**
2. Compares it with the old one (**diffing/reconciliation**)
3. Calculates the **minimum set of changes** needed
4. Applies only those changes to the **real DOM**

This is faster because JS object operations are ~100x cheaper than real DOM operations (which trigger reflows and repaints).

```javascript
// Virtual DOM is just a plain JS object:
{
  type: 'div',
  props: { className: 'box' },
  children: [{ type: 'h1', props: {}, children: ['Hello'] }]
}
```

---

## Q3. What is JSX?

**Answer:**
JSX (JavaScript XML) is syntactic sugar that looks like HTML but compiles to `React.createElement()` calls.

```jsx
// What you write:
const element = <h1 className="title">Hello</h1>;

// What Babel compiles it to:
const element = React.createElement('h1', { className: 'title' }, 'Hello');
```

**Key JSX rules:**
- Must return a single root element (use `<>` Fragment if needed)
- All tags must be closed (`<img />`)
- Use `className` instead of `class`, `htmlFor` instead of `for`
- JavaScript expressions go inside `{}`

---

## Q4. What are Components? What is the difference between functional and class components?

**Answer:**
A component is a JavaScript function (or class) that returns JSX. It's the fundamental building block of React UIs.

| Aspect | Functional Component | Class Component |
|--------|---------------------|-----------------|
| Syntax | Plain JS function | ES6 class extending `Component` |
| State | `useState` hook | `this.state` |
| Lifecycle | `useEffect` hook | Lifecycle methods |
| `this` keyword | Not needed | Required |
| Modern standard | ✅ Yes | ❌ Legacy |

```jsx
// Functional (modern — use this)
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// Class (legacy)
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

> **Note:** Component names MUST start with a capital letter.

---

## Q5. What are Props? Why are they immutable?

**Answer:**
Props (properties) are read-only arguments passed from a parent component to a child component. They allow components to communicate.

```jsx
// Parent passes props
<UserCard name="Alice" age={28} isAdmin={true} />

// Child receives props
function UserCard({ name, age, isAdmin }) {
  return <div>{name} — {age} {isAdmin && '(Admin)'}</div>;
}
```

**Why immutable?** Props are immutable because React uses reference equality to detect changes. If you mutate props, React cannot track what changed, making data flow unpredictable and debugging hard. Always derive new values instead of mutating.

---

## Q6. What is State? How is it different from Props?

**Answer:**

| | Props | State |
|--|-------|-------|
| Owned by | Parent component | The component itself |
| Mutable? | No (read-only) | Yes (via setter) |
| Purpose | Pass data in | Internal data that changes |
| Triggers re-render? | When parent re-renders | Yes, when updated via setter |

```jsx
function Counter() {
  const [count, setCount] = useState(0); // state — internal, mutable

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  );
}
```

---

## Q7. What is useState? How does it work?

**Answer:**
`useState` is a hook that adds state to functional components. It returns an array with the current value and a setter function.

```jsx
const [state, setState] = useState(initialValue);
```

**Important behaviors:**
- State updates are **asynchronous** — the new value is available on the next render
- When new state depends on old state, use the **functional update form** to avoid stale closure bugs:

```jsx
// ❌ Can cause bugs (stale closure)
setCount(count + 1);

// ✅ Always correct
setCount(prev => prev + 1);
```

- Never mutate state directly — always return a new reference:

```jsx
// ❌ Wrong
state.name = 'Alice'; setState(state);

// ✅ Correct
setState({ ...state, name: 'Alice' });
```

---

## Q8. What is useEffect? Explain its dependency array.

**Answer:**
`useEffect` lets you perform side effects (API calls, subscriptions, timers, DOM manipulation) after the component renders.

```jsx
useEffect(() => {
  // effect code
  return () => { /* cleanup */ };
}, [/* dependencies */]);
```

**Three modes:**

```jsx
useEffect(fn)          // Runs after EVERY render
useEffect(fn, [])      // Runs ONCE after mount
useEffect(fn, [dep])   // Runs after mount + whenever dep changes
```

**Always return a cleanup function** to prevent memory leaks:

```jsx
useEffect(() => {
  const timer = setInterval(() => setCount(c => c + 1), 1000);
  return () => clearInterval(timer); // Cleans up on unmount
}, []);
```

---

## Q9. What are the Rules of Hooks?

**Answer:**
There are two strict rules:

**Rule 1: Only call hooks at the top level**
Never inside loops, conditions, or nested functions. React tracks hooks by their call order — changing the order breaks internal bookkeeping.

```jsx
// ❌ Wrong
if (isLoggedIn) {
  const [user, setUser] = useState(null);
}

// ✅ Correct
const [user, setUser] = useState(null);
if (isLoggedIn) { /* use it here */ }
```

**Rule 2: Only call hooks in React functions**
Only inside functional components or custom hooks (functions starting with `use`).

---

## Q10. What is useRef? How is it different from useState?

**Answer:**
`useRef` returns a mutable object `{ current: value }` that persists across renders **without triggering re-renders**.

| | useState | useRef |
|--|---------|--------|
| Triggers re-render | ✅ Yes | ❌ No |
| Persists across renders | ✅ Yes | ✅ Yes |
| Use for | UI data | DOM refs, timers, previous values |

```jsx
// Use 1: DOM reference
const inputRef = useRef(null);
<input ref={inputRef} />
inputRef.current.focus(); // Direct DOM access

// Use 2: Mutable value without re-render
const timerRef = useRef(null);
timerRef.current = setInterval(...);
clearInterval(timerRef.current);
```

---

## Q11. What is the difference between controlled and uncontrolled components?

**Answer:**

**Controlled:** React state is the single source of truth for the input's value. Every change goes through state.

```jsx
const [value, setValue] = useState('');
<input value={value} onChange={e => setValue(e.target.value)} />
```

**Uncontrolled:** The DOM manages its own value. You read it via a `ref` when needed.

```jsx
const inputRef = useRef();
<input ref={inputRef} defaultValue="initial" />
// Read: inputRef.current.value
```

**When to use which:**
- Controlled → real-time validation, programmatic value changes, most forms
- Uncontrolled → simple one-time reads, file inputs, integrating non-React code

---

## Q12. What is conditional rendering? List the methods.

**Answer:**
Conditional rendering means displaying different UI based on conditions. React supports multiple patterns:

```jsx
// 1. Early return (cleanest for loading/auth checks)
if (isLoading) return <Spinner />;

// 2. Ternary (choose between two options)
{isAdmin ? <AdminPanel /> : <UserPanel />}

// 3. Logical AND (show or nothing)
{hasError && <ErrorMessage />}

// 4. Return null (render nothing)
if (!isVisible) return null;
```

**⚠️ Common gotcha with `&&`:**
```jsx
// ❌ Renders "0" when count is 0
{count && <List />}

// ✅ Correct
{count > 0 && <List />}
```

---

## Q13. Why are Keys important in lists? What happens without them?

**Answer:**
Keys help React identify which list items have changed, been added, or removed during reconciliation. Without keys, React uses position to match items — causing wrong state, bugs, and unnecessary DOM updates.

```jsx
// ✅ Correct — stable unique ID
{users.map(user => <UserCard key={user.id} {...user} />)}

// ❌ Avoid — index shifts when list changes
{users.map((user, index) => <UserCard key={index} {...user} />)}
```

**When index as key is okay:** only for static lists that are never reordered, filtered, or inserted into.

---

## Q14. What is lifting state up?

**Answer:**
When two sibling components need to share state, you "lift" that state up to their closest common ancestor. The parent holds the state and passes it down via props, and passes callback functions for children to update it.

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

This maintains React's unidirectional data flow: data goes **down** via props, events go **up** via callbacks.

---

## Q15. What is the Context API? When should you use it?

**Answer:**
Context lets you share data across the component tree without passing props at every level (solving "props drilling").

```jsx
// 1. Create
const ThemeContext = createContext('light');

// 2. Provide
<ThemeContext.Provider value="dark">
  <App />
</ThemeContext.Provider>

// 3. Consume (anywhere in the tree)
const theme = useContext(ThemeContext);
```

**Use Context for:** theme, auth user, language/locale, user preferences — data that many components need but changes infrequently.

**Don't use Context for:** high-frequency updates (e.g. every keystroke) — it causes all consumers to re-render. Use Zustand or Redux for that.

---

## Q16. What is React.memo? How does it help performance?

**Answer:**
`React.memo` is a Higher-Order Component that wraps a component and skips re-rendering if its props haven't changed (using shallow comparison).

```jsx
// Without memo: re-renders every time parent re-renders
function ProductCard({ product, onAdd }) { ... }

// With memo: only re-renders when product or onAdd changes
const ProductCard = React.memo(function ProductCard({ product, onAdd }) { ... });
```

**Important:** Since `React.memo` uses shallow comparison (`===`), pass stable references for object/function props using `useMemo` and `useCallback` respectively, otherwise memo won't help.

---

## Q17. What is the difference between useMemo and useCallback?

**Answer:**

| | useMemo | useCallback |
|--|---------|-------------|
| Memoizes | A **computed value** | A **function reference** |
| Returns | The result of the function | The function itself |
| Use when | Expensive calculations | Callbacks passed to memoized children |

```jsx
// useMemo — cache expensive computation
const sortedList = useMemo(() => {
  return bigArray.sort(complexComparator);
}, [bigArray]);

// useCallback — cache function reference
const handleDelete = useCallback((id) => {
  deleteItem(id);
}, []);
```

Both only recalculate/recreate when their dependency array changes.

---

## Q18. What is prop drilling and how do you avoid it?

**Answer:**
Prop drilling is when data has to pass through many intermediate components that don't actually use it — just to reach a deeply nested child.

```jsx
// Drilling through Layout and Sidebar just to reach Avatar
function App() { return <Layout user={user} />; }
function Layout({ user }) { return <Sidebar user={user} />; }
function Sidebar({ user }) { return <Avatar src={user.avatar} />; }
```

**Solutions:**
1. **Context API** — for global data (auth, theme)
2. **Zustand / Redux** — for complex shared state
3. **Component composition** — restructure to avoid unnecessary nesting

---

## Q19. What are Custom Hooks? Why are they useful?

**Answer:**
Custom hooks are JavaScript functions that start with `use` and can call other hooks. They extract reusable stateful logic that can be shared between components — this was one of the main reasons hooks were introduced.

```jsx
// Custom hook — reusable fetch logic
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .catch(err => setError(err.message))
      .finally(() => setLoading(false));
  }, [url]);

  return { data, loading, error };
}

// Usage — clean and reusable
function UserProfile({ id }) {
  const { data, loading, error } = useFetch(`/api/users/${id}`);
  if (loading) return <Spinner />;
  return <Profile user={data} />;
}
```

---

## Q20. What are common mistakes/pitfalls in React?

**Answer:**
These are the most common bugs interviewers love to ask about:

**1. Mutating state directly**
```jsx
// ❌ No re-render
items.push(newItem); setItems(items);
// ✅ Create new reference
setItems(prev => [...prev, newItem]);
```

**2. Stale closure in async/timer**
```jsx
// ❌ count is always stale
setInterval(() => setCount(count + 1), 1000);
// ✅ Use functional form
setInterval(() => setCount(c => c + 1), 1000);
```

**3. `0 && <Component>` renders "0"**
```jsx
// ❌
{items.length && <List />}
// ✅
{items.length > 0 && <List />}
```

**4. Missing cleanup in useEffect**
```jsx
// ✅ Always clean up timers, subscriptions, listeners
useEffect(() => {
  const sub = subscribe(handler);
  return () => sub.unsubscribe();
}, []);
```

**5. Using index as key in dynamic lists**
```jsx
// ❌ Causes wrong state on reorder/filter
{items.map((item, i) => <Item key={i} />)}
// ✅
{items.map(item => <Item key={item.id} />)}
```

**6. Calling async directly in useEffect**
```jsx
// ❌ useEffect can't be async
useEffect(async () => { ... }, []);
// ✅ Define async inside
useEffect(() => {
  async function load() { ... }
  load();
}, []);
```

---

## Quick Revision Cheatsheet

```
Virtual DOM    → in-memory JS object; diffing = efficient real DOM updates
JSX            → compiles to React.createElement(); not HTML
Props          → read-only, passed from parent; immutable
State          → internal data; triggers re-render when updated
useState       → [value, setter]; use functional form for derived state
useEffect      → side effects after render; clean up with return fn
useRef         → mutable value; no re-render; DOM refs & timers
useMemo        → cache computed value
useCallback    → cache function reference
React.memo     → skip re-render if props unchanged (shallow compare)
useContext     → consume context without prop drilling
Custom hooks   → functions starting with 'use'; share stateful logic
Keys           → stable IDs for list items; never use index for dynamic lists
Lifting state  → move state to closest common ancestor for sharing
```

---

*Good luck with your interview! Build something, break something, fix it — that's React.*
