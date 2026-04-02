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
