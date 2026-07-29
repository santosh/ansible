# caddy

Reverse proxy for the tailnet, running on `hermes-agent` (the `oracle` ssh alias).

Each site in `caddy_vhosts` gets its own public fqdn with a cert issued via
Cloudflare DNS-01 (no port 80/443 inbound needed to validate). The domains
resolve publicly, but Caddy binds only to the tailnet address
(`caddy_bind_address`), so nothing is actually reachable off the tailnet.

## Routes

Defined by `caddy_vhosts` in `defaults/main.yml`:

| Fqdn                        | Routing      | Path       | Upstream               | What               |
|-----------------------------|--------------|------------|------------------------|--------------------|
| `hermes-agent.muresine.top` | path prefix  | `/ollama`  | `100.125.231.39:11434` | Ollama on titan    |
| `hermes-agent.muresine.top` | path prefix  | `/cockpit` | `100.125.231.39:9090`  | Cockpit on titan   |
| `vikunja.muresine.top`      | whole domain | —          | `100.118.241.39:4444`  | Vikunja on voyager |

A vhost with `sites` gets path-prefix routing (`handle_path`, prefix stripped
before proxying) — add a route by appending to that vhost's `sites` list. A
vhost with a bare `upstream` proxies the whole domain instead; use this for
backends that build absolute URLs, since path-prefix stripping breaks those.

`upstream` must be reachable *from* hermes-agent — a tailnet peer, or
`127.0.0.1:<port>` for a service on the box itself. Set `host_header` when the
backend validates the `Host` header. Adding a new fqdn also needs a DNS A/AAAA
record for it pointing at hermes-agent's tailnet IP (Cloudflare, out-of-band —
not managed by this role), since Caddy only handles TLS/proxying, not the
record itself.

## Run

    ansible-playbook playbooks/caddy.yml

Requires working SSH to hermes-agent. Tailscale SSH may demand a browser check
first; run `ssh oracle` once interactively and complete the auth URL.

## Caveats

- **Cockpit** rejects proxied requests unless the origin is allowlisted. On titan:
  `sudo mkdir -p /etc/systemd/system/cockpit.socket.d` and set
  `Origins = https://hermes-agent.muresine.top` in cockpit.conf, or it will
  return "connection refused" through the proxy.
- Path-prefix routing strips the prefix (`handle_path`), which breaks backends
  that generate absolute URLs. Give those their own fqdn with a bare `upstream`
  instead — the certs come from Cloudflare DNS-01, so a new subdomain only needs
  a DNS record, not a MagicDNS name.