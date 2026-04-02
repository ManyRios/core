# Consumer Configuration

## Local File

The Consumer runtime reads `config.json`.

Validated fields:

```json
{
  "relay_url": "wss://relay-jp.runveil.io",
  "relay_discovery": "bootstrap",
  "bootstrap_url": "https://bootstrap.runveil.io",
  "gateway_port": 9960,
  "consumer_pubkey": "<64 hex chars>",
  "encryption_pubkey": "<64 hex chars>"
}
```

## Field Semantics

- `relay_url`: static Relay fallback URL
- `relay_discovery`: preferred Relay acquisition mode
- `bootstrap_url`: Bootstrap endpoint used when `relay_discovery` is `bootstrap`
- `gateway_port`: required local HTTP port
- `consumer_pubkey`: required signing public key
- `encryption_pubkey`: required encryption public key

## Why This Config Exists

- `relay_discovery`, `relay_url`, and `bootstrap_url` keep accountless access routable without coupling clients to a single hard-coded operator.
- identity keys preserve signature and encryption boundaries so routing metadata and prompt plaintext remain split across roles.
- quote-related fields tune access-time budgeting and must remain separate from settlement asset semantics.

## Additional Runtime Fields

The CLI and gateway also read optional values when present:

- `default_quote_budget`
- `quote_unit`

These values refine runtime behavior and quote budgeting. They must not be interpreted as final settlement outputs.

## Runtime Inputs

- `VEIL_HOME` controls where `config.json` is loaded from
- `VEIL_PASSWORD` can bypass interactive password prompts

## API Auth

The gateway can enforce a local API key through the `apiKey` runtime option. When enabled, callers must send:

```http
Authorization: Bearer <token>
```
