# caddy

Reverse proxy for the tailnet, running on `hermes-agent` (the `oracle` ssh alias).

Serves `https://hermes-agent.rohu-nunki.ts.net/` using a certificate issued by
`tailscale cert`, renewed by a systemd timer. Caddy's own ACME client is not used
— a `.ts.net` name has no public DNS record, so Let's Encrypt cannot validate it
directly; Tailscale issues the cert instead.

Caddy binds to the tailnet address only, so nothing is published on the host's
public interface.

## Routes

Defined by `caddy_sites` in `defaults/main.yml`, served as path prefixes:

| Path       | Upstream                | What |
|------------|-------------------------|------|
| `/ollama`  | `100.125.231.39:11434`  | Ollama on titan |
| `/cockpit` | `100.125.231.39:9090`   | Cockpit on titan |

Add a route by appending to `caddy_sites`. `upstream` must be reachable *from*
hermes-agent — a tailnet peer, or `127.0.0.1:<port>` for a service on the box
itself. Set `host_header` when the backend validates the `Host` header.

## Run

    ansible-playbook playbooks/caddy.yml

Requires working SSH to hermes-agent. Tailscale SSH may demand a browser check
first; run `ssh oracle` once interactively and complete the auth URL.

## Caveats

- **Cockpit** rejects proxied requests unless the origin is allowlisted. On titan:
  `sudo mkdir -p /etc/systemd/system/cockpit.socket.d` and set
  `Origins = https://hermes-agent.rohu-nunki.ts.net` in cockpit.conf, or it will
  return "connection refused" through the proxy.
- Path-prefix routing strips the prefix (`handle_path`). Backends that generate
  absolute URLs may need subdomain routing instead — that requires extra MagicDNS
  names, which a single node does not get by default.