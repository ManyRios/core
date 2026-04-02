# Bootstrap Configuration

## Purpose

Bootstrap is the Relay discovery service.

## Default Values

- bootstrap URL: `https://bootstrap.runveil.io`
- discovery cache TTL: `60000 ms`
- discovery max relays: `20`

## Discovery Client Inputs

`RelayDiscoveryClient` accepts:

- `bootstrapUrl`
- `cacheTtlMs`
- `maxRelays`

## Selection Inputs

Relay selection currently scores using:

- latency
- fee
- uptime
- reputation
- exploration bonus

## API Surface

Bootstrap exposes:

- `POST /v1/relays/register`
- `POST /v1/relays/heartbeat`
- `GET /v1/relays`
- `DELETE /v1/relays/:pubkey`
- `GET /health`
