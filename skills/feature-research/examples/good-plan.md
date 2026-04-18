# Good Plan — Worked Example

The example below is the Plan body produced for the request *"Add real-time WebSocket notifications on top of the existing REST API"*. Aim for this level of specificity in Plan mode.

---

## Goal

Add a WebSocket endpoint that pushes real-time order-status notifications to the client, replacing the current 5-second polling to improve UX and reduce server load.

## Strategy

**Why WebSocket over SSE.**
- Upcoming work (read receipts, user presence) needs client → server messages; bidirectional transport is a better fit.
- `socket.io` is already present as a devDependency, so no new dependency is required.

**Auth approach.**
- Reuse the existing JWT. During the WebSocket handshake, pass the token as a query parameter on the first connection.
- Extract the JWT verification logic from `authMiddleware` into a pure `validateJWT(token): Promise<User>` so WebSocket can call it directly; keep `authMiddleware` as a thin wrapper for backward compatibility.

## Change Plan

| File | Change Type | Concrete Change |
|------|-------------|-----------------|
| `src/middleware/auth.ts` | modify | Extract JWT verification into `validateJWT(token): Promise<User>`; rewrite `authMiddleware` to call it |
| `src/websocket/handler.ts` | create | Exports `handleConnection(socket)`, `handleDisconnect(socket)`, `broadcastToUser(userId, event)` |
| `src/websocket/index.ts` | create | socket.io server bootstrap; registers auth middleware that uses `validateJWT` |
| `src/services/order.service.ts` | modify | At the end of `updateOrderStatus()`, emit `broadcastToUser(order.userId, { type: 'ORDER_STATUS', payload: order })` (1 line) |
| `src/index.ts` | modify | After creating the HTTP server, call `initWebSocket(server)` (2 lines) |
| `src/types/websocket.ts` | create | Define `WebSocketEvent` and `OrderStatusEvent` types |
| `test/websocket/handler.test.ts` | create | Unit tests for connection, auth failure, broadcast |

## Reuse

- `src/middleware/auth.ts:15-28` — JWT verification; extract as a pure function.
- `src/types/order.ts:OrderStatus` — import for the WebSocket event payload type.
- `src/services/order.service.ts:45` — append the broadcast call to the existing `updateOrderStatus()`.
- `test/helpers/auth.helper.ts:createTestToken()` — reused verbatim in the WebSocket tests.

## Task Order

### 1. Extract validateJWT as a pure function
- **Detail**: Refactor the logic at `auth.ts:15-28` into `validateJWT(token): Promise<User>`. Rewrite `authMiddleware` to call it; existing routes stay unchanged.
- **Depends on**: none

### 2. Define WebSocket event types
- **Detail**: Create `src/types/websocket.ts` with `WebSocketEvent` and `OrderStatusEvent`. Import `OrderStatus` from `types/order.ts` for the payload type.
- **Depends on**: none (parallel with Task 1)

### 3. Implement the WebSocket handler
- **Detail**: Create `src/websocket/handler.ts` exporting `handleConnection`, `handleDisconnect`, and `broadcastToUser`.
- **Depends on**: Tasks 1 and 2

### 4. Bootstrap the WebSocket server and attach to HTTP server
- **Detail**: Create `src/websocket/index.ts` initializing socket.io and registering the `validateJWT` auth middleware. In `src/index.ts`, call `initWebSocket(server)` (2 lines).
- **Depends on**: Task 3

### 5. Wire broadcast into the order service
- **Detail**: In `order.service.ts:updateOrderStatus()`, add `broadcastToUser(order.userId, { type: 'ORDER_STATUS', payload: order })` as the final statement.
- **Depends on**: Task 3 (parallel with Task 4)

### 6. Tests
- **Detail**: Create `test/websocket/handler.test.ts` covering connect, auth failure, and broadcast. Reuse `createTestToken()`.
- **Depends on**: all prior tasks

## External Notes

- socket.io v4 requires `createServer` before `listen`; the existing `app.listen()` pattern must change to `httpServer.listen()`.
- Passing the JWT as a query parameter can leak into server logs. This Plan uses query-param auth for the first delivery; a follow-up iteration should switch to a "send token as the first message after connect" flow.

---

## Why this plan is good

1. **Concrete.** Every change names the file, function, and the edit being made.
2. **Justified.** The Strategy section explains why WebSocket over SSE and why this auth approach.
3. **Reuse first.** The `Reuse` section lists exactly which existing code feeds each task.
4. **Directly executable.** Each `Task Order` entry maps 1:1 to TaskCreate. Parallel tasks are marked.
5. **Risk-aware.** External Notes flag both a migration gotcha (HTTP server creation) and a security caveat (JWT in query params) with a staged mitigation.
