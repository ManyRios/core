# OpenAI-Compatible Surface

## Supported Endpoints

Current local gateway endpoints:

- `GET /health`
- `GET /v1/models`
- `POST /v1/chat/completions`

## Request Shape

The gateway currently accepts the common chat-completions fields:

- `model`
- `messages`
- `stream`
- `temperature`
- `max_tokens`
- `top_p`
- `stop`

It also accepts a Veil-specific extension in runtime handling:

```json
{
  "budget": {
    "max_cost_usdc": 1.0,
    "max_duration_ms": 120000
  }
}
```

`max_cost_usdc` is currently a quote-budget field for routing and comparison, not a declaration of final settlement asset semantics.

## Authentication

When local API auth is enabled, callers must send:

```http
Authorization: Bearer <token>
```

## Error Semantics

Current gateway error shapes follow an OpenAI-style envelope:

```json
{
  "error": {
    "message": "No providers available",
    "type": "api_error",
    "code": "no_providers"
  }
}
```

## Current Non-Goals

The gateway does not currently expose:

- embeddings
- responses API
- assistants
- images
- file APIs
