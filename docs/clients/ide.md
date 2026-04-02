# IDE and Tooling

## Base URL

For OpenAI-compatible IDEs and tools, use:

```text
http://localhost:9960/v1
```

## Typical Flow

1. start the local gateway
2. configure the IDE base URL
3. set the local token if API auth is enabled
4. choose one of the exposed Veil model ids

## Compatibility Goal

The client-facing goal is to make Veil look like a local AI provider, not a custom protocol endpoint.
This compatibility surface is an entrypoint into Veil's market network, where Relay and Provider roles still define routing and supply behavior.

For trust and role ownership behind this local surface, see [System Model](../technical-design/system-model/README.md).
