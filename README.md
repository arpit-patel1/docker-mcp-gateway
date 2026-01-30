# docker-mcp-gateway

Compose file for [Docker MCP Gateway](https://github.com/docker/mcp-gateway/blob/main/docs/mcp-gateway.md).

## How to run

```bash
docker compose up
```

Or run in the background:

```bash
docker compose up -d
```

## Secrets (`.env`)

Secrets are managed via a `.env` file in the project root. The compose file mounts it into the gateway container as `/secrets/.env` and passes `--secrets=/secrets/.env` so the gateway can supply them to MCP servers/tools.

1. Copy the example and add your values:
   ```bash
   cp .env.example .env
   ```
2. Edit `.env` with `KEY=value` entries (one per line) as required by your enabled servers.

For fallback behavior (e.g. Docker Desktop then `.env`), see the [official gateway docs](https://github.com/docker/mcp-gateway/blob/main/docs/mcp-gateway.md) and the `--secrets` flag.

## Configuration

You can change enabled servers and other options by editing `compose.yaml`. The `command` section supports any [gateway CLI flags](https://github.com/docker/mcp-gateway/blob/main/docs/mcp-gateway.md) (e.g. `--servers`, `--tools`, `--verbose`).

## Authentication

When the gateway runs in a container, it **disables authentication** and logs "Authentication disabled (running in container)". There is no supported CLI flag or environment variable to re-enable auth in this setup; see [GitHub issue #243](https://github.com/docker/mcp-gateway/issues/243).

To protect the gateway when exposing it:

- **Reverse proxy with auth** – Put a proxy (e.g. Caddy, nginx, or Traefik) in front that requires an API key, basic auth, or OAuth, and forward only authenticated requests to the gateway. The gateway stays auth-disabled but is only reachable through the proxy.
- **Network isolation** – Do not publish the gateway port to the host; allow only other containers (e.g. your MCP client) on a private Docker network to reach it.
- **Track the issue** – Watch or comment on [docker/mcp-gateway#243](https://github.com/docker/mcp-gateway/issues/243) for any future option to enable auth when running in a container.
