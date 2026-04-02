# Proxy Configuration

## Purpose

The local proxy is an optional process that keeps upstream API keys outside the main Veil runtime.

It is designed for a narrow trust boundary:

- bind to `127.0.0.1`
- require a shared secret on every request
- keep the upstream API key only in proxy memory

## Runtime Inputs

- `ANTHROPIC_API_KEY`: required upstream credential
- `PROXY_SECRET`: shared secret checked through `x-proxy-secret`
- `PROXY_PORT`: local bind port, defaults to `4000`
- `UPSTREAM_URL`: upstream base URL, defaults to `https://api.anthropic.com`

## Interaction Model

1. start the proxy locally
2. capture or preconfigure `PROXY_SECRET`
3. point the Provider to `PROXY_URL`
4. forward `/v1/*` traffic through the proxy instead of sending the API key to the Provider process

## Scope

The proxy is a local isolation boundary. It is not a public API surface and should not be exposed on external interfaces.
