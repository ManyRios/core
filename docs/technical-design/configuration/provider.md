# Provider Configuration

## Local File

The Provider runtime reads `provider.json`.

Validated fields:

```json
{
  "version": 1,
  "models": ["claude-sonnet-4-20250514"],
  "api_keys": [
    {
      "provider": "anthropic",
      "salt": "...",
      "iv": "...",
      "ciphertext": "...",
      "tag": "..."
    }
  ],
  "max_concurrent": 5
}
```

## Field Semantics

- `version`: must currently be `1`
- `models`: non-empty list of exposed model ids
- `api_keys`: encrypted upstream credentials
- `max_concurrent`: optional positive integer

## Why This Config Exists

- Provider config turns supply-side intent into executable market capacity instead of background infrastructure.
- encrypted credentials and concurrency limits keep execution inside clear security and risk boundaries.
- discovery and Relay endpoint settings preserve market reachability while keeping policy-driven publication possible for automation paths.

## Runtime Inputs

The Provider path also reads:

- `relay_mode`: whether the node uses Bootstrap discovery, static Relay endpoints, or operator-provided Relay lists
- `bootstrap_url`: Bootstrap endpoint when discovery is enabled
- `relay_url` or `relay_urls`: static fallback Relay endpoint(s)
- `ANTHROPIC_API_KEY`: fallback credential if no decrypted key is usable
- `PROXY_URL`: optional upstream proxy base URL
- `PROXY_SECRET`: optional shared secret for proxy requests
- `VEIL_PROVIDER_HEALTH_PORT`: override the default health port

## Model Mapping

Provider-visible model ids are mapped internally through `MODEL_MAP` before calling the upstream AI API.
