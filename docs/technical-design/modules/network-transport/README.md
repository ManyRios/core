# Network Transport

## Purpose

This module provides the shared connection layer used by Consumer, Relay, and Provider.
Although transport-only, it directly supports `Access` and `Automation` outcomes by keeping gateway connectivity and low-touch node operation reliable.

## Responsibility Boundary

- create WebSocket clients and servers
- serialize and send protocol messages
- detect liveness with ping and pong
- reconnect under policy

## Out Of Scope

- does not define business routing policy
- does not verify application signatures
- does not store persistent runtime records

## Interface

```ts
interface Connection {
  send(msg: WsMessage): void;
  close(): void;
  readonly readyState: 'connecting' | 'open' | 'closing' | 'closed';
}

function connect(options: ConnectionOptions): Promise<Connection>;
function createServer(
  options: { port: number; onConnection: (...) => void },
): ServerHandle;
```

## Data Flow

Input: `WsMessage` objects.  
Process: JSON serialization, socket send and receive, reconnect, heartbeat.  
Output: message callbacks and connection callbacks.

## State

- memory: WebSocket object, reconnect counter, ping timer, pong timeout
- persistence: none

## Errors

- first-connect failure
- invalid JSON frames
- pong timeout
- closed socket send

## Security Constraints

- bound message size
- drop dead peers aggressively
- never treat a closed socket as a successful send

## Test Requirements

- connect and reconnect
- heartbeat timeout
- invalid frame handling
- max payload behavior

## Dependencies

- calls: `config`, `logger`
- called by: `consumer`, `provider`, `relay`

---

## Implementation Details

**Source:** `src/network/index.ts`

### Key Data Structures

```ts
export interface Connection {
  ws: WebSocket;                    // underlying ws instance
  send(msg: WsMessage): void;       // JSON.stringify + send if OPEN
  close(): void;
  readonly readyState: 'connecting' | 'open' | 'closing' | 'closed';
}

export interface ConnectionOptions {
  url: string;
  onMessage: (msg: WsMessage) => void;
  onClose: (code: number, reason: string) => void;
  onError: (err: Error) => void;
  reconnect?: boolean;               // default true
  pingIntervalMs?: number;           // default PING_INTERVAL_MS
}
```

### Core Flow

**Client (`connect`)**:
1. Create WebSocket with `maxPayload: MAX_MESSAGE_SIZE` (10 MB)
2. On open: reset reconnect counter, setup ping interval
3. On message: `JSON.parse()` → `onMessage` callback
4. On close: cleanup timers, if `reconnect && !intentionallyClosed` → exponential backoff reconnect
5. Shared `conn` object with swappable `ws` — callers hold stable reference across reconnects

**Server (`createServer`)**:
1. Create `WebSocketServer` with `maxPayload: MAX_MESSAGE_SIZE`
2. Wrap each connection in `Connection` interface
3. Delegate to `onConnection` callback

### Heartbeat (Ping/Pong)

- Ping sent every `PING_INTERVAL_MS` (30s)
- If no pong within `PONG_TIMEOUT_MS` (10s) → `ws.terminate()`
- Pong handler clears timeout

### Reconnection

- Exponential backoff: `WS_RECONNECT_BASE_MS * 2^attempt` (1s base)
- Capped at `WS_RECONNECT_MAX_MS` (60s)
- `intentionallyClosed` flag prevents reconnect after `close()` call
- First connect failure rejects the promise; subsequent failures handled by `onClose`

### State Management

- **In-memory only**: WebSocket object, reconnect counter, ping timer, pong timeout
- State map: `WS_STATE_MAP` maps WebSocket readyState numbers to string union

### Error Handling

- Invalid JSON frames: caught and logged as `invalid_ws_message`
- Send on closed socket: silently dropped (checks `readyState === OPEN`)
- Pong timeout: `ws.terminate()` forces immediate close

## API Specification

### `connect(options: ConnectionOptions): Promise<Connection>`

Returns a stable `Connection` object. The underlying WebSocket is swapped on reconnect.

### `createServer(options: { port, onConnection }): { close(), port, address() }`

Creates a WebSocket server. Returns handle with port info.

### Constants (from `src/config/bootstrap.ts`)

| Constant | Value | Description |
|----------|-------|-------------|
| `PING_INTERVAL_MS` | 30,000 | Heartbeat interval |
| `PONG_TIMEOUT_MS` | 10,000 | Pong deadline |
| `WS_RECONNECT_BASE_MS` | 1,000 | Initial reconnect delay |
| `WS_RECONNECT_MAX_MS` | 60,000 | Max reconnect delay |
| `MAX_MESSAGE_SIZE` | 10 MB | Max WebSocket payload |

## Integration Protocol

- **Used by Consumer**: `connect()` to relay, receives `provider_list`, sends `request`
- **Used by Provider**: `connect()` to relay(s), sends `provider_hello`, handles `request`
- **Used by Relay**: `createServer()` to accept connections from both consumers and providers
- **Wire format**: `WsMessage` (JSON) with `type`, `request_id?`, `payload?`, `timestamp`
- **Dependencies**: `ws` npm package, `src/config/bootstrap.ts` for constants

## Current Implementation Status

- ✅ WebSocket client with auto-reconnect [IMPLEMENTED]
- ✅ WebSocket server with connection wrapping [IMPLEMENTED]
- ✅ Ping/pong heartbeat with configurable intervals [IMPLEMENTED]
- ✅ Exponential backoff reconnection [IMPLEMENTED]
- ✅ Max message size enforcement [IMPLEMENTED]
- ✅ Stable connection reference across reconnects [IMPLEMENTED]
- ❌ TLS certificate validation configuration [DESIGN ONLY]
- ❌ Connection multiplexing [DESIGN ONLY]
- ❌ Binary frame support (currently JSON only) [DESIGN ONLY]
