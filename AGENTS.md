# Agent Guidelines — docker_agent Project

Guidelines for AI agents editing this repository.

## Scope

Top-level repo orchestrates docker-agent in containerized stack. `docker-agent/` is an upstream Go project with own `AGENTS.md` — refer to that for code-level work.

## Project Map

| Path | Purpose |
|---|---|
| `docker-compose.yml` | Stack definition: agent + proxy + networks + secrets |
| `config/agent.yml` | Production agent config (mounted ro at /work/config/) |
| `proxy/squid.conf` | Squid ACL — add domains here to whitelist |
| `secrets/api_keys.txt` | API keys (gitignored) |
| `secrets/api_keys.example.txt` | Template for keys |
| `cagent/` | Runtime SQLite DBs + logs (gitignored) |
| `sandbox/` | Agent working directory (gitignored contents) |
| `docker-agent/` | Upstream git clone (gitignored) |

## Key Commands

```bash
git clone https://github.com/docker/docker-agent # If does not exist.
cd docker-agent/
task build        # compile Go binary
task test         # run test suite
task lint         # golangci-lint + custom linters
task build-image  # docker build -t localhost/dockeragent
```

## Conventions

- YAML: 2-space indent, no tabs
- Shell scripts: `set -euo pipefail`
- Comments: explain *why*, never *what*. Keep minimal.
- Agent YAMLs must validate against `docker-agent/agent-schema.json`
- Never commit real API keys. Only commit `secrets/api_keys.example.txt`.
