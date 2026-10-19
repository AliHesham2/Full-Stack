# Hook Patterns

## Custom hooks
Extract reusable logic — not just to reduce line count. Name must start with `use`.

```tsx
function useDebounce<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);        // mandatory cleanup
  }, [value, delay]);
  return debounced;
}
```

## Effect patterns (2026)
- Prefer event handlers over effects for user-triggered logic
- Use `useEffectEvent` (stable in React 19.2) for logic that reads the latest state but must not be a dependency:

```tsx
function ChatRoom({ roomId }: { roomId: string }) {
  const onMessage = useEffectEvent((message: string) => {
    // Reads latest props/state without becoming a dependency
    showNotification(message);
  });

  useEffect(() => {
    const connection = createConnection(roomId);
    connection.on('message', onMessage);     // ✓ allowed call site
    return () => connection.close();
  }, [roomId]);                              // onMessage intentionally omitted
}
```
**Constraints**: Call effect events only from inside effects (or handlers delegated by them) — never during render, never passed to other components. The linter enforces this.

## When NOT to use useEffect
- Data fetching → TanStack Query or RSC → [data-fetching.md](./data-fetching.md)
- Responding to user events → event handler
- Derived state → compute during render → [props-and-state.md](./props-and-state.md)

→ Sources: [react.dev/learn/reusing-logic-with-custom-hooks](https://react.dev/learn/reusing-logic-with-custom-hooks), [react.dev/reference/react/useEffectEvent](https://react.dev/reference/react/useEffectEvent)
