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

## Connecting to the gateway

Use the gateway URL that matches where your client runs:

- **From the host machine:** `http://localhost:8811/mcp`
- **From other containers or services:** `http://host.docker.internal:8811/mcp`

`host.docker.internal` resolves to the host from inside a container, so MCP clients running in containers (or pointing at the host) can reach the gateway. This works on Docker Desktop for Mac and Windows; on Linux you may need to add `extra_hosts: ["host.docker.internal:host-gateway"]` to the client service in Compose.

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

To **enable authentication**, run the gateway from the Docker CLI on the host (not in a container):

```bash
docker mcp gateway run --transport streaming
```

When auth is enabled, the gateway logs output like:

```
  > mcp-discover: prompt for learning about dynamic server management
- Starting OAuth notification monitor
- Starting OAuth provider loops...
- Watching for configuration updates...
> Initialized in 1.015867167s
> Start streaming server on port 8811
> Gateway URL: http://localhost:8811/mcp
> Use Bearer token: Authorization: Bearer 7mvy2h6cyh0rz0ptwlkjnwj4njagcjia05p9za9mqzwy2h6mhq
- Connecting to OAuth notification stream at http://localhost/notify/notifications/channel/external-oauth
```

Use the Bearer token in the `Authorization` header when connecting MCP clients to the gateway URL.

When the gateway runs **in a container** (e.g. via `docker compose up`), it disables authentication and logs "Authentication disabled (running in container)". There is no supported way to re-enable auth inside the container; see [GitHub issue #243](https://github.com/docker/mcp-gateway/issues/243).

To protect the gateway when running it in a container:

- **Reverse proxy with auth** – Put a proxy (e.g. Caddy, nginx, or Traefik) in front that requires an API key, basic auth, or OAuth, and forward only authenticated requests to the gateway. The gateway stays auth-disabled but is only reachable through the proxy.
- **Network isolation** – Do not publish the gateway port to the host; allow only other containers (e.g. your MCP client) on a private Docker network to reach it.
- **Track the issue** – Watch or comment on [docker/mcp-gateway#243](https://github.com/docker/mcp-gateway/issues/243) for any future option to enable auth when running in a container.
