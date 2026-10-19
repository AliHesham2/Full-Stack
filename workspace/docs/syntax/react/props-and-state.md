# Props and State

## Props

- Props are read-only; never mutate them
- React 19: `ref` and `children` are regular props — no special syntax
- Use ES6 default parameters instead of `defaultProps` (removed in React 19 on function components)

```tsx
function Button({ variant = 'primary' }: { variant?: 'primary' | 'ghost' }) {
  return <button className={variant}>Click</button>;
}
```

→ Source: [react.dev/learn/passing-props-to-a-component](https://react.dev/learn/passing-props-to-a-component)

## State

- `useState` for independent values; `useReducer` for related state transitions with multiple sub-values or complex update logic
- State is immutable; always create new references

```tsx
const [count, setCount] = useState(0);
const [form, setForm] = useReducer(reducer, initialState);
```

### Rule: Update functions must be pure — use the updater form when next state depends on previous
```tsx
setCount(count + 1);        // ✗ if multiple updates queue in one event
setCount(c => c + 1);       // ✓ queued correctly
```

### Rule: Derive during render instead of syncing state
```tsx
// ✗ Duplicate state + effect to sync
const [fullName, setFullName] = useState('');
useEffect(() => { setFullName(first + ' ' + last); }, [first, last]);

// ✓ Computed value — no state, no effect
const fullName = `${first} ${last}`;
```
**Why**: Syncing effects cause extra render passes and stale-state bugs.

### Rule: Keys reset state; identity resets components
Changing the `key` prop unmounts and remounts a subtree (state lost) — useful deliberately (`key={userId}` on profile pages).

→ Sources: [react.dev/learn/state-a-components-memory](https://react.dev/learn/state-a-components-memory), [react.dev/learn/you-might-not-need-an-effect](https://react.dev/learn/you-might-not-need-an-effect)
