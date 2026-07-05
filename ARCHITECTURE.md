# Architecture

## Overview

```
┌───────────────────────────────────────────────────────┐
│                    Docker Host                         │
│                                                        │
│  ┌──────────────┐      ┌─────────────────────────┐    │
│  │  agent-proxy  │◄─────│         agent           │    │
│  │  (Squid)      │      │  (docker-agent binary)  │    │
│  │  port 3128    │      │                         │    │
│  │  proxy-net    │      │  /work/sandbox/    ◄────│────│──► host sandbox/
│  └──────┬───────┘      │  /work/config/     ◄────│────│──► host config/
│         │               │  /home/docker-agent/ ◄──│────│──► host cagent/
│         │               └─────────────────────────┘    │
│         ▼                                              │
│  .openrouter.ai     agent-net (internal, no internet)  │
│  .opencode.ai                                          │
└───────────────────────────────────────────────────────┘
```

## Services

### agent

- Go binary from `docker-agent/` (upstream: https://github.com/docker/docker-agent)
- Entrypoint: `docker-agent/main.go`
- AI agent runtime: TUI, MCP tools, multi-agent orchestration, A2A server, API server
- Runs as user `100:101`
- No auto-restart (interactive sessions)
- `tty: true` + `stdin_open: true` for TUI attach

### agent-proxy

- Ubuntu Squid container, restricts all outbound traffic
- Default whitelist: `.openrouter.ai`, `.opencode.ai`

## Networks

| Network | Driver | Purpose |
|---|---|---|
| `agent-net` | internal | Agent ↔ proxy, no internet |
| `proxy-net` | bridge | Proxy → internet |

Agent has no direct internet — all HTTP/HTTPS goes through proxy.

## Volumes

| Host | Container | Purpose |
|---|---|---|
| `./sandbox` | `/work/sandbox` | Agent working directory (rw) |
| `./config/` | `/work/config/` | Agent YAML configs (ro) |
| `./cagent/` | `/home/docker-agent/.cagent` | Runtime data: SQLite DBs, logs, history |

## Secrets

- `secrets/api_keys.txt` → Docker secret `/run/secrets/API_KEYS`
- Passed to agent via `--env-from-file /run/secrets/API_KEYS`
- Format: `KEY="value"` lines (shell-parseable)

## Agent Config

`config/agent.yml` (570 lines):

- Single agent `root` with cascading `first_available` model labels
- Providers: OpenRouter (15 free models) + OpenCode-Zen (4 free models)
- Toolsets: think, filesystem (sandbox-only), model_picker, memory (SQLite)

## Build

```bash
git clone https://github.com/docker/docker-agent
cd docker-agent/
task build        # compile Go binary → bin/docker-agent
task build-image  # Docker image → localhost/dockeragent:latest
task test         # go test ./...
task lint         # golangci-lint + custom linters
```

`Dockerfile` multi-stage: Go 1.26.4 Alpine builder → Alpine 3.22 runtime with docker-cli + mcp-gateway.

