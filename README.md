# Docker Agent — Containerized AI Agent Stack

Orchestrated environment for running [docker-agent](https://github.com/docker/docker-agent) with proxy-restricted networking, sandboxed execution, and configurable AI backends.

## Quickstart

```bash
# 1. Clone
cd docker_agent/

# 2. Install go-task (if not installed)
#   macOS:  brew install go-task
#   Linux:  sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b ~/.local/bin
#   Arch:   pacman -S go-task
#   See:    https://taskfile.dev/installation/

# 3. Build Docker image
git clone https://github.com/docker/docker-agent
cd docker-agent/

# Edit the Taskfile and change the name of the image to `localhost/dockeragent:latest`.
task build-image     # builds localhost/dockeragent:latest
cd ..

# 4. Set API keys
#    Edit secrets/api_keys.txt with your keys (see secrets/api_keys.example.txt)

# 5. Run agent
docker compose -f docker-compose.yml up -d       # start agent + proxy
docker attach agent                              # attach to TUI
docker compose run --rm agent  # run ad-hoc session
docker compose -f docker-compose.yml logs -f     # tail logs
```

## Permission problem

If the agent is not able to write something, you need to edit permissions:

```bash
chown -R 100:101 sandbox/
chown -R 100:101 cagent/
```

## Proxy Whitelist

Outbound traffic restricted via Squid proxy. Edit `proxy/squid.conf`:

```conf
# Add domain to whitelist
acl allowed_domains dstdomain .example.com

# Add port if non-standard
acl Safe_ports port 8080
```

Then restart proxy:

```bash
docker compose -f docker-compose.yml restart agent-proxy
```

## Secrets

API keys stored in `secrets/api_keys.txt` (git-ignored). Format:

```
OPENCODE_API_KEY="sk-..."
OPENROUTER_API_KEY="sk-or-v1-..."
```

## TODO — Sandbox Investigation

- [ ] **NVIDIA OpenShell** — microVM sandbox for containerized workloads. Does not support docker-agent. https://github.com/NVIDIA/OpenShell/tree/main
- [ ] **k8s Agent Sandbox** — Kubernetes sandbox agent. Still in development. https://github.com/kubernetes-sigs/agent-sandbox/tree/main
- [ ] **Docker Sandbox (sbx)** — Proprietary microVM sandbox (nerdbox-based). Binaries in `docker-sbx/`. Not usable on Arch Linux. https://github.com/docker/sbx-releases

## Links

- [docker-agent docs — secrets guide](https://docs.docker.com/ai/docker-agent/guides/secrets/)
- [docker-agent repo](https://github.com/docker/docker-agent)
- [OpenShell](https://github.com/NVIDIA/OpenShell/tree/main)
- [k8s Agent Sandbox](https://github.com/kubernetes-sigs/agent-sandbox/tree/main)
- [Docker Sandbox Releases](https://github.com/docker/sbx-releases)
- [go-task](https://taskfile.dev/)

### EOF

