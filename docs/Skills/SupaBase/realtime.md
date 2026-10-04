# Realtime

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

> Project boundary: these are provider capabilities, not frontend permissions. All provider
> SDK calls, DB/Auth/Storage/Realtime integrations run in Express on Node.js for this project.
> Browser → Next.js → Express → Supabase. Provider-specific URLs/keys/tokens stay behind Express.
> The frontend uses same-origin relays; generic direct-client examples are not this project's flow.


Three features, one WebSocket server (Elixir/Phoenix):

| Feature | Purpose | Authorization |
|---------|---------|---------------|
| **Postgres Changes** | Listen to INSERT/UPDATE/DELETE on tables | RLS policies (since 2024) |
| **Broadcast** | Send arbitrary messages between clients | RLS authorization for explicitly private channels |
| **Presence** | Track shared state (cursors, online status) | RLS authorization for explicitly private channels |

Source: https://supabase.com/blog/supabase-realtime-broadcast-and-presence-authorization (Aug 2024)

## Postgres Changes

The flagship feature. Subscribes to Postgres WAL via logical replication.

```typescript
// Express only; REST headers do not authenticate the WebSocket.
await supabase.realtime.setAuth(userAccessToken)
const subscription = supabase
  .channel('table-changes')
  .on('postgres_changes',
    { event: '*', schema: 'public', table: 'todos' },
    (payload) => console.log(payload)
  )
  .subscribe()
```

### Scalability Consideration

- Each change to a table is authorized against **every subscriber**. One change with 100 subscribers = 100 RLS checks.
- For high-churn tables (many writes, many subscribers), this is expensive.
- Broadcast from Database is a separate trigger-based Broadcast approach, not an automatic upgrade of Postgres Changes. Evaluate it for high fan-out.

Source: https://supabase.com/blog/realtime-broadcast-from-database (Apr 2025)

### When to use vs not use

| Scenario | Recommendation |
|----------|---------------|
| Collaborative editing (few users per document) | Postgres Changes ✓ |
| Live feed (many readers, few writers) | Postgres Changes ✓ |
| High-churn analytics/counter tables | Polling or debounce instead |
| Multi-room chat (6+ tables per user) | Monitor performance; may need per-room channels |

Source: https://supabase.com/docs/guides/realtime/postgres-changes

## Broadcast

Application messaging, sent by clients or from database triggers. Useful for:
- Typing indicators
- Game state
- UI coordination

```typescript
await supabase.realtime.setAuth(userAccessToken)
const channel = supabase.channel('game-room', { config: { private: true } })
channel.subscribe((status) => {
  if (status === 'SUBSCRIBED') {
    channel.send({
      type: 'broadcast',
      event: 'cursor-move',
      payload: { x: 100, y: 200 },
    })
  }
})
```

## Presence

Track shared state with automatic join/leave detection.

```typescript
await supabase.realtime.setAuth(userAccessToken)
const channel = supabase.channel('room-1', { config: { private: true } })
channel.on('presence', { event: 'sync' }, () => {
  const state = channel.presenceState()
  console.log(state) // { user1: { x: 1, y: 2 }, user2: { x: 3, y: 4 } }
})
channel.subscribe(async (status) => {
  if (status === 'SUBSCRIBED') {
    await channel.track({ x: 0, y: 0 })
  }
})
```

### Operational limits (the ones you actually hit)

Two different limit systems get confused constantly. The **billing quota** is monthly
(2M messages/mo Free, 5M/mo Pro → [pricing.md](./pricing.md)). The **technical limits are
per-second and per-connection**, and they are what produce errors in production:

| Limit | Free | Pro | Pro (no spend cap) / Team |
|---|---|---|---|
| Concurrent connections | 200 | 500 | 10,000 |
| Messages **per second** | 100 | 500 | 2,500 |
| Channel joins **per second** | 100 | 500 | 2,500 |
| Channels per connection | 100 | 100 | 100 |
| Broadcast payload max | 256 KB | 3,000 KB | 3,000 KB; Enterprise custom |
| Postgres-changes payload max | 1,024 KB | 1,024 KB | 1,024 KB; Enterprise custom |

- Provider connection quotas count Express-to-Supabase connections in this architecture,
  not browser tabs directly. With separate USER clients and per-user WebSocket auth,
  budget roughly one provider socket per concurrently subscribed user if Express shares
  that user's client across their subscriptions. Separate request/session clients can
  create more sockets. This is a capacity estimate, not a provider guarantee; measure
  actual connections. Next/Express relay connection limits are separate.
- **Channel joins per second** bites on deploys and page reloads: everyone reconnecting at
  once is a join storm, independent of how few messages you send.
- When the Postgres Changes payload limit is reached, `new` and `old` retain only fields
  whose individual value size is at most 64 bytes, according to the provider documentation.
  Do not assume complete rows: select narrow columns where supported or broadcast an ID
  and re-fetch authorized state. Handle subscription/provider errors explicitly.

Source: https://supabase.com/docs/guides/realtime/limits

### Authorization, replay and delivery

Private Broadcast/Presence requires `config: { private: true }` and appropriate
`realtime.messages` policies for receive/send or Presence actions. Refresh the
WebSocket token after rotation and remove channels when sessions end. Relay only
authorized events through Express and Next.

Postgres Changes DELETE events do not apply row visibility policies like ordinary reads.
Current documentation describes DELETE filtering with `REPLICA IDENTITY FULL`;
review payload visibility before relaying and never assume SELECT policy protection.

Live delivery is not a durable business queue. Private database-origin Broadcast supports
bounded replay (up to 25 messages; nominal 72-hour retention, partition cleanup can extend
it to roughly four days). Persist required events in a durable outbox/queue, track
delivery and retry safely. `pg_net` or `waitUntil` alone does not supply this guarantee.

Sources: https://supabase.com/docs/guides/realtime/authorization
and https://supabase.com/docs/guides/realtime/postgres-changes
and https://supabase.com/docs/guides/realtime/broadcast

See [gotchas.md](./gotchas.md) for Realtime-specific pitfalls.

