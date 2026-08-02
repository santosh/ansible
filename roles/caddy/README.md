# caddy

Reverse proxy for the tailnet, running on `hermes-agent` (the `oracle` ssh alias).

Each site in `caddy_vhosts` gets its own public fqdn with a cert issued via
Cloudflare DNS-01 (no port 80/443 inbound needed to validate). The domains
resolve publicly, but Caddy binds only to the tailnet address
(`caddy_bind_address`), so nothing is actually reachable off the tailnet.

## Routes

Defined by `caddy_vhosts` in `defaults/main.yml`:

| Fqdn                   | Upstream              | What               |
|------------------------|-----------------------|--------------------|
| `vikunja.muresine.top` | `100.118.241.39:4444` | Vikunja on voyager |

Every service gets its own subdomain and is proxied whole — there is no
path-prefix routing. Add a service by appending an `fqdn`/`upstream` pair to
`caddy_vhosts`.

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

- Backends that check the `Host` header (Ollama, for one) reject the public
  fqdn — set `host_header` on that vhost to whatever the backend expects.
- Some backends also refuse proxied requests unless the origin is allowlisted.
  Cockpit is the usual example: it needs `Origins = https://<fqdn>` under
  `/etc/systemd/system/cockpit.socket.d`, or it returns "connection refused"
  through the proxy.