# Realtime — WebSockets, SSE, and Live State

> **Part of:** Async & Scale — Backend Project Guide
> **Read when:** Server needs to push updates to clients: notifications, live dashboards, collaborative state, chat, progress.
> **Related:** `../03-auth/authorization.md`, `../06-async-and-scale/events-and-messaging.md`, `../07-reliability/reliability.md`
> **Status:** Stable decision framework · Library specifics in the realtime/framework guide

---

## 1. Transport Selection

| Transport | Fits | Constraints |
|---|---|---|
| **Polling** | Rare updates, simple clients, low connection counts | Wasteful at scale; latency = poll interval; fine for admin screens |
| **SSE** (server→client only) | Notifications, feeds, progress, dashboards — **the default for one-way push** | One-way; HTTP-native (proxies/CDNs mostly OK); reconnect semantics built into the browser EventSource |
| **WebSockets** | Bidirectional: chat, collaborative editing, games, high-frequency client→server messages | Connection lifecycle is YOURS: auth, heartbeat, reconnect, backpressure, scaling |
| **Long-lived proprietary (gRPC streams, MQTT)** | Service-to-service streaming, IoT | Ecosystem-specific |

Decision discipline: choose the **least powerful transport that meets the need** — SSE handles most "live updates to the UI" cases and skips the entire WebSocket lifecycle tax. WebSockets are for genuinely bidirectional traffic, not for avoiding one polling endpoint.

## 2. Connection Lifecycle (Where Realtime Systems Break)

Design each stage explicitly — this checklist IS the architecture:

1. **Authentication at connect:** the upgrade/first-connect request authenticates (cookie session or short-lived token in query/header — beware tokens in URLs: they leak into logs; prefer header/subprotocol). NO connection without verified identity; anonymous connections are an explicit, separately-justified pattern.
2. **Authorization per subscription, not per connection** (the #1 realtime vuln): authenticating once at connect, then letting the client subscribe to ANY channel (`subscribe: orders:*`) is BOLA over WebSockets (`../03-auth/authorization.md` §4). Every subscribe = server-side authorization decision on that exact topic scope.
3. **Heartbeat/ping-pong:** dead TCP connections (NAT timeouts, mobile sleep) look alive; heartbeat detects and reaps them (server-side timeout → disconnect).
4. **Reconnection with backoff + resumability:** clients reconnect with exponential backoff + jitter (thundering-herd on your own realtime layer otherwise); on reconnect, state must sync — either resumable streams (event IDs + replay-from-cursor, SSE-native pattern) or a state re-fetch. "Reconnect and hope" = missing-update bugs.
5. **Message delivery contract:** define ordering and at-least/at-most-once per channel; attach sequence numbers/cursors when the client must detect gaps; make gap-detection trigger a resync.
6. **Backpressure:** slow clients can't block the server (bounded per-connection send buffers; drop-policy per channel priority — a lagging dashboard socket drops old frames; a chat socket queues bounded).

## 3. Scaling Realtime

- **Sticky sessions or a pub-sub backplane:** with N server instances, the sender's instance isn't the receiver's instance — either route by connection affinity (sticky) or fan out through a shared bus (Redis pub/sub-class per the Redis guide / broker per `events-and-messaging.md`). Backplane is the standard answer beyond trivial scale.
- **Connection budget:** each open socket = memory + FD; per-instance caps + total caps + per-user connection caps (tab storms are real: one user, 40 tabs).
- **Fan-out cost analysis:** 10k subscribers × 1KB × 10 msg/s = your real egress bill; design payloads and fan-out structure knowing this multiplication (per-room topics vs per-user streams).
- Stateless business logic: realtime layer **transports**; decisions happen in use cases; the socket layer is an edge (`../01-foundation/architecture.md` §4.3) — no business rules inside the connection handler.

## 4. Security Review (Per Channel Design)

- [ ] Connect = authenticated; subscribe = authorized per topic scope (tenant/user/room)
- [ ] No tokens in URL query strings (log/proxy leakage — header or first-message auth)
- [ ] Message size + rate limits per connection (a websocket is a write endpoint — validate inbound frames like API input: `../02-domain-and-api/validation.md` applies fully)
- [ ] Origin checks on upgrade (cross-origin websocket attacks from attacker pages riding a logged-in session)
- [ ] No sensitive data in channel topics/names themselves (topic strings appear in logs)
- [ ] Disconnect on privilege change (permission revoked / logout ⇒ server closes or downgrades subscriptions — authorization is a moment-in-time decision; long-lived connections re-check)

## 5. Failure Modes

- **Connect-once-authorize-forever:** permissions change; socket stays subscribed for hours (§2.2, §4) — re-authorization on subscribe and on privilege change.
- **Silent dead connections:** no heartbeat; dashboards show stale "live" data; memory fills with zombie sockets.
- **Reconnect stampede:** deploy restarts all instances; every client reconnects at t+0 with no jitter; the second outage is self-inflicted (§2.4).
- **Gap-blind clients:** reconnect resubscribes without resync; user misses the update that happened while disconnected and never learns (sequence cursors, §2.5).
- **The room key that knew too much:** `subscribe("user:1234")` accepted for any authenticated user — subscription topics are authorization objects, treat them like URLs (per-object checks).
- **Business logic in handlers:** rules accrete inside `onMessage` because "it's just the chat socket" — extract to use cases; the socket translates.
- **Single-instance ceiling:** in-memory pub/sub on one box; instance #2 deploys; users on different instances can't see each other's messages — backplane from the start of multi-instance life (§3).
