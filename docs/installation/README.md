# Installation

## Purpose

This section explains how to install Veil from source and prepare a local runtime directory for first use.

## Read This Section If

- you are setting up Veil for the first time
- you need the required runtime prerequisites
- you want the default local file layout before running any role

## Requirements

- Node.js `22+`
- `npm`
- network access for upstream model APIs when running a Provider
- a writable local home directory or a custom `VEIL_HOME`

## Install From Source

```bash
git clone https://github.com/runveil-io/core.git
cd core
npm install
npm test
```

## Local Runtime Layout

By default, Veil stores local runtime files under `~/.veil`.

Typical files:

- `wallet.json`
- `config.json`
- `provider.json`
- `data/*.db`

You can override the base directory with:

```bash
export VEIL_HOME=/path/to/veil-home
```

## First-Time Setup

Initialize a wallet:

```bash
veil init
```

This creates:

- a signing keypair
- an encryption keypair
- local config for the gateway and Relay connection

## Run As Consumer

Use the local gateway with an OpenAI-compatible client:

```bash
veil init
# point your client at http://localhost:9960/v1
```

Default gateway port: `9960`

## Run As Provider

Configure and start a Provider:

```bash
veil provide init
veil provide start
```

Default Provider health port: `9962`

## Run As Relay

Start a Relay node:

```bash
veil relay start
```

Default Relay port: `8080`

## Verification

Run the test suite:

```bash
npm test
```

For more runtime usage, continue to the [manual](../manual/README.md).
